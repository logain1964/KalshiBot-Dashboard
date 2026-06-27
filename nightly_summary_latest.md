# J@rv1s Morning Briefing
# Prepared by Archie | Saturday June 27, 2026 -- Afternoon/Evening
# For: J@rv1s Web Session -- Sunday June 28, 2026
# Priority read: Foundation Document section -- architectural pivot complete

---

## TOP OF FILE -- MAJOR STRATEGIC SESSION TODAY

Today was not a normal session. Read the Foundation section before
anything else. The project direction has changed in a meaningful way
and you need full context before the morning brief.

---

## THE ARCHITECTURAL PIVOT -- SUMMARY

This morning Rus, J@rv1s, and Archie ran a full FORGE/BIAS/PRP protocol
on SEDE's MLB model architecture. The finding: MLB_GAME was operating as
inter-market arbitrage (sportsbook vs Kalshi price comparison), not as
a genuine predictive model.

J@rv1s responded with Foundation Document v1.0.0 (uploaded by Rus).
Archie ran a BIAS protocol on that document, found three key amendments:
  1. MLB verdict: FAIL → PENDING (inter-market arbitrage not inherently
     invalid -- needs clean data before condemnation)
  2. SEEKS node independence: overstated -- non-perfect correlation
     sufficient, not complete independence
  3. Economic models: deserve caveats, not automatic passes

Result: Foundation Document v1.0.1 ratified. Model Integrity Gate
system built and committed. The broader strategic frame: SEDE is the
foundation for SEEKS. Every model must earn its place as a genuine
SEEKS node. Papa Ralph Protocol applies to architecture, not just code.

**What changed:**
- NFL build: PAUSED until MLB parallel track has 2 weeks data
- New model additions: PAUSED
- Subscriber onboarding: PAUSED
- Vegas: reframed -- show 8rain the architecture, not a win rate

**What continues:**
- Economic models (GDP, Jobs, CPI) -- active, paper trading
- WC calibration -- Poisson model is STRONG NODE, accumulate data
- Pipeline health -- maintain everything
- MLB parallel track -- begins immediately (see below)

---

## MODEL INTEGRITY GATE SYSTEM -- WHAT WAS BUILT

New directory: C:\KalshiBot\model_integrity\

Contents committed to repo (commit c724b2b):
  - SEDE_Foundation_Document_v1.0.1.md (508 lines -- governing document)
  - integrity_gdp.md -- ACCEPTABLE NODE with caveat
  - integrity_jobs.md -- ACCEPTABLE NODE with caveat
  - integrity_cpi.md -- ACCEPTABLE NODE with caveat
  - integrity_fed.md -- WEAK NODE (architectural review needed)
  - integrity_btc.md -- WEAK NODE (implied vol comparison needed)
  - integrity_wc_game.md -- STRONG NODE (reference implementation)
  - integrity_soccer_game.md -- STRONG NODE
  - integrity_mlb_game.md -- PENDING (parallel track, July 25 decision)
  - data_source_registry.md -- single-glance dependency map all models
  - amendment_log.md -- tracks all Foundation Document changes
  - quarterly_review_template.md -- September 27 first review

**The four questions every model must answer (Model Integrity Gate):**
1. What is the primary probability source?
2. Would our model generate a different number than consensus, and why?
3. If all market prices disappeared, what would the model output?
4. What would make this model wrong systematically?

These questions are now REQUIRED before any model enters the pipeline,
before any primary data source changes, and at every quarterly review.

---

## MLB PARALLEL TRACK -- WHAT NEEDS TO BE BUILT

The MLB_GAME integrity check specifies a parallel track approach.
Both tracks run simultaneously on clean SharpAPI data for 4 weeks.

**Track A (current -- measure it properly):**
Immediate priority: Edge decay logging.
When mlb_refresh.py runs at 12PM or 4PM, it flags a signal at say 15c.
We currently log nothing about what happens to that Kalshi price by game time.
Need to add: at game start, fetch and log Kalshi price again.
New columns in signals_log.csv:
  - kalshi_price_at_gametime
  - realized_edge_cents (signal edge vs game time edge)
This tells us whether the inter-market arbitrage gap persists.
Priority P1 -- needs to be in before Thursday's games.

**Track B (genuine model -- build it):**
P2: Park factors lookup table (static, one-time build)
P3: FIP lookup to replace ERA in pitcher adjustment
P4: Team Elo ratings system (most complex -- July 7-14)
P5: Track B signal generation running in parallel (July 14-25)

Decision date: July 25, 2026.
Track A success: realized edge >8c average, WR >58% on >15c realized edge
Track B success: P(model_b) diverges from P(sharpapi) ≥20% of games,
                 better Brier score on divergent signals

---

## TODAY'S COMPLETED WORK

### Bugs/fixes (all pre-9PM)
1. JOBS month filter: was filtering by release month (July) not data
   month (June). Fixed in models/jobs_model.py. 13 unemployment markets
   now correctly pass filter. July 2 report in 5 days.

2. Debug script cleanup: 80+ check_*.py, find_*.py, test_*.py, fix_*.py
   moved to tmp/. Root now contains only production files.

### Analysis committed
- thread4_analysis.py: 12PM vs 4PM edge inflation -- inconclusive (n=3
  overlap games), mitigated by SharpAPI migration. Thread 4 CLOSED.
- thread6_analysis.py: fast/slow calibration state -- non-issue.
  Scorer runs before alerts at L1435. Thread 6 CLOSED.
- mlb_period_split.py: calibration split by pipeline period.
  Period 1 (May 27-Jun 9): n=36, 58.3% WR, Brier 0.2534
  Period 2 (bug window Jun 10-22): n=22, 77.3% WR (small sample)
  Periods 3+4: n=0 (mlb_refresh crash silenced signals)
  Finding: insufficient clean data to draw conclusions in any period.
- test_sharp_historical.py: SharpAPI does not support historical odds.
  /events endpoint returns historical schedules but not historical prices.
  Historical backtest not buildable with available tools.

### All 7 threads from Rus's gut feeling: RESOLVED
| Thread | Finding | Status |
|--------|---------|--------|
| 1 OddsAPI quota | ROOT CAUSE of dark games | Fixed -- SharpAPI |
| 2 CPI week gap | Caused by quota exhaustion | Closed |
| 3 Post-fix baseline | Patience, clock starts Jun 23 | Closed |
| 4 12PM vs 4PM edges | Inconclusive, n=3, mitigated | Closed |
| 5 MLS touching WC | Confirmed, fixed Jun 26 | Fixed |
| 6 Fast/slow calibration | Non-issue, scorer runs first | Closed |
| 7 JOBS model scan | Month filter bug, fixed Jun 27 | Fixed |

---

## OPEN POSITIONS -- STATUS (live prices 10AM CST approx)

### SEDE Portfolio
| ID | Market | Dir | Entry | Live | Drift |
|----|--------|-----|-------|------|-------|
| #1 | BTC <50k Jan '27 | NO | 43.5c | ~39.5c | -4c |
| #3 | GDP >2.5% Q2 | YES | 47c | ~43.5c | -3.5c |

GDP >2.5% has been drifting down all week (49.5c Thu → 43.5c Sat).
Next GDPNow update July 1 is the decision point.
Not a crisis but worth watching at the 7PM cron tonight.

### Paper Trades
| # | Market | Dir | Entry | Live | Status |
|---|--------|-----|-------|------|--------|
| #8 | Fed 1x cut | YES | 21c | 15.7c | Documented loss, hold |
| #12 | GDP >2.5% | YES | 40c | 43.5c | Marginally positive |
| #13 | GDP >2.0% | YES | 60c | 64.5c | Positive buffer |

---

## WC RESULTS -- STILL PENDING FOR TODAY

June 27 group stage final round (last group games):
- 5PM CT: Panama vs England | Croatia vs Ghana -- LIVE/COMPLETED?
- 7:30PM CT: Colombia vs Portugal | DR Congo vs Uzbekistan -- not started
- 10PM CT: Jordan vs Argentina | Algeria vs Austria -- not started

Need to backfill all 6 games into signal_scorer.py tonight or tomorrow
morning once results confirmed. Check scores before backfilling.

---

## SYSTEM HEALTH

| Component | Status |
|-----------|--------|
| SharpAPI | Running clean -- 7AM cron confirmed |
| JOBS fix | Deployed -- verify in 9PM run tonight |
| Model integrity gate | Built and committed c724b2b |
| Foundation Document | v1.0.1 ratified |
| Oracle | Healthy -- last confirmed push this morning |
| signals_log.csv | 1,564 scored, Brier 0.1752 |
| MLB parallel track | Track A starts now, Track B builds this week |

---

## KEY DATES

| Date | Event |
|------|-------|
| Jun 28 | WC Round of 32 begins |
| Jun 29 | GDPNow tracking -- watch for market moves |
| Jul 1 | GDPNow update -- GDP position decision point |
| Jul 2 | Jobs report 8:30 AM CT -- JOBS model first live test |
| Jul 4 | Edge decay logging should be in by this date (Track A P1) |
| Jul 14 | CPI release |
| Jul 25 | MLB parallel track decision date |
| Aug 2 | Vegas -- 8rain demo (architecture story, not win rate) |
| Sep 3 | NFL season (build AFTER July 25 MLB decision) |
| Sep 27 | First quarterly architecture review |

---

## MORNING PRIORITIES FOR J@rv1s

1. WC backfill: check scores for all June 27 games, backfill signal_scorer.py
2. Check 9PM cron output: did JOBS signals fire? (13 markets should now pass)
3. Check 9PM cron: GDPNow pipeline still showing 2.5438%?
4. Check BTC position: BTC was at $60,260 this morning -- any movement?
5. Read Foundation Document v1.0.1 fully before any model questions.
   It governs everything from here.

---

## FOR RUS WHEN HE RETURNS

The Model Integrity Gate is live. The Foundation Document is ratified.
The MLB parallel track specification is documented and ready to build.

The next concrete build task is Track A P1: edge decay logging in
mlb_refresh.py. Estimated 1-2 hour session. Can begin whenever you're ready.

The SEEKS vision is now explicitly embedded in the governing document.
Every model decision from here is made with SEEKS in mind.

Papa Ralph Protocol. Foundation first. Let the data decide.

---

*Prepared by Archie | Saturday June 27, 2026 | Afternoon session*
*Commit c724b2b -- Model Integrity Gate system live*
*37 days to Vegas. The story just got better.*
