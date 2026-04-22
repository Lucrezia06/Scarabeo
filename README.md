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

---
## 📂 Struttura del Progetto

Il progetto è diviso in due macro-aree:

### ⚙️ BACKEND (Server)
- `src/index.js`: Il punto d'ingresso del server Express.
- `src/db/database.js`: Il motore di persistenza che gestisce utenti, partite e mosse.
- `src/routes/`: Definizione degli endpoint API (`auth.js` e `games.js`).
- `src/utils/bot-utils.js`: Logica decisionale dell'IA e calcolo matematico dei punteggi.
- `src/middleware/auth.js`: Protezione delle rotte tramite API Key.

### 🎨 FRONTEND (Client)
- `static/index.html`: La struttura base della Single Page Application (SPA).
- `static/js/state.js`: Gestione centralizzata dello stato (Single Source of Truth).
- `static/js/ui.js`: Rendering dinamico della plancia e interazione utente.
- `static/js/scrabble.js`: Motore delle regole (validazione posizionamento e dizionario).
- `static/js/api.js`: Client HTTP con logica di retry automatica.

---

## 🛠️ Requisiti e Installazione

### Prerequisiti
- Node.js (versione 14 o superiore)
- npm (Node Package Manager)

### Installazione
1. Clona il repository:
   ```bash
   git clone [https://github.com/tuo-username/scarabeo-arena.git](https://github.com/tuo-username/scarabeo-arena.git)
   cd scarabeo-arena
Installa le dipendenze:

Bash
npm install
Avvia il server:
cd /workspaces/Scarabeo/Scarabeo/GameApi
npm start

Bash
npm start
Il server sarà attivo su http://localhost:3000. Puoi visualizzare la documentazione delle API su http://localhost:3000/api-docs.
