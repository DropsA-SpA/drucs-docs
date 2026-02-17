# SilentTrack

<span class="badge badge-st">ST</span> **Railway rail lubrication system / Sistema lubrificazione rotaie ferroviarie**

---

## Overview / Panoramica

SilentTrack is a railway-specific lubrication system that applies lubricant to rail tracks. It operates across 4 independent zones, adapts to weather conditions, and counts passing trains via sensors.

*SilentTrack e' un sistema di lubrificazione specifico per ferrovie che applica lubrificante sulle rotaie. Opera su 4 zone indipendenti, si adatta alle condizioni meteorologiche e conta i treni in transito tramite sensori.*

---

## Key Specifications / Specifiche Principali

| Spec / *Specifica* | Value / *Valore* |
|---|---|
| Product code / *Codice prodotto* | ST |
| BLE prefix | `ST_` |
| Zones / *Zone* | 4 independent / *4 indipendenti* |
| Weather mode / *Modalita' meteo* | Manual or Automatic / *Manuale o Automatica* |
| GPS | Yes — for weather data / *Si — per dati meteo* |
| Power / *Alimentazione* | Mains (with emergency button) / *Rete (con pulsante emergenza)* |

---

## Dashboard Variables / Variabili Dashboard

| Metric / *Metrica* | Variable IDs | What it shows / *Cosa mostra* |
|---|---|---|
| **Total Rail Cycles** | `100110-100113` (sum) | *Total activations across all 4 zones / Attivazioni totali su tutte e 4 le zone* |
| **Daily Rail Cycles** | `100120-100123` (sum) | *Today's activations per zone / Attivazioni odierne per zona* |
| **Zone Status** | `100230-100233` | *Stopped / Pause / Suspend / Lubrication / Queue* |
| **Solenoid Valves** | `100220-100223` | *On/Off per zone / On/Off per zona* |
| **Train Count** | `100240-100243` | *Trains detected per zone / Treni rilevati per zona* |
| **Rail Temperature** | `100067` | *Temperature in C / Temperatura in C* |
| **System Status** | `100200` | *Running / Standby (time/voltage/weather/temp)* |

---

## Zone Configuration / Configurazione Zone

Each of the 4 zones can be configured independently:

*Ognuna delle 4 zone puo' essere configurata in modo indipendente:*

| Setting / *Impostazione* | IDs | Description / *Descrizione* |
|---|---|---|
| Zone cycles | 30-31 | *Number of pump cycles per activation / Numero cicli pompa per attivazione* |
| Zone trains | 32-33 | *Trains between activations / Treni tra le attivazioni* |
| Sensor link | 36-37 | *Link zone to specific sensor / Collega zona a sensore specifico* |

---

## Weather-Adaptive Mode / Modalita' Adattiva al Meteo

When weather mode is set to **Automatic** (setting #64), the device uses GPS coordinates (settings #66-67) to fetch weather data and adjust lubrication:

*Quando la modalita' meteo e' impostata su **Automatica** (impostazione #64), il dispositivo usa le coordinate GPS (impostazioni #66-67) per ottenere dati meteo e regolare la lubrificazione:*

- **Low temperature** → different lubrication formula / formula di lubrificazione diversa
- **Rain/snow** → modified solenoid valve timing / temporizzazione elettrovalvola modificata
- Temperature thresholds configurable (settings #40-43) / *Soglie temperatura configurabili*
---

## Common Alarms / Allarmi Comuni

| Alarm / *Allarme* | ID | Cause / *Causa* |
|---|---|---|
| Emergency button | 72 | Physical emergency button pressed / *Pulsante emergenza fisico premuto* |
| Low level | 60 | Lubricant reservoir low / *Serbatoio lubrificante basso* |
| Cycle timeout | 61 | Lubrication cycle exceeded max time / *Ciclo lubrificazione ha superato tempo max* |

Full reference in the [Variable Map](../platform/variable-map.md).
