# Agent Notes — decisions, conventions & gotchas

Read this before making changes. It records choices the traveller has already made,
how the interactive site is built, and the traps that cost real time so you don't
repeat them. (Companion to `CLAUDE.md`.)

---

## 0. Workflow reminders

- **Push directly to `master`. No PRs.** (Also in CLAUDE.md.)
- Three docs, three altitudes — keep each at its own level:
  - `itinerary.md` — glanceable, whole-leg overview.
  - `daily-schedule.md` — hour-by-hour operational plan (the "what do we do this hour" doc).
  - `docs/index.html` — the interactive map/timeline site (GitHub Pages).
- ⚠️ Known un-reconciled: `itinerary.md` Day 6 still shows the old "Tokyo TBD" options
  table even though Kamakura was chosen. Don't treat that stale table as the decision.

---

## 1. Trip decisions already made (don't re-litigate)

### Schedule
- **Wake times: 09:00 on every movable day.** Only hard-anchored days start earlier and
  must stay: flights (Days 1, 13, 16, 19), Ghibli timed entry (Day 4, 08:00),
  DisneySea & USJ rope-drops (Day 3 R&A 07:00, Day 9 07:30), and **Day 8 Kyoto stays
  06:30** because it's anchored by Fushimi-Inari-at-dawn + the kimono appointment.
  (There's an inline crowd-tradeoff note on Day 8/Day 10 — leave them.)
- **Day 3 (Aleks & Ceci)**: options kept open for the group to browse. Nail appt at
  **Sin Den** (Omotesando, English-run). Default plan = Harajuku + Shibuya.
- **Days 5 & 6**: treated as ONE interchangeable pool of full-day Tokyo options —
  "pick any two." Currently pencilled: Yokohama+Nakano (Fri 10th) and Kamakura (Sat 11th).
- **Day 10 (Osaka free day)**: options kept open. Default = Arashiyama.
- **Omoide Yokocho** is the whole-group dinner on **11 Jul** (last Tokyo night). On
  **7 Jul** Aleks & Ceci do robatayaki *or* sushi omakase instead (their choice).

### People / passports (verified with the traveller)
- Travellers: **Aleks** & **Ralph** (Australian, full trip), **Angela** (Filipino
  passport), **Ceci** (Vietnamese passport). Angela & Ceci split off at Jeju (20 Jul) → Manila.
- **Angela & Ceci already HAVE their Japan tourist visas** (confirmed). Do not re-flag
  Japan entry as a blocker.
- Jeju entry (18 Jul) is covered by Jeju's visa-waiver (direct CJU arrival) for both.
- Still open/unverified (lower priority): Ceci's Hong Kong transit (must stay **airside**
  during the 20 Jul HKG layover) and Ceci's Philippines entry rules.

### Food preferences (drove the map's food feature)
- Breakfast suggestions must **not** be konbini — use highly-rated cafes / onigiri
  shops near the accommodation or on the way.
- Each food pin should offer **2–3 browsable picks** that are region/suburb-specific,
  unique, or highly-rated (the traveller called out the Nagoya hitsumabushi, Kamakura
  shirasu-don, and Nakamise-dori snacks as the *kind* of suggestion they want).

---

## 2. The interactive site (`docs/index.html`)

Single self-contained file. Data-driven: a `DAYS[]` array + a `P` dict of coordinates,
rendered into a scrollytelling timeline with a sticky Leaflet map.

### Hosting
- **GitHub Pages, served from `/docs` on `master`.** Live URL:
  `https://astefanovic.github.io/JapanKorea2026/`
- We chose Pages over Vercel deliberately (repo's already on GitHub, no extra account).

### Key libraries / data sources
- **Leaflet is VENDORED locally** at `docs/vendor/leaflet/` (leaflet.js, leaflet.css,
  images/). **Do NOT switch back to a unpkg/CDN `<script>` tag** — see gotcha #3.
- Map tiles: CARTO dark (`basemaps.cartocdn.com`). External, loads in real browsers.
- Destination/dish **photos are fetched at runtime from the Wikipedia REST API**
  (`en.wikipedia.org/api/rest_v1/page/summary/<title>`) — CORS-enabled, works in real
  browsers. Only needs an article *title* (redirects are followed); a wrong/missing
  title just yields no photo (graceful).

### Data model / helpers (in the inline `<script>`)
- `m(coord,label,time,wiki,branch)` — a normal numbered stop. `wiki` = Wikipedia title for
  its photo. `branch` is optional (see "Split days" below).
- `acc(coord,label,branch)` — accommodation. Renders a **green 🏠 pin**, no photo. Should be
  the **first** marker of each day.
- `food(coord,meal,picks,branch)` — a **red 🍴 pin**. `picks` is an array of
  `{name, dish, note}`; `dish` is a Wikipedia title for that pick's thumbnail. The popup
  is a compact list of the picks (this is the "2+ options to browse" UI — keep it inside
  the popup; do NOT add it to the timeline/screen).
- Marker pin colours (also in the page's footer legend): 🟢 green = accommodation,
  🔴 red = food, 🟠 orange = sight, 🔵 blue = an option being previewed.
- **Food pins show a 🍴 emoji instead of a number**; numbering (`num`) only increments for
  non-food markers so the numbered stops stay sequential.

### Option days (Day 3, Days 5–6 pool, Day 10)
- These use `base:[...]` + `options:[...]` instead of a plain `markers:[...]`.
- `base` = the always-on pins (accommodation + breakfast, plus any fixed stop like
  DisneySea on Day 3). `activateOption()` renders `[...base, ...option.markers]`.
- On scroll-in, an option day shows **exactly one option** (the `chosen:true` one, else
  the first) — NOT all options pooled. This was an explicit fix; don't revert it.
- ⚠️ Section index ≠ day number: Days 5 & 6 are a single combined card, so everything
  after it is shifted by one (e.g. `n:10` Osaka is DOM section idx 8). When testing by
  index, account for this.

### Split days (the group forks — currently only Day 16)
- Pass a `branch` string (`"a"` or `"b"`) as the last arg to `m()`/`acc()`/`food()` for
  every marker *after* the fork point. Markers with no `branch` are the shared "trunk".
- `showMarkers()` draws the trunk as the usual dashed orange line, then draws one extra
  dashed polyline per branch, starting from the last trunk point, in that branch's colour
  (`BRANCH_COLOR = {a:'#4aa3e8' (blue), b:'#e8bf4a' (gold)}` — same hex as the existing
  `alt`/`gold` marker icon classes, so pins and lines match). Non-food branch markers also
  get the matching icon colour; `home`/`food` styling still takes priority over branch
  colour for accommodation/food pins.
- This assumes markers are ordered trunk-then-branch-then-branch in the array (true for
  Day 16). If a future split day interleaves them, the "last trunk point" anchor logic in
  `showMarkers()` needs revisiting.

### Desktop card focus (dim + highlight the active day)
- On screens ≥861px, `section.day` sits at `opacity:.5` by default; the IntersectionObserver
  adds an `active` class to whichever day is currently driving the map (same intersection
  logic that already picks which day's markers to show), which bumps it to `opacity:1` with
  a blue (`--accent2`) border/box-shadow. This was added because with many similarly-styled
  cards in a row it was hard to tell which one you were "on" while scrolling.
- Desktop cards also got `scroll-snap-align:center` + `html{scroll-snap-type:y proximity}`
  for a light snap-to-card feel. `proximity` (not `mandatory`) so it doesn't fight normal
  scrolling or trap the user on a card. Mobile layout is untouched.

---

## 3. GOTCHAS / mistakes already made (the expensive ones)

### 3.1 The agent sandbox blocks most of the web
The execution sandbox's egress proxy **only allows package registries** (npm, pypi,
crates, etc.). It **BLOCKS**: `github.io`, `unpkg.com`, `basemaps.cartocdn.com`, and
`en.wikipedia.org`. Consequences:
- You **cannot `curl` the live site** to verify it (returns `000`/403). Don't conclude
  the site is broken from that.
- In a test browser, **map tiles render black and Wikipedia photos don't load** — this
  is the environment, **NOT a bug**. The code is written to degrade gracefully (photos
  fall back to a 🍴 glyph / the box removes itself). Do not "fix" it or chase it.
- You **cannot fetch images to self-host** them here. `npm` works, so Leaflet was
  vendored via `npm pack leaflet@1.9.4`.
- **Testing must be local:** `cd docs && python3 -m http.server PORT`, load
  `http://localhost:PORT/` in headless Chromium. Leaflet is vendored so it loads; verify
  DOM/behaviour/graceful-degradation, and **ignore console errors that are cartocdn or
  en.wikipedia.org failures** (flag any OTHER error).

### 3.2 GitHub Pages deploy is flaky — always verify the deploy
- A push **sometimes does not trigger a Pages build at all** (silent no-deploy). After
  every push, **verify a `pages build and deployment` run exists for your commit SHA and
  its `conclusion == success`.** If there's no run, push an empty commit to force one:
  `git commit --allow-empty -m "Redeploy" && git push`.
- Enabling Pages + the first push can **collide** and leave a "ghost" in-progress
  deployment that **cannot be cancelled via the API** (returns 409 "not yet queued" /
  403 "already running"). It blocks new deploys until GitHub garbage-collects it
  (~15–30 min) or the user toggles Settings → Pages source. If stuck, the reliable
  user-side fix is that Settings→Pages source toggle; don't burn 30 min retrying the API.

### 3.3 Do NOT put SRI `integrity=` on external Leaflet tags
A wrong Subresource-Integrity hash **silently blocks** the script/stylesheet, so the map
vanishes while the rest of the page renders (looks like "map missing"). This bit us. We
removed SRI and then vendored Leaflet entirely to kill the whole class. Keep it vendored.

### 3.4 `mcp__github__actions_list` output is huge
The response often exceeds the tool-result token limit and gets saved to a file. Parse it
with `python3 -c "import json; ..."` (read `head_sha`, `status`, `conclusion`) instead of
reading it raw.

### 3.5 Map can render at 0-size
Sticky/flex layout can init Leaflet before layout settles → blank map. The page calls
`map.invalidateSize()` on a timer and on `load`; keep that.

### 3.6 Coordinates for neighbourhood sub-spots are approximate
For the niche-neighbourhood options (Shimokitazawa, Yanaka, etc.) and some food spots,
coords are in-neighbourhood approximations, and labels are honest area/category names
("vintage strip", "record shops") rather than exact verified venues. Don't present them
as precise addresses.

---

## 4. How we test changes (do this every round)

1. Make the change; **syntax-check the inline JS** before committing, e.g. extract the
   last `<script>` block and `new Function(code)` in node; also validate every `P.xxx`
   reference resolves to a key.
2. Commit + push to `master`.
3. **Verify the Pages deploy** for your SHA succeeded (see 3.2).
4. **Run an isolated adversarial browser test** (the traveller specifically wants Sonnet):
   spawn an agent that serves `docs/` locally and drives headless Chromium
   (`/opt/pw-browsers/chromium`; do NOT run `playwright install`), checks the specific
   change + regressions, and **ignores only cartocdn/wikipedia network failures**.
5. Report honestly: the sandbox can't confirm live tiles/photos render — only the user
   can, in a real browser. Say so; don't claim the photos were verified.
