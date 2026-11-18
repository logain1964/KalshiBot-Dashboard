# J@rv1s EOD Briefing, Wed Nov 18, 2026 (written 16:55 CST)

One consolidated briefing. No addenda. SEDE-SEEKS only; non-SEDE work excluded at Rus's request.

## 0. PASTE TO ARCHIE (project docs, exact paths, in this order)
1. `claude/jarvls_eod_briefing_20261118.md` (this briefing)
2. `claude/jarvls_calibration_scored_ncei_list_accepted_20261117.md` (the g-with-and-without method rules and the calibration commit terms; Archie's nightly summary should already carry the rest)
The Nov 17 EOD briefing (`jarvls_eod_briefing_20261117`) is carried forward here; Archie does not need it again.

## 1. Bottom line
- **Nothing new happened in SEDE on my side today, and the project shows nothing new from Archie.** The newest SEDE project doc is my Nov 17 ruling (18:01 CST). No Archie note, nightly summary or ruling is dated Nov 18. I did no SEDE analysis today; my session work was non-SEDE and is excluded.
- **Not evidenced as done (do not assume they happened):** the Wednesday calibration commit, the Oracle read-only log check, and Archie's g-with-and-without method proposal. If any of them happened in a session I did not see, it is not in the project record, so it is unverified to me.
- State is unchanged from Nov 17: **KXHIGHLAX Stage 1 label UNRESOLVED** (g +2.80c, 90% [-0.19, +5.83]c, n 667, C 3.45c); series closed NOT DETECTED at the stated n, slot released, no collector, no Stage 2. That is not "calibrated" and not evidence for or against Thesis H. Nothing in SEDE is demonstrated. Phase 2 NFL_GAME verdict Thu Jan 14, 2027 (Ruling 1a: counts and points only until then). Falsification rule v0.2 unchanged.

## 2. Oracle Cloud status
Carried, not re-verified. Last confirmed: Archie's summary closing Sun Nov 15; sole pipeline runner; auto-pulls before the 06:00 / 10:15 / 20:00 CT runs; no signal, weight or trading-rule change; seeds copy clean at the Sept 24 commit. **Runs since Nov 15 are unverified, including today's 06:00 and 10:15.** The 20:00 CT run is still ahead tonight. Rus runs every Oracle command; nothing pushed or pulled around the auto-updates. The one read-only log check (tracebacks, manifest hashes, freeze, positions, runs since Nov 15) is still owed.

## 3. Open positions
Last confirmed, marks not re-pulled: open GDP positions on the Jan 28, Apr 29 and Jul 29, 2027 releases (unrealized, thin markets, not evidence); the BTC NO leftover (no BTC signal since Sep 21). `sede_portfolio.json` and `paper_trades.json` differ by the known explained gap. No new entries; nothing has cleared the entry criteria. Criterion 4 caveat: `auto_monitor.py` covers only `paper_trades.json`.

## 4. Validation tracker (unchanged, none re-read today)
- **NFL_GAME:** no demonstrated edge, entry weight 0, accumulating, verdict Jan 14, 2027.
- **GDP:** n_events = 2, Gate 1 not met; next event Jan 28, 2027.
- **JOBS:** unproven; the Nov 6 print has passed; state not re-read. **CPI:** blocked until a real survey median with a source link; not re-read.
- **MLB_GAME:** fully suspended; report only the verified tickered subset (61 rows, 24 wins 39.3%; 44 games, first row per game 19 of 44, 43.2%), descriptive; offseason.
- **CLAIMS:** suspended. **NFL_SPREAD, MLS_GAME, SOCCER_GAME:** tail finding stands (SOCCER_GAME rests on the unverified hand table). **NHL_GAME / NBA_GAME:** log-only, suspended, no pre-registered test.
- **Weather (KXHIGHLAX):** Stage 1 UNRESOLVED (section 1). Daily rain unresolved, count re-check Dec 1 only. Gate 1 project-wide provisionally suspended since Aug 11.

## 5. Calibration
Seven rows scored Nov 17, **local and uncommitted as far as the record shows**. Hashes after scoring (from Archie's Nov 17 note): jarvls `8C65B5A646340D5E349CC3233FB509CDE363841BBC4C59C712618BA38CE2A9E9`, archie `E4645B1FC8697E47D747900A7A0003631F93BB27793FF7A8354F8F6D22A7AF5C`. Six jarvls rows are one label outcome and two process facts, not six trials; the Oct 31 55% row (Brier 0.3025) is a badly overconfident miss inside that; mean 0.066 is not calibration. Line 80 (Judge-line, 55%) stays PENDING to Nov 19. I have not seen the files themselves, only Archie's note. I verify the published `jarvls_calibration.csv` after the push.

## 6. Sports monitoring
No sports positions known open; not checked.

## 7. Work order for Archie
**Tonight, only if Rus opens a session:**
1. **Commit window.** The ruled terms stand: pull first, merge not rebase, commit the two calibration CSVs and nothing else, seeds state file out. First confirm both files still hash to the values in section 5; if either differs, stop and report. Because the 20:00 CT run is coming, push only if it is done by about 19:30 CST (my margin, not a ruled number); otherwise wait until after the 20:00 run has finished and do it Thursday after the 10:15 CT run. A one-day delay costs nothing.
2. **Oracle log check (Rus runs it):** after a run has completed, one read-only pass, same items as section 2.
3. **g-with-and-without method proposal:** send it as a doc; nothing runs before I rule. The driver must first reproduce n 667, g +2.80c, [-0.19, +5.83], yes-rate 48.3%, mean m 45.5%, C 3.448c at seed 20261114, B 5000, or stop and report. The second read-only sealed exception is decided when I rule on the method.
4. **Do not touch:** alert patch (waits for my ruling; the DATA-block `_safe_text` fix stays its own later commit), Phase 2 hashed files, scheduling, SEDE code, new models. No admission to the empty slot.

**Judge:** the calibration commit contains the two CSVs and nothing else, with both pre-commit hashes quoted and matching; the g-with-and-without driver reproduces the printed primary digits before it reports any row.

## 8. Dates
Thu Nov 19 Judge-line review (mine; include the source-independence criterion Rus asked for on Nov 6, and confirm it is in the Nov 6 EOD). Mon Nov 23 income-side decision date (parked, not chased). Dec 1 daily-rain count re-check only. Thu Jan 14, 2027 Phase 2 verdict. Thu Jan 28 GDP print. Jan 31 falsification-rule checkpoint and sunset date.

## 9. Honest flags
- **Two days now with no verified SEDE movement on the open items.** That is fine for a quiet stretch, but the Oracle runs since Nov 15 have been unchecked for three days. Do the log check before anything else gets built on top of them.
- **The slot is empty and nobody is entitled to fill it by reflex.** Admission is a separate decision under one-in, one-out; crypto-threshold-vs-Deribit and the book-vs-Kalshi poller stay queued, Rus's hold stands.
- **Decision owed to Rus before Jan 14:** the sunset date is Jan 31 unless a second pre-registered test is locked by then (which moves it to Jun 30, 2027). Stage 1 does not count as that test. If NFL_GAME is not demonstrated on Jan 14 and no candidate exists for a second test, Jan 31 sunset is the default. Name this early so it is a choice and not a surprise.
- **The near-miss is a trap.** The lower bound is -0.19c; a slightly different sample gives GAP BELOW COST, which fires about 8% on noise alone. The 2026-only row (+3.71c) and price-bin yes-rates are unlabelled, correlated, not pre-registered. Nothing is built on them.
- **Mistakes of mine carried from Nov 17, already corrected in the saved rulings:** the estimated "~17:50" timestamp in my clear-to-run doc; the Oct 31 row missing from my first scoring table; the double-named column on archie line 15.
- **Not re-read by me:** Oracle runs since Nov 15, JOBS after Nov 6, CPI, laptop repo state (last seen Nov 16: branch `mlb-backfill-fix-20261102`, HEAD `9c2c36d`, dirty files `data/seeds_bridge_state.json` plus the two locally edited calibration CSVs).
- **My bias to watch:** a quiet day with a clean Nov 17 behind it feels like progress. It is not; the wait for Jan 14 is the real test.

J@rv1s
