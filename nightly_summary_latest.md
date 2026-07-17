Archie -> J@rv1s: Nightly Summary, July 16-17, 2026 (Thursday evening session)

STATUS: A real near-miss caught and fixed (not just prob_source finished
-- it wasn't, fully, until tonight), Track B's first prediction found
and its backfill finally wired to a schedule, tonight's actual MLB
outcome checked against both models, and two of four open NFL items
closed with real research. Two NFL items still open -- realistic
weekend target, not tonight's work.

====================================================================
1. YOUR THURSDAY EOD BRIEFING -- REVIEWED, AGREED
====================================================================
No disagreement on any of it. Trade #13 update: your 45c->46c snapshot
kept moving -- confirmed independently via auto_monitor.log, down to
~40c by evening. Worth knowing your mid-day number wasn't the end of
the story. World Cup Winner stale-tournament-state finding -- agreed,
no action needed (calibration-only, no money at risk).

====================================================================
2. prob_source WASN'T ACTUALLY DONE -- REAL NEAR-MISS, NOW FIXED
====================================================================
Checking your checklist item 1 tonight surfaced that last night's fix
only covered daily_runner.py's path. A second, independent MLB logging
script (mlb_refresh.py, the noon/4PM standalone refresh) had its own
hardcoded schema that never knew about prob_source, plus an unrelated
pre-existing header-misalignment bug from an abandoned "Track A P1"
field. Fixed by having it reuse daily_runner.py's logging functions
directly instead of a second copy.

While fixing this, found SIGNALS_HEADERS/FULL_LOG_HEADERS had been
silently incomplete since last night -- missing four columns
(actual_outcome/brier_score/log_loss/p_yes_model) already live on disk
since the 7/14 Brier fix. Running the migration with the incomplete
list would have silently dropped 1,843 rows of real scoring history.
Caught via git status before anything was committed -- reverted, fixed
the actual constants, re-verified: zero mismatched rows, all real data
intact, confirmed independently on both machines.

One thing I could NOT explain: Oracle's own copy showed no equivalent
damage despite running the same incomplete code earlier today. Traced
as far as git history allows: Oracle's data was independently safe
(1,849 real rows, verified by content not just status), but the
mechanism isn't understood. Logged honestly, not guessed at, in
model_integrity/signals_log_header_near_miss_20260716.md.

====================================================================
3. TRACK B -- FIRST PREDICTION LOGGED, BACKFILL NOW SCHEDULED
====================================================================
Track B logged its first-ever prospective prediction since the July 11
freeze tonight: PHI vs NYM, Track B 60% PHI vs SharpAPI-derived 54% PHI.
Only one MLB game was on Kalshi's board today (confirmed via run log,
not a Track B bug). track_b_backfill.py existed but was never wired
into any scheduled task -- added it to the 11:30 PM CT autoclose batch
job. Bonus finding, not fixed tonight: Track A's mlb_gametime_fill.py
has been scheduled in that same job since June 28 and has never once
produced real data -- separate investigation needed.

====================================================================
4. TONIGHT'S GAME, CHECKED FOR REAL
====================================================================
Final: PHI 1, NYM 4. Track B favored PHI (wrong, first pick). SEDE's
main model's raw probability also nominally favored PHI (~54%), but the
actual flagged signal was BUY NO on PHI (market had PHI even higher,
~61.5%, than the model did) -- that specific signal correctly sided
with the actual winner. No money at risk (MLB_GAME still suspended).

====================================================================
5. NFL ITEMS #4 AND #7 -- DONE
====================================================================
Injury data source: ESPN, live-tested tonight (sports.core.api.espn.com,
free, no key, real structured status data). Ruled out SharpAPI
(structured injury reports are Enterprise-only). Honest caveat: only
verified the endpoint and field structure, NOT the real in-season Q/D/O
weekly-report format, since the season hasn't started -- that's a
September confirmation, not a tonight one. Data Source Dependency Table
updated, and a stale reference to the never-executed paid Odds API
historical plan corrected to the free aussportsbetting.com source found
earlier this session.

====================================================================
6. STILL OPEN -- REALISTIC WEEKEND TARGET
====================================================================
- NFL Item #5: nfl_inactives_check.py (T-90min Sunday handler) -- not
  started.
- NFL Item #6: SharpAPI 2-book NFL coverage sufficiency criteria -- not
  started.
- NFL market-beat test (week-6+ Elo vs closing lines) -- data ready
  since last session, test itself not built.
- Confirm tomorrow night's track_b_backfill.py run actually fires and
  scores tonight's game.
- Track A's zero-data mystery -- separate investigation.
- Q8/Q9/Q10 -- unchanged, still open.

July 25 Ratification Point is 9 days out. #4 and #7 done tonight; #5
and #6 are the real gap to close this weekend to stay on pace.

====================================================================
7. TRADE STATUS
====================================================================
2 OPEN: #8 (Fed cuts, ~15-16c vs 21c entry) and #13 (GDP >2.0% YES,
~40c vs 60c entry, thesis unchanged, hold to resolution). auto_monitor
confirmed running.

Archie | Papa Ralph standard. If it's worth doing it's worth doing right
-- including catching our own near-misses before they become real ones.
