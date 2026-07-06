# J@rv1s Daily Briefing
# For Archie — Evening Session Handoff
# Monday July 6, 2026 | Prepared by J@rv1s (Web Claude)

---

## SESSION OPENER — MANDATORY FIRST STEP (EVERY SESSION)

Before ANYTHING else:
1. Rus manually runs `sede-pull` from Oracle terminal
2. Archie confirms commit hash matches GitHub
3. Session begins only after Oracle sync confirmed
Archie does NOT diagnose Oracle connectivity. Always manual. Always first.

---

## TOP OF FILE — PRIMARY BUILD TONIGHT: LAMBDA FIX

The WC_GAME lambda compression problem is confirmed systematic
across five consecutive games. Tonight we fix it. Signal Contract
carries to tomorrow. Everything else is secondary.

Five confirmed lambda compression cases:
1. England 36.3% → drew 0-0 (underestimate)
2. Germany 42.9% → eliminated by Paraguay PKs (defensible)
3. USA 53.3% → won 2-0 (clear underestimate)
4. France 37-40% → won 3-0 multiple times (systematic)
5. France 37.7% → beat Paraguay 1-0 (confirmed again)

This is not variance. This is a confirmed systematic flaw in
the model's core probability generation. Every signal SEDE
generates right now is slightly wrong because of it.
Champions League is the deployment target. Fix it now,
not after we've built on a miscalibrated foundation.

---

## TWO-LINE DEFENSE — BOTH GO IN TONIGHT

### Line 1 — Fix the root cause (lambda conversion function)

The Elo-to-lambda conversion is too flat. Large quality gaps
between teams (300+ FIFA Elo points) are being translated into
expected goals differentials that are too small (~0.3-0.5 when
they should be ~0.8-1.2). This compresses win probability
toward 50/50 when reality is 70/30 or 75/25.

STEP 1 — Diagnostic first (30 minutes):
Pull the actual λ values the model assigned to France, England,
and USA vs their opponents from the signals_log.csv or model
internals. What expected goals did it give each team?

If France vs Paraguay shows France λ=1.4, Paraguay λ=1.1 —
curve is too flat. France of that quality should be ~λ=1.8-2.0.
The gap between those numbers tells us exactly how much
steepening is needed.

STEP 2 — Fix the conversion function:
For quality gaps above 150 Elo points, apply a non-linear
multiplier that widens the expected goals differential.
Approximately 10-15 lines of code in the lambda calculation.

STEP 3 — Validate before deploying:
Run corrected lambda against full 1,600+ signal history.
Check two things specifically:
  A) Does corrected model give France 65%+, England 60%+,
     USA 65%+ win probability? (These are confirmed underestimates)
  B) Does it still give Morocco reasonable probability vs
     Netherlands? (That was a genuine close game — 1-1 through
     120 minutes. Don't overcorrect so that close games break.)

If validation passes → deploy.
If it overcorrects on genuinely close games → tune it.

### Line 2 — Safety net gate (one line of code)

Regardless of how well the lambda fix works, add a minimum
win probability gate:

Do NOT generate a signal on a team Kalshi prices above 55c
unless the model gives them at least 60% win probability.

This catches any residual cases where the lambda fix doesn't
fully compensate. One is the cure, the other is the bandage
while the cure takes hold. They are not mutually exclusive.

Location: signal generation logic in soccer_game_model.py
Condition: if kalshi_price > 55 and model_prob < 60: skip signal

IMPORTANT: This gate should log clearly when it fires so we
can track how often it's catching residual lambda issues.
Format: "[GATE] Skipped [TEAM] signal — Kalshi 58c but model
only 54%. Lambda correction needed."

This logging tells us whether the lambda fix is working
(gate fires less often over time) or whether we still have
a residual problem (gate fires constantly).

---

## MORNING ALERT DISCREPANCY — VERIFY FIRST

7AM alert: 2 open trades (Trade #8 and #13), $49.59 at risk.
7:45 AM alert: 0 open trades, $0 capital at risk.

This is a discrepancy that needs explanation before anything else.

Two possible explanations:
A) The 7:45 AM alert shows sede_portfolio.json (which legitimately
   has 0 open positions — Position 3 was closed July 1, BTC is
   the only remaining open but may not show in this view).
   If this is the case, everything is fine.

B) Something auto-closed Trade #8 and Trade #13 overnight.
   If this is the case, this is Priority 0 above all else.

CHECK TONIGHT FIRST:
Open paper_trades.json directly. Are Trade #8 and Trade #13
still in the open_positions array? 30 seconds. Answers everything.

If both are open → Explanation A, proceed to lambda fix.
If either is closed → Explanation B, investigate immediately.

---

## TRADE #13 — PRICE WARNING FLAGGED, HOLD CONFIRMED

7AM alert flagged: Trade #13 GDP >2.0% YES moved -17c (60c → 43c).
GDPNow at 1.1889%. Next update tomorrow July 7.

HOLD per calibration-value reasoning locked last session:
paper_trades.json exists to answer whether the model's stated
confidence matches reality. Only July 30 resolution answers that.
Kalshi's 43c price is other traders' opinion, not ground truth.
Cutting early censors exactly the data calibration needs most.

This is NOT a sede_portfolio.json position (that was closed
July 1 correctly). This is paper_trades.json only.

GDPNow July 7 update tomorrow — flag if it drops below 1.0%.

---

## WORLD CUP — TONIGHT + UPCOMING

### USA vs Belgium — Tonight 8 PM ET Seattle
Biggest USA game of the tournament. Balogun suspended.
Belgium is a resilient team — came back from 2 down vs Senegal.

WC_GAME calibration watch: What probability does the model
give USA tonight? After the lambda fix is in, this result
will be the first post-fix calibration data point.
Note the pre-fix model probability AND the post-fix model
probability if the fix goes in before tonight's 9PM pipeline.

### Quarterfinals This Week
Jul 9 — Morocco vs France (Boston)
Jul 10 — Spain/Portugal vs USA/Belgium (Los Angeles)
Jul 11 — Norway vs England (Miami)

France vs Morocco signal already appearing in 7AM alert:
BUY NO, model 55.6%, edge +17.1c. France should be favored.
55.6% is more defensible than 37.7% from earlier — either
the fix is already partially working or this is a tighter
matchup. Watch it as calibration data post-fix.

### France wins WC — MEDIUM confidence, BUY NO, +26c, model 92%
Second consecutive MEDIUM confidence on this signal.
Model assigns France only 8% chance of winning the tournament
while Kalshi has them at 66c. Calibration watch as tournament
progresses. CALIBRATION ONLY — no paper trade.

### WC Backfill Needed Tonight
Morocco 3-0 Canada (Jul 4)
France 1-0 Paraguay (Jul 4) — lambda test #5 confirmed
Brazil 1-2 Norway (Jul 5) — major upset
England 3-2 Mexico (Jul 6)
USA vs Belgium (Jul 6) — add tomorrow once result confirmed
Verify all Kalshi market strings before scoring.

---

## OPEN ITEMS — CARRYING FORWARD (MUST NOT SLIP FURTHER)

These have been flagged multiple sessions. Tonight each one
gets either fixed or explicitly decided not to fix with a reason.
No more silent carries.

1. June 30 session archive — STILL MISSING (flagged 4 sessions)
   Create it tonight. 20 minutes. Done.

2. UNEMP_CONSENSUS — 4.2% vs Dow Jones 4.3%
   Rus's call: keep 4.2% (model was right) or align to 4.3%?
   Archie asks Rus directly at session start. One decision. Done.

3. Trades #23/#24 close_note vs PnL mismatch
   15 minutes. Fix the close_note text. Done.

4. jobs_model.py "Next release" showing June 5 (past date)
   Fix the date calculation. July 14 CPI is 8 days away.

5. Oracle cron code-staleness structural question
   Should the cron git pull code before each run?
   Recommendation: yes — git fetch + checkout Python files only.
   Leaving this unresolved means every multi-day session gap
   risks Oracle running stale code with no warning. Decide tonight.

---

## OPEN POSITIONS

### SEDE Portfolio
| ID | Market | Dir | Entry | Status |
|----|--------|-----|-------|--------|
| 1 | BTC<$50k Dec31 | NO | 43.5c | BTC ~$65k, thesis intact |

Bankroll: $994.17

### paper_trades.json
| # | Description | Entry | Live | Status |
|---|---|---|---|---|
| 8 | Fed 1x cut YES | 21c | 18c | ⚠️ Documented loss, hold |
| 13 | GDP >2.0% YES | 60c | 43c | HELD — calibration value |

Record: 8W-5L-5EE | P&L: +$139.14

---

## VALIDATION TRACKER

| Model | Status | Gate 1 Progress |
|---|---|---|
| JOBS | ✅ GO-LIVE ELIGIBLE | Real cron test: Jul 14 CPI |
| GDP | Active, own track | Trade #13 held, Jul 30 |
| CPI | Active | 4.0% consensus, Jul 14 — 8 days |
| Claims | SUSPENDED | 3 consecutive losses |
| MLB_GAME | Track A — Day 10 clean | n=73, Brier 0.2558, Jul 25 |
| WC_GAME | 🔴 LAMBDA FIX TONIGHT | Two-line defense |
| NFL | Week 1 — Signal Contract | Carries to tomorrow |
| Fed | Active, own track | Trade #8 documented loss |
| SEDE Portfolio | ✅ Live | $994.17, 1 open |
| GrokBot v8 | Day 9 of 60 | Running clean, 1W/0L |

---

## TONIGHT'S PRIORITIES (strict order)

0. Oracle sync — Rus runs sede-pull. Archie confirms hash.

1. Verify paper_trades.json — are Trade #8 and #13 still open?
   Resolves the 7AM vs 7:45 AM alert discrepancy.

2. Lambda diagnostic — pull actual λ values for France, England,
   USA vs their opponents. How flat is the curve?

3. Lambda fix — steepen the Elo-to-lambda conversion for quality
   gaps above 150 Elo points. Validate against 1,600+ signal
   history before deploying.

4. Minimum win probability gate — if Kalshi >55c and model <60%:
   skip signal, log the gate firing. One line.

5. Open items resolution — UNEMP_CONSENSUS decision (ask Rus),
   Trades #23/#24 fix, jobs_model.py date fix, June 30 archive.

6. Oracle cron code-pull decision — yes or no. Decide tonight.

7. WC backfill — all R16 results. Verify Kalshi strings first.

8. Signal Contract spec — if time allows after lambda fix.
   If not, tomorrow is Week 1 Day 2 and it starts then.

---

## KEY DATES

| Date | Event |
|---|---|
| Jul 6 TONIGHT | USA vs Belgium R16 8 PM ET Seattle |
| Jul 7 | GDPNow next update — Trade #13 watch |
| Jul 9 | Morocco vs France QF (Boston) |
| Jul 10 | Spain/Portugal vs USA/Belgium QF (LA) |
| Jul 11 | Norway vs England QF (Miami) |
| **Jul 14** | **CPI release — REAL first 7:45 AM cron test** |
| Jul 19 | World Cup Final |
| Jul 25 | MLB Track A/B verdict + NFL architecture |
| Jul 30 | GDP advance estimate — Trade #13 resolves |
| **Aug 2** | **Vegas — Anthony (8rain) demo — 27 days** |
| Aug 26 | GrokBot v8 go-live eligibility |
| Sep 3 | NFL season opens |

---

## HARD RULES (standing, non-negotiable)

1. Oracle sync is always Step 0. Manual. Human-executed. Always.
2. Never claim a file is created/done without Get-Item/Test-Path
   verification. Read it back. Check the timestamp.
3. "Confirmed done" claims from prior sessions are unverified
   until independently checked.
4. NFL dual-gate: total edge ≥20c AND proprietary ≥10c. Both.
5. Signal Contract spec written before any NFL model code.
6. paper_trades.json hold to resolution unless thesis structurally
   broken. Calibration value outweighs real-money risk heuristics.
7. sede_portfolio.json and paper_trades.json serve different
   purposes. Never apply one ledger's logic to the other.

---

## CONFIRMED API REFERENCE

| API | Base URL | Auth | Notes |
|---|---|---|---|
| Kalshi | external-api.kalshi.com/trade-api/v2 | RSA key | Integrated |
| SharpAPI | sharpapi.io | API key | Free tier, MLB + NFL |
| Polymarket Gamma | gamma-api.polymarket.com | None | 10 req/sec |
| GDPNow | atlantafed.org/cqer/research/gdpnow | None | Next update Jul 7 |

---

*Prepared by J@rv1s (Web Claude) | Session: Monday July 6, 2026*
*Lambda fix is tonight's primary build. Two lines of defense.*
*Diagnostic first. Fix second. Validate before deploying.*
*USA vs Belgium tonight. 27 days to Vegas.*
*Papa Ralph standard. If it's worth doing it's worth doing right.*
