# 🏠 Home Search Tracker

A personal, mobile-optimized home search tracker for Jugal's active Chicago home search. Tracks 31 properties across West Town, West Loop, River West, Fulton Market, East Village, Wicker Park, and Ukrainian Village.

**Live app:** https://jugaljugaljugal-personal.github.io/home-tracker/
**PIN:** 1936

---

## What It Does

- **Tracks 31 properties** with price, status, beds/baths, type, neighborhood, walk time to Dyson HQ, Redfin + Zillow links
- **Star rankings** and a **weighted scoring system** (location, layout, condition, financial) per property
- **Tour checklist** — 90+ items across 11 sections to evaluate each home during a visit
- **Interactive map** (Leaflet + OpenStreetMap) with color-coded status markers
- **Editable notes** per property, synced in real-time across devices via Firebase
- **Mobile-first card view** with Apple Maps deep links for on-the-go touring
- **Automated monitoring** — scheduled tasks check Redfin/Zillow every 8 hours and scan Gmail every 6 hours for new listings from Kevin

---

## Tech Stack

| Layer | Tech |
|---|---|
| App | Single-file HTML/CSS/JS (`index.html`) |
| Hosting | GitHub Pages (auto-deploys from `main` branch) |
| Database | Firebase Realtime Database (v8 compat SDK via CDN) |
| Map | Leaflet.js v1.9.4 + OpenStreetMap |
| Auth | PIN gate with SHA-256 hash stored in Firebase |
| Automation | Cowork scheduled tasks (Claude Code agents) |

---

## Project Structure

```
home-tracker/
  index.html       # Entire app — ~1450 lines
  README.md        # This file
  CLAUDE.md        # Context file for Claude Code sessions
  TODO.md          # Feature backlog and bug tracker

Local workspace (iCloud):
  ~/Library/Mobile Documents/.../House hunt/
    index.html     # Source of truth — edit here, push to repo
    CLAUDE.md
    TODO.md
    migrate.html   # One-time migration tool (deprecated, keep for reference)
```

---

## Data Architecture

### Property data (L_DATA)
Defined as a JavaScript array in `index.html` (~line 395). Each property object:
```js
{
  n: 1,                     // sequential ID (Firebase key)
  addr: "123 N Example St", // full address
  hood: "West Town",        // neighborhood
  zip: "60622",
  price: 950000,            // in dollars
  beds: 3,
  baths: 2.5,
  sqft: 2500,               // or null
  type: "Condo",
  st: "ACTV",               // NEW | ACTV | PRIV-ACTV | PRIV-CTG | CTG | PEND | SOLD
  cr: "YES",                // YES | CTG | PEND | NO (Jugal's interest level)
  listed: "2026-03-20",     // YYYY-MM-DD or null
  notes: "...",             // static notes / change history
  rf: "https://redfin.com/...",
  zl: "https://zillow.com/...",
  lat: 41.8983,
  lon: -87.6558
}
```

**To update a property:** Edit `L_DATA` in `index.html`, commit, push.
**To add a property:** Append to the array with the next sequential `n` value.

### User data (Firebase)
User annotations stored at `https://home-tracker-4a4f2-default-rtdb.firebaseio.com`:
```
/config/pinHash          → SHA-256 hash of PIN
/config/lastUpdated      → ISO timestamp of last automated update
/tracker/ranks/{n}       → 1–5 star rating
/tracker/notes/{n}       → user's personal notes (editable in UI)
/tracker/scores/{n}      → { critId: 1-5 | "NA", ... }
/tracker/checklists/{n}  → { sectionId: { "item text": true, ... } }
/tracker/listed/{n}      → listing date override (YYYY-MM-DD)
/tracker/weights         → { location:25, layout:25, condition:25, financial:25 }
```

Property facts (price, status, address) live **only in L_DATA** — not in Firebase.
User annotations (stars, notes, scores) live **only in Firebase** — not in L_DATA.

---

## Development Workflow

### Making changes
```bash
# 1. Edit index.html in the local project folder
#    ~/Library/Mobile Documents/.../House hunt/index.html
#    (or via Claude Code)

# 2. Copy to the cloned repo and push
cp "[House hunt path]/index.html" /tmp/home-tracker/
cd /tmp/home-tracker
git add index.html
git commit -m "Update: describe the change"
git push origin main

# GitHub Pages deploys automatically in ~60 seconds
```

### With Claude Code
Open a session in the project folder and describe the change:
```
> Update property #5 price to $950K and mark it Contingent
> Add a new property: 123 N Example St, West Town, $975K, 3 bed 2 bath
> Fix the mobile filter bug
```
Claude Code will edit `index.html`, commit, and push.

### Fresh clone (e.g., on a new machine)
```bash
git clone https://[PAT]@github.com/jugaljugaljugal-personal/home-tracker.git
cd home-tracker
# Edit index.html, commit, push as above
```

---

## Automated Tasks

Two Cowork scheduled tasks keep the tracker up to date automatically:

### `home-search-status-check` — every 8 hours
- Reads current `index.html` from GitHub
- Web-searches Redfin/Zillow for each active/contingent/pending property
- Detects price changes, status changes, off-market listings
- Updates L_DATA and appends change history to `notes` field
- Commits and pushes to GitHub if any changes found
- Notifies on completion

### `home-search-email-scan` — every 6 hours
- Searches Gmail for emails from Kevin Patel (kprealty19@gmail.com) and Redfin/Zillow
- Extracts new listing recommendations and price alerts
- Adds new qualifying properties to L_DATA (3+ beds, $700K–$1.3M, target neighborhoods)
- Commits and pushes to GitHub if any changes found
- Notifies on completion

Both tasks clone the repo, make edits, and push back — they do not modify Firebase.

---

## Search Context

| Field | Value |
|---|---|
| Buyer | Jugal Patel |
| Budget | $700K – $1.3M |
| Must-have | 3+ bedrooms |
| Target areas | West Town, West Loop, River West, Fulton Market, East Village, Wicker Park, Ukrainian Village |
| Commute anchor | Dyson HQ — 1330 W Fulton Market St, Chicago |
| Realtor | Kevin Patel (kprealty19@gmail.com) |
| Properties tracked | 31 (as of March 29, 2026) |

---

## Firebase Config

```js
const firebaseConfig = {
  apiKey: "AIzaSyAKzsJnzwF893IZiSsAKRyAG2bD-0rJwwY",
  authDomain: "home-tracker-4a4f2.firebaseapp.com",
  databaseURL: "https://home-tracker-4a4f2-default-rtdb.firebaseio.com",
  projectId: "home-tracker-4a4f2",
  storageBucket: "home-tracker-4a4f2.firebasestorage.app",
  messagingSenderId: "43212958169",
  appId: "1:43212958169:web:7a7d80c133ab89dae211c8"
};
```

Firebase is on the free Spark plan. No auth beyond the PIN gate. The repo is public but sensitive data (personal notes, scores) is in Firebase behind the PIN.

---

## Key Files for Claude Code

When starting a new Claude Code session on this project, read:
1. `CLAUDE.md` — full project context, architecture, line number map
2. `TODO.md` — current backlog and bug tracker
3. `README.md` — this file

The CLAUDE.md file has a line-number index of the key sections in `index.html` to help navigate the single-file architecture quickly.
