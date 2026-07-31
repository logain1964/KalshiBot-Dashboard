# J@rv1s -> Archie: Monday July 27 + Tuesday July 28 Combined Briefing
## STATUS: Both nightly summaries reviewed. One open question carried
## from Monday, two good bug catches confirmed from Tuesday, one trade
## trend flagged for awareness. Nothing built or ratified from this
## side. For Rus's call.

---

## FROM MONDAY (ITEM #15 -- ELO CORE)

Reviewed in full. Genuinely solid session:
- Unit-test values traced back to the ratified specs directly (wind
  >25mph -10 + precipitation -3 = -13 stacking, confirmed matches
  item #11/#12's ratified numbers) -- real confirmation the code
  implements what was actually ratified, not just code that compiles.
- 63.5% vs. 62.1% accuracy improvement (n=3,903) measured directly, not
  assumed from "the constants are better now."
- NBA_GAME's zero-rows-ever-logged bug found incidentally, correctly
  flagged and deferred rather than becoming unplanned scope creep.

**One open question, carried forward, not urgent:** the relocated-team
bug fixed in item #15 (stale Oakland/San Diego/St. Louis entries) is
the SAME class of bug already found once before, in the July 20
weekend's Elo convergence recheck (STL->LA, SD->LAC, OAK->LV). Finding
and fixing it again here, in a different file, was correct for that
session -- but worth asking directly: **is relocation-handling logic
centralized anywhere reusable, or is it being independently
rediscovered file by file?** If the latter, worth centralizing once
(similar spirit to linux_paths.py's role on the SEDE side) rather than
risking a third rediscovery in some future file. Not blocking, just
worth a real answer before it happens a third time.

---

## FROM TUESDAY (ITEMS #16-18 -- WIRED TOGETHER)

Two real, well-handled bug catches, both worth naming specifically:

**1. QB tier determination (Item #16).** "Most experienced QB on
roster" as a starter proxy was a reasonable-sounding heuristic that
turned out concretely wrong (returned Mac Jones over Brock Purdy for
SF) -- not a theoretical risk, a real wrong answer, caught by testing
directly rather than trusting the logic on inspection. Good that the
fix got verified against two independent real cases: SF corrected, plus
an unplanned, current real-world test (Cousins named over Las Vegas's
rookie #1 pick) that the function handled correctly. Right level of
confirmation before trusting this function going forward.

**2. Ticker-slicing (full signal loop).** Fixed-width team-code
assumptions broke the first time the loop ran against realistic
tickers (2-letter codes like KC, NE, LA, SF, GB, TB) -- exactly why
"verified end-to-end" matters more than "verified in isolation." Good
this surfaced now, against synthetic data, rather than in September
against real markets.

**Trade #13 -- flagging plainly, not alarmingly.** Move from the
46-54c range down to 34c is a real, sizable drift in the wrong
direction, right as the July 30 GDP resolution approaches. Nothing
actionable per Archie's own note, "hold to resolution" stands -- but
this is trending toward a genuine loss rather than a coin-flip. Worth
having that expectation set going into Thursday rather than being
surprised by the resolution.

---

## NET FOR RUS

1. Both sessions reviewed clean -- no corrections needed on the actual
   engineering.
2. One real open question for Archie: is relocation-handling logic
   centralized, or independently rediscovered per file? Worth a
   definitive answer before a third instance of this same bug class
   shows up somewhere else.
3. Trade #13 worth mentally preparing for a likely loss at the July 30
   GDP resolution -- no action needed, just not a surprise if it lands
   that way.

Nothing built. Nothing ratified from this side. For Rus's call.

---

*J@rv1s | Papa Ralph standard. If it's worth doing it's worth doing right.*
