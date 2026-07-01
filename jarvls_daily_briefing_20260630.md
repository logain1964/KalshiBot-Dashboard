# J@rv1s Daily Briefing
# For Archie — Evening Session Handoff
# Tuesday June 30, 2026 | Prepared by J@rv1s (Web Claude)

---

## TOP OF FILE — FOUR ITEMS TONIGHT

1. WC backfill — today's three R32 games plus NED/MAR from last night
2. NFL build window opens TOMORROW — pull design doc tonight
3. ADP tomorrow morning — JOBS consensus check before Jul 2 report
4. MLB Track A monitoring — edge values still thin, Brier watch

No major fires tonight. Clean session ahead.

---

## MLS/WC IDENTITY BUG — CONFIRMED FIXED IN PRODUCTION ✅

This morning's 7AM alert confirmed both layers of the fix are working.
WC R32 games now correctly showing under [WORLD CUP GAME] not [MLS GAME]:

  United Sta vs Bosnia — WORLD CUP GAME ✅
  France vs Sweden — WORLD CUP GAME ✅
  Portugal vs Croatia — WORLD CUP GAME ✅
  Mexico vs Ecuador — WORLD CUP GAME ✅

USA vs Bosnia tonight (8 PM ET, Levi's Stadium Santa Clara) will route
correctly. No further action needed on this bug. Closed.

---

## WORLD CUP — FULL R32 RESULTS + BACKFILL NEEDED

### Confirmed results needing backfill tonight:

| Date | Match | Result | Notes |
|------|-------|--------|-------|
| Jun 28 | Canada 1-0 South Africa | ✅ Done | Eustaquio 90+2 |
| Jun 29 | Brazil 2-1 Japan | ✅ Done | Martinelli 90+5 |
| Jun 29 | Germany 1-1 Paraguay (pens) | ✅ Done | PAR wins 4-3 PKs |
| Jun 29 | Netherlands 1-1 Morocco | ⏳ BACKFILL TONIGHT | MAR wins 3-2 PKs |
| Jun 30 | Norway 2-1 Ivory Coast | ⏳ BACKFILL TONIGHT | Haaland 5th goal |
| Jun 30 | France vs Sweden | ⏳ BACKFILL TONIGHT | 10PM ET kickoff |
| Jun 30 | Mexico vs Ecuador | ⏳ BACKFILL TONIGHT | 9PM CT kickoff |

### Tonight's results to watch and backfill:
France vs Sweden — 5PM ET MetLife Stadium NJ (10PM UK)
Mexico vs Ecuador — 9PM ET Estadio Azteca Mexico City

France are massive favorites — public bettors have 95% of handle
on France at -340 odds, 98% of total money at DraftKings on France
to win on the three-way moneyline. WC_GAME calibration watch —
does the post-fix model correctly identify France as a strong
favorite or is it still generating BUY NO fade signals on them?

Norway 2-1 Ivory Coast is confirmed — Haaland scored his 5th
goal of the tournament on a Patrick Berg cross in the late stages.
Norway advances to face Brazil in the Round of 16 on July 5.

### WC Calibration Notes for June 26 Review Addendum

Lambda compression confirmed on TWO massive upsets this week:
- Germany -300 favorite → eliminated by Paraguay on penalties
  (model had Germany at 42.9% — correctly low? No, WAY too low for -300)
- Netherlands → eliminated by Morocco on penalties
  (evenly matched result, 1-1 through 120 mins — this one is defensible)

USA vs Bosnia tonight at 8 PM ET is the third lambda test.
This morning's alert had USA at 53.3% model probability.
USA should be a clear favorite as Group D winners vs a third-place team.
Watch the result — if USA wins comfortably, that's lambda compression
confirmed again on a moderate-confidence signal.

Add all results to RESOLVED_MARKETS tonight.
Verify Kalshi market strings before scoring.

---

## NFL BUILD WINDOW — OPENS TOMORROW JULY 1

This is the biggest build item of the project after SEDE itself.
$3.8B Kalshi volume. September 3 kickoff. 64 days away.

Tonight's prep task: pull NFL model design doc from session archive.
The architecture is FORGE'd. SharpAPI confirmed as data source.
Linux paths from first commit (mandatory — no literal path bugs).

KEY DECISIONS ALREADY LOCKED:
- Data source: SharpAPI ("americanfootball_nfl" sport key)
- Target markets: KXNFLGAME, KXNFLSPREAD, KXNFLSUPERBOWL
- Weather API: already wired (Anthony's contribution)
- Architecture: signal_engine → signal_validator → portfolio_manager
- linux_paths.py: environment-aware resolver from commit one

FIRST COMMIT TOMORROW — start clean, start right.

---

## JOBS — ADP TOMORROW MORNING (JULY 1)

ADP employment change drops Wednesday July 1 at 8:15 AM ET.
Current SEDE consensus: 130K NFP / 4.2% unemployment.
May actual came in at 172K — strong print.

If ADP diverges significantly from 130K (above 160K or below 100K)
update NFP_CONSENSUS in jobs_model.py before Thursday's report.
ADP is the best real-time signal we have before the official BLS
number drops at 8:30 AM CT Thursday July 2.

JOBS model live test is Thursday. Three days of build since
KXPAYROLLS was fixed. First real signal opportunity since June.

---

## MLB TRACK A — MONITORING

Day 3 of clean SharpAPI data post-fix. Current status:
n=73, 53.4% WR, Brier 0.2558 (above 0.25 random baseline)

Today's signals from morning's alert: zero MLB flagged.
Edge values from recent signals: 6-14c range (mostly below 20c threshold)
SharpAPI confirmed operational — 12PM + 4PM refreshes both firing.

The model is generating signals but they're thin.
Track A clock runs to July 25 decision date.
No action needed — just monitoring.

One diagnostic still outstanding from last week:
Thread 4 (12PM vs 4PM edge distribution) — if time allows
run the comparison. Do 12PM signals show materially higher
average edge than 4PM? If yes, stale odds artifact confirmed,
time gate on MLB auto-entry recommended.

---

## GDPNOW — UPDATE TOMORROW JULY 1

Last reading: 2.5438% (June 25). Next update tomorrow July 1.
Trade #13 (GDP >2.0% YES @ 60c, currently 64-66c) has 54bps
buffer above 2.0% threshold. Virtually certain to hold.
Watch as a formality — no action expected.

---

## OPEN POSITIONS

### SEDE Portfolio
| ID | Market | Dir | Entry | Status |
|----|--------|-----|-------|--------|
| 1 | BTC<$50k Dec31 | NO | 43.5c | BTC ~$65k, thesis intact |
| 3 | GDP >2.5% Q2 | YES | 47c | ✅ In the green |

### paper_trades.json
| # | Description | Entry | Live | Status |
|---|---|---|---|---|
| 8 | Fed 1x cut YES | 21c | 15c | ⚠️ Documented loss, hold |
| 13 | GDP >2.0% YES | 60c | 64c | ✅ Comfortable, Jul 30 |

Record: 7-5 | P&L: +$139.14 | 3 open slots

---

## VALIDATION TRACKER

| Model | Status | Gate 1 Progress |
|---|---|---|
| JOBS | ✅ GO-LIVE ELIGIBLE | n=81, 58.0% — Jul 2 live test |
| GDP | Active, own track | Trade #13 near resolution |
| CPI | Active | 4.0% consensus, Jul 14 signal |
| Claims | SUSPENDED | 3 consecutive losses |
| MLB_GAME | Track A — Day 3 clean | n=73, Brier 0.2558, Jul 25 |
| WC_GAME | CALIBRATION | R32 active, fixes deployed |
| Fed | Active, own track | Trade #8 documented loss |
| SEDE Portfolio | ✅ Live | $1,000, 2 open, green |

---

## TONIGHT'S PRIORITIES (strict order)

1. NED/MAR backfill — Morocco 2-1 Netherlands on penalties (3-2 PKs)
   Gakpo 72', Diop 90+1', penalties 3-2 Morocco.
   Add to RESOLVED_MARKETS. Verify Kalshi market string first.

2. Norway 2-1 Ivory Coast backfill
   Haaland 5th goal of tournament, late winner.
   Add to RESOLVED_MARKETS. Verify Kalshi market string.

3. France vs Sweden — watch and backfill once result confirmed
   France massive favorite. WC_GAME calibration data.

4. Mexico vs Ecuador — watch and backfill once result confirmed

5. NFL design doc — pull from session archive tonight.
   First commit tomorrow. Read it before starting.

6. ADP check — Wednesday morning before Archie's session.
   Flag if diverges significantly from 130K.

7. USA vs Bosnia — 8PM ET tonight. Watch for WC_GAME signal routing.
   Should appear as [WORLD CUP GAME] in tomorrow's 7AM alert.
   Note result for calibration data (lambda compression test 3).

8. Oracle sede-pull — sync all commits.

9. Carried: Thread 4 MLB edge distribution, threshold auto-apply
   (own session), Dixon-Coles rho adjustment.

---

## KEY DATES

| Date | Event |
|---|---|
| Jun 30 TONIGHT | France vs Sweden, Mexico vs Ecuador R32 |
| Jun 30 TONIGHT | USA vs Bosnia 8PM ET — first USA knockout game |
| **Jul 1 TOMORROW** | **NFL build window opens — first commit** |
| Jul 1 | GDPNow next update |
| Jul 1 | ADP employment change 8:15 AM ET |
| **Jul 2 THURSDAY** | **Jobs report 8:30 AM CT — JOBS model live test** |
| Jul 4 | Canada vs Morocco R16 (Houston) |
| Jul 4 | Paraguay vs France/Sweden R16 (Philadelphia) |
| Jul 5 | Brazil vs Norway R16 |
| Jul 14 | CPI release — first signal with 4.0% consensus |
| Jul 25 | MLB Track A decision |
| **Aug 2** | **Vegas — Anthony (8rain) demo — 33 days** |
| Sep 3 | NFL season opens |

---

## SYSTEM STATUS

| Component | Status |
|---|---|
| MLS/WC identity bug | ✅ FIXED — confirmed in production |
| SharpAPI MLB | ✅ Operational — 12PM + 4PM firing |
| SharpAPI NFL | Ready — first commit tomorrow |
| JOBS KXPAYROLLS | ✅ Fixed — Jul 2 live test |
| WC_GAME fixes | ✅ Deployed — calibration only |
| MLB Track A | Day 3 clean — monitoring Brier |
| GDP trades | ✅ Both healthy |
| SEDE Portfolio | ✅ $1,000, 2 open, green |
| Oracle SSH | ✅ Fixed — sede_production.key documented |
| Oracle cron | ✅ Healthy |
| Vegas trip | ✅ Booked — Allegiant Aug 2 |

---

## CONFIRMED API REFERENCE

| API | Base URL | Auth | Notes |
|---|---|---|---|
| Kalshi | external-api.kalshi.com/trade-api/v2 | RSA key | Already integrated |
| SharpAPI | sharpapi.io | API key | MLB + NFL primary |
| Polymarket Gamma | gamma-api.polymarket.com | None | 10 req/sec |
| ClubElo | api.clubelo.com | None | CSV, clubs only |
| eloratings.net | eloratings.net | None | National teams |
| GDPNow | atlantafed.org/cqer/research/gdpnow | None | Next update Jul 1 |

---

*Prepared by J@rv1s (Web Claude) | Session: Tuesday June 30, 2026*
*MLS fix confirmed. NFL build starts tomorrow. JOBS report Thursday.*
*USA vs Bosnia tonight. France vs Sweden tonight. Big night for calibration.*
*33 days to Vegas. Papa Ralph standard. If it's worth doing it's worth doing right.*
