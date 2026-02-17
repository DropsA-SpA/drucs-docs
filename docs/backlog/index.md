# Pending Requests & Fixes / *Richieste e Fix in Sospeso*

!!! info "Last updated: 17 February 2026 / *Ultimo aggiornamento: 17 Febbraio 2026*"
    This page tracks all open bugs, feature requests, and infrastructure tasks across the DRUCS platform.

    *Questa pagina traccia tutti i bug aperti, le richieste di funzionalita' e le attivita' infrastrutturali sulla piattaforma DRUCS.*

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

---

## Firmware (BC Project) / *Firmware (Progetto BC)*

| ID | Description / *Descrizione* | Owner | Status |
|---|---|---|---|
| BC-45 | **Smart 4 refactor** — Major firmware restructure / *Refactor Smart 4 — Ristrutturazione firmware principale* | Yan | In Progress |
| BC-FW060 | **Firmware v0.60** — Next release / *Firmware v0.60 — Prossima release* | Yan | To Do |
| BC-FW059 | Firmware v0.59 — Current release in testing / *Firmware v0.59 — Release corrente in test* | Yan | Testing |

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
