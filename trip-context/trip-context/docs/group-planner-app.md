# Group Planner App (Google Apps Script)

## What it is
An interactive day-planner widget built across several chat sessions, deployed
as a Google Apps Script web app and shared with the group via a link. Lets
anyone in the group drag/assign activities into a day-by-day planner without
touching code.

## Architecture
- **`Code.gs`** — server-side. Key functions: `doGet`, `getState`/`saveState`
  (current schema) — earlier iterations used `saveSlots`/`loadSlots` +
  `saveCustomActivities`/`loadCustomActivities`, since superseded.
- **`Index.html`** — full frontend, single file, DM Sans + DM Serif Display
  fonts, city-specific colour palettes.
- **State persistence**: `ScriptProperties` (shared across all users — chosen
  deliberately over `UserProperties` so the whole group sees the same plan).
  Redeploying the script does **not** erase saved state, since state lives in
  ScriptProperties independently of the deployed code/HTML.

## Data model (latest known schema)
- Three regional sections: Northern Japan, Southern Japan, Korea.
- Each section has a `rows` array (day rows) with **stable string IDs**
  (`r0`, `r1`, ...) so slot data is keyed by ID, not position — removing or
  reordering a row doesn't scramble other rows' data.
- Per-row city picker (click a row to change its city) — replaced an earlier
  single route-dropdown design.
- Add/remove day rows via UI (`×` to remove, hidden if only 1 row left; `＋ Add day`
  to append, defaulting to the last row's city).
- `ACTIVITIES` object keyed by city code → time of day (`morning`/`midday`/`evening`),
  feeding both an "Activity Bank" tab and a contextual picker filtered by each
  row's `data-cities`.
- Activity entries carry: name, duration (shown inline in slots, e.g. "· 2h"),
  description (shown in Activity Bank cards).
- "Add activity" feature lets anyone with the link add custom activities
  (with optional duration/description) without editing code — this solved the
  need for collaborative customization, since chat sessions themselves are
  account-specific and not shareable/collaborative.
- A Leaflet.js map per route: city markers with July average temps (colour-coded),
  animated route lines styled by transport mode (shinkansen/flight/ferry/KTX),
  travel-time labels per leg.

## Sections added over time (in order)
1. Initial Northern Japan route (Sendai → Hakodate → Sapporo, or skip Hakodate)
2. Korea section (default layout: Busan ×2, Jeju ×2, Seoul ×2)
3. Southern Japan (Kyoto/Osaka), activities pulled from the Drive master itinerary doc
4. Tokyo section (Jul 6–9, defaulting to 3 planning days from Jul 7), plus Tokyo day-trip options

## Schema migration note (historical)
When the row-based schema replaced the old flat `route` string, upgrading an
existing deployment with saved state required running `resetState()` once from
the Apps Script editor (Extensions → Apps Script → run `resetState`) before
redeploying — otherwise old-schema state would just get wiped by the migration
code in `loadStorage` anyway. This is resolved in the current schema but worth
knowing if a future schema change happens again.

## Deployment pattern
- **Don't** create a new deployment each time — it generates a new URL every time.
- **Do** edit the existing deployment (Deploy → Manage deployments → Edit →
  new version) to preserve the shared URL.
- Execute as: Me / Who has access: Anyone (so friends don't need a Google account).

## Pulling the live source for this repo
The chat history above is a *summary of iterations*, not a reliable diff of the
current file contents — the widget has been rebuilt/extended many times. To
seed a repo with the actual current code:
1. Open the Apps Script project at script.google.com (it lives under My Drive).
2. Copy `Code.gs` and `Index.html` content directly into this repo.
3. Treat this doc as the design-decisions/architecture record, not the source of truth for code.
