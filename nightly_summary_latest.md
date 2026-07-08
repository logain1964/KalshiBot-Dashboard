# KalshiBot Nightly Summary
## For J@rv1s -- Tuesday July 7, 2026 (Evening Session)
Prepared by Archie | Session ~7:00 PM - 10:00 PM CT

---

## TL;DR FOR TOMORROW MORNING

Four real bugs found and fixed tonight, all committed and pushed to
Oracle. One wrong conclusion of my own caught before it reached Rus as
fact -- worth flagging so you don't repeat the same false step if this
thread comes up. Tomorrow's 7AM CT run is the key verification moment:
first real test of the cron timezone fix.

---

## BUGS FIXED TONIGHT (4)

1. **Cron timezone mislabeling.** Oracle runs `Etc/UTC`. The crontab line
   meant to fire at 7AM CT (`0 7 * * *`) was actually landing at 2AM CT.
   Confirmed via `timedatectl` + log timestamps. Email/Telegram were
   both sending correctly at 2AM the whole time (`[OK] Sent` in logs) --
   this was never a delivery failure, just a silent scheduling error.
   Fixed: crontab now reads `0 12 * * *`. **Needs tomorrow's 7AM run to
   fully verify end-to-end.**

2. **Hardcoded Windows path in trade_monitor.py** (`TRADES_FILE`).
   Silently broke `check_all_trades()` on Oracle -- `os.path.exists()`
   returned False every run, so `load_open_trades()` returned `[]`.
   This is why tonight's 2AM report showed "0/5 open, $0.00 at risk"
   despite paper_trades.json actually having 2 real open trades (#8, #13).
   Fixed to branch on `SEDE_MACHINE=="oracle"`, matching the pattern
   already used elsewhere in the codebase. Commit 148f5aa.

3. **Same hardcoded-path bug in mlb_outcome_backfill.py** (`SIGNALS_LOG`).
   Found via a full-codebase audit after bug #2. Was silently no-op'ing
   `run_mlb_backfill()` on every single Oracle cron run. Fixed. Commit
   a945a1b.

4. **Inverted sign in auto_monitor.py's `calculate_current_loss()` NO
   branch.** Computed `current_price - entry` for NO trades; should be
   `entry - current_price`. This meant a NO price RISING (favorable --
   market agreeing with the NO thesis) was misread as a LOSS. Root
   cause of trades #23 and #24 both getting auto-stop-lossed on June 9
   -- both were actually winning trades (CPI stayed under threshold,
   NO price rose 50c->92c and 38c->92c) that got closed early because
   the bug flagged the good move as bad. `pnl_dollars` for both was
   always correct (+21.0, +35.1); only the close_note mischaracterized
   what happened. Fixed formula, corrected both close_notes to explain
   the real story. Commit 05c24ee.

Also fixed (not a functional bug, but wrong output): stale "Next
release: June 5, 2026" date references in jobs_model.py -- corrected to
Aug 7, 2026 (July jobs data). UNEMP_CONSENSUS (4.2%) confirmed correct
and untouched -- it was Capital Economics' forecast, which beat Dow
Jones' stale 4.3% when the actual June print landed at 4.2%.

---

## ONE MISTAKE OF MINE, CAUGHT BEFORE IT WENT ANYWHERE

While investigating #23/#24, I initially concluded the ledger's P&L
signs were systematically wrong across all 13 closed NO trades, and
"corrected" the total to -$151.62 (from the real +$139.14) using a
branching, direction-aware P&L formula. This was backwards -- checked
it against Trade #1's known-good WON outcome before reporting it as
fact, found the SIMPLE non-branching formula (`(exit-entry)/100*contracts`,
no direction check needed because entry/exit are always quoted in terms
of the held side) was actually correct. Walked it back before Rus acted
on it. Real total P&L is still +$139.14, confirmed correct.

If this comes up again: the branching formula LOOKS more careful but is
wrong for this codebase's price convention. Don't reintroduce it.

---

## STILL OPEN (unchanged from your briefing, not touched tonight)

- June 30 session archive still missing
- Oracle cron code-staleness decision (git pull before each cron run?)
- NFL Week 1 Signal Contract spec -- not started
- nba_game_model.py / nhl_game_model.py dict-vs-tuple mismatch (dormant)

## NEWLY OPEN (from tonight)

- 9 files with hardcoded C:\KalshiBot paths, unverified whether they run
  on Oracle: auto_monitor.py, mlb_autoclose.py, mlb_gametime_fill.py,
  mlb_refresh.py, oci_retry.py, paper_trades.py, polymarket_monitor.py,
  post_seeds_bridge.py, claims_model_review.py. Note auto_monitor.py
  turned out to be live (touched tonight for bug #4) -- bump priority
  on auditing the rest.
- jobs_model.py UNEMP_CONSENSUS/NFP_CONSENSUS need real forecaster
  numbers ~Aug 5, ahead of the Aug 7 release.

---

## GDPNOW

Updated today (July 7) to **1.4%**, up slightly from 1.1889%/1.2% on
July 1. Not below the 1.0% flag threshold. Trade #13 (GDP >2.0% YES)
still well under thesis threshold -- small bounce doesn't change the
calibration-hold reasoning.

---

## COMMITS PUSHED TO ORACLE TONIGHT

```
148f5aa  Fix hardcoded Windows path in TRADES_FILE
a945a1b  Fix hardcoded Windows path in SIGNALS_LOG
fd7dae8  Log hardcoded-path audit findings to backlog
6e9a648  Fix stale Next release date in jobs_model.py
09a9325  Log Aug 5 jobs_model.py consensus reminder
05c24ee  Fix inverted sign in calculate_current_loss()
[+1]     Correct close_note text for trades #23/#24
```

Plus crontab fix on Oracle (not a git commit): `0 7 * * *` -> `0 12 * * *`.

---

*Archie | Session end ~10:00 PM CT | July 7, 2026*
*Papa Ralph standard. If it's worth doing it's worth doing right.*
