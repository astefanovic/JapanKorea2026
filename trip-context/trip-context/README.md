# Japan & Korea Trip 2026 — Context Pack

Seed context for Claude Code, covering planning history that lives in past Claude
chats rather than in the shared Google Drive. Drive remains the source of truth
for bookings, confirmations, and the master itinerary doc — these files capture
the *deliberation* and *tooling* that produced/surrounds those decisions.

## Trip basics
- 4 travelers, ~17 nights: Melbourne → Tokyo (Jul 6) → ... → Incheon → Manila → Melbourne (Jul 23)
- Two travelers hold non-Australian passports with Korean mainland visa
  constraints — this shaped the whole Korea routing (see `docs/korea-visa-constraint.md`)
- Drive is the coordination hub: folders "01 Flights", "Korea - Accommodation",
  master itinerary doc (`1G2onruIS4j5cVUINpfaZVuXfkX1ad_DZVhwKPsKe1Z8`)

## Contents
- `docs/group-planner-app.md` — the Google Apps Script day-planner widget: architecture, decisions, current state, how to pull the live source
- `docs/seoul-airport-logistics.md` — the Jul 21–23 endgame: Busan→Seoul→Incheon overnight plan
- `docs/jeju-itinerary.md` — the Jeju Island two-day plan (Jul 18–20)
- `docs/korea-visa-constraint.md` — the constraint that shaped the whole Korea leg
- `docs/open-issues.md` — unresolved items flagged across past sessions

## How to use this with Claude Code
These are point-in-time summaries reconstructed from past chat sessions, **not**
live data. Before trusting a detail (a flight ref, a price, a booking status),
cross-check against Drive. Where a doc references the Apps Script project, pull
the live `Code.gs`/`Index.html` directly from script.google.com rather than
reconstructing from chat history — the widget has been iterated on many times
and the chat history is not a reliable diff.
