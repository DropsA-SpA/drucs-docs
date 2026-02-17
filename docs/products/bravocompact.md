# BravoCompact 4.0

<span class="badge badge-bc">BC</span> **Single-point autonomous lubricator / Lubrificatore autonomo singolo punto**

---

## Overview / Panoramica

The BravoCompact 4.0 is a compact, self-contained lubrication unit for single-point applications. It operates autonomously on a timed cycle, pumping grease or oil to a single lubrication point.

*Il BravoCompact 4.0 e' un'unita' di lubrificazione compatta e autonoma per applicazioni singolo punto. Funziona autonomamente con un ciclo temporizzato, pompando grasso o olio a un singolo punto di lubrificazione.*

---

## Key Specifications / Specifiche Principali

| Spec / *Specifica* | Value / *Valore* |
|---|---|
| Product code / *Codice prodotto* | BC |
| BLE prefix | `BC_` |
| Reservoir / *Serbatoio* | 400cc or 700cc cartridge / *Cartuccia 400cc o 700cc* |
| Power / *Alimentazione* | 24V DC |
| Lubrication mode / *Modalita'* | Time-based, pulse-based, or external trigger / *Temporizzata, a impulsi, o trigger esterno* |
| Connectivity / *Connettivita'* | WiFi + BLE |
| Level sensor / *Sensore livello* | Optional LP sensor (0-100% analog) / *Sensore LP opzionale (analogico 0-100%)* |

---

## Dashboard Variables / Variabili Dashboard

These are the key variables shown on the DRUCS v2 dashboard for this product:

*Queste sono le variabili chiave mostrate nella dashboard DRUCS v2 per questo prodotto:*

| Metric / *Metrica* | Variable ID | What it shows / *Cosa mostra* |
|---|---|---|
| **Motor Hours / Ore Motore** | `100100` | *Total motor runtime in seconds (displayed as hours) / Tempo motore totale in secondi (mostrato in ore)* |
| **Pump Rotations / Rotazioni Pompa** | `100101` | *Total pump rotation count / Conteggio totale rotazioni pompa* |
| **Reservoir Level / Livello Serbatoio** | `100017` | *Current level percentage (0-100%) / Percentuale livello attuale* |
| **Pump Status / Stato Pompa** | `100050` | *Stop / Pause / Work / Suspension* |
| **Active Alarms / Allarmi Attivi** | `100016` | *Semicolon-separated alarm event IDs / ID eventi allarme separati da punto e virgola* |

---

## Common Alarms / Allarmi Comuni

| Alarm / *Allarme* | ID | Cause (EN) | Causa (IT) |
|---|---|---|---|
| Low level / *Livello basso* | 60 | Reservoir empty or near-empty | *Serbatoio vuoto o quasi vuoto* |
| Cycle timeout / *Timeout ciclo* | 61 | Lubrication cycle took too long | *Il ciclo di lubrificazione ha impiegato troppo tempo* |
| Motor overcurrent / *Sovracorrente motore* | 62 | Motor drawing excessive current | *Motore che assorbe corrente eccessiva* |
| Low voltage / *Tensione bassa* | 65 | Supply voltage below threshold | *Tensione di alimentazione sotto soglia* |

---

## Settings / Impostazioni

### Basic Setup / Configurazione Base

| Setting / *Impostazione* | ID | Default | Description (EN) | Descrizione (IT) |
|---|---|---|---|---|
| Lubrication mode | 1 | Time | Controls how lubrication cycles are triggered | *Controlla come vengono attivati i cicli di lubrificazione* |
| Cycle time | 5 | 10 sec | Duration of each lubrication cycle | *Durata di ogni ciclo di lubrificazione* |
| Interval time | 7 | 3600 sec | Time between lubrication cycles | *Tempo tra i cicli di lubrificazione* |
| Cycle timeout | 10 | 60 sec | Max cycle duration before alarm | *Durata max ciclo prima dell'allarme* |
| Reservoir type | 24 | — | Cartridge 400cc or 700cc | *Cartuccia 400cc o 700cc* |

### Connectivity / Connettivita'

| Setting / *Impostazione* | ID | Description / *Descrizione* |
|---|---|---|
| Enable WiFi | 40000 | *Turn WiFi on/off / Attiva/disattiva WiFi* |
| SSID | 40010 | *WiFi network name / Nome rete WiFi* |
| Password | 40020 | *WiFi password / Password WiFi* |
| Enable Bluetooth | 30000 | *Turn BLE on/off / Attiva/disattiva BLE* |
| BLE PIN | 30010 | *4-6 digit pairing PIN / PIN di accoppiamento 4-6 cifre* |

---

## Remote Commands / Comandi Remoti

| Command / *Comando* | ID | Description / *Descrizione* |
|---|---|---|
| Start lubrication / *Avvia lubrificazione* | 200001 | *Trigger an immediate lubrication cycle / Attiva un ciclo di lubrificazione immediato* |
| Stop lubrication / *Arresta lubrificazione* | 200002 | *Stop the current cycle / Arresta il ciclo corrente* |
| Reset alarms / *Resetta allarmi* | 200003 | *Clear all active alarms / Cancella tutti gli allarmi attivi* |
| Extra cycle / *Ciclo extra* | 200004 | *Run one additional cycle / Esegui un ciclo aggiuntivo* |
| Update firmware / *Aggiorna firmware* | 200010 | *Start OTA firmware update / Avvia aggiornamento firmware OTA* |
| Restart device / *Riavvia dispositivo* | 200011 | *Reboot the device / Riavvia il dispositivo* |

---

## Troubleshooting / Risoluzione Problemi

!!! tip "Low Level Alarm keeps triggering / L'allarme livello basso continua ad attivarsi"
    1. Check the reservoir is not empty / *Verificare che il serbatoio non sia vuoto*    2. If using LP sensor: check cable connection / *Se si usa sensore LP: verificare il collegamento del cavo*    3. If no LP sensor: alarm triggers from follower-plate switch / *Se senza sensore LP: l'allarme si attiva dall'interruttore a piatto seguente*    4. Disable low level alarm (setting #19) only for testing / *Disabilitare l'allarme livello basso (impostazione #19) solo per test*
!!! tip "Device not connecting to WiFi / Dispositivo non si connette al WiFi"
    1. Verify SSID and password are correct (settings #40010, #40020) / *Verificare che SSID e password siano corretti*    2. Check WiFi is enabled (setting #40000) / *Verificare che il WiFi sia abilitato*    3. Ensure the WiFi network is 2.4 GHz (5 GHz not supported) / *Assicurarsi che la rete WiFi sia 2.4 GHz (5 GHz non supportato)*    4. Check device voltage (variable #100010) — low voltage can cause WiFi failures / *Controllare tensione dispositivo — bassa tensione puo' causare problemi WiFi*
---

*See the full [Variable Map](../platform/variable-map.md) for all variable IDs.*

*Consultare la [Mappa Variabili](../platform/variable-map.md) completa per tutti gli ID variabili.*
