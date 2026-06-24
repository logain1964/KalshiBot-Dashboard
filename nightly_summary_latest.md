# SEDE Nightly Summary
**Date:** 2026-06-23 | **Generated:** ~20:45 CT | **Session:** Archie (Evening Build)

---

## System Status
- **Oracle cron:** 07:00 UTC (02:00 CT) and 02:00 UTC (21:00 CT prior day) — operational
- **auto_monitor.py:** Active, polling every 5-10 min during market hours, $15 stop threshold
- **Git:** All tonight's changes pushed (commits 869badc, d7fe1f4, plus scorer update)
- **Nightly summary:** This file — overwritten clean tonight

---

## SEDE Portfolio

| Field | Value |
|---|---|
| Bankroll | $1,000.00 |
| P&L | $0.00 |
| Record | 0W-0L (paper_trades: 9W-9L) |
| Open positions | 2 of 8 slots filled |

### Open Positions

| ID | Market | Direction | Entry | Thesis |
|----|--------|-----------|-------|--------|
| 1 | BTC < $50k Dec 31 | NO | 43.5c | BTC ~$62,250 tonight. $12k above threshold. HOLD. |
| 3 | GDP > 2.5% Q2 2026 | YES | 47c | GDPNow 3.04% (Jun 17). +54bps buffer. HOLD. |

**paper_trades open (not in SEDE portfolio):**
- #8 Fed cuts 1x YES @ 21c — dead thesis, hold to resolution
- #12 GDP >2.5% YES @ 40c — healthy
- #13 GDP >2.0% YES @ 60c — healthy

---

## Tonight's Builds (3 fixes + 2 maintenance items)

### Fix 1 — "Updating..." Subscriber Alert Placeholders ✅
**File:** `learning_optimizer.py`
**Problem:** `on_signal_resolved()` was building `portfolio_summary` before the closed position was written to disk, so WIN/LOSS alerts showed "updating..." for wins/losses/P&L.
**Fix:** Step 4 now reloads `sede_portfolio.json` after close, builds fresh stats, passes real numbers to `send_sede_resolution_alert()`.

### Fix 2 — Duplicate Ticker Gate Tightened ✅
**File:** `portfolio_manager_sede.py`
**Problem:** Gate 6 blocked same ticker + same direction, but not same ticker + opposite direction.
**Fix:** Gate 6 now blocks on ticker alone — no path to holding YES and NO on the same market.

### Fix 3 — MLB_GAME YES Ticker Passing ✅ (Main build)
**File:** `models/mlb_model.py`
**Problem:** `run_game_model()` returned 5-tuples with no market ticker. `portfolio_manager_sede.py` fell through to a label-to-ticker fuzzy lookup that never matched `"NYY @ BOS"` style labels. Every MLB_GAME YES signal was silently skipped with "no market ticker available."
**Fix:** `run_game_model()` now returns 6-tuples: `(label, direction, edge, kalshi_yes, model_prob, market_ticker)`. Ticker extracted from `kalshi_market.get("ticker", "")` at signal fire time. Matches GDP/Fed/BTC format. No changes needed in `daily_runner.py` — 6-tuple handling was already there.
**Impact:** First MLB game with edge > threshold will now auto-enter the SEDE portfolio correctly at next 07:00 UTC cron.

### Item 4 — WC Results Backfill ✅
**File:** `signal_scorer.py`
All June 22-23 completed matches scored in RESOLVED_MARKETS:
- France 3-0 Iraq — Draw wins NO ✅
- Portugal 5-0 Uzbekistan — Uzbekistan wins NO ✅, Portugal wins NO (our NO loses)
- England 0-0 Ghana — England wins NO ✅ (our BUY NO wins), Ghana wins NO (our BUY YES loses)
- Croatia 1-0 Panama — Croatia wins YES ✅, Draw NO ✅, Panama wins NO ✅

### Item 5 — WC_GAME June 26 Checkpoint Discussion ✅
Conclusion: June 26 = model review only, not trading approval. WC_GAME Brier 0.1858 is healthy but 35.3% WR reflects model systematically flagging NO signals on favorites who keep winning. Root cause likely lambda weighting for strong favorites (same Spain/Cape Verde problem). Review lambda values and signal selection logic before considering any live trading of WC_GAME signals.

---

## Calibration Report (post-session scorer run)

| Model | N | Accuracy | Brier | Status |
|-------|---|----------|-------|--------|
| GDP | 4 | 50.0% | 0.4566 | ⚠️ small sample |
| JOBS | 81 | 58.0% | 0.1351 | ✅ |
| SOCCER_GAME | 142 | 45.1% | 0.1495 | ✅ |
| WC_GAME | 949 | 35.3% | 0.1858 | ✅ (calibrated despite low WR) |
| **OVERALL** | **1176** | **38.1%** | **0.1788** | ✅ |

Edge bucket health (above threshold signals):
- 8-11c: 27.3% WR, Brier 0.1493 ✅
- 12-15c: 49.5% WR, Brier 0.2061 ✅
- 16-24c: 51.2% WR, Brier 0.2359 ✅

---

## Go-Live Milestone Tracker

| Milestone | Target | Current | Status |
|-----------|--------|---------|--------|
| Closed trades | 75 | 0 (SEDE) / 19 (paper) | 🔴 Pre-live |
| Win rate | >55% | — / 47.4% paper | 🟡 |
| Brier | <0.20 | 0.1788 overall | ✅ |
| Subscriber alert system | Live | ✅ Complete | ✅ |
| on_signal_resolved() | Live | ✅ Complete | ✅ |
| MLB ticker fix | Live | ✅ Tonight | ✅ |

---

## Pending Items (next session)

1. **Panama/Croatia scored tonight** — no further WC backfill needed until June 24 games complete
2. **June 24 games to score next session:** Colombia vs DR Congo (9 PM CT tonight — don't wait), Switzerland vs Canada, Bosnia vs Qatar, Scotland vs Brazil, Morocco vs Haiti
3. **Threshold auto-apply (MIN_EDGE_CENTS)** — deferred, own focused session
4. **NFL model build window:** July–August 2026, target before Sep 3 kickoff
5. **WC Phase 2 (KXWCGAME Monte Carlo):** June 26–July 4 window

---

## Key Dates
- **June 26:** WC group stage ends — model review checkpoint
- **July 4:** WC knockout Phase 2 window closes
- **~Aug 2:** Vegas demo with Anthony (8rain)
- **Sep 3:** NFL season kickoff — NFL model must be ready

---

*Generated by Archie (Claude Desktop) | Push from laptop → Oracle pulls at 02:00 CT cron*
