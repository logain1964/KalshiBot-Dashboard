# J@rv1s Daily Briefing
# For Archie — Evening Session Handoff
# Monday June 15, 2026 | Prepared by J@rv1s (Web Claude)

---

## TOP OF FILE — BIG PRODUCT DAY

Today was the first dedicated product design session. Two significant
decisions were made and one major document was produced. Read the
PRODUCT section before the ops items tonight.

---

## SPORTS — NO OPEN POSITIONS

Both series closed over the weekend per Archie's nightly summary:
- CAR Stanley Cup ✅ WON +$19.36 (Trade #15)
- NYK NBA Championship ❌ LOST -$24.92 (Trade #16, held deliberately)

No live sports positions. No scores to monitor tonight.

MLB_GAME direction bug fix (Steps 3+4 from Sunday night's directive)
is tonight's highest-priority build task — once deployed, MLB_GAME
YES signals start generating clean data and the Gate 1 clock runs.

---

## OPEN POSITIONS

| #  | Description    | Entry | Status                        |
|----|----------------|-------|-------------------------------|
| 8  | Fed 1x cut YES | 21c   | ~20c, long-dated, no action   |
| 12 | GDP >2.5% YES  | 40c   | ~50c, +10c, GDPNow 3.29%     |
| 13 | GDP >2.0% YES  | 60c   | ~66c, +6c, comfortable        |

FOMC decision Wednesday June 17 — watch Trade #8. Model flagging
"Fed cuts 0x 2026 BUY YES" at 77% confidence — not a new entry
recommendation (correlated with existing position), just worth
knowing the model thinks no-cut is the more likely outcome heading
into Wednesday.

---

## VALIDATION TRACKER

| Criteria      | Current  | Status                        |
|---------------|----------|-------------------------------|
| Closed trades | 17       | Per-model gates now in effect |
| JOBS win rate | 58.9%    | ✅ Gate 1 passes               |
| JOBS n        | 73       | ✅ Gate 1 passes               |
| JOBS Brier    | 0.134    | ✅ Gate 1 passes               |
| P&L           | +$137.90 | Strong                        |

---

## PIPELINE ALERT — STALE MARKETS IN THIS MORNING'S OUTPUT

This morning's daily alert contained:
- "Spurs Win NBA Championship — BUY NO" (NYK won yesterday)
- "Thunder Win NBA Championship — BUY NO" (OKC eliminated weeks ago)
- "Unemp>4.0% Apr" and "Unemp>4.4% Apr" (April data already released)

This is now P1 BLOCKING for subscriber launch. Stale markets reaching
subscribers would immediately destroy trust. The Kalshi API returns a
status field — resolved/closed markets must be excluded from signal
output before any subscriber sees it.

P1 filter: check Kalshi API market status before including ANY market
in flagged edges. Resolved or closed → skip entirely.

This is also directly referenced in the new product spec (see below).
Flag this as a dedicated build item tonight alongside the MLB bug fix.

---

## PRODUCT DECISIONS — MAJOR SESSION TODAY

### Decision 1: Delivery Infrastructure — Web Dashboard Primary

Email and Telegram are not the primary delivery method. They are the
NOTIFICATION LAYER only.

The primary subscriber experience is a WEB DASHBOARD. Here's why:

- Email open rates ~20-25% — 75% of subscribers miss most signals
- Telegram is better but you're building on a platform you don't control
- Both are push-only, passive, no subscriber engagement or agency
- Neither allows subscribers to see the live portfolio, track record,
  or upcoming catalysts without a signal arriving

The SEDE model portfolio trading its own money in real time is the
differentiator. That story is only visible on a dashboard. It can't
be communicated through notifications alone.

ARCHITECTURE:
- Primary: Web dashboard (live portfolio, signal history, upcoming
  catalysts, model performance by tier)
- Supporting layer: Telegram and/or email notifications when something
  happens ("New signal — see dashboard", "Trade closed WIN — see
  dashboard")
- No-signal days: brief notification only, no dashboard click required

This differentiates SEDE from every other signal service. Most are
Telegram channels or email lists. SEDE is a live performance you
can watch. The dashboard makes that real.

UPDATE TO PRODUCT SPEC: P6 (Delivery Infrastructure) is now partially
defined. Primary experience = web dashboard. Supporting layer =
Telegram/email notifications. The "scope delivery infrastructure"
session still needed for full build scoping, but the architecture
decision is made.

### Decision 2: Trade Closing From Work — Resolved

Current state: auto_monitor.py handles stop-losses automatically on
the laptop. Manual trade closes require either the laptop or direct
Kalshi web access.

PRACTICAL ANSWER: For urgent manual closes during business hours,
Rus logs into kalshi.com from any browser directly. No pipeline
required. Takes 60 seconds. Pipeline picks up the closure on the
next run and updates paper_trades.json automatically.

Oracle remote control (web interface or API endpoint for triggering
trade actions remotely) is a Phase 3 build item — useful eventually,
not a current blocker while paper trading. Don't build it now.

---

## PRODUCT SPEC — NOW IN PROJECT FILES

sede_product_spec_v1 is now the fifth permanent project document.
Clean version — no stale dates, no specific model status, no named
individuals beyond Rus, no market examples that expire.

Key items from the spec relevant to tonight's build work:

P1 — Stale Market Filter: BLOCKING (stale markets confirmed in today's
alert — this is now urgent, not theoretical)

P3 — sede_portfolio.json: new file needed for SEDE's autonomous
portfolio, separate from paper_trades.json. Starting parameters:
  - Bankroll: $1,000 (paper → real at go-live)
  - Position sizing: $15 MEDIUM / $25 HIGH / $30 HIGH + edge ≥25c
  - Max concurrent: 8
  - Hard drawdown stop: -$200

These are separate tracks:
  paper_trades.json = Rus's validation record (keep as-is)
  sede_portfolio.json = SEDE's autonomous subscriber-facing portfolio

P6 — Delivery infrastructure: web dashboard primary, Telegram/email
notification layer. Architecture decided. Full build scope TBD.

---

## PRODUCT SPEC UPDATE NEEDED (minor)

The product spec section on P6 still reads "Not yet decided." Given
today's decision (web dashboard primary, notifications secondary),
update the spec to reflect the architecture decision. This is a
minor text update, not a FORGE session — the decision was made today
and is logged here. Rus will update the project file directly.

---

## TONIGHT'S PRIORITIES (suggested order)

1. MLB_GAME direction bug fix — Steps 3+4. Four surgical changes.
   Gate 1 clock starts after this. Highest-value build available.
   Manual verification confirmed Sunday, API field confirmed (yes_sub_title).
   
2. P1 Stale Market Filter — confirmed urgent by today's alert.
   Check Kalshi API status field on each market before signal
   generation. Resolved/closed → skip entirely. This is BLOCKING
   for subscriber launch and now confirmed as a real daily problem.

3. Oracle pull — weekend commits not yet on Oracle per Sunday summary.

4. WC_GAME scanner name mismatch — "United Sta vs Australia" label.
   Display issue or mapping issue? Needs G-gate check before WC_GAME
   accuracy data is trusted.

5. Claims Fixes 1/2/5 — holiday detection, aftermath detection,
   confidence tagging. ~3 hours total. Gets the weekly-resolution
   model clock running. Worth scheduling if time allows.

6. Carried low priority: hardcoded-path grep, folder move, bashrc
   dedupe, cron timing discrepancy, paper_trades.json #17-20 status
   field consistency.

---

## KEY DATES

| Date    | Event                                        |
|---------|-----------------------------------------------|
| Jun 17  | FOMC decision — watch Trade #8               |
| Jun 26  | World Cup group stage ends — WC_GAME checkpoint |
| Jul 2   | Jobs report 8:30 AM CT                        |
| Jul 30  | Q2 2026 GDP advance estimate                  |
| Aug 2   | Vegas departure                               |
| Aug 15  | SEDE checkpoint/review (not a hard deadline)  |
| Sep 3   | NFL season opens                              |

---

## SYSTEM STATUS

| Component         | Status                                      |
|--------------------|----------------------------------------------|
| Dashboard handoff  | SOLVED, both directions confirmed           |
| Oracle Cloud       | LIVE — needs weekend commit pull            |
| JOBS               | Validated, go-live eligible                 |
| MLB_GAME YES       | PAUSED — direction bug fix tonight          |
| MLB_GAME NO        | SUSPENDED — same bug, fix tonight           |
| WC_GAME            | CALIBRATION ONLY, scanner mismatch TBD     |
| Claims             | SUSPENDED, Fixes 1/2/5 outstanding         |
| GDP                | Active, weight 0.60, 3 open positions      |
| Fed                | Active, Trade #8 open, FOMC Wednesday      |
| P1 Stale Filter    | NOT BUILT — confirmed urgent today         |
| sede_portfolio.json| NOT BUILT — needed for product launch      |
| Product spec       | ✅ In project files as sede_product_spec_v1 |

---

*Prepared by J@rv1s (Web Claude) | Session: Monday June 15, 2026*
*Papa Ralph standard. If it's worth doing it's worth doing right.*
