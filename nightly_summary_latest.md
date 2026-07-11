# KalshiBot Nightly Summary
## For J@rv1s -- Saturday July 11, 2026 (Archie session, afternoon/evening)
Prepared by Archie | Opus 4.8

---

## TL;DR FOR J@RV1S

Rus is back after a few days out sick. Picked up from your July 8 briefing.
Two things resolved to decisions tonight, one real second bug found, and
one FORGE handoff waiting for your independent pass. Details below.

**ACTION FOR YOU:** Read `FORGE_handoff_winrate_definition.md` (in the
dashboard repo) and do a genuine adversarial pass on the win-rate
definition. It is written to be attacked, not rubber-stamped. Then back
to Rus for final ratification. Nothing is written to governance docs yet.

---

## 1. WIN-RATE SCHEMA BUG (#17-20) -- FIXED AT ROOT, COMMITTED

Your July 8 flag was correct and is now resolved in code, not just data.

- Root cause was in `mlb_autoclose.py` `auto_close()`: it wrote the
  WON/LOST outcome directly into the `status` field instead of
  `status=CLOSED` + `result=<outcome>`. That's why #17-20 were excluded
  from every `status=="CLOSED"` filter.
- IMPORTANT: this bug would have silently excluded future live MLB WINS
  too, not just the four May 31 losses. It only looked loss-specific
  because the first live batch happened to go 0-for-4.
- Fixed the function so all future live MLB trades use correct schema.
- Normalized #17-20 in the production ledger (C:\KalshiBot\data\
  paper_trades.json) to standard schema.
- Committed + pushed: commit 3f96d0c.

Process note / self-correction: Archie first edited the WRONG copy of
paper_trades.json (the C:\KalshiBot-Dashboard mirror, not the C:\KalshiBot
production repo). Caught it via git history check before committing, then
fixed the real file. Same numbers, no harm, but logging it honestly.

**True win rate with #17-20 included: 45.5% (22 closed, 10 wins) by the
pnl>0 method.** Below Gate 1's 55%. See item 3 -- the definition itself
is now in question.

---

## 2. TRADE #13 DECISION -- HOLD TO RESOLUTION (Rus ratified)

GDP >2.0% YES @ 60c. GDPNow at 1.3% as of July 8 (down from 1.4% Jul 1).
Thesis has decayed hard; this is very likely a loss by the Jul 30 event.
Rus decided to hold to resolution regardless -- "data is data, good or
bad." Same principle as #16. Clean data point over a protected record.

---

## 3. WIN-RATE DEFINITION -- FORGE PASS RUN, AWAITING YOUR REVIEW

While fixing #1, found a SECOND, deeper issue: the two reporting scripts
define "win" differently and disagree.
- trade_monitor.py: win = pnl_dollars>0 -> 45.5% (10/22)
- validation_dashboard.py: win = result=="WON" -> 36.4% (8/22)
- Gap = 5 EARLY_EXIT trades, incl. #23/#24 which made money via the
  stop-loss bug you and Rus fixed last session.

Ran full Bias/FORGE/Papa Ralph in Opus. Key findings:
- Papa Ralph gate reframed it: Gate 1 needs >=75 closed trades, we have
  22. This is NOT a live go-live blocker. Zero time pressure.
- "win rate" is undefined in the Foundation Document (appears once, line
  398, in passing). Real definitional gap.
- Recommendation (Rus PROVISIONALLY accepted, pending your pass):
  resolution-only win rate = WON/(WON+LOST) = 47.1% (8/17), with
  EARLY_EXIT as a separate third category. Home: Foundation Document.

Rus's instruction: write it up for your independent pass FIRST, do NOT
edit the Foundation Document, do NOT treat his acceptance as final until
you review and he confirms. Handoff doc is
`FORGE_handoff_winrate_definition.md`. It includes the dissenting
pure-pnl case and four specific attacks on the recommendation. Please
actually try to break it.

---

## STILL OPEN (carried, not touched tonight)

- MLB Foundation Document verdict -- due July 25 (structural: genuine
  model vs disguised arbitrage). NOT moved tonight.
- NFL Signal Contract spec -- not started, hard deadline Sept 3.
- on_signal_resolved() stub -- hard prereq before subscriber onboarding.
- 9-file hardcoded-path audit -- still outstanding.
- June 30 session archive -- still missing.

Rus was out sick several days, so the refocus-to-deadline-work push from
your July 8 briefing is still the right call for the next working session.

---

## OPEN POSITIONS (verified against ledger tonight)

Only TWO open, not five (my earlier memory was stale):
- #8  Fed cuts 1x 2026 YES @ 21c -- market ~19-21%, edge roughly flat
- #13 GDP >2.0% YES @ 60c -- GDPNow 1.3%, holding to resolution (see #2)

Closed since last briefing: #12 WON +1.24, #15 WON +19.36, #16 LOST -24.92.

---

## COMMITS PUSHED TONIGHT

```
3f96d0c  Fix schema bug excluding live MLB trades from win-rate calcs
```

Plus one un-committed governance handoff doc (dashboard repo, not the
code repo): FORGE_handoff_winrate_definition.md.

---

*Archie | Opus 4.8 | Session ~4:00 PM CT | July 11, 2026*
*Papa Ralph standard. If it's worth doing it's worth doing right.*
*Nothing ratified to governance docs. Win-rate definition awaits J@rv1s pass.*
