# DRUCS v2 Dashboard / Dashboard DRUCS v2

**URL**: [app.dropsa.app/v2/](https://app.dropsa.app/v2/)

The new DRUCS v2 dashboard is a modern React application that replaces the Angular v1 frontend. It features a dark glass theme, real-time device monitoring, and predictive analytics.

*La nuova dashboard DRUCS v2 e' un'applicazione React moderna che sostituisce il frontend Angular v1. Presenta un tema vetro scuro, monitoraggio dispositivi in tempo reale e analisi predittiva.*

---

## Dashboard Layout / Layout Dashboard

### Summary Strip / Barra Riepilogo

The top row shows 6 clickable status boxes. Click any box to filter the device list below.

*La riga superiore mostra 6 riquadri stato cliccabili. Clicca su un riquadro per filtrare la lista dispositivi sottostante.*

| Box / *Riquadro* | Color / *Colore* | Meaning / *Significato* |
|---|---|---|
| **Total** | Blue / Blu | All devices / *Tutti i dispositivi* |
| **OK** | Green / *Verde* | Operating normally / *Funzionamento normale* |
| **Alarm** | Red / *Rosso* | Active alarm / *Allarme attivo* |
| **Warning** | Amber / *Ambra* | Active warning / *Avviso attivo* |
| **Inactive** | Slate / *Ardesia* | Marked inactive by user / *Contrassegnato inattivo dall'utente* |
| **Offline** | Grey / *Grigio* | Not connected / *Non connesso* |

### Device Cards / Schede Dispositivo

Each device appears as a full-width card showing:

*Ogni dispositivo appare come una scheda a larghezza piena che mostra:*

- Device name and product badge / *Nome dispositivo e badge prodotto*- Connection status (online/offline with last seen time) / *Stato connessione*- Operational status (OK/Alarm/Warning) / *Stato operativo*- Motor hours, pump cycles, reservoir level / *Ore motore, cicli pompa, livello serbatoio*- Color-coded left border matching status / *Bordo sinistro colorato in base allo stato*
Click a card to open the device detail page.

*Clicca una scheda per aprire la pagina dettaglio dispositivo.*

### Problems Panel / Pannello Problemi

Shows actionable alerts that need attention:

*Mostra avvisi che richiedono attenzione:*

- Devices in **alarm for more than 2 hours** / Dispositivi in **allarme da piu' di 2 ore**
- Devices **offline for more than 24 hours** / Dispositivi **offline da piu' di 24 ore**
- Sorted by severity (critical first) / *Ordinati per gravita' (critici prima)*
### Predictive LubeOut / LubeOut Predittivo

Predicts when each device will run out of lubricant:

*Prevede quando ogni dispositivo esaurira' il lubrificante:*

- **Devices with level sensor (analog)**: Shows current level as a gauge / Mostra livello attuale come indicatore
- **Devices without level sensor (binary)**: Analyzes refill history to estimate days remaining / Analizza storico riempimenti per stimare giorni rimanenti

| Severity / *Gravita'* | Condition / *Condizione* |
|---|---|
| Green / *Verde* | >60% remaining or >60% of avg refill interval / *>60% rimanente* |
| Orange / *Arancione* | 20-60% remaining / *20-60% rimanente* |
| Red / *Rosso* | <20% remaining — refill soon / *<20% — riempire presto* |

---

## Dark Theme / Tema Scuro

The dashboard uses a dark glass UI optimized for control room displays and low-light environments. Toggle between light and dark mode using the theme switch.

*La dashboard usa una UI vetro scura ottimizzata per display sale controllo e ambienti con poca luce. Passa tra modalita' chiara e scura usando il selettore tema.*

---

## How Data Flows / Come Fluiscono i Dati

1. Devices send telemetry every 5 seconds via WiFi / *I dispositivi inviano telemetria ogni 5 secondi via WiFi*2. Data passes through the gateway to the Laravel API / *I dati passano attraverso il gateway all'API Laravel*3. Dashboard fetches data via REST API every 30 seconds / *La dashboard recupera i dati via REST API ogni 30 secondi*4. Real-time metrics are calculated client-side / *Le metriche in tempo reale sono calcolate lato client*
---

## Supported Products / Prodotti Supportati

All DRUCS-connected products are supported. Each product shows relevant metrics:

*Tutti i prodotti connessi DRUCS sono supportati. Ogni prodotto mostra metriche rilevanti:*

| Product | Key Metrics / *Metriche Chiave* |
|---|---|
| BravoCompact / Bravo / *Smart* | Motor hours, pump rotations, reservoir level |
| SilentTrack | Rail cycles (4 zones), daily cycles, train count, rail temp |
| VipAir | Micropump cycles (8 pumps), air pressure, system pressure |

See the [Variable Map](../platform/variable-map.md) for the complete list of displayed variables.
