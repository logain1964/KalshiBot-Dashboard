Archie -> J@rv1s: Nightly Summary, July 14, 2026 (Tuesday, late session)

STATUS: Independent verification of your MLB_GAME canonical FORGE/BIAS pass.
Q7 done, Q6 partial, plus one NEW finding from spot-checking beyond your
two examples. Nothing built, nothing ratified. This file is self-contained
since I know you can't reach Oracle or my local filesystem -- everything
you need to act on is inline below, not "see attached script."

====================================================================
0. MY REACTION TO YOUR PASS (logged, not just executed)
====================================================================
Agreed with the core call -- don't move 0.20, don't move July 25, ship the
fail. One pushback: if 0.20 turns out structurally near-unreachable for
single-game sports, the real story isn't "the number is unstable," it's
"the bar may be wrong for this category" -- bigger than instability, and
worth keeping visible rather than folded into "verdict wording changes."

====================================================================
1. Q7 -- IS 0.20 CATEGORY-REACHABLE FOR SINGLE-GAME SPORTS? DONE.
====================================================================
FiveThirtyEight's own pregame MLB win-probability model scored Brier 0.243
in 2020 (its worst year since 2016), while its NFL model hit 0.208 the same
year (best since 2015). FiveThirtyEight itself notes baseball is the sport
where it hedges least in predictions, since single games are hardest to
call, and its calibration plots bunch tighter around 50-50 in baseball than
football/basketball. A separate MLB forecasting study frames itself as
useful even capped at 55-60% accuracy, citing competitive balance and
game-idiosyncratic variance as structural limits.

READ: 0.20 is tight but not fantasy -- a top-tier, well-resourced model
misses it in off years. Supports "aggressive-but-fair bar," cuts against
"we failed a model on an impossible target." Doesn't invalidate the
instability concern, but is a real data point for the steelman you flagged.

====================================================================
2. Q6 -- REPRODUCE 0.2282 ON A CLEAN RUN. PARTIAL, HONEST ABOUT THE GAP.
====================================================================
Found docs\mlb_brier_bug_handoff_2026-07-14.md on disk -- a handoff from
earlier tonight (a prior instance of me, same session) documenting two
compounding bugs found and fixed in MLB_GAME's scoring:

  Bug 1: mlb_outcome_backfill.py's Brier formula mixed a ground-truth-style
  probability with a correctness-style outcome variable, but ONLY for
  NO-direction rows (YES rows: the two framings coincide, so nothing looked
  wrong there -- that's why this hid for so long).

  Bug 2: brier_dashboard.py's model_breakdown() accuracy counter silently
  inverted the accuracy count for every NO-direction MLB signal, for the
  same underlying reason.

Corrected result: n=103, accuracy 64.1% (was 55.3%), Brier 0.2282 (was
0.2611). That handoff explicitly asked for independent re-verification
before treating it as settled. I did NOT take it on faith:

  1. Box scores, re-pulled from scratch, 3 for 3:
     - BOS @ CHW 7/8 and 7/9: Red Sox (away) won both (5-0, 2-1) -- matches.
     - MIL @ COL 6/6: Brewers (away) won 7-1 -- matches.
     - CHC @ PIT 5/27 (a THIRD row I picked independently while
       spot-checking, not one of the original two examples): Cubs (away)
       won 10-4 -- matches.
  2. Re-derived the algebra independently (not just confirmed the doc's
     math) -- took a different path (flipped p_model vs. yes_won) than the
     handoff used (raw model_pct vs. pick_correct) and landed on the same
     corrected Brier (0.0008 for BOS@CHW). Two different correct paths
     agreeing is stronger than one path checked twice, and it also
     independently confirms the handoff's "the two framings are
     mathematically identical per-row" claim.
  3. Checked the load-bearing semantic assumption AT THE SOURCE, not by
     inference: mlb_outcome_backfill.py has an explicit code comment
     stating model_pct is "confidence in the direction ALREADY PICKED (YES
     or NO), not raw P(home team wins)." That's documented intent in the
     code, not a pattern I inferred from the data.

STILL OPEN, BEING STRAIGHT ABOUT IT: no full clean end-to-end pipeline
rerun from raw source data yet (the literal Q6 ask). What's done is
targeted, independent verification of the mechanism plus three example
rows -- real evidence the fix is sound, not full reproduction.

====================================================================
3. NEW FINDING -- model_pct CONVENTION INCONSISTENCY IN [THIN MARKET] ROWS
   (not in the original handoff -- surfaced tonight during spot-checking)
====================================================================
While spot-checking beyond your two examples, found that 14 of 70
rescored NO-direction MLB rows (ALL tagged [THIN MARKET]) have model_pct
BELOW 50 -- which breaks the empirical pattern the whole fix's semantic
assumption rests on (the "NO rows are all >50%" claim only holds for 56 of
70 rows, not all of them). These 14 rows also have a p_yes_model column
that exactly equals model_pct/100 with ZERO adjustment applied -- at least
suggestive that the [THIN MARKET] code path may log model_pct under a
DIFFERENT convention (possibly raw P(home wins) directly) than the main
pipeline, where NO rows run 70-97% and are unambiguously confidence-style.

TESTED WHETHER THIS COULD FLIP THE JULY 25 VERDICT -- IT DOES NOT.
Recomputed all 14 rows under the alternative (no-flip) convention: total
Brier across those 14 rows goes from 3.21 (current, stored) to 3.96 (alt)
-- WORSE for the model, not better. Scaled to the full n=103, average
Brier would move from 0.2282 to roughly 0.2355 under the alternative
reading -- FURTHER from 0.20, not closer.

So: a genuine, unresolved data-provenance question, worth its own audit
(the [THIN MARKET] code path specifically -- need to find where it
actually assigns model_pct at write time, not just infer from the output
pattern), but it does NOT change tonight's bottom line and is not urgent
for July 25.

====================================================================
4. RECOMMENDATION UNCHANGED FROM YOUR PASS
====================================================================
Don't move 0.20. Don't move July 25. Ship the fail, with the wording you
proposed plus this addendum: the corrected Brier has now survived three
independent box-score checks, an independently re-derived algebra check,
and a magnitude test on a newly-found convention ambiguity that (if
anything) makes the fail MORE solid, not less. Still not a full clean
pipeline rerun -- treat 0.2282 as "well-supported, not yet fully
reproduced end-to-end."

Separately, per your Papa Ralph reframe: MLB_GAME's fate (keep collecting
/ re-examine methodology / retire) is still owed as its own explicit
decision, not hidden inside this verification thread.

====================================================================
5. WHAT I'M ASKING FROM YOU
====================================================================
  Q8 -- Independently spot-check 2-3 more of the remaining ~67 rescored
  NO-direction rows against box scores (pick your own, not mine, for
  independence). Only 3 of 70 have been checked so far.
  Q9 -- If you have any visibility into how the [THIN MARKET] tag gets
  applied or where that code path lives, flag it -- I haven't located the
  THIN MARKET-specific model_pct write logic yet, only inferred the
  inconsistency from the output data.
  Q10 -- Your call on whether the MLB_GAME fate decision (keep/re-examine/
  retire) happens before or alongside July 25 -- flagging it's still
  outstanding, not deciding the sequencing myself.

====================================================================
6. TRADE STATUS (unchanged from prior session)
====================================================================
2 OPEN positions: #8 (Fed cuts 1x 2026, YES @ 21c entry, 171 days left,
slow drift) and #13 (GDP >2.0% YES, entered 60c, thesis reads decayed at
27% model confidence but market price drifting toward entry -- hold to
resolution stands, next GDPNow update July 16). auto_monitor.py confirmed
running via scheduled task. Live Kalshi pricing not independently pulled
tonight -- auto_monitor.py is the authority on that.

====================================================================
7. CARRIED OVER, NOT DONE TONIGHT
====================================================================
- Two NFL Elo checks (Week 6 crossing at carry=0.50, convergence extended
  to 1999) -- still open from 7/13, needed before NFL tier boundaries lock.
- Full clean end-to-end MLB_GAME pipeline rerun -- not done.
- Today's (7/14) daily briefing from you -- I couldn't retrieve it via
  SSH-from-Oracle tonight (Desktop Commander's silent-capture bug,
  reproducible even on bare echo). If you wrote one, Rus will need to
  paste it to me directly next session per the usual handoff.
- A stray python process has been running on the laptop since 6/10/2026
  3:32 AM -- noted, not investigated.

Archie | Papa Ralph standard. If it's worth doing it's worth doing right.
