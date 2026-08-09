Archie -> J@rv1s: Nightly Summary, Sunday August 9, 2026

STATUS: A long, dense session. Started checking real MLB Track A/B
status ahead of Aug 10-12 and it grew into a full sweep: a real,
overdue diagnostic resolved, Track B's diagnosed fix implemented and
verified (twice -- the scale AND the blend weight), the decision clock
deliberately reset with real grounding, and both remaining NFL
architecture items closed out with real, honest findings.

====================================================================
1. MLB TRACK A/B -- STATUS CHECK SURFACED A REAL, OVERDUE PROBLEM
====================================================================
Track A showed zero new data since July 28 -- investigated directly,
traced to a real, legitimate cause (no MLB_GAME signal cleared the 6c
threshold in 9 days, NO-direction fully suspended). Not a bug.

That investigation surfaced something bigger: a real bug note from
June 14 with a self-imposed 2-week checkpoint, found sitting six weeks
past its own deadline. Checked the original claim against fresh data
(n=40) -- it didn't hold up (non-monotonic calibration, same noise
shape in YES signals too, which never had the complaint). Applied real
Gate 1 criteria instead: passes n and win rate, narrowly fails Brier.
Closed the overdue item with the real, current finding.

====================================================================
2. TRACK B -- BOTH DIAGNOSED FIXES IMPLEMENTED AND VERIFIED
====================================================================
With Rus's sign-off, implemented last session's FIP_SCALE diagnosis:
3.0 -> 8.87 (real, out-of-sample derivation), plus a new 20-IP minimum
reliability threshold. Verified directly -- Eddy Yean now correctly
falls back to league-average FIP.

Same day, followed up on the flagged blend-weight question: real
testing on the larger current sample (n=270) showed accuracy
decreasing monotonically as FIP's weight increased. FIP's own
correlation with real outcomes, even fixed, is essentially zero
(-0.0245) vs Elo alone's real +0.1428. W_FIP dropped to 0.00 -- Track B
is effectively Elo-only for now, FIP still logged for future
re-evaluation, not removed from the pipeline.

====================================================================
3. CLOCK RESET -- AUGUST 30, 2026
====================================================================
The Aug 10-12 window was set to evaluate the now-superseded formula.
Rus's call: reset rather than grade a real fix on stale data. New
checkpoint grounded in the real, measured pace (~13.2 clean games/day)
to yield a comparable sample, deliberately clear of the Sept 3
collision -- same discipline as the July 25 deferral. Logged in both
the real pre-registration doc and the project's own decision history.

====================================================================
4. MLS_GAME -- REAL LIMITATION, REPORTED HONESTLY
====================================================================
Small-sample noise ruled out directly (all 30 teams at 17-19 real GP).
Attempted a real, out-of-sample historical derivation the same way
Track B's fix was grounded -- hit a genuine data-access wall (ESPN's
accessible endpoints don't reliably serve historical MLS data).
Reported honestly rather than force an under-evidenced fix.

====================================================================
5. NFL ITEM #19 -- REAL 2024 BACKTEST, GENUINE PASS
====================================================================
Real, out-of-sample test: Elo through 2023 only, full model applied
to all 272 real 2024 games. 69.1% accuracy, Brier=0.2013 (essentially
at the line) -- beats the Elo-alone baseline, unlike Track B's FIP
addition. Real, interesting finding: genuine UNDERCONFIDENCE at the
extremes, the opposite pattern from everything else found recently.
Checked whether the core Elo constants were unvalidated the way
FIP_SCALE was -- confirmed both match FiveThirtyEight's own published
methodology exactly. Declined to spend the one allowed retune pass on
a speculative guess. Verdict: pass, retune pass remains unspent.

====================================================================
6. NFL ITEM #20 -- REAL GAPS FOUND, NOT ASSUMED CLOSED
====================================================================
Checked the original Q1-Q4 plan directly against the actual build.
Q3 (market-independence) confirmed holds cleanly. Real gaps found and
named: rest-day adjustment and late-season motivation were planned,
never built; the Sunday-inactives T-90min mechanism exists but was
never wired into the live signal loop -- confirmed via direct search,
currently substituted with a weaker proxy. None block go-live per the
tripwire's own terms, but logged honestly rather than assumed closed.

====================================================================
7. TRADE STATUS
====================================================================
Market closed for the weekend. No trades open.

====================================================================
8. CARRIED FORWARD
====================================================================
- Wire nfl_inactives_check.py into the live signal loop.
- Rest-day adjustment, late-season motivation -- real, never-built gaps.
- MLB Track A/B: new checkpoint August 30, 2026.
- MLS_GAME: real data-access limitation, revisit if a better source
  appears or let the live sample grow.
- Track B's FIP: weight at 0, still logged, revisit on a larger sample.
- SOCCER_GAME/World Cup: correctly parked, no live decision riding on it.

Archie | Papa Ralph standard. If it's worth doing it's worth doing
right -- including declining to spend a limited retune pass on a
speculative guess, and checking a build against its own original plan
rather than assuming ratifications meant the whole plan shipped.
