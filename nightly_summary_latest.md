# SEDE Nightly Summary
**Date:** 2026-06-24 | **Generated:** ~22:30 CT | **Session:** Archie (Evening Build)

---

## System Status

| Component | Status |
|---|---|
| False auto-close | ✅ Holding stable — status-only close confirmed |
| BTC/GDP price read | ✅ FIXED TONIGHT — _dollars field rename |
| MLB_GAME auto-trading | ✅ Ticker fixed (Jun 23) — awaiting right signal |
| Subscriber alerts | ✅ Real numbers on next resolution |
| Duplicate ticker gate | ✅ Tightened (Jun 23) |
| WC diagnostic script | ✅ Written, run, output in hand for Jun 26 review |
| WC_GAME | CALIBRATION — Jun 26 review tomorrow |
| SEDE Portfolio | ✅ $1,000, 2 open, prices now reading live |
| JOBS | ✅ Validated, go-live eligible |
| GDPNow | ✅ 3.04% — next update TOMORROW Jun 25 |
| Oracle cron | ✅ 07:00 UTC (02:00 CT) — pulling tonight's commits |

---

## SEDE Portfolio

| Field | Value |
|---|---|
| Bankroll | $1,000.00 |
| P&L | $0.00 |
| Record | 0W-0L |
| Open | 2 of 8 slots |

### Open Positions (live prices now working)

| ID | Market | Dir | Entry | Live Mid | Drift | Thesis |
|----|--------|-----|-------|----------|-------|--------|
| 1 | BTC < $50k Dec 31 | NO | 43.5c | **36.5c** | -7c | BTC ~$63k, well above $50k. Thesis intact, market drifting against us short term. |
| 3 | GDP > 2.5% Q2 | YES | 47c | **44.5c** | -2.5c | GDPNow 3.04%. Next update TOMORROW. Watch closely. |

**paper_trades open:**
- #8 Fed 1x cut YES @ 21c — documented loss, hold to resolution
- #12 GDP >2.5% YES @ 40c — watch GDPNow tomorrow
- #13 GDP >2.0% YES @ 60c — healthy at 65c

---

## Tonight's Builds

### Fix — current_price_cents: 0.0 ✅ (Priority 1)
**File:** `portfolio_manager_sede.py` — `_fetch_kalshi_price()`
**Root cause:** Private fetch function used bare field names (`yes_bid`, `yes_ask`) but Kalshi API returns `yes_bid_dollars` / `yes_ask_dollars`. `price_fetcher.py` was already correct — only this internal function was stale.
**Fix 1:** Renamed all four fields to `*_dollars` variants, multiply by 100 for cents.
**Fix 2:** Added 0c safety guard — if both sides return 0c on active market, treat as fetch failure (return None, hold position). Belt-and-suspenders on top of status-only close.
**Verified live:** BTC NO mid=36.5c, GDP YES mid=44.5c. Both reading correctly.
**Commit:** f78c7a5

### WC Diagnostic Script ✅ (Priority 2)
**File:** `wc_game_diagnostic.py` (new)
J@rv1s's script adapted with correct `signals_log.csv` column names (`model`, `market`, `kalshi_cents`, `model_pct` — not the names J@rv1s assumed). Ran locally against 1,091 resolved WC/SOCCER signals.
**Commit:** f2d5703

### WC Backfill — June 23-24 ✅ (Priority 3)
**File:** `signal_scorer.py` — 21 new RESOLVED_MARKETS entries
7 games scored: Colombia/DR Congo, Switzerland/Canada, Bosnia/Qatar, Brazil/Scotland, Morocco/Haiti, Mexico/Czechia, South Africa/Korea.
Scorer: **1,378 signals | Brier 0.1734 | Accuracy 35.8%**
**Commit:** 4527d2e

---

## WC Diagnostic Results — June 26 Review Brief

**1,091 resolved WC_GAME + SOCCER_GAME signals analyzed.**

### Theory 1 — Draw Underweighting
NOT CONFIRMED via data (notes column doesn't tag draws). Needs manual verification. Do not rule out — draw losses are real, just not measurable from current logging.

### Theory 2 — Lambda Compression ⚠️ KEY FINDING
- YES signals where model_prob 0-35%: **7.9% WR on 354 signals**
- YES signals where model_prob 36-45%: **0.0% WR on 26 signals**
- YES signals where model_prob 66-80%: **100% WR on 73 signals**

The model is generating YES signals on teams it gives 7-35% win probability. That is the dominant source of losses. This is not lambda compression — it's a signal generation bug. YES signals are being fired when edge is positive even though model_prob is very low, meaning Kalshi price is even lower (buying 10-20c YES on 92% underdogs).

**Fix for Thursday:** Add minimum model probability gate for YES signals. Don't fire YES unless model_prob ≥ 45%. This alone eliminates 380 losing signals.

### Theory 3 — Edge Threshold
15-19c edge signals: 88% WR (100 signals) — inverted from expectation. Likely data artifact or scoring alignment issue. Flag for Thursday review.

### Theory 4 — Retail Bias ✅ CONFIRMED
- Fading underpriced teams (Kalshi <20c): **18.7% WR** — terrible
- Fading fairly-priced teams (Kalshi 20-35c): **63.9% WR** — excellent

Don't generate NO signals where faded team is already priced below 20c on Kalshi. Market has already done the work. One-line filter.

### Bonus — Fading Strong Favorites
13 signals at 60c+ Kalshi: 100% WR — but tiny sample (Panama/Croatia duplicates). Not meaningful yet.

### Thursday Review Agenda (data-driven)
1. Implement YES signal min probability gate (model_prob ≥ 45%)
2. Implement NO signal Kalshi floor (don't fade teams already <20c)
3. Investigate Theory 3 edge inversion — possible scoring artifact
4. Verdict: continue calibration through knockouts, NO auto-trading approval
5. Champions League September as primary deployment target

---

## Calibration Report

| Model | N | Accuracy | Brier | Status |
|-------|---|----------|-------|--------|
| GDP | 4 | 50.0% | 0.4566 | ⚠️ tiny sample |
| JOBS | 81 | 58.0% | 0.1351 | ✅ |
| SOCCER_GAME | 219 | 32.9% | 0.1415 | ✅ |
| WC_GAME | 1074 | 34.6% | 0.1817 | ✅ (calibrated despite low WR) |
| **OVERALL** | **1378** | **35.8%** | **0.1734** | ✅ |

---

## Tomorrow — June 25

| Time CT | Event | Priority |
|---------|-------|----------|
| Morning | GDPNow update | 🔴 Watch Trade #12 — if drops below 3.0% thesis review |
| Morning | Kalshi opens June unemployment markets | 🟡 JOBS signals resume |
| 3 PM | ECU vs GER, CUW vs CIV | Score next session |
| 6 PM | TUN vs NED, JPN vs SWE | Score next session |
| 9 PM | TUR vs USA, PAR vs AUS | Score next session |
| Evening | Jun 25 WC backfill | Standard |

**Note on USA tonight:** TUR vs USA kicks off 9 PM CT June 25. Win probability: USA 50.6%, Turkey 25.5%, Draw 23.9%. Model will likely flag this. Don't force an entry — let gates decide.

---

## Key Dates

| Date | Event |
|------|-------|
| Jun 25 | GDPNow update — critical for Trade #12 |
| Jun 25 | June unemployment markets open on Kalshi |
| **Jun 26** | **WC group stage ends — model review session** |
| Jul 1 | NFL build window opens |
| Jul 2 | Jobs report 8:30 AM CT |
| Jul 14 | CPI release |
| Jul 30 | Q2 2026 GDP advance estimate |
| Aug 2 | Vegas — Anthony (8rain) demo |
| Sep 3 | NFL season opens |
| Sep | Champions League — soccer primary target |

---

*Generated by Archie (Claude Desktop) | Push from laptop → Oracle pulls at 02:00 CT*
*Papa Ralph standard. If it's worth doing, it's worth doing right.*
