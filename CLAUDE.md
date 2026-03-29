# Home Search Tracker — Project Context

This file gives Claude Code (or any future Claude session) full context to pick up this project without re-explanation.

## What This Is

A personal home search tracker for Jugal's active Chicago home search. Single-file web app (`index.html`) hosted on GitHub Pages, with Firebase Realtime Database for cross-device sync. Mobile-optimized, PIN-gated.

**Live URL:** https://jugaljugaljugal-personal.github.io/home-tracker/
**GitHub repo:** `jugaljugaljugal-personal/home-tracker` (public, main branch)
**PIN:** 1936

---

## Repository Setup

```bash
# Clone
git clone https://github.com/jugaljugaljugal-personal/home-tracker.git
cd home-tracker

# The entire app is one file
ls  # → index.html
```

GitHub Pages is configured to serve from the `main` branch root. Every push to `main` auto-deploys (usually within 60 seconds).

---

## File Structure

```
home-tracker/
  index.html       # Entire app — 1445 lines
  CLAUDE.md        # This file (copy here too after first push)
  TODO.md          # Backlog
```

In the local workspace folder:
```
House hunt/
  index.html       # Source of truth (push this to repo)
  CLAUDE.md        # This file
  TODO.md          # Backlog
  migrate.html     # One-time migration helper (no longer needed)
  Migration_Prompt_GitHub_Pages_Firebase.md  # Original migration spec
```

---

## Firebase

**Project:** `home-tracker-4a4f2`
**DB URL:** https://home-tracker-4a4f2-default-rtdb.firebaseio.com
**SDK:** Firebase v8 compat (loaded via CDN in index.html)

### Config (embedded in index.html ~line 379)
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

### Firebase Data Structure
```
/config/pinHash        → SHA-256 hash of PIN (1936)
/tracker/ranks/        → { "1": 4, "2": 3, ... }  (1–5 stars per property)
/tracker/notes/        → { "1": "...", "2": "...", ... }
/tracker/scores/       → { "1": { location:80, layout:70, ... }, ... }
/tracker/checklists/   → { "1": { "item-key": true, ... }, ... }
/tracker/listed/       → { "1": "2026-01-15", ... }  (listing dates for DOM calc)
/tracker/weights/      → { location:30, layout:30, condition:20, financial:20 }
```

### PIN
- **Value:** 1936
- **Hash (SHA-256):** `3f46bdea034f311a14efe877f5592d84a7a6c97d9b917be3f55573311e6cdda7`
- **Stored at:** `/config/pinHash` in Firebase
- **Session:** `sessionStorage.setItem('hst_authed', 'true')` (clears on tab close)

---

## index.html Architecture

### Key sections by line number
| Line | Section |
|------|---------|
| 1–11 | HTML head, Leaflet + Firebase CDN imports |
| 12–375 | All CSS (desktop + mobile responsive) |
| 379–393 | Firebase init |
| 395–585 | `L_DATA` array — 31 property objects |
| 586–660 | In-memory cache + data read/write functions |
| 661–900 | Render functions (table rows, mobile cards) |
| 901–1100 | Filter/sort/search logic |
| 1101–1200 | Map (Leaflet) init + markers |
| 1201–1269 | Tour checklist modal (~90 items, 11 sections) |
| 1270–1380 | `setupFirebaseListeners()` — real-time onValue listeners |
| 1381–1402 | PIN verification + `showTracker()` |
| 1403–1445 | App init, PIN gate DOM, startup logic |

### Data layer pattern
```js
const _cache = { ranks:{}, notes:{}, scores:{}, checklists:{}, listed:{}, weights:null };

// All saves: update cache + push to Firebase
function saveRank(n, v)  { _cache.ranks[n] = v;  db.ref('tracker/ranks/' + n).set(v); }
function loadRank(n)     { return _cache.ranks[n] || 0; }
// Same pattern: saveNote/loadNote, saveScore/loadScore,
//               saveChecklist/loadChecklist, saveListed/loadListed, saveWeights/loadWeights
```

### L_DATA property object shape
```js
{
  n: 1,                          // property number (used as Firebase key)
  addr: "123 N Example St #4W",
  price: 875,                    // in $K
  beds: 3,
  baths: 2,
  sqft: 1800,
  hoa: 350,                      // monthly HOA; 0 if none
  status: "Active",              // Active | Contingent | Under Contract | Sold
  lat: 41.8900,
  lng: -87.6700,
  redfin: "https://www.redfin.com/...",
  zillow: "https://www.zillow.com/...",
  notes: "Kevin: Interested. JP: Great location..."
}
```

**Important:** Property #12 has `beds: 4` (not 3 — was a bug in an earlier draft, already fixed).

---

## Adding/Updating Properties

Edit the `L_DATA` array in `index.html` directly. Properties are numbered sequentially (`n: 1` through `n: 31`). Add new ones at the end with the next number.

To add a property:
1. Get lat/lng from Google Maps (right-click → copy coordinates)
2. Add entry to `L_DATA` in `index.html`
3. Commit and push → GitHub Pages auto-deploys

---

## Search Context

| Field | Value |
|-------|-------|
| Buyer | Jugal Patel |
| Email | jugaljugaljugal@gmail.com |
| Realtor | Kevin Patel (kprealty19@gmail.com) |
| Budget | $700K – $1.3M |
| Must-have | 3+ bedrooms |
| Target areas | West Town, West Loop, River West, Fulton Market, East Village, Wicker Park, Ukrainian Village |
| Commute anchor | Dyson HQ — 1330 W Fulton Market St, Chicago |
| Properties tracked | 31 (as of March 29, 2026) |

---

## Automated Tasks (Cowork Scheduled Tasks)

Two tasks run in Cowork and update the tracker by editing `index.html` in the repo:

| Task name | Schedule | What it does |
|-----------|----------|--------------|
| `home-search-email-scan` | Every 6 hours | Scans Gmail for new listing emails from Kevin Patel + Redfin |
| `home-search-status-check` | Every 8 hours | Checks Redfin/Zillow for price drops, status changes (contingent/pending/sold) |

Both tasks: clone repo → edit L_DATA → commit → push to main.

---

## Current Status (as of March 29, 2026)

- ✅ Firebase project created, config embedded in index.html
- ✅ All 31 properties in L_DATA with Redfin/Zillow links, coordinates, notes
- ✅ Firebase data layer complete (replaces localStorage)
- ✅ PIN gate working (PIN: 1936)
- ✅ User data (notes, star rankings) migrated from localStorage to Firebase via migrate.html
- ✅ `jugaljugaljugal-personal/home-tracker` GitHub repo created
- ⏳ **index.html not yet pushed to GitHub repo** — needs to be uploaded to go live
- ⏳ GitHub Pages not yet confirmed live
- ⏳ DevOps pipeline not set up (see TODO.md)
- ⏳ Scheduled tasks not yet updated for git push workflow

---

## Known Issues / Watch-outs

- **Cowork sandbox** blocks `api.github.com` and `firebaseio.com` via proxy — use Claude Code for git operations
- The original iCloud file (`Home_Search_Tracker.html`) is deprecated; don't edit it
- `migrate.html` was a one-time tool (localStorage → Firebase) — no longer needed but kept for reference
- Firebase free Spark plan — no auth beyond PIN gate; keep repo public is fine (data is in Firebase, not HTML)

---

## Quick Start for Claude Code

```bash
# 1. Clone the repo
git clone https://github.com/jugaljugaljugal-personal/home-tracker.git
cd home-tracker

# 2. Copy the latest index.html from iCloud if not yet pushed
# (check if index.html is already present and up to date)

# 3. Make edits to index.html

# 4. Push to deploy
git add index.html
git commit -m "Update: [describe change]"
git push origin main
# GitHub Pages deploys automatically in ~60 seconds
```

See `TODO.md` for the full backlog, including setting up a proper local dev workflow.
