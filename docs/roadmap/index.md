# DRUCS Unified Platform Modernization Roadmap

**Author / Autore:** Walter Divisi — Dropsa S.p.A.
**Date / Data:** February 2026 / *Febbraio 2026*
**Classification / Classificazione:** Internal / *Interno*

---

## Current Status (March 2026) / Stato Attuale (Marzo 2026)

| Phase / *Fase* | Status / *Stato* | Description / *Descrizione* |
|---|---|---|
| **Phase 0** | **DONE** | *Security audit + access lockdown / Audit sicurezza + blocco accessi* |
| **Phase 1** | **IN PROGRESS** | *Web v2 dashboard (React) / Dashboard web v2 (React)* |
| **Active Fixes** | **ONGOING** | *Parallel bug-fix track / Track bug fix parallelo* |
| **Phase 2** | Planned / *Pianificata* | *Infrastructure & cloud scalability / Infrastruttura e scalabilita' cloud* |
| **Phase 3** | Planned / *Pianificata* | *Protocol bridge — legacy NestJS bridge replaces Java gateway / Bridge protocollo* |
| **Phase 4** | Planned / *Pianificata* | *Core API modernization — NestJS replaces import-web-app / Modernizzazione API core* |
| **Phase 5** | Planned / *Pianificata* | *MQTT for new devices + firmware modernization / MQTT per nuovi dispositivi + firmware* |
| **Phase 6** | Future / *Futuro* | *Mobile app sync + generic dashboard / Sincronizzazione app mobile + dashboard generica* |
| **Phase 7** | Future / *Futuro* | *Advanced analytics + AI / Analisi avanzata + AI* |

---

## Background: What's Wrong / Contesto: I Problemi Attuali

### 1. The Protocol is Fragile and Inefficient / Il Protocollo e' Fragile e Inefficiente

HTTP/1.0 polling every 5 seconds means 17,280 connections per device per day. Each cycle requires DNS lookup + TCP handshake + HTTP request + wait + response + close. Commands from server to device have up to 5-second latency. If the server is down, telemetry is lost with no retry or QoS guarantee.

The AES-128-CFB encryption uses a zero IV — a known cryptographic weakness that allows pattern analysis on the first block.

*Il polling HTTP/1.0 ogni 5 secondi comporta 17.280 connessioni per dispositivo al giorno. Nessuna garanzia QoS, nessun retry. La crittografia AES-128-CFB usa un IV a zero — debolezza crittografica nota.*

### 2. The Java Gateway is a Bottleneck / Il Gateway Java e' un Collo di Bottiglia

Each device's HTTP POST is processed synchronously: decrypt → parse → DB query → build response → encrypt. With 500 devices at 5s heartbeat = 100 req/s. With 5,000 devices it breaks. The gateway repo has been inactive for 7 months.

### 3. The PHP API Cannot Scale Horizontally / L'API PHP Non Scala Orizzontalmente

`import-web-app` (the only production API, used by web and mobile) is based on stateless JWT with a Redis blacklist — authentication scales horizontally without issues. The real bottleneck is **connection pooling**: every HTTP request opens a new PDO connection to MariaDB. With multiple ECS tasks running in parallel, this rapidly exhausts the database's available connections. PHP sessions use the file-system, but this does not affect API routes (which are JWT-stateless); it only affects web routes if any are present.

### 4. The Database is a Shared Bottleneck / Il Database e' un Collo di Bottiglia Condiviso

All services — gateway, PHP API, 5 SQS consumers — read/write the same MariaDB instance. No read replicas, no query isolation, no time-series optimization for telemetry.

### 5. No Real-Time Push to Clients / Nessun Push in Tempo Reale ai Client

The web portal uses basic SSE. No WebSockets. Users cannot see live device telemetry.

### 6. Vendor Lock-In / Dipendenza dal Fornitore

Capaciteam's development velocity has collapsed 95% since Q1 2024. Only 3 Jira tickets active in February 2026. 23 unmerged branches. New features require either expensive external engagement or internal capability.

---

## Vision / La Visione

The goal is a **generic IoT platform** that: / *L'obiettivo e' una piattaforma IoT generica che:*

1. Works with existing DRUCS devices (backwards compatible with the HTTP/AES protocol)
2. Supports modern protocols (MQTT, WebSocket) for new devices
3. Is generic enough to reuse for other Dropsa electronics projects (not just lubrication)
4. Can be developed and maintained internally with Claude Code agentic assistance
5. Scales from hundreds to tens of thousands of devices without architecture changes
6. Provides real-time dashboards with live telemetry streaming

**Guiding Principles / Principi Guida:**

1. **Never break existing devices.** Every deployed STM32 device must continue working throughout the migration.
2. **Protocol bridge first.** Build a compatibility layer before changing anything on the device side.
3. **New devices get new protocol.** New firmware releases support MQTT alongside legacy HTTP.
4. **Incremental migration.** Devices upgrade via OTA at their own pace.
5. **Generic from day one.** Every new component must be device-type agnostic.

---

## Performance Architecture / Architettura Performance

Analysis of the six critical performance bottlenecks in the current system, with file-level attribution and the target architecture.

*Analisi dei sei colli di bottiglia critici nel sistema attuale, con attribuzione a livello di file e l'architettura target.*

### Bottlenecks / Colli di Bottiglia

#### Critical (immediate impact) / Critici (impatto immediato)

**A. Missing composite index on `events` table**
- File: `database/migrations/2019_09_03_092837_create_events_table.php`
- `SELECT WHERE dev_id=? AND evt_time > ?` → full table scan on ~500M rows
- Fix: `CREATE INDEX idx_events_device_time ON events (dev_id, evt_time DESC)` — 5 minutes

**B. N+1 queries on event relations**
- File: `GetDeviceEventsWithRelationsCriteria.php` (lines 24–32)
- 100 events × 4–8 relations = 400–800 queries per single dashboard API call
- Fix: DataLoader / eager loading in NestJS (Phase 4)

**C. Synchronous gateway with 150+ queries per heartbeat**
- Files: `DataProcessorServiceImpl.java` + `ValueServiceImpl.java`
- 5s heartbeat × 50 variables = 150+ synchronous DB queries → DB exhaustion at 500+ devices
- Fix: async SQS batch processor in NestJS bridge (Phase 3)

**D. SSE polling every 10 seconds**
- File: `SseStreamController.php` (line 119: `sleep(10)`)
- 1,000 clients × 5 devices × 6 polls/min = 30,000 queries/minute with no new data
- Fix: WebSocket event-driven push (Phase 4)

#### High / Alti

**E. Read-Modify-Write for device values** — 50 SELECT + 50 UPDATE per heartbeat, no batch UPSERT

**F. Sequential batch jobs** — `CalculateDeviceStatusStatisticJob.php` runs sequentially across 1,000 devices

### Before / After — Architecture / Architettura Prima/Dopo

```
BEFORE (2014 architecture):
  Device → HTTP sync POST → Gateway → 150+ DB queries → response → repeat every 5s
  Client → SSE → PHP polls DB every 10s → repeat per client

AFTER (Phase 3–4):
  Device → HTTP POST → NestJS Bridge
                         ├─ AES decrypt (sync)
                         ├─ Publish to SQS (async, <1ms)
                         └─ ACK immediately (device does not wait for DB)

  SQS "device.telemetry":
    → Batch Processor  (1s window, bulk INSERT TimescaleDB)
    → State Updater    (Redis HSET device:{id} — no read-modify-write)
    → WebSocket Emitter (push to clients)
```

### Fix Strategy by Phase

| Phase | Fix | Impact |
|---|---|---|
| **Phase 2 (immediate)** | Composite index `(dev_id, evt_time)` on `events` | Eliminates full table scan on ~500M rows — critical |
| **Phase 2 (immediate)** | Redis cache for device variable values (30min TTL) | Eliminates repeated DB reads per dashboard refresh |
| **Phase 3** | Async SQS dispatch from NestJS bridge | Device ACK in <5ms, DB writes decoupled |
| **Phase 3** | Redis HSET for device current state | Replaces SELECT+UPDATE per variable |
| **Phase 4** | WebSocket event-driven (Socket.IO) | Replaces SSE polling — eliminates 30K queries/min at 1K clients |
| **Phase 4** | DataLoader pattern on all relations | Eliminates N+1: 400–800 queries → 1 batch per relation |
| **Phase 4** | TimescaleDB hypertable `(device_id, time)` | 10–100x faster time-range queries vs MariaDB |

---

## Phase 0: Security & Cleanup (COMPLETE) / Sicurezza e Pulizia (COMPLETATA)

*Effort: 1–2 days / Impegno: 1–2 giorni*

- [x] GitHub org audit — downgraded vendor access / *Audit org GitHub — declassato accesso fornitore*
- [x] AWS IAM audit — deactivated unused keys / *Audit AWS IAM — disattivate chiavi non usate*
- [x] Org default permissions set to read-only / *Permessi org predefiniti impostati a sola lettura*
- [x] GitHub Teams created (Project Admins, Electronics, Management, Field Engineers)
- [x] Firmware repo migrated to DropsA-SpA org / *Repo firmware migrato nell'org DropsA-SpA*
- [ ] Branch cleanup — 163 stale branches to delete / *Pulizia branch — 163 branch obsoleti da eliminare*

**Remaining security items / Elementi sicurezza rimanenti:**

- [ ] **[SEC]** Store JWT secret in AWS SSM Parameter Store (not hardcoded in config) / *Conservare il segreto JWT in SSM*
- [ ] **[SEC]** Rotate AWS IAM key found hardcoded in `drucs-web-services/login.php` — audit CloudTrail / *Ruotare la chiave AWS hardcoded*
- [ ] **[SEC]** Generate random IV for AES-CFB on device (not all-zeros) + gateway update to read IV from payload / *IV casuale per AES-CFB*
- [ ] **[SEC]** Restrict Spring Boot actuator endpoints / *Limitare gli endpoint Spring Boot actuator*
- [ ] **[SEC]** Add WAF rate limiting on ALB / *Aggiungere WAF rate limiting sull'ALB*
- [ ] **[SEC]** Enable Redis encryption at rest + transit (Terraform change) / *Abilitare crittografia Redis*

---

## Phase 1: Web v2 Dashboard (IN PROGRESS) / Dashboard Web v2 (IN CORSO)

The new React-based dashboard replacing the Angular v1 frontend. Consumes the existing `import-web-app` API (v1 endpoints).

*La nuova dashboard basata su React che sostituisce il frontend Angular v1. Usa l'API `import-web-app` esistente (endpoint v1).*

> **Note / Nota:** The current dashboard uses polling (variables refreshed every 60s via `import-web-app`). Live WebSocket streaming will be available in Phase 4. / *La dashboard attuale usa il polling. Lo streaming WebSocket live sara' disponibile in Fase 4.*

### Completed / Completato

- [x] Dark theme glass UI with readable text / *UI vetro tema scuro con testo leggibile*
- [x] Clickable summary strip (Total/OK/Alarm/Warning/Inactive/Offline filters) / *Barra riepilogo cliccabile con filtri*
- [x] Full-width device card list with status, level gauge, cycle counters / *Lista schede dispositivo a larghezza piena*
- [x] Problems panel (alarm >2h, offline >24h detection) / *Pannello problemi (allarme >2h, offline >24h)*
- [x] Predictive LubeOut (estimates when lubricant runs out) / *LubeOut predittivo (stima esaurimento lubrificante)*
- [x] Status bar chart (7-day trend) / *Grafico barre stato (trend 7 giorni)*
- [x] Hours/cycles metrics with VipAir micropump aggregation / *Metriche ore/cicli con aggregazione micropompe VipAir*
- [x] Deployed to app.dropsa.app/v2/ / *Distribuito su app.dropsa.app/v2/*

### Next / Prossimo

- [ ] Device detail page — full variable display, settings editor, event history / *Pagina dettaglio dispositivo*
- [ ] Per-user "Inactive" device flag / *Flag dispositivo "Inattivo" per utente*
- [ ] Notifications panel (in-app) / *Pannello notifiche (in-app)*
- [ ] Help page integration (embed from this docs site) / *Integrazione pagine aiuto*

---

## Active Fixes (Parallel Track) / Fix Attivi (Track Parallelo)

These items run in parallel with Phase 1 and have no phase dependency.

*Questi item sono indipendenti e procedono in parallelo alla Fase 1.*

### Firmware / Firmware

- [ ] **[BC-48]** Fix CI pipeline — missing Docker Hub credentials for `sebdropsa/stm32-build-env` / *Riparare pipeline CI*
- [ ] **[BC-34]** ⚠️ **P0 — BLOCKER for Phase 3** BLE disconnect after MTU 517 negotiation — random BLE session drops / *Disconnessione BLE dopo negoziazione MTU 517 — **P0, blocca Fase 3***
- [ ] **[BC-16]** ⚠️ **P0 — BLOCKER for Phase 5** BLE disconnect during OTA update — if a device loses WiFi permanently, BLE OTA is the only upgrade path; this bug makes it unreliable / *Disconnessione BLE durante OTA — **P0, prereq Fase 5***
- [ ] **[C2]** Re-enable OTA CRC check (`#if 0` → `#if 1`) in both firmware and bootloader / *Riabilitare CRC check OTA*

### API & Platform / API e Piattaforma

- [ ] **[BC-14]** Web-to-mobile settings sync not working — parameter changed on web not reflected in Flutter app / *Sincronizzazione impostazioni web→mobile non funzionante*
- [ ] **[PUSH]** Verify end-to-end push notification delivery via SNS → APNs/FCM:
  `AwsSnsService.php` already implements `createPlatformEndpoint()` for both platforms (Android/FCM and iOS/APNs), `createTopic()` / `subscribe()` / `publish()`.
  Gap: confirm alarm events are correctly wired to `SNS publish()` in the event pipeline.
  Estimated: 0.5–1 day audit + fix if needed. / *Verificare la delivery push end-to-end via SNS → APNs/FCM: AwsSnsService.php già implementa createPlatformEndpoint(). Gap: verificare il collegamento eventi allarme → SNS publish().*

---

## Phase 2: Infrastructure & Cloud Scalability / Infrastruttura e Scalabilita' Cloud

Fix infrastructure bottlenecks without rewriting application code.

*Effort: 2–3 days / Impegno: 2–3 giorni*

*Correggere i colli di bottiglia infrastrutturali senza riscrivere il codice applicativo.*

- [ ] **Add composite index `(dev_id, evt_time DESC)` on `events` table** — eliminates full table scan on ~500M rows, critical for all dashboard queries (`SELECT WHERE dev_id=? AND evt_time > ?`). Fix: 5 minutes. / *Indice composito su events — elimina full table scan critico*
- [ ] **Redis caching for device variable values** (30min TTL) — eliminates repeated DB reads per dashboard refresh; keys: `device:{id}:vars` / *Cache Redis per valori variabili dispositivo*
- [ ] **Review SSE batch size and polling interval** (currently `BATCH_SIZE=100`, `sleep(10)` in `SseStreamController.php`) — reduce unnecessary polling load / *Ottimizzare batch SSE e intervallo polling*
- [ ] Add RDS Proxy for `import-web-app` — connection pooling (currently new DB connection per HTTP request) / *Aggiungere RDS Proxy — connection pooling*
- [ ] Move PHP sessions to Redis *(low priority — API routes are JWT-stateless; only affects web routes if present)* / *Spostare le sessioni PHP su Redis (bassa priorità — le route API sono JWT-stateless)*
- [ ] Enable ECS auto-scaling on all services (`enable_autoscaling = false` → `true`) / *Abilitare ECS auto-scaling*
- [ ] Increase ECS task sizes (1 vCPU / 2 GB) for headroom / *Aumentare dimensioni task ECS*
- [ ] Enable RDS storage encryption (`storage_encrypted = true`) / *Abilitare crittografia storage RDS*
- [ ] Reduce IAM wildcard permissions (`s3:*`, `sns:*` → minimum required) / *Ridurre i permessi IAM wildcard*
- [ ] Upgrade ElastiCache to t4g.small + HA (multi-AZ, replication) / *Aggiornare ElastiCache a HA*
- [ ] Add RDS read replica for query isolation / *Aggiungere read replica RDS*

---

## Phase 3: Protocol Bridge / Bridge Protocollo

Replace the stalled Java gateway (`drucs-gateway-new`) with a NestJS legacy bridge. Existing devices need zero changes.

*Effort: 4–5 days / Impegno: 4–5 giorni*

> **⚠️ Prerequisites before starting Phase 3:**
> - **BC-34 resolved** — BLE disconnects after MTU 517 make the offline field-technician workflow (Mode 2) unreliable. BLE is the primary control path when WiFi is unavailable; this is a P0 blocker.

*Sostituire il gateway Java fermo (`drucs-gateway-new`) con un bridge NestJS legacy. I dispositivi esistenti non richiedono modifiche.*

```
EXISTING DEVICES              NEW DEVICES
(HTTP/AES POST)               (MQTT/TLS)
     |                            |
     v                            v
+-------------------+      +--------------------+
|  LEGACY BRIDGE    |      |  AWS IoT Core      |
|  (NestJS)         |      |  - X.509 certs     |
|  - AES decrypt    |      |  - Device Shadow   |
|  - ACK <5ms       |      |  - Rules Engine    |
+---------+---------+      +---------+----------+
          |                          |
          +-------------+------------+
                        v
             +--------------------+
             |  SQS               |
             |  device.telemetry  |
             +---------+----------+
                       |
         +-------------+-------------+
         v             v             v
   Batch Processor  State Updater  WS Emitter
   (bulk INSERT    (Redis HSET    (Socket.IO
    TimescaleDB)    device:{id})   push)
```

The key insight: existing devices don't change. The bridge speaks their protocol but ACKs immediately (async SQS publish) — the device never waits for DB writes.

*Il punto chiave: i dispositivi esistenti non devono essere modificati. Il bridge parla il loro protocollo e risponde immediatamente (async SQS publish) — il dispositivo non attende la scrittura su DB.*

- [ ] Build NestJS legacy bridge — accepts the same HTTP/AES POST devices currently send / *Costruire il bridge NestJS legacy*
- [ ] AES-128-CFB decrypt/encrypt matching current device protocol / *Decifratura/cifratura AES-128-CFB compatibile*
- [ ] **Async SQS publish** from NestJS bridge — device receives ACK before DB write (eliminates 150+ sync queries per heartbeat) / *Pubblicazione SQS asincrona — ACK immediato al dispositivo*
- [ ] **Batch processor consumer**: accumulate 1s window, bulk INSERT TimescaleDB / *Consumer batch: finestra 1s, bulk INSERT TimescaleDB*
- [ ] **Redis HSET** for device current state — replaces SELECT+UPDATE per variable (sub-ms reads) / *Redis HSET per stato corrente dispositivo*
- [ ] Device registry service (lookup AES keys, device configs) / *Servizio registro dispositivi*
- [ ] Command queue per device (replaces synchronous DB `commands` table) / *Coda comandi per dispositivo*
- [ ] Load test: simulate 1,000 devices at 5s heartbeat / *Test di carico: simulare 1.000 dispositivi a heartbeat 5s*
- [ ] Decommission `drucs-gateway-new` Java service after validation / *Dismettere il servizio Java `drucs-gateway-new`*

---

## Phase 4: Core API Modernization / Modernizzazione API Core

Modernize the API layer using the **Strangler Fig pattern** — NestJS is added alongside Laravel, not as a big-bang replacement. Laravel 12 (`import-web-app`) continues serving all existing v1 endpoints. NestJS takes ownership of new capabilities only: the WebSocket gateway (real-time), v2 endpoints (analytics, device shadow sync, fleet management), and the async SQS processor for device events.

This approach reduces risk significantly: instead of rewriting 127 REST endpoints + 12 cron + 8 SQS jobs in a single effort (25–35 days), the core NestJS layer is operational in **10–15 days**. Laravel endpoints are deprecated incrementally, endpoint by endpoint, as v2 equivalents are validated in production.

*Effort: 10–15 days (core NestJS layer) + incremental Laravel migration / Impegno: 10–15 giorni (layer NestJS core) + migrazione incrementale Laravel*

> **Frontend note:** The web frontend has already converged on **React 19+** (`drucs-v2`, in production at `app.dropsa.app/v2/`). References to Angular 18+ as a target are superseded — all new frontend work targets React 19+. The WebSocket gateway (Socket.IO) introduced in this phase is the critical enabler for the real-time fleet view in Mode 3 (mobile online), which currently relies on 60-second polling.

*Modernizzazione API tramite pattern Strangler Fig: NestJS affianca Laravel (non lo sostituisce in un big-bang). Laravel mantiene gli endpoint v1; NestJS aggiunge WebSocket, endpoint v2 e processing SQS asincrono. Effort ridotto da 25–35 giorni a 10–15 giorni per il layer core.*

**Key design decisions / Decisioni progettuali chiave:**

1. **Device-type agnostic** — the API knows "devices" with "parameters" and "telemetry streams", not "lubrication pumps". Product logic lives in configuration, not code.
2. **Multi-tenant from day one** — companies, users, device groups scoped with proper authorization.
3. **Backwards compatible** — Laravel v1 continues serving existing React (drucs-v2) + Flutter API contracts without interruption.
4. **WebSocket for real-time** — clients subscribe to device telemetry streams and receive live updates.

**Database migration / Migrazione database:**

| Current (MariaDB) | Target | Reason |
|---|---|---|
| Device config, users, companies | **PostgreSQL** (RDS) | Better JSON support, extensions, scaling |
| Telemetry time-series | **TimescaleDB** (PostgreSQL extension) | 10–100x faster for time-range queries, auto-compression |
| Device state cache | **Redis** (ElastiCache HA) | Real-time last-known-state |
| Firmware binaries | **S3** (unchanged) | Already working |

**NestJS core layer (new capabilities):**
- [ ] **WebSocket gateway (Socket.IO) — event-driven push** on state change, no polling — live device telemetry for web and mobile — **critical for Mode 3 real-time** / *Gateway WebSocket event-driven — critico per Mode 3 real-time*
- [ ] Auth module (JWT + bcrypt + refresh tokens) / *Modulo autenticazione*
- [ ] Device module v2 (device shadow sync, fleet management endpoints) / *Modulo dispositivi v2*
- [ ] Event module v2 (analytics, filtering, pagination) + **DataLoader pattern** to eliminate N+1 queries (100 events × 4–8 relations = 400–800 queries per API call → 1 batched query per relation) / *Modulo eventi v2 + DataLoader per eliminare N+1*
- [ ] Firmware module (upload, versioning, OTA trigger) / *Modulo firmware*
- [ ] User/company module with multi-tenant authorization / *Modulo utenti/aziende multi-tenant*
- [ ] Async SQS processor for device events (replaces synchronous PHP consumers) / *Processore SQS asincrono per eventi device*
- [ ] Alert engine (threshold rules, notifications) / *Motore allarmi*
- [ ] API documentation v2 (Swagger/OpenAPI) / *Documentazione API v2*
- [ ] **TimescaleDB hypertable** on `(device_id, time)` + **continuous aggregates** for dashboard query performance (10–100x faster for time-range queries vs relational MariaDB) / *TimescaleDB hypertable + aggregate continue per performance dashboard*
- [ ] **Redis as source of truth for device current state** — all frequent reads from Redis HSET (sub-ms), not DB / *Redis come source of truth per stato corrente dispositivo*
- [ ] Prisma schema + migrations / *Schema Prisma + migrazioni*

**Laravel incremental migration (existing v1 endpoints):**
- [ ] Laravel (`import-web-app`) continues serving v1 — React (drucs-v2) + Flutter API contracts unchanged / *Laravel continua a servire v1 — nessun impatto su React + Flutter*
- [ ] Migrate 12 cron tasks to NestJS scheduled tasks or Lambda (incremental) / *Migrare 12 cron task (incrementale)*
- [ ] Migrate 8 SQS jobs (general, emails, event-report, summary-report, device-events) (incremental) / *Migrare 8 job SQS (incrementale)*
- [ ] Incremental deprecation — sunset each Laravel endpoint as the v2 NestJS equivalent is validated / *Deprecazione incrementale endpoint Laravel*

---

## Phase 5: MQTT + Firmware Modernization / MQTT e Modernizzazione Firmware

Add MQTT support to new device firmware while maintaining HTTP/AES as fallback for all deployed devices.

*Effort: 12–15 days firmware + 3–4 days cloud / Impegno: 12–15 giorni firmware + 3–4 giorni cloud*

*Aggiungere il supporto MQTT al firmware mantenendo HTTP/AES come fallback per tutti i dispositivi installati.*

> **Decision / Decisione:** AWS IoT Core selected as MQTT broker for new devices.
> Rationale: managed service, zero operational overhead at current device scale, native
> integration with existing AWS infrastructure (IAM, SQS, Lambda, Rules Engine).

**Unified flow via SQS — both device types share the same processing path:**

```
New MQTT device  → AWS IoT Core → Rules Engine → SQS device.telemetry → Batch Processor
Legacy device    → NestJS Bridge              → SQS device.telemetry → Batch Processor

Single unified processing path via SQS — no separate consumer per protocol.
```

### AWS IoT Core Setup (if AWS IoT Core is chosen)

| Component | Purpose |
|---|---|
| Thing Registry | Device identity (1 Thing per physical device) |
| X.509 Certificates | Per-device mutual TLS authentication |
| IoT Policies | Fine-grained pub/sub permissions per device |
| Device Shadow | Last-known-state (replaces polling) |
| Rules Engine | Route messages to SQS, Lambda, TimescaleDB |

**MQTT topic structure / Struttura topic MQTT:**

```
dropsa/{product_type}/{device_id}/telemetry    <- Device publishes
dropsa/{product_type}/{device_id}/commands      <- Device subscribes
dropsa/{product_type}/{device_id}/config        <- Device subscribes
dropsa/{product_type}/{device_id}/firmware      <- Device subscribes
dropsa/{product_type}/{device_id}/status        <- Device publishes (LWT)
```

### Firmware Architecture

The firmware gains a **transport abstraction layer** — application code is unaware whether it's using HTTP or MQTT:

```
CURRENT FIRMWARE              MODERNIZED FIRMWARE
-----------------             ---------------------
ESP32 AT Commands             ESP32 with MQTT client
  +-- WiFi connect              +-- WiFi connect
  +-- TCP open                  +-- TLS + MQTT connect
  +-- HTTP POST                 +-- MQTT publish/subscribe
  +-- HTTP response parse       +-- MQTT message callback
                                +-- Fallback: legacy HTTP
```

**Per-device migration path / Percorso di migrazione per dispositivo:**

1. Device on old firmware → HTTP → Legacy Bridge (always works)
2. OTA push new firmware with MQTT support (`protocol_mode=AUTO`)
3. Device connects via MQTT → IoT Core (preferred)
4. If MQTT fails, auto-fallback to HTTP → Legacy Bridge
5. Once stable, server sets `protocol_mode=MQTT_ONLY` via command
6. Legacy Bridge eventually decommissioned

**Firmware options / Opzioni firmware ESP32:**
- **Phase 5a (quick win):** Use ESP-AT built-in MQTT commands (`AT+MQTTCONN`, `AT+MQTTPUB`) — minimal STM32 code change
- **Phase 5b (full modernization):** Custom ESP32 firmware with ESP-IDF MQTT client — full MQTT 5.0, X.509 certs, QoS 1/2

### Tasks

> **⚠️ Prerequisite:** BC-16 (BLE OTA disconnect) must be resolved before Phase 5. If a deployed device loses WiFi permanently, BLE OTA is the only upgrade path to get it on the new firmware. BC-16 makes this path unreliable.

- [ ] AWS IoT Core Terraform modules (things, certificates, policies) / *Moduli Terraform IoT Core*
- [ ] MQTT topic structure + IoT Core rules / *Struttura topic MQTT + regole*
- [ ] Certificate provisioning flow / *Flusso provisioning certificati*
- [ ] Transport abstraction layer in firmware (interface + factory) / *Livello astrazione trasporto firmware*
- [ ] ESP-AT MQTT commands integration (Phase 5a) / *Integrazione comandi MQTT ESP-AT (Fase 5a)*
- [ ] MQTT topic pub/sub logic + JSON telemetry serializer / *Logica pub/sub + serializzatore JSON*
- [ ] Protocol mode config in flash (`AUTO` / `MQTT_ONLY` / `HTTP_ONLY`) / *Configurazione modalita' protocollo in flash*
- [ ] Auto-fallback logic (MQTT → HTTP) / *Logica auto-fallback*
- [ ] Device Shadow sync (reported/desired state) / *Sincronizzazione device shadow*
- [ ] X.509 certificate storage in W25Q32 flash / *Archiviazione certificati X.509 in flash*
- [ ] OTA firmware update via MQTT with QoS 1 resume / *Aggiornamento firmware OTA via MQTT con resume QoS 1*
- [ ] Refactor `webserver_fsm` (19 states) to protocol-agnostic FSM / *Refactoring FSM webserver*
- [ ] Custom ESP32 firmware with ESP-IDF MQTT client (Phase 5b) / *Firmware ESP32 personalizzato con ESP-IDF*
- [ ] Unit tests (Unity) + integration tests (Renode simulator) / *Test unitari + integrazione*

---

## Phase 6: Mobile Sync + Generic Dashboard / Sincronizzazione Mobile e Dashboard Generica

*Effort: 10–12 days / Impegno: 10–12 giorni*

The dashboard renders device types from JSON configuration — no per-product frontend code changes for new products.

*Effort: ~10-12 days / La dashboard renderizza i tipi di dispositivo da configurazione JSON — nessun codice frontend per prodotto.*

**Device type definition example / Esempio definizione tipo dispositivo:**
```json
{
  "deviceType": "modular-electronics-v2",
  "displayName": "BRAVO Lubrication Pump",
  "telemetry": [
    { "key": "motor_current", "label": "Motor Current", "unit": "mA", "range": [0, 5000] },
    { "key": "pressure", "label": "Pressure", "unit": "bar", "range": [0, 400] },
    { "key": "reservoir_level", "label": "Level", "type": "boolean", "labels": ["OK", "Low"] }
  ],
  "parameters": [
    { "key": "interval", "label": "Lubrication Interval", "unit": "s", "type": "number", "range": [60, 86400] }
  ],
  "commands": [
    { "key": "start_cycle", "label": "Start Lubrication", "confirm": true }
  ],
  "alerts": [
    { "key": "overpressure", "severity": "critical", "condition": "pressure > 350" }
  ]
}
```

- [ ] Update Flutter app to use Phase 4 NestJS API (v1 compat layer → native v2) / *Aggiornare l'app Flutter*
- [ ] WebSocket live telemetry in Flutter (replaces 5-minute cache) / *Telemetria live WebSocket in Flutter*
- [ ] Push notifications for device alarms and command completion / *Notifiche push per allarmi e comandi*
- [ ] Web-to-mobile settings sync fix (BC-14, persistent fix via shared NestJS API) / *Fix sincronizzazione web→mobile*
- [ ] Device-type definition schema + loader / *Schema definizione tipo dispositivo + loader*
- [ ] Dashboard layout engine (grid + widgets) / *Motore layout dashboard (griglia + widget)*
- [ ] Widget library (gauge, chart, table, map, boolean, enum) / *Libreria widget*
- [ ] Device detail page (parameters, commands, logs) / *Pagina dettaglio dispositivo*
- [ ] Alert configuration UI / *Interfaccia configurazione allarmi*
- [ ] Command progress feedback — user sees "queued / delivered / executed" states / *Feedback avanzamento comando*
- [ ] Embedded help pages via WebView / *Pagine aiuto incorporate tramite WebView*

---

## Phase 7: Advanced Analytics + Predictive Maintenance / Analisi Avanzata e Manutenzione Predittiva

*Effort: 15–20 days / Impegno: 15–20 giorni*

### Available telemetry features (already collected by firmware)

The DRUCS firmware telemetry struct already captures the five key features for predictive maintenance — no new sensor work required:

| Field | Unit | Predictive signal |
|---|---|---|
| `current` | A | Motor bearing wear → abnormal current draw |
| `pressure` | Bar | Pipe blockage → pressure drop |
| `temperature` | °C | Motor overheating → temperature drift |
| `rotations` | count | Lubricant consumption rate deviation |
| `voltage` | V | Power supply anomalies |

These are stored as time-series in TimescaleDB (Phase 4), which serves as the feature store for all ML pipelines.

### Step 1 — Statistical anomaly detection (no ML, no training data)

Z-score rolling window on `current`, `pressure`, and `temperature`. Flags values deviating > 3σ from the 7-day moving average per device. Works from day one without historical training data. Implementable as a Python Lambda or lightweight FastAPI microservice on ECS.

### Step 2 — ML pipeline

| Component | Technology |
|---|---|
| Runtime | Python 3.12, FastAPI microservice on ECS |
| Anomaly detection | Isolation Forest (unsupervised — no labels required) |
| Consumption forecasting | Prophet (handles seasonality and trends) |
| Model registry | MLflow (version tracking, metrics, artifact storage) |
| Feature store | TimescaleDB continuous aggregates |
| Inference trigger | EventBridge schedule + SQS event queue |

### Step 3 — OPC-UA / PLC integration

DRUCS already supports Modbus, CAN, IO-Link, and J1939 at the firmware level. For integration with Siemens PLCs, ABB drives, or SCADA systems (Ignition, WinCC), **OPC-UA** is the industrial de facto standard. If EMQX is adopted in Phase 5, OPC-UA protocol bridges are available natively. With AWS IoT Core, this requires IoT SiteWise (expensive add-on). Evaluate based on customer SCADA requirements.

### Data Retention Strategy

With ~16K records/device in ring buffer and fleet-scale device counts, TimescaleDB volume can grow rapidly without a retention policy:

| Tier | Retention | Storage |
|---|---|---|
| Raw telemetry | 90 days | TimescaleDB (hot) |
| Hourly aggregates | 2 years | TimescaleDB continuous aggregates |
| Daily aggregates | Indefinite | TimescaleDB compressed |
| Cold archive | Indefinite | S3 (Parquet via TimescaleDB tiered storage) |

### Tasks

- [ ] Data retention policy — raw 90d, hourly aggregates 2y, S3 cold tier / *Policy retention dati*
- [ ] TimescaleDB continuous aggregates for dashboard query performance / *Aggregate continue TimescaleDB*
- [ ] Statistical anomaly detection service (Z-score rolling on current/pressure/temperature, Python Lambda) / *Servizio anomaly detection statistica*
- [ ] Python FastAPI ML microservice on ECS (Isolation Forest + Prophet) / *Microservizio ML Python FastAPI su ECS*
- [ ] MLflow model registry + training pipeline / *Registry modelli MLflow + pipeline training*
- [ ] Fleet-level analytics and reporting / *Analisi e reportistica a livello flotta*
- [ ] Lubricant consumption optimization (Prophet-based forecasting per device) / *Ottimizzazione consumo lubrificante*
- [ ] OPC-UA bridge evaluation — EMQX native vs IoT SiteWise — for PLC/SCADA integration / *Valutazione bridge OPC-UA per integrazione PLC/SCADA*
- [ ] AI-assisted troubleshooting (LLM-based diagnostic from event history + anomaly context) / *Diagnostica assistita da AI*

---

## Timeline & Effort Summary / Timeline e Riepilogo Impegno

```
Week  1-2   Phase 0: Security Fixes          (1-2 days actual work)
Week  3-6   Phase 2: Cloud Scalability        (2-3 days actual work)
Week  4-8   Phase 3: Protocol Bridge          (4-5 days actual work)
Week  6-10  Phase 4: NestJS core layer        (10-15 days actual work, then incremental)
Week  8-14  Phase 5: MQTT + Firmware          (15-19 days actual work)
Week 14-20  Phase 6: Generic Dashboard        (10-12 days actual work)
```

| Phase | Cloud Work | Firmware Work | Total Days |
|---|---|---|---|
| Phase 0: Security | 1 day | 0.5 days | 1.5 days |
| Phase 2: Scalability | 2 days | — | 2 days |
| Phase 3: Protocol Bridge | **5 days** | — | 5 days |
| Phase 4: NestJS core layer | 10–15 days | — | 10–15 days |
| Phase 5: MQTT Cloud | 4 days | — | 4 days |
| Phase 5: Firmware MQTT | — | 15 days | 15 days |
| Phase 6: Dashboard | 12 days | — | 12 days |
| **TOTAL** | **~32–38 days** | **15.5 days** | **~48–54 days** |

Calendar time: **~5–6 months** (accounting for testing, hardware validation, staged OTA rollout, and non-consecutive work days).

*Tempo calendario: ~5–6 mesi (considerando test, validazione hardware, rollout OTA graduale e giorni di lavoro non consecutivi).*

---

## Before vs After / Prima e Dopo

| Aspect | Current | After Modernization |
|---|---|---|
| Device protocol | HTTP/1.0 POST, custom AES, 5s polling | MQTT/TLS, X.509 certs, persistent connection |
| Server→device latency | Up to 5 seconds | < 100 milliseconds |
| Offline handling | Data lost | MQTT QoS 1 (guaranteed delivery) |
| Security | JWT auth, zero-IV AES on device | bcrypt, TLS mutual auth, X.509 certs |
| Scalability | ~500 devices max | 100,000+ devices (IoT Core) |
| Real-time dashboard | 5s poll via SSE | Live WebSocket streaming |
| API | 640 PHP files, Laravel 12 | NestJS (WebSocket + v2) + Laravel (v1, incremental deprecation) |
| Database | Single MariaDB, no pooling | PostgreSQL + TimescaleDB + Redis HA |
| New device types | Requires code changes | JSON configuration only |
| Development velocity | Dependent on Capaciteam | Internal with Claude Code |
| Auto-scaling | Disabled | ECS auto-scaling (2–10 replicas) |
| Firmware updates | HTTP download, no resume | MQTT-based, resumable, verified |

---

## Appendix A: Repository Migration Plan / Piano di Migrazione Repository

| Current Repo | Fate | Replacement |
|---|---|---|
| `drucs-gateway-new` (Java) | Sunset after Phase 3 | Legacy Bridge (NestJS) + IoT Core |
| `frontend-web` (Angular 17) | **Retired** — replaced by `drucs-v2` | `drucs-v2` (React 19+) — generic dashboard in Phase 6 |
| `mobile` (Flutter) | Keep, update API calls | Phase 4 v1 compat layer → native v2 |
| `drucs-iac` (Terraform) | Extend | Add IoT Core, TimescaleDB, new ECS services |
| `import-web-app` (Laravel) | Sunset after Phase 4 | NestJS API (127 endpoints, 12 cron, 8 SQS jobs) |
| `S-00365-firmware` (STM32) | Evolve in Phase 5 | Add MQTT transport, keep legacy HTTP fallback |

---

## Appendix B: Technology Stack Comparison / Confronto Stack Tecnologici

| Layer | Current | Target |
|---|---|---|
| Device → Cloud | HTTP/1.0 + AES-128 | MQTT 3.1.1/5.0 + TLS 1.3 |
| Cloud broker | Java Spring Boot (custom) | **AWS IoT Core** (managed MQTT) |
| Internal message bus | SQS (5 existing queues) | SQS (unchanged + new `device.telemetry` queue) |
| Push notifications | SNS → Firebase/APNs | SNS (unchanged — verify end-to-end wiring) |
| API backend | PHP 8.4 (Laravel 12) | NestJS (TypeScript) |
| API auth | JWT (tymon/jwt-auth v2) + Redis blacklist | JWT + bcrypt + Redis sessions |
| Database (config) | MariaDB 10.11 | PostgreSQL 16 |
| Database (telemetry) | MariaDB (same instance) | TimescaleDB hypertable on `(device_id, time)` |
| Device state cache | DB read per request | **Redis HSET** `device:{id}` (sub-ms) |
| Cache | Redis micro (no HA) | Redis HA (multi-AZ, encrypted) |
| Real-time (clients) | SSE polling every 10s | **WebSocket** (Socket.IO, event-driven) |
| Frontend | Angular 17 (retired — replaced by drucs-v2) | **React 19** + Vite + TanStack Query + shadcn/ui (already in production at app.dropsa.app/v2/) |
| Mobile | Flutter 3.32 | Flutter (updated API client) |
| Device auth | AES shared key | X.509 mutual TLS |
| Device firmware | ESP-AT + custom HTTP | ESP-AT MQTT or ESP-IDF custom |
| OTA updates | HTTP download | MQTT-based with resume |
| IaC | Terraform + Terragrunt | Same (extended) |
| Monitoring | CloudWatch (basic) | CloudWatch + X-Ray tracing |

---

## Appendix C: Risk Register / Registro dei Rischi

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Legacy bridge has protocol bugs | Medium | High | Capture real device traffic, replay in integration tests |
| ESP-AT MQTT support is limited | Medium | Medium | Fall back to custom ESP-IDF firmware (Phase 5b) |
| OTA brick risk during firmware migration | Low | Critical | Bootloader always preserves last-known-good image |
| TimescaleDB migration data loss | Low | High | Parallel-write during migration, verify before cutover |
| React (drucs-v2) / Flutter breaks with new API | Medium | High | Laravel v1 continues serving existing API contracts; NestJS v2 endpoints are additive |
| AWS IoT Core regional limits | Low | Medium | eu-west-2 supports 500K connections per account |

---

## How to Request Features / Come Richiedere Funzionalita'

Field engineers and stakeholders can request features through the [Feature Request](../field/feature-request.md) page. No GitHub account needed.

*I tecnici sul campo e gli stakeholder possono richiedere funzionalita' tramite la pagina [Richiesta Funzionalita'](../field/feature-request.md). Non serve un account GitHub.*
