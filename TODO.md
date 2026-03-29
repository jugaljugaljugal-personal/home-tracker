# Home Search Tracker — Backlog & TODO

Last updated: March 29, 2026

---

## 🔴 Immediate / Blocking

### 1. Push index.html to GitHub (DEPLOY THE APP)
The app is built and ready but not yet live because `index.html` hasn't been pushed to the repo.

**Option A — GitHub web UI (no setup needed):**
1. Go to https://github.com/jugaljugaljugal-personal/home-tracker
2. Click "Add file" → "Upload files"
3. Drag in `index.html` from the House hunt folder
4. Click "Commit changes"
5. Wait ~60 seconds, then verify at https://jugaljugaljugal-personal.github.io/home-tracker/

**Option B — Claude Code (preferred for ongoing work):**
```bash
git clone https://github.com/jugaljugaljugal-personal/home-tracker.git
cd home-tracker
cp "[path to House hunt]/index.html" .
git add index.html CLAUDE.md TODO.md
git commit -m "Initial deploy: Firebase-backed home search tracker"
git push origin main
```

### 2. Verify GitHub Pages is enabled
- Go to repo Settings → Pages
- Source: Deploy from branch → `main` → `/ (root)`
- Confirm URL shows as active

### 3. End-to-end smoke test
- Open https://jugaljugaljugal-personal.github.io/home-tracker/
- Enter PIN 1936
- Confirm: all 31 properties load, stars show correctly, notes show, map renders
- Test on mobile (iPhone) — confirm mobile card view works
- Edit a star rating → open on another device → confirm it syncs

---

## 🟡 DevOps Pipeline

### 4. Set up local git workflow for fast iteration
Goal: edit `index.html` locally → push → live in 60 seconds.

```bash
# One-time setup
git clone https://github.com/jugaljugaljugal-personal/home-tracker.git ~/home-tracker
cd ~/home-tracker

# Daily workflow
# Edit index.html (or let Claude Code do it)
git add index.html
git commit -m "Fix: [describe]"
git push origin main
```

**With Claude Code**, from inside `~/home-tracker`:
```
claude
> Update property #5 price to $950K and mark it Contingent
```
Claude Code can edit the file and push directly.

### 5. Set up GitHub Personal Access Token (for automated tasks)
The Cowork scheduled tasks need to push to GitHub. They currently can't because the sandbox proxy blocks `api.github.com`.

**Steps:**
1. Go to https://github.com/settings/tokens → "Generate new token (classic)"
2. Scopes: `repo` (full control of private repositories)
3. Set expiration to 1 year
4. Save the token securely (1Password, etc.)
5. Update the Cowork scheduled tasks to use:
   ```
   git remote set-url origin https://[TOKEN]@github.com/jugaljugaljugal-personal/home-tracker.git
   ```
   Or configure as a repo secret if using GitHub Actions.

### 6. Add GitHub Actions CI/CD (optional but nice)
For validation before deploy:

```yaml
# .github/workflows/validate.yml
name: Validate HTML
on: [push]
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Check HTML validity
        run: |
          # Ensure L_DATA is valid JS, no syntax errors
          node -e "eval(require('fs').readFileSync('index.html','utf8').match(/const L_DATA = \[[\s\S]*?\];/)[0])"
          echo "L_DATA validates OK"
```

### 7. Update Cowork scheduled tasks for git push workflow
Both tasks (`home-search-email-scan`, `home-search-status-check`) need updating:
- Current approach: edit the local iCloud HTML file (now deprecated)
- New approach: clone repo → edit L_DATA in index.html → commit → push
- Requires GitHub PAT (see task #5 above)
- Ask Claude Code or Cowork to update the task definitions once PAT is set up

---

## 🟢 Feature Improvements

### 8. Add "Last Updated" timestamp to header
Show when the data was last refreshed by the scheduled tasks. Store update timestamp in Firebase at `/config/lastUpdated`.

### 9. Price history tracking
When a price change is detected by the status-check task, append to a history array in Firebase:
```
/tracker/priceHistory/[n]/[timestamp] → { old: 950, new: 875 }
```
Display as a small "📉 was $950K" badge on the listing.

### 10. Email/push notification on price drops or status changes
When the status-check task detects a change, send an email summary to jugaljugaljugal@gmail.com (via Gmail MCP).

### 11. "Toured" flag per property
Add a toggle per property (saved in Firebase) to mark properties that have been physically toured. Use a 🏠 icon. Filter by toured/not-toured.

### 12. Kevin's latest notes sync
Allow a quick way to add/update Kevin's notes from mobile (currently requires editing L_DATA and pushing). Consider a dedicated "Kevin notes" field in Firebase (separate from user notes in `/tracker/notes/`) that doesn't require a code deploy.

### 13. Comparison view
Select 2–3 properties and show them side-by-side with all stats, scores, and notes.

### 14. Export to PDF / share summary
Generate a shareable PDF summary of top-ranked properties (for discussion with Kevin or Janki).

---

## 🔵 Maintenance

### 15. Archive/hide sold/withdrawn properties
Add a toggle to hide properties that are Sold or Off Market, rather than deleting them from L_DATA (preserves history).

### 16. Migrate CLAUDE.md and TODO.md into the GitHub repo
Once index.html is pushed, also push CLAUDE.md and TODO.md so Claude Code can read them when working from the cloned repo.

```bash
git add CLAUDE.md TODO.md
git commit -m "Add project docs for Claude Code"
git push origin main
```

---

## ✅ Completed

- [x] Built full Firebase-backed single-file web app (index.html, 1445 lines)
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
