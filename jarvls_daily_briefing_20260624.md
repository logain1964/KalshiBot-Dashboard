# J@rv1s Daily Briefing
# For Archie — Evening Session Handoff
# Tuesday June 24, 2026 | Prepared by J@rv1s (Web Claude)

---

## TOP OF FILE — THREE GOALS TONIGHT

1. Fix BTC current_price_cents: 0.0 (last remaining plumbing issue)
2. Run the WC_GAME diagnostic script (pre-June 26 review data)
3. WC results backfill for today's six games

Everything else is secondary. Read all sections before starting.

---

## SEDE PORTFOLIO — CONFIRMED STABLE

sede_portfolio.json last updated 7:02 AM CT June 24.
Bankroll: $1,000 | Open: 2 | Closed: 0 | P&L: $0.00

Both positions survived overnight and morning cron runs clean.
The mid-price fix is holding. No false closes.

REMAINING ISSUE: Both positions show current_price_cents: 0.0
as of 7:02 AM CT. Price monitoring still not working for either
BTC or GDP position. This is the last significant plumbing issue.

RISK: If a position genuinely resolves to 0c overnight, SEDE
may not distinguish "real resolution" from "fetch returning 0c."
The safety guard (requires yes_ask also ≤3c OR status==resolved)
should handle this — but needs verification tonight.

FIX NEEDED:
- Check why price fetch returns 0c for both tickers
- BTC ticker: KXBTCMINY-27JAN01-50000.00
  BTC is ~$62,250 tonight. NO side should be worth ~44c.
- GDP ticker: KXGDP-26JUL30-T2.5
  GDP >2.5% should be ~40-48c range.
- If mid-price fix is deployed but still returning 0c:
  the issue may be in HOW the mid-price is calculated
  when bid=0 and ask=0 (empty orderbook = 0+0/2 = 0)
- Add explicit fetch-failure guard: if mid_price == 0c
  AND market status != "resolved" → treat as fetch failure,
  log WARNING, do NOT use for auto-close evaluation.
  Hold position. Try again next cycle.

---

## MLB_GAME YES — STATUS CHECK

Ticker fix deployed June 23 (commit 869badc). The 7AM cron
ran this morning with the fix in place. Check daily_report
for any MLB_GAME YES signals that evaluated through the
auto-entry gates.

If no MLB entry yet: that's fine — gates are selective.
The right game with edge ≥20c and confidence ≥60% will
enter automatically when it appears. Do not force an entry.

If an MLB entry DID fire: note the market, ticker, edge,
and confidence. This is SEDE's first baseball trade.

---

## WC_GAME DIAGNOSTIC SCRIPT — RUN TONIGHT (PRIORITY)

J@rv1s and Rus spent time today brainstorming four theories
about why WC_GAME has 35.3% WR on 949 signals despite good
Brier (0.1858). A diagnostic script was written to test all
four theories against the actual signal history before the
June 26 model review.

Script location: /mnt/user-data/outputs/wc_game_diagnostic.py
(Rus will provide or it can be fetched from today's outputs)

Run from /home/ubuntu/KalshiBot/:
  python3 wc_game_diagnostic.py

THE FOUR THEORIES BEING TESTED:

Theory 1 — Draw Underweighting
Do WC_GAME losses cluster on draw results vs outright wrong
calls? If draw losses dominate → Dixon-Coles rho adjustment
is the fix.

Theory 2 — Lambda Compression (Favorite Underestimation)
Do YES signals where model gives ≤45% win probability
actually resolve as wins at >50% rate? If so → lambda
conversion is too flat, underestimating strong teams.
Evidence: Spain 24.6% vs Cape Verde, England 36.3% vs Ghana,
Netherlands 39.2% vs Tunisia. All heavily favored teams.

Theory 3 — Signal Threshold Too Low
Do low-edge signals (15-20c) perform significantly worse
than high-edge signals (25c+)? If so → raise minimum
threshold for WC_GAME from 15c to 20c.

Theory 4 — Retail Bias on Underdog Pricing
Do NO signals where Kalshi has the faded team at >35c
perform differently from NO signals where Kalshi has them
at <20c? Separates retail bias from genuine edge.

BONUS ANALYSES:
- YES vs NO direction split (is one direction worse?)
- Performance when fading strong favorites (Kalshi ≥60c)
  This last one is probably the most revealing.

OUTPUT: Paste the full script output into J@rv1s morning
chat before the June 26 review session. We walk into
Thursday with actual data, not just theories.

---

## WC_GAME JUNE 26 REVIEW FRAMEWORK (2 days out)

Group stage ends Thursday. Here is the structured agenda
for the review — not an open-ended "what's wrong" session.

AGENDA FOR JUNE 26:

1. Diagnostic script results (from tonight's run)
   — Which theories confirmed, which rejected?
   — What does the data say the fix should be?

2. Lambda investigation
   — Pull actual λ values for England, Netherlands, Spain
     vs their opponents from the diagnostic run
   — If Spain λ ≈ Cape Verde λ, the curve is too flat
   — Fix: steeper Elo-to-lambda conversion for quality
     gaps > 150 Elo points

3. Signal selection threshold review
   — Minimum win probability threshold:
     Don't generate NO on a team unless model assigns
     opponent ≥65% win probability
     Don't generate YES on underdog unless ≥40% probability
   — This filters soft-edge signals where draw risk is highest
   — One line change, testable immediately on historical data

4. Dixon-Coles calibration check
   — Current DC_RHO=-0.15
   — International soccer may need -0.20 to -0.25
   — Small adjustment could meaningfully reduce draw
     underweighting without a full model rebuild

5. VERDICT (to be decided at review):
   — Continue calibration through knockout rounds?
   — Approve limited knockout paper trading?
   — Champions League September as primary target?

EXPECTED VERDICT based on current data:
Continue calibration. Do NOT approve knockout auto-trading.
The lambda issue is real. High-profile knockout games are
the worst place to discover a miscalibrated model.
Champions League September with properly validated lambdas
is the right deployment target.

---

## WORLD CUP — TODAY'S GAMES (June 24, results pending)

Add to RESOLVED_MARKETS once confirmed.
Verify Kalshi market strings before scoring.

3 PM ET games:
- Switzerland vs Canada
- Bosnia vs Qatar

6 PM ET games:
- Scotland vs Brazil ← KEY for WC_GAME calibration
  Model has been flagging Brazil wins BUY NO repeatedly.
  Brazil is a heavy favorite. Result = direct lambda data.
- Morocco vs Haiti

9 PM ET games:
- Czechia vs Mexico
- South Africa vs South Korea

---

## YESTERDAY'S CONFIRMED RESULTS (June 23)

Add any not yet in RESOLVED_MARKETS:
- England 0-0 Ghana (draw — neither wins, model had England
  BUY NO signal. Draw = model missed draw probability again)
- Portugal 5-0 Uzbekistan (Portugal dominant, Ronaldo 2 goals)
- Colombia 1-0 DR Congo
- Panama 0-1 Croatia

---

## OPEN POSITIONS (paper_trades.json — Rus validation record)

| #  | Description     | Entry | Current | Status                     |
|----|-----------------|-------|---------|----------------------------|
| 8  | Fed 1x cut YES  | 21c   | ~13c    | ⚠️ Documented loss, hold   |
| 12 | GDP >2.5% YES   | 40c   | ~40c    | ⚠️ Flat, GDPNow Thu        |
| 13 | GDP >2.0% YES   | 60c   | ~65c    | ✅ Comfortable              |

GDPNow next update TOMORROW June 25. Same day Kalshi opens
June unemployment markets and JOBS signals resume.
If GDPNow drops below 3.0%, Trade #12 needs a thesis review.

---

## VALIDATION TRACKER

| Model         | Status                   | Gate 1 Progress              |
|---------------|--------------------------|-------------------------------|
| JOBS          | ✅ GO-LIVE ELIGIBLE       | n=81, 58.0%, Brier 0.135     |
| GDP           | Active, own track        | 2 open positions, Jul 30     |
| CPI           | Active                   | Accumulating monthly         |
| Claims        | SUSPENDED                | 3 consecutive losses          |
| MLB_GAME YES  | ✅ TICKER FIXED           | First auto-entry possible     |
| WC_GAME       | CALIBRATION              | 949 signals, Brier 0.1858    |
| SOCCER_GAME   | CALIBRATION ONLY         | 142 signals, Brier 0.1495    |
| Fed           | Active, own track        | Trade #8 = documented loss   |
| SEDE Portfolio| ✅ Live                   | $1,000, 2 open, stable       |

---

## TONIGHT'S PRIORITIES (strict order)

1. BTC/GDP current_price_cents: 0.0 fix
   — Diagnose why mid-price returning 0c for both positions
   — Add fetch-failure guard (0c + not resolved = hold)
   — Verify positions NOT at risk of false auto-close

2. Run wc_game_diagnostic.py on Oracle
   — Full output paste into session archive
   — Paste output into nightly summary for J@rv1s morning

3. WC results backfill
   — Today's six games once results confirmed
   — Yesterday's England/Portugal/Colombia/Panama
   — Scotland vs Brazil result is highest priority
     (direct lambda calibration data point)

4. MLB auto-entry status check
   — Did any MLB signal clear gates from today's run?
   — If yes: log the entry, note what triggered it

5. Oracle sede-pull — sync any commits

6. Carried low priority: threshold auto-apply (own session),
   hardcoded-path grep

---

## KEY DATES

| Date      | Event                                               |
|-----------|-----------------------------------------------------|
| Jun 25    | GDPNow next update — watch Trade #12                |
| Jun 25    | Kalshi opens June unemployment markets              |
| Jun 26    | WC group stage ends — WC_GAME model review          |
| Jul 1     | NFL build window opens                              |
| Jul 2     | Jobs report 8:30 AM CT                              |
| Jul 14    | CPI release                                         |
| Jul 30    | Q2 2026 GDP advance estimate                        |
| Aug 2     | Vegas — Anthony (8rain) demo, 39 days               |
| Sep 3     | NFL season opens                                    |
| Sep       | Champions League returns — soccer primary target    |

---

## SYSTEM STATUS

| Component              | Status                                      |
|------------------------|----------------------------------------------|
| False auto-close       | ✅ Fixed — holding stable                    |
| BTC/GDP price read     | ⚠️ 0.0c — fix tonight                      |
| MLB_GAME auto-trading  | ✅ Ticker fixed — awaiting right signal      |
| Subscriber alerts      | ✅ Working, real numbers on next resolution  |
| Duplicate ticker gate  | ✅ Tightened                                 |
| WC diagnostic script   | 🔴 NOT RUN YET — tonight                   |
| WC_GAME                | CALIBRATION — Jun 26 review Thursday        |
| SEDE Portfolio         | ✅ $1,000, 2 open, stable                   |
| JOBS                   | ✅ Validated, go-live eligible               |
| GDPNow                 | ✅ 3.04%, next update tomorrow              |

---

## CONFIRMED API REFERENCE (standing section)

| API              | Base URL                             | Auth    | Notes             |
|------------------|--------------------------------------|---------|-------------------|
| Kalshi           | external-api.kalshi.com/trade-api/v2 | RSA key | Already integrated|
| Polymarket Gamma | gamma-api.polymarket.com             | None    | 10 req/sec        |
| ClubElo          | api.clubelo.com                      | None    | CSV, clubs only   |
| eloratings.net   | eloratings.net                       | None    | National teams    |
| GDPNow           | atlantafed.org/cqer/research/gdpnow  | None    | Next update Jun 25|

---

## From J@rv1s
Create this file on Oracle at /home/ubuntu/KalshiBot/wc_game_diagnostic.py — paste the script content directly, then run it.

#!/usr/bin/env python3
"""
WC_GAME Signal Diagnostic — June 26 Review Prep
SEDE-SEEKS Intelligence Network
Run from: /home/ubuntu/KalshiBot/
Usage: python3 wc_game_diagnostic.py

Tests four theories about WC_GAME's 35.3% win rate:
  Theory 1 — NO signals losing to draws (draw underweighting)
  Theory 2 — Model underestimating strong favorites (lambda compression)
  Theory 3 — Low edge signals performing worse than high edge
  Theory 4 — Retail bias on underdog pricing
"""

import csv
import json
from pathlib import Path
from collections import defaultdict

# ── Paths ──────────────────────────────────────────────────────────────────
BASE = Path(__file__).parent
SIGNALS_LOG = BASE / "data" / "signals_log.csv"
RESOLVED_MARKETS = BASE / "data" / "resolved_markets.json"

# ── Load resolved outcomes ─────────────────────────────────────────────────
def load_resolved():
    """Returns dict of market_label -> outcome (1=correct, 0=wrong)"""
    try:
        with open(RESOLVED_MARKETS) as f:
            data = json.load(f)
        return data
    except Exception as e:
        print(f"ERROR loading resolved markets: {e}")
        return {}

# ── Load WC_GAME signals ───────────────────────────────────────────────────
def load_wc_signals(resolved):
    """
    Reads signals_log.csv, filters to WC_GAME resolved signals only.
    Returns list of dicts with all fields needed for analysis.
    """
    signals = []
    if not SIGNALS_LOG.exists():
        print(f"ERROR: {SIGNALS_LOG} not found")
        return signals

    with open(SIGNALS_LOG, newline="", encoding="utf-8") as f:
        reader = csv.DictReader(f)
        for row in reader:
            model = row.get("model_name", "").strip()
            if model not in ("WC_GAME", "SOCCER_GAME"):
                continue

            label = row.get("signal_label", "").strip()
            direction = row.get("direction", "").strip().upper()
            outcome = row.get("actual_outcome", "").strip()

            # Only include resolved signals
            if outcome not in ("1", "0", 1, 0):
                continue

            try:
                model_prob = float(row.get("model_probability_pct", 0))
                kalshi_price = float(row.get("kalshi_price_cents", 0))
                edge = float(row.get("edge_cents", 0))
            except (ValueError, TypeError):
                continue

            signals.append({
                "model": model,
                "label": label,
                "direction": direction,
                "model_prob": model_prob,
                "kalshi_price": kalshi_price,
                "edge": edge,
                "outcome": int(outcome),
                "close_reason": row.get("close_reason", "").strip(),
            })

    return signals

# ── Analysis helpers ───────────────────────────────────────────────────────
def pct(wins, total):
    if total == 0:
        return 0.0
    return round(wins / total * 100, 1)

def summarize(label, signals):
    total = len(signals)
    wins = sum(s["outcome"] for s in signals)
    losses = total - wins
    print(f"  {label}: {wins}W / {losses}L / {total} total — WR: {pct(wins,total)}%")
    return wins, total

# ── Theory 1 — Draw underweighting ────────────────────────────────────────
def theory_1_draw_analysis(signals):
    print("\n" + "="*60)
    print("THEORY 1 — Draw Underweighting")
    print("Do WC_GAME losses cluster on draws vs outright wrong calls?")
    print("="*60)

    draw_losses = []
    outright_losses = []
    wins = []

    for s in signals:
        if s["outcome"] == 1:
            wins.append(s)
        else:
            reason = s.get("close_reason", "").lower()
            if "draw" in reason or "drew" in reason:
                draw_losses.append(s)
            else:
                outright_losses.append(s)

    total = len(signals)
    print(f"\n  Total resolved signals: {total}")
    print(f"  Wins: {len(wins)} ({pct(len(wins), total)}%)")
    print(f"  Losses — draw result: {len(draw_losses)} ({pct(len(draw_losses), total)}%)")
    print(f"  Losses — outright wrong: {len(outright_losses)} ({pct(len(outright_losses), total)}%)")

    if len(draw_losses) > len(outright_losses):
        print("\n  ⚠️  THEORY 1 CONFIRMED: Most losses are draws, not outright wrong calls.")
        print("     Fix: Dixon-Coles rho adjustment or minimum win prob threshold.")
    elif draw_losses:
        print(f"\n  🟡 THEORY 1 PARTIAL: {len(draw_losses)} draw losses present but not dominant.")
    else:
        print("\n  ✅ THEORY 1 NOT CONFIRMED: Losses are not primarily draws.")
        print("     Note: close_reason may not tag draws — check manually if uncertain.")

# ── Theory 2 — Lambda compression ─────────────────────────────────────────
def theory_2_lambda_analysis(signals):
    print("\n" + "="*60)
    print("THEORY 2 — Lambda Compression (Favorite Underestimation)")
    print("Do signals where model_prob <= 45% (should be favorites)")
    print("actually resolve as wins at a high rate?")
    print("="*60)

    # YES signals where model gives team < 50% (potential favorites being underestimated)
    buckets = {
        "model 0-35% YES": [],
        "model 36-45% YES": [],
        "model 46-55% YES": [],
        "model 56-65% YES": [],
        "model 66-80% YES": [],
        "model 80%+ YES":   [],
        "NO signals":       [],
    }

    for s in signals:
        if s["direction"] == "NO":
            buckets["NO signals"].append(s)
            continue
        p = s["model_prob"]
        if p <= 35:
            buckets["model 0-35% YES"].append(s)
        elif p <= 45:
            buckets["model 36-45% YES"].append(s)
        elif p <= 55:
            buckets["model 46-55% YES"].append(s)
        elif p <= 65:
            buckets["model 56-65% YES"].append(s)
        elif p <= 80:
            buckets["model 66-80% YES"].append(s)
        else:
            buckets["model 80%+ YES"].append(s)

    print()
    flag_lambda = False
    for label, bucket in buckets.items():
        if not bucket:
            continue
        w, t = summarize(label, bucket)
        # Flag if low model_prob YES signals are winning at high rate
        if "0-35" in label or "36-45" in label:
            actual_win_pct = pct(w, t)
            if actual_win_pct > 50:
                flag_lambda = True

    if flag_lambda:
        print("\n  ⚠️  THEORY 2 CONFIRMED: Low model_prob YES signals winning > 50%.")
        print("     Model is underestimating these teams — lambda conversion too flat.")
        print("     Fix: Steepen Elo-to-lambda curve for quality gap > 150 Elo points.")
    else:
        print("\n  ✅ THEORY 2 NOT CONFIRMED at this level.")
        print("     Lambda compression may still exist — check manually with specific games.")

# ── Theory 3 — Edge threshold too low ─────────────────────────────────────
def theory_3_edge_analysis(signals):
    print("\n" + "="*60)
    print("THEORY 3 — Signal Threshold Too Low")
    print("Do low-edge signals (15-20c) perform worse than high-edge (25c+)?")
    print("="*60)

    edge_buckets = defaultdict(list)
    for s in signals:
        e = s["edge"]
        if e < 15:
            edge_buckets["<15c (below threshold)"].append(s)
        elif e < 20:
            edge_buckets["15-19c (low edge)"].append(s)
        elif e < 25:
            edge_buckets["20-24c (medium edge)"].append(s)
        elif e < 35:
            edge_buckets["25-34c (high edge)"].append(s)
        else:
            edge_buckets["35c+ (very high edge)"].append(s)

    print()
    results = {}
    for label in ["<15c (below threshold)", "15-19c (low edge)",
                  "20-24c (medium edge)", "25-34c (high edge)", "35c+ (very high edge)"]:
        bucket = edge_buckets.get(label, [])
        if not bucket:
            continue
        w, t = summarize(label, bucket)
        results[label] = pct(w, t)

    low = results.get("15-19c (low edge)", 0)
    high = results.get("25-34c (high edge)", results.get("35c+ (very high edge)", 0))

    if low < high - 10:
        print(f"\n  ⚠️  THEORY 3 CONFIRMED: Low-edge signals ({low}% WR) significantly")
        print(f"     underperform high-edge signals ({high}% WR).")
        print("     Fix: Raise minimum edge threshold from 15c to 20c for WC_GAME.")
    else:
        print(f"\n  ✅ THEORY 3 NOT CONFIRMED: Edge level not strongly predictive of WR.")

# ── Theory 4 — Retail bias ────────────────────────────────────────────────
def theory_4_retail_bias(signals):
    print("\n" + "="*60)
    print("THEORY 4 — Retail Bias on Underdog Pricing")
    print("Do signals where Kalshi overprices underdogs (>35c) perform")
    print("differently from signals where underdogs are cheap (<20c)?")
    print("="*60)

    # For NO signals (betting against the team): kalshi_price is the price
    # of the team we're fading. High kalshi_price on a team we're fading
    # means Kalshi thinks they're likely to win — retail is pricing them high.
    no_signals = [s for s in signals if s["direction"] == "NO"]

    overpriced = [s for s in no_signals if s["kalshi_price"] > 35]
    fairly_priced = [s for s in no_signals if 20 <= s["kalshi_price"] <= 35]
    underpriced = [s for s in no_signals if s["kalshi_price"] < 20]

    print()
    summarize("NO signals — Kalshi >35c (fading overpriced team)", overpriced)
    summarize("NO signals — Kalshi 20-35c (fading fairly priced team)", fairly_priced)
    summarize("NO signals — Kalshi <20c (fading underpriced team)", underpriced)

    if overpriced and underpriced:
        op_wr = pct(sum(s["outcome"] for s in overpriced), len(overpriced))
        up_wr = pct(sum(s["outcome"] for s in underpriced), len(underpriced))
        if abs(op_wr - up_wr) > 10:
            print(f"\n  ⚠️  THEORY 4 CONFIRMED: Meaningful difference in WR based on")
            print(f"     Kalshi pricing ({op_wr}% vs {up_wr}%).")
            print("     Retail bias is affecting signal quality.")
        else:
            print(f"\n  ✅ THEORY 4 NOT CONFIRMED: WR similar across Kalshi price levels.")

# ── Bonus — NO vs YES signal performance ──────────────────────────────────
def bonus_direction_split(signals):
    print("\n" + "="*60)
    print("BONUS — YES vs NO Signal Performance")
    print("Is one direction systematically worse than the other?")
    print("="*60)
    print()
    yes_sigs = [s for s in signals if s["direction"] == "YES"]
    no_sigs  = [s for s in signals if s["direction"] == "NO"]
    summarize("YES signals", yes_sigs)
    summarize("NO signals", no_sigs)

# ── Bonus — Model performance by Kalshi favorite level ────────────────────
def bonus_favorite_level(signals):
    print("\n" + "="*60)
    print("BONUS — Performance When Fading Strong vs Weak Favorites")
    print("BUY NO signals split by Kalshi price of team being faded")
    print("="*60)
    no_sigs = [s for s in signals if s["direction"] == "NO"]
    strong  = [s for s in no_sigs if s["kalshi_price"] >= 60]
    medium  = [s for s in no_sigs if 40 <= s["kalshi_price"] < 60]
    weak    = [s for s in no_sigs if s["kalshi_price"] < 40]
    print()
    summarize("Fading strong favorites (Kalshi >=60c)", strong)
    summarize("Fading medium favorites (Kalshi 40-59c)", medium)
    summarize("Fading weak favorites (Kalshi <40c)", weak)
    if strong:
        sw = pct(sum(s["outcome"] for s in strong), len(strong))
        if sw < 35:
            print(f"\n  ⚠️  Fading strong favorites winning only {sw}%.")
            print("     Model should NOT be generating NO signals on 60c+ favorites.")
            print("     Fix: Add minimum threshold — don't fade teams above 55c on Kalshi.")

# ── Main ───────────────────────────────────────────────────────────────────
def main():
    print("\n" + "="*60)
    print("WC_GAME SIGNAL DIAGNOSTIC — June 26 Review Prep")
    print("SEDE-SEEKS Intelligence Network")
    print("="*60)

    resolved = load_resolved()
    signals = load_wc_signals(resolved)

    if not signals:
        print("\nERROR: No resolved WC_GAME signals found.")
        print("Check that signals_log.csv exists and has actual_outcome populated.")
        return

    print(f"\nLoaded {len(signals)} resolved WC_GAME/SOCCER_GAME signals.")

    theory_1_draw_analysis(signals)
    theory_2_lambda_analysis(signals)
    theory_3_edge_analysis(signals)
    theory_4_retail_bias(signals)
    bonus_direction_split(signals)
    bonus_favorite_level(signals)

    print("\n" + "="*60)
    print("DIAGNOSTIC COMPLETE")
    print("Paste full output into J@rv1s morning chat for June 26 review.")
    print("="*60 + "\n")

if __name__ == "__main__":
    main()

*Prepared by J@rv1s (Web Claude) | Session: Tuesday June 24, 2026*
*Run the diagnostic. Fix the price read. Scotland vs Brazil tonight.*
*39 days to Vegas. June 26 review is Thursday. PRP all the way.*
*Papa Ralph standard. If it's worth doing it's worth doing right.*
