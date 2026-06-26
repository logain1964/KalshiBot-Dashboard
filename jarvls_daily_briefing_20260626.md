# J@rv1s Daily Briefing
# For Archie — Evening Session Handoff
# Friday June 26, 2026 | Prepared by J@rv1s (Web Claude)

---

## TOP OF FILE — FOUR MAJOR ITEMS TONIGHT

1. MLS/WC identity bug — FIX BEFORE TONIGHT'S 9PM RUN (urgent)
2. SharpAPI migration — MLB + NFL, both sports, one session
3. Seven threads from Rus's gut feeling — full diagnosis
4. WC_GAME June 26 review — formal verdict

Read all sections before starting. The SharpAPI section is the
biggest strategic decision tonight — it changes the data
architecture for both MLB and NFL going forward.

---

## 🚨 URGENT — MLS MODEL TOUCHING WC MARKETS (Fix before 9PM)

This morning's 7AM alert confirmed Thread 5 in the worst way.
Four World Cup Round of 32 games appeared as MLS_GAME signals:

  Norway vs France — France wins — BUY NO (MLS GAME)
  DR Congo vs Uzbekistan — DR Congo wins — BUY NO (MLS GAME)
  Colombia vs Portugal — Portugal wins — BUY NO (MLS GAME)
  Uruguay vs Spain — Spain wins — BUY NO (MLS GAME)

These are WC KNOCKOUT ROUND games. WC Round of 32 starts TODAY.
If this isn't fixed before the 9PM pipeline run, every knockout
game will generate MLS_GAME signals with wrong reliability weight,
wrong calibration data, wrong model attribution.

FIX: Add explicit series prefix filter to MLS model scan in
daily_runner.py. Only scan KXMLSGAME prefixed markets.
One line. Must be in before 9PM tonight.

---

## SHARPAPI MIGRATION — MLB + NFL (Main Build Tonight)

### Why we're switching

OddsAPI free tier: 500 credits/MONTH (~16/day)
Today's single pipeline run burned 281 credits = 56% of monthly budget
Result: 98 dark MLB games, CPI week gap, silent mid-scan failures
The quota exhaustion IS the data quality problem we've been fighting

SharpAPI free tier: 12 req/min = **17,280 requests/DAY**
No monthly cap. No quota wall. No silent failures.
AND includes Pinnacle sharp lines (OddsAPI doesn't on free tier)
AND covers both MLB and NFL with identical API call structure
AND has a first-party Python SDK
AND costs $0/month

| | OddsAPI Free | OddsAPI $59/mo | SharpAPI Free |
|---|---|---|---|
| Daily calls | ~16 | ~3,226 | **17,280** |
| Monthly cap | 500 | 100,000 | **None** |
| Pinnacle lines | ❌ | ✅ | **✅** |
| MLB | ✅ | ✅ | **✅** |
| NFL | ✅ | ✅ | **✅** |
| Cost | $0 | $59/mo | **$0** |

SharpAPI free tier beats OddsAPI $59/month on quota AND adds
Pinnacle sharp lines for free. No argument for staying on OddsAPI.

### What to build tonight

Step 1 — Get SharpAPI key
Sign up at sharpapi.io — no credit card, free tier, instant key.

Step 2 — Install Python SDK
pip install sharpapi

Step 3 — Test MLB connection
```python
from sharpapi import SharpAPI
client = SharpAPI(api_key="YOUR_KEY")
odds = client.odds.list(sport="baseball_mlb", markets=["h2h"])
for event in odds.data:
    print(f"{event.away_team} @ {event.home_team}")
    for book in event.bookmakers:
        for outcome in book.markets[0].outcomes:
            print(f"  {book.title}: {outcome.name} {outcome.price}")
```

Step 4 — Migrate mlb_model.py
Replace OddsAPI calls with SharpAPI calls.
Sport key: "baseball_mlb"
Markets: ["h2h"] for moneyline (same as current)
Team name format may differ from OddsAPI — verify team name
matching against Kalshi ticker format before going live.

Step 5 — Wire NFL model to SharpAPI from day one
NFL build window opens July 1 — 5 days away.
Sport key: "americanfootball_nfl"
Build the NFL model on SharpAPI from first commit.
Never touch OddsAPI for NFL. Clean start.

Step 6 — Add quota logging to OddsAPI calls (optional cleanup)
If any model still uses OddsAPI after tonight, add graceful
degradation: check remaining quota before scan, log WARNING
if insufficient, stop cleanly rather than silently dropping games.
Low priority — if MLB migrates fully, OddsAPI may be unused.

Step 7 — Set SharpAPI key in Oracle .env
SHARPAPI_KEY=your_key_here
Add to both laptop and Oracle .env files.

### Vegas context (37 days)
The quota exhaustion was silently killing MLB signal coverage.
227 games over the next two weeks. With SharpAPI's 17,280
daily calls there's no reason SEDE can't scan every game
on every slate, multiple times per day. The track record
Anthony sees starts being built tonight.

---

## THE SEVEN THREADS — FULL DIAGNOSIS

Rus's gut feeling was right. Three layers of peeling last night
didn't fully satisfy it. Here is J@rv1s's fresh-eyes diagnosis
on all seven threads.

### Thread 1 — OddsAPI Quota (ROOT CAUSE OF MOST PROBLEMS)
This is the missing piece. At 281 credits per run on a 500/month
budget, the pipeline was exhausting quota in 2 runs. Everything
else flows from this:
- June 9 low-signal day (3 signals vs 22 on June 6): quota wall
- CPI week gap June 10-12: quota burned by CPI model, nothing
  left for MLB
- 98 dark games: pipeline stopped mid-scan, no one knew
RESOLUTION: SharpAPI migration above. Problem eliminated.

### Thread 2 — CPI Week Gap (June 10-12)
Working hypothesis: quota exhaustion from Thread 1.
Secondary check: were there pipeline error logs on those dates?
If quota logging goes in tonight, this becomes self-diagnosing
on the next occurrence. Low priority given Thread 1 resolution.

### Thread 3 — Post-Fix Baseline (Patience)
Real clock starts June 23. 65.5% WR on 59 unique games is
unreliable given contamination. The answer is 3 weeks of clean
SharpAPI-fed data. This is a waiting game, not an archaeology
task. Check back July 14 with a real sample.

### Thread 4 — Edge Calculation Reliability (12PM vs 4PM)
The NYY@BOS +45.2c edge on a fast run was a stale odds artifact.
Test tonight: compare average edge for MLB signals generated at
12PM vs 4PM refreshes on the same games. If 12PM average edge is
materially higher (>3c difference), stale odds inflation confirmed.
If confirmed: add time gate — MLB auto-entry only from 4PM+ signals.
SharpAPI's real-time data (sub-89ms latency) reduces this problem
significantly vs OddsAPI's potentially stale feeds.

### Thread 5 — MLS Model Touching WC Markets
CONFIRMED IN THIS MORNING'S ALERT. Fix is tonight's top priority.
See urgent section above.

### Thread 6 — Fast vs Slow Calibration State
Fast run: 1,020 signals, Brier 0.1626
Slow run: 1,378 signals, Brier 0.1734
This morning's alert showed 1,380 signals / Brier 0.1732 —
matching the SLOW run. Good news: the 7AM alert is reading the
slow run's scorer state. Thread 6 may be less critical than feared.
Verify by checking which pipeline step triggers the alert.
If it's already reading slow run state — no fix needed.

### Thread 7 — JOBS Model Scan
Zero JOBS signals despite 13 KXU3 and 24 KXPAYROLLS markets live.
Three possible explanations:
A) No edge found yet (plausible — July 2 release 6 days away,
   markets may be efficiently priced early in the cycle)
B) Series prefix mismatch in run_jobs_model()
   Check: what prefix is the JOBS model scanning for?
   Verify against actual Kalshi market names: kxu3-26jun,
   kxpayrolls-26jun
C) Quota exhaustion (pipeline hit the wall before JOBS scan)
   RESOLUTION: SharpAPI migration eliminates C for sports models.
   JOBS uses Kalshi API directly, not OddsAPI — so quota isn't
   the issue for JOBS. B is the more likely explanation.
   Check the series prefix tonight. July 2 is 6 days away.

---

## WC_GAME JUNE 26 REVIEW — FORMAL VERDICT

Group stage is complete. All results scored. Here is the verdict.

### Diagnostic Results (from last night's script)
1,091 resolved WC_GAME + SOCCER_GAME signals analyzed.

CONFIRMED FIX 1 (deployed commit 3458bf8):
YES signals where model_prob 0-35%: 7.9% WR on 354 signals.
YES signals where model_prob 36-45%: 0.0% WR on 26 signals.
Minimum model_prob ≥ 45% gate eliminates 380 losing signals.

CONFIRMED FIX 2 (deployed commit 3458bf8):
Fading teams below 20c Kalshi: 18.7% WR — terrible.
Fading teams 20-35c Kalshi: 63.9% WR — excellent.
No-fade-below-20c floor eliminates the worst signals.

UNRESOLVED: Theory 1 (draw underweighting) — can't confirm
from current logging. Draw results not tagged in notes column.
Manual spot check on 5-10 known draw games needed.

Theory 3 (edge inversion 15-19c showing 88% WR) — likely
scoring artifact. Don't change threshold based on this data.

### Calibration Status Post-Group-Stage
WC_GAME: 1,074 resolved signals, Brier 0.1817
SOCCER_GAME: 219 resolved signals, Brier 0.1415
Both beating random (0.25). Good calibration despite low WR.

### Knockout Round Games Starting Today
Norway vs France | DR Congo vs Uzbekistan (today)
Colombia vs Portugal | Uruguay vs Spain (today)
These are generating as MLS_GAME signals — fix first (see above).
After fix, they should appear as WC_GAME CALIBRATION ONLY signals.

### Turkey 3, USA 2 (last night's result)
Model had USA at 50.6%. Turkey won 3-2 in stoppage time 98'.
USA had already clinched Group D so result didn't matter for
advancement. Good calibration data — moderate confidence miss.
Add to RESOLVED_MARKETS tonight.

### Paraguay 0, Australia 0
Both advance. Add to RESOLVED_MARKETS tonight.

### FORMAL VERDICT

**WC_GAME: Continue calibration through knockout rounds.
Do NOT approve auto-trading on knockout stage games.**

Reasoning:
- Two confirmed fixes deployed — model will be significantly
  better but needs validation on clean post-fix data first
- Knockout rounds are highest-profile games — worst place to
  discover a miscalibrated model
- Sample of post-fix signals too small for Gate 1 verification
- Dixon-Coles rho adjustment (-0.15 → -0.20) still pending
  as a longer-term fix for draw underweighting

**Champions League September remains the primary deployment
target. Build the post-fix clean sample through August.**

---

## GDPNOW + OPEN POSITIONS — HEALTHY

GDPNow updated to 2.5438% on June 25. Market held firm:

SEDE Portfolio:
| ID | Market | Dir | Entry | Live | Status |
|----|--------|-----|-------|------|--------|
| 1 | BTC<$50k Dec31 | NO | 43.5c | 33.5c | BTC ~$63k, intact |
| 3 | GDP >2.5% Q2 | YES | 47c | 49.5c | ✅ In the green |

paper_trades.json:
| # | Description | Entry | Live | Status |
|---|---|---|---|---|
| 8 | Fed 1x cut YES | 21c | 16c | ⚠️ Documented loss, hold |
| 12 | GDP >2.5% YES | 40c | 45c | ✅ Profitable |
| 13 | GDP >2.0% YES | 60c | 73c | ✅ Strong +13c |

Despite GDPNow at 2.5438%, Kalshi markets not capitulating.
Hold both GDP positions. Next GDPNow update July 1.

---

## TONIGHT'S PRIORITIES (strict order)

1. MLS series prefix fix — one line in daily_runner.py
   KXMLSGAME only. In before 9PM. No exceptions.

2. SharpAPI account setup — sign up at sharpapi.io, get key,
   install Python SDK, test MLB connection (5 minutes)

3. MLB migration to SharpAPI — replace OddsAPI calls in
   mlb_model.py. Verify team name matching vs Kalshi tickers.

4. Thread 7 — JOBS series prefix check — verify run_jobs_model()
   is scanning kxu3-26jun and kxpayrolls-26jun correctly.
   July 2 is 6 days away. This cannot slip.

5. Thread 6 — Fast vs slow calibration state — confirm 7AM
   alert reads slow run scorer state. If confirmed, no fix needed.

6. Thread 4 — MLB 12PM vs 4PM edge distribution — run the
   comparison. If 12PM edges materially higher, add time gate.

7. WC backfill — Turkey 3 USA 2, Paraguay 0 Australia 0,
   plus today's Round of 32 results once confirmed.

8. NFL prep — document SharpAPI as confirmed NFL data source.
   Build window opens July 1. First commit uses SharpAPI.

9. Oracle sede-pull — sync all commits.

10. Carried: threshold auto-apply (own session), Dixon-Coles
    rho adjustment (-0.15 → -0.20) for draw probability.

---

## VALIDATION TRACKER

| Model | Status | Gate 1 Progress |
|---|---|---|
| JOBS | ✅ GO-LIVE ELIGIBLE | n=81, 58.0%, Brier 0.135 |
| GDP | Active, own track | 2 open positions, Jul 30 |
| CPI | Active | Accumulating monthly |
| Claims | SUSPENDED | 3 consecutive losses |
| MLB_GAME YES | Active — SharpAPI migration tonight | Post-fix clock running |
| WC_GAME | CALIBRATION — fixes deployed | CL September target |
| SOCCER_GAME | CALIBRATION ONLY | 219 signals, Brier 0.1415 |
| Fed | Active, own track | Trade #8 documented loss |
| SEDE Portfolio | ✅ Live | $1,000, 2 open, both green |

---

## KEY DATES

| Date | Event |
|---|---|
| Jun 26 TODAY | WC Round of 32 begins — MLS fix urgent |
| Jun 29 | More WC Round of 32 games |
| Jul 1 | NFL build window opens — SharpAPI from day one |
| Jul 1 | GDPNow next update |
| Jul 2 | Jobs report 8:30 AM CT — 6 days |
| Jul 14 | CPI release |
| Jul 30 | Q2 2026 GDP advance estimate |
| Aug 2 | Vegas — Anthony (8rain) demo, 37 days |
| Sep 3 | NFL season opens |
| Sep | Champions League — primary soccer target |

---

## SYSTEM STATUS

| Component | Status |
|---|---|
| MLS/WC identity bug | 🔴 CONFIRMED — fix before 9PM tonight |
| OddsAPI quota | 🔴 ROOT CAUSE — SharpAPI migration tonight |
| SharpAPI MLB | 🔴 NOT YET — migrate tonight |
| SharpAPI NFL | 🔴 Build window Jul 1 — first commit uses SharpAPI |
| WC_GAME Fix 1+2 | ✅ Deployed commit 3458bf8 |
| JOBS scan | ⚠️ Zero signals — verify series prefix tonight |
| Fast/slow calibration | ✅ Likely fine — verify tonight |
| GDP positions | ✅ Both profitable despite GDPNow at 2.5% |
| SEDE Portfolio | ✅ $1,000, 2 open, both green |
| BTC position | ✅ Thesis intact, BTC ~$63k |
| Oracle cron | ✅ Healthy |

---

## CONFIRMED API REFERENCE (updated)

| API | Base URL | Auth | Notes |
|---|---|---|---|
| Kalshi | external-api.kalshi.com/trade-api/v2 | RSA key | Already integrated |
| **SharpAPI** | **sharpapi.io** | **API key (free)** | **MLB + NFL from tonight** |
| Polymarket Gamma | gamma-api.polymarket.com | None | 10 req/sec |
| ClubElo | api.clubelo.com | None | CSV, clubs only |
| eloratings.net | eloratings.net | None | National teams |
| GDPNow | atlantafed.org/cqer/research/gdpnow | None | Next update Jul 1 |
| ~~OddsAPI~~ | ~~the-odds-api.com~~ | ~~Retiring~~ | ~~500/month quota wall~~ |

---

*Prepared by J@rv1s (Web Claude) | Session: Friday June 26, 2026*
*MLS fix before 9PM. SharpAPI tonight. NFL on SharpAPI from day one.*
*Rus's gut was right. The quota wall was the missing piece.*
*37 days to Vegas. Let's build the track record Anthony sees.*
*Papa Ralph standard. If it's worth doing it's worth doing right.*
