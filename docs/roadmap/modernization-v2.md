# DRUCS Unified Modernization Roadmap v2.0
# DRUCS Roadmap Unificata di Modernizzazione v2.0

**Author / Autore:** Walter Divisi
**Date / Data:** February 2026 / Febbraio 2026
**Version / Versione:** 2.0
**Classification / Classificazione:** Internal — Confidential / Interno — Riservato
**Status / Stato:** Draft for Review / Bozza per Revisione

---

> **What's new in v2.0 / Novità nella v2.0:**
> Full security audit findings with file paths · Predictive maintenance architecture · OTA deployment strategy for 2,000 intermittent devices · AWS cost analysis · Updated effort estimates.
> Findings dell'audit di sicurezza con path dei file · Architettura di manutenzione predittiva · Strategia OTA per 2.000 device intermittenti · Analisi dei costi AWS · Stime di effort aggiornate.

---

## Table of Contents / Indice

1. [Part 1: Platform Overview / Panoramica della Piattaforma](#part-1)
2. [Part 2: What's Wrong — Security Audit & Assessment / Cosa Non Va — Audit di Sicurezza e Valutazione](#part-2)
3. [Part 3: The Vision / La Visione](#part-3)
4. [Part 4: Modernization Roadmap / Roadmap di Modernizzazione](#part-4)
5. [Part 4b: OTA Deployment Strategy / Strategia di Deployment OTA *(new v2)*](#part-4b)
6. [Part 5: Timeline & Effort / Cronologia e Impegno](#part-5)
7. [Part 6: What You Get at the End / Cosa Si Ottiene alla Fine](#part-6)
8. [Appendix A: Repository Migration Plan / Piano di Migrazione dei Repository](#appendix-a)
9. [Appendix B: Technology Stack Comparison / Confronto Stack Tecnologico](#appendix-b)
10. [Appendix C: Risk Register / Registro dei Rischi](#appendix-c)
11. [Appendix D: Cost Analysis / Analisi dei Costi *(new v2)*](#appendix-d)

---

<a name="part-1"></a>
# PART 1: PLATFORM OVERVIEW / PANORAMICA DELLA PIATTAFORMA

## 1.1 System Architecture / Architettura del Sistema

The DRUCS platform connects STM32-based lubrication devices to a cloud backend via WiFi or BLE, providing remote monitoring, configuration, and control of industrial lubrication systems.

La piattaforma DRUCS connette dispositivi di lubrificazione basati su STM32 a un backend cloud via WiFi o BLE, fornendo monitoraggio remoto, configurazione e controllo di sistemi di lubrificazione industriali.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DRUCS SYSTEM ARCHITECTURE                         │
│                    ARCHITETTURA DEL SISTEMA DRUCS                    │
└─────────────────────────────────────────────────────────────────────┘

  STM32 Device (S-00365-firmware)
  ┌────────────────────────────────┐
  │  FreeRTOS + LVGL               │
  │  BSP: BLE, WiFi, LCD, Sensors  │
  │  AES-128-CFB (per-device key)  │◄── [VULN] Zero IV — see §2.1
  │  Heartbeat: 5,000 ms default   │
  │  OTA via variable ID 200010    │
  └───────────────┬────────────────┘
                  │ TCP :1256  (WiFi / AES encrypted)
                  │ BLE (PIN 4–6 digits, direct to mobile)
                  ▼
  ┌────────────────────────────────┐
  │  DRUCS Gateway                 │
  │  (drucs-gateway-new)           │
  │  Spring Boot 3 / Java 17       │◄── [VULN] Actuator exposed — §2.1
  │  TCP listener :1256            │
  │  Decrypts AES, parses var IDs  │
  └───────────────┬────────────────┘
                  │ AWS internal network
          ┌───────┴────────┐
          ▼                ▼
        AWS SQS       AWS SNS (push notifications)
          │
          ▼
  ┌────────────────────────────────┐
  │  Core API (import-web-app)     │
  │  Laravel 12 / PHP 8.4          │
  │  REST + JWT                    │◄── [VULN] SFTP key in config — §2.1
  │  Queue workers (5 queues)      │
  └───────┬────────────────────────┘
          │ HTTPS
  ┌───────┴──────────────────────────────────────┐
  │                                              │
  ▼                                              ▼
Frontend v2                               Mobile App
(React 19 + Vite)                         (Flutter/Dart)
app.dropsa.app/v2/                        iOS + Android

  Persistence / Persistenza:
  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
  │  PostgreSQL  │  │    Redis     │  │      S3      │
  │  (RDS)       │  │(ElastiCache) │  │  (files/OTA) │
  └──────────────┘  └──────────────┘  └──────────────┘
```

**Operational numbers / Dati operativi:**

| Metric / Metrica | Value / Valore |
|---|---|
| Devices in the field / Device in campo | ~2,000 |
| Permanently online / Permanentemente online | ~10 |
| Default heartbeat / Heartbeat default | 5,000 ms |
| Command delivery model / Modello di consegna comandi | Poll-based (next heartbeat) / Polling (prossimo heartbeat) |
| BLE pairing security / Sicurezza pairing BLE | PIN 4–6 digits / PIN 4–6 cifre |

---

## 1.2 Repository Map / Mappa dei Repository

| Folder | Status | Tech | Role / Ruolo |
|--------|--------|------|--------------|
| `import-web-app` | **Active** | PHP 8.4, Laravel 12 | Core API — single backend for web and mobile / API unica per web e mobile |
| `frontend-web` | **Retired** | Angular 17 | Old web dashboard (replaced by drucs-v2) / Vecchio dashboard (sostituito da drucs-v2) |
| `mobile` | **Active** | Flutter/Dart | iOS + Android app |
| `drucs-gateway-new` | **Stalled** | Java 17, Spring Boot 3 | Device TCP gateway / Gateway TCP dispositivi |
| `S-00365-firmware` | **Active** | C, FreeRTOS, LVGL | STM32L496 device firmware / Firmware dispositivo STM32L496 |
| `MEV2_bootloader` | **Active** | C, STM32L4 HAL | STM32L496 OTA bootloader |
| `S-00349-LP_sensor` | **Active** | C, STM32F1 HAL | Level/pressure sensor firmware / Firmware sensore livello/pressione |
| `Omega` | **Active** | C, STM32CubeMX | OmegaPump STM32 project |
| `drucs-iac` | **Active** | Terraform + Terragrunt | AWS infrastructure (ECS, RDS, S3, SQS…) |
| `drucs-docs` | **Active** | MkDocs | Platform documentation / Documentazione piattaforma |
| `github-actions` | **Active** | YAML | Shared CI/CD workflow templates / Template CI/CD condivisi |
| `fastlane-ios-certs` | **Active** | Fastlane Match | iOS code signing / Firma codice iOS |
| `architecture` | **Reference** | Markdown | Architecture overview / Panoramica architettura |

---

## 1.3 Device Variable ID System / Sistema di ID Variabile Dispositivo

Every device communicates using numeric variable IDs. This is the shared contract between firmware, gateway, API, and UI. Full map: `drucs-docs/docs/platform/variable-map.md`

Ogni device comunica tramite ID variabile numerici. Questo è il contratto condiviso tra firmware, gateway, API e UI. Mappa completa: `drucs-docs/docs/platform/variable-map.md`

| Range | Type / Tipo |
|-------|------------|
| 1 – 99,999 | Settings (user-configurable) / Impostazioni (configurabili dall'utente) |
| 100,000 – 199,999 | Live data (read-only telemetry) / Dati live (telemetria in sola lettura) |
| 200,000 – 300,000 | Commands (actions) / Comandi (azioni) |

**Key variable IDs / ID variabile principali:**

| ID | Meaning / Significato |
|----|----------------------|
| `100100` | Total motor hours (seconds) / Ore motore totali (secondi) |
| `100101` | Pump rotations / Rotazioni pompa |
| `100017` | Reservoir level % / Livello serbatoio % |
| `100015` | Active warnings (semicolon-separated event IDs) / Avvisi attivi |
| `100016` | Active alarms (semicolon-separated event IDs) / Allarmi attivi |
| `200001` | Start lubrication / Avvia lubrificazione |
| `200002` | Stop lubrication / Ferma lubrificazione |
| `200010` | OTA firmware update / Aggiornamento firmware OTA |

---

## 1.4 Product Variants / Varianti di Prodotto

The same firmware (`S-00365-firmware`) is compiled with different `#define` flags per product.
Lo stesso firmware (`S-00365-firmware`) è compilato con `#define` diversi per prodotto.

| Code / Codice | Product / Prodotto | BLE Prefix |
|---------------|--------------------|------------|
| BC | BravoCompact 4.0 | `BC_` |
| BR | Bravo 4.0 | `BR_` |
| SM | Smart 4.0 | `SM_` |
| OM | OmegaPump | `OM_` |
| ST | SilentTrack | `ST_` |
| NV | VipAir 4.0 | `NV_` |
| MA | Maxtreme | `MA_` |
| VP | VIP6 | `VP_` |

---

<a name="part-2"></a>
# PART 2: WHAT'S WRONG — SECURITY AUDIT & ASSESSMENT / COSA NON VA — AUDIT DI SICUREZZA E VALUTAZIONE

## 2.1 Critical Security Vulnerabilities / Vulnerabilità di Sicurezza Critiche

> **NEW IN V2 / NUOVO IN V2** — Full audit findings with exact file paths and remediation actions.

The security review conducted in February 2026 identified five critical vulnerabilities across the platform stack. All must be addressed before any cloud infrastructure expansion.

La revisione di sicurezza condotta a febbraio 2026 ha identificato cinque vulnerabilità critiche nello stack della piattaforma. Tutte devono essere risolte prima di qualsiasi espansione dell'infrastruttura cloud.

---

### Vulnerability 1 / Vulnerabilità 1: AES-128-CFB Zero IV

| Field / Campo | Value / Valore |
|---------------|----------------|
| **Severity / Gravità** | CRITICAL |
| **File** | `BSP/Connection/Wifi/socket.c:435` |
| **Repository** | `S-00365-firmware` |
| **Description / Descrizione** | AES-128-CFB encryption uses a zero Initialization Vector (IV) for all devices. Using a fixed IV with a stream cipher makes the keystream deterministic and allows attackers to XOR two ciphertexts from the same device to recover plaintext. / La cifratura AES-128-CFB usa un Initialization Vector (IV) zero per tutti i device. L'uso di un IV fisso con un cifrario a flusso rende il keystream deterministico e permette agli attaccanti di fare XOR tra due ciphertext dello stesso device per recuperare il plaintext. |
| **Action / Azione** | Generate a random IV per-message, prepend to payload. Update gateway to read IV from first 16 bytes. Use dual-format protocol during rollout (see §4b). / Generare un IV casuale per messaggio, anteporlo al payload. Aggiornare il gateway per leggere l'IV dai primi 16 byte. Usare protocollo dual-format durante il rollout (vedi §4b). |

---

### Vulnerability 2 / Vulnerabilità 2: Hardcoded AWS Access Key

| Field / Campo | Value / Valore |
|---------------|----------------|
| **Severity / Gravità** | CRITICAL |
| **File** | `drucs-web-services/login.php:56` (archived repo) |
| **Repository** | `drucs-web-services` (archived — key may still be active) |
| **Key ID** | `AKIASSAD2NHHUS6X34LK` |
| **Description / Descrizione** | An AWS IAM access key is hardcoded in source code in the deprecated PHP API. Even though the repo is deprecated, the key may still be active and the repository may be public or accessible to unauthorized parties. / Una chiave di accesso AWS IAM è hardcoded nel codice sorgente della vecchia API PHP. Anche se il repository è deprecato, la chiave potrebbe essere ancora attiva e il repository accessibile a parti non autorizzate. |
| **Action / Azione** | Immediately rotate the AWS key `AKIASSAD2NHHUS6X34LK` via AWS IAM console. Audit CloudTrail for unauthorized usage since the key was committed. / Ruotare immediatamente la chiave AWS `AKIASSAD2NHHUS6X34LK` via console AWS IAM. Auditare CloudTrail per eventuali usi non autorizzati da quando la chiave è stata committata. |

---

### Vulnerability 3 / Vulnerabilità 3: SFTP Private Key in Application Config

| Field / Campo | Value / Valore |
|---------------|----------------|
| **Severity / Gravità** | HIGH |
| **File** | `import-web-app/config/filesystems.php:109` |
| **Repository** | `import-web-app` |
| **Description / Descrizione** | An SFTP private key (or its path/passphrase) is stored in the Laravel filesystem configuration file, which is typically committed to version control. This exposes the key to anyone with repository access. / Una chiave privata SFTP (o il suo path/passphrase) è memorizzata nel file di configurazione del filesystem Laravel, tipicamente committato in version control. Questo espone la chiave a chiunque abbia accesso al repository. |
| **Action / Azione** | Move the SFTP private key to AWS SSM Parameter Store as a `SecureString`. Reference it via environment variable in `config/filesystems.php`. Remove from git history using `git filter-repo`. / Spostare la chiave privata SFTP in AWS SSM Parameter Store come `SecureString`. Referenziarla via variabile d'ambiente in `config/filesystems.php`. Rimuoverla dalla storia git con `git filter-repo`. |

---

### Vulnerability 4 / Vulnerabilità 4: Spring Boot Actuator Exposed Without Authentication

| Field / Campo | Value / Valore |
|---------------|----------------|
| **Severity / Gravità** | HIGH |
| **File** | `drucs-gateway-new/src/main/resources/application.yml:29` |
| **Repository** | `drucs-gateway-new` |
| **Description / Descrizione** | Spring Boot Actuator endpoints are exposed without authentication. This allows unauthenticated access to `/actuator/health`, `/actuator/env`, `/actuator/mappings`, and potentially `/actuator/shutdown`, exposing internal application state, environment variables, and configuration. / Gli endpoint di Spring Boot Actuator sono esposti senza autenticazione. Ciò consente l'accesso non autenticato a `/actuator/health`, `/actuator/env`, `/actuator/mappings` e potenzialmente `/actuator/shutdown`, esponendo lo stato interno dell'applicazione, le variabili d'ambiente e la configurazione. |
| **Action / Azione** | Add Spring Security dependency. Configure Actuator to expose only `/health` publicly and require `ROLE_ACTUATOR` for all other endpoints. Restrict access to internal VPC network only. / Aggiungere la dipendenza Spring Security. Configurare Actuator per esporre solo `/health` pubblicamente e richiedere `ROLE_ACTUATOR` per tutti gli altri endpoint. Limitare l'accesso alla rete VPC interna. |

---

### Vulnerability 5 / Vulnerabilità 5: Protocol-Level Weaknesses

| Field / Campo | Value / Valore |
|---------------|----------------|
| **Severity / Gravità** | MEDIUM |
| **Description / Descrizione** | Custom binary TCP protocol with no replay attack protection (no timestamp/nonce in AES payload), no message integrity verification (no HMAC), and no certificate-based mutual authentication. / Protocollo TCP binario custom senza protezione da replay attack (nessun timestamp/nonce nel payload AES), senza verifica dell'integrità dei messaggi (nessun HMAC) e senza autenticazione mutuale basata su certificati. |
| **Action / Azione** | Long-term: migrate to MQTT/TLS 1.3 with certificate-based device authentication (Phase 5). Short-term: add nonce/timestamp to AES payload as part of the IV fix. / Lungo termine: migrare a MQTT/TLS 1.3 con autenticazione dispositivo basata su certificati (Fase 5). Breve termine: aggiungere nonce/timestamp al payload AES come parte del fix dell'IV. |

---

## 2.2 Protocol Weaknesses / Debolezze del Protocollo

The current communication model is poll-based: devices send a heartbeat and receive pending commands only on the response. This means:

Il modello di comunicazione attuale è basato su polling: i device inviano un heartbeat e ricevono i comandi in sospeso solo nella risposta. Questo significa:

- **Command latency / Latenza comandi:** Up to 5,000 ms (heartbeat interval) before a command is received. / Fino a 5.000 ms (intervallo heartbeat) prima che un comando venga ricevuto.
- **No real-time telemetry / Nessuna telemetria real-time:** Web dashboard and mobile app must poll the API; they cannot receive live data events. / Dashboard web e app mobile devono fare polling sull'API; non possono ricevere eventi dati live.
- **No backpressure / Nessuna backpressure:** If the device goes offline, commands queue indefinitely with no expiry policy. / Se il device va offline, i comandi si accumulano indefinitamente senza policy di scadenza.
- **Custom binary format / Formato binario custom:** Difficult to debug, no standard tooling, tight coupling between gateway and firmware. / Difficile da debuggare, nessun tooling standard, forte accoppiamento tra gateway e firmware.

---

## 2.3 API & Backend Scalability Issues / Problemi di Scalabilità API & Backend

| Issue / Problema | Current State / Stato Attuale | Impact / Impatto |
|-----------------|-------------------------------|-----------------|
| No connection pooling / Nessun connection pooling | PHP-FPM + PostgreSQL, new connection per request / PHP-FPM + PostgreSQL, nuova connessione per richiesta | DB bottleneck at scale / Collo di bottiglia DB in scala |
| No WebSocket support / Nessun supporto WebSocket | Stateless Laravel HTTP only / Solo Laravel HTTP stateless | Real-time impossible without polling / Real-time impossibile senza polling |
| No time-series storage / Nessuno storage time-series | Telemetry in PostgreSQL relational tables / Telemetria in tabelle relazionali PostgreSQL | Slow historical queries, no analytics / Query storiche lente, nessun analytics |
| Single queue worker pool / Pool worker singolo | 5 named queues, all on same ECS tasks / 5 code con nome, tutte sugli stessi task ECS | No priority isolation / Nessun isolamento priorità |

---

<a name="part-3"></a>
# PART 3: THE VISION / LA VISIONE

## 3.1 Strategic Goals / Obiettivi Strategici

The modernization of DRUCS aims to transform the platform from a product-specific monitoring tool into a generic, extensible industrial IoT platform capable of supporting new product lines, predictive maintenance, and real-time operations at 10× the current device count.

La modernizzazione di DRUCS mira a trasformare la piattaforma da uno strumento di monitoraggio specifico per prodotto in una piattaforma IoT industriale generica ed estensibile, capace di supportare nuove linee di prodotto, manutenzione predittiva e operazioni real-time a 10× il numero attuale di device.

**Vision pillars / Pilastri della visione:**

1. **Security-first / Sicurezza prima di tutto** — All critical vulnerabilities resolved before expanding features.
2. **Real-time communications / Comunicazioni real-time** — MQTT/WebSocket replacing custom TCP polling.
3. **Generic IoT platform / Piattaforma IoT generica** — Support any device type via variable ID contracts, not hardcoded product logic.
4. **Predictive maintenance / Manutenzione predittiva** — Analytics pipeline to prevent device failures before they occur.
5. **Cloud-native scalability / Scalabilità cloud-native** — ECS auto-scaling, connection pooling, managed time-series storage.

---

## 3.2 Target Architecture / Architettura Target

```
  STM32 Device (updated firmware)
  ┌────────────────────────────────┐
  │  transport.h abstraction layer │
  │  MQTT/TLS 1.3 (target)        │
  │  or TCP/AES-v2 (transition)    │
  │  Random IV per message         │
  └───────────────┬────────────────┘
                  │
        ┌─────────┴────────────┐
        ▼                      ▼
  AWS IoT Core            NestJS Protocol Bridge
  (MQTT broker)           (TCP legacy support)
        │                      │
        └─────────┬────────────┘
                  │ MQTT internal
                  ▼
  ┌────────────────────────────────┐
  │  NestJS Gateway Service        │
  │  WebSocket (Socket.IO)         │
  │  Real-time event forwarding    │
  └───────────────┬────────────────┘
                  │
    ┌─────────────┼─────────────────┐
    ▼             ▼                 ▼
  New API       Analytics         Web/Mobile
  (NestJS +     Pipeline          Clients
  Prisma +      (below)           (WebSocket
  PostgreSQL)                      real-time)

  Analytics Pipeline / Pipeline Analytics:
  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
  │ Kinesis  │→ │Timestream│→ │    S3    │→ │ Athena / │
  │  Data    │  │(real-time│  │(long-    │  │SageMaker │
  │ Streams  │  │ queries) │  │ term)    │  │(ML/pred.)│
  └──────────┘  └──────────┘  └──────────┘  └──────────┘
```

---

## 3.3 Predictive Maintenance Architecture / Architettura di Manutenzione Predittiva

> **NEW IN V2 / NUOVO IN V2**

The analytics and predictive maintenance layer represents a significant value-add for DRUCS customers, enabling proactive service before device failure.

Il layer di analytics e manutenzione predittiva rappresenta un significativo valore aggiunto per i clienti DRUCS, abilitando interventi proattivi prima che il device si guasti.

### Data Pipeline / Pipeline dei Dati

```
Device Telemetry                    Predictive Output
(variable IDs)                      (action triggers)
     │                                     ▲
     ▼                                     │
┌─────────────┐    ┌──────────────┐   ┌───┴───────────┐
│ IoT Core /  │───▶│   Kinesis    │──▶│  SageMaker    │
│  NestJS GW  │    │ Data Streams │   │  (ML models)  │
└─────────────┘    └──────┬───────┘   └───────────────┘
                          │                    ▲
                          ▼                    │
               ┌──────────────────┐   ┌───────┴───────┐
               │   Timestream DB  │──▶│  Athena (SQL) │
               │ (30-day hot tier)│   │   on S3 data  │
               └──────────────────┘   └───────────────┘
                          │
                          ▼
                   ┌──────────┐
                   │    S3    │
                   │(Parquet, │
                   │long-term)│
                   └──────────┘
```

### Three-Tier Prediction Model / Modello di Predizione a Tre Livelli

**Tier 1: Rule-Based Alerting / Alerting Basato su Regole** *(Weeks 10-16 / Settimane 10-16)*
- Threshold-based on variable IDs. Simple, immediate, no ML required.
- Basato su soglie sugli ID variabile. Semplice, immediato, nessun ML richiesto.
- Example / Esempio: `100017` (reservoir level) < 15% → schedule refill alert / livello serbatoio < 15% → pianifica alert di rifornimento.

**Tier 2: Time-Series Anomaly Detection / Rilevamento Anomalie Time-Series** *(Phase 4 / Fase 4)*
- Statistical models on Timestream data. Detect degradation patterns over days/weeks.
- Modelli statistici su dati Timestream. Rileva pattern di degradazione su giorni/settimane.
- Key signals / Segnali chiave: `100101` rotation count trends, motor hours `100100` vs expected service intervals.

**Tier 3: ML-Based Failure Prediction / Predizione Guasti ML** *(Phase 6 / Fase 6)*
- SageMaker models trained on multi-device historical data. Predict remaining useful life (RUL).
- Modelli SageMaker addestrati su dati storici multi-device. Predice la vita utile residua (RUL).

### Key Variable IDs for ML / ID Variabile Chiave per ML

| Variable ID | Signal | ML Relevance / Rilevanza ML |
|------------|--------|----------------------------|
| `100100` | Total motor hours (sec) | Primary wear indicator / Indicatore principale di usura |
| `100101` | Pump rotation count | Mechanical stress accumulation / Accumulo stress meccanico |
| `100017` | Reservoir level % | Consumption rate trends / Trend tasso di consumo |
| `100015` | Active warnings | Precursor event sequence / Sequenza eventi precursore |
| `100016` | Active alarms | Failure correlation / Correlazione guasti |

---

<a name="part-4"></a>
# PART 4: MODERNIZATION ROADMAP / ROADMAP DI MODERNIZZAZIONE

## Overview / Panoramica

The roadmap is structured in 7 phases (0–6), ordered by risk and dependency. Security fixes must come first. Infrastructure and protocol work follows. Firmware changes come last to allow safe OTA rollout.

La roadmap è strutturata in 7 fasi (0–6), ordinate per rischio e dipendenza. I fix di sicurezza devono venire per primi. Il lavoro su infrastruttura e protocollo segue. Le modifiche al firmware vengono per ultime per consentire un rollout OTA sicuro.

```
Week:  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30
       │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │
Ph.0   ████                                                                                        Security fixes
Ph.1         ████████                                                                               Cloud scalability
Ph.2            ████████                                                                            NestJS bridge
Ph.3               █████████████                                                                    New API + real-time
Ph.4                        ████████████                                                            MQTT cloud + analytics
Ph.5                              ██████████████████████████████████████                           Firmware MQTT (zero-base)
Ph.6                                       █████████████████                                       Dashboard + analytics UI (parallel from Wk16)
```
Note: Ph.6 depends on Ph.4 analytics, NOT Ph.5 — runs in parallel from Week 16, unaffected.

---

## Phase 0: Security Fixes / Fase 0: Correzioni di Sicurezza
**Timeline / Tempi:** Week 1–2 / Settimana 1–2
**Effort / Impegno:** 1–2 days / giorni
**Priority / Priorità:** MANDATORY BEFORE ANYTHING ELSE / OBBLIGATORIO PRIMA DI TUTTO

### 0.1 Rotate AWS Key (Day 1, ~1 hour / Giorno 1, ~1 ora)

1. Log in to AWS IAM console and deactivate key `AKIASSAD2NHHUS6X34LK`.
2. Create a new key with minimum required permissions.
3. Store in AWS SSM Parameter Store as SecureString.
4. Check CloudTrail for unauthorized activity in the last 90 days.
---

1. Accedere alla console AWS IAM e disattivare la chiave `AKIASSAD2NHHUS6X34LK`.
2. Creare una nuova chiave con i permessi minimi necessari.
3. Conservarla in AWS SSM Parameter Store come SecureString.
4. Controllare CloudTrail per attività non autorizzate negli ultimi 90 giorni.

### 0.2 Move SFTP Key to SSM (Day 1–2, ~2 hours / Giorno 1–2, ~2 ore)

**File to modify / File da modificare:** `import-web-app/config/filesystems.php:109`

```php
// Before / Prima:
'private_key' => '/path/to/private.key',  // hardcoded

// After / Dopo:
'private_key' => env('SFTP_PRIVATE_KEY_PATH'),  // from SSM via ECS task definition env
```

Then remove the key from git history:
Poi rimuovere la chiave dalla storia git:

```bash
git filter-repo --path config/filesystems.php --force
# or use BFG Repo-Cleaner for simpler cases
```

### 0.3 Secure Spring Boot Actuator (Day 2, ~3 hours / Giorno 2, ~3 ore)

**File to modify / File da modificare:** `drucs-gateway-new/src/main/resources/application.yml:29`

```yaml
# Before / Prima:
management:
  endpoints:
    web:
      exposure:
        include: "*"

# After / Dopo:
management:
  endpoints:
    web:
      exposure:
        include: "health"   # only health is public / solo health è pubblico
  endpoint:
    health:
      show-details: never
```

Add Spring Security dependency and configure `ROLE_ACTUATOR` for internal endpoints. Restrict via AWS Security Groups to VPC-internal traffic only.

Aggiungere dipendenza Spring Security e configurare `ROLE_ACTUATOR` per endpoint interni. Limitare tramite AWS Security Groups al solo traffico interno VPC.

### 0.4 AES-128-CFB IV Fix — Dual-Format Strategy (Day 2–10 / Giorno 2–10)

This fix spans firmware and gateway and requires the OTA deployment strategy described in §4b. The gateway must be updated first to detect and support both the old (zero IV) and new (random IV) formats.

Questo fix copre firmware e gateway e richiede la strategia di deployment OTA descritta in §4b. Il gateway deve essere aggiornato per primo per rilevare e supportare entrambi i formati vecchio (IV zero) e nuovo (IV casuale).

**File to modify / File da modificare:** `BSP/Connection/Wifi/socket.c:435`

```c
// Before / Prima — zero IV (VULNERABLE):
uint8_t iv[16] = {0};
AES_CFB_Encrypt(plaintext, ciphertext, len, key, iv);

// After / Dopo — random IV prepended to payload (SECURE):
uint8_t iv[16];
TRNG_Generate(iv, 16);                           // hardware RNG
uint8_t packet[16 + len + 1];
packet[0] = 0x02;                                // protocol version byte
memcpy(packet + 1, iv, 16);                      // IV prepended
AES_CFB_Encrypt(plaintext, packet + 17, len, key, iv);
```

See §4b for the full deployment strategy and rollout phases.
Vedere §4b per la strategia di deployment completa e le fasi di rollout.

---

## Phase 1: Cloud Scalability / Fase 1: Scalabilità Cloud
**Timeline / Tempi:** Week 3–6 / Settimana 3–6
**Effort / Impegno:** 2–3 days / giorni
**Dependency / Dipendenza:** Phase 0 complete / Fase 0 completata

### Goals / Obiettivi

- Add PgBouncer connection pooler in front of PostgreSQL RDS to handle concurrent API connections efficiently.
- Migrate SQS queue configuration to use separate ECS task definitions per queue type for priority isolation.
- Enable RDS Performance Insights and configure CloudWatch alarms for P95 query latency.
---

- Aggiungere PgBouncer connection pooler davanti a PostgreSQL RDS per gestire efficientemente le connessioni API concorrenti.
- Migrare la configurazione delle code SQS per usare definizioni di task ECS separate per tipo di coda per l'isolamento delle priorità.
- Abilitare RDS Performance Insights e configurare allarmi CloudWatch per la latenza P95 delle query.

### Infrastructure Changes / Modifiche Infrastruttura

Add to `drucs-iac` Terraform modules: / Aggiungere ai moduli Terraform in `drucs-iac`:

```hcl
# PgBouncer ECS service (new module)
module "pgbouncer" {
  source         = "./modules/ecs-service"
  image          = "pgbouncer/pgbouncer:1.22"
  pool_mode      = "transaction"
  max_client_conn = 1000
  default_pool_size = 25
}
```

---

## Phase 2: NestJS Protocol Bridge / Fase 2: Bridge Protocollo NestJS
**Timeline / Tempi:** Week 4–8 / Settimana 4–8
**Effort / Impegno:** 3–4 days / giorni
**Dependency / Dipendenza:** Phase 1 complete / Fase 1 completata

### Rationale / Motivazione

Rather than rewriting the existing Java gateway (complex, risky), introduce a NestJS service that sits alongside it and provides:
1. WebSocket real-time event forwarding to web/mobile clients.
2. A typed MQTT internal bus for inter-service communication.
3. The foundation for the new REST API (Phase 3).

---

Piuttosto che riscrivere il gateway Java esistente (complesso, rischioso), si introduce un servizio NestJS che affianca il gateway esistente e fornisce:
1. Inoltro eventi WebSocket real-time a client web/mobile.
2. Un bus MQTT interno tipizzato per la comunicazione inter-servizio.
3. Le fondamenta per la nuova API REST (Fase 3).

### NestJS Service Structure / Struttura del Servizio NestJS

```
drucs-gateway-new/nestjs/
├── src/
│   ├── gateway/
│   │   ├── tcp-bridge.gateway.ts      # receives events from Java gateway via SQS/SNS
│   │   └── websocket.gateway.ts       # forwards to web/mobile via Socket.IO
│   ├── device/
│   │   ├── device.service.ts
│   │   └── device.dto.ts
│   ├── mqtt/
│   │   └── mqtt.service.ts            # internal MQTT broker (Mosquitto ECS)
│   └── app.module.ts
└── package.json
```

### Event Flow / Flusso degli Eventi

```
Java Gateway → SQS "device-events" → NestJS Bridge → Socket.IO
                                   ↓
                              MQTT internal bus → (Phase 3+) New API consumers
```

---

## Phase 3: New API + Real-Time / Fase 3: Nuova API + Real-Time
**Timeline / Tempi:** Week 6–12 / Settimana 6–12
**Effort / Impegno:** 10–12 days / giorni
**Dependency / Dipendenza:** Phase 2 complete / Fase 2 completata

### Migration Strategy / Strategia di Migrazione

The new API will run in parallel with the existing Laravel API (`import-web-app`). Migration will be feature-by-feature with traffic gradually shifted via the ALB. Laravel handles writes during the transition; NestJS handles reads and real-time subscriptions.

La nuova API sarà in esecuzione in parallelo con l'API Laravel esistente (`import-web-app`). La migrazione sarà feature-by-feature con traffico gradualmente spostato tramite ALB. Laravel gestisce le scritture durante la transizione; NestJS gestisce le letture e le subscription real-time.

### Tech Stack / Stack Tecnologico

| Concern | Choice | Reason / Motivo |
|---------|--------|----------------|
| Runtime | Node.js 22 LTS + NestJS 11 | Shared codebase with bridge service / Codebase condiviso con bridge service |
| ORM | Prisma 6 | Type-safe, excellent migration tooling / Type-safe, ottimi strumenti di migrazione |
| Database | PostgreSQL (same RDS instance) | No data migration needed initially / Nessuna migrazione dati necessaria inizialmente |
| Auth | JWT (same secret as Laravel) | Token compatibility during transition / Compatibilità token durante la transizione |
| Real-time | Socket.IO 4 | Client SDK for web and Flutter / Client SDK per web e Flutter |
| Validation | Zod + class-validator | Runtime type safety / Sicurezza dei tipi a runtime |

### API Structure / Struttura API

```
src/
├── devices/          # device CRUD, variable reads
├── telemetry/        # real-time subscriptions, historical queries
├── commands/         # send variable ID commands to devices
├── auth/             # JWT auth, user management
├── notifications/    # push notifications via SNS
└── analytics/        # aggregated queries (Phase 4+)
```

### Real-Time Subscription Protocol / Protocollo di Subscription Real-Time

```typescript
// Client subscribes to device telemetry
socket.emit('subscribe:device', { deviceId: 'BC_001234' });

// Server emits on new telemetry received
socket.on('telemetry', (data: {
  deviceId: string;
  variableId: number;
  value: number;
  timestamp: number;
}) => { /* update UI */ });
```

---

## Phase 4: MQTT Cloud Side + Analytics / Fase 4: MQTT Lato Cloud + Analytics
**Timeline / Tempi:** Week 10–16 / Settimana 10–16
**Effort / Impegno:** 3–4 days (MQTT) + 2 days (analytics) / giorni
**Dependency / Dipendenza:** Phase 3 API stable / API Fase 3 stabile

### MQTT Cloud Setup / Setup MQTT Cloud

Two options evaluated (see Appendix D for cost comparison):

Due opzioni valutate (vedi Appendice D per il confronto costi):

**Option A: AWS IoT Core** — Fully managed, best for < 5,000 devices. See Appendix D.
**Option B: Self-managed Mosquitto on ECS** — Better cost at scale (> 5,000 devices), more operational burden.
**Recommendation / Raccomandazione:** Start with AWS IoT Core. Evaluate self-managed at 5,000+ devices.

---

**Opzione A: AWS IoT Core** — Completamente gestito, meglio per < 5.000 device. Vedi Appendice D.
**Opzione B: Mosquitto self-managed su ECS** — Costo migliore in scala (> 5.000 device), più overhead operativo.
**Raccomandazione:** Iniziare con AWS IoT Core. Valutare il self-managed a 5.000+ device.

### Analytics Pipeline / Pipeline Analytics

```
IoT Core (MQTT) → Kinesis Data Streams → Lambda (transform)
                                       ↓
                              Amazon Timestream (30-day hot)
                                       ↓
                              S3 (Parquet, long-term archive)
                                       ↓
                              Athena (ad-hoc SQL queries)
                                       ↓
                              SageMaker (ML training, Phase 6)
```

**Timestream table design / Design tabella Timestream:**

```sql
CREATE TABLE drucs_telemetry (
  device_id     VARCHAR,      -- dimension
  product_code  VARCHAR,      -- dimension (BC, BR, SM, etc.)
  variable_id   BIGINT,       -- dimension
  value         DOUBLE,       -- measure
  time          TIMESTAMP     -- time
);
```

---

## Phase 5: Firmware MQTT + Transport Abstraction / Fase 5: Firmware MQTT + Astrazione Trasporto
**Timeline / Tempi:** Week 12–30 / Settimana 12–30
**Effort / Impegno:** 21.5–29 days / giorni
**Dependency / Dipendenza:** Phase 4 MQTT cloud stable / MQTT cloud Fase 4 stabile

> **Updated in V2.1 / Aggiornato in V2.1** — MQTT is implemented **entirely from scratch** in
> firmware. No MQTT code currently exists. The ESP32-WROOM-32E AT firmware v3.5.0.0 supports
> MQTT TLS natively via AT commands, but a pre-built binary constraint means no recompilation
> is possible — all certificate operations must be performed via AT commands already available
> in the distributed binary. A lab validation gate (§5.0) **must pass before development
> begins**. The current firmware uses a custom HTTP-like protocol over TCP/IP via ESP32 AT
> commands (port 1256, AES-128-CFB, `<ID><ACK><payload>` framing). MQTT payload format: JSON
> with fields `varId` / `value` / `ts`, compatible with AWS IoT Rules Engine.
> Certificate provisioning strategy: **X.509 Fleet Provisioning OTA** (provisioning claim cert
> pre-installed via OTA, device requests its final certificate on first MQTT boot).
>
> ---
>
> MQTT è implementato **interamente da zero** nel firmware. Non esiste attualmente alcun codice MQTT.
> Il firmware AT ESP32-WROOM-32E v3.5.0.0 supporta MQTT TLS nativamente tramite comandi AT, ma il
> vincolo del binario precompilato non consente ricompilazione — tutte le operazioni sui certificati
> devono essere eseguite tramite comandi AT già disponibili nel binario distribuito. Un gate di
> validazione lab (§5.0) **deve essere superato prima che lo sviluppo abbia inizio**. Il firmware
> attuale usa un protocollo HTTP-like custom su TCP/IP tramite comandi AT ESP32 (porta 1256,
> AES-128-CFB, framing `<ID><ACK><payload>`). Formato payload MQTT: JSON con campi `varId` /
> `value` / `ts`, compatibile con AWS IoT Rules Engine.
> Strategia di provisioning certificati: **X.509 Fleet Provisioning OTA** (certificato di
> provisioning claim pre-installato via OTA, il device richiede il proprio certificato finale
> al primo boot MQTT).

### Firmware Implementation Status / Stato Implementativo Firmware

| Layer | Status / Stato |
|---|---|
| ESP32 AT cmd — TCP/socket (`esp32.c`, `socket.c`) | Complete / Completo |
| ESP32 AT cmd — MQTT (`AT+MQTTPUB`, `AT+MQTTSUB`, `AT+MQTTCONN`, ...) | **Not implemented / Non implementato** |
| Inbound MQTT event parsing (`+MQTTSUBRECV`, `+MQTTCONNECTED`, ...) | Passive routing to WiFi queue only — no application logic |
| X.509 certificate loading onto ESP32 (`AT+SYSMFG`) | **Not implemented — lab validation required** |
| Transport abstraction `transport.h` | **Does not exist / Non esiste** |
| `mqtt_transport.c` | **Does not exist / Non esiste** |
| Certificate storage on W25Q32 (LittleFS) | **Not implemented / Non implementato** |
| JSON serializer/parser for payload | **Not implemented / Non implementato** |

### Sub-Tasks / Sotto-attività

#### 5.0 — [GO/NO-GO] Lab Validation: ESP32 MQTT TLS + Certificate Loading / Validazione Lab: ESP32 MQTT TLS + Caricamento Certificati
**Effort / Impegno:** 0.5–1 day / giorno | **Gate / Vincolo:** BLOCKING — no MQTT development before this passes / BLOCCANTE — nessuno sviluppo MQTT prima del superamento

Verify against the pre-built ESP32 AT v3.5.0.0 binary:
1. `AT+SYSMFG` (or equivalent) for runtime certificate write via UART from STM32
2. `AT+MQTTUSERCFG` scheme=5 (mutual TLS, client cert + CA root)
3. `AT+MQTTCONN` to a local MQTT broker on port 8883 with a self-signed certificate
4. `+MQTTSUBRECV` reception with 45-byte JSON payload

If `AT+SYSMFG` is unavailable: evaluate `AT+SYSFLASH`, the ESP32 NVS partition approach,
or fall back to server-side auth without mutual TLS.

---

Verificare sul binario ESP32 AT v3.5.0.0 precompilato:
1. `AT+SYSMFG` (o equivalente) per scrittura certificato a runtime via UART da STM32
2. `AT+MQTTUSERCFG` schema=5 (TLS mutuale, certificato client + CA root)
3. `AT+MQTTCONN` verso un broker MQTT locale su porta 8883 con certificato self-signed
4. Ricezione `+MQTTSUBRECV` con payload JSON di 45 byte

Se `AT+SYSMFG` non è disponibile: valutare `AT+SYSFLASH`, l'approccio tramite partizione NVS
di ESP32, oppure ricorrere ad autenticazione lato server senza TLS mutuale.

#### 5.1 — ESP32 MQTT AT Command Layer / Layer Comandi AT MQTT ESP32
**Files / File:** `Drivers/ESP32/esp32.c`, `Drivers/ESP32/esp32.h`, `Drivers/ESP32/tests/esp32_test.c`
**Effort / Impegno:** 4–5 days / giorni

New functions (prefix `prj_esp32_mqtt_*`): / Nuove funzioni (prefisso `prj_esp32_mqtt_*`):
- `AT+CIPSNTPCFG` + wait for `+TIME_UPDATED` — **mandatory TLS prerequisite** (not currently
  implemented; `+TIME_UPDATED` is already broadcast-routed in `esp32.c`) /
  **prerequisito obbligatorio per TLS** (non attualmente implementato; `+TIME_UPDATED` è già instradato in broadcast in `esp32.c`)
- `AT+SYSMFG` write — load cert/key/CA into ESP32 PKI slot 0 at runtime (official Espressif
  method for AWS IoT Core without recompiling AT firmware) /
  carica cert/chiave/CA nello slot PKI 0 di ESP32 a runtime (metodo ufficiale Espressif per AWS IoT Core senza ricompilare il firmware AT)
- `AT+MQTTUSERCFG` — scheme=5 (mTLS), cert_key_ID=0, CA_ID=0
- `AT+MQTTSNI` — **mandatory for AWS IoT Core** (multi-tenant TLS; call after `AT+MQTTUSERCFG`) /
  **obbligatorio per AWS IoT Core** (TLS multi-tenant; chiamare dopo `AT+MQTTUSERCFG`)
- `AT+MQTTCONNCFG` — LWT topic/payload, keepalive
- `AT+MQTTCONN` — broker connection (port 8883) / connessione al broker (porta 8883)
- `AT+MQTTDISCONN` — disconnection / disconnessione
- `AT+MQTTPUB` — publish QoS 0 (telemetry) / QoS 1 (ack, critical commands) /
  pubblica QoS 0 (telemetria) / QoS 1 (ack, comandi critici)
- `AT+MQTTSUB` / `AT+MQTTUNSUB` — subscribe/unsubscribe / sottoscrizione/annullamento sottoscrizione
- Event handlers for already-routed events: `+MQTTCONNECTED`, `+MQTTDISCONNECTED`,
  `+MQTTSUBRECV`, `+MQTTPUB:OK`, `+MQTTPUB:FAIL` /
  Gestori eventi per eventi già instradati: `+MQTTCONNECTED`, `+MQTTDISCONNECTED`,
  `+MQTTSUBRECV`, `+MQTTPUB:OK`, `+MQTTPUB:FAIL`
- Error logic for dictionary entries already present: `NO CERT FOUND`, `NO PRVT_KEY FOUND`,
  `NO CA FOUND` (already routed to WiFi queue — add error FSM state transitions) /
  Logica errori per voci dizionario già presenti: `NO CERT FOUND`, `NO PRVT_KEY FOUND`,
  `NO CA FOUND` (già instradate nella coda WiFi — aggiungere transizioni stato FSM per errori)

Full AT sequence for AWS IoT Core + mTLS boot: / Sequenza AT completa per boot AWS IoT Core + mTLS:
```
AT+SYSMFG=<write mqtt_ca>      // Amazon Root CA-1
AT+SYSMFG=<write mqtt_cert>    // device certificate
AT+SYSMFG=<write mqtt_key>     // device private key
AT+CIPSNTPCFG=1,0,"pool.ntp.org"
// wait +TIME_UPDATED
AT+MQTTUSERCFG=0,5,"<device_id>","","",0,0,""
AT+MQTTSNI=0,"<endpoint>.iot.<region>.amazonaws.com"
AT+MQTTCONN=0,"<endpoint>.iot.<region>.amazonaws.com",8883,1
AT+MQTTSUB=0,"drucs/<device_id>/commands",1
AT+MQTTSUB=0,"drucs/<device_id>/params/set",1
```

Unit tests with mock UART in `tests/` (follow pattern of existing `esp32_test.c`).
Unit test con UART mock in `tests/` (seguire il pattern di `esp32_test.c` esistente).

#### 5.2 — JSON Payload Serializer / Parser / Serializzatore e Parser Payload JSON
**Files / File:** new `BSP/Connection/Mqtt/mqtt_payload.c`, `BSP/Connection/Mqtt/mqtt_payload.h`
**Effort / Impegno:** 1–2 days / giorni

Stack-based (no `malloc`), minimal JSON, no external library: / Stack-based (nessun `malloc`), JSON minimale, nessuna libreria esterna:

```c
// Telemetry publish: {"varId":100017,"value":85,"ts":1741782000}
prj_status_t prj_mqtt_payload_serialize_var(
    prj_u32_t var_id, prj_i32_t value, prj_u32_t timestamp_s,
    prj_char_t *out_buf, prj_u16_t buf_len);

// Inbound command parse
prj_status_t prj_mqtt_payload_deserialize_cmd(
    const prj_char_t *json, prj_u32_t *act_id, prj_char_t *params_buf);
```

Parsing via `sscanf`/`strstr` (same approach as current protocol parser). No `malloc`.
Parsing tramite `sscanf`/`strstr` (stesso approccio del parser di protocollo attuale). Nessun `malloc`.

#### 5.3 — X.509 Certificate Storage + Fleet Provisioning / Storage Certificati X.509 + Fleet Provisioning
**Files / File:** `BSP/Storage/` (existing), new `BSP/Connection/Mqtt/mqtt_cert.c`
**Effort / Impegno:** 4–5 days / giorni

**Critical constraint:** `ESP32_MAX_MESSAGE_LEN = 1024` bytes. Fleet Provisioning response
from AWS IoT Core (`+MQTTSUBRECV` with device cert + private key) exceeds 1024 bytes in PEM.
**Solution:** dedicated 4096-byte provisioning buffer (stack-allocated in provisioning task),
using existing partial-message reassembly logic confirmed tested in `test_partial_message_parsing`.
The normal 1024-byte buffer remains unchanged for operational MQTT (JSON telemetry < 200 bytes).

**Vincolo critico:** `ESP32_MAX_MESSAGE_LEN = 1024` byte. La risposta di Fleet Provisioning da
AWS IoT Core (`+MQTTSUBRECV` con certificato device + chiave privata) supera 1024 byte in PEM.
**Soluzione:** buffer di provisioning dedicato da 4096 byte (allocato sullo stack nel task di
provisioning), utilizzando la logica di riassemblaggio messaggi parziali già testata in
`test_partial_message_parsing`. Il buffer normale da 1024 byte rimane invariato per il funzionamento
MQTT operativo (telemetria JSON < 200 byte).

Fleet Provisioning OTA flow: / Flusso OTA Fleet Provisioning:
1. OTA firmware update installs **provisioning claim cert** in LittleFS on W25Q32:
   `/certs/claim.crt`, `/certs/claim.key`, `/certs/root-ca.pem` /
   L'aggiornamento OTA installa il **certificato di provisioning claim** in LittleFS su W25Q32
2. First MQTT boot: load claim cert via `AT+SYSMFG` /
   Primo boot MQTT: carica il certificato claim tramite `AT+SYSMFG`
3. Sync NTP via `AT+CIPSNTPCFG` + wait `+TIME_UPDATED` /
   Sincronizza NTP tramite `AT+CIPSNTPCFG` + attesa `+TIME_UPDATED`
4. Connect to AWS IoT Core using claim cert /
   Connessione ad AWS IoT Core usando il certificato claim
5. Subscribe to `$aws/provisioning-templates/DRUCS/provisioning/json/accepted` /
   Sottoscrizione a `$aws/provisioning-templates/DRUCS/provisioning/json/accepted`
6. Publish to `$aws/provisioning-templates/DRUCS/provisioning/json` with device serial /
   Pubblicazione su `$aws/provisioning-templates/DRUCS/provisioning/json` con seriale device
7. Receive device cert + private key (via 4096-byte provisioning buffer) /
   Ricezione certificato device + chiave privata (tramite buffer di provisioning da 4096 byte)
8. Save to LittleFS: `/certs/device.crt`, `/certs/device.key` /
   Salvataggio su LittleFS: `/certs/device.crt`, `/certs/device.key`
9. Set "provisioned" flag via `prj_storage_registration_*`; subsequent boots use device cert /
   Impostazione flag "provisioned" tramite `prj_storage_registration_*`; i boot successivi usano il certificato device
10. Reload device cert via `AT+SYSMFG`, reconnect /
    Ricaricamento certificato device tramite `AT+SYSMFG`, riconnessione

Subsequent boots (already provisioned): read `/certs/device.crt` + `/certs/device.key` +
`/certs/root-ca.pem` from LittleFS → `AT+SYSMFG` write (~2–4s at 115200 baud, acceptable
for industrial use) → NTP sync → `AT+MQTTUSERCFG` + `AT+MQTTSNI` + `AT+MQTTCONN`.

Boot successivi (già provisioned): lettura `/certs/device.crt` + `/certs/device.key` +
`/certs/root-ca.pem` da LittleFS → scrittura `AT+SYSMFG` (~2–4s a 115200 baud, accettabile
per uso industriale) → sync NTP → `AT+MQTTUSERCFG` + `AT+MQTTSNI` + `AT+MQTTCONN`.

#### 5.4 — Transport Abstraction Layer / Layer di Astrazione Trasporto
**Files / File:** new `BSP/Connection/transport.h`, refactor `BSP/Connection/Wifi/socket.c`
**Effort / Impegno:** 2–3 days / giorni

```c
typedef struct {
    prj_status_t (*connect)(const prj_char_t *host, prj_u16_t port);
    prj_status_t (*send)(const prj_u8_t *data, prj_u16_t len);
    prj_status_t (*recv)(prj_u8_t *buf, prj_u16_t max_len, prj_u32_t timeout_ms);
    void         (*disconnect)(void);
} Transport_t;

extern Transport_t Transport_TCP_AES;   // socket.c wrapped
extern Transport_t Transport_MQTT_TLS;  // mqtt_transport.c (new)
extern Transport_t *ActiveTransport;    // selected at boot via #define
```

Transport selected at compile time via `#define TRANSPORT_MQTT` in Makefile/CMake
(same mechanism as existing product variants). No runtime feature flags.

Trasporto selezionato a tempo di compilazione tramite `#define TRANSPORT_MQTT` in Makefile/CMake
(stesso meccanismo delle varianti di prodotto esistenti). Nessun feature flag a runtime.

#### 5.5 — MQTT Topic Design / Design dei Topic MQTT
**Files / File:** new `BSP/Connection/Mqtt/mqtt_topics.h`; update `drucs-docs/docs/platform/variable-map.md`
**Effort / Impegno:** 1 day / giorno

| Direction / Direzione | Topic | QoS | ID Type / Tipo ID |
|---|---|---|---|
| Device → Cloud | `drucs/{device_id}/telemetry` | 0 | Var (100000+) |
| Device → Cloud | `drucs/{device_id}/params/ack` | 1 | Par (write ack) |
| Device → Cloud | `drucs/{device_id}/status` | 1 (LWT) | Online/Offline |
| Cloud → Device | `drucs/{device_id}/commands` | 1 | Act (200000+) |
| Cloud → Device | `drucs/{device_id}/params/set` | 1 | Par (1–99999) |

Client ID = `{BLE_prefix}_{serial_number}` (e.g., `BC_001234`) — available from
`prj_bsp_connection_get_device_name()`.

Client ID = `{BLE_prefix}_{serial_number}` (es. `BC_001234`) — disponibile tramite
`prj_bsp_connection_get_device_name()`.

#### 5.6 — Connection FSM Refactoring + mqtt_transport.c / Refactoring FSM di Connessione + mqtt_transport.c
**Files / File:** `BSP/Connection/webserver_connection.c`, `BSP/Connection/webserver_connection.h`,
new `BSP/Connection/Mqtt/mqtt_transport.c`
**Effort / Impegno:** 5–6 days / giorni

The current FSM handles AES encryption, message ID sequencing, HTTP-like framing, and ACK
handshake. With MQTT/TLS these are replaced: TLS by ESP32, sequencing by QoS 1, framing
eliminated, ACK by MQTT PubAck.

La FSM attuale gestisce cifratura AES, sequenziamento ID messaggio, framing HTTP-like e handshake
ACK. Con MQTT/TLS questi vengono sostituiti: TLS da ESP32, sequenziamento dal QoS 1, framing
eliminato, ACK da MQTT PubAck.

New FSM states: / Nuovi stati FSM:
`DISCONNECTED → CERT_LOADING → CONNECTING → CONNECTED → SUBSCRIBING → READY`

Additional logic: / Logica aggiuntiva:
- Reconnect with exponential backoff / Riconnessione con backoff esponenziale
- LWT: `drucs/{device_id}/status` payload `{"online":false}`
- Heartbeat: periodic publish to `drucs/{device_id}/status` `{"online":true}` (not relying
  solely on MQTT keepalive, for cloud-side device visibility) /
  pubblicazione periodica su `drucs/{device_id}/status` `{"online":true}` (non affidandosi al
  solo MQTT keepalive, per la visibilità del device lato cloud)

`webserver_connection_message_proc()` adapted to receive JSON payload instead of current
`<ID><ACK=NN><data>` format.

`webserver_connection_message_proc()` adattato per ricevere payload JSON al posto del formato
attuale `<ID><ACK=NN><data>`.

#### 5.7 — AWS IoT Core Firmware Config / Configurazione Firmware AWS IoT Core
**Dependency / Dipendenza:** Phase 4 complete — IoT Core endpoint available / Fase 4 completata — endpoint IoT Core disponibile
**Effort / Impegno:** 1–2 days / giorni

- IoT Core endpoint (e.g., `abcdefg-ats.iot.eu-west-1.amazonaws.com`) via compile-time
  `#define` or Flash storage parameter /
  Endpoint IoT Core (es. `abcdefg-ats.iot.eu-west-1.amazonaws.com`) tramite `#define` a tempo
  di compilazione o parametro storage Flash
- Port 8883 (MQTTS) / Porta 8883 (MQTTS)
- IoT Core policy `drucs-device-policy`: `iot:Publish` on `drucs/{clientId}/*`,
  `iot:Subscribe` on `drucs/{clientId}/*` /
  Policy IoT Core `drucs-device-policy`: `iot:Publish` su `drucs/{clientId}/*`,
  `iot:Subscribe` su `drucs/{clientId}/*`

#### 5.8 — Testing and OTA Validation / Test e Validazione OTA
**Files / File:** `tests/` (host-side unit tests / test host-side), physical lab devices / device fisici in laboratorio
**Effort / Impegno:** 3–4 days / giorni

- Unit tests: MQTT AT command layer with UART mock (follow `Drivers/ESP32/tests/esp32_test.c` pattern) /
  Unit test: layer comandi AT MQTT con UART mock (seguire pattern `Drivers/ESP32/tests/esp32_test.c`)
- Integration test on physical device: AWS IoT Core connect, publish, subscribe /
  Test di integrazione su device fisico: connessione AWS IoT Core, pubblicazione, sottoscrizione
- Fleet Provisioning end-to-end: claim cert → device cert /
  Fleet Provisioning end-to-end: certificato claim → certificato device
- Backward compat validation: `Transport_TCP_AES` must continue to function /
  Validazione compatibilità retroattiva: `Transport_TCP_AES` deve continuare a funzionare
- Canary OTA on 10 permanently-online devices (per §4b strategy) /
  OTA Canary su 10 device permanentemente online (secondo strategia §4b)
- MEV2 bootloader rollback test / Test di rollback bootloader MEV2

### Effort Summary / Riepilogo Effort

| Sub-task / Sotto-attività | Previous v2 / Precedente v2 | Revised v2.1 / Rivisto v2.1 | Delta |
|---|---|---|---|
| 5.0 Lab validation GO/NO-GO / Validazione lab GO/NO-GO | (not present / non presente) | 0.5–1 day / giorno | new — blocking gate / nuovo — gate bloccante |
| 5.1 ESP32 MQTT AT Layer / Layer AT MQTT ESP32 | (included generic / incluso generico) | 4–5 days / giorni | +1d: SNTP, SNI, AT+SYSMFG |
| 5.2 JSON serializer/parser / Serializzatore/parser JSON | (not present / non presente) | 1–2 days / giorni | new — protocol change / nuovo — cambio protocollo |
| 5.3 Cert Storage + Fleet Provisioning / Storage Certificati + Fleet Provisioning | (not present / non presente) | 4–5 days / giorni | new + 4096-byte buffer / nuovo + buffer 4096 byte |
| 5.4 Transport Abstraction / Astrazione Trasporto | ~3 days / giorni | 2–3 days / giorni | — |
| 5.5 Topic Design / Design Topic | ~1 day / giorno | 1 day / giorno | — |
| 5.6 Connection FSM + mqtt_transport.c | ~5 days / giorni | 5–6 days / giorni | — |
| 5.7 AWS IoT Core Config firmware / Configurazione firmware | ~3 days / giorni | 1–2 days / giorni | — |
| 5.8 Testing & OTA Validation / Test e Validazione OTA | ~3 days / giorni | 3–4 days / giorni | — |
| **TOTAL / TOTALE** | **12–15 days / giorni** | **21.5–29 days / giorni** | **+9–14 days / giorni** |

### OTA Rollout Integration / Integrazione Rollout OTA

The firmware MQTT update will be delivered via the existing OTA mechanism (variable ID `200010`)
following the phased strategy described in §4b. Phase 6 (Dashboard + Analytics UI) depends on
Phase 4 analytics, **not on Phase 5**, and proceeds in parallel from Week 16 — unaffected by
the Phase 5 extension.

L'aggiornamento firmware MQTT verrà distribuito tramite il meccanismo OTA esistente (ID variabile
`200010`) seguendo la strategia graduale descritta in §4b. La Fase 6 (Dashboard + Analytics UI)
dipende dagli analytics della Fase 4, **non dalla Fase 5**, e procede in parallelo dalla
Settimana 16 — non influenzata dall'estensione della Fase 5.

---

## Phase 6: IoT Dashboard + Analytics Integration / Fase 6: Dashboard IoT + Integrazione Analytics
**Timeline / Tempi:** Week 16–24 / Settimana 16–24
**Effort / Impegno:** 8–10 days / giorni
**Dependency / Dipendenza:** Phase 4 analytics pipeline running / Pipeline analytics Fase 4 in esecuzione

### Deliverables / Deliverable

- **Live telemetry dashboard / Dashboard telemetria live:** Real-time variable ID charts using Socket.IO subscriptions. / Grafici ID variabile real-time tramite subscription Socket.IO.
- **Historical analytics / Analytics storici:** Timestream + Athena-backed charts for motor hours, pump cycles, reservoir consumption. / Grafici basati su Timestream + Athena per ore motore, cicli pompa, consumo serbatoio.
- **Predictive maintenance alerts / Alert manutenzione predittiva:** Rule-based (Tier 1) and anomaly-based (Tier 2) alerts integrated into web and mobile notification flow. / Alert basati su regole (Livello 1) e anomalie (Livello 2) integrati nel flusso di notifiche web e mobile.
- **SageMaker model deployment / Deployment modello SageMaker:** Initial RUL (Remaining Useful Life) model for motor and pump, trained on accumulated historical data. / Modello RUL (Vita Utile Residua) iniziale per motore e pompa, addestrato sui dati storici accumulati.
- **Mobile analytics screens / Schermate analytics mobile:** Flutter feature following web-leads-mobile-follows principle. / Feature Flutter seguendo il principio web-leads-mobile-follows.

---

<a name="part-4b"></a>
# PART 4b: OTA DEPLOYMENT STRATEGY / STRATEGIA DI DEPLOYMENT OTA

> **NEW IN V2 / NUOVO IN V2**

## Context / Contesto

The DRUCS fleet presents a specific challenge for OTA firmware updates:

La flotta DRUCS presenta una sfida specifica per gli aggiornamenti firmware OTA:

- ~2,000 devices in the field / ~2.000 device in campo
- ~10 permanently online / ~10 permanentemente online
- Remaining ~1,990 connect intermittently (installation sites, maintenance periods, sleep modes) / I restanti ~1.990 si connettono in modo intermittente

This means a firmware update (e.g., the AES IV fix) cannot be assumed to reach all devices within any predictable timeframe. The system must remain backward-compatible until the last device updates.

Questo significa che un aggiornamento firmware (es. il fix AES IV) non può assumere di raggiungere tutti i device in un timeframe prevedibile. Il sistema deve rimanere retrocompatibile fino all'aggiornamento dell'ultimo device.

---

## Gateway-First Principle / Principio Gateway-First

**Rule / Regola:** Cloud infrastructure updates are ALWAYS deployed before the corresponding firmware update is rolled out. / Gli aggiornamenti all'infrastruttura cloud sono SEMPRE deployati prima che il corrispondente aggiornamento firmware venga distribuito.

This ensures that:
1. Old firmware still works after gateway updates.
2. New firmware is never deployed to a device whose gateway cannot handle it.

---

Questo garantisce che:
1. Il firmware vecchio funzioni ancora dopo gli aggiornamenti al gateway.
2. Il nuovo firmware non venga mai deployato a un device il cui gateway non può gestirlo.

---

## Protocol Version Detection / Rilevamento Versione Protocollo

The new protocol version byte `0x02` is the key to dual-format detection:

Il byte di versione protocollo `0x02` è la chiave del rilevamento dual-format:

```
Legacy packet (v1, zero IV) / Pacchetto legacy (v1, IV zero):
┌──────────────────────────────────┐
│ [AES-CFB ciphertext, IV=0x00...0]│
└──────────────────────────────────┘
  First byte after decryption can be anything

New packet (v2, random IV) / Pacchetto nuovo (v2, IV casuale):
┌────────┬────────────────┬─────────────────────────────────┐
│  0x02  │   IV (16 bytes)│  AES-CFB ciphertext (random IV)  │
└────────┴────────────────┴─────────────────────────────────┘
  Version byte (plaintext, not encrypted)
```

**Gateway detection logic / Logica di rilevamento gateway:**

```java
// In drucs-gateway-new: DeviceMessageParser.java
public DeviceMessage parse(byte[] rawPacket) {
    if (rawPacket[0] == 0x02) {
        // New format: read IV from bytes 1-16, decrypt bytes 17+
        byte[] iv = Arrays.copyOfRange(rawPacket, 1, 17);
        byte[] ciphertext = Arrays.copyOfRange(rawPacket, 17, rawPacket.length);
        return decryptAES_CFB(ciphertext, deviceKey, iv);
    } else {
        // Legacy format: decrypt with zero IV
        byte[] zeroIV = new byte[16];
        return decryptAES_CFB(rawPacket, deviceKey, zeroIV);
    }
}
```

---

## Rollout Phases / Fasi di Rollout

### Phase A: Gateway Update (Week 1–2 / Settimana 1–2)

Deploy updated gateway with dual-format support. Zero impact on existing devices. All devices continue to work normally.

Deployare il gateway aggiornato con supporto dual-format. Impatto zero sui device esistenti. Tutti i device continuano a funzionare normalmente.

**Verification / Verifica:**

```sql
-- Confirm all devices still reporting after gateway update
-- Confermare che tutti i device stiano ancora inviando report dopo l'aggiornamento gateway
SELECT COUNT(*) as active_devices
FROM device_heartbeats
WHERE last_seen > NOW() - INTERVAL '24 hours'
-- Expected: same as before gateway deploy / Atteso: stesso valore di prima del deploy
```

### Phase B: Gradual OTA Rollout (Week 2–16 / Settimana 2–16)

Use variable ID `200010` to trigger OTA on device groups. Prioritize permanently-online devices first, then expand.

Usare l'ID variabile `200010` per attivare OTA su gruppi di device. Dare priorità ai device permanentemente online, poi espandere.

```
Group / Gruppo   Size    Target                    Wait / Attesa
─────────────────────────────────────────────────────────────────
Canary           10      Permanently online       3 days / giorni
Stage 1          50      High-connectivity sites  1 week / settimana
Stage 2          250     Normal connectivity      2 weeks / settimane
Stage 3          690     Low connectivity         4 weeks / settimane
Tail             1,000   Intermittent / Offline   Monitor ongoing / Monitor continuo
```

**Rollout monitoring query / Query di monitoring rollout:**

```sql
SELECT
    firmware_version,
    COUNT(*) as device_count,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 1) as percentage
FROM devices
GROUP BY firmware_version
ORDER BY firmware_version DESC;
```

### Phase C: Legacy Support Cleanup (Month 6+ / Mese 6+)

When > 95% of devices are on v2, disable zero-IV support in gateway. Notify remaining device owners for manual intervention or field service.

Quando > 95% dei device è su v2, disabilitare il supporto IV-zero nel gateway. Notificare i proprietari dei device rimanenti per intervento manuale o assistenza in campo.

**Threshold for cleanup / Soglia per cleanup:** > 95% adoption OR > 12 months since Phase B start.
**Soglia per cleanup:** > 95% adozione O > 12 mesi dall'inizio della Fase B.

---

## OTA Deployment Risk Table / Tabella dei Rischi di Deployment OTA

| Risk / Rischio | Likelihood / Probabilità | Impact / Impatto | Mitigation / Mitigazione |
|----------------|-------------------------|-----------------|-------------------------|
| Device bricked during OTA / Device brickato durante OTA | Low / Bassa | Critical / Critico | MEV2 bootloader with rollback. Test on canary group first. / MEV2 bootloader con rollback. Test prima su gruppo canary. |
| Gateway rejects old firmware packets / Gateway rifiuta pacchetti firmware vecchi | High without dual-format / Alta senza dual-format | Critical / Critico | Dual-format detection is mandatory (Phase A). / Il rilevamento dual-format è obbligatorio (Fase A). |
| Long tail of offline devices / Lunga coda di device offline | Certain / Certo | Medium / Medio | 12-month legacy support window. Field service notification at 6 months. / Finestra di supporto legacy 12 mesi. Notifica assistenza in campo a 6 mesi. |
| OTA packet corruption / Corruzione pacchetto OTA | Low / Bassa | High / Alto | CRC32 verification in bootloader. S3 pre-signed URL with checksum. / Verifica CRC32 nel bootloader. URL pre-firmato S3 con checksum. |

---

<a name="part-5"></a>
# PART 5: TIMELINE & EFFORT / CRONOLOGIA E IMPEGNO

## Summary Table / Tabella Riepilogativa

| Phase / Fase | Duration / Durata | Effort / Impegno | Depends On / Dipende Da | Deliverable / Deliverable |
|--------------|------------------|-----------------|------------------------|--------------------------|
| 0: Security Fixes | Week 1–2 | 1–2 days | — | All 4 vulnerabilities resolved / 4 vulnerabilità risolte |
| 1: Cloud Scalability | Week 3–6 | 2–3 days | Phase 0 | PgBouncer, queue isolation / PgBouncer, isolamento code |
| 2: NestJS Bridge | Week 4–8 | 3–4 days | Phase 1 | WebSocket real-time, MQTT internal bus / WebSocket real-time, bus MQTT interno |
| 3: New API + Real-Time | Week 6–12 | 10–12 days | Phase 2 | NestJS API in parallel, live telemetry subscriptions / API NestJS in parallelo, subscription telemetria live |
| 4: MQTT Cloud + Analytics | Week 10–16 | 3–4 days + 2 days analytics | Phase 3 | IoT Core / Mosquitto, Kinesis, Timestream pipeline / Pipeline Kinesis, Timestream |
| 4b: OTA Strategy (execution) | Week 2–16+ | Ops effort / Effort ops | Phase 0 gateway | All 2,000 devices on v2 firmware / 2.000 device su firmware v2 |
| 5: Firmware MQTT | Week 12–30 | 21.5–29 days | Phase 4 | Lab gate (§5.0), ESP32 AT MQTT layer, JSON payload, X.509 Fleet Provisioning, transport abstraction, Connection FSM, AWS IoT config, OTA validation |
| 6: Dashboard + Analytics UI | Week 16–24 | 8–10 days | Phase 4 | Live dashboard, predictive alerts, SageMaker model / Dashboard live, alert predittivi, modello SageMaker |
| **TOTAL / TOTALE** | **~30 weeks** | **~49–66 days** | | **Full platform modernization / Modernizzazione completa piattaforma** |

> **Note / Nota:** Effort estimates are for a single experienced developer. / Le stime di effort sono per un singolo sviluppatore esperto. Actual timelines depend on team size and parallel execution. / I tempi effettivi dipendono dalla dimensione del team e dall'esecuzione parallela.

---

## Phase Dependency Graph / Grafo delle Dipendenze tra Fasi

```
[Phase 0: Security] ──────────┬──────────────────────────────────────┐
                               │                                      │
                    [Phase 1: Cloud Scalability]          [Phase 4b: OTA Execution]
                               │
                    [Phase 2: NestJS Bridge]
                               │
                    [Phase 3: New API + Real-Time]
                               │
                    [Phase 4: MQTT + Analytics] ──────────┐
                               │                          │
                    [Phase 5: Firmware MQTT]   [Phase 6: Dashboard + Analytics UI]
```

---

<a name="part-6"></a>
# PART 6: WHAT YOU GET AT THE END / COSA SI OTTIENE ALLA FINE

## Operational Benefits / Benefici Operativi

After successful completion of all phases, the DRUCS platform will have:

Al completamento di tutte le fasi, la piattaforma DRUCS avrà:

| Capability | Before / Prima | After / Dopo |
|-----------|---------------|-------------|
| Command delivery latency / Latenza consegna comandi | Up to 5,000ms / Fino a 5.000ms | < 500ms (MQTT push) |
| Real-time telemetry / Telemetria real-time | Not available / Non disponibile | WebSocket push to all clients / WebSocket push a tutti i client |
| Protocol security / Sicurezza protocollo | AES with zero IV (vulnerable) / AES con IV zero (vulnerabile) | AES random IV (transition) → MQTT/TLS 1.3 (final) |
| Authentication / Autenticazione | JWT + no device auth / JWT + nessuna auth device | JWT + certificate-based device auth / JWT + auth device basata su certificati |
| Telemetry storage / Storage telemetria | PostgreSQL relational / PostgreSQL relazionale | Timestream (hot) + S3 Parquet (cold) |
| Analytics / Analytics | None / Nessuno | Kinesis + Athena + SageMaker pipeline |
| Scalability / Scalabilità | ~2,000 devices | 20,000+ devices |
| New product onboarding / Onboarding nuovi prodotti | Code changes required / Modifiche al codice necessarie | Variable ID contract only / Solo contratto ID variabile |

## Predictive Maintenance Benefits / Benefici Manutenzione Predittiva

> **NEW IN V2 / NUOVO IN V2**

| Benefit / Beneficio | Description / Descrizione | Variable IDs Used |
|--------------------|--------------------------|-------------------|
| Reservoir refill scheduling / Pianificazione rifornimento serbatoio | Predict refill need 48h in advance / Predire necessità rifornimento 48h prima | `100017` |
| Pump wear prediction / Predizione usura pompa | Estimate remaining pump cycles / Stimare cicli pompa residui | `100101`, `100100` |
| Failure precursor detection / Rilevamento precursori guasto | Detect warning sequences before alarm / Rilevare sequenze di avviso prima dell'allarme | `100015`, `100016` |
| Service interval optimization / Ottimizzazione intervalli di servizio | Data-driven service scheduling / Pianificazione manutenzione basata sui dati | `100100` |
| Anomaly detection / Rilevamento anomalie | Statistical outliers vs. device population / Outlier statistici vs. popolazione device | All telemetry IDs / Tutti gli ID telemetria |

---

<a name="appendix-a"></a>
# APPENDIX A: REPOSITORY MIGRATION PLAN / PIANO DI MIGRAZIONE DEI REPOSITORY

## Repositories to Be Retired / Repository da Dismettere

| Repository | Action / Azione | Timeline / Tempi |
|-----------|----------------|-----------------|
| `frontend-web` (Angular 17) | Already retired. Remove from CI/CD pipeline if still present. / Già dismesso. Rimuovere dalla pipeline CI/CD se ancora presente. | Phase 3 / Fase 3 |

## Repositories to Be Created / Repository da Creare

| Repository | Purpose / Scopo | Phase / Fase |
|-----------|----------------|-------------|
| `drucs-gateway-v2` (or NestJS module) | NestJS Protocol Bridge + new API / Bridge Protocollo NestJS + nuova API | Phase 2–3 / Fase 2–3 |

## Repositories with Major Changes / Repository con Modifiche Importanti

| Repository | Changes / Modifiche |
|-----------|-------------------|
| `S-00365-firmware` | AES IV fix + transport abstraction + MQTT transport |
| `drucs-gateway-new` | Dual-format detection + Actuator security + MQTT support |
| `import-web-app` | SFTP key removal + parallel operation with new API |
| `drucs-iac` | PgBouncer module + IoT Core + Kinesis + Timestream resources |

---

<a name="appendix-b"></a>
# APPENDIX B: TECHNOLOGY STACK COMPARISON / CONFRONTO STACK TECNOLOGICO

| Concern | Current / Attuale | Target / Target | Phase / Fase |
|---------|------------------|----------------|-------------|
| Device protocol | Custom TCP binary + AES-128-CFB (IV=0) | MQTT/TLS 1.3 + random IV | 0, 5 |
| API runtime | PHP 8.4 + Laravel 12 | Node.js 22 + NestJS 11 | 3 |
| API database ORM | Eloquent | Prisma 6 | 3 |
| Real-time | HTTP polling | WebSocket (Socket.IO) | 2–3 |
| Message broker (internal) | AWS SQS | MQTT (internal) + SQS | 2 |
| IoT broker | None (custom TCP) / Nessuno | AWS IoT Core or Mosquitto | 4 |
| Database (main) | PostgreSQL (RDS) | PostgreSQL (RDS, same) | — |
| Database (telemetry) | PostgreSQL (relational) | **Amazon Timestream** | **4** |
| Analytics | None / Nessuno | **Kinesis + S3 + Athena + SageMaker** | **4–6** |
| Connection pooling | None / Nessuno | PgBouncer | 1 |
| Firmware transport | TCP/AES only | Transport abstraction (TCP/AES + MQTT/TLS) | 5 |
| Mobile framework | Flutter/Dart | Flutter/Dart (unchanged) | — |
| Web framework | React 19 + Vite | React 19 + Vite (unchanged) | — |
| Infrastructure | Terraform + Terragrunt | Terraform + Terragrunt (expanded) | 1, 4 |
| CI/CD | GitHub Actions | GitHub Actions (expanded) | Throughout |

---

<a name="appendix-c"></a>
# APPENDIX C: RISK REGISTER / REGISTRO DEI RISCHI

| ID | Risk / Rischio | Category | Likelihood | Impact | Mitigation / Mitigazione |
|----|---------------|----------|-----------|--------|-------------------------|
| R01 | AWS key `AKIASSAD2NHHUS6X34LK` already exploited / già sfruttata | Security | Medium | Critical | Rotate immediately + CloudTrail audit / Ruotare immediatamente + audit CloudTrail |
| R02 | Zero IV enables plaintext recovery / IV zero permette recupero plaintext | Security | Low (requires network MITM) | High | Phase 0 IV fix + Phase 5 MQTT/TLS |
| R03 | Actuator endpoint data leakage / Fuga dati endpoint Actuator | Security | Medium | High | Spring Security in Phase 0 / Spring Security nella Fase 0 |
| R04 | **NEW v2:** OTA rollout on 2,000 intermittent devices for IV fix / Rollout OTA su 2.000 device intermittenti per il fix IV | Operational | High (long tail certain) / Alta (lunga coda certa) | Medium | Dual-format protocol + 12-month legacy window / Protocollo dual-format + finestra legacy 12 mesi |
| R05 | **NEW v2:** AWS IoT Core cost overrun if telemetry interval < 30s / Sforamento costi AWS IoT Core se intervallo telemetria < 30s | Financial | Medium | High | Enforce minimum 30s interval in firmware + IoT Core billing alerts / Applicare intervallo minimo 30s nel firmware + alert fatturazione IoT Core |
| R06 | Java gateway stability during NestJS parallel operation / Stabilità gateway Java durante operazione parallela NestJS | Technical | Medium | Medium | Feature flags for traffic splitting / Flag funzionalità per divisione traffico |
| R07 | Data loss during API migration / Perdita dati durante migrazione API | Technical | Low | Critical | Parallel operation: Laravel writes, NestJS reads. No cutover until 100% tested. / Operazione parallela: Laravel scrive, NestJS legge. Nessun cutover fino al 100% testato. |
| R08 | Flutter BLE backward compatibility / Compatibilità BLE Flutter retroattiva | Technical | Low | Medium | BLE path is independent — no changes in Phases 0–5 / Il percorso BLE è indipendente — nessuna modifica nelle Fasi 0–5 |
| R09 | SageMaker model accuracy on small fleet / Accuratezza modello SageMaker su flotta ridotta | ML | High (at 2,000 devices) / Alta | Low (Phase 6, not critical path) | Start with rule-based (Tier 1) and time-series (Tier 2). Defer ML until data sufficient. / Iniziare con rule-based (Livello 1) e time-series (Livello 2). Rinviare ML finché i dati non sono sufficienti. |
| R10 | STM32 OTA brick if bootloader not compatible / Brick STM32 OTA se bootloader non compatibile | Firmware | Low | Critical | MEV2_bootloader provides rollback. Test on canary group. / MEV2_bootloader fornisce rollback. Test su gruppo canary. |
| R11 | **NEW v2.1:** Pre-built ESP32 AT binary does not support AT+SYSMFG / cert loading at runtime | Firmware | Medium | Critical | Lab validation gate §5.0 before any MQTT development; if fails: evaluate broker without mutual TLS or AT+SYSFLASH alternative / Gate di validazione lab §5.0 prima di qualsiasi sviluppo MQTT; se fallisce: valutare broker senza TLS mutuale o alternativa AT+SYSFLASH |
| R12 | **NEW v2.1:** Fleet Provisioning OTA complexity for 2,000 field devices / Complessità Fleet Provisioning OTA per 2.000 device in campo | Operational | High | High | Canary group of 10 permanently-online devices first; rollback to TCP/AES if provisioning fails / Gruppo canary di 10 device permanentemente online prima; rollback a TCP/AES se il provisioning fallisce |
| R13 | **NEW v2.1:** `+MQTTSUBRECV` Fleet Provisioning payload (cert+key) > 1024-byte WiFi queue buffer | Firmware | High | High | Dedicated 4096-byte provisioning buffer + partial message reassembly (already tested in `esp32_test.c`) / Buffer di provisioning dedicato da 4096 byte + riassemblaggio messaggi parziali (già testato in `esp32_test.c`) |
| R14 | **NEW v2.1:** SNTP sync failure prevents TLS handshake on AWS IoT Core / Fallimento sync SNTP impedisce handshake TLS su AWS IoT Core | Firmware | Medium | High | Retry NTP with fallback servers; 30s timeout + dedicated FSM error state / Retry NTP con server di fallback; timeout 30s + stato di errore FSM dedicato |
| R15 | **NEW v2.1:** SNI missing before `AT+MQTTCONN` causes TLS handshake failure on AWS IoT Core | Firmware | Medium | High | Unit tests must cover full AT sequence with SNI; FSM enforces strict AT command ordering / Gli unit test devono coprire la sequenza AT completa con SNI; la FSM applica un ordinamento rigoroso dei comandi AT |
| R16 | **NEW v2.1:** AT+SYSMFG cert loading adds ~3–4s latency on each boot / Caricamento certificati AT+SYSMFG aggiunge ~3–4s di latenza ad ogni boot | Firmware | Low | Low | Acceptable for industrial use; cache "cert loaded" flag in ESP32 NVS if supported by pre-built binary / Accettabile per uso industriale; cachare il flag "cert loaded" nella NVS di ESP32 se supportato dal binario precompilato |

---

<a name="appendix-d"></a>
# APPENDIX D: COST ANALYSIS / ANALISI DEI COSTI

> **NEW IN V2 / NUOVO IN V2**

## AWS IoT Core Pricing Model / Modello di Prezzi AWS IoT Core

AWS IoT Core charges per message (billed in 5KB increments) and per connected device-minute.

AWS IoT Core addebita per messaggio (fatturato in incrementi da 5KB) e per device-minuto connesso.

| Pricing Component | Rate (estimated / stimato) |
|------------------|--------------------------|
| Connectivity / Connettività | $0.042 per million device-minutes |
| Messaging / Messaggistica | $1.00 per million messages (5KB each) |
| Rules Engine | $0.15 per million rules triggered |

---

## Estimated Monthly Costs / Costi Mensili Stimati

> All figures are **estimated / stimato** and should be validated against current AWS pricing at time of deployment.

### Scenario: 1,000 Devices, 60-Second Telemetry Interval / 1.000 Device, Intervallo Telemetria 60 Secondi

| Component | Calculation | Monthly Cost |
|-----------|-------------|-------------|
| Connectivity (1k devices, 24/7) | 1,000 × 43,200 min × $0.042/M | ~$1.81 |
| Messages (1k devices, 1 msg/60s, 24/7) | 1,000 × 43,200 × $1.00/M | ~$43.20 |
| Rules Engine (1 rule per msg) | 43.2M × $0.15/M | ~$6.48 |
| **Subtotal / Subtotale** | | **~$51/month** |

### Scenario: 10,000 Devices, 60-Second Interval / 10.000 Device, Intervallo 60 Secondi

| Component | Calculation | Monthly Cost |
|-----------|-------------|-------------|
| Connectivity | 10,000 × 43,200 × $0.042/M | ~$18.10 |
| Messages | 10,000 × 43,200 × $1.00/M | ~$432 |
| Rules Engine | 432M × $0.15/M | ~$64.80 |
| **Subtotal / Subtotale** | | **~$515/month** |

### Caution: Short Intervals Cost Exponentially More / Attenzione: Intervalli Brevi Costano Esponenzialmente Di Più

| Interval | 1k devices / device | 10k devices / device |
|----------|--------------------|--------------------|
| 60s | ~$51/month | ~$515/month |
| 30s | ~$95/month | ~$950/month |
| 10s | ~$270/month | ~$2,700/month |
| 5s (current heartbeat) | ~$535/month | ~$5,350/month |

**Recommendation / Raccomandazione:** Enforce minimum 60-second telemetry interval in firmware for MQTT mode. Current 5,000ms heartbeat is for TCP polling and does not need to be replicated 1:1 in MQTT. / Applicare intervallo di telemetria minimo di 60 secondi nel firmware per la modalità MQTT. L'heartbeat attuale di 5.000ms è per il polling TCP e non deve essere replicato 1:1 in MQTT.

---

## Analytics Layer Additional Costs / Costi Aggiuntivi Layer Analytics

| Service | Estimated Cost | Notes / Note |
|---------|---------------|-------------|
| Kinesis Data Streams (2 shards) | ~$25/month | Scales with throughput / Scala con throughput |
| Amazon Timestream (1k devices, 60s) | ~$30/month | 30-day retention / Ritenzione 30 giorni |
| S3 (Parquet archive, 1 year) | ~$5/month | Grows with time / Cresce nel tempo |
| Athena (ad-hoc queries) | ~$5/month | $5 per TB scanned / $5 per TB scansionato |
| SageMaker (inference endpoint) | ~$35/month | ml.t3.medium / |
| **Analytics Subtotal** | **~$100/month** | **+65% of IoT Core base cost** |

---

## Break-Even Analysis / Analisi Break-Even

**AWS IoT Core vs. Self-Managed Mosquitto on ECS:**

```
Self-managed Mosquitto (ECS + NLB + ops overhead):
  Fixed cost / Costo fisso: ~$150-200/month regardless of device count
                             indipendentemente dal numero di device

AWS IoT Core crossover point / Punto di crossover:
  At 60s interval: IoT Core cheaper below ~3,000 devices
                   IoT Core più economico sotto ~3.000 device
  At 30s interval: IoT Core cheaper below ~1,500 devices

Recommendation / Raccomandazione:
  < 3,000 devices: AWS IoT Core (fully managed, no ops)
  > 5,000 devices: Evaluate self-managed Mosquitto on ECS
  2,000 devices (current): AWS IoT Core recommended
```

---

## Current Infrastructure Cost Baseline / Baseline Costi Infrastruttura Attuale

For comparison, the current DRUCS infrastructure in AWS (estimated / stimato):

Per confronto, l'infrastruttura DRUCS attuale in AWS (stimato):

| Service | Estimated Monthly | Notes |
|---------|------------------|-------|
| ECS (gateway + API + workers) | ~$200-300 | Depends on task sizing / Dipende dal dimensionamento task |
| RDS PostgreSQL (db.t3.medium) | ~$60-80 | Multi-AZ / |
| ElastiCache Redis | ~$30-50 | cache.t3.micro |
| S3 + CloudFront | ~$20-40 | Storage + CDN |
| SQS + SNS | ~$5-15 | Per usage |
| ALB | ~$20-30 | Per LCU |
| **Current total / Totale attuale** | **~$335-515/month** | |

Adding AWS IoT Core at 2,000 devices (~$51/month) and analytics (~$100/month) represents a **~30% cost increase** over the current baseline, with significant capability gains.

Aggiungere AWS IoT Core a 2.000 device (~$51/mese) e analytics (~$100/mese) rappresenta un **aumento dei costi del ~30%** rispetto alla baseline attuale, con significativi guadagni in termini di capacità.

---

*End of Document / Fine del Documento*

---

**Document Control / Controllo Documento**

| Field / Campo | Value / Valore |
|-------|-------|
| Author / Autore | Walter Divisi |
| Version / Versione | 2.0 |
| Date / Data | February 2026 / Febbraio 2026 |
| Status / Stato | Draft for Review / Bozza per Revisione |
| Classification / Classificazione | Internal — Confidential / Interno — Riservato |
| Supersedes / Sostituisce | DRUCS-Unified-Modernization-Roadmap-DUAL.docx (v1) |

**Change Log / Registro Modifiche**

| Version / Versione | Date / Data | Changes / Modifiche |
|---------|------|---------|
| 1.0 | 2025 | Initial version / Versione iniziale |
| 2.0 | Feb 2026 | Added security audit findings with file paths · Predictive maintenance architecture · OTA deployment strategy · Cost analysis · Updated effort estimates · Operational device count data / Aggiunti risultati audit sicurezza con path file · Architettura manutenzione predittiva · Strategia deployment OTA · Analisi costi · Stime effort aggiornate · Dati conteggio device operativi |
| 2.1 | Mar 2026 | Phase 5 revised: MQTT firmware zero-base confirmed · Sub-tasks 5.0–5.8 added · Effort 12–15 → 21.5–29 days · Timeline Week 12–24 → Week 12–30 · Total ~24 → ~30 weeks · 6 new risks R11–R16 in Appendix C · Gantt extended / Fase 5 rivista: MQTT firmware zero-base confermato · Sotto-attività 5.0–5.8 aggiunte · Effort 12–15 → 21.5–29 giorni · Settimane 12–24 → 12–30 · Totale ~24 → ~30 settimane · 6 nuovi rischi R11–R16 in Appendice C · Gantt esteso |
