# SEDE Nightly Session Summary
## For J@rv1s Morning Intelligence Pull

**Last updated:** 2026-09-20 | **Covers:** Friday night (9/18-19) through tonight
**Prepared by:** Archie (Claude Desktop)

---

## THIS WEEKEND IN ONE PARAGRAPH

Three real sessions, a lot of ground. Friday night fixed a genuine
suspended-model leak into the ACT NOW tier (not just a display
section). Saturday morning found and fixed the actual mechanical
reason fixes have been "declared fixed but weren't" all month --
Python doesn't hot-reload, so a mid-run auto-pull never affected the
run that pulled it. Saturday's main session built the NFL outcome
backfill (NFL_SPREAD instantly clears Gate 1's n>=30 floor) and fixed
a real MLB_GAME dedup bug that was silently merging back-to-back
series games. Sunday verified J@rv1s's full external-repo referral
list (2 of 9 items don't check out -- flagged), researched
calibration fixes (not ready yet at our sample sizes), tested the
Kalshi candlesticks API live and got one concrete real number showing
a signal's true spread exceeded its claimed edge, resolved the NFL
win-rate scare from Saturday (no bug -- real, moderate overconfidence),
and gave Rus a straight "not yet, here's the actual gate" on pitching
8rain. Tonight: worked an 8-item punch list end to end, shipped real
guard-rails against the next silent-convention-mismatch bug
(confirmed live on Oracle), and closed out two smaller open questions.
**Everything code-related below is confirmed live on Oracle** -- no
pending pulls tonight, a first in a while.

---

## 1. FRIDAY NIGHT (~12:27 AM CT) -- SUSPENDED-MODEL LEAK INTO THE ACT
## NOW TIER ITSELF, NOT JUST A DISPLAY SECTION

Followed up on the "TB >13.5 in SEDE Signal Confidence" oddity your
Sept 18 briefing flagged as low-priority. It was worse than that.

**Root cause:** `sede_scores` (and `act_now_email`/`act_now_telegram`,
derived from it) were computed from `all_flagged`, not
`actionable_flagged`. The Sept 16 fix only ever repointed the
`flagged_econ=` argument on the two digest calls -- it never touched
`sede_scores`, which was computed one block earlier off the unfiltered
list. Net effect: a suspended model's signal could reach not just a
display section but the actual ACT NOW recommendation tier.

Fixed by moving `actionable_flagged`'s definition above that block and
scoring it instead of `all_flagged`. No change to `all_flagged` itself
or to `write_summary()`/console report/`signals_log.csv` attribution.
Verified with a synthetic suspended+live signal pair: the suspended
one is correctly excluded before `compute_sede_confidence()` ever sees
it. Committed `c99b750`.

---

## 2. SATURDAY MORNING (~9:31 AM CT) -- THE REAL REASON THINGS HAVE
## BEEN "DECLARED FIXED BUT WEREN'T" ALL MONTH

This is the most structurally important fix of the weekend, and it
directly answers the pattern you (J@rv1s) flagged as a meta-question
after Gate 4c and the MLB_GAME stale-text incident.

**Confirmed live, same morning:** the Sept 19 07:00 CT run's own
auto-pull succeeded (its post-run commit descended from `c99b750`,
pushed hours earlier) -- yet that same run's email still shipped with
the pre-fix leak behavior. Root cause: `auto_pull_if_safe()` updates
the `.py` files on disk, but the already-running Python process had
already read and compiled the OLD version of the file (and everything
it imports) the moment the interpreter started. Python doesn't
hot-reload -- a successful pull mid-script does nothing for the
process already executing it.

**Fix:** when `auto_pull_if_safe()` reports "pulled," immediately
`os.execv()` this same script as a fresh process, so the just-pulled
code is what actually runs that cycle. Guarded by a `_SEDE_REEXECED`
env var so it can only fire once per invocation chain. Verified the
mechanism in isolation (file-based evidence): first invocation logs,
sets the guard, re-execs; second invocation sees the guard set and
does not loop. Confirmed on Windows; Oracle's Linux execv is the
standard, better-behaved case. Committed as part of the Saturday
morning work.

**Honest limit stated in the commit itself:** this fix is subject to
the exact bug it fixes -- the first Oracle run that pulls THIS commit
still ran on pre-fix code for that one cycle. Every fix shipped after
this one takes effect the same run it's deployed, going forward.

---

## 3. SATURDAY -- SPREAD STRESS-TEST STATUS, NFL OUTCOME BACKFILL
## BUILT, MLB_GAME DEDUP BUG FOUND AND FIXED -- ALL LIVE ON ORACLE

Rus asked how to proceed on the (still 11+ days overdue) Sept 8 spread
stress-test checkpoint. Pulled fresh numbers rather than reuse the
Sept 6-8 write-ups.

**Finding: NFL_GAME and NFL_SPREAD had zero outcome-backfill mechanism
at all** -- confirmed in code, not inferred. 272 combined spread-tagged
NFL signals sat unresolved not because "too early to tell" but because
nothing wrote `actual_outcome` for NFL. Recommendation: build the
backfill first -- fastest real path to a meaningful stress-test answer,
bigger volume than MLB or GDP, and accumulating live during the
season.

**Built and ran `nfl_outcome_backfill.py`** (Step 12c, mirrors the
MLB/MLS pattern), sourced from `nfl_data_py`/nflverse -- not ESPN
(confirmed 403-blocked from Oracle's IP since Sept 11). Hand-verified
against real known scores before trusting it. Real result on
production data: **1,893 rows filled, 0 no-match, 343 not yet played,
89 preseason (permanently unmatchable -- nflverse carries no preseason
schedule data, flagged in code so it's never mistaken for a bug
later). NFL_SPREAD now has 210 distinct resolved signals -- clears
Gate 1's n>=30 floor immediately.** NFL_GAME has 11.

**Found and fixed a real MLB_GAME dedup bug** while building an
unrelated edge-bucket calibration check: the shared dedup fallback
(`MAX_GAP_DAYS=3`, for rows with no `market_ticker`) was tuned for
weekly-cadence sports and was silently merging real, separate MLB_GAME
series games 1-2 days apart into one event. Added
`PER_MODEL_MAX_GAP_DAYS={"MLB_GAME": 0}`. Verified before/after on
real data: **n=116 -> n=134 real distinct resolved games** (WR 53.4%
-> 54.5%, Brier 0.2294 -> 0.2290) -- 18 real games recovered that were
being silently discarded. Doesn't flip the Gate 1 verdict either way,
but means whatever number MLB_GAME lands on at season-end will now be
the real one.

**Decision:** let MLB_GAME ride to its natural season-end ceiling
(~Sept 27-28) rather than forcing an early verdict.

Committed `ca931c6`. **Confirmed live on Oracle** (git ancestry check
tonight: Oracle's own Sept 19 21:00 CT and later auto-update commits
descend from `ca931c6`) -- no pending action here.

---

## 4. SATURDAY -- MLB_GAME EDGE-BUCKET CALIBRATION: THE "~7PT
## UNDERCONFIDENCE" NUMBER ISN'T FLAT

Answering your Sept 17 question about whether the gap is flat, grows
with edge, or flips at the extremes: **it grows with edge, and does so
consistently** -- overconfident at the smallest edges, increasingly
underconfident at the largest:

| bucket | n | model conf% | actual WR% | gap |
|---|---|---|---|---|
| 0-3c | 220 | 53.5 | 44.5 | **-9.0** |
| 4-7c | 44 | 53.5 | 50.0 | -3.5 |
| 8-11c | 30 | 53.4 | 53.3 | -0.0 |
| 12-15c | 19 | 53.3 | 63.2 | +9.9 |
| 16-24c | 20 | 59.7 | 70.0 | +10.3 |
| 25c+ | 14 | 68.9 | 85.7 | **+16.8** |

Holds independently on YES-only and NO-only splits -- real, not one
noisy aggregate line. Your pushback (a pure selection effect should
produce overconfidence at the tails via winner's-curse, not
underconfidence) is directly supported by this shape -- argues for
real, compressed signal rather than a selection artifact. Caveat: 25c+
is thin (n=14). Separate finding worth tracking: the 0-3c bucket's raw
WR (44.5%, n=220) sits below the 50% coin-flip line, not just below
stated confidence -- smallest-edge MLB_GAME picks may carry near-zero
information content.

This is also where the dedup bug in Section 3 was actually discovered
(comparing with/without the production dedup algorithm on the same
436 rows disagreed by 5.5%).

---

## 5. SUNDAY -- J@RV1S REFERRAL LIST FULLY VERIFIED: 2 OF 9 DON'T
## CHECK OUT

Went through both Tier 1 and Tier 2 of your external-repo scan,
fetching actual content rather than trusting descriptions.

**Real and verified:** `briancox730/unofficial-kalshi-api-docs` (the
V1 order endpoint was sunset 2026-06-18, replaced by a very different
V2 wire format -- moot for us today since SEDE has zero live
order-placement code, 100% forward-looking value, bookmark it for
whenever real execution gets built); `ByteLogic214/mlb-proyeccion`
(substituted for the unverifiable mlb-pe -- confirmed real park-factor
bug, and confirmed our own `mlb_model.py` has zero park-factor
modeling at all, a real gap); `penaltyblog` (real, ships Brier/RPS/
implied-odds tools -- also surfaced that Shin's de-vig method's own
reference-implementation author published a 2026 piece arguing to
retire it in favor of the simpler multiplicative method for
low-overround markets like Kalshi); `fivethirtyeight/nfl-elo-game`
(real, legitimate, not deep-audited); `pespila` (real but a 1-star
personal project, low-confidence as a technique source); `pybaseball`
(real, well-established -- the single most directly useful item, since
we have zero park-factor modeling); `pykalshi` (real but ambiguous --
two unrelated repos share the name).

**Did not check out:** `WalrusQuant/mlb-pe` and `kalshi-pandas` --
neither exists under any search strategy tried (direct search,
site:github.com, literal URL). Sent you a direct process note (not
routed through Rus) asking for a quick existence check on future
referrals before they reach me -- not a big deal, just cheaper to
catch at your end.

---

## 6. SUNDAY -- CALIBRATION FIX RESEARCH: REAL TECHNIQUE EXISTS, NOT
## SAFE TO USE YET AT OUR SAMPLE SIZES

Standard approach is post-hoc recalibration via sklearn's
`CalibratedClassifierCV`: Platt scaling (one logistic curve, works
with less data, better for symmetric miscalibration) or isotonic
regression (more powerful, can fix a non-monotonic curve like
NFL_SPREAD's, but sklearn's own docs say it needs 1,000+ samples to
avoid overfitting to noise). NFL_SPREAD sits at n=210 -- a fifth of
that threshold. Building an isotonic correction now would very likely
fit noise, not a real curve. **Recommendation: Platt scaling is the
defensible option today; isotonic goes on the roadmap for whichever
model first clears ~1,000 resolved signals (none are close).** Not a
tonight-build either way.

---

## 7. SUNDAY -- THE TWO BEST FINDS, ACTUALLY TESTED LIVE

**Kalshi's historical candlesticks API is real and works** (correcting
an earlier wrong path -- it needs the series ticker too:
`/series/{series_ticker}/markets/{ticker}/candlesticks`). Pulled a
real 1-minute candle for an actual logged NFL_SPREAD signal
(`KXNFLSPREAD-26SEP13ARILAC-LAC10`, claimed edge 6.3c): **real spread
at that minute was 7-10c -- bigger than the signal's entire claimed
edge.** One data point, not a stress test, but a real, concrete,
measurable example of exactly the failure mode the Sept 8 checkpoint
exists to catch.

**The real blocker is bigger and on two axes, not one:** `market_ticker`
is populated in only 52% of `signals_log.csv`'s 9,518 rows. Of the
rows that DO have a ticker, only 447 (4.7% of the total) already have
bid/ask/spread captured -- across just 4 of 19 models, all logged since
Aug 29. So a full backfill needs both ticker reconstruction AND new
spread capture for most of the historical log, not just one missing
piece. Where spread data does exist, it's internally consistent (zero
bid>=ask violations, spread math exact) -- the gap is coverage, not
correctness.

**Pandera, installed and run against all 9,518 real rows:** passed
clean on schema and cross-field checks. Confirms the pipeline code
that IS writing spread data does it correctly -- not a bug catch
today, but a real, cheap guard-rail worth adding so the next
convention-mismatch bug (see Section 9) fails loudly instead of
silently. Uninstalled after testing; not adopted into the codebase.

---

## 8. SUNDAY -- NFL WIN-RATE SCARE FROM SATURDAY: RESOLVED, NO BUG

Checked and ruled out directly: wrong `actual_outcome` convention (no
-- the two conventions never collide in practice), a home/away ticker
swap (hand-verified 4 real Week 1 games against an independent
nflverse pull, all correct), and a global sign-inversion (no -- both
directions sit below their own stated confidence, not mirror-image).

**Real, separate findings per model:** NFL_GAME (n=11) is just too
small to conclude anything yet -- needs 30. NFL_SPREAD (n=210, Brier
0.2345, technically beats random's 0.25 but barely) was being compared
against the wrong baseline -- the model's own average stated confidence
on its picks is 42.5%, not 50%. The real question is whether 30.5%
actual tracks 42.5% predicted, and it's noisy: the 70%+ confidence
bucket (the model's own highest-conviction picks) undershoots hardest
-- 58.1% actual vs 81.0% stated. Real, moderate overconfidence, not a
broken model. No code fix -- keep tracking as volume grows.

---

## 9. SUNDAY -- 8RAIN DISCORD PITCH: NOT YET, HERE'S THE ACTUAL GATE

Rus floated pitching Anthony's 8rain Discord on NFL_SPREAD if it beats
Brier. Gave the straight answer: not pitch-ready, for five concrete
reasons -- Gate 1 is still project-wide suspended; 0.2345 vs 0.25 on
n=210 is a thin, unchecked margin, not a demonstrated edge; NFL_SPREAD
does not clear its own Gate 1 bar today (30.5% WR vs required 55%);
calibration is noisiest exactly where a pitch would lean hardest
(the 70%+ bucket); and beating Brier isn't the same as being
profitable after real spreads -- that gap is precisely what the
suspended stress-test exists to catch. Laid out what would actually
make it pitch-ready (stress-test resolved, model clears its own Gate 1
independently, Brier edge shown real at 400-600+ signals, a monotonic
calibration curve). Standing plan: keep tracking, flag the moment it
actually clears -- not a vibe check.

---

## 10. TONIGHT -- 8-ITEM PUNCH LIST WORKED 1-8, NEW GUARD-RAILS
## SHIPPED AND CONFIRMED LIVE ON ORACLE

1. **Bid/ask-vs-edge check** across all 447 populated spread rows: 0%
   violations across all 4 models -- verified this is real (live
   fetches, not synthetic), not suspiciously clean.
2. **The 48% market_ticker gap**, broken down by model: almost
   entirely dead/retired models plus already-fixed historical debt,
   not a live blocker. Side-effect finding, still open: **CPI has
   logged zero signals since June 10** despite three real release
   dates (Jul 15, Aug 13, Sep 10) already passing. Root cause not yet
   diagnosed -- flagged plainly, deliberately not chased tonight to
   keep moving through the list.
3. **GDP's apparent Sept 6 spread-capture "stall"**: not a bug.
   Correct dedup behavior on already-logged, still-unresolved signals
   being re-logged repeatedly. Corrects Saturday's write-up.
4. **Fresh MLB_GAME baseline**, current production numbers: n=134,
   62.7% WR, 0.229 Brier -- both directions still individually fail
   the Brier<=0.20 bar. Real 2026 MLB season end confirmed externally
   as Sept 27.
5. **Sent the J@rv1s referral note** directly (Section 5) rather than
   waiting to relay it through Rus.
6. **Shipped real guard-rails** in `signal_scorer.py` and
   `brier_dashboard.py`: validate `direction` normalizes to exactly
   YES/NO and `actual_outcome` is exactly 0/1 before either is used in
   a correctness computation, matching the existing warn-loudly-drop-
   the-row style. Hand-rolled, no new dependency (Rus's explicit call
   over adding pandera as a real dependency). Verified compile-clean,
   zero regression against real production data, and fire-tested
   against a deliberately corrupted row (correct warning fired, row
   dropped). **Committed `ce52ce5`, pushed, and confirmed
   fast-forward-pulled onto Oracle by Rus directly tonight
   (`99a1ed1..ce52ce5`, exact match).**
7. **paper_trades.json's non-canonical tickers**: not a code bug --
   `position_manager.py`'s manual entry wizard already shows a correct
   real-format example, the shorthand tickers are human entry. Live
   API check confirmed the one currently-OPEN paper trade's ticker
   (`KXGDP-26OCT30-T3.5`) is valid and resolves fine -- zero live
   impact. No fix recommended; workflow note only for future manual
   entries. Not worth backfilling the 24 historical tickers --
   `paper_trades.json` isn't used for Gate 1 scoring.
8. **Deferred, not actionable yet** -- same conclusion as Section 6:
   no model is close to the sample size needed for a real
   recalibration build.

---

## OPEN POSITIONS (per `sede_portfolio.json`, current as of tonight's
## 11:20 AM CT pipeline run)

| # | Description | Entry | Current |
|---|---|---|---|
| 1 | BTC<$50k Dec31, NO | 43.5c | 88.5c |
| 3 | GDP>1.5% (Oct30), YES | 70.5c | 88.5c |
| 4 | GDP>1.5% (Jan28), YES | 74.5c | 76.5c |
| 5 | GDP>2.5% (Oct30), YES | 58.0c | 70.5c |
| 6 | GDP>1.0% (Jan28), YES | 78.0c | 84.0c |
| 7 | GDP>2.0% (Apr29), YES | 50.5c | 52.5c |
| 8 | GDP>4.0% (Oct30), YES | 18.5c | 30.5c |
| 9 | GDP>1.5% (Jul29), YES | 59.5c | 77.5c |

Bankroll: $994.17 (unchanged). No new entries since early August.

---

## GATE 1 / VALIDATION STATUS

Project-wide Gate 1 remains provisionally suspended (since Aug 11,
pending the bid/ask-spread stress test -- Sept 8 checkpoint now 12
days overdue). The Kalshi candlesticks API (Section 7) is the most
promising unblock path found so far, but only tested on one signal.

| Model | Status |
|---|---|
| JOBS | Caveated, not unconditionally validated |
| GDP | Reduced weight (0.60); spread capture confirmed working correctly (Section 10.3); resolves Oct 30 |
| MLB_GAME | n=134, 62.7% WR / 0.229 Brier -- fails Brier only; dedup fixed and live; riding to Sept 27-28 season end |
| MLS_GAME | Retired from further live development |
| CLAIMS | Suspended |
| NFL_GAME | n=11 -- not evaluable, needs 30 |
| NFL_SPREAD | n=210, real overconfidence pattern (Section 8), fails its own Gate 1 (30.5% WR); not pitch-ready (Section 9) |
| EPL_GAME | Suspended pending validation |
| CPI | **Silently producing zero signals since June 10 -- open, undiagnosed (Section 10.2)** |

---

## CARRIED FORWARD, STILL OPEN

- **CPI's silent zero-output since June 10** (Section 10.2) -- the
  single most concerning open item from tonight, arguably more urgent
  than most items above it since it's a live model producing nothing
  and nobody had noticed.
- Monitoring/alerting Stage 1 build (external heartbeat, whole-
  pipeline state alerting, short daily digest) -- FORGE'd, scoped,
  not started.
- Monitoring Stage 2 needs an answer from Archie on how cheaply
  `daily_runner.py` supports per-component manifest logging before it
  can be sized honestly -- still owed.
- Single-source-of-truth refactor for duplicate status/classification
  logic -- the real long-term fix for the drift pattern Section 2's
  re-exec fix only partially addresses.
- Gate 4c/4d cross-model resolution-window cap -- design can proceed,
  build waits for real entry activity.
- An untracked `Claude outputs/` folder appeared in the KalshiBot repo
  (one old file, predates this weekend) -- flagged, not investigated,
  doesn't affect any commit.
- MIA@SF anomaly from an earlier Sunday check (both YES and NO flagged
  simultaneously at identical 37.0% model_pct, 21.0c edge each) --
  still needs follow-up verification.

---

## TOMORROW'S WORK ORDER

1. Diagnose CPI's silent zero-output since June 10 -- this is now the
   top real-model item, not a nice-to-have.
2. Run the Section 7 spread-vs-edge check across a bigger slice as
   more `market_ticker`+spread coverage accumulates (currently only
   447 rows have it).
3. J@rv1s: weigh in on whether Stage 1 monitoring or the CPI diagnosis
   is the better use of tomorrow's build time -- both are real, only
   one can go first.
4. Keep tracking NFL_SPREAD's calibration as volume grows toward a
   size where Platt scaling is worth actually building (Section 6).
5. MLB_GAME: no action needed until ~Sept 27-28 season end.

---

## SPORTS MONITORING

No live in-progress games flagged at session end. NFL backfill now
running as Step 12c alongside MLB (Step 12) and MLS (Step 12b) every
cycle.

---

## ORACLE CLOUD STATUS

Pipeline running normally. **Everything committed this weekend is
confirmed live on Oracle** -- `c99b750`, the re-exec fix, `ca931c6`
(NFL backfill + MLB dedup), and `ce52ce5` (guard-rails, pulled and
verified by Rus directly tonight). No pending manual pull for the
first time in several nightly summaries.

---

Archie | Papa Ralph standard -- full detail in commits `c99b750`,
`ca931c6`, `ce52ce5` (GitHub) and this weekend's project docs
(`spread_stress_test_status_and_recommendation`,
`mlb_game_edge_bucket_calibration_and_dedup_bug`,
`nfl_backfill_and_mlb_dedup_fix_built`,
`jarvls_external_repo_referral_verified`,
`calibration_research_and_tier2_referral_verification`,
`spread_backfill_live_api_test_and_pandera_prototype`,
`nfl_winrate_investigation_no_bug_found`,
`nfl_spread_8rain_pitch_gate`, `note_for_jarvls_verify_referrals`).
