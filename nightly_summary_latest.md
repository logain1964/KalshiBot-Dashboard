# Archie Nightly Summary -- 2026-08-20 (evening session)
For J@rv1s -- Rus will paste this in the morning.

## ORACLE CLOUD STATUS
Healthy. Real 9PM production run (2026-08-20) confirmed both of
tonight's MLB fixes live and correct: "Games today: 9 total," Stats
API fetch working from Oracle where ESPN is now Akamai-blocked.
Tonight's second commit (NFL Elo + SharpAPI pagination fix, a611edf)
was pulled and independently verified on Oracle after the fact, but
has NOT yet been exercised by a real scheduled cron run -- the next
2AM or 9PM CT run is the first real confirmation of both in production.

## VALIDATION TRACKER / MODEL STATUS
- JOBS: flagged tonight as the most concerning open item. Zero signals
  of any kind since June 17 -- over two months. This is the one model
  the project's own instructions call VALIDATED and trustworthy
  (67%+, 54+ signals, Brier 0.1351, beats_random TRUE); that status is
  currently resting on two-month-old data. Root cause NOT chased yet.
- GDP: still generating raw signals daily but nothing has been SCORED
  since July 30 (679 blank actual_outcome rows in signals_log.csv as
  of tonight). Possibly related to the JOBS gap, not confirmed.
- MLB_GAME: suspended (unchanged), but its data pipeline got a real
  fix tonight -- see below.
- Aug 25 Gate 1 stress-test checkpoint: 4 days out at session close.
  Not checked tonight for real forward-spread-data volume.
- Aug 30 MLB Track A/B checkpoint: unchanged.
- NFL: not yet live-trading (still suspended pending validation, per
  design), but its Elo mechanism got a real structural fix tonight --
  see below.

## TONIGHT'S WORK -- WHAT SHIPPED

**MLB ESPN block (confirmed, fixed, verified in real production):**
ESPN's scoreboard endpoint is Akamai-blocked from Oracle's IP
specifically (403, confirmed via direct test) -- not a code bug, not
intermittent. fetch_mlb_schedule() (mlb_model.py) and
mlb_outcome_backfill.py's outcome-fetch both rewritten to MLB Stats
API, the same source pregame_context.py already used reliably from
Oracle the entire time. One real bug caught by testing before
shipping: an ARI/AZ alias-map fix that looked right on paper failed a
real Diamondbacks@Padres test case, rewritten to independent per-side
matching. Verified twice each (laptop + independent Oracle scripts,
byte-identical results), then confirmed live in the real 9PM
production run itself.

**SEDE calibration static line -- root cause found, a second finding
surfaced, sent to J@rv1s, replied to, chased further:**
The static "SEDE calibration" number traces to RESOLVED_MARKETS
(signal_scorer.py) being unmaintained since June 30. Separately, a
GDP/JOBS actual_outcome convention conflict was found (echoes the
July 14 precedent). J@rv1s asked whether that conflict is live or
historical before assigning urgency -- checked against real row data:
historical only, confirmed by grep + row inspection, not the code path
alone. That check is what surfaced the JOBS/GDP scoring-silence
finding above -- arguably the more important discovery of the night.

**NFL Sept 3 readiness -- two real bugs found and fixed, both
verified live:**
1. NFL Elo ratings were structurally pinned at through_season=2025
   for the entire 2026 season (nfl_season - 1 never changes),
   directly contradicting the documented week-by-week Elo update
   design. Fixed to through_season=nfl_season -- verified safe
   (byte-identical output right now, since no 2026 games have
   resolved yet; will correctly start incorporating real 2026 results
   once Week 1 finishes).
2. SharpAPI's shared odds-pagination function (_fetch_raw_odds, used
   by both MLB and NFL) has been crashing past ~500 rows since it was
   written -- root cause: the API returns an explicit null next_offset
   past that point instead of omitting the key, which dict.get()'s
   default doesn't catch. Fixed to follow next_cursor instead. This
   also delivered the actual 2-book coverage re-test: real number is
   30/271 events (11.1%), not the stale 1.6% from July 18 -- that
   figure was almost certainly measured under this same broken
   pagination and truncated. Worth not relying on the old number for
   Sept 3 planning.

Both NFL fixes committed (a611edf), verified live on the laptop, and
independently re-verified on Oracle (8f4f05c) before session close.

## OPEN POSITIONS / SPORTS MONITORING
Not run tonight. This was a continuation session already mid-
investigation at start (context carried over from earlier tonight),
not a fresh session-opener -- the standard trade health check
(paper_trades.json review, live Kalshi prices, sports scores for
game-tied positions) was not performed. Flag this for the next real
session open rather than assume it's covered.

## TONIGHT'S WORK ORDER FOR J@RV1S
1. JOBS silence since June 17 -- this is the priority. Worth deciding
   whether to chase root cause before or alongside Aug 25 prep.
2. GDP scoring stalled since July 30 -- likely related, needs its own
   look.
3. RESOLVED_MARKETS dynamism -- direction already agreed (make it
   read real backfilled outcomes instead of a hand list); build
   explicitly deferred past Aug 25 unless it turns out necessary to
   unblock #1/#2.
4. Confirm real production run (next 2AM/9PM CT) exercises tonight's
   Elo + SharpAPI fixes cleanly -- first real confirmation beyond
   tonight's manual verification.

Papa Ralph standard. If it's worth doing it's worth doing right.
