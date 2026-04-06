# 🎰 BetX — Fantasy Betting Platform

> A real-time fantasy sports betting web app where all users share the same live game round. Built with pure HTML/CSS/JS, Firebase Realtime Database, and The Odds API.

🔗 **Repo:** [github.com/prasanthmukkiri/BetX](https://github.com/prasanthmukkiri/BetX)  
👤 **Author:** Prasanth Mukkiri

---

## 📁 Project Structure

```
BetX/
├── Dashboard/              # Admin panel — controls game for all users
│   └── *.html / *.js
│
└── Fantacygames/           # Main user-facing betting game
    └── colourgame.html     # Core game page (live betting UI)
```

---

## 🛠️ Tech Stack

| Technology | Purpose |
|---|---|
| HTML / CSS / JavaScript | Frontend — no frameworks |
| Firebase Realtime Database | Live game state sync across all users |
| Firebase Authentication | User login (if enabled) |
| The Odds API | Fetching live sports odds |

---

## 🔥 Firebase Realtime Database Structure

```
Firebase RTDB: parione-ee49d-default-rtdb.firebaseio.com
│
├── currentRound/
│   ├── phase: "betting"           # Game phase: betting | locked | result
│   ├── roundNum: 1                # Current round number
│   └── startTime: 1775459381221  # Unix timestamp (ms)
│
├── users/
│   └── {userId}/
│       ├── name: "username"
│       ├── balance: 1000
│       └── bets/...
│
└── rounds/
    └── {roundNum}/                # Historical round data
```

---

## 🎮 How the Game Works

```
ADMIN (Dashboard)                    USERS (Fantacygames)
      │                                      │
      ▼                                      ▼
Start Round                        See "Betting is Open"
phase = "betting"      ────►       Place their bets
      │                                      │
      ▼                                      ▼
Lock Round                         See "Round Locked"
phase = "locked"       ────►       Bets are frozen
      │                                      │
      ▼                                      ▼
Declare Result                     See Winner/Loser
phase = "result"       ────►       Balance updated
      │
      ▼
Start Next Round
roundNum + 1
```

> ✅ All users see the **same game state** in real-time via Firebase `.on()` listener.

---

## 📡 Real-time Data Sync

```javascript
// ✅ CORRECT — live listener, updates all users instantly
db.ref("currentRound").on("value", (snapshot) => {
  const { phase, roundNum, startTime } = snapshot.val();
  // Update UI based on phase
});

// ❌ WRONG — one-time fetch, users won't get live updates
db.ref("currentRound").get().then(...)
```

---

## 🌐 The Odds API

```javascript
const apiKey = "YOUR_API_KEY"; // Get from: https://the-odds-api.com

fetch(`https://api.the-odds-api.com/v4/sports/soccer_epl/odds?apikey=${apiKey}&regions=uk&markets=h2h`)
  .then(res => res.json())
  .then(data => console.log(data));
```

**Common Errors:**

| Error | Cause | Fix |
|---|---|---|
| `MISSING_KEY` | Key expired or invalid | Refresh from dashboard |
| Key is `undefined` | Firebase returning null | `console.log` your key before using |
| No data returned | Free quota exceeded | Check usage on API dashboard |

---

## 🔧 Admin Panel (`/Dashboard`)

The admin controls the entire game for ALL users simultaneously.

```javascript
// Start new round
db.ref("currentRound").set({
  phase: "betting",
  roundNum: newRoundNum,
  startTime: Date.now()
});

// Lock round (no more bets)
db.ref("currentRound/phase").set("locked");

// Declare result
db.ref("currentRound/phase").set("result");
```

---

## ⚙️ Firebase Setup

```javascript
const firebaseConfig = {
  apiKey: "YOUR_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  databaseURL: "https://parione-ee49d-default-rtdb.firebaseio.com",
  projectId: "YOUR_PROJECT",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_ID",
  appId: "YOUR_APP_ID"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.database();
```

---

## 🚀 Running Locally

> ⚠️ **Never open HTML files directly** with `file:///` — causes CORS errors with Firebase & APIs.

**Option 1 — VS Code Live Server (Recommended)**
1. Install **Live Server** extension
2. Right-click any `.html` file → **Open with Live Server**

**Option 2 — Python**
```bash
cd BetX
python -m http.server 8080
# Visit: http://localhost:8080
```

**Option 3 — Node.js**
```bash
npx serve .
```

---

## 🔐 Firebase Security Rules

```json
{
  "rules": {
    "currentRound": {
      ".read": true,
      ".write": false
    },
    "users": {
      "$uid": {
        ".read": "auth.uid === $uid",
        ".write": "auth.uid === $uid"
      }
    }
  }
}
```

---

## 🐛 Common Issues

| Problem | Cause | Fix |
|---|---|---|
| CORS error in console | Opened via `file:///` | Use Live Server |
| Users not syncing live | Using `.get()` not `.on()` | Switch to `.on("value", ...)` |
| API key not working | Key undefined from Firebase | `console.log` before fetch |
| Admin changes not reflecting | Wrong DB ref path | Verify path matches DB structure |

---

## 📌 Key Notes for New Developers / AI Assistants

- This is a **pure HTML/JS project** — no React, no Vue, no bundler
- All files run from the browser — use a **local server** for development
- **`currentRound`** in Firebase is the single source of truth — changing it affects ALL users instantly
- **Admin panel** (`/Dashboard`) is the only place that writes to `currentRound`
- Users in `/Fantacygames` only **read** from Firebase (except placing bets)
- API keys should **never be pushed to GitHub** — use Firebase to store them securely

---

## 🗺️ Roadmap (Planned Features)

- [ ] Leaderboard
- [ ] User wallet & transaction history
- [ ] Multiple game types
- [ ] Mobile responsive UI
- [ ] Notifications for round changes
