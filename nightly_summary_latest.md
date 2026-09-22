# SEDE Nightly Session Summary
## For J@rv1s Morning Intelligence Pull

**Last updated:** 2026-09-21 | **Covers:** tonight's session only (~9:00 PM - 10:14 PM CT)
**Prepared by:** Archie (Claude Desktop)

---

## TONIGHT IN ONE PARAGRAPH

Rus opened by asking us to check in on the NFL now that Week 2 is
done, then set the order: CPI first (season-critical urgency low,
but a live model silently producing nothing needed answering), then
straight into the NFL issue. Confirmed real current NFL_GAME (n=31)
and NFL_SPREAD (n=403) numbers and traced the Week 2 "confidence
bucket collapse" scare to just 2-3 real bad game-reads, not broad
model degradation -- but that dig surfaced something bigger: raw n
massively overstates real independent sample size for several
models, because one real game often produces many correlated Kalshi
contracts (spread thresholds, per-side tickers). Diagnosed and fixed
CPI's zero-signal-since-June-10 bug (a backwards month-mapping,
confirmed against live Kalshi ticker data). Cut CPI's MoM feed as a
dormant landmine rather than either building it properly or leaving
it as a silent risk. Then built and verified the fix for the
correlated-sample-size problem: `brier_dashboard.py` now reports a
second, honest `n_events` number next to raw `n`, and Gate 1's n>=30
floor is now checked against it. **Nothing committed tonight has been
pulled to Oracle yet -- three commits pending your pull.**

---

## 1. NFL WEEK 2 CHECK-IN -- REAL NUMBERS, NOT A BROAD PROBLEM

Confirmed via real production data: NFL_GAME n=31 resolved signals,
NFL_SPREAD n=403. Investigated the Week 2 confidence-bucket pattern
your briefing flagged. Traced it to 2-3 real bad game-reads
(MINCHI, CLETB) rather than a systemic model issue -- the scare was
real data, but narrow, not broad degradation.

That investigation is what surfaced the bigger correlated-sample-size
issue below: NFL_SPREAD's raw n=403 collapses to just 30 real
distinct games once you account for the many strike-threshold
tickers Kalshi lists per game; NFL_GAME's raw n=31 collapses to 25
distinct games (two tickers per game, one per side).

---

## 2. CPI FIX -- ZERO SIGNALS SINCE JUNE 10, ROOT-CAUSED AND FIXED

`models/cpi_model.py` had a `RELEASE_TO_DATA_MONTH` dict that shifted
a ticker's month code back one month before looking up consensus --
on the assumption that a ticker's month code was the release month,
not the data month. That assumption was backwards. Confirmed live
against the real Kalshi API: `KXECONSTATCPIYOY-26SEP-T3.5` is titled
"CPI year-over-year in Sep 2026?" with `close_time: 2026-10-14` -- a
"26SEP" ticker IS September data, releasing ~mid-October (BLS's
normal ~1-month lag). The old code silently evaluated every live CPI
YoY market against the wrong month's consensus and release date, and
outright skipped it whenever the shift landed on an already-passed
release date -- which is why nothing had logged since June 10.

**Fix:** removed the translation entirely; the ticker's own month
code is looked up directly. Verified end-to-end through the real
`run_cpi_model()` call against live Kalshi data -- recovered 2 real
flagged September signals (`-T3.5`, `-T3.7`, both FLAG NO), and
confirmed October/November buckets now show correct release dates
(Nov 12, Dec 10) matching the real BLS calendar. Committed `eac66e4`.

---

## 3. CPI MoM CUT FROM THE FEED -- DORMANT LANDMINE, NOT A SAFE NO-OP

While fixing YoY, found `daily_runner.py` was also feeding
`KXECONSTATCPI` (MoM) markets into the same CPI model. Real liquidity
exists on these (confirmed ~$14K volume on some thresholds), so this
wasn't dead code -- it just never produced a signal, because
`cpi_model.py`'s "skip trivially-true thresholds" filter (`<1.0`) is
tuned for YoY's 2-4% scale and happens to catch every real MoM
threshold (-0.2% to 0.9%) too. That's a coincidence, not a safeguard:
a high-inflation month with a MoM print over 1.0% could slip that
filter and get scored against YoY-scale consensus numbers, producing
a wildly wrong probability and a fake outsized "edge."

Cut `KXECONSTATCPI` from the feed tonight rather than either quietly
leaving that risk live or building real MoM support under time
pressure. Real MoM support needs its own consensus table, its own
trivial-threshold cutoff, and its own monthly upkeep -- flagged as a
real backlog item, not a quick add-back. Committed `986ad15`. Project
doc: `cpi_mom_backlog_20260922`.

---

## 4. THE "RECURRING THEME" -- CORRELATED/DUPLICATE SIGNAL COUNTING

Rus's instinct that this is a recurring theme across projects checked
out, with real evidence, but it's actually two distinct bug classes,
not one:

- **Class A (duplicate logging):** the same real event gets counted
  more than once due to a logging bug or a mistuned dedup heuristic.
  This is the MLB_GAME bug fixed Sept 19.
- **Class B (correlated-but-legitimate):** genuinely different,
  legitimately separate Kalshi contracts (spread thresholds, or
  per-side tickers) that all settle on one real-world outcome, but
  get counted as independent trials for Gate 1 -- inflating apparent
  sample size and making one bad model read look like many
  independent failures.

Verified real numbers per model (rows -> distinct real events):
NFL_SPREAD 403->30 (97% of rows correlated), MLS_GAME 49->29 (59%),
NFL_GAME 31->25 (24%), MLB_GAME 135->134 (barely any -- mostly a
separate, unrelated problem: 90 of its rows have no `market_ticker`
at all, so real-event verification isn't even possible for them --
that's a coverage gap, not a correlation problem, and needs its own
future investigation). SOCCER_GAME and WC_GAME show no correlation
issue (WC_GAME's rows are 100% ticket-less, so trivially 1:1).

Also found, while scoping this: THREE separate places in the
codebase independently implement "count real events for Gate 1" --
`signal_scorer.py`, `brier_dashboard.py`'s `model_breakdown()`, and
`daily_runner.py`'s MLB-specific `get_mlb_game_direction_stats()`.
That matches the single-source-of-truth refactor J@rv1s already
flagged as backlog item 7. Deliberately scoped tonight's fix to
`model_breakdown()` only -- the other two were NOT touched, to avoid
turning one night's fix into an uncontrolled rewrite. Project doc:
`correlated_signal_counting_theme_20260922`.

**Fix shipped and verified against real data:** `model_breakdown()`
now reports both the raw row count `n` and a new honest `n_events`
(distinct real-event count) side by side. Gate 1's n>=30 floor is now
measured against `n_events`. All existing per-row Brier/win-rate/
beats_random metrics are completely unchanged -- this is additive,
not a rewrite of what's already reported. Verified against real
production `signals_log.csv` data (not just compiled): NFL_SPREAD and
NFL_GAME's `n_events` matched tonight's investigation exactly (30 and
25); two other models' quoted counts needed a self-correction during
verification (see project doc for the honest accounting -- both
turned out to be memory imprecision on my end, not a code bug).
Committed `d1d46d1`.

---

## UPDATED GATE 1 SAMPLE-SIZE READING (n_events, not raw n)

This is the new honest number as of tonight -- use this over raw `n`
going forward for anything Gate-1-related:

| Model | n (rows) | n_events | Clears n>=30 floor? |
|---|---|---|---|
| NFL_SPREAD | 403 | 30 | Yes (barely) |
| MLB_GAME | 135 | 134 | Yes |
| SOCCER_GAME | 49 | 49 | Yes |
| WC_GAME | 70 | 70 | Yes |
| NFL_GAME | 31 | 25 | No |
| MLS_GAME | 49 | 29 | No (barely) |
| GDP | 6 | 6 | No |
| JOBS | 6 | 6 | No |
| CLAIMS | 5 | 1 | No (already suspended) |

Note this is `n_events` clearing the *count* floor only -- it says
nothing about win rate or Brier, and does NOT touch the still-open,
still-overdue project-wide Gate 1 suspension pending the real
bid/ask-spread stress test (Sept 8 checkpoint, now 13 days overdue).
Don't read "clears n>=30" as "cleared for trading."

---

## GATE 1 / VALIDATION STATUS

Project-wide Gate 1 remains provisionally suspended (since Aug 11,
pending the bid/ask-spread stress test -- Sept 8 checkpoint now 13
days overdue, no new movement on this tonight).

| Model | Status |
|---|---|
| JOBS | Caveated, not unconditionally validated |
| GDP | Reduced weight (0.60); scoring stalled since July 30, root cause still open |
| CPI | **Fixed tonight** -- YoY month-mapping bug resolved, MoM feed cut as dormant risk |
| MLB_GAME | n_events=134; no-ticket coverage gap on 90 rows flagged as a separate open item |
| MLS_GAME | n_events=29, just under the floor |
| CLAIMS | Suspended |
| NFL_GAME | n_events=25 -- Week 2 scare traced to 2-3 bad reads, not systemic |
| NFL_SPREAD | n_events=30 -- clears the count floor for the first time using the honest metric |
| SOCCER_GAME / WC_GAME | No correlation issue found; not otherwise investigated tonight |

---

## CARRIED FORWARD, STILL OPEN

- **Project-wide Gate 1 suspension** -- bid/ask-spread stress test
  still not built, checkpoint 13 days overdue. This is the biggest
  standing blocker in the whole project and got no new work tonight.
- **Single-source-of-truth refactor** for "count real events for
  Gate 1" logic -- now confirmed THREE independent implementations
  (`signal_scorer.py`, `brier_dashboard.py`, `daily_runner.py`'s MLB
  function). Tonight only fixed the dashboard's copy.
- **MLB_GAME / WC_GAME no-`market_ticker` coverage gap** (90 and all-
  of-70 rows respectively) -- a different, more basic problem than
  the correlation issue, needs its own future investigation.
- **CPI MoM real build** -- separate consensus table, own trivial-
  threshold cutoff, own monthly upkeep. Not a quick add-back.
- Monitoring/alerting Stage 1 build -- FORGE'd, scoped, not started.
- Gate 4c/4d cross-model resolution-window cap -- design can proceed,
  build waits for real entry activity.
- An untracked `Claude outputs/` folder in the KalshiBot repo --
  flagged repeatedly, still not investigated, doesn't affect commits.
- MIA@SF anomaly from an earlier session (both YES and NO flagged
  simultaneously at identical model_pct/edge) -- still needs
  follow-up verification.

---

## TOMORROW NIGHT'S WORK ORDER (ranked, most urgent first)

1. **Pull tonight's 3 commits to Oracle** (`eac66e4`, `986ad15`,
   `d1d46d1`) -- nothing tonight is live yet.
2. **Project-wide Gate 1 spread stress test** -- the single biggest
   overdue item in the project (13 days past checkpoint). Needs a
   real decision on scope/approach, not just another flag.
3. Decide whether to start the single-source-of-truth refactor for
   the three duplicated "count real events" implementations, or hold
   it until more of Gate 1 resolves.
4. MLB_GAME / WC_GAME no-ticket coverage gap -- scope an investigation.
5. CPI MoM real build, if/when it's worth prioritizing (not urgent --
   currently safely cut from the feed).
6. MIA@SF anomaly follow-up.
7. Stage 1 monitoring/alerting build.

---

## SPORTS MONITORING

No live in-progress games flagged at session end.

---

## ORACLE CLOUD STATUS

Pipeline running normally on its existing cadence. **Nothing from
tonight has been pulled yet** -- `eac66e4`, `986ad15`, `d1d46d1` are
pushed to GitHub and waiting on Rus's manual `git pull` on Oracle,
per standing protocol (Archie never runs commands on Oracle itself).

---

Archie | Papa Ralph standard -- full detail in commits `eac66e4`,
`986ad15`, `d1d46d1` (GitHub) and tonight's project docs
(`cpi_mom_backlog_20260922`, `correlated_signal_counting_theme_20260922`).
