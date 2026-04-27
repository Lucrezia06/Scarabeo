# SCARABEO-lucrezia e cristiano 🎮

Backend REST API per il gioco del Scarabeo (Scrabble italiano) con supporto per giocatori umani, bot e partite multiplayer.

## 📋 Indice

- [Come Funziona il Progetto](#come-funziona-il-progetto)
- [Setup e Avvio](#setup-e-avvio)
- [Autenticazione](#autenticazione)
- [Endpoint API](#endpoint-api)
- [Esempi di Utilizzo](#esempi-di-utilizzo)
- [Come Funziona il Matching](#come-funziona-il-matching)
- [Regole di Scarabeo](#regole-di-scarabeo)

---

## Come Funziona il Progetto

### Architettura

```
┌─────────────────────────────────────┐
│   Client Frontend (Static)           │
│  (HTML, JS, CSS, Dizionario IT)     │
└────────────────┬────────────────────┘
                 │ HTTP/REST
┌────────────────▼────────────────────┐
│    Express.js Backend API           │
│  - Autenticazione (API Key)         │
│  - Gestione Partite                 │
│  - Gestione Giocatori               │
│  - Bot Automatico                   │
│  - Calcolo Punteggi                 │
└────────────────┬────────────────────┘
                 │ File System
┌────────────────▼────────────────────┐
│    Database JSON (data/)             │
│  - users.json (utenti + API key)    │
│  - games.json (partite + mosse)     │
└─────────────────────────────────────┘
```

### Flusso Principali

#### 1️⃣ Registrazione Utente
```
[Cliente] → POST /auth/register
           ↓
[Backend] → Crea nuovo user + API Key
           ↓
[Storage] → Salva in users.json
           ↓
[Cliente] ← Riceve API Key (per autenticarsi)
```

#### 2️⃣ Creazione Partita
```
[Cliente] → POST /games (con X-API-Key header)
           ↓
[Backend] → Verifica API Key
           ↓
[Backend] → Crea nuova partita (public o private)
           ↓
[Storage] → Salva in games.json
           ↓
[Cliente] ← Riceve gameId
```

#### 3️⃣ Aggiunta Giocatore
```
[Cliente] → POST /games/{gameId}/players
           ↓
[Backend] → Verifica gameId valido
           ↓
[Backend] → Aggiunge giocatore a players[]
           ↓
[Storage] → Salva in games.json
           ↓
[Cliente] ← Riceve player object
```

#### 4️⃣ Aggiunta Mossa
```
[Cliente] → POST /games/{gameId}/moves
           ↓
[Backend] → Valida giocatore e mossa
           ↓
[Backend] → Calcola punteggio con regole Scarabeo
           ↓
[Backend] → Aggiunge a moves[]
           → **Se esiste bot, schedula mossa bot automatica**
           ↓
[Storage] → Salva in games.json
           ↓
[Cliente] ← Riceve move object + punteggio
```

#### 5️⃣ Bot Automatico
```
Quando bot viene aggiunto a partita:
   ├─ Genera nome casuale (es: "Bot-Max")
   ├─ Schedula mossa ogni 2-5 secondi
   ├─ Sceglie parola dal pool italiano
   ├─ Calcola punteggio automaticamente
   ├─ Se indietro nei punti: preferisce parole lunghe
   └─ Continua finché partita è "active"
```

---

## Setup e Avvio

### Prerequisiti
- Node.js 16+
- npm

### Installazione

```bash
cd GameApi
npm install
```

### Avvio Server

```bash
npm start
# Server avviato su http://localhost:3000
```

### Accesso API Documentation

Swagger UI disponibile su:
```
http://localhost:3000/api-docs
```

### Frontend

Accedi su:
```
http://localhost:3000
```

---

## Autenticazione

### Come Funziona

Tutte le route protette richiedono header:
```
X-API-Key: <your-api-key>
```

### Flow di Autenticazione

1. **Registrazione** (No auth required):
   ```bash
   POST /auth/register
   ```
   Risposta contiene l'`apiKey`

2. **Uso API Key**: Includi in ogni richiesta protetta:
   ```bash
   curl -H "X-API-Key: abc123..." http://localhost:3000/games
   ```

3. **Errore di Autenticazione**:
   ```json
   {
     "error": "Invalid API key"
   }
   ```

---

## Endpoint API

### 🔓 Public Endpoints (No Auth)

#### Registrazione Utente
```
POST /auth/register
Content-Type: application/json

{
  "username": "john_doe"
}

Response 201:
{
  "message": "User registered successfully",
  "user": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "username": "john_doe",
    "apiKey": "550e8400-e29b-41d4-a716-446655440000550e8400e29b41d4a716446655440000",
    "createdAt": "2025-04-27T10:30:00Z"
  }
}
```

---

### 🔒 Protected Endpoints (Require X-API-Key)

#### Creare Partita
```
POST /games
Headers: X-API-Key: <your-api-key>
Content-Type: application/json

{
  "name": "Partita Veloce",
  "isPrivate": false,
  "addBotOnStart": false
}

Response 201:
{
  "message": "Game created successfully",
  "game": {
    "id": "660f9511-f30c-52e5-b827-557766551111",
    "name": "Partita Veloce",
    "ownerId": "550e8400-e29b-41d4-a716-446655440000",
    "players": [],
    "moves": [],
    "status": "active",
    "isPrivate": false,
    "createdAt": "2025-04-27T10:30:00Z",
    "expiresAt": null,
    "botAddedAt": null
  }
}
```

#### Creare Partita Privata (con Bot automatico dopo 5 min)
```
POST /games
Headers: X-API-Key: <your-api-key>

{
  "name": "Sfida Bot",
  "isPrivate": true,
  "addBotOnStart": false
}

Comportamento:
- Partita scade dopo 5 minuti
- Se nessuno si unisce prima della scadenza,
  un bot viene aggiunto automaticamente
```

#### Ottenere tutte le partite
```
GET /games
Headers: X-API-Key: <your-api-key>

Response 200:
{
  "count": 2,
  "games": [
    { game object 1 },
    { game object 2 }
  ]
}
```

#### Ottenere dettagli partita
```
GET /games/{gameId}
Headers: X-API-Key: <your-api-key>

Response 200:
{
  "game": { game object }
}

Response 404:
{
  "error": "Game not found"
}
```

#### Aggiungere giocatore a partita
```
POST /games/{gameId}/players
Headers: X-API-Key: <your-api-key>

{
  "name": "Alice"
}

Response 201:
{
  "message": "Player added successfully",
  "player": {
    "id": "770g0622-g41d-63f6-c938-668877662222",
    "name": "Alice",
    "joinedAt": "2025-04-27T10:35:00Z",
    "isBot": false
  }
}
```

#### Ottenere giocatori partita
```
GET /games/{gameId}/players
Headers: X-API-Key: <your-api-key>

Response 200:
{
  "gameId": "660f9511-f30c-52e5-b827-557766551111",
  "count": 2,
  "players": [
    {
      "id": "770g0622-g41d-63f6-c938-668877662222",
      "name": "Alice",
      "isBot": false,
      "joinedAt": "2025-04-27T10:35:00Z"
    },
    {
      "id": "880h1733-h52e-74g7-d949-779988773333",
      "name": "Bot-Pro",
      "isBot": true,
      "joinedAt": "2025-04-27T10:40:00Z",
      "difficulty": "medium"
    }
  ]
}
```

#### Aggiungere Bot automatico
```
POST /games/{gameId}/bot
Headers: X-API-Key: <your-api-key>

{
  "botName": "Campione",
  "difficulty": "hard"
}

Response 201:
{
  "message": "Bot added successfully",
  "bot": {
    "id": "880h1733-h52e-74g7-d949-779988773333",
    "name": "Campione",
    "isBot": true,
    "difficulty": "hard",
    "joinedAt": "2025-04-27T10:40:00Z"
  }
}

⚠️ Note: Il bot inizia subito a giocare automaticamente
```

#### Aggiungere Mossa
```
POST /games/{gameId}/moves
Headers: X-API-Key: <your-api-key>

{
  "playerId": "770g0622-g41d-63f6-c938-668877662222",
  "data": {
    "word": "AMORE",
    "score": 45,
    "tiles": [
      { "letter": "A", "multiplier": "2L", "isJolly": false },
      { "letter": "M", "multiplier": null, "isJolly": false },
      { "letter": "O", "multiplier": null, "isJolly": false },
      { "letter": "R", "multiplier": "3P", "isJolly": false },
      { "letter": "E", "multiplier": null, "isJolly": false }
    ]
  }
}

Response 201:
{
  "message": "Move added successfully",
  "move": {
    "id": "990i2844-i63f-85h8-e050-8800991118844",
    "playerId": "770g0622-g41d-63f6-c938-668877662222",
    "data": { move data },
    "timestamp": "2025-04-27T10:45:00Z"
  }
}

🤖 Nota: Se esiste un bot, entro 2-5 secondi
          il bot gioca automaticamente!
```

#### Ottenere tutte le mosse
```
GET /games/{gameId}/moves
Headers: X-API-Key: <your-api-key>

Response 200:
{
  "gameId": "660f9511-f30c-52e5-b827-557766551111",
  "count": 3,
  "moves": [ array di moves ]
}
```

#### Ottenere Leaderboard
```
GET /games/{gameId}/leaderboard
Headers: X-API-Key: <your-api-key>

Response 200:
{
  "gameId": "660f9511-f30c-52e5-b827-557766551111",
  "leaderboard": [
    {
      "id": "880h1733-h52e-74g7-d949-779988773333",
      "name": "Bot-Pro",
      "isBot": true,
      "score": 250,
      "moveCount": 5
    },
    {
      "id": "770g0622-g41d-63f6-c938-668877662222",
      "name": "Alice",
      "isBot": false,
      "score": 185,
      "moveCount": 4
    }
  ]
}
```

#### Rimuovere giocatore
```
DELETE /games/{gameId}/players/{playerId}
Headers: X-API-Key: <your-api-key>

Response 200:
{
  "message": "Player removed successfully"
}
```

#### Cancellare partita
```
DELETE /games/{gameId}
Headers: X-API-Key: <your-api-key>

Response 200:
{
  "message": "Game deleted successfully"
}
```

---

## Esempi di Utilizzo

### Scenario 1: Creare una Partita e Aggiungere Giocatori

```bash
# 1. Registra utente
curl -X POST http://localhost:3000/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username": "alice"}'

# Risposta:
# {
#   "apiKey": "abc123def456..."
# }

# 2. Crea partita
curl -X POST http://localhost:3000/games \
  -H "X-API-Key: abc123def456..." \
  -H "Content-Type: application/json" \
  -d '{"name": "Sfida", "isPrivate": false}'

# Risposta:
# {
#   "game": {
#     "id": "game123..."
#   }
# }

# 3. Aggiungi giocatore 1
curl -X POST http://localhost:3000/games/game123.../players \
  -H "X-API-Key: abc123def456..." \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice"}'

# 4. Aggiungi giocatore 2
curl -X POST http://localhost:3000/games/game123.../players \
  -H "X-API-Key: abc123def456..." \
  -H "Content-Type: application/json" \
  -d '{"name": "Bob"}'

# 5. Visualizza partita
curl -X GET http://localhost:3000/games/game123... \
  -H "X-API-Key: abc123def456..."
```

### Scenario 2: Giocare con Bot

```bash
# 1. Crea partita
curl -X POST http://localhost:3000/games \
  -H "X-API-Key: abc123def456..." \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Sfida Bot",
    "isPrivate": false,
    "addBotOnStart": false
  }'

# 2. Aggiungi giocatore umano
curl -X POST http://localhost:3000/games/game123.../players \
  -H "X-API-Key: abc123def456..." \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice"}'

# 3. Aggiungi bot
curl -X POST http://localhost:3000/games/game123.../bot \
  -H "X-API-Key: abc123def456..." \
  -H "Content-Type: application/json" \
  -d '{
    "botName": "Campione",
    "difficulty": "medium"
  }'
# Il bot inizia a giocare AUTOMATICAMENTE!

# 4. Gioca una mossa (Alice)
curl -X POST http://localhost:3000/games/game123.../moves \
  -H "X-API-Key: abc123def456..." \
  -H "Content-Type: application/json" \
  -d '{
    "playerId": "alice-player-id",
    "data": {
      "word": "GATTO",
      "score": 30,
      "tiles": [...]
    }
  }'
# Dopo pochi secondi, il bot gioca automaticamente!

# 5. Visualizza leaderboard
curl -X GET http://localhost:3000/games/game123.../leaderboard \
  -H "X-API-Key: abc123def456..."
```

### Scenario 3: Partita Privata con Timeout

```bash
# Crea partita privata
curl -X POST http://localhost:3000/games \
  -H "X-API-Key: abc123def456..." \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Privata",
    "isPrivate": true,
    "addBotOnStart": false
  }'

# Comportamento:
# - Partita scade in 5 minuti
# - Se nessun giocatore si unisce entro 5 min,
#   un bot viene aggiunto automaticamente
# - Se un giocatore si unisce prima di 5 min,
#   il timeout viene resettato
```

---

## Come Funziona il Matching

### Tipologie di Partite

#### 🌍 Partita Pubblica
```
- Accessibile a qualcuno che ha l'ID
- Senza limite di tempo
- Può avere sia giocatori umani che bot
- Rimane attiva finché status = "active"
```

#### 🔒 Partita Privata
```
- Timeout di 5 minuti dalla creazione
- Se timeout scade senza giocatori,
  bot viene aggiunto automaticamente
- Se almeno un giocatore si unisce,
  timeout viene resettato
```

### Ordine di Gioco

Le mosse vengono registrate nell'ordine in cui vengono ricevute:

```
Timeline:
  14:00 - Alice gioca "CANE" (45 pts)
  14:02 - Bob gioca "MARE" (38 pts)
  14:03 - Bot gioca "AMORE" (55 pts)  ← Bot ha indietro, gioca word più lunga
  14:05 - Alice gioca "ORO" (12 pts)
```

### Bot Intelligence

Il bot cambia strategia in base al punteggio:

```javascript
if (botScore < averageOtherPlayersScore) {
  // Indietro: preferisci parole lunghe (6+ lettere)
  useLongerWords = true;
  bonusPoints = +30;
} else {
  // In vantaggio: scegli parola casuale normale
  useLongerWords = false;
}
```

### Calcolo Leaderboard

La leaderboard è calcolata in tempo reale:

```
1. Per ogni giocatore:
   - Somma tutti i punteggi delle sue mosse
   - Conta il numero di mosse giocate

2. Ordina per punteggio (descrescente)

3. Se pareggio, mantiene ordine di join

Esempio:
  Posizione 1: Bot-Pro (5 mosse, 250 punti)
  Posizione 2: Alice (4 mosse, 185 punti)
  Posizione 3: Bob (3 mosse, 92 punti)
```

---

## Regole di Scarabeo

### Valori Lettere

```
1 punto:  A, C, E, I, O, R, S, T
2 punti:  L, M, N
3 punti:  P
4 punti:  B, D, F, G, U, V
8 punti:  H, Z
10 punti: Q
Jolly:    0 punti
```

### Moltiplicatori

- **Lettere**: 2L (×2) o 3L (×3)
- **Parola**: 2P (×2) o 3P (×3)

### Bonus Lunghezza

```
6 lettere:  +10 punti
7 lettere:  +30 punti
8+ lettere: +50 punti
```

### Bonus "Puro"

Se una parola con bonus NON contiene jolly:
```
Bonus Totale = BaseBonusPoints + 10
```

### Bonus Speciale "SCARABEO"

Se la parola è "SCARABEO" o "SCARABEI":
```
Bonus = +100 punti
```

### Calcolo Esempio

```
Parola: "AMORE" (5 lettere, no jolly)
Tiles:
  A: 1 pt × 2L = 2
  M: 2 pt × 1 = 2
  O: 1 pt × 1 = 1
  R: 1 pt × 3P = 3
  E: 1 pt × 1 = 1
  ───────────────
  Base Score: 9

Con multiplicatori:
  Base (9 pts) × 3P (parola) = 27 + 10 (bonus lunghezza) = 37 punti

Senza jolly: +10 bonus puro
Total: 37 + 10 = 47 punti ✓
```

---

## Limitazioni Attuali e Miglioramenti Futuri

### ⚠️ Limitazioni Attuali
- Database in JSON (no persistence su crash)
- Nessun rate limiting
- CORS permissivo
- API key non hashate

### 🔄 Miglioramenti Pianificati
- Migrare a SQLite/PostgreSQL
- Implementare rate limiting
- Hash delle API key con bcrypt
- WebSocket per real-time updates
- Validazione completa UUID
- Bot con IA più avanzata

---

## 📞 Supporto

Per bug o suggerimenti, apri una issue su GitHub.
