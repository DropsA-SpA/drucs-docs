# Platform Overview / Panoramica della Piattaforma

## What is DRUCS? / Cos'e' DRUCS?

DRUCS (Dropsa Remote Universal Controller System) is a full-stack IoT platform for DropsA industrial lubrication systems.

*DRUCS (Dropsa Remote Universal Controller System) e' una piattaforma IoT full-stack per i sistemi di lubrificazione industriale DropsA.*

```
STM32 Firmware (S-00365) --> BLE/WiFi --> DRUCS Gateway --> Laravel API --> Web / Mobile App
```

---

## System Architecture / Architettura del Sistema

| Layer / *Livello* | Technology / *Tecnologia* | Repository | Status / *Stato* |
|---|---|---|---|
| Firmware | STM32L496, FreeRTOS, LVGL | S-00365-firmware | Active (v0.59) / *Attivo* |
| Gateway | Java (Spring Boot) | drucs-gateway-new | Stale — needs decision / *In stallo* |
| Backend API | PHP (Laravel) | import-web-app | Active / *Attivo* |
| Frontend v1 | Angular 17 | frontend-web | Retired — replaced by v2 / *Sostituito da v2* |
| **Frontend v2** | **React 19 + Vite + Tailwind** | **drucs-v2** | **Active / Attivo** |
| Mobile App | Flutter/Dart | mobile | Active / *Attivo* |
| Infrastructure | Terraform (AWS ECS/ECR/S3) | drucs-iac | Under review / *In revisione* |

---

## How Devices Communicate / Come Comunicano i Dispositivi

### Device --> Cloud / Dispositivo --> Cloud

1. STM32 connects to WiFi via ESP32 co-processor / *STM32 si connette al WiFi tramite co-processore ESP32*2. Opens TCP connection to `s2.dropsa.com:1256` / *Apre connessione TCP a `s2.dropsa.com:1256`*3. Sends HTTP POST with AES-128 encrypted payload every 5 seconds / *Invia HTTP POST con payload crittografato AES-128 ogni 5 secondi*4. Server responds with any pending commands / *Il server risponde con eventuali comandi in sospeso*
### Cloud --> Device / Cloud --> Dispositivo

Commands are queued on the server and delivered in the next heartbeat response.

*I comandi vengono accodati sul server e consegnati nella risposta heartbeat successiva.*

| Command / *Comando* | Purpose / *Scopo* |
|---|---|
| `ACTID` | Trigger remote action (start/stop lubrication) / *Attiva azione remota* |
| `CRYPTO` | Rotate AES encryption key / *Ruota chiave crittografia* |
| `FW` / `FWID` | Firmware update (OTA) / *Aggiornamento firmware (OTA)* |
| `SETV` | Change device setting / *Modifica impostazione dispositivo* |
| `GETV` | Read device variable / *Leggi variabile dispositivo* |

---

## Development Strategy / Strategia di Sviluppo

```
Web v2 (React) -----> new features built here first
       |               le nuove funzionalita' si sviluppano prima qui
       |
       |---> Mobile (Flutter) ----> features replicated from web
       |                            funzionalita' replicate dalla web
       |
       '---> Help Docs (MkDocs) --> embedded in web + app, standalone site
                                    integrati in web + app, sito autonomo
```

- **Web leads, mobile follows** — every new feature is designed for web first, then adapted to Flutter
- **Shared backend** — both web and mobile use the same Laravel API
- **Shared docs** — help pages served from this MkDocs site, embeddable in both web and mobile

*Web guida, mobile segue — ogni nuova funzionalita' e' progettata prima per il web, poi adattata per Flutter. Backend condiviso — sia web che mobile usano la stessa API Laravel. Documentazione condivisa — le pagine di aiuto servite da questo sito MkDocs, integrabili sia nel web che nel mobile.*

---

## URLs / Indirizzi

| Service / *Servizio* | URL | Status / *Stato* |
|---|---|---|
| Backend API | `api.dropsa.app` | Running on AWS ECS / *In esecuzione su AWS ECS* |
| Frontend v1 (Angular) | `app.dropsa.app` | Deployed, being retired / *Distribuito, in fase di ritiro* |
| **Frontend v2 (React)** | `app.dropsa.app/v2/` | **Active / Attivo** |
| Mobile App | iOS + Android stores | Published / *Pubblicata* |
| Firmware Simulator | `enghelper.dropsa.net:8765` | Running / *In esecuzione* |
| **This docs site** | `enghelper.dropsa.net/docs/` | **You are here / Sei qui** |
