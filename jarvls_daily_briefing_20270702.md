# J@rv1s Daily Briefing
# For Archie — Evening Session Handoff
# Wednesday July 2, 2026 | Prepared by J@rv1s (Web Claude)

---

## TOP OF FILE — SESSION OPENER PROTOCOL CHANGE (PERMANENT)

### MANDATORY FIRST STEP — EVERY SESSION, NO EXCEPTIONS

Before ANY build work begins:
1. Rus manually runs `sede-pull` from the Oracle terminal
2. Archie confirms pull succeeded by checking latest commit hash
   matches GitHub
3. If pull fails → Archie provides manual commands, Rus executes
4. Session begins only after Oracle sync is confirmed

Archie does NOT spend session time diagnosing Oracle connectivity
issues he cannot resolve. Oracle sync is always a human-executed
step. Always. This is now a standing protocol, not a reminder.

Context: Last night's session was consumed by Archie attempting
to solve an Oracle connectivity problem that is architecturally
outside his capability. That cannot happen again. Two minutes at
the top of every session eliminates this variable permanently.

---

## JOBS REPORT — CONFIRMED NUMBERS (July 2, 8:30 AM CT)

NFP ACTUAL: +57,000 (SEDE consensus 110K — missed by 53K)
UNEMPLOYMENT ACTUAL: 4.2% (SEDE consensus 4.2% — PERFECT CALL)

**Key context:**
- May NFP revised DOWN from 172K to 129K (-43K revision)
- April NFP revised DOWN from 179K to 148K (-31K revision)
- April + May combined 74K lower than previously reported
- May's "blockbuster" print was partly illusory
- Leisure & hospitality: -61K in June (World Cup hiring reversed)

**JOBS model performance — first live test:**
- Unemployment call: DEAD-ON (4.2% actual vs 4.2% consensus)
- NFP call: significantly below (57K vs 110K consensus)
- ADP (98K) was directionally right but BLS came in even lower

This is genuinely useful calibration data. The model's unemployment
consensus was precise. NFP directional signal (weak) was correct
but the magnitude was harder to call. Log both for the record.

**Trade #8 (Fed 1x cut YES @ 21c):**
57K NFP is weak enough to keep Fed cuts on the table in theory.
But Warsh's hawkish stance + PCE inflation at 3.6% likely
overrides one weak jobs print. Trade #8 probably doesn't move
materially. Hold to December 31 resolution.

**JOBS signals tonight:** Pipeline should generate signals on
June unemployment markets based on today's 57K actual vs
consensus. Check tonight's 9PM pipeline run output for JOBS
signal activity. This is what the KXPAYROLLS fix was for.

---

## GDPNOW — NOW AT 1.1889%

Last update: July 1. Next update: July 7.
GDPNow has fallen from 4.3% (May 21) to 1.19% today.
That trajectory is the single biggest threat to Trade #13.

### Trade #13 (GDP >2.0% YES @ 60c, currently 56c)
GDPNow at 1.19% is 81bps BELOW the 2.0% threshold.
Average absolute error: 0.77pp. The error band barely covers
the gap to 2.0%. This is no longer a comfortable hold.

ARCHIE'S TASK TONIGHT: Check live Kalshi price on
KXGDP-26JUL30-T2.0 before the 9PM pipeline run.

If market has dropped below 45c → serious consideration
to close. Taking a controlled loss at 45c (-$15 on position)
is better than riding to 0c (-$60).

If market is still holding 55c+ → the market is pricing more
uncertainty than GDPNow implies. Hold and watch July 7 update.

This is Rus's call ultimately. But don't let this drift without
checking the live price tonight.

---

## NFL AMENDMENT — OUTSTANDING ITEMS BEFORE RATIFICATION

The drafts exist. Rus has not yet re-read them post-J@rv1s audit.
The ratification decision is Rus's call. Before Rus signs off,
four specific items from the J@rv1s independent second pass
need to be in the documents:

**Item 1 — Sportsbook anchor answer (most important)**
Add this exact sentence to the amendment:
"Signals are only generated when SEDE's proprietary adjustment
layer causes the ≥20c divergence from Kalshi's implied
probability. If the base sportsbook line and Kalshi are already
divergent by ≥20c before proprietary adjustments are applied,
no signal is generated."

This closes the MLB_GAME loophole. Without it, the amendment
doesn't resolve the question that caused MLB_GAME's PENDING status.

**Item 2 — July 25 as explicit architectural decision point**
Add: "Probability architecture selection is deferred to July 25,
2026. Weeks 1-3 build is architecture-independent."
Currently implied but not stated. Make it explicit.

**Item 3 — Signal Contract deferral**
J@rv1s recommends building NFL to Contract from day one.
Amendment should either accept this or explicitly reject it
with a counter-argument. "Deferred" without rationale isn't
good enough for a permanent reference document.

**Item 4 — Three missing BIAS entries**
BIAS 11: Divisional familiarity effect
BIAS 12: Injury report gamesmanship (coaches gaming report)
BIAS 13: Primetime game selection bias
These need to be in the integrity check BIAS matrix.

STATUS: DRAFT. Not ratified. Not pushed to git.
Archie adds the four items tonight. Rus re-reads this weekend
(July 3-4 off). Ratification decision next session.

---

## NFL WEEK 1 BUILD — ARCHITECTURE-INDEPENDENT ONLY

Build window is open. SharpAPI confirmed operational.
Tonight's tasks (if Oracle is synced and time allows):

1. SharpAPI NFL connection test
   sport key: "americanfootball_nfl"
   Confirm data returning for upcoming NFL preseason games
   (preseason starts August — data may already be available)

2. linux_paths.py from first commit
   Environment-aware path resolver. Mandatory. No literal paths.

3. Directory structure
   models/nfl_game/, data/nfl/, logs/nfl/

4. .env variables
   SHARPAPI_KEY (already set from MLB migration)
   NFL_MODEL_VERSION=1.0.0-draft

5. signal_state.json stub
   For SEEKS ecosystem integration. Empty scaffold for now.

NO signal logic. NO probability architecture.
July 25 is the architectural decision point.
Weeks 1-3 are foundation only.

---

## SHARAPI +EV DETECTION — CONFIRMED NOT ON FREE TIER

Confirmed today from SharpAPI docs:
- Free tier: raw odds from 2 books, 60s delay — $0
- Hobby: raw odds from 5 books, real-time — $79/mo
- Pro: +EV detection, 15 books — $229/mo ← threshold

SEDE will NOT be paying $229/month pre-go-live.
Instead: build our own devig function using Pinnacle lines
from SharpAPI free tier. Same underlying math, $0/month, we own it.

Add to NFL Week 1 build list:
Implement `devig_pinnacle(home_odds, away_odds)` function
that strips juice from Pinnacle lines and returns true
implied probabilities. This is the base probability input
for the NFL model's adjustment layer.

Math (for Archie):
```python
def devig_pinnacle(home_american, away_american):
    # Convert American to implied probability
    home_imp = 100/(home_american + 100) if home_american > 0 \
               else abs(home_american)/(abs(home_american) + 100)
    away_imp = 100/(away_american + 100) if away_american > 0 \
               else abs(away_american)/(abs(away_american) + 100)
    # Remove vig (normalize to 100%)
    total = home_imp + away_imp
    home_true = home_imp / total
    away_true = away_imp / total
    return home_true, away_true
```

15 lines. Permanent. $0/month. Ours.

---

## WORLD CUP — TODAY'S RESULTS + TOMORROW'S GAMES

**Today July 2:**
England vs DR Congo 12 PM ET Atlanta
Spain vs Austria 3 PM ET Los Angeles  
Switzerland vs Algeria 11 PM ET Vancouver

Add results to RESOLVED_MARKETS once confirmed.

**Tomorrow July 3 (Rus is off work):**
Portugal vs Croatia, Australia vs Egypt, Argentina vs Cape Verde,
Colombia vs Ghana — all R32 elimination games.

**July 4:**
Canada vs Morocco R16, Houston — WC_GAME calibration
Paraguay vs France R16, Philadelphia — lambda test #5
(model has France at 37.7% — France should be heavy favorite)

---

## GROKBOT V8 — QUICK STATUS

Started June 27 at 10:23 PM CT. Day 5 of 60-day paper clock.
Go-live eligibility: August 26, 2026.
First trade: BTC/USDT +$1.80 net profit (+1.44%).
Balance: $126.80. WR: 1/1 (100%, n=1 — too early to read).
Running clean. No crashes. No false entries on ETH/LINK/AVAX.
This is a J@rv1s + Rus side project. Archie not involved.

---

## OPEN POSITIONS

### SEDE Portfolio
| ID | Market | Dir | Entry | Status |
|----|--------|-----|-------|--------|
| 1 | BTC<$50k Dec31 | NO | 43.5c | BTC ~$65k, thesis intact |
| 3 | GDP >2.5% Q2 | CLOSED | 47c | Closed — GDPNow 1.19% |

### paper_trades.json
| # | Description | Entry | Live | Status |
|---|---|---|---|---|
| 8 | Fed 1x cut YES | 21c | 14c | ⚠️ Documented loss, hold |
| 13 | GDP >2.0% YES | 60c | 56c | ⚠️ Check live price tonight |

Record: 8W-5L-5EE | P&L: +$139.14

---

## VALIDATION TRACKER

| Model | Status | Gate 1 Progress |
|---|---|---|
| JOBS | ✅ GO-LIVE ELIGIBLE | Live test done — 4.2% PERFECT |
| GDP | Active, own track | Trade #13 at risk, Jul 30 |
| CPI | Active | 4.0% consensus, Jul 14 signal |
| Claims | SUSPENDED | 3 consecutive losses |
| MLB_GAME | Track A — Day 6 clean | n=73, Brier 0.2558, Jul 25 |
| WC_GAME | CALIBRATION | Lambda fix confirmed needed |
| NFL | DRAFT pending ratification | Week 1 foundation if ratified |
| Fed | Active, own track | Trade #8 documented loss |
| SEDE Portfolio | ✅ Live | GDP closed, BTC open |

---

## TONIGHT'S PRIORITIES (strict order)

0. **ORACLE SYNC FIRST** — Rus runs sede-pull manually before
   anything else. Archie confirms commit hash matches GitHub.
   Session does not begin until this is confirmed. No exceptions.

1. **Trade #13 live price check** — KXGDP-26JUL30-T2.0 current
   price. Close/hold decision based on current market pricing.
   GDPNow 1.19%, 28 days to resolution. Don't let this drift.

2. **NFL Amendment four items** — add Items 1-4 from above.
   30 minutes of work. Then Rus re-reads over the holiday weekend.

3. **JOBS signal check** — did tonight's 9PM pipeline generate
   JOBS signals based on 57K actual vs consensus? What markets?
   First real JOBS signal since KXPAYROLLS was fixed.

4. **NFL Week 1 foundation** — if time allows after Items 1-3.
   SharpAPI NFL connection test + devig_pinnacle() function.
   linux_paths.py setup. Directory structure. Nothing else.

5. **WC backfill** — today's R32 games once results confirmed.
   Verify Kalshi market strings before scoring.

6. **WC_GAME lambda addendum** — document the four confirmed
   lambda compression cases for the June 26 review addendum:
   England 36.3% (drew), Germany 42.9% (eliminated),
   USA 53.3% (won 2-0), France 37-40% (won 3-0, 3-0).

7. Oracle sede-pull — already done in Step 0. Confirm commits
   are synced after tonight's work.

---

## HOLIDAY WEEKEND NOTE

Rus is off July 3-4. Sessions may be from home PC via J@rv1s.
Archie may or may not have a session Friday July 4.
If no Archie session July 3-4 — that's fine. NFL is in Week 1
foundation mode, nothing is time-critical until July 7 (GDPNow)
and July 14 (CPI).

Jobs report result is logged. Trade #13 decision is tonight.
Everything else can breathe over the holiday weekend.

---

## KEY DATES

| Date | Event |
|---|---|
| Jul 3 | Rus off work — WC R32 games continue |
| Jul 4 | Independence Day — Canada vs Morocco R16 |
| Jul 4 | Paraguay vs France R16 — lambda test #5 |
| Jul 7 | GDPNow next update — Trade #13 watch |
| Jul 14 | CPI release — first signal with 4.0% consensus |
| Jul 25 | MLB Track A/B verdict + NFL architectural decision |
| Jul 30 | GDP advance estimate — Trade #13 resolves |
| Aug 2 | Vegas — Anthony (8rain) demo — 31 days |
| Aug 26 | GrokBot v8 go-live eligibility |
| Sep 3 | NFL season opens — sole hard deadline |

---

## SYSTEM STATUS

| Component | Status |
|---|---|
| Oracle sync protocol | 🔴 NEW STANDING RULE — manual first step |
| JOBS model | ✅ Live test complete — 4.2% perfect call |
| Trade #13 | ⚠️ GDPNow 1.19% — decision tonight |
| NFL Amendment | ⚠️ 4 items needed before ratification |
| SharpAPI free tier | ✅ Operational — devig function to build |
| MLB Track A | Day 6 clean — monitoring Brier to Jul 25 |
| WC_GAME routing | ✅ Fixed and confirmed |
| GrokBot v8 | ✅ Day 5 of 60 — running clean |
| Vegas trip | ✅ Booked — Allegiant Aug 2 |

---

## CONFIRMED API REFERENCE

| API | Base URL | Auth | Notes |
|---|---|---|---|
| Kalshi | external-api.kalshi.com/trade-api/v2 | RSA key | Integrated |
| SharpAPI | sharpapi.io | API key | Free tier, MLB + NFL |
| Polymarket Gamma | gamma-api.polymarket.com | None | 10 req/sec |
| ClubElo | api.clubelo.com | None | CSV, clubs only |
| GDPNow | atlantafed.org/cqer/research/gdpnow | None | Next update Jul 7 |

---

*Prepared by J@rv1s (Web Claude) | Session: Thursday July 2, 2026*
*Step 0 is Oracle sync. Every session. No exceptions. Starting tonight.*
*Jobs report done. 4.2% unemployment — perfect call. NFP weak at 57K.*
*Trade #13 needs a decision tonight before the weekend.*
*Enjoy the 4th, Rus. Talk tomorrow from the home PC.*
*31 days to Vegas. Papa Ralph standard.*
*If it's worth doing it's worth doing right.*
