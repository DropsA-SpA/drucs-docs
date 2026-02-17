# Product Catalog / Catalogo Prodotti

DropsA manufactures lubrication systems for industrial, railway, and automotive applications. Each product uses the same STM32 firmware compiled with a different product flag.

*DropsA produce sistemi di lubrificazione per applicazioni industriali, ferroviarie e automotive. Ogni prodotto utilizza lo stesso firmware STM32 compilato con un flag prodotto diverso.*

---

## Products / Prodotti

### Single-Point Lubrication / Lubrificazione Singolo Punto

| Product | Code | Description (EN) | Descrizione (IT) |
|---|---|---|---|
| [**BravoCompact 4.0**](bravocompact.md) | BC | Compact autonomous lubricator for single-point applications | *Lubrificatore autonomo compatto per applicazioni singolo punto* |
| [**Smart 4.0**](smart.md) | SM | Compact lubricator with IO-Link support | *Lubrificatore compatto con supporto IO-Link* |

### Multi-Point Lubrication / Lubrificazione Multi-Punto

| Product | Code | Description (EN) | Descrizione (IT) |
|---|---|---|---|
| [**Bravo 4.0**](bravo.md) | BR | Multi-point lubricator, CAN bus + IO-Link capable | *Lubrificatore multi-punto, compatibile CAN bus + IO-Link* |
| **OmegaPump** | OM | Motor-hours based lubricator | *Lubrificatore basato su ore motore* |
| **Maxtreme** | MA | High-pressure progressive lubricator | *Lubrificatore progressivo ad alta pressione* |
| **VIP6** | VP | Legacy VIP platform | *Piattaforma VIP legacy* |

### Specialized Systems / Sistemi Specializzati

| Product | Code | Description (EN) | Descrizione (IT) |
|---|---|---|---|
| [**SilentTrack**](silenttrack.md) | ST | Railway rail lubrication system, 4 zones, weather-adaptive | *Sistema lubrificazione rotaie, 4 zone, adattivo al meteo* |
| [**VipAir 4.0**](vipair.md) | NV | Air-oil minimal quantity lubrication, 8 micropumps | *Lubrificazione minimale aria-olio, 8 micropompe* |

---

## Connectivity / Connettivita'

All products support:

- **WiFi** — via ESP32 co-processor, connects to DRUCS cloud / tramite co-processore ESP32, si connette al cloud DRUCS
- **Bluetooth LE** — for local configuration via mobile app / per configurazione locale tramite app mobile

Some products additionally support:

- **CAN bus** — Bravo 4.0, VipAir (industrial bus integration) / integrazione bus industriale
- **IO-Link** — Bravo 4.0, Smart 4.0 (Industry 4.0 sensor standard) / standard sensori Industria 4.0
- **Modbus** — selected models (industrial protocol) / protocollo industriale

---

## Firmware Versions / Versioni Firmware

| Version / *Versione* | Date / *Data* | Key Changes / *Modifiche Principali* |
|---|---|---|
| **v0.59** | Feb 2026 | Current release / *Rilascio attuale* |
| v0.58 | Jan 2026 | BLE MTU improvements / *Miglioramenti MTU BLE* |
| v0.57 | Dec 2025 | VipAir micropump enhancements / *Miglioramenti micropompe VipAir* |

Full release history in [Release Notes](../roadmap/releases.md).

*Cronologia completa dei rilasci nelle [Note di Rilascio](../roadmap/releases.md).*
