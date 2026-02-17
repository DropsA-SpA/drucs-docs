# VipAir 4.0

<span class="badge badge-nv">NV</span> **Air-oil minimal quantity lubrication / Lubrificazione minimale aria-olio**

---

## Overview / Panoramica

VipAir 4.0 is an air-oil (MQL — Minimum Quantity Lubrication) system with 8 independent micropumps. Each micropump delivers precise quantities of lubricant mixed with compressed air to the cutting tool.

*VipAir 4.0 e' un sistema aria-olio (MQL — Lubrificazione a Quantita' Minima) con 8 micropompe indipendenti. Ogni micropompa fornisce quantita' precise di lubrificante miscelato con aria compressa all'utensile da taglio.*

---

## Key Specifications / Specifiche Principali

| Spec / *Specifica* | Value / *Valore* |
|---|---|
| Product code / *Codice prodotto* | NV |
| BLE prefix | `NV_` |
| Micropumps / *Micropompe* | 8 independent / *8 indipendenti* |
| Air pressure / *Pressione aria* | Monitored per pump / *Monitorata per pompa* |
| Version | With Tank / Without Tank / Con Serbatoio / *Senza Serbatoio* |

---

## Dashboard Variables / Variabili Dashboard

| Metric / *Metrica* | Variable IDs | What it shows / *Cosa mostra* |
|---|---|---|
| **MicroPump Cycles** | `100320-100327` (sum) | *Total cycles across all 8 pumps / Cicli totali su tutte e 8 le pompe* |
| **Pump Status** | `100300-100307` | *Per-pump: Alarm/Disabled/Stopped/Filling/Pre-lub/Lub* |
| **Air Pressure** | `100310-100317` | *Pressure per pump (bar) / Pressione per pompa* |
| **System Pressure** | `100260` | *Current system pressure / Pressione sistema attuale* |
| **Setpoint Pressure** | `100261` | *Target pressure / Pressione obiettivo* |
| **Air Solenoid** | `100263` | *Air valve Off/On / Elettrovalvola aria Off/On* |
| **Oil Solenoid** | `100264` | *Oil valve Off/On / Elettrovalvola olio Off/On* |

---

## MicroPump Settings / Impostazioni Micropompe

Each of the 8 micropumps (MP1-MP8) has individual settings:

*Ognuna delle 8 micropompe (MP1-MP8) ha impostazioni individuali:*

| Setting / *Impostazione* | IDs | Description / *Descrizione* |
|---|---|---|
| Interval time | 69-76 | *Time between pump activations / Tempo tra le attivazioni pompa* |
| Interval pulses | 77-84 | *Pulses between activations / Impulsi tra le attivazioni* |
| Min air pressure | 96-103 | *Minimum required pressure / Pressione minima richiesta* |
| Max air pressure | 104-111 | *Maximum allowed pressure / Pressione massima consentita* |

### System Settings / Impostazioni Sistema

| Setting / *Impostazione* | ID | Description / *Descrizione* |
|---|---|---|
| Cycle sensor timeout | 85 | *Max time to detect cycle / Tempo max per rilevare ciclo* |
| Pump refill time | 86 | *Refill duration / Durata ricarica* |
| Air SV mode | 90 | *Continuous / Normal / Spray / Off* |
| VipAir version | 116 | *With Tank / Without Tank* |

---

## Common Alarms / Allarmi Comuni

| Alarm / *Allarme* | IDs | Cause / *Causa* |
|---|---|---|
| Low pressure MP1-8 | 80-87 | Pump pressure below minimum / *Pressione pompa sotto il minimo* |
| High pressure MP1-8 | 90-97 | Pump pressure above maximum / *Pressione pompa sopra il massimo* |
| Low residual pressure MP1-8 | 100-107 | Residual pressure too low / *Pressione residua troppo bassa* |
| Cycle sensor alert MP1-8 | 110-117 | Cycle sensor not detecting / *Sensore ciclo non rileva* |
| Incomplete cycle MP1-8 | 120-127 | Pump cycle did not complete (warning) / *Ciclo pompa non completato (avviso)* |
| Pressure timeout | 74 | System pressure not reached in time / *Pressione sistema non raggiunta in tempo* |
| Refill timeout | 75 | Refill took too long / *Ricarica ha impiegato troppo tempo* |
| Emergency button | 72 | Physical emergency button pressed / *Pulsante emergenza premuto* |

Full reference in the [Variable Map](../platform/variable-map.md).
