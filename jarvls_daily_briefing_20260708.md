# Briefing for Archie — from J@rv1s
## Wednesday July 8, 2026 (morning session)

---

## 1. CRON FIX CONFIRMED WORKING

7AM CT run fired exactly on time today (07:00 AM CT daily report timestamp).
Timezone fix from last night is verified end-to-end. No further action needed
on this one.

---

## 2. WIN RATE DISCREPANCY FOUND — NEEDS A DECISION, NOT JUST A FIX

While cross-checking this morning's Daily Report against `paper_trades.json`,
found that trades **#17–#20** (the first live $1 real-money MLB trades,
entered 2026-05-31) are excluded from the "18 closed / 10 wins" figures the
Daily Report generates.

**Root cause:** those four trades were entered manually and use
`"status": "LOST"` directly, instead of `"status": "CLOSED"` + a separate
`"result"` field like every other trade in the file. Whatever script builds
the Daily Report is filtering on `status=="CLOSED"`, so it silently skips
all four.

**Effect on the numbers:**
- Reported: 18 closed, 10 wins → **55.6%** WR (just above Gate 1's ≥55% bar)
- Actual if #17-20 included: 22 closed, 10 wins → **45.5%** WR (below the bar)

All four of the excluded trades were losses (real-money, $1 each, MLB game
model). Their exclusion isn't malicious or deliberate — it's a schema
artifact of manual entry — but right now it's an *accidental* exclusion that
happens to make the win rate clear the Gate 1 threshold. That's worth fixing
regardless of which way the decision goes.

**Two legitimate options, pick one on purpose:**
1. Normalize #17-20 to match the standard `status: CLOSED` + `result` schema
   and include them in Gate 1 calculations — live trades are real evidence,
   arguably *more* meaningful than paper signals.
2. Deliberately carve out live-money test trades as a separate track from
   the paper-validation Gate 1 pool, and document that decision explicitly
   in the model integrity docs so it doesn't look like cherry-picking later.

Either is defensible. What's not defensible is leaving it as an unexamined
side effect of a status-field typo. Flag for Rus + Archie to jointly decide,
then normalize the schema either way so this doesn't recur with future manual
entries.

---

## 3. THREE OVERLAPPING DATA FILES CAUSED CONFUSION THIS MORNING

Spent real time this morning reconciling: nightly_summary_latest.md (bug
report, no portfolio data) vs. sede_portfolio.json (separate $1000
subscriber-demo track, 2 trades total) vs. paper_trades.json (the actual
24-trade validation ledger). All three are legitimately separate by design
per the file headers, but there's no single doc that says "here's which
file answers which question." Worth a one-time index/README so this doesn't
eat another morning.

---

## 4. PRIORITY CHECK — REFOCUS RECOMMENDED

Flagged to Rus, repeating here for Archie: recent sessions (this one
included) have been reactive — bug-fixing and report reconciliation — while
the two items with actual hard deadlines haven't moved:

- **MLB Foundation Document verdict** — due July 25, structural question
  (genuine model vs. disguised arbitrage) still open
- **NFL Signal Contract spec** — not started, hard deadline Sept 3, and
  it's supposed to be built spec-first specifically to avoid repeating the
  MLB mistake

Also still open, lower urgency: `on_signal_resolved()` stub (hard
prerequisite before subscriber onboarding), 9-file hardcoded-path audit
(one file already confirmed live/broken), June 30 archive still missing.

Recommend next session prioritizes MLB verdict or NFL spec progress over
further reporting/infrastructure cleanup, now that today's two loose ends
(cron confirmation, win-rate root cause) are closed.

---

*J@rv1s | Papa Ralph standard. If it's worth doing it's worth doing right.*
