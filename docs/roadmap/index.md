# Development Roadmap / Roadmap di Sviluppo

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

## Phase 0: Security & Cleanup (COMPLETE) / Sicurezza e Pulizia (COMPLETATA)

- [x] GitHub org audit — downgraded vendor access / *Audit org GitHub — declassato accesso fornitore*
- [x] AWS IAM audit — deactivated unused keys / *Audit AWS IAM — disattivate chiavi non usate*
- [x] Org default permissions set to read-only / *Permessi org predefiniti impostati a sola lettura*
- [x] GitHub Teams created (Project Admins, Electronics, Management, Field Engineers)
- [x] Firmware repo migrated to DropsA-SpA org / *Repo firmware migrato nell'org DropsA-SpA*
- [ ] Branch cleanup — 163 stale branches to delete / *Pulizia branch — 163 branch obsoleti da eliminare*

---

## Phase 1: Web v2 Dashboard (IN PROGRESS) / Dashboard Web v2 (IN CORSO)

The new React-based dashboard replacing the Angular v1 frontend. Consumes the existing `import-web-app` API (v1 endpoints).

*La nuova dashboard basata su React che sostituisce il frontend Angular v1. Usa l'API `import-web-app` esistente (endpoint v1).*

> **Note / Nota:** The current dashboard uses polling (variables refreshed every 60s via `import-web-app`). Live WebSocket streaming will be available in Phase 4. / *La dashboard attuale usa il polling (variabili aggiornate ogni 60s). Lo streaming WebSocket live sara' disponibile in Fase 4.*

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

- [ ] Device detail page — full variable display, settings editor, event history / *Pagina dettaglio dispositivo — visualizzazione variabili, editor impostazioni, storico eventi*
- [ ] Per-user "Inactive" device flag / *Flag dispositivo "Inattivo" per utente*
- [ ] Notifications panel (in-app) / *Pannello notifiche (in-app)*
- [ ] Help page integration (embed from this docs site) / *Integrazione pagine aiuto (incorporate da questo sito docs)*

---

## Active Fixes (Parallel Track) / Fix Attivi (Track Parallelo)

These items run in parallel with Phase 1 and have no phase dependency.

*Questi item sono indipendenti e procedono in parallelo alla Fase 1.*

### Firmware / Firmware

- [ ] **[BC-48]** Fix CI pipeline — missing Docker Hub credentials for `sebdropsa/stm32-build-env` / *Riparare pipeline CI — credenziali Docker Hub mancanti*
- [ ] **[BC-34]** BLE disconnect after MTU 517 negotiation — random BLE session drops / *Disconnessione BLE dopo negoziazione MTU 517 — drop casuali sessioni BLE*
- [ ] **[C2]** Re-enable OTA CRC check (`#if 0` → `#if 1`) in both firmware and bootloader / *Riabilitare CRC check OTA (`#if 0` → `#if 1`) in firmware e bootloader*

### API & Platform / API e Piattaforma

- [ ] **[BC-14]** Web-to-mobile settings sync not working — parameter changed on web not reflected in Flutter app / *Sincronizzazione impostazioni web→mobile non funzionante*
- [ ] **[SEC]** Store JWT secret in AWS SSM Parameter Store (not hardcoded in config) / *Conservare il segreto JWT in AWS SSM Parameter Store*
- [ ] **[SEC]** Rotate AWS IAM key found hardcoded in `drucs-web-services/login.php` — audit CloudTrail for unauthorized usage / *Ruotare la chiave AWS hardcoded in `drucs-web-services/login.php` — audit CloudTrail*

---

## Phase 2: Infrastructure & Cloud Scalability / Infrastruttura e Scalabilita' Cloud

Fix infrastructure bottlenecks without rewriting application code.

*Correggere i colli di bottiglia infrastrutturali senza riscrivere il codice applicativo.*

- [ ] Add RDS Proxy for `import-web-app` — connection pooling (currently new DB connection per HTTP request) / *Aggiungere RDS Proxy — connection pooling (attualmente nuova connessione DB per ogni richiesta)*
- [ ] Move PHP sessions to Redis — enables horizontal ECS scaling / *Spostare le sessioni PHP su Redis — abilita lo scaling orizzontale ECS*
- [ ] Enable ECS auto-scaling on all services (`enable_autoscaling = false` → `true`) / *Abilitare ECS auto-scaling su tutti i servizi*
- [ ] Enable RDS storage encryption (`storage_encrypted = true`) / *Abilitare crittografia storage RDS*
- [ ] Reduce IAM wildcard permissions (`s3:*`, `sns:*` → minimum required) / *Ridurre i permessi IAM wildcard al minimo necessario*
- [ ] Upgrade ElastiCache to HA (multi-AZ, replication) / *Aggiornare ElastiCache a HA (multi-AZ, replica)*
- [ ] Add RDS read replica for query isolation / *Aggiungere read replica RDS per l'isolamento delle query*

---

## Phase 3: Protocol Bridge / Bridge Protocollo

Replace the stalled Java gateway (`drucs-gateway-new`) with a NestJS legacy bridge. Existing devices need zero changes.

*Sostituire il gateway Java fermo (`drucs-gateway-new`) con un bridge NestJS legacy. I dispositivi esistenti non richiedono modifiche.*

- [ ] Build NestJS legacy bridge — accepts the same HTTP/AES POST devices currently send / *Costruire il bridge NestJS legacy — accetta lo stesso HTTP/AES POST dei dispositivi attuali*
- [ ] AES-128-CFB decrypt/encrypt matching current device protocol / *Decifratura/cifratura AES-128-CFB compatibile con il protocollo dispositivo attuale*
- [ ] Command queue per device (replaces `commands` DB table as synchronous queue) / *Coda comandi per dispositivo (sostituisce la tabella `commands` come coda sincrona)*
- [ ] Publish unified internal message format to SQS/EventBridge / *Pubblicare il formato messaggi interno unificato su SQS/EventBridge*
- [ ] Load test: simulate 1,000 devices at 5s heartbeat / *Test di carico: simulare 1.000 dispositivi a heartbeat 5s*
- [ ] Decommission `drucs-gateway-new` Java service after validation / *Dismettere il servizio Java `drucs-gateway-new` dopo la validazione*

---

## Phase 4: Core API Modernization / Modernizzazione API Core

Replace `import-web-app` (Laravel 12, 127 REST endpoints, 12 cron tasks, 8 SQS jobs) with a modular NestJS API.
Existing Angular v2 and Flutter clients must continue working via a v1 compatibility layer throughout the migration.

*Sostituire `import-web-app` (Laravel 12, 127 endpoint REST, 12 cron task, 8 job SQS) con un'API NestJS modulare.
I client Angular v2 e Flutter esistenti devono continuare a funzionare tramite un livello di compatibilita' v1 durante tutta la migrazione.*

- [ ] Auth module (JWT + bcrypt + refresh tokens) / *Modulo autenticazione (JWT + bcrypt + refresh token)*
- [ ] Device module (CRUD, registry, variable/parameter read-write) / *Modulo dispositivi (CRUD, registro, lettura/scrittura variabili e parametri)*
- [ ] Event module (history, filtering, pagination, SSE → WebSocket) / *Modulo eventi (storico, filtri, paginazione, SSE → WebSocket)*
- [ ] Firmware module (upload, versioning, OTA trigger via `import-web-app` API key endpoint) / *Modulo firmware*
- [ ] User/company module with multi-tenant authorization / *Modulo utenti/aziende con autorizzazione multi-tenant*
- [ ] Migrate 12 cron tasks to NestJS scheduled tasks or Lambda / *Migrare 12 cron task a NestJS scheduled tasks o Lambda*
- [ ] Migrate 8 SQS jobs (general, emails, event-report, summary-report, device-events) / *Migrare 8 job SQS*
- [ ] WebSocket gateway (Socket.IO) — live device telemetry for web and mobile / *Gateway WebSocket (Socket.IO) — telemetria live per web e mobile*
- [ ] v1 compatibility layer — serve existing Angular v2 + Flutter API contracts exactly / *Livello compatibilita' v1 — rispettare esattamente i contratti API di Angular v2 e Flutter*
- [ ] API documentation (Swagger/OpenAPI) / *Documentazione API (Swagger/OpenAPI)*
- [ ] TimescaleDB for telemetry time-series (replaces relational telemetry tables) / *TimescaleDB per telemetria time-series*
- [ ] Sunset `import-web-app` after full validation / *Dismettere `import-web-app` dopo validazione completa*

---

## Phase 5: MQTT + Firmware Modernization / MQTT e Modernizzazione Firmware

Add MQTT support to new device firmware while maintaining HTTP/AES as fallback for all deployed devices.

*Aggiungere il supporto MQTT al firmware dei nuovi dispositivi mantenendo HTTP/AES come fallback per tutti i dispositivi installati.*

- [ ] AWS IoT Core provisioning (Thing registry, X.509 certificates, IoT policies) / *Provisioning AWS IoT Core (registro Thing, certificati X.509, policy IoT)*
- [ ] MQTT topic structure: `dropsa/{type}/{id}/telemetry|commands|firmware|status` / *Struttura topic MQTT*
- [ ] Firmware transport abstraction layer (HTTP legacy fallback + MQTT preferred) / *Livello astrazione trasporto firmware (fallback HTTP legacy + MQTT preferito)*
- [ ] ESP-AT MQTT commands integration (Phase 5a quick win) / *Integrazione comandi MQTT ESP-AT (Fase 5a risultato rapido)*
- [ ] OTA firmware update via MQTT with QoS 1 resume / *Aggiornamento firmware OTA via MQTT con resume QoS 1*
- [ ] Per-device migration path: HTTP → AUTO → MQTT_ONLY via OTA / *Percorso migrazione per dispositivo: HTTP → AUTO → MQTT_ONLY via OTA*
- [ ] Custom ESP32 firmware with ESP-IDF MQTT client (Phase 5b full modernization) / *Firmware ESP32 personalizzato con client MQTT ESP-IDF (Fase 5b modernizzazione completa)*

---

## Phase 6: Mobile Sync + Generic Dashboard / Sincronizzazione Mobile e Dashboard Generica

- [ ] Update Flutter app to use Phase 4 NestJS API (v1 compat layer → native v2) / *Aggiornare l'app Flutter per usare l'API NestJS della Fase 4*
- [ ] WebSocket live telemetry in Flutter (replaces 5-minute cache + no-freshness-indicator) / *Telemetria live WebSocket in Flutter (sostituisce cache 5 minuti)*
- [ ] Push notifications for device alarms and command completion / *Notifiche push per allarmi dispositivo e completamento comandi*
- [ ] Web-to-mobile settings sync fix (BC-14, persistent fix via shared NestJS API) / *Fix sincronizzazione impostazioni web→mobile (BC-14, fix definitivo via API NestJS condivisa)*
- [ ] Device-type-agnostic dashboard (JSON device-type definitions, no per-product frontend code) / *Dashboard agnostica rispetto al tipo di dispositivo (definizioni JSON, nessun codice frontend per prodotto)*
- [ ] Command progress feedback — user sees "queued / delivered / executed" states / *Feedback avanzamento comando — l'utente vede gli stati "in coda / consegnato / eseguito"*
- [ ] Embedded help pages via WebView / *Pagine aiuto incorporate tramite WebView*

---

## Phase 7: Advanced Analytics / Analisi Avanzata

- [ ] Predictive maintenance models / *Modelli manutenzione predittiva*
- [ ] Fleet-level analytics and reporting / *Analisi e reportistica a livello flotta*
- [ ] Lubricant consumption optimization / *Ottimizzazione consumo lubrificante*
- [ ] AI-assisted troubleshooting / *Risoluzione problemi assistita da AI*

---

## How to Request Features / Come Richiedere Funzionalita'

Field engineers and stakeholders can request features through the [Feature Request](../field/feature-request.md) page. No GitHub account needed.

*I tecnici sul campo e gli stakeholder possono richiedere funzionalita' tramite la pagina [Richiesta Funzionalita'](../field/feature-request.md). Non serve un account GitHub.*

---

*Full detailed roadmap: DRUCS-Unified-Modernization-Roadmap (internal document, available from project lead).*

*Roadmap dettagliata completa: DRUCS-Unified-Modernization-Roadmap (documento interno, disponibile dal project lead).*
