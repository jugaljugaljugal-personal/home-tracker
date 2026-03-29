# Home Search Tracker — Backlog & TODO

Last updated: March 29, 2026 — Batch 2 complete

> **How to use this file:**
> When an item is implemented, move it to the ✅ Completed section at the bottom and update the "Last updated" date above.
> When Claude Code completes work, it should update this file and push it to GitHub along with index.html.

---

## 🔴 Bug Fixes (should fix soon)

### B1. Filters and sorting don't apply to mobile cards
**Status:** ✅ Completed (Batch 1)
**Detail:** `applyFilters()` and `sort()` only affect `#tb tbody tr` rows. On mobile the table is hidden and cards are shown, but card visibility and order are never updated when filters or sort are changed. Mobile users effectively can't filter or sort.
**Fix:** In `applyFilters()`, also show/hide `.m-card` elements based on the same criteria. In `sort()`, also reorder `.m-card` elements inside `#cards-wrap`.

### B2. Tour modal address-parsing bug
**Status:** ✅ Completed (Batch 1)
**Detail:** The `toggleCheck()` function may call `parseInt()` on an address string to get the property ID (returns NaN). If confirmed, checklist saves silently fail.
**Fix:** Store the property `n` in a `data-n` attribute on the modal root or a hidden element, and read from that instead.

### B3. `clearScore()` has no confirmation dialog
**Status:** ✅ Already fixed — `clearScore()` has `if (!confirm(...)) return;` at line 1133
**Detail:** `clearChecklist()` shows a `confirm()` before wiping data. `clearScore()` does not — one tap immediately wipes all scoring data for a property.
**Fix:** Add `if (!confirm('Clear all scores for this property?')) return;` to `clearScore()`.

---

## 🟡 Quick Wins

### Q1. Search / text filter bar
**Status:** ✅ Completed (Batch 1)
**Detail:** No way to search properties by address, neighborhood, or notes. With 31+ properties, scrolling to find one is tedious.
**Implementation:** Add a text `<input>` to the filter bar. On `input` event, filter table rows and mobile cards where `addr`, `hood`, or `notes` contains the search string (case-insensitive).

### Q2. Save feedback toast
**Status:** ✅ Completed (Batch 1)
**Detail:** Notes, stars, scores, and dates save silently. There's no visual confirmation that a save succeeded, and Firebase errors are only logged to the console.
**Implementation:** Add a small toast element (fixed bottom-right). Show "Saved ✓" for 1.5s on successful Firebase writes. Show "Save failed ✗" in red on error.

### Q3. Empty state message when filters match nothing
**Status:** ✅ Completed (Batch 1)
**Detail:** If filters exclude all properties, the table and card list go completely blank with no explanation.
**Implementation:** After `applyFilters()`, check if zero rows/cards are visible. If so, show a "No properties match your filters" message inside the table body and cards container.

### Q4. Escape key closes modals
**Status:** ✅ Completed (Batch 1)
**Detail:** Modals (tour, score) can only be closed via the ✕ button or clicking the overlay. No keyboard shortcut.
**Implementation:** Add a `keydown` listener on `document` for `Escape` that calls the active modal's close function.

### Q5. Sort controls on mobile
**Status:** ✅ Completed (Batch 1)
**Detail:** The sortable column headers only exist in the desktop table. Mobile users have no way to sort cards by price, rank, score, or walk time.
**Implementation:** Add a compact sort bar above the mobile cards (shown only on mobile) with a dropdown or pill buttons: Price ↑↓, Walk, Rank, Score.

### Q6. "Last updated" timestamp in header
**Status:** ✅ Completed (Batch 1) — app listens to Firebase `/config/lastUpdated`; scheduled tasks still need updating to write this value (see task prompt updates)
**Detail:** No indication of when the scheduled tasks last ran or when data was last refreshed.
**Implementation:** Scheduled tasks write current timestamp to Firebase at `/config/lastUpdated` after each run. App reads this on load and displays "Last updated: 2h ago" in the header stats bar.

---

## 🟢 High-Value Features

### F1. "Toured" toggle per property
**Status:** ✅ Completed (Batch 2)
**Detail:** No way to distinguish properties you've physically visited from ones you've only researched online.
**Implementation:**
- Add a `toured` boolean per property in Firebase at `/tracker/toured/{n}`
- Show a 🏠 toggle button on each table row and mobile card
- Add a "Toured" filter option in the filter bar
- Stats bar shows "X Toured" count

### F2. Price history tracking
**Status:** ✅ Completed (Batch 2) — display side complete; scheduled tasks write `/tracker/priceHistory/{n}/{timestamp}` → `{ oldPrice, newPrice }` when changes are detected
**Detail:** When a price changes, you lose the old price. Would be valuable to see "was $950K → $875K" without digging through notes.
**Implementation:**
- Scheduled status-check task writes to Firebase: `/tracker/priceHistory/{n}/{timestamp}` → `{ old: 950000, new: 875000 }`
- App reads price history and shows a small badge on listings with changes: "📉 was $950K"
- Clicking the badge shows full price history timeline

### F3. Email notifications on price drops / status changes
**Status:** Open
**Detail:** Currently you only find out about changes when you open the app. Changes happen every 8 hours.
**Implementation:**
- After the status-check task detects changes, send a summary email to jugaljugaljugal@gmail.com via Gmail MCP
- Email format: subject "Home Tracker: 2 changes detected 3/29", body lists each change with property address, what changed, Redfin link

### F4. Property comparison view
**Status:** Open
**Detail:** Hard to compare top candidates side-by-side. Currently requires switching between cards/rows.
**Implementation:**
- Add a checkbox to each property row and mobile card
- "Compare (X)" button appears when 2–3 are selected
- Opens a full-screen modal with a side-by-side grid showing all stats, scores, walk time, notes, links
- X-out removes a property from comparison

### F5. Kevin's notes as editable Firebase field
**Status:** Open
**Detail:** Kevin's commentary is currently either baked into L_DATA `notes` (requires a code push to update) or in the static Kevin section (disconnected from individual properties).
**Implementation:**
- Add `/tracker/kevinNotes/{n}` to Firebase
- Show a "Kevin" field on each property card/row, editable from the UI (separate from Jugal's personal notes)
- Kevin section at the bottom can pull from Firebase instead of being hardcoded
- No code deploy needed to update Kevin's input

### F6. Archive / hide sold and off-market properties
**Status:** ✅ Completed (Batch 2)
**Detail:** Properties with `st: SOLD` or off-market status clutter the active list but shouldn't be deleted (preserves history).
**Implementation:**
- Add "Hide archived" toggle (default ON) to the filter bar
- Properties with `st: SOLD` and `cr: NO` are hidden when toggle is ON
- Toggle OFF to show full history

---

## 🔵 Polish & Quality of Life

### P1. Dark mode
**Status:** Open
**Detail:** Often checked at night on mobile. No dark mode support.
**Implementation:** Add CSS variables for dark theme. Toggle via button in header (moon/sun icon). Persist preference in `localStorage`. Also respect `prefers-color-scheme: dark` media query.

### P2. Score weight auto-normalize
**Status:** Open
**Detail:** The score modal shows a red warning if category weights don't sum to 100 but still allows saving, which produces incorrect weighted scores.
**Implementation:** Add an "Auto-normalize" button that proportionally adjusts weights to sum to 100. Or: auto-normalize silently on save.

### P3. Neighborhood grouping view
**Status:** Open
**Detail:** No way to quickly see all West Town properties vs. all West Loop properties together.
**Implementation:** Add a "Group by neighborhood" toggle above the table/cards. When active, insert neighborhood header rows between groups of properties sorted by `hood`.

### P4. Export to PDF / shareable summary
**Status:** Open
**Detail:** For discussions with Kevin or Janki, a clean one-pager of top-ranked properties would be useful.
**Implementation:** "Export" button generates a print-optimized or PDF view of the top-starred/scored properties with address, price, score, notes, and Redfin link.

### P5. Accessibility improvements
**Status:** Open
**Detail:** Several accessibility gaps identified in code review.
**Specific items:**
- Add `role="dialog"` and `aria-modal="true"` to tour and score modals
- Add `aria-label` to star rating buttons
- Add `aria-expanded` to filter dropdowns
- Ensure Escape key closes modals (see Q4)
- Check color contrast on yellow/green badges

### P6. Rotate GitHub PAT
**Status:** Open
**Detail:** The GitHub Personal Access Token used by the scheduled tasks was exposed in chat and should be considered compromised.
**Steps:**
1. Go to https://github.com/settings/tokens
2. Revoke the current token
3. Generate a new token with `repo` scope, 1-year expiry
4. Update both scheduled tasks (`home-search-status-check`, `home-search-email-scan`) via the Cowork sidebar with the new token in the clone URL

---

## 🔵 DevOps / Infrastructure

### D1. GitHub Actions validation on push
**Status:** Open
**Detail:** No automated check that L_DATA is valid JavaScript before deploying.
**Implementation:**
```yaml
# .github/workflows/validate.yml
name: Validate
on: [push]
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Validate L_DATA
        run: |
          node -e "eval(require('fs').readFileSync('index.html','utf8').match(/const L_DATA = \[[\s\S]*?\];/)[0])"
          echo "L_DATA OK"
```

---

## ✅ Completed

- [x] Built full Firebase-backed single-file web app (index.html)
- [x] Replaced localStorage with Firebase Realtime Database
- [x] PIN gate with SHA-256 hashing (PIN: 1936)
- [x] All 31 properties in L_DATA with lat/lng, Redfin/Zillow links, notes
- [x] Property #12 beds corrected to 4
- [x] Mobile card view + responsive CSS
- [x] Star ratings, weighted scoring modal, tour checklist modal
- [x] Map view (Leaflet) with color-coded markers
- [x] Created GitHub repo `jugaljugaljugal-personal/home-tracker`
- [x] Migrated localStorage data (notes, star ratings) to Firebase via migrate.html
- [x] CLAUDE.md written for Claude Code context
- [x] index.html, CLAUDE.md, TODO.md pushed to GitHub repo — app live at https://jugaljugaljugal-personal.github.io/home-tracker/
- [x] GitHub Pages confirmed serving from main branch root
- [x] Redfin links added for 20 of 31 properties (3 not indexed on Redfin individually; 8 already had links)
- [x] Redfin/Zillow link buttons added to desktop table rows and mobile cards
- [x] Scheduled tasks created and updated for GitHub-based push workflow:
  - `home-search-status-check` — every 8 hours
  - `home-search-email-scan` — every 6 hours
