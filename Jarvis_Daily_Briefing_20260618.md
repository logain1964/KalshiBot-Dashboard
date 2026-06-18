# J@rv1s Daily Briefing
# For Archie — Evening Session Handoff
# Thursday June 18, 2026 | Prepared by J@rv1s (Web Claude)

---

## TOP OF FILE — ONE PRIMARY DELIVERABLE TONIGHT

`sede_portfolio.json` gets built tonight. That's the session.
Everything else is secondary. Read the FORWARD PROGRESS section
before anything else.

---

## FORWARD PROGRESS — BREAKING THE REACTIVE PATTERN

Two weeks of necessary but reactive work. Bugs fixed, Oracle
restored, direction bug caught, stale signals cleaned up. All
real and necessary. But the subscriber product hasn't moved.

`sede_portfolio.json` has been Priority 1 on eight consecutive
briefings. Tonight it gets built — not started, not scoped, BUILT.

**The rule for tonight:** bugs get fixed in the same session they
surface, then you go back to the portfolio build. The build does
not get bumped.

**Why this matters now:** 8rain (Anthony from Chicago, Rus's friend
who runs 8rain Station — the sports betting analytics platform that
inspired SEDE) has been invited to be SEDE's first signal recipient.
He's the Vegas demo target, he built real predictive infrastructure,
and he'll understand SEDE immediately. Having something to show him
means `sede_portfolio.json` needs to exist. That's the forcing
function that's been missing.

Full session tomorrow morning (J@rv1s + Rus) on the 8rain
relationship, Vegas framing, and SEDE-to-SEEKS roadmap. Archie
should be aware this is happening but doesn't need to prepare
anything for it.

---

## TONIGHT'S PRIORITIES (strict order, no exceptions)

### Priority 1 — MLB post-fix accuracy pull (15 minutes max)
Filter `signals_log.csv` for MLB_GAME signals entered after
June 14 (direction bug fix, commit b9304e4). Resolved signals
only. Calculate:
- Count of resolved post-fix signals
- Win rate on those signals
- Brier score if calculable
Report the number. Even 10-15 resolved signals gives a
directional read. If WR is 55%+, Gate 1 is on track.
If hovering near 50%, flag for further investigation.
15 minutes. Then move to Priority 2 immediately.

### Priority 2 — sede_portfolio.json (primary build, rest of session)
This is BLOCKING for subscriber launch. Build it tonight.

WHAT IT IS:
SEDE's autonomous portfolio — separate from paper_trades.json
(Rus's validation record). This is the subscriber-facing track
record. SEDE trades its own signals with a defined bankroll.

FILE STRUCTURE (minimum viable):
```json
{
  "meta": {
    "created": "2026-06-18",
    "starting_bankroll": 1000.00,
    "current_bankroll": 1000.00,
    "currency": "USD",
    "status": "paper",
    "last_updated": "2026-06-18T00:00:00"
  },
  "performance": {
    "total_trades": 0,
    "open_trades": 0,
    "closed_trades": 0,
    "wins": 0,
    "losses": 0,
    "early_exits": 0,
    "win_rate_pct": null,
    "total_pnl": 0.00,
    "running_bankroll": 1000.00
  },
  "open_positions": [],
  "closed_positions": [],
  "daily_snapshots": []
}
```

POSITION SIZING (from product spec):
- MEDIUM confidence: $15/trade
- HIGH confidence: $25/trade
- HIGH confidence + edge ≥25c: $30/trade
- Max concurrent: 8 positions
- Hard drawdown stop: -$200 (20% of bankroll)

AUTO-ENTRY CRITERIA (what SEDE trades autonomously):
- Edge ≥20c (higher than paper trading threshold of ≥15c)
- Model confidence ≥60%
- SEDE confidence rating HIGH
- Market confirmed active and unresolved (P1 filter)
- Max 8 concurrent positions

WHAT TO BUILD TONIGHT:
1. Create sede_portfolio.json with the structure above
2. Build portfolio_manager_sede.py (separate from
   portfolio_manager.py which tracks Rus's paper trades)
3. Wire auto-entry logic: on each pipeline run, check for
   HIGH confidence signals meeting auto-entry criteria,
   enter if under 8 concurrent positions
4. Wire auto-close logic: on resolution, update position,
   calculate P&L, update running bankroll
5. Update daily_runner.py to call portfolio_manager_sede
   after signal generation

IMPORTANT: sede_portfolio.json is SUBSCRIBER-FACING.
paper_trades.json is RUS'S VALIDATION RECORD.
Never conflate them. Never write to paper_trades.json
from portfolio_manager_sede.py.

### Priority 3 — SOCCER_GAME scoring fix (if time allows)
Fix signal_scorer.py for 3-outcome markets.
"Team X wins NO" should score WIN when Team X does NOT win.
On a 3-outcome market, a NO signal wins if EITHER Draw OR
the other team wins — not just if the other team wins.
Current 34.9% WR is understated because of this bug.
Run corrected WR after fix. Report true number.
This is important but secondary to Priority 2 tonight.

### Priority 4 — Oracle git pull
SSH is fixed. Run from /home/ubuntu/KalshiBot:
git pull origin main --no-edit
Picks up ALL commits including the addendum fixes from last
night (715ce11, 134e669, 69eed86) that Oracle hasn't seen.

---

## THURSDAY 7AM ALERT — VERIFICATION RESULTS

Today's alert was dramatically cleaner. Confirmed working:
✅ Thunder/Spurs Win NBA Championship — GONE
✅ CAR/COL NHL Championship signals — GONE
✅ April payrolls signals — GONE
✅ Next-season NHL tickers — GONE (season-over gate working)
✅ Total flagged edges: 46 (down from 63-75, noise removed)

Still showing (needs fix):
⚠️ Aug 15 stale text still in KEY DATES output
  "Aug 15 -- Go-live target (75 trades / >55% WR...)"
  Archie removed this last night but it's still appearing.
  Either commit didn't push or Oracle hasn't pulled yet.
  Confirm daily_runner.py change is live on both machines.

⚠️ "Fed cuts 0x 2026 -- BUY NO, +19.9c, model 39.3%"
  Confusing signal post-FOMC. Model says 39.3% chance of
  zero cuts, flagging edge on buying NO on that market.
  BUY NO on "cuts 0x" = betting Fed DOES cut. But post-FOMC
  reality is 60.7% probability of hike. This signal may be
  a FedWatch calibration artifact. Investigate and flag.

---

## 8RAIN CONTEXT (read, no action needed)

8rain Station (8rainstation.com) is Anthony's platform.
Rus has invited him to be SEDE's first signal recipient.

What 8rain does: aggregates 100+ sportsbook odds, lets members
upload their own models, one-click EV analysis vs live market
prices. Invite-only, $99/month. Sports betting only —
traditional sportsbooks, not Kalshi.

Why this matters for SEDE:
- Zero competition — 8rain is sportsbooks, SEDE is Kalshi
- Completely complementary — same +EV framework, different markets
- Anthony will understand SEDE immediately (same architecture)
- He's the right first signal recipient — sophisticated,
  honest, and has real skin in the game

Full session tomorrow morning with J@rv1s + Rus on the Vegas
framing, SEDE-to-SEEKS roadmap for the conversation, and what
to show vs what to hold back. Archie doesn't need to prepare
anything for this — it's a strategy session.

The practical implication: `sede_portfolio.json` needs to exist
and have a track record (even a short one) before the Vegas trip
(August 2). That's the real forcing function behind tonight's
Priority 2. Not an abstract subscriber launch — a specific person
is going to see this in 45 days.

---

## OPEN POSITIONS

| #  | Description     | Entry | Current | Status                      |
|----|-----------------|-------|---------|------------------------------|
| 8  | Fed 1x cut YES  | 21c   | ~19c    | ⚠️ Documented loss — hold   |
| 12 | GDP >2.5% YES   | 40c   | ~42c    | ✅ GDPNow 3.04%, 54bps buf  |
| 13 | GDP >2.0% YES   | 60c   | ~68c    | ✅ GDPNow 3.04%, comfortable|

GDPNow: 3.0377% (June 17 update, auto-healed). Next update June 25.
Trade #12 buffer recovered from yesterday's 31bps scare back to
54bps. Both GDP trades healthy.

---

## VALIDATION TRACKER

| Model        | Status                   | Gate 1 Progress              |
|--------------|--------------------------|------------------------------|
| JOBS         | ✅ GO-LIVE ELIGIBLE       | n=73, 58.9%, Brier 0.134     |
| GDP          | Active, own track        | 3 open positions, Jul 30     |
| CPI          | Active                   | Accumulating monthly         |
| Claims       | SUSPENDED, clock running | Holiday test PASSED ✅        |
| MLB_GAME YES | PAUSED, fix deployed     | Post-fix WR = Priority 1     |
| SOCCER_GAME  | CALIBRATION — bug        | True WR unknown until fix    |
| Fed          | Active, own track        | Trade #8 = documented loss   |

---

## KEY DATES

| Date      | Event                                              |
|-----------|----------------------------------------------------|
| Jun 19    | Juneteenth — federal holiday (today)               |
| Jun 25    | GDPNow next update                                 |
| Jun 25-27 | Kalshi opens June unemployment markets             |
| Jun 26    | World Cup group stage ends — SOCCER_GAME checkpoint|
| Jul 1     | NFL build window opens                             |
| Jul 2     | Jobs report 8:30 AM CT                             |
| Jul 30    | Q2 2026 GDP advance estimate                       |
| Aug 2     | Vegas departure — 8rain demo                       |
| Aug 15    | SEDE checkpoint/review                             |
| Sep 3     | NFL season opens                                   |
| Sep       | Champions League returns                           |

---

## SYSTEM STATUS

| Component           | Status                                        |
|---------------------|-----------------------------------------------|
| Oracle SSH          | ✅ Fixed — needs git pull tonight             |
| Oracle sync         | ⚠️ Multiple commits behind — pull tonight    |
| P1 Stale Filter     | ✅ Working — alert clean today               |
| Season-over gates   | ✅ Working — NBA/NHL season complete logs     |
| JOBS month filter   | ✅ Working — quiet until Jun 25-27           |
| FedWatch data       | ✅ Updated post-FOMC                         |
| GDPNow              | ✅ 3.04%, next update Jun 25                 |
| Claims holiday test | ✅ PASSED                                    |
| MLB_GAME YES        | PAUSED — post-fix WR check tonight           |
| SOCCER_GAME scoring | ⚠️ BUG — fix if time after Priority 2       |
| sede_portfolio.json | 🔴 NOT BUILT — PRIMARY DELIVERABLE TONIGHT  |
| Trade #8            | ⚠️ ~19c, documented loss, hold              |
| Aug 15 KEY DATES    | ⚠️ Stale text still appearing — fix tonight |
| Fed cuts 0x signal  | ⚠️ Confusing post-FOMC — investigate        |

---

## CONFIRMED API REFERENCE (standing section)

| API              | Base URL                              | Auth    | Notes            |
|------------------|---------------------------------------|---------|------------------|
| Kalshi           | external-api.kalshi.com/trade-api/v2  | RSA key | Already integrated|
| Polymarket Gamma | gamma-api.polymarket.com              | None    | 10 req/sec       |
| ClubElo          | api.clubelo.com                       | None    | CSV, clubs only  |
| eloratings.net   | eloratings.net                        | None    | National teams   |
| GDPNow           | atlantafed.org/cqer/research/gdpnow   | None    | Next update Jun 25|

---

*Prepared by J@rv1s (Web Claude) | Session: Thursday June 18, 2026*
*One deliverable tonight: sede_portfolio.json gets built.*
*45 days to Vegas. Anthony is watching. Papa Ralph standard.*
*If it's worth doing it's worth doing right.*
