# SEDE Nightly Session Summary
## For J@rv1s Morning Intelligence Pull

**Covers:** Fri Sept 25 through Sun Oct 4, 2026 | **Session end:** ~8:00 PM CT, Oct 4
**Prepared by:** Archie (Claude Desktop)
**Replaces** the Oct 2-4 version of `nightly_summary.md` and `nightly_summary_latest.md` (identical content pushed to both). This one is the single consolidated record for the whole ten days.

---

## THE TEN DAYS IN ONE PARAGRAPH

Sept 25-26 were a build-and-close stretch (fee fix, NFL_SPREAD suspended for a real calibration defect, staleness and cross-check guards, CPI sourcing). Sept 27-29 had no build sessions; Oracle ran the pipeline on schedule. Sept 30 and Oct 1 were the strategy turn: the Sept 8 Gate 1 checkpoint was closed as "not resolved", the raw win-rate gate was found to be the wrong yardstick for underdog-skewed models and was replaced (staged) by a market-relative test, JOBS and CLAIMS got real bid/ask capture and the CLAIMS model's stale-data bug was fixed, and macro outcomes now auto-resolve from Kalshi settlement. **The big event is the Oct 3 data-integrity bug:** the NFL_GAME, NFL_SPREAD, MLS_GAME and MLB_GAME outcome backfills wrote the YES-frame result for every NO-direction row, and the dashboard read them as "signal won." Repaired in 9385fd7. The Oct 1 "NFL_GAME watch" tag and the Oct 3 Phase 1 report were computed on inverted data and are void. On corrected data **no model has demonstrated an edge**; MLB_GAME flipped from "no edge" to "strong but least verified" and must NOT be promoted on this look. On Oct 4 the work moved to whether our own test can detect an edge at all: an external audit packet was built, reviewed internally, and is ready to go to three outside AI reviewers (Rus sends it Oct 5 from work).

---

## 1. FRI SEPT 25 - SAT SEPT 26: BUILD AND CLOSE

**Shipped Sept 25 (all pushed):**
- Kalshi taker fee in the real-ask stress test (9cf547c): `0.07 * cost * (1 - cost)` on both legs. NFL_SPREAD real-ask P&L at that point -389c over 324 rows.
- NFL_SPREAD suspended from paper-trade eligibility (80e6e39) after a system-wide calibration audit found it genuinely overconfident (win rate fell from 37% in the lowest edge bucket to 21% in the highest). Root cause traced to `margin_cover_probability()` treating the Elo margin estimate as certain. The parameter-uncertainty fix was scoped and deliberately not built (FORGE: wait for more real games). On the record: I told Rus NFL_SPREAD was not formally suspended; that was wrong, a second list in `daily_runner.py` had it suspended since Sept 18. No practical harm.
- One source of truth for model-suspension state (61af8bd); per-model staleness alert (6c593ff); scorer-vs-dashboard accuracy cross-check (adf35fb); Hypothesis property tests (e7105b6); CPI Part B real June/July `RESOLVED_MARKETS` entries (fa65b00); MLB park-factor reference data, not integrated (8b35194).
- Closed: manual-trading "stall" was not a stall (three correct reasons); WC_GAME/SOCCER_GAME gap-clustering verified clean.

**Sept 26 (investigation, no code):** Gate 4d design question answered (no blocking cross-model gate); WC_GAME/SOCCER_GAME relabel closed (deliberate, the sample-merge fix was already live); SOCCER_GAME's 36.1% win rate traced to legacy data (87% of the sample predates the June 25 fix; post-fix 46.2% on n=13). CPI Sept consensus sourced and release date fixed to Oct 14 (3a4fef6). Deep tooling research delivered to Rus; the Graphify repo reviewed critically and not installed.

**Sept 27-29:** no build sessions. Oracle auto-updates ran at 07:00, 11:15 and 21:00 CT as scheduled.

---

## 2. TUE SEPT 30: GATE 1 CHECKPOINT CLOSED, REAL FINDING ON WHAT THE MODELS ARE DOING

- **Sept 8 checkpoint closed** as "NOT RESOLVED - insufficient model coverage" (the Sept 22 stress test covered NFL_SPREAD / MLB_GAME / NFL_GAME only). Suspension continues as a dated, per-model process: Oct 30 (GDP resolution) and the Oct 9 JOBS spread-capture build deadline (confirmed by Rus Oct 1). Verdict doc: `gate1_checkpoint_closed_verdict_20260930`.
- **`real_event_id()` already existed** (the "unbuilt" backlog item and my Sept 26 repeat of it were stale). What was missing was event-level win rate, Brier and P&L; added in c4896e5.
- **JOBS "n=73, 58.9% WR" was repeated signals on the same markets counted as independent trials.** JOBS is unproven. CPI's 24 "events" and 87.5% win rate are the same inflation (CPI rows have no tickers).
- **Model vs market:** realized win rate tracks the market's price, not the model's stated probability, for NFL_SPREAD, SOCCER_GAME, MLB_GAME, MLS_GAME and NFL_GAME. NFL_SPREAD real-ask P&L reversed from +717c (Sept 22, early-sample luck) to -2233c as Week 2-3 resolved. Shrinkage test (in-sample, 40-137 events): NFL_SPREAD and MLS_GAME add no information over the market.
- Laptop synced to Oracle-current data non-destructively (stash, not discard); origin/main level on laptop, GitHub and Oracle.

---

## 3. WED OCT 1: STRATEGY TURN AND PIPELINE FIXES

**Criteria:** raw >=55% win rate / Brier<=0.20 does not measure edge for models buying ~33c underdogs (33% WR on 33c is break-even). J@rv1s's FORGE pass: replace it as the binding test with a market-relative one, **staged**. Phase 1 (now) reports Brier-vs-market, win-rate-vs-price-paid and shrinkage weight w with bootstrap CI, nothing gating. Phase 2 (checkpoint Nov 1 or +4 weeks of events) locks the threshold once, on real-ask prices, written down before the data is examined. Sizing weight is the **lower bound** of w's CI. Evidence supports "stop trading the extreme disagreement tail" on NFL_SPREAD, MLS_GAME, SOCCER_GAME, MLB_GAME; it does not prove zero information elsewhere.

**Built and shipped:**
- JOBS real bid/ask/spread now persisted on every flagged signal (dfede9c). First captured row Oct 1 21:00 CT. Deadline was Oct 9.
- CLAIMS: found a confirmed bug, a 15-week-stale price history spliced into the consensus since mid-September (every September signal was YES with 40-60c "edges", which is a data bug, not an edge). Fixed (c21ae77): 30-week FRED history, errors-based std with a 12K floor, fail-closed on stale data, bid/ask captured, rows tagged `CLAIMS_FRED_v2`; older CLAIMS rows no longer count as evidence (dashboard CLAIMS n 5 -> 0, intended). Not yet verified in a real production run; first chance is the Oct 7-8 CLAIMS markets. CLAIMS stays suspended; replay-first reinstatement path (59 settled Kalshi events) is in its own referral.
- Macro outcomes were hand-entered into a table nobody had updated since Jun/Jul, which was the root cause of the Jul 30 GDP scoring stall. **Kalshi settlement auto-resolution built and switched on** (e1e413f, e17cb07): GDP, JOBS, CPI, CLAIMS. A label collision had mis-scored 69 older GDP rows; after repair GDP has n=1 real event.
- Kalshi historical macro prices are feasible to pull for the market side (public, no auth; 42 PAYROLLS, 22 GDP, 59 CLAIMS settled events). The model side is not: a pooled-macro test needs point-in-time consensus we do not have, except for CLAIMS, which is replayable from FRED.

**Other:** Nov 1 realism check written (Phase 2 is buildable but the most likely honest result for NFL_GAME is "not distinguishable"; lock the threshold by about Oct 20; do not redefine again). J@rv1s's commercialization assessment: no new project, the product is already specced, the gap is evidence; legal "signal service vs investment advice" research has had zero progress since June 15.

---

## 4. FRI OCT 2: KALSHI-VS-SPORTSBOOK CROSS-CHECK SIZED (nothing built)

Answered J@rv1s's scoping doc (`claude/kalshi_vs_sportsbook_crosscheck_phase1_sizing_20261002.md`). Verdict: worth doing as a cheap, backtest-only **benchmark** (score our models against a sharp, de-vigged price), not as a trade signal. Probability it finds a gap that survives Kalshi fees and the real spread: about 10%. Holes found in the scoping doc: vig (a naive comparison manufactures about 2c per side), DraftKings is a retail book (sharp reference should be Pinnacle or a consensus), fees plus half-spread (about 3-4c) kill most 1-3c gaps, timing, resolution rules, regulatory status unverified. Must not displace Oct 9, Oct 30 or Nov 1; shares its price-pull with the Nov 1 real-ask rebuild.

---

## 5. SAT OCT 3: THE OUTCOME-CONVENTION BUG (found, fixed, repaired)

**Cause.** `nfl_outcome_backfill.py`, `mls_outcome_backfill.py`, `mlb_outcome_backfill.py` wrote `outcome` (did the YES side happen) where the project convention is "did the signal win." Equal on YES rows, exact complements on NO rows. The Sept 24 dashboard change moved to the signal-won reading and nobody reconciled the backfills.

**How verified (three independent ways).** Code read; Kalshi settlement vs stored `actual_outcome` (NO rows mismatched 100%: NFL_GAME 164/164, NFL_SPREAD 837/837, MLS_GAME 113/113; YES rows 0%); and a logical impossibility (52 of 77 tickers scored in both directions had both "won" or both "lost"). GDP (888 rows), JOBS (243) and CLAIMS (23) checked clean: they use Kalshi auto-resolution.

**Fix (9385fd7).** Backfills now write signal-won and `p_yes_model`. `tools/repair_backfill_convention.py` (idempotent, dry-run default, backs up) flipped 1,258 NO rows (NFL_GAME 164, NFL_SPREAD 895, MLS_GAME 116, MLB_GAME 83) and marked 1,786 YES rows as repaired. After repair: 0 mismatches vs Kalshi on 2,700 ticker rows.

**Corrected dashboard (deduped, as of the Oct 4 11:15 CT run).**

| Model | Accuracy before -> after | Brier |
|---|---|---|
| NFL_GAME | 45.3% -> 37.7% | .2353 (unchanged) |
| NFL_SPREAD | 33.1% -> 32.9% | .2167 |
| MLS_GAME | 39.8% -> 49.4% (NO rows 31.9% -> 68.1%) | .2757 |
| MLB_GAME | 54.3% -> 62.3% (NO rows 37.8% -> 59.2%) | .2274 |

**Market-relative (Phase 1) on corrected data.** NFL_GAME real ask (27 events): model Brier .2330 vs market .2162, w* 0.13 [0.00, 1.00], net P&L about +2.2 to +2.6c per contract, interval spanning zero. On mid (41 events) Brier .2353 vs .2098, w* 0.00. NFL_SPREAD real ask (46 events): w* 0.17 [0.00, 0.73], P&L +0.7c [-7.9, +9.3]. MLB_GAME mid (137 events): w* 1.00 [0.85, 1.00], t=4.0 (fee-free, in-sample, five-model multiple comparison); real ask only 15 events, w* [0.00, 1.00].

**MLB_GAME: do not trust yet.** Only 29 ticker rows are cleanly verified; 34 rows have mixed provenance and disagree with Kalshi settlement on 16 of 34; a Kalshi-truth-only check on 63 ticker rows (43 events) shows NO edge (w* 0.61 [0, 1], WR 41.3% vs 48.1c). The home-team vs ticker-team YES frame is unverified against MLB Stats API scores; 163 MLB rows have no ticker. MLB is the one model that flipped, so selection risk applies. A separate defect is also open: the MLB backfill scores the game on the logged day rather than the ticker's game date (repair pending).

**Retractions (`claude/retraction_notices_20261003.md`).** The Sept 20 "NFL win rate: no bug found" investigation is withdrawn as guidance. The Sept 26 scorer-vs-dashboard cross-check was blind to a column-wide error because both read the same column; agreement between two readers of one input proves consistency, not truth. The Sept 24 CPI-backfill doc's NFL/MLS/MLB figures are wrong. My own Oct 3 line "I reproduced the Sept 30 numbers, so the rebuild is sound" only showed I used the same wrong input. Reproduction is not validation.

**Also shipped Oct 3.**
- `tools/outcome_invariant_check.py` (d3c9432): checks every scored ticker row against Kalshi settlement, independent of the backfills. Result: 0 blocking mismatches (CLAIMS 23/0, GDP 888/0, JOBS 243/0, MLS_GAME 309/0, NFL_GAME 432/0, NFL_SPREAD 2,102/0); MLB_GAME advisory only (46 ok, 17 bad). **Not yet wired to block publication** (plan: fail-closed on a definite mismatch only, test in daylight on both machines, then J@rv1s reviews).
- `tools/market_relative_report.py`: read-only real-ask report; `FEE_MODE` now supports `round1` (Kalshi per-order cent rounding, verified against the fee schedule effective July 7, 2026). Effect on net P&L per contract: NFL_GAME +2.64c -> +2.17c, NFL_SPREAD +0.75c -> +0.28c. No conclusion changes.
- Macro hand-check against official prints: GDP/JOBS/CLAIMS 66/66 tickers agree.
- MLB_GAME frame definition pre-registered before the audit (358d234), with verbatim Kalshi MLB rules text appended (23fbc15).
- Repair/report tools derive `ROOT` from their file location, so they run on Oracle as well as the laptop (63672cb).
- Calibration log updated (the log had not been touched since June 6).

---

## 6. SUN OCT 4: INFRASTRUCTURE AND THE AUDIT PACKET

**Oracle is now the sole pipeline runner.** On the laptop the scheduled tasks Full AM, Full PM, Refresh Noon, Refresh 4PM, Refresh 10AM Weekend and MLB AutoClose are Disabled; Mirror Pull is Ready; Auto Monitor is still Running (laptop-only). The Oct 3 21:00 CT, Oct 4 07:00 and 11:15 CT auto-update commits are authored by `Archie-OracleCloud`. The weekend 10 AM refresh was set for MLB and now needs NFL on Sundays; Rus's call: park it and revisit when MLB returns next season. **Not re-verified tonight:** that Oracle has pulled 9385fd7 (until it does, an Oracle backfill writes new NO rows in the old, wrong convention; the repair tool is idempotent, so re-run it after the confirmed pull).

**External audit packet (v1 -> v3.1).** Purpose: let three outside AIs from three providers tear apart the market-relative test and our power claims, without seeing any real data, results, project, exchange, sport or model names. Packet contains a method write-up, a simulation harness, results, and three one-page briefs (A: estimand, B: small-sample inference, C: microstructure/selection). J@rv1s read each version cold and found the deliberately planted error in each; v3 also exposed unintended defects (fee mismatch, incomplete results file, first-vs-last table run in the wrong regime, two interpretive sentences handing reviewers a conclusion). All fixed in v3.1 and J@rv1s approved sending after two harness edits. Details of the planted errors and the scoring key stay in the private answer key and are deliberately NOT in this public file.
- Reviewer-facing files are in the **private** `KalshiBot` repo under `evaluation/` (commit 46bbc5e), with three ready-to-paste per-brief files and a send checklist so Rus can use them from work. The answer key is not committed anywhere.
- Rules locked before sending: first response from each reviewer counts; no hints; verifiable findings trigger action regardless of reviewer count, other findings need two; the planted error is scored first.

**What the simulations say (synthetic model, untested, under review).** At about 27 flagged events, the pass test (CI lower bound > 0 on the ask basis) passes a forecaster that is half-real only about 8-19% of the time; roughly 350 flagged events are needed for 80% power if the forecaster's disagreement is persistent run to run, more if it is not. False-pass rate with no information is about 4-8% per model, which is about 46-63% chance that at least one of 12 tests passes by luck. Two-sided events (flagged on both sides at different times) lose about 1.2-1.9c per contract at every skill level. Implication, stated plainly: "no edge demonstrated" at current sample sizes is the expected outcome even for a real edge. The result is "not demonstrated at this sample size," not "no edge."

**Persistence of model-vs-market disagreement in the real log** (`claude/persistence_of_disagreement_real_log_20261004.md`; exploratory). 35-56% of first-two-row pairs for NFL_GAME, NFL_SPREAD, MLS_GAME and GDP are exact re-logs, not independent runs. NFL_SPREAD first-two-row correlation is 0.26 over all pairs, 0.06 without duplicates; sign flips 32%; the two diagnostics disagree on persistence, so no single number. Correlation is not the persistence parameter. An earlier claim that NFL_SPREAD might resolve this season was withdrawn.

**Phase 2 wording adopted** (J@rv1s's version, in `claude/phase2_power_wording_for_project_instructions_20261004.md` and the revised Gate 1 section file): "not demonstrated at this sample size," models stay at market weight until demonstrated, not redefined after the fact. Archie cannot edit project instructions: Rus pastes the revised section.

**Calibration log.** `data/jarvls_calibration.csv` in the private repo has 27 data rows. The public dashboard copy was updated (0659034) with Oct 3-4 entries, **withholding 4 audit-packet rows** until the review round ends so reviewers cannot find the scoring.

---

## OPEN POSITIONS (`sede_portfolio.json`, marks as of Oct 4 11:23 CT)

| # | Description | Entry | Current |
|---|---|---|---|
| 1 | BTC<$50k Dec31, NO | 43.5c | 92.5c |
| 3 | GDP>1.5% (Oct30), YES | 70.5c | 93.0c |
| 4 | GDP>1.5% (Jan28), YES | 74.5c | 67.0c |
| 5 | GDP>2.5% (Oct30), YES | 58.0c | 80.0c |
| 6 | GDP>1.0% (Jan28), YES | 78.0c | 78.0c |
| 7 | GDP>2.0% (Apr29), YES | 50.5c | 49.5c |
| 8 | GDP>4.0% (Oct30), YES | 18.5c | 21.0c |
| 9 | GDP>1.5% (Jul29), YES | 59.5c | 62.5c |

Bankroll $994.17 (started $1,000; one early-exit close at -$5.83; no wins or losses recorded; 8 open). Three positions (#3, #5, #8) resolve on the Oct 30 GDP print. GDP stays at reduced weight, and its real sample is n=1 resolved event, so these marks tell us nothing about the model. No new entries; nothing has cleared the entry criteria.

---

## GATE 1 / VALIDATION STATUS

- Project-wide Gate 1 remains **provisionally suspended**. The Sept 8 checkpoint was closed Sept 30 as "not resolved -- insufficient model coverage"; next per-model checkpoint **Oct 30 (GDP resolution)**; Phase 2 market-relative checkpoint **Nov 1**.
- **Corrected win-rate, P&L and bucket figures for NFL_GAME, NFL_SPREAD, MLS_GAME, MLB_GAME computed between Sept 24 and Oct 3 are suspect** (decision-impact audit of the dependent docs is pending; first-pass list is in `claude/jarvls_work_order_oct3_progress_20261003.md`).
- NFL_GAME: no demonstrated edge, entry weight 0. NFL_SPREAD: no edge in the extreme-disagreement tail. MLS_GAME and SOCCER_GAME: tail finding stands. MLB_GAME: see warning above; NO direction stays suspended. JOBS: n=4 verifiable events, unproven. GDP: n=1. CLAIMS: suspended; fix shipped Oct 1, evidence clock restarted, reinstatement replay not yet built.
- Dashboard staleness flags: CPI (116 days), GDP (66 days), SOCCER_GAME (96 days). Scorer cross-check: no mismatches, but see the shared-column blind spot above.
- Phase 2 (Nov 1) realistically closes "not demonstrated" for most models; that outcome is expected, not a failure of the process.

---

## WORK FOR TOMORROW (priority order, Archie's honest read)

1. **Rus (from work): send the packet** to three AIs (Briefs A, B, C; fresh chats, memory/web search/training off), save each first reply verbatim, bring them back. Files are in the private `KalshiBot` repo under `evaluation/`; start with its `README.md`. Archie scores the planted error first, then synthesizes with J@rv1s.
2. **Confirm Oracle has pulled 9385fd7**, then re-run the repair tool and `outcome_invariant_check.py` on Oracle.
3. **Before Oct 9:** confirm JOBS rows logged since Oct 1 carry bid/ask in `signals_log.csv` (first captured row Oct 1 21:00 CT; one row so far), and write the rules text for GDP / JOBS / CLAIMS (J@rv1s's work order). Check that CLAIMS emits correctly on Oct 7-8 and that Oracle has a FRED key.
4. Work-order step 1: re-run the Sept 22 stress test on repaired data. Step 2: fix the MLB backfill to score by the ticker's game date.
5. Independent MLB audit: all 230 MLB_GAME rows against MLB Stats API finals, to settle the YES frame.
6. Random-sample audit (30-50 rows) of the models Kalshi cannot check (WC_GAME, SOCCER_GAME, CPI, JOBS tickerless, MLB tickerless) against external results.
7. Written decision-impact audit (each dependent doc read and marked rerun / unaffected); wire the invariant into publication.
8. Rus: paste the revised Gate 1 section into project instructions (if not already done).
9. Standing dates: Oct 9 (JOBS), Oct 14 (CPI release), Oct 20 (target to lock the Phase 2 threshold in writing), Oct 30 (GDP), Nov 1 (Phase 2, and the Oracle DST cron adjustment). Four laptop stashes exist (data churn); do not drop them. Intraday stop-loss alerts (Auto Monitor) still run only on the laptop; decide whether to port to Oracle. J@rv1s's honest-commercialization item: the legal "signal service vs investment advice" research track has not been started; Rus's call on the free-beta soft-launch idea.

## ORACLE / SPORTS MONITORING

Oracle: running as sole pipeline runner, last confirmed auto-update Oct 4 11:15 CT. Sports monitoring: no open sports positions; nothing to monitor.

---

Archie | Papa Ralph standard.
