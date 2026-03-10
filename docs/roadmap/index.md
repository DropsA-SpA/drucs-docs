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

No connection pooling (new DB connection per HTTP request), no middleware pipeline (auth/rate-limiting/validation are ad-hoc). Sessions not in Redis, so ECS horizontal scaling is blocked.

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
- [ ] **[BC-34]** BLE disconnect after MTU 517 negotiation — random BLE session drops / *Disconnessione BLE dopo negoziazione MTU 517*
- [ ] **[C2]** Re-enable OTA CRC check (`#if 0` → `#if 1`) in both firmware and bootloader / *Riabilitare CRC check OTA*

### API & Platform / API e Piattaforma

- [ ] **[BC-14]** Web-to-mobile settings sync not working — parameter changed on web not reflected in Flutter app / *Sincronizzazione impostazioni web→mobile non funzionante*

---

## Phase 2: Infrastructure & Cloud Scalability / Infrastruttura e Scalabilita' Cloud

Fix infrastructure bottlenecks without rewriting application code.

*Effort: 2–3 days / Impegno: 2–3 giorni*

*Correggere i colli di bottiglia infrastrutturali senza riscrivere il codice applicativo.*

- [ ] Add RDS Proxy for `import-web-app` — connection pooling (currently new DB connection per HTTP request) / *Aggiungere RDS Proxy — connection pooling*
- [ ] Move PHP sessions to Redis — enables horizontal ECS scaling / *Spostare le sessioni PHP su Redis*
- [ ] Enable ECS auto-scaling on all services (`enable_autoscaling = false` → `true`) / *Abilitare ECS auto-scaling*
- [ ] Increase ECS task sizes (1 vCPU / 2 GB) for headroom / *Aumentare dimensioni task ECS*
- [ ] Enable RDS storage encryption (`storage_encrypted = true`) / *Abilitare crittografia storage RDS*
- [ ] Reduce IAM wildcard permissions (`s3:*`, `sns:*` → minimum required) / *Ridurre i permessi IAM wildcard*
- [ ] Upgrade ElastiCache to t4g.small + HA (multi-AZ, replication) / *Aggiornare ElastiCache a HA*
- [ ] Add RDS read replica for query isolation / *Aggiungere read replica RDS*

---

## Phase 3: Protocol Bridge / Bridge Protocollo

Replace the stalled Java gateway (`drucs-gateway-new`) with a NestJS legacy bridge. Existing devices need zero changes.

*Effort: 3–4 days / Impegno: 3–4 giorni*

*Sostituire il gateway Java fermo (`drucs-gateway-new`) con un bridge NestJS legacy. I dispositivi esistenti non richiedono modifiche.*

```
EXISTING DEVICES              NEW DEVICES
(HTTP/AES POST)               (MQTT/TLS)
     |                            |
     v                            v
+-------------------+      +--------------------+
|  LEGACY BRIDGE    |      |  AWS IoT Core      |
|  (NestJS)         |      |  (Managed MQTT)    |
|  - Accepts POST   |      |  - X.509 certs     |
|  - Decrypts AES   |      |  - Device shadow   |
|  - Translates     |      |  - Rules engine    |
+---------+---------+      +---------+----------+
          |                          |
          +-------------+------------+
                        v
             +--------------------+
             |  MESSAGE BUS       |
             |  (SQS/EventBridge) |
             |  Unified format:   |
             |  { deviceId,       |
             |    deviceType,     |
             |    timestamp,      |
             |    telemetry: {},  |
             |    commands: [] }  |
             +---------+----------+
```

The key insight: existing devices don't change. The bridge speaks their protocol but translates everything into the new unified format internally.

*Il punto chiave: i dispositivi esistenti non devono essere modificati. Il bridge parla il loro protocollo ma traduce internamente tutto nel formato unificato.*

- [ ] Build NestJS legacy bridge — accepts the same HTTP/AES POST devices currently send / *Costruire il bridge NestJS legacy*
- [ ] AES-128-CFB decrypt/encrypt matching current device protocol / *Decifratura/cifratura AES-128-CFB compatibile*
- [ ] Device registry service (lookup AES keys, device configs) / *Servizio registro dispositivi*
- [ ] Command queue per device (replaces synchronous DB `commands` table) / *Coda comandi per dispositivo*
- [ ] Publish unified internal message format to SQS/EventBridge / *Pubblicare il formato messaggi unificato su SQS/EventBridge*
- [ ] Load test: simulate 1,000 devices at 5s heartbeat / *Test di carico: simulare 1.000 dispositivi a heartbeat 5s*
- [ ] Decommission `drucs-gateway-new` Java service after validation / *Dismettere il servizio Java `drucs-gateway-new`*

---

## Phase 4: Core API Modernization / Modernizzazione API Core

Replace `import-web-app` (Laravel 12, 127 REST endpoints, 12 cron tasks, 8 SQS jobs) with a modular NestJS API.
Existing Angular v2 and Flutter clients must continue working via a v1 compatibility layer throughout the migration.

*Effort: 25–35 days / Impegno: 25–35 giorni*

*Sostituire `import-web-app` con un'API NestJS modulare. I client Angular v2 e Flutter devono continuare a funzionare tramite un livello di compatibilita' v1.*

**Key design decisions / Decisioni progettuali chiave:**

1. **Device-type agnostic** — the API knows "devices" with "parameters" and "telemetry streams", not "lubrication pumps". Product logic lives in configuration, not code.
2. **Multi-tenant from day one** — companies, users, device groups scoped with proper authorization.
3. **Backwards compatible** — v1 compat layer serves existing Angular/Flutter API contracts exactly.
4. **WebSocket for real-time** — clients subscribe to device telemetry streams and receive live updates.

**Database migration / Migrazione database:**

| Current (MariaDB) | Target | Reason |
|---|---|---|
| Device config, users, companies | **PostgreSQL** (RDS) | Better JSON support, extensions, scaling |
| Telemetry time-series | **TimescaleDB** (PostgreSQL extension) | 10–100x faster for time-range queries, auto-compression |
| Device state cache | **Redis** (ElastiCache HA) | Real-time last-known-state |
| Firmware binaries | **S3** (unchanged) | Already working |

- [ ] Auth module (JWT + bcrypt + refresh tokens) / *Modulo autenticazione*
- [ ] Device module (CRUD, registry, variable/parameter read-write) / *Modulo dispositivi*
- [ ] Event module (history, filtering, pagination, SSE → WebSocket) / *Modulo eventi*
- [ ] Firmware module (upload, versioning, OTA trigger) / *Modulo firmware*
- [ ] User/company module with multi-tenant authorization / *Modulo utenti/aziende multi-tenant*
- [ ] Migrate 12 cron tasks to NestJS scheduled tasks or Lambda / *Migrare 12 cron task*
- [ ] Migrate 8 SQS jobs (general, emails, event-report, summary-report, device-events) / *Migrare 8 job SQS*
- [ ] WebSocket gateway (Socket.IO) — live device telemetry for web and mobile / *Gateway WebSocket*
- [ ] Alert engine (threshold rules, notifications) / *Motore allarmi*
- [ ] v1 compatibility layer — serve existing Angular v2 + Flutter API contracts exactly / *Livello compatibilita' v1*
- [ ] API documentation (Swagger/OpenAPI) / *Documentazione API*
- [ ] TimescaleDB for telemetry time-series (replaces relational telemetry tables) / *TimescaleDB*
- [ ] Prisma schema + migrations / *Schema Prisma + migrazioni*
- [ ] Sunset `import-web-app` after full validation / *Dismettere `import-web-app`*

---

## Phase 5: MQTT + Firmware Modernization / MQTT e Modernizzazione Firmware

Add MQTT support to new device firmware while maintaining HTTP/AES as fallback for all deployed devices.

*Effort: 12–15 days firmware + 3–4 days cloud / Impegno: 12–15 giorni firmware + 3–4 giorni cloud*

*Aggiungere il supporto MQTT al firmware mantenendo HTTP/AES come fallback per tutti i dispositivi installati.*

### AWS IoT Core Setup

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

## Phase 7: Advanced Analytics / Analisi Avanzata

- [ ] Predictive maintenance models / *Modelli manutenzione predittiva*
- [ ] Fleet-level analytics and reporting / *Analisi e reportistica a livello flotta*
- [ ] Lubricant consumption optimization / *Ottimizzazione consumo lubrificante*
- [ ] AI-assisted troubleshooting / *Risoluzione problemi assistita da AI*

---

## Timeline & Effort Summary / Timeline e Riepilogo Impegno

```
Week  1-2   Phase 0: Security Fixes          (1-2 days actual work)
Week  3-6   Phase 2: Cloud Scalability        (2-3 days actual work)
Week  4-8   Phase 3: Protocol Bridge          (3-4 days actual work)
Week  6-12  Phase 4: New NestJS API           (25-35 days actual work)
Week 10-16  Phase 5: MQTT + Firmware          (15-19 days actual work)
Week 16-22  Phase 6: Generic Dashboard        (10-12 days actual work)
```

| Phase | Cloud Work | Firmware Work | Total Days |
|---|---|---|---|
| Phase 0: Security | 1 day | 0.5 days | 1.5 days |
| Phase 2: Scalability | 2 days | — | 2 days |
| Phase 3: Protocol Bridge | 4 days | — | 4 days |
| Phase 4: New API | 12 days | — | 12 days |
| Phase 5: MQTT Cloud | 4 days | — | 4 days |
| Phase 5: Firmware MQTT | — | 15 days | 15 days |
| Phase 6: Dashboard | 12 days | — | 12 days |
| **TOTAL** | **35 days** | **15.5 days** | **~50 days** |

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
| API | 640 PHP files, Laravel 12 | Modular NestJS with TypeScript |
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
| `frontend-web` (Angular) | Evolve in Phase 6 | Same repo, upgraded to generic dashboard |
| `mobile` (Flutter) | Keep, update API calls | Phase 4 v1 compat layer → native v2 |
| `drucs-iac` (Terraform) | Extend | Add IoT Core, TimescaleDB, new ECS services |
| `import-web-app` (Laravel) | Sunset after Phase 4 | NestJS API (127 endpoints, 12 cron, 8 SQS jobs) |
| `S-00365-firmware` (STM32) | Evolve in Phase 5 | Add MQTT transport, keep legacy HTTP fallback |

---

## Appendix B: Technology Stack Comparison / Confronto Stack Tecnologici

| Layer | Current | Target |
|---|---|---|
| Device → Cloud | HTTP/1.0 + AES-128 | MQTT 3.1.1/5.0 + TLS 1.3 |
| Cloud broker | Java Spring Boot (custom) | AWS IoT Core (managed) |
| API backend | PHP 8.4 (Laravel 12) | NestJS (TypeScript) |
| API auth | JWT (tymon/jwt-auth v2) + Redis blacklist | JWT + bcrypt + Redis sessions |
| Database (config) | MariaDB 10.11 | PostgreSQL 16 |
| Database (telemetry) | MariaDB (same instance) | TimescaleDB (on PostgreSQL) |
| Cache | Redis micro (no HA) | Redis HA (multi-AZ, encrypted) |
| Real-time (clients) | SSE polyfill | WebSocket (Socket.IO) |
| Frontend | Angular 17 | Angular 18+ (generic dashboard) |
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
| Existing Angular/Flutter breaks with new API | Medium | High | v1 compat layer serves old API format exactly |
| AWS IoT Core regional limits | Low | Medium | eu-west-2 supports 500K connections per account |

---

## How to Request Features / Come Richiedere Funzionalita'

Field engineers and stakeholders can request features through the [Feature Request](../field/feature-request.md) page. No GitHub account needed.

*I tecnici sul campo e gli stakeholder possono richiedere funzionalita' tramite la pagina [Richiesta Funzionalita'](../field/feature-request.md). Non serve un account GitHub.*
