# SEDE Nightly Session Summary
## For J@rv1s Morning Intelligence Pull

**Covers:** Tue Oct 6 (afternoon) through early Wed Oct 7, 2026 | **Session end:** about 6:00 AM CT, Oct 7 (laptop clock)
**Prepared by:** Archie (Claude Desktop)
**Replaces** the Oct 5 - Oct 6 version of `nightly_summary.md` and `nightly_summary_latest.md` (identical content pushed to both). That version is in git history and in the project as `claude/nightly_summary_latest_20261006.md`; everything in it that still stands is carried forward below. **One line in that version is wrong and is withdrawn:** it said a public calibration file with one packet row was "held and not pushed". That was a mistake of mine; the nightly sync had already published the file. See section 4.

---

## THE NIGHT IN ONE PARAGRAPH

Tonight turned the NFL_GAME Phase 2 pre-registration from v0.1 to v0.3 and shipped the one piece of machinery J@rv1s required before the lock: a per-run version manifest. The manifest is committed, tested and pushed, and because **Oracle auto-pulls the private repo at the start of every run, it deploys at the Oct 7 07:00 CT run** (first line expected then; Rus confirms). MLB_GAME is now fully suspended in both directions with every figure labelled unverified (J@rv1s's ruling). The analysis script gained the pre-specified secondary run, a tolerance count and a stale-settlement listing. One public-publishing process failure was found and is being contained (section 4). **Nothing changed in any signal, model or trading rule tonight.** The NFL_GAME model files are unchanged since Sept 15 and I am treating them as frozen.

---

## 1. PRE-REGISTRATION v0.3 (draft, not locked; project doc `claude/preregistration_nfl_game_phase2_DRAFT_v03_20261007.md`)

- **Design (all ruled by J@rv1s):** NFL_GAME is the only confirmatory model. Two arms, each passing only if its 90% event-bootstrap CI lower bound is above zero: **arm I** the paired Brier gain of the model over the market mid; **arm P** net cents per contract per event at the real ask after the round1 fee, one position per event (earliest row). Both pass = demonstrated; one = partial/unresolved; neither = unresolved. Minimum 30 events (below that, counts and points only). Regular-season games only; confirmatory games are those dated after the lock date; one verdict at the end of the season; a "demonstrated" result triggers a FORGE referral, never automatic trading.
- **Stated up front: the likeliest outcome is "unresolved" on both arms.** Design data (42 events, not confirmatory): arm I -0.0086 CI [-0.0422, +0.0272]; arm P +3.64c CI [-8.17, +15.93]; unresolved; w* 0.23 [0.00, 1.00].
- **Concentration is two separate statements.** (a) By outcome: the four best events by P&L contribute +294c against a +153c total; the other 38 average -3.71c. (b) By structure: the model backed both teams at different times in 4 of 42 events (+119c); the other 38 total +34c (+0.89c each). My v0.2 conflated the two sets; corrected. Also corrects an earlier count: "12 of 41 NFL_GAME events have both directions logged" was labels, not opposing bets; the real figure is 4 of 42.
- **Settlement edge cases:** outcomes come from Kalshi settlement; voided or cancelled markets are excluded by status and listed; ties as Kalshi settled; nothing else is excluded. The settlement lookup cannot tell void from "not yet settled", so any event more than 3 days past its game date and still unsettled is listed as STALE and checked by hand.
- **Lock Oct 13** (hard stop Oct 20), and it waits for one full day of manifest lines by Oct 12. Cost of Oct 13 vs Oct 10: 15 flagged events dated Oct 8-12 fall into design data, about 14% of the projected sample.
- **Nov 1 is a checkpoint only** (about 30 events at most): points and counts, no verdict.

## 2. VERSION FREEZE AND THE MANIFEST (commit 7e29a4d, pushed)

- `run_manifest.py` appends one JSON line per run to `data/run_manifest.jsonl` (Oracle only): CT time, pipeline commit, SHA-256 of the NFL model files, `daily_runner.py`, `brier_dashboard.py` and `model_suspension.py`, the NFL_GAME call-site hash and weight line, fee mode, a hash of all package versions (nfl_data_py 0.3.3, pandas 2.2.1, numpy 1.26.4, requests 2.31.0, pytz), and the feed hosts (ESPN site and core APIs). **Fail-open:** it cannot stop a run or change a signal; tested with bad paths. Oracle only because an untracked data file on the laptop would block the laptop's auto-pull.
- **Version history, Oct 7 (git):** `nfl_model.py` last changed Sept 15 (6620452); `nfl_inactives_check.py` Aug 10 (d7383c1). `daily_runner.py` changed since Sept 29 only by three unrelated commits (JOBS text Oct 5, MLB label Oct 6, manifest call Oct 7); **the NFL_GAME call-site hash and weight line (0.55) are identical across all of them and the last commit before Sept 29.** Before the first manifest line, "Oracle ran that code" rests on the auto-pull design and Oracle's own reports, not a per-run record; the manifest closes that from its first line.
- Limit, stated: the manifest records code, packages and feed identity, not what a feed returned.

## 3. MLB_GAME (J@rv1s ruling, four conditions)

- **Fully suspended, both directions** (f3ec5ad; `MLB_GAME_YES_EXPERIMENTAL = False`); NFL_GAME behaviour unchanged.
- **Every MLB_GAME win-rate, P&L and bucket figure is labelled UNVERIFIED** in the daily report and, as an additive top-level key `unverified_models`, in `brier_dashboard.json` (b12e305). Oracle's emission is confirmed only after its Oct 7 07:00 run.
- The backfill game-date fix and the independent MLB Stats API audit of the 230 MLB_GAME rows are deferred to the offseason under the Sept 26 MLB hold (`areas/mlb-game-offseason-backlog`): **week of Nov 2**, with a scheduled reminder set. The Oct 3 outcome-convention repair stands; the game-date defect is still open.

## 4. PUBLIC-PUBLISHING PROCESS FINDING (details in private project docs)

The nightly sync copies ten named data files from the private repo to this public repo every run, overwriting the public copy. A manual hold I placed on some calibration rows on Oct 4 lived only in the public file, so the first nightly copy erased it (Oct 5, about 02:07 CT). That is a process failure of mine: I did not check the sync path when I set the hold. What was done: a private-only calibration file now exists for any row that must not publish (never in the sync list); nothing already public was edited or rewritten, because deleting rows from a public track record is itself a red flag; the audit of all ten synced files found one file with such rows. **Hand-pasted files in this repo are published too** (the dashboard update uses `git add -A`), and no gate covers them. A fail-visible sync gate that holds a file and prints a loud line is designed and goes in as a small separate commit before the matcher (J@rv1s's window). Outside-review exposure checks are not finished; two need Rus.

## 5. SCRIPT AND CALIBRATION

- `tools/phase2_nfl_game_analysis.py` (commits 5204beb, 557e3d5, 9649241): outcomes from settlement with every mismatch listed (0 on the design data), mid-in-[bid, ask] integrity stop with a printed count of rows relying on the 0.5c tolerance (0), minimum-30 rule, `--through`, secondary descriptive `--row-rule last` ("counts for nothing"), STALE listing, both concentration statements.
- Calibration: J@rv1s's Oct 6 predictions logged and one scored (public file: the git-log prediction FALSE on the freeze-set reading, reading flagged for J@rv1s to confirm; matcher-by-Oct-19 PENDING). The earlier fail-closed-audit prediction scored TRUE (Brier 0.4225).

---

## OPEN POSITIONS (`sede_portfolio.json`)

8 open (BTC<$50k NO; seven GDP YES positions on the Oct 30, Jan 28, Apr 29 and Jul 29 prints), bankroll $994.17, no wins or losses recorded, one early exit at -$5.83. No new entries; nothing has cleared the entry criteria. Marks were not re-pulled tonight. GDP stays at reduced weight with n=1 real resolved event.

## GATE 1 / VALIDATION STATUS (carried forward)

- Project-wide Gate 1 remains **provisionally suspended**; checkpoints Oct 30 (GDP) and the Nov 1 Phase 2 checkpoint (report only). Phase 2 threshold is written in the pre-registration and locked about Oct 13, hard stop Oct 20, before Phase 2 data is examined.
- **NFL_GAME: no demonstrated edge, entry weight 0 (trade at market).** Latest (Oct 6, real-ask, 42 events): arm I -0.0086, arm P +3.64c per contract on one row per event, both far from passing; w* 0.23 [0.00, 1.00] (entry weight is the lower bound, zero). One event moved the one-row mean from +2.27c to +3.64c. The project-instructions NFL_GAME paragraph quotes older figures; Rus pastes the replacement.
- NFL_SPREAD, MLS_GAME, SOCCER_GAME: tail finding stands. **MLB_GAME: fully suspended, all figures unverified.** JOBS: n=4 verifiable events, unproven. GDP: n=1. CLAIMS: suspended; fix shipped Oct 1, first real test Oct 7-8 (reminder set for about noon CT Oct 7).
- Win-rate, P&L and bucket figures for NFL_GAME, NFL_SPREAD, MLS_GAME and MLB_GAME computed Sept 24 - Oct 3 remain suspect; decision-impact audit pending.

## WORK AHEAD (dated order)

- **Wed Oct 7:** CLAIMS first-emission check (about noon CT); check Oracle's 07:00 and 11:15 runs: manifest line present, `unverified_models` in the public JSON, no manifest error line (scheduled check about 12:45 CT).
- **Thu Oct 8:** recheck the Oct 2 JOBS ticker gap. **Fri Oct 9:** pre-registration final to J@rv1s after his answers.
- **By Oct 12:** one full day of manifest lines. **Oct 13:** lock target. **Oct 14:** CPI. **Oct 14-17:** tickerless random-sample audit. **Oct 14-19:** matcher (not before Oct 14, ship by Oct 19 or slip) with the sync gate before it. **Oct 19-23:** decision-impact audit. **Oct 20:** hard stop for the lock; NBA opener. **Oct 30:** GDP print. **Nov 1:** Phase 2 checkpoint. **Week of Nov 2:** MLB backfill game-date fix and 230-row audit.
- **Rus:** (1) after 07:00 CT, confirm the manifest line and the MLB label on Oracle's output; (2) answer the two open exposure checks (reviewer reply times against Oct 5 02:07 CT; whether any reviewer chat had browsing or search on); (3) paste the Phase 2 bullet and NFL_GAME line replacements into the project instructions, and the MLB line after the four MLB conditions are seen done; (4) one-minute public lock-timestamp line at the lock; (5) note J@rv1s's Oct 5 briefing in this repo is public.
- Standing: four laptop stashes exist (data churn), do not drop them; Auto Monitor stop-loss alerts still laptop-only; wire the outcome invariant into publication.

## ORACLE / SPORTS MONITORING

Oracle: sole pipeline runner; auto-pulls the private repo (fast-forward only) at the start of every run and re-executes on the new code, so tonight's pushes (head 9649241 plus the calibration commit) are picked up at the Oct 7 07:00 CT run. Last verified Oracle auto-update before tonight: Oct 5 21:00 CT. Sports monitoring: no open sports positions.

---

Archie | Papa Ralph standard.
