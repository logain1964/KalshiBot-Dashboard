# SEDE Nightly Session Summary
## For J@rv1s Morning Intelligence Pull

**Covers:** Wed Oct 7 (evening) into early Thu Oct 8, 2026 | **Written:** about 10 PM CT Oct 7 (laptop clock); the session ran late, so the file date is Oct 8. Oracle commit timestamps are UTC (the 21:00 CT run shows as 02:08 on Oct 8).
**Prepared by:** Archie (Claude Desktop)
**Replaces** the Oct 6-7 version of `nightly_summary.md` and `nightly_summary_latest.md` (identical content pushed to both). That version is in git history and in the project as `claude/nightly_summary_latest_20261007.md`; everything in it that still stands is carried forward below.

---

## THE NIGHT IN ONE PARAGRAPH

Two real bugs were found and fixed tonight, and Oracle's own output already confirms both fixes ran. The CLAIMS model was mapping a Thursday release to the wrong FRED week (release date plus 2 days instead of minus 5), so the Oct 8 release would have been skipped as a "Columbus Day" holiday week. The run manifest was writing two lines per run instead of one. Both are fixed, pushed and deployed (commits 35542be and d886afb). **Nothing changed in any signal, model weight or trading rule**, and the NFL_GAME call-site weight line is still 0.55 in every manifest line. The project instructions were revised and pasted by Rus tonight and the stored text was read back and verified.

---

## 1. VERIFIED FROM ORACLE'S REAL OUTPUT (21:00 CT run, commit 61f8f1e)

- **Manifest is live.** `data/run_manifest.jsonl` has 5 lines: Oct 7 07:00:03, 07:05:19, 11:15:02, 11:23:13 and 21:00:02 CT. The first two pairs are the double-line bug (two lines per run); **the 21:00 run wrote exactly one line**, so the guard (d886afb) works. Each line records the pipeline commit; the 21:00 line shows 35542be, so **Oracle ran the CLAIMS fix**. `nfl_game_weight_lines` is `"NFL_GAME": 0.55` in all five; fee mode `round1`; 99 packages recorded; no tracked file dirty.
- **MLB label emits.** `unverified_models` is present in `brier_dashboard.json` in both the private `data/` copy and the public dashboard copy (J@rv1s's MLB condition 1 is confirmed on Oracle's real output).
- **CLAIMS fix reached its model.** The 21:00 report prints the CLAIMS section with the suspension notice and "Release day... no CLAIMS signals this run"; there is no HOLIDAY WEEK skip line. The last CLAIMS row in the signals log is Oct 1, so no CLAIMS rows were logged Oct 2-7 (consistent with the bug: the next release mapped onto the Columbus Day key). **The first real post-fix emission is the Oct 8 07:00 CT run.**
- **Observation, not yet diagnosed:** the 21:00 CT report treated Oct 7 as "release day" for the Oct 8 print, which suggests Oracle's date is UTC at that hour. Rus to confirm `date` on Oracle; matters for any date-keyed logic near the cron times.
- **Not mine:** the 21:00 auto-update also includes an edit to `models/fed_model.py` (the Fed consensus table refreshed, consensus date Oct 4 to Oct 7), made on Oracle and committed by its auto-update. NFL files unchanged.

## 2. CLAIMS WEEK-MAPPING FIX (commit 35542be, pushed)

- A Thursday release covers the week that ended the preceding Saturday, so `fred_week_ending = release_date - 5 days`, not `+ 2`. Arithmetic and comment only; holiday tables untouched. Offline check: Oct 8 maps to Oct 3 (normal), Oct 15 to Oct 10 (holiday), Oct 22 to Oct 17 (aftermath). CLAIMS stays suspended; no trading change.
- **Open ruling for J@rv1s, before the Oct 15 release:** under the corrected mapping, the Oct 15 release is skipped as "Columbus Day" although Columbus Day (Mon Oct 12) falls in the week ending Oct 17 (released Oct 22, labelled "aftermath"). Should Monday holidays key to the preceding Saturday week, or to the week containing the holiday?

## 3. ENTRY-EDGE WIRING (J@rv1s's question, answered)

**The Oct 1 shrinkage decision (CI-lower-bound weight) is not wired into the live gate.** `paper_trade_gates.py` uses a raw 15c minimum edge; SEDE portfolio admission uses a raw 20c minimum; neither applies a shrinkage weight. Every model the finding matters for is trade-suspended, so there is no live consequence today. Recommended: wire it as a small separate commit after the Phase 2 lock. Full reply in the project: `claude/archie_to_jarvls_wiring_claims_20261007.md`.

## 4. STALE CONSENSUS VALUES (the "Manual update required" line)

- **CPI (`models/cpi_model.py`)**: the September entry (3.7%, std 0.40) dates from Sept 26 and needs a refresh before the Oct 14 release; the hard block falls Oct 17. Reference points found Oct 8: Cleveland Fed nowcast 3.60% (updated 10/06), Kalshi year-over-year market median about 3.6% (84% for above 3.5%, 42% for above 3.6%), one forecaster at 3.7%. No sourced Street survey figure found. Rus to read the consensus from an economic calendar around Oct 10-12; Archie then edits the `'SEP'` entry and the `CONSENSUS_LAST_UPDATED` date and pushes before Oct 14. The October placeholder (2.8%) is a 0.9-point drop from September and is not sensible; fix by about Nov 2.
- **NFP / JOBS (`models/jobs_model.py`)**: still the September setup (interim 70K estimate from Sept 7, not a survey; report date Oct 2). JOBS is blocked by the freshness gate. Leave it blocked until a real survey exists (about Nov 2-3 for the Nov 6 print); refreshing the date with a guess would only defeat the gate. Possible design gap, unverified: the freshness check reads only the date in the file, so a live feed value may not clear the block. Check before November.

## 5. PROJECT INSTRUCTIONS (Rus pasted tonight; verified by read-back)

Revised: Phase 2 bullet (NFL_GAME sole primary, two arms, Oct 13 lock, Oct 20 hard stop, Nov 1 checkpoint only, "unresolved" the likeliest outcome and not the same as "no edge"); NFL_GAME line refreshed to the 42-event figures; shrinkage bullet notes it is not yet wired; MLB_GAME fully suspended in both directions (condition 1 now also confirmed on Oracle's output, see section 1); CLAIMS fix and pending holiday ruling; NHL_GAME/NBA_GAME matcher window Oct 14-19; JOBS stale-consensus note; the one-in-one-out admission rule added. The Oct 4 power sentence (8-19% at 27 events) was not restored: those figures described the old w* test, not the two arms now in force.

## 6. J@RV1S'S OCT 7 BRIEFING, STATUS

- **One-in-one-out admission rule** (ratified Oct 7): in the instructions now. Weather-market and crypto-threshold work stay scoping-only until a slot opens (earliest about Oct 19-20, only one of the two).
- **Weather-market work order:** needs Rus to provide read-only Kalshi API access; it stays scoping-only regardless of what it finds. The Oct 2 doc's "free, no auth" claim for Kalshi's series endpoints was reported wrong by J@rv1s (401 without a key); correction to that doc is open.
- **Housekeeping:** one private item for Rus, details kept in a private project doc, not here.

## 7. NFL_GAME PHASE 2 / PRE-REGISTRATION (v0.3, draft)

Unchanged tonight apart from the manifest now confirmed live. Design data (42 events, not confirmatory): Arm I -0.0086 CI [-0.0422, +0.0272]; Arm P +3.64c per contract CI [-8.17, +15.93]; unresolved; w* 0.23 [0.00, 1.00]. **For the Oct 9 final**, J@rv1s's addition to the lock checklist (section 13): the calibration leak was found and closed before the lock (private-file fix abd5139; exposure-check resolution 882b06d). Lock target Oct 13 (needs one full day of manifest lines by Oct 12: the 07:00, 11:15 and 21:00 CT runs of one date; Oct 8 will be the first full day of single lines); hard stop Oct 20.

---

## OPEN POSITIONS

`sede_portfolio.json` (per the Oct 6-7 read): 8 open (BTC<$50k NO; seven GDP YES on the Oct 30, Jan 28, Apr 29 and Jul 29 prints), bankroll $994.17, one early exit at -$5.83. `paper_trades.json` (J@rv1s, Oct 7): 1 open, Trade #25, GDP >T3.5 YES; the two sources disagree as before (see the Sept 25 `manual_trading_stall_investigation`). Both files were re-written by the 21:00 auto-update; marks not re-pulled tonight. No new entries; nothing cleared the entry criteria. GDP stays at reduced weight (n=1 real event; next print Oct 30).

## GATE 1 / VALIDATION STATUS (carried forward)

Project-wide Gate 1 provisionally suspended; checkpoints Oct 30 (GDP) and Nov 1 (Phase 2 checkpoint, report only). NFL_GAME: no demonstrated edge, entry weight 0. NFL_SPREAD, MLS_GAME, SOCCER_GAME: tail finding stands. MLB_GAME: fully suspended, all figures unverified. JOBS: n=4 verifiable events, unproven. GDP: n=1. CLAIMS: suspended; fix now deployed, first real emission Oct 8. Win-rate, P&L and bucket figures computed Sept 24 - Oct 3 for NFL_GAME, NFL_SPREAD, MLS_GAME and MLB_GAME remain suspect; decision-impact audit pending.

## WORK AHEAD (dated order)

- **Thu Oct 8:** CLAIMS first real emission at the 07:00 CT run; recheck the Oct 2 JOBS ticker gap.
- **Fri Oct 9:** pre-registration final to J@rv1s (with the leak-closed line above).
- **Oct 10-12:** CPI consensus read and edit; by Oct 12 one full day of manifest lines (Oct 8 qualifies if all three runs write one line each).
- **Oct 13:** lock target. **Oct 14:** CPI release; tickerless random-sample audit starts. **Oct 14-19:** sync-gate commit, then the matcher (ship by Oct 19 or slip). **Oct 19-23:** decision-impact audit. **Oct 20:** hard stop for the lock. **Oct 30:** GDP print. **Nov 1:** Phase 2 checkpoint. **Week of Nov 2:** MLB backfill game-date fix and 230-row audit.
- **Rus:** (1) confirm CLAIMS output after the Oct 8 07:00 run; (2) confirm Oracle's `date` and timezone (observation in section 1); (3) CPI consensus number around Oct 10-12; (4) the private housekeeping item; (5) read-only Kalshi key for the weather work order, when a slot opens; (6) one-minute public lock-timestamp line at the lock.
- Not done: sync denylist gate, D4 join, tickerless audit, decision-impact audit, matcher. Standing: four laptop stashes exist (data churn), do not drop them; Auto Monitor stop-loss alerts still laptop-only; session-opener items (live trade health check, `auto_monitor.py` check on the laptop, session-archive read) not run tonight.

## ORACLE / SPORTS MONITORING

Oracle: sole pipeline runner; auto-pulls the private repo (fast-forward only) at the start of every run. Verified Oct 7: 07:00, 11:15 and 21:00 CT auto-updates all present; the 21:00 run executed commit 35542be. Sports monitoring: no open sports positions.

---

Archie | Papa Ralph standard.
