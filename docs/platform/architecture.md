# Architecture / Architettura

## System Diagram / Diagramma di Sistema

```
                          AWS Cloud
  +---------------------------------------------------------+
  |                                                         |
  |   +-------------+    +-----------+    +-------------+   |
  |   | Laravel API  |<-->| PostgreSQL|    | S3 + CDN    |   |
  |   | (ECS)       |    | (RDS)     |    | (Frontend)  |   |
  |   +------^------+    +-----------+    +------^------+   |
  |          |                                   |          |
  +---------------------------------------------------------+
             |                                   |
             |  REST API                   Static assets
             |                                   |
  +----------v-----------+          +------------v----------+
  | DRUCS Gateway (Java) |          | Web v2 (React + Vite) |
  | s2.dropsa.com:1256   |          | app.dropsa.app/v2/    |
  +-----------^----------+          +-----------------------+
              |
    TCP + AES-128                   +----------------------+
              |                     | Mobile (Flutter)     |
  +-----------v----------+          | iOS + Android        |
  | STM32 Device         |          +----------------------+
  | (WiFi via ESP32)     |
  | (BLE direct)         |
  +-----------------------+
```

---

## Repositories / Repository

| Repo | What / *Cosa* | Tech |
|---|---|---|
| **S-00365-firmware** | STM32 device firmware | C, FreeRTOS, LVGL |
| **import-web-app** | Laravel backend API | PHP 8, Laravel 11 |
| **drucs-v2** | New React web dashboard | React 19, Vite, Tailwind |
| **mobile** | iOS + Android app | Flutter, Dart |
| **drucs-gateway-new** | Device gateway | Java, Spring Boot |
| **drucs-web-services** | Auth microservice (2FA) | PHP, Laravel |
| **drucs-iac** | AWS infrastructure | Terraform |

---

## Data Flow / Flusso Dati

### Telemetry (Device --> Cloud) / Telemetria

1. Device reads sensors every 5 seconds / *Il dispositivo legge i sensori ogni 5 secondi*2. Firmware packs variable IDs + values into a message / *Il firmware impacchetta ID variabili + valori in un messaggio*3. Message encrypted with AES-128-CFB (per-device key) / *Messaggio crittografato con AES-128-CFB (chiave per dispositivo)*4. HTTP POST to gateway / *HTTP POST al gateway*5. Gateway decrypts, parses, stores in database / *Il gateway decrittografa, analizza, memorizza nel database*6. API serves data to web and mobile apps / *L'API serve i dati alle app web e mobile*
### Commands (Cloud --> Device) / Comandi

1. User taps "Start Lubrication" in app / *L'utente tocca "Avvia Lubrificazione" nell'app*2. API queues command for device / *L'API accoda il comando per il dispositivo*3. Next heartbeat: gateway sends command in response / *Prossimo heartbeat: il gateway invia il comando nella risposta*4. Device executes command, reports back / *Il dispositivo esegue il comando, riporta lo stato*
---

## Security Model / Modello di Sicurezza

| Layer / *Livello* | Method / *Metodo* | Notes / *Note* |
|---|---|---|
| Device <-> Gateway | AES-128-CFB | Per-device key, rotatable via CRYPTO command |
| Gateway <-> API | Internal AWS network | ECS service mesh |
| API <-> Web/Mobile | HTTPS + JWT | Standard auth tokens |
| BLE (local) | PIN code | 4-6 digit device PIN |

!!! warning "Known Limitation / Limitazione Nota"
    The AES-128 encryption uses a zero IV (initialization vector). This is a known weakness being addressed in the MQTT modernization roadmap.

    *La crittografia AES-128 utilizza un IV (vettore di inizializzazione) a zero. Questa e' una debolezza nota che sara' risolta nella roadmap di modernizzazione MQTT.*
