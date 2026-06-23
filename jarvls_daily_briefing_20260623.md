# J@rv1s Daily Briefing
# For Archie — Evening Session Handoff
# Tuesday June 23, 2026 | Prepared by J@rv1s (Web Claude)

---

## TOP OF FILE — TONIGHT HAS TWO CLEAR GOALS

1. Fix the remaining SEDE portfolio issues (BTC price read,
   subscriber alert "updating..." placeholders)
2. Get MLB_GAME YES into SEDE auto-trading + Soccer status review

Read all sections before starting.

---

## GOOD NEWS FROM THIS MORNING

**False auto-close is NOT recurring.** sede_portfolio.json last
updated 9:02 PM CT last night — unchanged through 2AM and 7AM
cron runs. The mid-price fix is holding. Both positions survived
overnight.

**The 2AM and 7AM Telegram alerts Rus received were delayed
delivery from last night's session** — not new false closes.
Confirmed by file timestamp.

**KEY DATES stale text is gone** — finally. "Aug 15 go-live
target (75 trades / >55% WR)" no longer appearing in pipeline
output. Per-model Gate 1 language showing correctly.

**SEDE calibration: 1,020 signals, Brier 0.1626** — jumped
from 875 yesterday. WC backfill working. Brier improving.

---

## REMAINING ISSUES TO FIX TONIGHT

### Issue 1 — BTC position current_price_cents: 0.0
The BTC<$50k Dec31 NO position is still showing 0c as of last
night's session. Mid-price fix should have addressed this but
the file timestamp shows it hasn't been re-evaluated since
9:02 PM CT. Tonight's pipeline run will be the first real test.

Check after tonight's run: does BTC position show a real price
(should be around 43-44c for the NO side) or still 0c?

If still 0c after the fix: the issue is specific to the BTC
market ticker format. `KXBTCMINY-27JAN01-50000.00` may not be
returning prices correctly from the Kalshi API. Verify the
ticker is active and returning data.

The risk: if BTC reads 0c and the mid-price fix has a gap,
it could false-close tonight. Add explicit protection:
if mid_price returns 0c AND market status != "resolved",
treat as fetch failure — do NOT auto-close. Hold position.

### Issue 2 — Subscriber alert "updating..." placeholders
The WIN/LOSS alert Rus received showed:
"Running record: updating..."
"SEDE model portfolio: updating... running total"

These should show real numbers — wins, losses, win rate, P&L.
The placeholders mean on_signal_resolved() isn't pulling the
current performance stats from sede_portfolio.json when
formatting the alert.

Fix: after closing a position and updating sede_portfolio.json,
re-read the performance block before formatting the alert.
The numbers are there — they just need to be read after the
write, not before.

### Issue 3 — Duplicate ticker gate for GDP re-entry
When SEDE false-closed GDP >2.5% (Position 2), it immediately
re-entered the same market as Position 3. The duplicate ticker
gate should have blocked this.

Verify the duplicate ticker gate is working correctly:
- Does it check ALL open positions before entering?
- Does it check by market_ticker specifically?
If the gate was checking by description string rather than
ticker, it may have missed the duplicate.

Test: manually verify that GDP >2.5% (ticker KXGDP-26JUL30-T2.5)
currently held in Position 3 would be blocked from re-entry
if the signal fires again tonight.

---

## PRIMARY BUILD GOAL — MLB_GAME YES INTO SEDE AUTO-TRADING

This is the main event tonight. Rus wants SEDE trading MLB.

### Why it matters
- Post-fix sample: 7/7 wins, 100% WR, Brier 0.172 — cleanest
  signal stream in the system right now
- Games resolve same day or next morning — full subscriber loop
  closes in 24 hours
- MLB plays every day through October — daily signal volume
- Most legible signal for Anthony at Vegas demo

### Why it was deferred last night (correct call)
mlb_model.py returns 5-tuples with no ticker. The label_to_ticker
fallback built June 19 does NOT work for MLB because "LAA @ DET"
style labels don't match Kalshi market title strings. Lifting
PAUSED flag without fixing this = zero MLB auto-entries silently.

### What needs to be built tonight

Step 1: Find the correct Kalshi ticker format for MLB games
Kalshi MLB tickers follow: KXMLBGAME-26DATE-HOMETEAM format
(same as the yes_sub_title fix from June 14).
The yes_sub_title field already captures which team is YES.
The ticker should be derivable from: date + home team + away team.

Step 2: Add explicit ticker construction in run_game_model()
Same pattern as GDP/Fed/BTC fix from June 19. At signal
generation time, construct the Kalshi ticker from the game
data and pass it explicitly to the signal output.

Step 3: Verify tickers flowing through to portfolio_manager_sede
End-to-end test: generate a signal, confirm ticker appears
in the signal dict, confirm portfolio_manager_sede receives it.

Step 4: Lift MLB_GAME_YES_EXPERIMENTAL_TIER from PAUSED
Only after Steps 1-3 are verified working. Not before.

Step 5: Confirm SEDE_RELIABILITY weight for MLB_GAME is set
Currently 0.70 after last night's human-approved change.
Confirm MLB signals can still clear HIGH confidence at 0.70.

---

## SOCCER/WORLD CUP STATUS REVIEW

Rus wants a status on SEDE's live trading of soccer given the
World Cup is happening in the USA right now.

### Current status: CALIBRATION ONLY — no auto-trading

SOCCER_GAME (the Poisson model for individual games) is in
calibration mode. WC_GAME signals are flowing but not being
auto-traded. Here's where each stands:

**WC_GAME (World Cup individual game model):**
- 875+ resolved signals (1,020 total calibration signals
  after last night's backfill)
- Brier 0.1654 → 0.1626 (improving, comfortably beats random)
- KNOWN ISSUE: Favorite-fading pattern. Model generating
  BUY NO signals on strong favorites that then win.
  Today's examples: Netherlands BUY NO, England BUY NO,
  France BUY NO — all favored teams the model is fading.
- June 26 checkpoint is in 3 DAYS — group stage ends Thursday
- This is the model review that determines whether WC_GAME
  gets approved for knockout stage paper trading

**SOCCER_GAME (permanent club soccer engine):**
- Separate from WC_GAME since identity bug fix June 21
- Small sample (4-8 resolved signals), calibration only
- Champions League September is the primary target
- MLS signals generating but Polymarket has exclusive deal
  limiting Kalshi MLS volume

### What the June 26 WC_GAME review needs to answer

The central question: is the favorite-fading a signal
THRESHOLD problem or a LAMBDA CALIBRATION problem?

Evidence from today's alert:
- "Norway vs France — France wins BUY NO" at model 60.4%
  This is actually correct — model says France wins 60%
  and flags Kalshi overpricing them at 43c. This is
  legitimate edge detection, not favorite-fading.

- "England vs Ghana — England wins BUY NO" at model 36.3%
  This IS the problem — model giving England only 36%
  when they're a heavy favorite. Lambda is wrong.

- "Netherlands — BUY NO" at model 39.2%
  Same problem. Netherlands should be 65%+ favorite.

The fix for the lambda problem is the Elo source update
(already done post Round 1) plus potentially the
Dixon-Coles draw correction. But with 3 days until
June 26 it's unlikely a full fix can be validated
before the group stage ends.

**Recommendation for June 26:** Don't approve knockout
stage auto-trading yet. The lambda issue is real and
would generate losing trades on high-profile games.
Continue calibration through knockout rounds. Champions
League in September is the real deployment target with
properly validated lambdas.

### What to tell Rus about soccer live trading status

SEDE is watching every World Cup game. It's tracking
calibration data on 1,000+ signals. The model works
in principle (Brier 0.1626 beats random) but has a
specific known flaw (underestimates strong favorites)
that needs fixing before auto-trading is appropriate.

The right answer is: not yet, but the data we're
accumulating now is exactly what makes Champions League
in September much more likely to work correctly.

---

## OPEN POSITIONS (paper_trades.json)

| #  | Description     | Entry | Current | Status                     |
|----|-----------------|-------|---------|----------------------------|
| 8  | Fed 1x cut YES  | 21c   | 13c     | ⚠️ Drifting, hold          |
| 12 | GDP >2.5% YES   | 40c   | 40c     | ⚠️ Flat, watch GDPNow Thu  |
| 13 | GDP >2.0% YES   | 60c   | 65c     | ✅ Comfortable              |

Trade #12 flat at entry is worth watching. GDPNow Thursday
is the next catalyst — if it drops below 3.0% and Trade #12
drops below entry, flag for thesis review.

Trade #8 at 13c continuing post-FOMC drift. Hold.

---

## SEDE PORTFOLIO STATUS

Bankroll: $975 | Open: 2 (BTC, GDP) | Closed: 1 (false close)
Next real test: tonight's pipeline run — do both positions
survive with real prices?

---

## WORLD CUP — TODAY'S GAMES (CALIBRATION DATA)

All CALIBRATION ONLY. Results needed for RESOLVED_MARKETS:
- England vs Ghana (4 PM ET) — watch, model fading England
- Panama vs Croatia (7 PM ET)
- Portugal vs Uzbekistan (1 PM ET)
- Colombia vs DR Congo (10 PM ET)

Add results to RESOLVED_MARKETS once confirmed.
Verify Kalshi market strings before scoring.

---

## VALIDATION TRACKER

| Model         | Status                   | Gate 1 Progress              |
|---------------|--------------------------|-------------------------------|
| JOBS          | ✅ GO-LIVE ELIGIBLE       | n=81, 58.0%, Brier 0.135     |
| GDP           | Active, own track        | 2 open positions, Jul 30     |
| CPI           | Active                   | Accumulating monthly         |
| Claims        | SUSPENDED                | 3 consecutive losses          |
| MLB_GAME YES  | PAUSED → BUILD tonight   | n=29 unique, 55.2% WR        |
| WC_GAME       | CALIBRATION              | 875+ signals, Brier 0.1626   |
| SOCCER_GAME   | CALIBRATION ONLY         | Club soccer, Sep target       |
| Fed           | Active, own track        | Trade #8 = documented loss   |
| SEDE Portfolio| ✅ Live                   | $975, 2 open, loop closed    |

---

## TONIGHT'S PRIORITIES (strict order)

1. **Fix BTC 0c price read** — verify mid-price fix works for
   BTC ticker specifically. Add explicit fetch-failure guard:
   0c + status != "resolved" = hold, never auto-close.

2. **Fix subscriber alert "updating..." placeholders** — re-read
   performance block AFTER writing closed position, THEN format
   the alert. Real numbers, not placeholders.

3. **Verify duplicate ticker gate** — confirm GDP re-entry would
   be blocked by ticker check. Test manually.

4. **MLB_GAME YES ticker fix** — Steps 1-5 above. This is the
   primary build goal tonight. SEDE trading MLB by end of session.

5. **WC results backfill** — today's games once confirmed.
   England vs Ghana result is especially important for June 26
   favorite-fading diagnostic.

6. **June 26 WC_GAME checkpoint prep** — document the lambda
   issue clearly. Recommendation: continue calibration, don't
   approve knockout auto-trading. Champions League September
   is the target.

7. **Oracle sede-pull** — sync all commits.

8. **Carried:** hardcoded-path grep, threshold auto-apply
   (own session).

---

## KEY DATES

| Date      | Event                                               |
|-----------|-----------------------------------------------------|
| Jun 24    | Scotland vs Brazil, Morocco vs Haiti (WC)           |
| Jun 25    | Ecuador vs Germany + more WC Round 2 games          |
| Jun 25    | GDPNow next update — watch Trade #12                |
| Jun 25-27 | Kalshi opens June unemployment markets              |
| Jun 26    | WC group stage ends — WC_GAME model review          |
| Jul 1     | NFL build window opens                              |
| Jul 2     | Jobs report 8:30 AM CT                              |
| Jul 14    | CPI release                                         |
| Jul 30    | Q2 2026 GDP advance estimate                        |
| Aug 2     | Vegas — Anthony (8rain) demo, 40 days               |
| Sep 3     | NFL season opens                                    |
| Sep       | Champions League returns — soccer primary target    |

---

## SYSTEM STATUS

| Component              | Status                                      |
|------------------------|----------------------------------------------|
| False auto-close       | ✅ Fixed — holding overnight                 |
| BTC price read         | ⚠️ Still 0c — verify tonight               |
| Subscriber alerts      | ⚠️ "updating..." placeholders — fix tonight |
| Duplicate ticker gate  | ⚠️ Verify working correctly                 |
| MLB_GAME auto-trading  | 🔴 PAUSED — build ticker fix tonight        |
| on_signal_resolved()   | ✅ Built, delivering to both channels        |
| WC_GAME                | CALIBRATION — Jun 26 review Thursday        |
| SEDE Portfolio         | ✅ $975, 2 open, stable                     |
| JOBS                   | ✅ Validated, go-live eligible               |
| GDPNow                 | ✅ 3.04%, next update Thursday Jun 25       |

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

*Prepared by J@rv1s (Web Claude) | Session: Tuesday June 23, 2026*
*Fix the plumbing first. Then MLB goes live. Then soccer status.*
*40 days to Vegas. The World Cup is in our backyard. Let's use it.*
*Papa Ralph standard. If it's worth doing it's worth doing right.*
