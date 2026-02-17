# Development Roadmap / Roadmap di Sviluppo

## Current Status (February 2026) / Stato Attuale (Febbraio 2026)

| Phase / *Fase* | Status / *Stato* | Description / *Descrizione* |
|---|---|---|
| **Phase 0** | **DONE** | *Security audit + access lockdown / Audit sicurezza + blocco accessi* |
| **Phase 1** | **IN PROGRESS** | *Web v2 dashboard (React) / Dashboard web v2 (React)* |
| **Phase 2** | Planned / *Pianificata* | *API modernization + new endpoints / Modernizzazione API + nuovi endpoint* |
| **Phase 3** | Planned / *Pianificata* | *Protocol bridge (HTTP --> MQTT) / Bridge protocollo* |
| **Phase 4** | Future / *Futuro* | *Mobile app feature sync / Sincronizzazione funzionalita' app mobile* |
| **Phase 5** | Future / *Futuro* | *Advanced analytics + AI / Analisi avanzata + AI* |

---

## Phase 0: Security & Cleanup (COMPLETE) / Sicurezza e Pulizia (COMPLETATA)

- [x] GitHub org audit — downgraded vendor access / *Audit org GitHub — declassato accesso fornitore*- [x] AWS IAM audit — deactivated unused keys / *Audit AWS IAM — disattivate chiavi non usate*- [x] Org default permissions set to read-only / *Permessi org predefiniti impostati a sola lettura*- [x] GitHub Teams created (Project Admins, Electronics, Management, Field Engineers)
- [x] Firmware repo migrated to DropsA-SpA org / *Repo firmware migrato nell'org DropsA-SpA*- [ ] Branch cleanup — 163 stale branches to delete / *Pulizia branch — 163 branch obsoleti da eliminare*
---

## Phase 1: Web v2 Dashboard (IN PROGRESS) / Dashboard Web v2 (IN CORSO)

The new React-based dashboard replacing the Angular v1 frontend.

*La nuova dashboard basata su React che sostituisce il frontend Angular v1.*

### Completed / Completato

- [x] Dark theme glass UI with readable text / *UI vetro tema scuro con testo leggibile*- [x] Clickable summary strip (Total/OK/Alarm/Warning/Inactive/Offline filters) / *Barra riepilogo cliccabile con filtri*- [x] Full-width device card list with status, level gauge, cycle counters / *Lista schede dispositivo a larghezza piena*- [x] Problems panel (alarm >2h, offline >24h detection) / *Pannello problemi (allarme >2h, offline >24h)*- [x] Predictive LubeOut (estimates when lubricant runs out) / *LubeOut predittivo (stima esaurimento lubrificante)*- [x] Status bar chart (7-day trend) / *Grafico barre stato (trend 7 giorni)*- [x] Hours/cycles metrics with VipAir micropump aggregation / *Metriche ore/cicli con aggregazione micropompe VipAir*- [x] Deployed to app.dropsa.app/v2/ / *Distribuito su app.dropsa.app/v2/*
### Next / Prossimo

- [ ] Device detail page — full variable display, settings editor, event history / *Pagina dettaglio dispositivo — visualizzazione variabili, editor impostazioni, storico eventi*- [ ] Per-user "Inactive" device flag / *Flag dispositivo "Inattivo" per utente*- [ ] Notifications panel (in-app) / *Pannello notifiche (in-app)*- [ ] Help page integration (embed from this docs site) / *Integrazione pagine aiuto (incorporate da questo sito)*
---

## Phase 2: API Modernization / Modernizzazione API

- [ ] New REST endpoints for v2 dashboard needs / *Nuovi endpoint REST per le esigenze della dashboard v2*- [ ] Device settings read/write API / *API lettura/scrittura impostazioni dispositivo*- [ ] Event history with filtering and pagination / *Storico eventi con filtro e paginazione*- [ ] User preferences API (per-user device flags, notification settings) / *API preferenze utente*
---

## Phase 3: Protocol Bridge / Bridge Protocollo

- [ ] Evaluate MQTT broker (Mosquitto or cloud-managed) / *Valutare broker MQTT*- [ ] Build protocol bridge: translate current HTTP+AES to MQTT+TLS / *Costruire bridge protocollo*- [ ] Maintain backward compatibility with existing devices / *Mantenere compatibilita' con dispositivi esistenti*- [ ] New firmware builds to support MQTT natively / *Nuove build firmware per supporto MQTT nativo*
---

## Phase 4: Mobile App Sync / Sincronizzazione App Mobile

- [ ] Replicate web v2 features to Flutter app / *Replicare funzionalita' web v2 nell'app Flutter*- [ ] Shared API — same backend, same data / *API condivisa — stesso backend, stessi dati*- [ ] Embedded help pages via WebView / *Pagine aiuto incorporate tramite WebView*- [ ] Push notifications / *Notifiche push*
---

## Phase 5: Advanced Analytics / Analisi Avanzata

- [ ] Predictive maintenance models / *Modelli manutenzione predittiva*- [ ] Fleet-level analytics and reporting / *Analisi e reportistica a livello flotta*- [ ] Lubricant consumption optimization / *Ottimizzazione consumo lubrificante*- [ ] AI-assisted troubleshooting / *Risoluzione problemi assistita da AI*
---

## How to Request Features / Come Richiedere Funzionalita'

Field engineers and stakeholders can request features through the [Feature Request](../field/feature-request.md) page. No GitHub account needed.

*I tecnici sul campo e gli stakeholder possono richiedere funzionalita' tramite la pagina [Richiesta Funzionalita'](../field/feature-request.md). Non serve un account GitHub.*

---

*Full detailed roadmap available as a separate document (DRUCS-Unified-Modernization-Roadmap).*

*Roadmap dettagliata completa disponibile come documento separato (DRUCS-Unified-Modernization-Roadmap).*
