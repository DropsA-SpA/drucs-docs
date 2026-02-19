# Release Notes / *Note di Rilascio*

## Web Dashboard v2 / *Dashboard Web v2*

### February 2026 — Initial Release / *Febbraio 2026 — Primo Rilascio*

**URL**: [app.dropsa.app/v2/](https://app.dropsa.app/v2/)

New React-based dashboard deployed alongside the Angular v1 frontend.

*Nuova dashboard basata su React distribuita accanto al frontend Angular v1.*

#### Features / *Funzionalita'*

- Dark theme with glass UI — readable text and visible card borders / *Tema scuro con UI vetro — testo leggibile e bordi scheda visibili*
- Summary strip with 6 clickable status filters (Total, OK, Alarm, Warning, Inactive, Offline) / *Barra riepilogo con 6 filtri stato cliccabili*
- Full-width device card list with color-coded status borders / *Lista schede dispositivo con bordi colorati per stato*
- Problems panel — detects devices in alarm >2h or offline >24h / *Pannello problemi — rileva dispositivi in allarme >2h o offline >24h*
- Predictive LubeOut — estimates when devices will run out of lubricant / *LubeOut predittivo — stima quando i dispositivi esauriranno il lubrificante*
- 7-day status trend chart / *Grafico trend stato 7 giorni*
- VipAir micropump cycle aggregation / *Aggregazione cicli micropompe VipAir*
- SilentTrack zone cycle aggregation / *Aggregazione cicli zone SilentTrack*
- Support for all product types (BC, BR, SM, ST, NV, MA, VP) / *Supporto per tutti i tipi di prodotto*

---

## Firmware / Firmware

### v0.59 — February 2026 / *Febbraio 2026*

**Current production release** / *Rilascio produzione attuale*

#### Features / *Funzionalita'*
- [BC-36] ModBus: update LP sensor parameters default value and range / *ModBus: aggiornamento valori predefiniti e range parametri sensore LP*
- [BC-29] LP Sensor behaviour with new "Pressure Switch Type" parameter / *Comportamento sensore LP con nuovo parametro "Tipo Interruttore Pressione"*
- [BC-32] Revert QR code web link from "www.dropsa.app" to "drucs.net" / *Ripristino link QR code*

#### Bug Fixes / *Correzioni Bug*
- [BC-27] CANbus pause command does not stop timeout countdown / *Comando pausa CANbus non ferma countdown*
- [BC-31] Firmware fails to send alarm value in var 100016 via Bluetooth / *Firmware non invia valore allarme via Bluetooth*

#### CI/CD
- [BC-33] Add STG_PUMBCP100004 into release build / *Aggiunto STG_PUMBCP100004 nella build*

---

### v0.58.01 — February 2026 / *Febbraio 2026* — Hotfix

#### Bug Fixes / *Correzioni Bug*
- [BC-49] BLE heap exhaustion causes system reboot on startup with CAN bus — reverted `PRJ_BLE_MSG_APP_QUEUE_SIZE` from 5 to 2 / *Heap FreeRTOS esaurito all'avvio con CAN bus — ripristinato `PRJ_BLE_MSG_APP_QUEUE_SIZE` da 5 a 2*

---

### v0.58 — January 2026 / *Gennaio 2026*

#### Features / *Funzionalita'*
- Add "Low Battery" Warning / *Aggiunto avviso "Batteria Scarica"*
- Remote FW upgrade can resume download from stop point / *Aggiornamento FW remoto riprende dal punto di stop*
- Partial improvement of BLE communication / *Miglioramento parziale comunicazione BLE*
- Suspend Matrix state: DISPLAY_ACTUAL_CURRENT / *Stato matrice sospesa*

#### Bug Fixes / *Correzioni Bug*
- Remove the last "reload period" before "pause" / *Rimosso ultimo "periodo ricarica" prima pausa*
- Matrix Applied on App (input1&2 state updated) / *Matrice applicata su App*
- m_fsm_state_literals bugs fix on BLE / *Correzione bug su BLE*

---

### v0.57 — December 2025 / *Dicembre 2025*

#### Features / *Funzionalita'*
- Add release notes into GitHub / *Aggiunte note rilascio su GitHub*
- Add setting parameters matrix constraints / *Aggiunti vincoli matrice parametri*
- Add LED behaviours to enter EOL mode / *Aggiunti comportamenti LED modalita' EOL*
- Add switch to disable alarm for abnormal supply voltage / *Aggiunto interruttore disabilitazione allarme tensione*

#### Bug Fixes / *Correzioni Bug*
- FW version display on LCD / *Visualizzazione versione FW su LCD*
- Web settings state (error/hidden) / *Stato impostazioni web*

---

### v0.56 — November 2025 / *Novembre 2025*

#### Features / *Funzionalita'*
- EOL Mode transfer through buttons behaviour / *Trasferimento modalita' EOL tramite pulsanti*
- Network Optimization / *Ottimizzazione rete*

#### Bootloader
- **Bootloader v1.00.04** — upgrade by version number instead of FLAG / *aggiornamento per numero versione*
- Bootloader cannot be downgraded / *Bootloader non degradabile*

---

### v0.55 — October 2025 / *Ottobre 2025*

#### Features / *Funzionalita'*
- "Do Not Power Off" image during upgrade (from v0.55 onwards) / *Immagine "Non Spegnere" durante aggiornamento*

#### Bootloader Integration / *Integrazione Bootloader*
- Add ssd1306 drivers to bootloader / *Aggiunti driver ssd1306*
- Enable SPI1 / *Abilitato SPI1*
- Enable watchdog / *Abilitato watchdog*
- Images for FW upgrading and recovery / *Immagini per aggiornamento e recupero FW*

---

### v0.54 — September 2025 / *Settembre 2025*

- Optimization of FW download from server to less than 30 mins / *Ottimizzazione download FW a meno di 30 minuti*

---

### v0.53 — August 2025 / *Agosto 2025*

- Adjusted registration input voltage range and low level delay / *Regolato range tensione e ritardo livello basso*

---

### v0.52 — July 2025 / *Luglio 2025*

- Voltage alarms displayed with 10-second delay and not automatically dismissed / *Allarmi tensione con ritardo 10s*

---

### v0.51 — June 2025 / *Giugno 2025*

#### Features / *Funzionalita'*
- **Watchdog enabled** with 32-second timeout / *Watchdog abilitato timeout 32s*
- **Automatic restart** after Error_Handler() with 10-second delay / *Riavvio automatico dopo Error_Handler()*
- Trailer Mode: added checkbox option / *Modalita' Trailer: opzione checkbox*

#### Bootloader Integration / *Integrazione Bootloader*
- Bootloader v1.0 embedded into firmware / *Bootloader v1.0 incorporato*
- On first start, bootloader updated and device reboots (~1 second) / *Al primo avvio, bootloader aggiornato*

---

### v0.50 — May 2025 / *Maggio 2025*

#### Features / *Funzionalita'*
- **Firmware update over BLE enabled** / *Aggiornamento firmware via BLE abilitato*

#### Bug Fixes / *Correzioni Bug*
- Fixed Dummy firmware restart when flat cable disconnected / *Corretto riavvio firmware Dummy*
- Added notification screen for hardware issues / *Aggiunta schermata notifica problemi hardware*

---

*For complete release history, see `/docs/ReleaseNote.txt` in the S-00365-firmware repository.*

*Per cronologia completa, vedere `/docs/ReleaseNote.txt` nel repository S-00365-firmware.*
