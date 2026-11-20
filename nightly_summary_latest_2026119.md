# Nightly summary - Archie - Thu Nov 19, 2026 (session ended 23:05 CST clock read)

## 1. State in one line
Step B v2 ran live once and passed; J@rv1s ACCEPTED it. No label moved: Stage 1 stays UNRESOLVED (g +2.80c, 90% [-0.19, +5.83]c, n 667, C 3.45c), series closed NOT DETECTED, slot released. Nothing in SEDE is demonstrated.

## 2. Step B v2 live run (22:28:49 to 22:28:51 CST, exit 0)
- Pre-run checks (clock 22:28:39): git status only data/seeds_bridge_state.json dirty; script SHA256 2650BB2AD4BE7F57C7906407E776DF24550A85B17E2FEC1EA00071333E41F282 (all 64 equal); Python 3.12.10 and numpy 1.26.4 equal to the Nov 17 pre-run note (pytz not printed); output file did not exist.
- Sealed access: outcomes_sealed.csv only (110,238 bytes, SHA256 03E36E51...17A4 equal to its SHA256SUMS line); register exactly two opens of that one path ('byte read for hash', 'read_outcomes open'); markets_raw_sealed.json never opened or hashed.
- ACCEPTANCE 1 PASS, ACCEPTANCE 2 PASS (character for character vs the saved Stage 1 lines).
- Sensitivity row (no label): WITHOUT 2026-10-22 n=666, yes-rate 48.2%, mean m 45.5%, g +2.73c, 90% [-0.28, +5.80]c, C 3.45c. Dropped set: 1 NEAR-EDGE, 0 CLEAR. Printed g is INSIDE the sanity bound [2.65, 2.81]c. Primary label stays UNRESOLVED whatever it shows.
- Output file gwwo_stepB_v2_live_output_20261119.txt: 3,975 bytes, SHA256 1F6C3580624079485E0946F25B93200ED1102AC01D2FFE65931FA866445364F6. The posted copy is a transcription; the disk file and hash are authoritative (C:\Claude AI\weather_forward_record is not a connected folder, so it could not be staged byte-exact).
- Sealed-read register to date (outcomes_sealed.csv only): Stage 1 run Nov 17; NCEI cross-check Nov 17; Step B v2 Nov 19 22:28:49 (script hash 2650BB2A...F282, two opens in the one run). Windows may update last-access times on the sealed file; I have not inspected the sealed folder.
- Docs: claude/archie_to_jarvls_gwwo_stepB_v2_live_result_20261119.md, claude/gwwo_stepB_v2_live_output_20261119.txt.

## 3. Late Nov 18 summary
The Nov 18 summary was written to the laptop and pasted in chat but not posted to the project that night (Archie's miss). Posted Nov 19 as claude/nightly_summary_latest_20261118.md. J@rv1s has not yet read it and will reconcile it with the EOD tomorrow. Note: C:\KalshiBot\tmp\nightly_summary_latest.md is the laptop copy; the project copy is the one J@rv1s reads.

## 4. Oracle read-only pass (Rus ran it ~22:45 CST; Oracle clock Nov 20 04:45 UTC)
- HEAD ca1ec25 "Auto-update 2026-11-19 20:00 CT"; tree clean.
- Open positions: 5 in sede_portfolio.json. BTC NO id 1 (KXBTCMINY-27JAN01-50000.00, entered Jun 20); GDP YES ids 4 (Jan28 T1.5), 6 (Jan28 T1.0), 7 (Apr29 T2.0), 9 (Jul29 T1.5), entered Jul 30 to Aug 1. None entered after Aug 1. Entered since Nov 15 (open + closed): 0. Open trades 0/8 in all 15 reports (paper_trades.json, 25 closed).
- Commits since Nov 15 that are not Auto-update: only d87571b (calibration). models/nfl_model.py full 64-char match to 3eceaf6dd376d884b97f1ecadf841f800180d1f0f3da00fc84e99454341240fb.
- Reports: 15 daily reports, Nov 15 06:00 through Nov 19 20:00, none missing. (I said 13 for a while; that was a truncated listing. 15 is right.)
- 429 lines per report: 21 to 27, none zero (retry attempt numbers checked only in the Nov 19 20:00 report: all "attempt 1/5").
- Matcher date_mismatch: NBA 1 (Nov 15 20:00) then 2 each in Nov 16 06:00, 10:15, 20:00, each with an ALERT line; NHL 1 each in Nov 17 06:00, 10:15, 20:00, no ALERT line; none in either on Nov 18 or 19. no_odds_game is the largest reason bucket; game_started appears in 20:00 reports.
- SUPPRESSED_SIGNAL: 9 lines over 15 reports (Nov 15 10:15: 2; Nov 15 20:00: 2; Nov 16 20:00: 2; Nov 19 20:00: 3, the IND@HOU lines for MLS_GAME, NFL_GAME, NFL_SPREAD with reason=ESPN). Reason text seen only for the Nov 19 lines. Flagged for J@rv1s; cause not known to Archie.
- Nov 19 20:00 report freshness: market_cache healed (543 markets); cpi_consensus BLOCK (CPI blocked, last 2026-09-26); nfp_consensus WARN (last 2026-11-05); mlb_schedule WARN. Not interpreted.
- For J@rv1s: the ESPN suppressions, and whether the NBA (Nov 15-16) and NHL (Nov 17) date_mismatch runs need his eye. Matcher was declared STABLE on operational grounds only.

## 5. GitHub token
Rus will rotate the token and re-point the C:\Claude AI remote THIS WEEKEND. Until then: no git add in C:\Claude AI, no remote commands there, never print the remote URL (it holds the token), do not copy the token anywhere. After rotation Rus tells Archie; then Archie adds weather_forward_record/ to .git/info/exclude (create the replacement token before revoking the old one if anything else uses it).

## 6. Next
- J@rv1s, first thing Fri Nov 20: Judge-line review (overdue a day; scores calibration line 80), reads the late Nov 18 summary and this one, rules on the Oracle items above.
- Step B is done; EOD work order items 1 and 2 dropped.
- Rus: token rotation (weekend), then tell Archie for the exclude entry.
- Dates: Mon Nov 23 income-side decision (parked); Dec 1 daily-rain count re-check only; Thu Jan 14, 2027 Phase 2 verdict; Thu Jan 28 GDP print; Jan 31 falsification checkpoint.
- Untouched by order: alert patch, Phase 2 hashed files, scheduling, SEDE code, new models, the empty slot.

## 7. Process notes
Estimates are never used as times; clock reads only. Posted script/output text is a transcription of disk; hashes of disk files are the authority. My earlier slip this session: block 5b of the Oracle commands picked cron_env_diagnostic.txt by mtime instead of a report; corrected with the next block.
