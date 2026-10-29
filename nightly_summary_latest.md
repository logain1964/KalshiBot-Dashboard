# SEDE Nightly Session Summary
## For J@rv1s Morning Intelligence Pull

**Covers:** Tue Oct 27 (no session) and Wed Oct 28, 2026, evening session (about 18:30 to 21:00 CT). Laptop clock in CT; Oracle clock and commit timestamps are UTC.
**Prepared by:** Archie (Claude Desktop)
**Replaces** the Oct 26 version of `nightly_summary_latest.md`. A copy is in the project as `claude/nightly_summary_latest_20261028.md`.

---

## THE SESSION IN ONE PARAGRAPH

No briefing from you tonight (Rus took the night off), so this session was Rus's own questions. The FORGE-Loop answer key is deleted from the shared project. Production is healthy since the matcher ship. Rus asked where the NFL stands, which led to three pieces of read-only analysis: the post-lock Phase 2 event count (my earlier estimate was wrong, see section 2), an extended market-relative report on four models (the model is worse than the market on Brier in every comparison), and an inventory of the whole model list. Nothing changed in any signal, weight, trading rule, manifest file or Oracle setting. No new calibration rows were logged tonight.

---

## 1. Production health since the matcher ship (merge 9a6ce82), read-only

- Every run from Oct 26 11:15 CT through Oct 28 11:15 CT checked: 0 tracebacks, matcher reason counts sum to markets, no trade entered.
- Freeze hashes unchanged: `daily_runner` `aa23eed2...`, `model_suspension` `9444a343...`. Manifest weight line unchanged (`"NFL_GAME":  0.55,`).
- New observation, low priority: NBA `date_mismatch=1` on `KXNBAGAME-26OCT30ORLTOR-ORL`, refused on every run since Oct 27 21:00. A refusal is the designed outcome; the cause is unknown.
- Tonight's `paper_trade_candidates.py`: 18 signals logged, 0 cleared every gate. `paper_trade_shadow_logger.py`: 0 new rows, 234 signals evaluated. Both run by hand with the full Python path; the scheduled version needs Rus to switch on "Require this computer" (it failed Oct 26 with no bound device).
- Oracle cron (UTC, no `CRON_TZ`): daylight saving ends Nov 1, so runs move from 07:00 / 11:15 / 21:00 CT to 06:00 / 10:15 / 20:00 CST. Rus chose to leave the crons alone. Report filenames will change to `_0600`, `_1015`, `_2000` from Nov 1.

## 2. Open positions and the NFL Phase 2 count

**Open paper positions (Oct 28):** one BTC NO position and seven GDP YES positions; bankroll $994.17 at the last confirmed snapshot. Three GDP positions sit on the Oct 30 print:
- `KXGDP-26OCT30-T1.5`: 42 contracts at 70.5c, now 96.5c.
- `KXGDP-26OCT30-T2.5`: 51 at 58c, now 77c.
- `KXGDP-26OCT30-T4.0`: 162 at 18.5c, now 4.5c.
I did not pull ticker-level detail for the other four GDP positions or the BTC position tonight. I also have not checked what produced the BTC position (see section 4).

**Phase 2 post-lock count (count-only; I did not compute Arm I or Arm P):**
- 25 NFL_GAME events with game dates after Oct 14 have been played and have logged rows.
- 16 more are logged for unplayed games: 12 on Nov 1 to 2 and 4 on Nov 8.
- **Correction:** I told Rus earlier I did not expect 30 post-lock events by Nov 1. That was wrong. With the Nov 1 to 2 games the count reaches about 37 if all twelve log and settle. This is events with logged rows; I have not confirmed each has a Kalshi settlement outcome and a real ask.
- **Needs a ruling from you:** once the count is 30 or more, `phase2_nfl_game_analysis.py` stops hiding the confidence intervals. The pre-registration says Nov 1 is counts and points only and the verdict is season end. I suggest we state now that the Nov 1 run reports counts and point estimates only and that nobody reads the interval bounds until the verdict. Otherwise it is optional stopping.
- Dashboard NFL_GAME, all-time and mostly pre-lock (descriptive): 96 events, 40.6% win rate, event-level Brier 0.2501 (a coin flip scores 0.25). Not the Phase 2 test.
- Verdict date stays Jan 14, 2027. Re-check Week 18 dates after Week 17 (Rus's own date line on the lock checklist is still blank).

League snapshot for context (Oct 28): Minnesota 6-0 is the only undefeated team and the Chargers are 0-6. Week 8 opens with GB vs CAR on Thursday; the Nov 1 slate includes DET-MIN, BUF-BAL and DEN-KC.

## 3. Model-vs-market evidence (relevant to your pre-Nov-1 failure rule)

Method: `market_relative_report.py` logic, round1 fee, 90% event-bootstrap, run from a temp copy so no file was overwritten. MLB_GAME excluded (unverified).

| Model | Events | Brier vs market | w* [5%, 95%] | Net per contract at real ask |
|---|---|---|---|---|
| NFL_GAME | 82 | -0.0139 (model worse) | 0.07 [0.00, 0.74] | +0.8c [-7.1, +8.9] |
| NFL_SPREAD | 104 | -0.0082 (worse) | 0.24 [0.00, 0.69] | -0.3c [-5.8, +5.3] |
| MLS_GAME | 71 | -0.0337 (worse) | 0.00 [0.00, 0.37] | no real-ask rows; at mid -4.2c |
| SOCCER_GAME | 97 | -0.0009 (tied) | 0.47 [0.00, 1.00] | no real-ask rows; at mid +4.1c [-2.4, +11.0] |

- Probability the model beats the market on Brier: 3 to 18% for NFL_GAME, NFL_SPREAD and MLS_GAME; about 47% for SOCCER_GAME. Every lower bound on w is zero, i.e. trade at market.
- Real-ask edge is gone after spread and fee: NFL_GAME win rate 43.5% vs break-even 42.7%. NFL_GAME at the real ask would need about 10,900 events for t=2, so a small true edge would be undetectable, not disproved.
- MLS and SOCCER have no real-ask rows (bid/ask capture began Oct 1). SOCCER_GAME's last resolved signal is Jun 30.
- Edge-bucket check (first row per ticker and direction, mid price, flagged side, 2,410 pooled rows excluding MLB): the model's stated probability is 7 to 20 points above the actual win rate in every bucket, and win rate does not rise with claimed edge. Win rate minus price paid is about +1 to +3 points, which spread and fee absorb.
- **I overstated one thing** earlier: I said the bigger the claimed edge, the more likely it is model error (winner's curse). The data does not show win rate falling with claimed edge; it shows claimed edge carries no information beyond price. The rows are not independent (NFL_SPREAD has 1,452 rows but about 106 events), so those standard errors are too small.
- **Unreconciled:** the NFL_SPREAD 30c-and-above bucket shows a 54.8% win rate vs 21% paid (31 correlated rows, possibly one or two games), which conflicts with the recorded tail finding. My basis (mid, first row) and dedup differ from the tail-finding analysis; I have not reconciled them.
- Caveat: rows come after the Oct 3 outcome-convention repair; I did not re-verify the repair. Bid, ask and edge sanity checks passed.

## 4. Model inventory

- Only NFL_GAME has an active confirmatory path. Suspended or log-only: CLAIMS (reinstatement referral), MLB_GAME, NHL_GAME, NBA_GAME, EPL_GAME; JOBS blocked on stale consensus; GDP at weight 0.60 with n=1 real event.
- Never resolved a signal in the staleness report: BTC, FED, EPL_GAME, NHL_GAME, NBA_GAME, NBA_CHAMP, NHL_CHAMP, SEEDS_MLB.
- BTC: an open paper position exists on a model with no resolved history (file last edited Jun 19). I have not checked what produced it. This is the item I would look at first.
- FED: `fed_model.py` was modified Oct 25 (I have no note on why in this session). I believe the FOMC decision is Oct 28; I did not check whether FED logged a signal.
- Possibly dead code, not in the dashboard: `world_cup_model.py` (Jun), `nba_series_model.py` and `payrolls_model.py` (May). Archive or keep.
- Staleness flags: GDP 90 days (Oct 30 resolves it), SOCCER_GAME 120 days.
- The dashboard's `edge_buckets` and `signal_dist` fields are empty.
- Pattern worth stating plainly: the models with the largest samples (NFL_SPREAD, SOCCER_GAME, MLS_GAME, MLB_GAME) have the weakest evidence; the best-looking numbers (CPI 84.8% on 25 events, event Brier 0.1407) come from samples whose events are probably strike levels within a few releases. I have not verified the CPI dedup.

## 5. Book-vs-Kalshi poller idea (held, not proposed)

Rus asked whether a separate process could compare sportsbook moneylines to Kalshi (not a swap of NFL_GAME). Read-only findings:
- `cross_book_gap_log.csv` is written from `daily_runner.py` (about line 2794); NHL only, 193 rows. NFL_GAME is an Elo model and uses SharpAPI only as a secondary check.
- Pointing the existing sidecar at NFL means editing a frozen file. A standalone log-only script that imports its helpers avoids moving the manifest hashes.
- Per June code comments the SharpAPI free tier is 12 requests a minute with two books (DraftKings, FanDuel); I did not call the API to re-verify.
- In the sidecar rows the odds fetch ran about 6 minutes after the Kalshi snapshot, which would blur any lag-edge study.
- Early NHL gaps were small (mean 0.6c, max 1.2c), inside the fee.
- Rus said hold. It has not been written up. If you want it, the slot-rule question (does a log-only sidecar count against one-in-one-out) is yours to rule on.

## 6. Housekeeping

- Rus deleted `jarvls_forge_loop_test_answer_key_RUS_ONLY_20261026` from the shared project (doc count 207 to 206). I did not open it. You may still hold the original in your own chat.
- `session_state.md` (laptop only, excluded from git) now carries the Oct 28 entries. Rus still has to paste the read-first and append-habit lines into `Archie_Session_Protocol`.
- Rus's SSH-key decision stands (deferred); not re-raised.

## 7. Open, by owner

**For you:** draft the "what would show SEDE does not work" rule before Nov 1 (Rus said yes); rule on the Nov 1 counts-and-points-only run; weather work order not before Oct 31, kill criteria first.
**For Rus:** switch on "Require this computer" for the Paper Trade Candidates scheduled task; paste the running-log lines into `Archie_Session_Protocol`; tell you the answer key is out; fill in his verdict-date line.
**Dated:** Oct 30 GDP print (three open YES positions on it); Oct 31 score the first-week matcher row (8%) after the 21:00 CT run; Nov 1 Phase 2 checkpoint and the end of daylight saving; week of Nov 2 MLB_GAME audit; about Nov 2 to 3 JOBS survey and Nov 6 print; Jan 14, 2027 verdict.
**Carried:** NHL ratings hardening; heartbeat dead-man's switch; entry-shrinkage wiring after the lock; cache-bypass script and Elo determinism test; ticker-level detail for open positions and NHL signals in these summaries (your request; partly done above); BTC and FED checks; dead-file decision; four laptop stashes (do not drop).

Archie | Papa Ralph standard.
