# Bravo 4.0

<span class="badge badge-br">BR</span> **Multi-point autonomous lubricator / Lubrificatore autonomo multi-punto**

---

## Overview / Panoramica

The Bravo 4.0 is a multi-point lubrication system with CAN bus and IO-Link connectivity. It can serve multiple lubrication points from a single unit and integrates with industrial automation systems.

*Il Bravo 4.0 e' un sistema di lubrificazione multi-punto con connettivita' CAN bus e IO-Link. Puo' servire piu' punti di lubrificazione da una singola unita' e si integra con i sistemi di automazione industriale.*

---

## Key Specifications / Specifiche Principali

| Spec / *Specifica* | Value / *Valore* |
|---|---|
| Product code / *Codice prodotto* | BR |
| BLE prefix | `BR_` |
| Reservoir / *Serbatoio* | Multiple options / *Opzioni multiple* |
| Power / *Alimentazione* | 24V DC |
| Connectivity / *Connettivita'* | WiFi + BLE + CAN bus + IO-Link |
| Level sensor / *Sensore livello* | Optional LP sensor / *Sensore LP opzionale* |

---

## Dashboard Variables / Variabili Dashboard

| Metric / *Metrica* | Variable ID | What it shows / *Cosa mostra* |
|---|---|---|
| **Motor Hours** | `100100` | *Total runtime (seconds, displayed as hours)* |
| **Pump Rotations** | `100101` | *Total rotation count* |
| **Reservoir Level** | `100017` | *Level % (with LP sensor)* |
| **Pressure** | `100018` | *System pressure (bar)* |

---

## What Makes Bravo Different / Cosa Distingue il Bravo

- **CAN bus** — connects to industrial PLCs and automation systems / si connette a PLC e sistemi di automazione industriali
- **IO-Link** — Industry 4.0 sensor standard integration / integrazione standard sensori Industria 4.0
- **Multi-point** — serves multiple lubrication points from one unit / serve piu' punti di lubrificazione da una singola unita'

Full settings and variable reference in the [Variable Map](../platform/variable-map.md).

*Impostazioni complete e riferimento variabili nella [Mappa Variabili](../platform/variable-map.md).*
