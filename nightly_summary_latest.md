# SEDE Nightly Session Summary
## For J@rv1s Morning Intelligence Pull

**Last updated:** 2026-06-18 | **Session end:** ~11:30 PM CT
**Prepared by:** Archie (Claude Desktop)

---

## TONIGHT IN ONE SENTENCE

Six deliverables: MLB post-fix WR confirmed healthy (65.4%), sede_portfolio.json
built and wired, signal_scorer.py Unicode crash fixed, Fed cuts 0x NO signal
suppressed in hike-bias environment, WC Elo updated post Round 1, England/France
results scored -- WC_GAME WR now 44% on 18 signals, genuine model flag.

---

## 1. MLB POST-FIX ACCURACY CHECK

Post-fix (June 14+): 7 resolved signals, 7/7 wins (100% WR), Brier 0.172.
Sample too small to be conclusive but directionally excellent.

All-time deduped: 81 resolved signals, **65.4% WR**, Brier 0.254.
Well above Gate 1 threshold (55%). Green light trajectory.

Only YES direction in post-fix window -- NO direction hasn't had resolved
signals since the fix, so the direction bug impact still can't be isolated.

---

## 2. SEDE_PORTFOLIO.JSON -- BUILT (commit c01c465)

Primary deliverable. Subscriber-facing autonomous portfolio operational.

**Files created:**
- `data/sede_portfolio.json` -- initialized at $1,000 bankroll, paper status,
  zero trades. On both GitHub repos including public dashboard.
- `portfolio_manager_sede.py` -- autonomous manager with full feature set:
  - 6-gate auto-entry: edge>=20c, confidence>=60%, rating=HIGH,
    max 8 concurrent, hard drawdown stop at -$200, no duplicate tickers
  - Auto-close: polls live Kalshi prices, closes on resolution/97c/3c
  - Daily snapshot for subscriber charting
  - `--status` CLI for quick checks
- Wired into `daily_runner.py` as Step 10b -- fires after signal generation,
  before Telegram/email delivery
- `sede_portfolio.json` added to dashboard .gitignore allowlist

**Known gap:** ticker lookup in daily_runner wire is fuzzy match. Will miss
some signals and log "no market ticker" skip. Fix: pass tickers explicitly
from each model at signal generation time. Low priority follow-up.

**This is what Anthony sees in 45 days.**

---

## 3. SIGNAL_SCORER.PY UNICODE CRASH -- FIXED (commit 88f6bb4)

Unicode box-drawing characters (U+2500) and emoji (U+2705, U+1F6AB)
caused CP1252 encode errors on Windows console, silently crashing
score_full_log() partway through the table print. Fixed by replacing
with ASCII equivalents ([OK], [X], -).

**Key finding from running clean scorer:**
WC_GAME was already at **58% WR** in full log -- the 34.9% figure
J@rv!s had was stale, predating the Draw entries in RESOLVED_MARKETS.
The scoring logic was correct all along.

---

## 4. FED CUTS 0X NO SIGNAL SUPPRESSED (commit ef3ef3e)

J@rv!s flagged: "Fed cuts 0x 2026 -- BUY NO +19.9c, model 39.3%" as
confusing post-FOMC.

**Root cause:** BUY NO on "cuts 0x" = betting the Fed cuts at least
once. Post-FOMC with dot plot projecting a hike, Kalshi pricing 81c on
zero cuts is rational -- it agrees with the hike bias. The apparent 20c
edge was a FedWatch/Kalshi alignment artifact, not a real trade.

**Fix:** Suppression gate added to `run_fed_model()`. When cuts_0 > 55%
(hike-bias environment), "Fed cuts 0x BUY NO" is suppressed with a
log line explaining why. Auto-reinstates when cuts_0 drops below 55%.

---

## 5. WC ELO STRENGTH UPDATE -- POST ROUND 1 (commit 2e5f18d)

`models/world_cup_model.py` WC_TEAMS strength values updated.

**Source:** eloratings.net January 2026 Elo points, normalized to 0-1
scale anchored on Spain=2171=0.94.

**Key corrections from pre-tournament values:**
- Spain correctly moved to #1 (was incorrectly behind France)
- Colombia and Ecuador elevated per actual Elo standing
- Germany up after 7-1 Curacao
- Australia up after 2-0 Turkey
- Japan held -- drew Netherlands, confirmed mid-pack quality
- Turkey, Curacao, Haiti, South Africa all down post R1 losses
- Norway elevated -- Elo 1922, higher than originally assigned

---

## 6. WC RESULTS SCORED -- ENGLAND/FRANCE -- MODEL FLAG

**Added to RESOLVED_MARKETS:**
- England 4-2 Croatia (June 15) -- England wins YES
- France 3-1 Senegal (June 16) -- France wins YES

**Result: WC_GAME WR dropped from 57% (14 signals) to 44% (18 signals)**

**This is a genuine model flag, not a scoring artifact.**

The model had flagged:
- England wins NO (model 55.7%, betting against England) -- LOST
- Croatia wins YES (model 26.9%, backing Croatia) -- LOST
- France wins NO (model 42%, betting against France) -- LOST
- Senegal wins YES (model 19.8%, backing Senegal) -- LOST

Pattern: model is systematically underestimating tournament favorite
win probability and generating NO signals on favorites that then win
comfortably. This is a calibration problem, not a scoring bug.

**Action for J@rv!s:**
- WC_GAME remains CALIBRATION ONLY
- DO NOT allow portfolio_manager_sede.py to trade WC_GAME signals
  until WR recovers above 55% with 30+ signals
- Investigate: are lambda values underweighting strong favorites?
  The Spain/Cape Verde problem pattern may be recurring
- June 26 group stage checkpoint is now a model review, not just
  a sample size check

---

## OPEN POSITIONS

| # | Description | Entry | Status |
|---|-------------|-------|--------|
| 8 | Fed 1x cut YES | 21c | HOLD -- thesis dead, documented loss |
| 12 | GDP >2.5% YES | 40c | GDPNow 3.04%, buffer +54bps, HOLD |
| 13 | GDP >2.0% YES | 60c | GDPNow 3.04%, comfortable, HOLD |

---

## SEDE PORTFOLIO STATUS (new tonight)

Bankroll: $1,000.00 | Trades: 0 | Status: paper
No auto-entries yet -- portfolio manager needs tickers from models
to fire. First real entries expected on tomorrow's pipeline run.

---

## COMMIT LOG (tonight)

| Commit | Description |
|--------|-------------|
| c01c465 | Build sede_portfolio.json + portfolio_manager_sede.py |
| 88f6bb4 | Fix signal_scorer.py Unicode/emoji crash (CP1252) |
| ef3ef3e | Suppress Fed cuts 0x NO in hike-bias environment |
| 2e5f18d | WC Elo post Round 1 + England/France scored |
| 762744d | Merge: Oracle pipeline data |

Oracle: run `sede-pull` to sync all tonight's commits.

---

## SYSTEM STATUS

| Component | Status |
|-----------|--------|
| Oracle SSH | ✅ Fixed (key permissions) |
| Oracle sync | ✅ sede-pull confirmed clean before dinner |
| sede_portfolio.json | ✅ BUILT -- $1,000 bankroll, 0 trades |
| portfolio_manager_sede.py | ✅ Built and wired into daily_runner |
| FedWatch data | ✅ Post-FOMC, cuts_0=60.7% |
| Fed cuts 0x NO | ✅ Suppressed in hike-bias environment |
| GDPNow | ✅ 3.04%, next update Jun 25 |
| GDP trades | ✅ Both healthy |
| Trade #8 Fed | ⚠️ Documented loss, HOLD |
| MLB_GAME | ✅ 65.4% WR all-time, 100% post-fix (n=7) |
| WC_GAME | ⚠️ 44% WR (18 signals) -- MODEL FLAG |
| WC Elo | ✅ Updated post Round 1 |
| signal_scorer.py | ✅ Unicode crash fixed, runs clean |
| JOBS | ✅ Go-live eligible |
| Claims | SUSPENDED -- holiday test passed |
| sede-pull alias | ✅ Live on Oracle |

---

## J@rv1s MORNING ACTIONS (ordered)

1. **Oracle sede-pull** -- sync tonight's commits (c01c465 through 762744d)

2. **WC_GAME model flag** -- 44% WR on 18 signals. Pattern is systematic:
   model underestimates strong favorites, generates losing NO signals.
   Flag for dedicated investigation before June 26 checkpoint.
   Do NOT allow portfolio_manager_sede.py to trade WC_GAME signals.

3. **Portfolio manager first run** -- check daily_report for "SEDE PORTFOLIO
   MANAGER" section. Were any signals passed with valid tickers? If zero
   entries, the ticker lookup is the gap -- flag for Archie.

4. **sede_portfolio.json** -- now public on dashboard. Confirm file is
   visible at github.com/logain1964/KalshiBot-Dashboard

5. **WC results backfill** -- Several Round 1 results not yet in
   RESOLVED_MARKETS (NED 2-2 JPN, GER 7-1 CUW, IVC 1-0 ECU, USA 4-1 PAR,
   BEL vs IRN, AUS 2-0 TUR). Verify Kalshi market name strings in
   signals_log.csv before adding -- wrong name = wrong score.

6. **Today's June 18 results** -- Switzerland vs Bosnia, Canada vs Qatar,
   Mexico vs South Korea, Czechia vs South Africa all played today.
   Add to RESOLVED_MARKETS once Kalshi market strings confirmed.

---

## VALIDATION TRACKER

| Model | Status | Gate 1 Progress |
|-------|--------|-----------------|
| JOBS | ✅ GO-LIVE ELIGIBLE | n=73, 58.9%, Brier 0.134 |
| GDP | Active | 3 open positions, Jul 30 |
| CPI | Active | Accumulating monthly |
| Claims | SUSPENDED | Holiday test passed |
| MLB_GAME YES | Active | 65.4% WR, 7/7 post-fix |
| WC_GAME | ⚠️ CALIBRATION -- MODEL FLAG | 44% WR (18), below random |
| Fed | Active | Trade #8 = documented loss |
| SEDE Portfolio | ✅ BUILT | $1,000 starting bankroll |

---

## KEY DATES

| Date | Event |
|------|-------|
| Jun 19 | Juneteenth -- federal holiday |
| Jun 25 | GDPNow next update |
| Jun 25-27 | Kalshi opens June unemployment markets |
| Jun 26 | WC group stage ends -- WC_GAME model review |
| Jul 1 | NFL build window opens |
| Jul 2 | Jobs report 8:30 AM CT |
| Jul 30 | Q2 2026 GDP advance estimate |
| Aug 2 | Vegas -- 8rain demo (Anthony) |
| Sep 3 | NFL season opens |

---

*Session | Model: Sonnet 4.6 | Identity: Archie*
*sede_portfolio.json exists. Anthony has something to see in 45 days.*
*WC_GAME at 44% WR is a real flag -- don't paper over it.*
*sede-pull alias live on Oracle. No more manual conflict resolution.*
*Papa Ralph standard. If it's worth doing it's worth doing right.*
