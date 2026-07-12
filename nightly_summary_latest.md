# KalshiBot Nightly Summary — COMBINED (July 11 + July 12 sessions)
## For J@rv1s | Prepared by Archie (Opus 4.8)
## Covers two dense sessions deliberately deferred into one combined write.

---

## TL;DR FOR J@RV1S

Two big sessions. Session 1 (Jul 11): ratified + deployed the win-rate
definition (your hybrid), discovered MLB Track A/B was never actually
built, and built the frozen Track B model core. Session 2 (Jul 12): built
the full prospective Track B pipeline, then Rus's instinct about weekend
cron windows uncovered THREE real pipeline bugs (all found, all fixed,
none flipped any conclusion), fixed the cron, and repaired a silently-broken
Dashboard sync (60 unpushed commits).

**Your three independent passes this weekend all landed** — win-rate
dissent, date-bug Claim-2 pressure-test, dedup n-reconciliation. Thank you;
they materially changed outcomes. Nothing below needs a pass unless flagged.

**Standing rules ratified (apply from now, cross-instance):**
1. "DEPLOYED" = verified producing output, not merely committed.
2. Every nightly summary carries a DEADLINE WATCH block (below).

---

## DEADLINE WATCH (standing block — honest status each item)

| Item | Hard date | Status |
|------|-----------|--------|
| MLB verdict | ~~Jul 25~~ SLIPPED | Deliberate: prospective Track B now accumulating. Re-eval when n sufficient. Documented. |
| NFL Signal Contract spec | Sep 3 | NOT STARTED. Highest forward priority. Bake in day-one signal capture (your lesson). |
| Track B first real capture | next game day | Code deployed + verified on Oracle; first WRITE happens when scheduled games hit the 7am/11:15am cron. Not yet demonstrated. |

---

## SESSION 1 — Friday... er, Saturday July 11

### 1. Win-rate definition — RATIFIED + DEPLOYED (your hybrid won)
Archie proposed resolution-only; you dissented and broke the redundancy
argument (Brier already excludes EARLY_EXIT — brier_score:null). Rus
ratified YOUR hybrid:
- GATE = pnl>0 across ALL closed trades (45.5%, 10/22)
- COMPANION = resolution-only WON/(WON+LOST) (47.1%), never gates
- EARLY_EXIT split -> DISCRETIONARY_EXIT / BUG_INDUCED_EXIT
Foundation Doc bumped to v1.1.0 (Part 11), amendment logged, code
(validation_dashboard.py) + data (paper_trades.json subtypes) updated.
DEPLOYED to Oracle and verified. Also corrected: per-model Gate 1 = 30
resolved; portfolio go-live = 75 closed (Archie had wrongly said Gate 1=75).

### 2. Win-rate schema root-cause fix
mlb_autoclose.py wrote outcome into `status` instead of status=CLOSED +
result. Fixed at root; #17-20 normalized. Would have silently excluded
future live MLB WINS too, not just the 4 losses.

### 3. MLB Track A/B readiness — THE DISCOVERY
Found the Foundation Doc Part 6 two-track test CANNOT run as written:
- Track A (edge decay): ZERO data. kalshi_price_at_gametime column never
  existed; all 188 MLB signals empty. Unrecoverable historically.
- Track B (independent model): NEVER BUILT. No Elo/FIP/park anywhere.
Your reframe (correct): this is a PROCESS failure — measurement infra was
designed AFTER MLB went live. The real lesson is for NFL: day-one capture.

### 4. Track B method decision
Rus's gut: only a clean PROSPECTIVE test will satisfy him — no retro
reconstruction. So: build prospective Track B, July 25 verdict SLIPS
(documented). Model core built + FROZEN (pre-registered spec, off-the-shelf
constants, git-timestamp freeze). Committed 11ad04f.

---

## SESSION 2 — Sunday July 12

### 5. Track B full pipeline — BUILT + DEPLOYED + VERIFIED
- track_b_logger.py: logs P(model) vs P(sharpapi) for EVERY scheduled game
  (Option A, unbiased sample), blind to outcome, own CSV (data/track_b_log.csv).
- Wired into models/mlb_model.py run_game_model (used by BOTH daily_runner
  and mlb_refresh, so all runs log). Isolated in try/except — BR outage
  can't touch signals.
- track_b_backfill.py: scores logged games after they finish (past-only,
  writes only actual_outcome, never predictions).
- Verified on Oracle: real run showed "[Track B] tables ready (1429 games,
  744 pitchers) -- logging ON", logged 0 (all games final -- correct,
  blind-to-outcome preserved).
- First live divergences seen in testing: ARI@LAD model 0.556 vs sharp
  0.794 (-0.238), TOR@SDP 0.521 vs 0.707 (-0.186). Machinery produces
  real divergence, not market-parroting.

### 6. THREE pipeline bugs found (Rus's weekend-cron instinct uncovered them)
**Bug A — UTC/CST date-boundary.** mlb_model.py used date.today() (UTC on
Oracle), so the evening run fetched TOMORROW's games while stamping CST
run_date. Claim 2 (does this corrupt the 52.9%/0.2563 Brier?) — YOU pushed
the confidence-distribution check: exposed signals DO skew 2x more confident
(0.366 vs 0.166), so a mis-score would carry outsized weight. But
game-by-game check of all 9 exposed / 6 splits: **0 actually mis-scored**
(all scored against correct run_date). Claim 2 CLOSED. Numbers clean, model
genuinely mediocre. Fixed (CST-aware helpers) — forward-looking.

**Bug B — backfill cutoff.** Same date.today() in the "past games only"
cutoff would let the evening run try to score TODAY's in-progress games.
Fixed CST + boundary-tested.

**Bug C — dedup undercount.** brier_dashboard.py deduped by market NAME
alone, collapsing the same matchup on different dates (LAD@ARI Jun 1/3/4 =
1 game). Undercounted. Fixed to (run_date, market): **n 83 -> 103**.
Accuracy/Brier essentially unchanged -> still BELOW RANDOM, still fails
Gate 1 on CALIBRATION not sample size. n=103 is now AUTHORITATIVE; retire
n=85. Both corrections disclosed in track_b_prereg_spec_v1.md.

Three bugs, none flipped the "MLB genuinely mediocre" read — itself evidence
the verdict conclusion is robust to noise, not an artifact of it.

### 7. Cron fixed — weekend coverage gap CLOSED
Oracle cron was 7am/7:45am/9pm CST (memory note "7am confirmed" was right
for summer CDT; the 7:45 was cruft). Only the 9pm run hit the MLB window,
too late for weekend day games. Added 11:15am CDT run (15 16 UTC) inside
the window; removed the 7:45 stray. Backup saved on Oracle.

### 8. Dashboard sync — SILENTLY BROKEN, repaired
The daily_runner's push to KalshiBot-Dashboard had been failing for **60
commits** — J@rv1s's morning data was stale and nobody knew. Surfaced by
ABV on a push-fail message. Resolved (Oracle owns data), pushed, verified
paper_trades.json intact (24 trades, #8 + #13 open). Sync now works going
forward.

### 9. Track B idempotency-with-upgrade
Multi-run schedule meant a 7am fallback-pitcher log would block the 11:15am
confirmed-pitcher upgrade. Fixed: re-log now UPGRADES in place when more SPs
confirmed; scored rows FROZEN; confirmed rows LOCKED. All 4 cases tested.
Deployed + verified on Oracle. (First demonstrated upgrade = tomorrow's
cron cycle.)

---

## OPEN POSITIONS (verified vs ledger)
- #8 Fed rate-cut count >=1 (KXRATECUTCOUNT) YES @21c — long-dated (Dec 31)
- #13 GDP >2.0% (KXGDP-26JUL30) YES @60c — HOLDING to resolution. GDPNow
  1.3% as of Jul 8 (next update Jul 16). Thesis decayed, likely loss.
  "Data is data." Only 2 open; 22 closed.

## COMMITS (both sessions)
```
Code repo (C:\KalshiBot):
3f96d0c  win-rate schema root fix
ea95ddd  Foundation Doc v1.1.0 win-rate definition
11ad04f  Track B pre-registered model core (frozen)
9fb9eec  Track B logger wired into pipeline
255cc04  Track B outcome backfill
3025f6b  UTC/CST date-boundary fix
39796e7  dedup fix (n 83->103) + disclosures
3e81dce  Track B idempotency-with-upgrade
Dashboard: 60-commit backlog repaired + pushed (3f4474b)
Oracle: all commits pulled + verified live.
```

## STILL OPEN / NEXT SESSION
- NFL Signal Contract spec — NOT STARTED, Sep 3, day-one capture requirement
- on_signal_resolved() stub — prereq before subscriber onboarding
- 9-file hardcoded-path audit
- June 30 session archive still missing
- Track B: watch tomorrow's cron for first real WRITE + first upgrade demo
- Telegram Markdown HTTP 400 (benign, self-recovers to plain text) — low pri

---

*Archie | Opus 4.8 | Combined summary, sessions Jul 11 + Jul 12*
*Three bugs found and fixed, none flipped a conclusion. Dashboard sync*
*repaired. Track B live. Your three passes shaped every major call.*
*Papa Ralph standard. If it's worth doing, it's worth doing right.*
