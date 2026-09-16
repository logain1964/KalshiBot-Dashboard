# J@rv1s Daily Briefing — 2026-09-15

## STATUS

Reviewed last night's Archie summary (EPL_GAME build, both open items
closed, the home/draw/away correction, the nightly_summary.md staleness
find) plus today's 11:20 AM KalshiBot alert email. Four real findings
from the alert review, on top of items carried forward from last
night.

## TOP PRIORITY — CLAIMS HAS ITS OWN UNFIXED "NAMED OUTCOME" BUG

Today's report shows GDP rows correctly displaying real labels (e.g.
"GDP > 4.0%") — the Sept 1 fix is holding there. But every CLAIMS row
still shows the literal placeholder: "Kalshi P(named outcome): 8c."
This means the Sept 1 fix only ever covered GDP's specific display
path — CLAIMS has its own separate, still-broken code path with the
identical bug. Same pattern this project has hit repeatedly (the
NFL_SPREAD label bug, the duplicate FLAGGED MARKET EDGES path) — a fix
that looked general but was actually narrow. Ask for tonight: was this
already known and just not yet scheduled, or is this a fresh find?
Either way, needs its own fix, not an assumption that the Sept 1 work
covered it.

## SECOND — MLS_GAME STILL ACTIVELY SIGNALING DESPITE "RETIRED" STATUS

Today's report shows MLS_GAME with 4 HIGH-confidence signals (BRE vs
CFC, NFO vs COV, POR vs ATL) plus 9 more at medium tier — actively
running, despite last night's summary stating it was "RETIRED from
further live development" (Sept 3 verdict). This may be entirely
intentional — retired from engineering investment but still running
track-only, same as any other suspended model — but that's an
assumption, not a confirmed fact, and the two readings have real,
different implications. Ask for tonight: explicit confirmation this is
working as designed (track-only, no further tuning) rather than
something that should have gone fully silent and didn't.

## THIRD — EPL_GAME MISSING FROM TODAY'S REPORT ENTIRELY

Despite being built, live-tested (7 real signals flagged against real
KXEPLGAME markets), and wired into the pipeline last night, EPL_GAME
doesn't appear anywhere in today's report — not in flagged signals,
not even in the static "MODELS EVALUATED" list at the bottom. Two real
possibilities: the "MODELS EVALUATED" list is a separate, likely
hardcoded list that just wasn't updated when EPL shipped (this project
has hit exactly this stale-list pattern before), or something is
actually preventing EPL from generating real signals in production.
Given last night's live test succeeded, my lean is "stale display
list," but this needs direct confirmation, not assumption — especially
since tonight's Leeds/Newcastle kickoff (19:00 UTC) is supposed to be
the first live test of EPL's SharpAPI-only blackout check with zero
ESPN fallback. Worth knowing the model is actually running in
production before that test matters.

## FOURTH — TREAT THE +58c CLAIMS EDGE WITH SUSPICION, NOT EXCITEMENT

"Claims>215K Sep17 -- BUY YES | Edge: +58.0c" (Kalshi 8c vs. model
66%) is an enormous edge, and this project has already explicitly
learned — as recently as last night, flagging EPL's own 15-22c edges
as "more likely a sign of model error than confirmed market
inefficiency" on an untested model — that outsized edges are usually a
red flag about the model or its inputs, not a real opportunity. CLAIMS
specifically has a rough recent history here too (91 days of silent
zero-output, only fixed Sept 3). Ask for tonight: check this edge
against real, current data before treating its size as exciting rather
than suspicious.

## CARRIED FORWARD FROM LAST NIGHT, UNCHANGED

- Decide `nightly_summary.md` vs. `nightly_summary_latest.md` as the
  one real target going forward. Worth clarifying first whether the
  automated GitHub fetch has actually been the operating channel this
  whole session, or whether Rus relaying the summary directly has been
  the real pattern — changes how urgent this fix actually was, though
  it's good to have regardless.
- Confirm tonight's Leeds vs. Newcastle kickoff held clean as the
  first live test of EPL's SharpAPI-only blackout check — no ESPN
  fallback exists for EPL, so this is a real, unforgiving first test.
- The stale "Checkpoint: September 8, 2026" text in the project's
  standing instructions — low priority cleanup, still open.
- Both EPL FORGE items (taper re-derivation, beats_random resolution)
  are genuinely closed — no further action needed there.

## VALIDATION TRACKER

Unchanged since Sept 8: Gate 1 project-wide suspended, real trigger is
GDP's Oct 30 resolution or MLB_GAME hitting n=15-20 spread-tagged
signals, whichever comes first. MLB_GAME n=109, 62.4%/0.2305 (narrow
Brier miss, only 4 real spread-tagged signals so far). MLS_GAME
retired from further development per Sept 3 verdict, still generating
track-only signals per today's report (see item 2 above). EPL_GAME:
built and shipped, real-world signal generation unconfirmed as of
today's report (see item 3). JOBS n=6, not validated. GDP has real
spread data, no resolved sample before Oct 30.

---
J@rv1s | Papa Ralph standard.
