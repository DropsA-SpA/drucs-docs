# DRUCS Variable Map / Mappa Variabili DRUCS
### The Rosetta Stone / La Stele di Rosetta

**What is this?** Every Dropsa IoT device sends data using numeric IDs. This document maps each ID to what it means in the firmware, the API, and the user-facing display.

*Ogni dispositivo IoT Dropsa invia dati usando ID numerici. Questo documento mappa ogni ID al suo significato nel firmware, nell'API e nel display utente.*

---

## Quick Reference / Riferimento Rapido

| ID Range / *Intervallo* | Type / *Tipo* | Example / *Esempio* |
|---|---|---|
| **1 — 99,999** | Settings (user-configurable) / *Impostazioni* | `5` = Cycle time / *Tempo ciclo* |
| **100,000 — 199,999** | Live data (read-only) / *Dati live* | `100100` = Motor hours / *Ore motore* |
| **200,000 — 300,000** | Commands (buttons) / *Comandi* | `200001` = Start lubrication / *Avvia lubrificazione* |

---

## Products / Prodotti

Each product compiles the same firmware with a different `#define`. Not all variables exist on every product.

*Ogni prodotto compila lo stesso firmware con un `#define` diverso. Non tutte le variabili esistono su ogni prodotto.*

| Code | Product | BLE Prefix | Key Feature |
|---|---|---|---|
| **BC** | BravoCompact 4.0 | `BC_` | Single-point, autonomous |
| **BR** | Bravo 4.0 | `BR_` | Multi-point, autonomous/CAN/IO-Link |
| **SM** | Smart 4.0 | `SM_` | Compact, autonomous/IO-Link |
| **OM** | OmegaPump | `OM_` | Motor hours tracking |
| **ST** | SilentTrack | `ST_` | Rail lubrication, 4 zones, weather |
| **NV** | VipAir 4.0 | `NV_` | Air-oil, 8 micropumps |
| **MA** | Maxtreme | `MA_` | High-pressure progressive |
| **VP** | VIP6 | `VP_` | Legacy VIP platform |

---

## LIVE DATA — Variables (100,000+) / DATI LIVE — Variabili

### Core — All Products / Comuni a tutti i prodotti

| ID | What it shows (EN) | Cosa mostra (IT) | Unit | Dashboard Use |
|---|---|---|---|---|
| **100010** | Device voltage | *Tensione dispositivo* | V | Health monitoring |
| **100011** | Motor current | *Corrente motore* | A | Health monitoring |
| **100012** | MCU temperature | *Temperatura MCU* | C | Health monitoring |
| **100013** | Battery voltage | *Tensione batteria* | V | Health monitoring |
| **100014** | FW upload progress | *Avanzamento FW* | % | OTA update screen |
| **100015** | Active warnings | *Avvisi attivi* | — | Warning badge (semicolon-separated event IDs) |
| **100016** | Active alarms | *Allarmi attivi* | — | Alarm badge (semicolon-separated event IDs) |
| **100050** | Pump status | *Stato pompa* | — | Status badge: Stop/Pause/Work/Pause between cycles/Suspension |
| **100100** | **Total motor hours** | **Ore motore totali** | sec | **Hours box** on dashboard (sec / 3600 = hours) |
| **100101** | **Total pump rotations** | **Rotazioni pompa totali** | rot | **"Pump Rotations"** counter on dashboard |
| **100052** | Lub. pulse counter | *Contatore impulsi lub.* | pulses | Fallback counter ("Lub. Pulses") |
| **100054** | Cycle counter | *Contatore cicli* | cycles | Fallback counter ("Cycles") |

### Lubrication Timers / Timer Lubrificazione

| ID | What it shows (EN) | Cosa mostra (IT) | Unit | Notes |
|---|---|---|---|---|
| 100051 | Pre-lubrication time | *Tempo pre-lubrificazione* | sec | Elapsed since start |
| 100053 | Cycle time (running) | *Tempo ciclo (in corso)* | sec | Linked to setting #5 |
| 100055 | Interval time (running) | *Tempo intervallo (in corso)* | sec | Linked to setting #7 |
| 100056 | PS delay timer | *Timer ritardo pressostato* | sec | |
| 100057 | PS delay input 1 | *Ritardo PS ingresso 1* | sec | |
| 100058 | PS delay input 2 | *Ritardo PS ingresso 2* | sec | |
| 100059 | Suspension timer | *Timer sospensione* | sec | |
| 100060 | Cycle timeout timer | *Timer timeout ciclo* | sec | |
| 100061 | Cycle pulses detected | *Impulsi ciclo rilevati* | pulses | |
| 100062 | Interval pulses detected | *Impulsi intervallo rilevati* | pulses | |

### Level & Pressure Sensor / Sensore Livello e Pressione

| ID | What it shows (EN) | Cosa mostra (IT) | Unit | Products |
|---|---|---|---|---|
| **100017** | **Reservoir level** | **Livello serbatoio** | % | BC, BR, SM (with LP sensor) |
| **100018** | Pressure | *Pressione* | bar | BC, BR, SM (with LP sensor) |
| 100260 | Current pressure | *Pressione attuale* | bar | VipAir, Maxtreme |
| 100261 | Setpoint pressure | *Pressione di riferimento* | bar | VipAir, Maxtreme |

### SilentTrack — Zone Data / Dati Zone

| ID | What it shows (EN) | Cosa mostra (IT) | Unit | Dashboard Use |
|---|---|---|---|---|
| **100110** | Zone 1 total activations | *Attivazioni totali zona 1* | count | **RailCycles** (sum 100110-100113) |
| **100111** | Zone 2 total activations | *Attivazioni totali zona 2* | count | |
| **100112** | Zone 3 total activations | *Attivazioni totali zona 3* | count | |
| **100113** | Zone 4 total activations | *Attivazioni totali zona 4* | count | |
| **100120** | Zone 1 daily activations | *Attivazioni giornaliere zona 1* | count | **Daily RailCycles** (sum 100120-100123) |
| **100121** | Zone 2 daily activations | *Attivazioni giornaliere zona 2* | count | |
| **100122** | Zone 3 daily activations | *Attivazioni giornaliere zona 3* | count | |
| **100123** | Zone 4 daily activations | *Attivazioni giornaliere zona 4* | count | |
| 100230-100233 | Zone 1-4 status | *Stato zona 1-4* | — | Stopped/Pause/Suspend/Lubrication/Queue |
| 100220-100223 | Zone 1-4 solenoid valve | *EV zona 1-4* | — | On/Off |
| 100240-100243 | Zone 1-4 train count | *Conteggio treni zona 1-4* | trains | |
| 100067 | Rail temperature | *Temperatura rotaia* | C | |

### VipAir — MicroPump Data / Dati Micropompe

| ID | What it shows (EN) | Cosa mostra (IT) | Unit | Dashboard Use |
|---|---|---|---|---|
| **100320-100327** | **Total cycles MP1-MP8** | **Cicli totali MP1-MP8** | cycles | **MicroPump Cycles** (sum all 8) |
| 100300-100307 | Status MP1-MP8 | *Stato MP1-MP8* | — | Alarm/Disabled/Stopped/Filling/Pre-lub/Lub |
| 100310-100317 | Air pressure MP1-MP8 | *Pressione aria MP1-MP8* | bar | |
| 100263 | Air solenoid valve | *EV aria* | — | Off/On |
| 100264 | Oil solenoid valve | *EV olio* | — | Off/On |

### Plant Status / Stato Impianto (SilentTrack + VipAir)

| ID | What it shows (EN) | Cosa mostra (IT) | Values |
|---|---|---|---|
| 100200 | System status | *Stato impianto* | Stopped/Running/Standby(Time/Voltage/Weather/HighTemp/LowTemp/NoData) |
| 100201 | Mains voltage | *Tensione di rete* | Absent/Present |
| 100202 | Emergency button | *Pulsante emergenza* | Pressed/Released |

---

## SETTINGS — Parameters (1-99,999) / IMPOSTAZIONI — Parametri

### Lubrication Setup / Configurazione Lubrificazione

| ID | Setting (EN) | *Impostazione (IT)* | Type | Options / Range |
|---|---|---|---|---|
| **1** | Lubrication mode | *Modalita' lubrificazione* | Mode | PS/SEP, Pulse counting, Rotation, Time, External |
| **2** | Interval mode | *Modalita' intervallo* | Mode | Time, Pulse counting, Time & Pulses, Pulses & TOut |
| **3** | Input 1 mode | *Ingresso 1* | Mode | Disconnect, PS, SEP/PROX, Pulse, Suspend, External |
| **4** | Input 2 mode | *Ingresso 2* | Mode | (same as above) |
| **5** | Cycle time | *Tempo ciclo* | Seconds | Dynamic range — **linked to live var 100053** |
| **6** | Cycle pulses | *Impulsi ciclo* | Pulses | |
| **7** | Interval time | *Tempo intervallo* | Seconds | Dynamic range — **linked to live var 100055** |
| **8** | Interval pulses | *Impulsi intervallo* | Pulses | |
| **9** | Number of cycles | *Numero di cicli* | Count | |
| **10** | Cycle timeout | *Timeout ciclo* | Seconds | Max duration before alarm |
| **11** | Pause between cycles | *Pausa tra cicli* | Seconds | |
| **12** | Suspension time | *Tempo sospensione tra impulsi* | Seconds | Max delay between pulses |
| **13** | PS delay | *Ritardo pressostato* | Seconds | |
| **14** | Interval timeout | *Timeout intervallo* | Seconds | |
| **15** | Suspension mode | *Modalita' sospensione* | Mode | Paused / During lubrication / Always |
| **16** | Suspension contact | *Contatto sospensione* | Mode | Normally Open / Normally Closed |
| **17** | Start from | *Avvio da* | Mode | Lubrication / Non-Lub. Cycle / Last Status |
| **18** | Pre-lubrication cycles | *Cicli pre-lubrificazione* | Count | |
| **19** | Disable min level alarm | *Disab. allarme min. livello* | On/Off | |
| **20** | Lub. status output role | *Uscita stato lubrificazione* | Mode | Only lub. / Lub.(steady) & Pause(blink) |
| **21** | Lub. status output type | *Tipo uscita lub.* | Mode | Type N / Type P |
| **22** | Alarm output role | *Uscita stato allarme* | Mode | Alarm only / Coded / Alarm & Warning / Alarm(steady) & Warn(blink) |
| **23** | Alarm output type | *Tipo uscita allarme* | Mode | Type N / Type P |
| **24** | Reservoir type | *Tipo serbatoio* | Mode | Unconfigured / Follower plate / Cart. 400cc / Cart. 700cc |
| **25** | Check rotation sensor | *Controllo sensore rotazione* | On/Off | |

### Wi-Fi / Wi-Fi

| ID | Setting (EN) | Impostazione (IT) | Type |
|---|---|---|---|
| **40000** | Enable Wi-Fi | *Abilita Wi-Fi* | On/Off |
| **40010** | SSID | *Nome rete* | Text (32 chars) |
| **40020** | Password | *Password Wi-Fi* | Password |
| **40030** | DHCP | *Abilita DHCP* | On/Off |
| **40040** | IP address | *Indirizzo IP* | IP |
| **40050** | Gateway | *IP gateway* | IP |
| **40060** | Subnet mask | *Maschera di sottorete* | IP |
| **40090** | Connection check | *Verifica connessione* | Mode: Off/10m/30m/1h/2h/3h/6h/12h/24h |

### Bluetooth / Bluetooth

| ID | Setting (EN) | Impostazione (IT) | Type |
|---|---|---|---|
| **30000** | Enable Bluetooth | *Abilita Bluetooth* | On/Off |
| **30010** | BLE PIN | *PIN Bluetooth* | 4-6 digits |

### System / Sistema

| ID | Setting (EN) | Impostazione (IT) | Type |
|---|---|---|---|
| **20000** | Store all logs | *Memorizza tutti i log* | On/Off |
| **20100** | Screen lock timeout | *Timeout blocco schermo* | Mode: 30s/1m/10m/30m/60m |
| **20130** | Time zone | *Fuso orario* | Mode (49 options) |

### Lubricant / Lubrificante

| ID | Setting (EN) | Impostazione (IT) | Type |
|---|---|---|---|
| **21000** | Lube type | *Tipo lubrificante* | Mode: Not set/Custom/G000-G004 |
| **21010** | Package format | *Formato confezione* | Mode: Not set/Cart.400cc/Cart.700cc/5KG/10KG/50KG |
| **21020** | Request reorder | *Richiedi ordine* | On/Off |
| **21030** | Notify refill | *Notifica riempimento* | On/Off |
| **21040** | Custom lube name | *Nome personalizzato* | Text (32 chars) |

### Display Widgets / Widget Display

| ID | Setting (EN) | Impostazione (IT) | Type |
|---|---|---|---|
| 60000-60040 | Widget 1-5 visible | *Widget 1-5 visibile* | On/Off |
| 60050 | Auto-scroll | *Auto-scorrimento* | On/Off |
| 60060 | Auto-scroll time | *Tempo auto-scorrimento* | Mode: 5s/10s/30s/1m/5m |

### VipAir-Specific / Specifici VipAir

| ID | Setting (EN) | Impostazione (IT) | Products |
|---|---|---|---|
| 69-76 | Interval time MP1-MP8 | *Tempo intervallo MP1-MP8* | VipAir |
| 77-84 | Interval pulses MP1-MP8 | *Impulsi intervallo MP1-MP8* | VipAir |
| 85 | Cycle sensor timeout | *Timeout sensore ciclo* | VipAir |
| 86 | Pump refill time | *Tempo ricarica pompa* | VipAir |
| 90 | Air SV mode | *Modalita' EV aria* | VipAir: Continuous/Normal/Spray/Off |
| 96-103 | Min air pressure MP1-MP8 | *Pressione min. aria MP1-MP8* | VipAir |
| 104-111 | Max air pressure MP1-MP8 | *Pressione max. aria MP1-MP8* | VipAir |
| 116 | VipAir version | *Versione VipAir* | VipAir: With Tank / Without Tank |

### SilentTrack-Specific / Specifici SilentTrack

| ID | Setting (EN) | Impostazione (IT) | Products |
|---|---|---|---|
| 30-31 | Zone 1-2 cycles | *Cicli zona 1-2* | SilentTrack |
| 32-33 | Zone 1-2 trains | *Treni zona 1-2* | SilentTrack |
| 36-37 | Zone 1-2 sensor link | *Zona 1-2 sensore* | SilentTrack |
| 40 | Min temperature | *Temperatura minima* | SilentTrack |
| 41-43 | Max temperature T1-T3 | *Temperatura max T1-T3* | SilentTrack |
| 44-47 | Working pressure / *constants T1-T3* | *Pressione lavoro / costanti T1-T3* | SilentTrack |
| 48-50 | SV opening time T1-T3 | *Tempo apertura EV T1-T3* | SilentTrack |
| 64 | Weather mode | *Modalita' meteo* | SilentTrack: Manual/Automatic |
| 66-67 | GPS coordinates | *Coordinate GPS* | SilentTrack |

---

## COMMANDS — Actions (200,000+) / COMANDI — Azioni

| ID | Button Label (EN) | Etichetta Pulsante (IT) | Products |
|---|---|---|---|
| **200001** | **Start lubrication** | **Avvia lubrificazione** | All |
| **200002** | **Stop lubrication** | **Arresta lubrificazione** | All |
| **200003** | **Reset alarms** | **Resetta allarmi** | All |
| **200004** | **Extra cycle** | **Ciclo extra** | All |
| 200005 | System start | *Avvio sistema* | SilentTrack, VipAir |
| 200006 | System stop | *Arresto sistema* | SilentTrack, VipAir |
| 200007 | System start/stop toggle | *Avvio/arresto sistema* | SilentTrack, VipAir |
| **200010** | **Update firmware** | **Aggiorna firmware** | All |
| **200011** | **Restart device** | **Riavvia dispositivo** | All |
| 200020 | Start zone 1 | *Avvia zona 1* | SilentTrack |
| 200021 | Start zone 2 | *Avvia zona 2* | SilentTrack |

---

## ALARMS & WARNINGS / ALLARMI E AVVISI

These IDs appear in event logs and in the active alarm/warning bitfields (variables 100015, 100016).

*Questi ID compaiono nei log eventi e nei campi allarmi/avvisi attivi (variabili 100015, 100016).*

### Alarms (Red) / Allarmi (Rosso)

| ID | Alarm (EN) | Allarme (IT) | Products |
|---|---|---|---|
| **60** | **Low level** | **Livello basso** | All with level sensor |
| **61** | **Cycle timeout** | **Timeout ciclo** | All |
| **62** | **Motor overcurrent** | **Sovracorrente motore** | All |
| 65 | Low voltage | *Tensione bassa* | All |
| 66 | High voltage | *Tensione alta* | All |
| 67 | Rotation sensor fault | *Sensore rotazione guasto* | All (with sensor) |
| 68 | PS already active | *Pressostato gia' attivo* | All |
| 69 | Pause timeout | *Timeout pausa* | All |
| 70 | Interval timeout | *Timeout intervallo* | All |
| 72 | Emergency button | *Pulsante emergenza* | SilentTrack, VipAir |
| 74 | Pressure timeout | *Timeout pressione* | VipAir, Maxtreme |
| 75 | Refill timeout | *Timeout ricarica* | VipAir |
| 80-87 | Low pressure MP1-8 (EV on) | *Bassa pressione MP1-8* | VipAir |
| 90-97 | High pressure MP1-8 (EV on) | *Alta pressione MP1-8* | VipAir |
| 100-107 | Low residual pressure MP1-8 | *Bassa pressione residua MP1-8* | VipAir |
| 110-117 | Cycle sensor alert MP1-8 | *Allarme sensore ciclo MP1-8* | VipAir |
| 200-204 | LP sensor faults | *Guasti sensore LP* | All (with LP sensor) |
| 210-211 | Command timeouts | *Timeout comandi* | All |

### Warnings (Amber) / Avvisi (Ambra)

| ID | Warning (EN) | Avviso (IT) | Products |
|---|---|---|---|
| **63** | **Low voltage** | **Tensione bassa** | All |
| **64** | **High voltage** | **Tensione alta** | All |
| **71** | **Low level** | **Livello basso** | All with level sensor |
| 73 | Supply voltage failure | *Guasto alimentazione* | All |
| 120-127 | Incomplete cycle MP1-8 | *Ciclo incompleto MP1-8* | VipAir |
| 205-207 | LP sensor warnings | *Avvisi sensore LP* | All (with LP sensor) |

---

## How the Dashboard Uses This / Come la Dashboard Usa Questi Dati

The drucs-v2 dashboard picks variables in this priority order for each metric box:

*La dashboard drucs-v2 seleziona le variabili in questo ordine di priorita' per ogni indicatore:*

### Hours Box / Indicatore Ore
`100100` (total motor seconds / 3600) → if missing, shows "No data"

### Cycles Box / Indicatore Cicli
1. `100110-100113` sum → **RailCycles** (SilentTrack: 4 zones summed)
2. `100320-100327` sum → **MicroPump Cycles** (VipAir: 8 pumps summed)
3. `100101` → **Pump Rotations** (single counter)
4. `100052` → **Lub. Pulses** (fallback)
5. `100054` → **Cycles** (fallback)

### Level Box / Indicatore Livello
`100017` → Level % gauge (0-100%)

### Will Go Empty / Stima Esaurimento
Calculated from alarm event #60 (low level) history — average days between refills, days since last refill.

---

*Document version 2.0 — 2026-02-17*
*Versione documento 2.0 — 17-02-2026*
