# Nightly Summary - Archie - Thu Nov 5, 2026 (written 23:35 CST; replaces the Nov 4 version)

Read-only on production except one JOBS constants commit. Nothing pushed to Oracle, no outcome statistic computed on any data, weather work is prices only.

## Done tonight (Judge evidence)
1. JOBS entered. Judge evidence: I opened the FactSet page in the built-in browser and quoted it - "Total Nonfarm Payrolls for October 2026 Are Projected to Rise By 65,000", John Butters, Nov 5, 2026; NFP median 65,000 from 24 estimates (50,000 to 125,000); unemployment median 4.2% from 25 estimates (4.1% to 4.3%). Commit bdd1b64 on origin/main, pushed 23:02 CST (about 7 hours before the 06:00 CT run); origin file shows NFP_CONSENSUS 65_000, UNEMP_CONSENSUS 4.2, REPORT_DATE 2026-11-06, CONSENSUS_LAST_UPDATED 2026-11-05. Freshness check on the edited file: 10.99 hours, no stale flag. I also updated REPORT_DATE and REPORT_MONTH (the file's own instruction; the past-release branch keys on REPORT_DATE). jobs_model.py is not a Phase 2 hashed file. JOBS stays UNPROVEN (n=4), not a recommendation.
2. Opener proof and the 3 questions answered in chat and restated in claude/archie_to_jarvls_era_audit_sims_cost_20261105.md. Paper Trade Candidates reads only C:\KalshiBot\data\signals_log.csv (laptop path), so it needs "Require this computer". session_state.md is C:\KalshiBot\session_state.md, exists. No PreCompact hook is configured; Sept 7 was a compaction-boundary event per my own Sept 7 archive.
3. Public repo: all 285 MLB rows match the repaired outcomes (actual_outcome only, 4-col key, as of Nov 4 20:00 CT). Your project-instruction line "public repo NOT updated (check pending)" is stale; I need replacement text from you.
4. Project instructions: B5 text pasted by Rus and read back by me from the live project (both edits present; compared by reading, not byte-diff).
5. Push log claude/archie_push_log_mlb_repair_20261103.md updated.
6. Built paper_trade_nightly.py (SHA256 af73d93f...9630) and registered Windows task Paper_Trade_Nightly (daily 20:35 CT, wake-to-run) at Rus's explicit go. It runs the same two scripts as the cloud task and Telegrams Rus every night, with a same-event concentration warning. Live test reached his phone. Cloud task stays enabled as a cross-check for a few nights. Not committed to git yet. Local branch is mlb-backfill-fix-20261102 tracking origin/main; local main is 13 behind; I push with HEAD:main.

## Weather (option a): prices only. Full detail with hashes: claude/archie_to_jarvls_era_audit_sims_cost_20261105.md
- Panel built: 4,008 markets, 668 events, 147,036 candle rows, 0 fetch failures; panel_prices.csv has no outcome column; outcomes sealed, read-only, never opened.
- Era audit: 668 events = 668 consecutive dates 2025-01-05 to 2026-11-03, no gaps, no duplicate dates (your 70% claim: TRUE). Fields consistent across eras. Early books are thin: D05:00 nearest-bracket spread median 16c (2025Q1), 17c (2025Q2), then 4, 2, 2, 2, 1, 1c.
- Full-pool cell coverage replaces the live-pool numbers: D05:00 635/668 usable (95.1%), 386 in the 40-60c band (57.8%); D01:00 654 usable, 392 in band (58.7%) now leads by 6 dates; D03:00 and D04:00 only about 70% have a candle and fail the 75% line.
- Measured cost line C (half-spread + round1 fee, re-implemented from brier_dashboard.py): pooled median 3.0c (D05) / 3.5c (D01); by era 2025H1 10.5c / 6.5c, 2025H2 3.5c, 2026 2.5c. External Kalshi fee-schedule citation still owed.
- Rehearsal (Y simulated from the real m panel, 1,000 runs, wider of date and 7-day block bootstrap): calibrated market reads MISCALIBRATED 0.0-0.5%, GAP 8-10%; true 5c gap reads MISCALIBRATED about 36-40% at C=2.5c (17-27% at the measured C); true 8c gap about 76-91%. Interval half-width about 3.2c at n of about 640. Real SD of Y-m 0.46-0.49.
- Decisions needed from J@rv1s (v0.3 due Sat Nov 7, lock Wed Nov 11, reservation lapses Mon Nov 16): (1) pooled vs era-restricted Stage 1 (2025H1 costs 6.5-10.5c); (2) cell D05:00 vs D01:00; (3) C source (2.5c vs measured) and the fee citation.

## Not done (carried)
Re-run of the compare script and rules/MLS-404 probe with saved output and hashes; RESOLVED_MARKETS audit (top 20 + 20 random) and direction sample (30 NFL_GAME + 30 MLS_GAME); id-3 greps (propose max(id)+1 diff only, no renumber); silent-fetch-error read-only check (NFL and MLS first); MLS Oct 25 two-row trace; 285 vs 230 MLB row count reconciliation and brier_score/log_loss check; calibration rows into jarvls_calibration.csv (not written; schema not seen); Jan 14 blank question (is the lock checklist hash-covered?).

## Rus to-dos
Disable the cloud Paper Trade Candidates task after the laptop job has matched it for a few nights (Telegram arrives around 20:35 CT; no message by 20:45 means tell Archie). Replace-the-line decision on the stale public-repo instruction after J@rv1s supplies text. SEEDS decisions (stop tracking seeds_bridge_state.json; unvalidated label) remain Rus's call.
