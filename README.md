# 🧩 Scarabeo Arena

Scarabeo Arena è una piattaforma multiplayer online per il gioco dello Scarabeo, sviluppata con un'architettura **Node.js (Backend)** e **Vanilla JavaScript (Frontend)**. Il progetto include un sistema di gestione lobby, un bot intelligente per sfide in solitaria e una validazione completa delle regole ufficiali italiane.

---

## 🚀 Funzionalità Principali

- **Multigiocatore in tempo reale:** Gestione dei turni sincronizzata tramite polling intelligente.
- **Bot Integrato:** Un avversario IA con diversi livelli di difficoltà che gioca automaticamente se attivato.
- **Sistema di Punteggio Ufficiale:** Include i bonus per lettere (2L, 3L) e parole (2P, 3P), oltre ai bonus "Scarabeo" (100 punti).
- **Documentazione API:** Integrazione con Swagger per testare gli endpoint in tempo reale.
- **Persistenza Dati:** Gestione dei dati tramite file JSON per una configurazione rapida e portabile senza necessità di database complessi.

---

## 📂 Struttura delle Cartelle

```text
.
├── data/               # File JSON per la persistenza (auto-generati)
├── src/                # BACKEND
│   ├── db/             # Logica database e gestione file JSON
│   ├── middleware/     # Autenticazione API Key
│   ├── routes/         # Endpoint API (Auth e Games)
│   ├── utils/          # Logica Bot e calcolo punteggi
│   ├── index.js        # Entry point del server Express
│   └── swagger.js      # Configurazione documentazione API
├── static/             # FRONTEND
│   ├── js/             # Logica client-side (State, UI, API, Scrabble)
│   ├── styles/         # Fogli di stile CSS
│   └── index.html      # Entry point dell'applicazione
└── package.json        # Dipendenze e script Node.js
