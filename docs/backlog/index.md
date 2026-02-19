# Pending Requests & Fixes / *Richieste e Fix in Sospeso*

!!! info "Last updated: 17 February 2026 / *Ultimo aggiornamento: 17 Febbraio 2026*"
    This page tracks all open bugs, feature requests, and infrastructure tasks across the DRUCS platform. Firmware tasks synced from Jira (BC-1 to BC-47).

    *Questa pagina traccia tutti i bug aperti, le richieste di funzionalita' e le attivita' infrastrutturali sulla piattaforma DRUCS. Task firmware sincronizzati da Jira (BC-1 a BC-47).*

---

## Critical / *Critici*

| ID | Type | Description / *Descrizione* | Area | Owner |
|---|---|---|---|---|
| SEC-001 | :material-shield-alert: Security | **AES-128 zero IV** — Cloud device encryption uses zero initialization vector. Must fix. / *Cifratura cloud dispositivi con vettore inizializzazione zero. Da correggere.* | Gateway / FW | Yan + Walter |
| DROP-175 | :material-bug: Bug | **Incorrect alarm status** displayed on frontend / *Stato allarme errato visualizzato sul frontend* | Frontend | — |
| DROP-177 | :material-bug: Bug | **Null tooltip crash** — app crashes on null tooltip data / *Crash tooltip nullo* | Frontend | — |

---

## Bugs / *Bug*

| ID | Type | Description / *Descrizione* | Area | Status |
|---|---|---|---|---|
| DROP-292 | :material-bug: Bug | Email notifications not working correctly / *Notifiche email non funzionanti correttamente* | Backend | Open |
| DROP-297 | :material-bug: Bug | CI pipeline issues / *Problemi pipeline CI* | DevOps | Open |
| DROP-298 | :material-bug: Bug | CI pipeline issues (related) / *Problemi pipeline CI (correlato)* | DevOps | Open |
| BC-34 | :material-bug: Bug | **ESP32 BLE disconnect** — Bluetooth drops connection intermittently / *ESP32 BLE si disconnette in modo intermittente* | Firmware | Open |
| BC-14 | :material-bug: Bug | **Web-to-phone sync** — data not syncing between web and mobile / *Sincronizzazione web-telefono non funzionante* | Mobile | Open |
| EMAIL-001 | :material-check-circle: Fixed | Email spam loop + OOM — fixed 2026-02-16 / *Loop spam email + OOM — corretto 16/02/2026* | Backend | **FIXED** |

---

## Feature Requests / *Richieste Funzionalita'*

| ID | Description / *Descrizione* | Area | Priority |
|---|---|---|---|
| GW-001 | **Gateway rewrite** — Replace dead Spring Boot gateway with new stack / *Riscrittura gateway — Sostituire gateway Spring Boot morto con nuovo stack* | Gateway | HIGH |
| FE-001 | **React frontend** — Complete v2 dashboard, replace Angular / *Frontend React — Completare dashboard v2, sostituire Angular* | Frontend | HIGH |
| BLE-001 | **BLE/MLDP new roadmap** — Redesign Bluetooth communication protocol / *Nuovo roadmap BLE/MLDP — Ridisegnare protocollo comunicazione Bluetooth* | Firmware | HIGH |
| MQTT-001 | **Protocol bridge** — Migrate from TCP+AES to MQTT+TLS / *Bridge protocollo — Migrare da TCP+AES a MQTT+TLS* | Gateway / FW | MEDIUM |
| FE-002 | Device detail page — variables, settings, event history / *Pagina dettaglio dispositivo — variabili, impostazioni, storico eventi* | Frontend | MEDIUM |
| FE-003 | Per-user "Inactive" device flag / *Flag dispositivo "Inattivo" per utente* | Frontend | LOW |
| FE-004 | In-app notifications panel / *Pannello notifiche in-app* | Frontend | LOW |
| API-001 | Device settings read/write API / *API lettura/scrittura impostazioni dispositivo* | Backend | MEDIUM |
| API-002 | Event history with filtering and pagination / *Storico eventi con filtro e paginazione* | Backend | MEDIUM |
| MOB-001 | Replicate v2 web features to Flutter app / *Replicare funzionalita' web v2 nell'app Flutter* | Mobile | FUTURE |
| MOB-002 | Push notifications / *Notifiche push* | Mobile | FUTURE |
| AI-001 | Predictive maintenance models / *Modelli manutenzione predittiva* | Analytics | FUTURE |

---

## Infrastructure / *Infrastruttura*

| ID | Description / *Descrizione* | Status |
|---|---|---|
| INF-001 | **Branch cleanup** — Delete 163 stale branches (approved by Fabio) / *Pulizia branch — Eliminare 163 branch obsoleti (approvato da Fabio)* | TO DO |
| INF-002 | **Close 14 stale PRs** from Capaciteam / *Chiudere 14 PR obsolete di Capaciteam* | TO DO |
| INF-003 | **Remove Capaciteam members** from GitHub org / *Rimuovere membri Capaciteam dall'org GitHub* | TO DO |
| INF-004 | **Terraform IaC** — 6 missing modules (VPC, ELB, ECS, ECR, S3, SQS) / *Terraform IaC — 6 moduli mancanti* | BACKLOG |
| INF-005 | **API documentation** — Currently empty, needs full spec / *Documentazione API — Attualmente vuota, serve specifica completa* | BACKLOG |
| INF-006 | **DB schema documentation** — No schema docs exist / *Documentazione schema DB — Non esiste documentazione schema* | BACKLOG |
| INF-007 | Set `SEND_MAILS=false` on staging to prevent emails to real users / *Impostare `SEND_MAILS=false` su staging per evitare email a utenti reali* | TO DO |
| INF-008 | **Staging gateway DNS** — Create `stg-s2.dropsa.com` endpoint for test devices / *DNS gateway staging — Creare endpoint `stg-s2.dropsa.com` per device di test* | TO DO |
| INF-009 | **Test device pool** — Yan prepares 5-10 devices, register MAC addresses for staging / *Pool device test — Yan prepara 5-10 device, registra MAC address per staging* | WAITING (Yan) |
| INF-010 | **Auto-provisioning on staging** — New gateway accepts unknown devices on stg only / *Auto-provisioning su staging — Il nuovo gateway accetta device sconosciuti solo su stg* | BLOCKED by GW-001 |
| INF-011 | **Device Pool Manager on EngHelper** — Web panel to register test device MAC addresses, auto-sync to staging gateway/DB / *Pannello web su EngHelper per registrare MAC address device test, sync automatico verso gateway/DB staging* | TO DO |

---

## Firmware (BC Project) / *Firmware (Progetto BC)*

### In Progress / *In Corso*

| ID | Description / *Descrizione* | Owner | Priority |
|---|---|---|---|
| **BC-41** | **STG TCP socket error found on V0.58** | Yan | Medium |
| **BC-42** | **Bluetooth App validation missing in FW v0.58** | Yan | Medium |
| **BC-45** | **Smart 4 Refactor** — Major firmware restructure / *Ristrutturazione firmware principale* | Yan | **HIGH** |
| **BC-47** | **Check Rotation Sensor Switch implementation** | Yan | **HIGH** |

### Testing / *In Test*

| ID | Description / *Descrizione* | Owner | Priority |
|---|---|---|---|
| BC-22 | Incorrect State Values for WiFi Menu Parameters (IDs 40050) | Fabio | Medium |
| BC-26 | Motor overcurrent | Yan | Medium |
| BC-29 | Smart 4.0 Digital New Feature | Yan | **HIGH** |
| BC-30 | LCD: Cycle Pulses hidden issue | Yan | Medium |
| BC-31 | Firmware fails to send alarm value in var 100016 via Bluetooth | Fabio | Medium |
| BC-38 | Matrix: Bug fix on Bravo on SEP mode | Yan | Medium |
| BC-39 | Release FW v0.59 | Yan | Medium |

### To Do / *Da Fare*

| ID | Description / *Descrizione* | Owner | Priority |
|---|---|---|---|
| BC-37 | Matrix Document: Correct Errors and add Matrix description | Yan | Medium |
| BC-40 | **Release FW v0.60** | Yan | Medium |
| BC-46 | Move all repositories from 'SebDropsa' GitHub account to DropsA-SpA org | Yan | Medium |
| BC-48 | **Fix S-00365 FW GitHub Actions workflow** — Current CI is broken due to missing Docker Hub credentials for `sebdropsa/stm32-build-env` (private image, no access to `sebdropsa` account). Steps to fix: (1) Create a Dropsa company Docker Hub account; (2) Log into `sebdropsa` Docker Hub and pull the image: `docker pull sebdropsa/stm32-build-env:latest`; (3) Tag and push to company account: `docker tag sebdropsa/stm32-build-env:latest dropsa/stm32-build-env:latest && docker push dropsa/stm32-build-env:latest`; (4) Create a Docker Hub Access Token on the company account; (5) Update GitHub secrets `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` in `DropsA-SpA/S-00365-firmware`; (6) Update workflow to pull from `dropsa/stm32-build-env:latest` | Yan | **HIGH** |

### Backlog

| ID | Description / *Descrizione* | Owner | Priority |
|---|---|---|---|
| BC-5 | TCP sockets ACKs are missed | Yan | Low |
| BC-12 | Retrieve and display older event records | Yan | **HIGH** |
| BC-14 | Settings changed on the Web do not sync to the phone | Yan | **HIGH** |
| BC-16 | BLE disconnection during FW upgrading | Yan | Medium |
| BC-18 | WiFi SSID and Password State "Error" | Yan | Medium |
| BC-23 | System Optimization | Yan | Medium |
| BC-24 | TOPLAND Touch Screen evaluation | Yan | Lowest |
| BC-34 | ESP32 firmware disconnects after successful MTU 517 negotiation | Yan | **HIGH** |
| **BC-49** | **[Hotfix] BLE heap exhaustion causes system reboot on startup with CAN bus** — `PRJ_BLE_MSG_APP_QUEUE_SIZE` increased from 2→5 combined with new CAN bus tasks exhausts FreeRTOS heap (64 KB) before BLE init completes. Fix: revert queue size to 2 or reduce Lubrication FSM stack (8192→6144 bytes). Affected: v0.59, CAN bus protocol only. | Yan | **HIGH** |

### Suspended / *Sospesi*

| ID | Description / *Descrizione* | Owner | Priority |
|---|---|---|---|
| BC-25 | RTC timeout during Power off | Yan | Medium |
| BC-27 | CANbus pause command does not stop timeout countdown | Yan | Medium |

### Recently Completed / *Completati Recentemente*

| ID | Description / *Descrizione* | Owner |
|---|---|---|
| BC-43 | LCD: Memory Leak in Parameter Converter | Yan |
| BC-44 | Move Parameters from ADVANCES to BASE | Fabio |
| BC-36 | ModBus communication debug | Yan |
| BC-35 | Smart 4.0 Digital EMC FW prepare | Yan |
| BC-33 | Add Bravo Compact firmware for staging environment to parallel test | Yan |
| BC-32 | Revert QR code endpoint from www.dropsa.app to drucs.net | Yan |

---

## How to Add Items / *Come Aggiungere Elementi*

Team members can add requests through:

1. **Field Engineers**: Use the [Request a Feature](../field/feature-request.md) or [Report an Issue](../field/report-issue.md) pages
2. **Dev team**: Edit this page directly via GitHub (`docs/backlog/index.md`)
3. **Anyone**: Email fabio.bigliardi@dropsa.com

*I membri del team possono aggiungere richieste tramite:*

1. *Tecnici sul campo: Usare le pagine [Richiedi Funzionalita'](../field/feature-request.md) o [Segnala Problema](../field/report-issue.md)*
2. *Team sviluppo: Modificare questa pagina direttamente da GitHub (`docs/backlog/index.md`)*
3. *Chiunque: Email a fabio.bigliardi@dropsa.com*
