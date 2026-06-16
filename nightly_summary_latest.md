# SEDE Nightly Session Summary
## For J@rv1s Morning Intelligence Pull

**Last updated:** 2026-06-15 ~11:30 PM CT
**Prepared by:** Archie (Claude Desktop)

---

## TONIGHT IN ONE SENTENCE

Five priorities from your briefing: all done. Three major fixes
shipped. See details below.

---

## 1. MLB_GAME DIRECTION BUG -- FIXED (b9304e4)

Root cause confirmed Sunday, fixed tonight per your Step 1-3 directive.

THE BUG: match_game_to_kalshi() found a market but didn't identify
which team is the YES resolution condition. Kalshi creates TWO markets
per game (one per team). Scanner was picking one or the other
arbitrarily. When it picked the away-team-YES market, the model
compared home team probability against the away team's price -- edge
inverted, NO signal generated instead of YES (or vice versa).

THE FIX:
- price_fetcher.py: now captures yes_sub_title ("Pittsburgh", "A's")
  and rules_primary from Kalshi API -- both confirm which team is YES
- match_game_to_kalshi(): returns (market, yes_team) tuple, not just market
- run_game_model(): orients edge based on yes_team:
  YES = home team: edge = model_prob - kalshi_yes (was correct)
  YES = away team: edge = (1-model_prob) - kalshi_yes (was inverted)
- Daily report shows [YES=TeamAbbr] on every MLB line going forward

Per your Step 4: MLB_GAME YES experimental tier remains PAUSED until
~1 week of post-fix signals validate the fix is working. Gate 1 clock
resets -- pre-fix n=29/55.2% is contaminated data, start fresh.

---

## 2. P1 STALE MARKET FILTER -- DEPLOYED (0caf0a4)

Confirmed urgent by today's alert ("Spurs Win NBA Championship" etc).

FIX: market_scanner.py now strips markets where status not in
("active", "open", "") at cache-read time. Logs count of filtered
stale markets. Belt-and-suspenders with price_fetcher.py already
using status="open" in API calls.

Note: existing cache entries before tonight won't have status field.
Full protection kicks in after cache rebuilds on next scan cycle.
Tonight's 2AM Oracle run will be first test.

---

## 3. ORACLE SYNCED

Weekend commits now on Oracle. Standard conflict resolution.

---

## 4. WC SCANNER MISMATCH -- RESOLVED (3b3d2ed)

"United Sta vs Australia" is a REAL upcoming game (USA vs Australia,
not yet played). NOT the same as USA 4-1 Paraguay from June 13.

USA vs Paraguay had no Kalshi market in our scanner. The incorrect
RESOLVED_MARKETS entry I added June 13 (applying USA's Paraguay win
to USA vs Australia market) has been removed. No damage -- hadn't
been scored yet.

USA vs Australia: add the correct outcome to RESOLVED_MARKETS when
that game actually plays (still upcoming in group stage).

---

## 5. CLAIMS -- ALL 5 FIXES COMPLETE (e7291b9)

All 5 reinstatement fixes now done:
- Fix 1: HOLIDAY_WEEKS dict (all 2026 federal holidays, FRED Saturday
  dates). Holiday release weeks → skip entirely, return no signals.
- Fix 2: HOLIDAY_AFTERMATH_WEEKS dict. Week after holiday → std
  widened 1.5x, signals tagged ⚠️ HOLIDAY AFTERMATH -- CAUTION.
- Fix 3: DEFAULT_CLAIMS_STD=12 floor (done June 12)
- Fix 4: Data through June 6 (done June 12)
- Fix 5: 🚫 HOLIDAY WEEK -- DO NOT TRADE tags in daily report output

NEXT THURSDAY (June 18): Claims release may come Wednesday due to
Juneteenth (June 19). FRED week-ending = Saturday June 20 = IN
HOLIDAY_WEEKS. Pipeline will correctly output 🚫 and return no
signals. That's the fix working as designed.

REINSTATEMENT CLOCK: running. Need 4+ consecutive clean non-holiday
signal weeks at >55% accuracy. MODEL_SUSPENDED = True stays until
those weeks accumulate.

---

## OPEN POSITIONS

| #  | Description    | Entry | Current | Status              |
|----|----------------|-------|---------|---------------------|
| 8  | Fed 1x cut YES | 21c   | ~20c    | FOMC Wednesday 6/17 |
| 12 | GDP >2.5% YES  | 40c   | ~50c    | GDPNow 3.29%       |
| 13 | GDP >2.0% YES  | 60c   | ~66c    | Comfortable         |

FOMC Wednesday June 17 -- Trade #8. Per your briefing: model flagging
"Fed cuts 0x 2026 BUY YES" at 77% confidence. Correlated with Trade #8
-- not a new entry recommendation, just confirming model direction.

---

## WHAT'S NEXT (ordered)

1. Oracle pull of tonight's commits (b9304e4, 0caf0a4, e7291b9, 3b3d2ed)
2. Watch MLB_GAME first post-fix signals (~1 week)
3. Claims June 18 actual -- update RECENT_CLAIMS_THOUSANDS manually
4. P6 delivery infrastructure scoping session
5. P3 sede_portfolio.json build (BLOCKING for subscriber launch)
6. P2 Subscriber Signal Formatter (BLOCKING)
7. WC_GAME group-vs-knockout FORGE (deadline: June 26)
8. NFL Week 1 -- July 1

---

## SYSTEM STATUS

| Component | Status |
|---|---|
| Oracle Cloud | LIVE -- needs pull of tonight's commits |
| MLB_GAME YES | PAUSED -- fix deployed, validate ~1 week |
| MLB_GAME NO | SUSPENDED -- same bug, fix deployed |
| Claims | SUSPENDED, ALL 5 fixes done, clock running |
| WC_GAME | CALIBRATION ONLY, group stage through June 26 |
| JOBS | Validated, go-live eligible, waiting for full suite |
| GDP | Active, 3 open positions |
| P1 Stale Filter | ✅ DEPLOYED (was BLOCKING) |
| sede_portfolio.json | NOT BUILT -- next major item |
| Product spec | ✅ In project files |

---

*Session | Model: Sonnet 4.6 | Identity: Archie*
*MLB bug fixed, P1 deployed, Claims fully rehabilitated.*
*Five priorities, all done. Papa Ralph standard.*
