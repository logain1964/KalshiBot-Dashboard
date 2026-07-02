# J@rv1s Daily Briefing
# For Archie — Evening Session Handoff
# Wednesday July 1, 2026 | Prepared by J@rv1s (Web Claude)

---

## TOP OF FILE — FOUR MAJOR ITEMS TONIGHT

1. GDPNow dropped to 1.2% — GDP trades need immediate review
2. ADP came in at 98K (below 130K consensus) — update JOBS model tonight
3. NFL Amendment ratification — sportsbook-anchor question now answered
4. WC backfill — tonight's results plus USA vs Bosnia

Read all sections. The GDPNow drop is the most urgent item.

---

## 🚨 GDPNOW — DROPPED TO 1.2% TODAY

This is a significant and urgent development.

**Trajectory:**
4.3% (May 21) → 3.8% → 3.0% → 3.3% → 2.8% → 3.0% (Jun 17)
→ 2.5% (Jun 25) → **1.2% (Jul 1)**

The July 1 update dropped 1.3 percentage points — the largest
single-day drop in the tracking window. Driven by:
- Private investment growth nowcast: 8.5% → 6.5%
- Net exports contribution: -0.59pp → -1.62pp

**Impact on open positions:**

SEDE Portfolio Position 3 (GDP >2.5% YES @ 47c):
GDPNow at 1.2% is now 130bps BELOW the 2.5% threshold.
GDPNow average absolute error: 0.77pp.
Even accounting for model error, the probability of Q2 GDP
coming in above 2.5% has dropped dramatically.
This position is now in serious thesis trouble.
Archie needs to check the live Kalshi price on KXGDP-26JUL30-T2.5
tonight and make a close/hold decision.

paper_trades.json Trade #12 (GDP >2.5% YES @ 40c):
Same market, same problem. This was closed earlier at 42c (+$1.24).
Already out — no action needed.

paper_trades.json Trade #13 (GDP >2.0% YES @ 60c, currently 62c):
GDPNow at 1.2% puts this position in the danger zone.
0.8pp gap to the 2.0% threshold. Within GDPNow's ±0.77pp error band.
This position is no longer comfortable. Check live Kalshi price.
If market has dropped significantly from 62c, evaluate whether
to close now or hold to July 30 resolution.

**Tonight's FIRST task:** Check live Kalshi prices on:
- KXGDP-26JUL30-T2.5 (SEDE Position 3)
- KXGDP-26JUL30-T2.0 (paper Trade #13)
Make close/hold decisions based on current market pricing.
Next GDPNow update: July 7.

---

## 📊 ADP — 98K, BELOW CONSENSUS (Update JOBS Model Tonight)

ADP June private payrolls: **98,000** (below 130K SEDE consensus,
below 110K Dow Jones estimate, below 118K Bloomberg estimate)
Down from 122K in May. Nearly half from education/health services.
Leisure/hospitality delivered sixth straight month of weak hiring.
Pay gains: 4.4% job-stayers, 6.6% job-changers.

**JOBS model update needed:**
NFP_CONSENSUS should be lowered from 130K to approximately 105-110K.

Why not 98K? ADP and BLS numbers correlate poorly — ADP recently
has been running below BLS actuals. The Wall Street consensus for
Thursday's official BLS nonfarm payrolls is 115K (Dow Jones) with
unemployment steady at 4.3%. ADP's 98K suggests downside risk
but BLS has consistently printed above ADP in recent months.

**Recommended update:** Lower NFP_CONSENSUS from 130K → 110K
for Thursday's JOBS model run. This puts SEDE in line with
the updated Wall Street consensus after ADP's weak print.

Update NFP_CONSENSUS in jobs_model.py TONIGHT before the
2AM cron run. Jobs report is tomorrow 8:30 AM CT.

**The JOBS model live test is TOMORROW.** This is the first
real signal opportunity since KXPAYROLLS was fixed. The model
needs the correct consensus going in.

---

## 🏈 NFL AMENDMENT — SPORTSBOOK-ANCHOR QUESTION ANSWERED

Today's strategic discussion with Rus resolved the single most
important pre-ratification question from the amendment:

**Does NFL signal generation anchor to sportsbook lines?**

**THE ANSWER (for the amendment):**

"NFL signal generation uses devigged sportsbook consensus as
the base probability estimate. Signals are only generated when
SEDE's proprietary adjustment layer causes the adjusted probability
to diverge from Kalshi's implied probability by ≥20c.
Inter-market sportsbook-to-Kalshi arbitrage is explicitly excluded
as a signal source."

**What this means architecturally:**

Architecture: Option C — Hybrid with Strong Node upgrade path

Primary input: Devigged SharpAPI/Pinnacle sharp-book consensus
as base probability (not independent model output)

Proprietary adjustment layer (REQUIRED, not optional):
- Tiered QB injury status (5-tier system, already designed)
- Weather adjustment (Anthony's API already wired)
- Rest advantage/disadvantage (short week vs bye week)
- Home field calibrated to actual Kalshi market efficiency
- Divisional familiarity factor (BIAS 11 from J@rv1s audit)

Signal generation gate (the key guard against MLB_GAME failure):
Only generate signal when PROPRIETARY ADJUSTMENTS cause the
≥20c divergence from Kalshi — not the base sportsbook line.
If Kalshi and sportsbooks are already 20c apart before our
adjustments, that's arbitrage detection, not model edge.
Do NOT generate a signal from base-line divergence alone.

Upgrade path: After regular season Weeks 1-4, evaluate whether
replacing sportsbook base with genuine independent Elo/EPA model
improves calibration. If yes, migrate. If no, hybrid stays.

**Other amendment positions from J@rv1s second pass:**
- Strong Node target: CONFIRMED — sharper reasoning documented
- Timeline: 3 weeks for signal logic build — add Week 5 milestone
  (working signal + one-week backtest confirmed by Aug 4)
- SharpAPI escalation thresholds: REMOVE specific numbers (6c/20%),
  replace with evidence-based thresholds after Week 1 research
- context_confidence: Rebuild from scratch + July 25 MLB checkpoint
- Injury source: Week 1 test all options, Week 2 commit based on data
- BIAS gaps: Three additions — BIAS 11 (divisional familiarity),
  BIAS 12 (injury report gamesmanship), BIAS 13 (primetime selection bias)
- Signal Contract: BUILD NFL TO CONTRACT FROM DAY ONE (disagrees
  with Archie's deferral — see rationale in morning briefing)

**STATUS: Drafts in DRAFT status. Not ratified. Not pushed to git.**
Rus re-reads tonight after this briefing. Ratification decision
is Rus's call alone. Do not push to git until Rus ratifies.

If ratified: push both documents to git and update amendment_log.md
in C:\KalshiBot\model_integrity\

NFL Week 1 build: architecture-independent tasks only.
No signal logic, no probability architecture until July 25.
SharpAPI NFL connection test, linux_paths.py setup, directory
structure, environment variables. Clean foundation only.

---

## ⚽ WORLD CUP — LAST NIGHT'S CONFIRMED RESULTS

France 3-0 Sweden — Mbappé brace. France advances to R16.
Mexico 2-0 Ecuador — Mexico through. Azteca celebration.
Norway 2-1 Ivory Coast — Haaland 5th goal. Norway advances.

WC_GAME calibration:
France 3-0 Sweden — model had France BUY NO (fading France).
France won comfortably. Lambda compression confirmed AGAIN.
This is now 4+ consecutive games where the model faded a
strong favorite who then won comfortably.

Mexico 2-0 Ecuador — model had Mexico BUY NO signals.
Mexico won. Same pattern.

**Tonight: USA vs Bosnia and Herzegovina 8 PM ET**
Levi's Stadium, Santa Clara. USA as Group D winners.
Model: USA at 53.3% win probability.
Lambda compression test #3 — if USA wins by multiple goals,
53.3% is significantly understating actual probability.
Watch the result. Add to RESOLVED_MARKETS tomorrow AM.

**WC Backfill needed tonight:**
- France 3-0 Sweden (Jun 30)
- Mexico 2-0 Ecuador (Jun 30)
- Norway 2-1 Ivory Coast (Jun 30)
Verify Kalshi market strings before scoring.

**Tomorrow's R32 games:**
England vs DR Congo 12 PM ET Atlanta
Belgium vs Senegal 4 PM ET Seattle

---

## OPEN POSITIONS — GDP UNDER PRESSURE

### SEDE Portfolio
| ID | Market | Dir | Entry | Status |
|----|--------|-----|-------|--------|
| 1 | BTC<$50k Dec31 | NO | 43.5c | BTC ~$65k, thesis intact |
| 3 | GDP >2.5% Q2 | YES | 47c | 🚨 GDPNow 1.2% — check tonight |

### paper_trades.json
| # | Description | Entry | Live | Status |
|---|---|---|---|---|
| 8 | Fed 1x cut YES | 21c | 15c | ⚠️ Documented loss, hold |
| 13 | GDP >2.0% YES | 60c | 62c | 🚨 GDPNow 1.2% — check tonight |

Record: 8W-5L-5EE | P&L: +$139.14
(Corrected from memory: actual record includes early exits)

---

## VALIDATION TRACKER

| Model | Status | Gate 1 Progress |
|---|---|---|
| JOBS | ✅ GO-LIVE ELIGIBLE | n=81, 58.0% — LIVE TEST TOMORROW |
| GDP | Active, own track | 🚨 GDPNow 1.2% — thesis at risk |
| CPI | Active | 4.0% consensus, Jul 14 signal |
| Claims | SUSPENDED | 3 consecutive losses |
| MLB_GAME | Track A — Day 4 clean | n=73, Brier 0.2558, Jul 25 |
| WC_GAME | CALIBRATION | Fixes deployed, CL Sep target |
| Fed | Active, own track | Trade #8 documented loss |
| SEDE Portfolio | ✅ Live | GDP position at risk |
| NFL | DRAFT PENDING RATIFICATION | Week 1 opens today |

---

## TONIGHT'S PRIORITIES (strict order)

1. GDP live price check — what is KXGDP-26JUL30-T2.5 trading
   at right now? What is KXGDP-26JUL30-T2.0 trading at?
   Make close/hold decisions based on current prices.
   GDPNow at 1.2% with July 30 resolution = serious thesis risk.

2. JOBS model update — lower NFP_CONSENSUS from 130K to 110K
   in jobs_model.py BEFORE tonight's 2AM cron run.
   Jobs report is tomorrow 8:30 AM CT. Must be correct going in.

3. NFL Amendment ratification — Rus re-reads drafts after this
   briefing. If ratified: push to git, update amendment_log.md.
   If not: hold in DRAFT, note specific questions for next session.

4. If ratified: NFL Week 1 build begins.
   Architecture-independent only:
   - SharpAPI NFL connection test ("americanfootball_nfl")
   - linux_paths.py setup from first commit
   - Directory structure: models/, data/, logs/
   - .env variables for SharpAPI key
   - signal_state.json stub for SEEKS ecosystem integration
   NO signal logic. NO probability architecture. July 25 is
   the architectural decision point.

5. WC backfill — France 3-0 Sweden, Mexico 2-0 Ecuador,
   Norway 2-1 Ivory Coast. Verify Kalshi strings before scoring.

6. USA vs Bosnia result — tonight 8 PM ET. Note for tomorrow AM.
   Lambda compression test #3. Add to RESOLVED_MARKETS tomorrow.

7. GDPNow pipeline update — update LAST_KNOWN_GOOD_GDPNOW to
   1.2000 in gdpnow_updater.py. Log to gdpnow_history.txt.

8. Oracle sede-pull — sync all commits.

9. data quality flag — Trades #23 and #24 (CPI June 10) show
   close_note "AUTO_STOP_LOSS: loss" but PnL shows positive
   values. Low priority but eyeball during a quiet moment.

---

## KEY DATES

| Date | Event |
|---|---|
| **Jul 2 TOMORROW** | **Jobs report 8:30 AM CT — JOBS model live test** |
| Jul 2 | England vs DR Congo 12 PM ET |
| Jul 2 | Belgium vs Senegal 4 PM ET |
| Jul 3 | Spain vs Austria, Portugal vs Croatia, Australia vs Egypt |
| Jul 3 | Argentina vs Cape Verde, Colombia vs Ghana |
| Jul 4 | Canada vs Morocco (Houston) R16 |
| Jul 7 | GDPNow next update |
| Jul 14 | CPI release |
| Jul 25 | MLB Track A/B verdict + NFL architectural decision |
| Jul 30 | GDP advance estimate — Trade #13 resolves |
| **Aug 2** | **Vegas — Anthony (8rain) demo — 32 days** |
| Sep 3 | NFL season opens — sole hard deadline |

---

## SYSTEM STATUS

| Component | Status |
|---|---|
| GDPNow | 🚨 1.2% — GDP positions at risk |
| ADP | ⚠️ 98K — JOBS consensus needs update tonight |
| JOBS report | 🔴 TOMORROW 8:30 AM CT — update consensus first |
| NFL Amendment | ⚠️ DRAFT — pending Rus ratification tonight |
| NFL build | Ready — Week 1 opens today if ratified |
| WC routing | ✅ Fixed — confirmed two consecutive runs |
| MLB Track A | Day 4 clean — monitoring Brier |
| SEDE Portfolio | GDP position at risk — check live price |
| Oracle SSH | ✅ Fixed and documented |
| Vegas trip | ✅ Booked — Allegiant Aug 2 |

---

## CONFIRMED API REFERENCE

| API | Base URL | Auth | Notes |
|---|---|---|---|
| Kalshi | external-api.kalshi.com/trade-api/v2 | RSA key | Integrated |
| SharpAPI | sharpapi.io | API key | MLB + NFL primary |
| Polymarket Gamma | gamma-api.polymarket.com | None | 10 req/sec |
| ClubElo | api.clubelo.com | None | CSV, clubs only |
| GDPNow | atlantafed.org/cqer/research/gdpnow | None | Next update Jul 7 |

---

*Prepared by J@rv1s (Web Claude) | Session: Wednesday July 1, 2026*
*GDPNow at 1.2% — biggest single-day drop of the tracking window.*
*ADP 98K — update JOBS consensus to 110K before 2AM cron.*
*Jobs report tomorrow. NFL build starts today if ratified.*
*USA vs Bosnia tonight. 32 days to Vegas.*
*Papa Ralph standard. If it's worth doing it's worth doing right.*
