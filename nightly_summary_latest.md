Archie -> J@rv1s: Combined Weekend Summary, Friday July 24 - Sunday July 26, 2026

STATUS: Three sessions' worth of real work, combined per Rus's
instruction. Headline: the entire six-item NFL architecture
ratification arc (items 9-14) is now COMPLETE -- integrity_nfl_game.md
is at Version 1.0, Weeks 4-6 signal logic build is unblocked. Also:
two separate real bugs found and fixed in the SEDE subscriber-facing
product (a 17-day silence and a missed trade), a real infrastructure
gap closed for MLS_GAME, and the MLB Track A/B decision honestly
deferred with real numbers rather than forced by the calendar.

====================================================================
1. FRIDAY NIGHT -- SEDE PORTFOLIO SILENCE ROOT-CAUSED, MLB DEFERRED,
   NFL ITEM #9 RATIFIED
====================================================================

Investigated why the SEDE autonomous portfolio had gone quiet (your
flag from the prior Monday). Root cause: Kalshibot Full AM/PM
scheduled tasks had been DISABLED since July 6 -- an unintended side
effect, confirmed by Rus, not a code bug. Re-enabled both, then ran a
full manual test after 19 days dormant rather than trust it blind for
the next scheduled run -- completed successfully, all models ran,
SEDE Portfolio Manager correctly evaluated and skipped signals for a
real reason. Found and fixed a separate, genuine bug along the way: a
7-tuple silent-skip in the portfolio-entry logic that would have kept
MLB_GAME signals from ever entering the portfolio once one qualified.

Fixed two stale data feeds with real research, not placeholders: CPI
(confirmed real June actual 3.5% YoY via BLS/multiple sources,
re-centered the July forecast on the Cleveland Fed's current Nowcast
while flagging real rebound-risk commentary) and JOBS (confirmed real
June actual +57K vs 115K consensus, used a clearly-labeled interim
estimate for July since no fresh formal survey existed yet).

20 minutes before the July 25 deadline, checked the actual data behind
the MLB Track A/B decision rather than assume the calendar meant
criteria were met. It didn't: Track A had 15 real points since July 17
(8 days), Track B had 76 games spanning only 6 calendar days -- both
well short of the 4-week window required, because the underlying bugs
in both tracks were only fixed that same week. Deferred to August
10-12 (not August 15, to avoid colliding with NFL's own soft build
deadline), explicitly decoupled from NFL's own July 25 items, which
proceeded on schedule regardless.

NFL Item #9 (Elo variant selection) fully ratified through two rounds
with you: home_adv 48->20 (re-verified by re-running the 7/18 market-
beat test with the corrected constant, at your request -- confirmed
the fix works on its narrower claim without rescuing bare Elo's
broader market-beat failure), carryover/turnover tiered by continuity
33/33/40/50 (revised from a single-source citation after you flagged
it, reconciled against a second source that complicated rather than
confirmed the original), rookie QB prior -50 to -75 with a concrete,
checkable revisit trigger (first 10 rookie/first-year-starter QB
games in 2026) per your request for something pinned rather than
open-ended.

====================================================================
2. SATURDAY -- NFL ITEMS #10 AND #11, SEDE SIGNALS DIAGNOSED, MLS
   BACKFILL BUILT, SOCCER MODEL DIAGNOSTIC, ORACLE RECONCILIATION
====================================================================

NFL Item #10 (injury adjustment tiers) ratified through two rounds.
Round 1's above-average (+30) and below-average (-25) tiers rested on
a guessed percentage and a role-vs-performance category conflation
respectively -- you held both. Round 2 replaced the guess with a real
empirical ratio derived from actual 2024 ESPN QBR data (elite anchor
Josh Allen 77.6, average anchor ESPN's own 50.0 convention, a real
Stroud/Mayfield/Carr cluster at ~64.2 for the good-but-not-elite tier),
landing at +40. Below-average switched its anchor from "backup" (a
role) to QBR directly (sustained performance, the correct axis),
landing at -23 -- close to the original guess, reported honestly as a
real, undramatic result rather than only reporting when a number moves.

NFL Item #11 (weather adjustment weights) ratified through THREE
rounds -- the most rigorous of the six. Rather than cite secondary
blog sources for wind/temp effects, pulled and computed directly from
15 seasons of real nflverse/Pro-Football-Reference play-by-play and
schedule data. This genuinely overturned part of the original
proposal: the smooth "worse and worse as wind increases" story did
NOT hold past 20mph -- the 20-25mph bucket showed no reliable
suppression at all (era-normalized: +1.08, still above baseline),
contradicting the original secondary-sourced curve. Temperature's
milder tiers (25-50F, >85F) were also revised substantially down --
the real effects were roughly 1/3 to 1/2.5 the size secondary sources
claimed. Round 3 ran two methodological checks you specifically
requested (dome-game contamination of the baseline, era-normalization
of the wind reversal) -- both came back clean, confirming the
overturned findings were real, not artifacts of the analysis itself.

Investigated why SEDE Signals subscriber updates had gone silent for
17 days (last real update July 7). Different root cause than Friday's
Full AM/PM incident: a dedicated ~2AM CT scheduled task, whose sole
purpose was triggering this specific alert, had been silently DELETED
(not disabled -- confirmed via Task Scheduler query showing it simply
doesn't exist, where a disabled task would still appear) sometime
around July 7-8. Full AM (7:00 AM CT, unchanged since its April 30
creation) kept the rest of the pipeline running fine the whole time;
the alert's own hour-of-day check just never fired again. Fixed by
tying the trigger to Full AM's real, long-standing schedule instead of
recreating a task that already proved fragile once. Verified live with
a real send (Telegram + email both confirmed).

Separately, investigated why SEDE Signals had never actually traded
despite months of running. Found a real, distinct bug: MLB_GAME has
split suspension status (NO direction suspended, YES direction
reinstated as an "experimental tier" back in June) -- but the blanket
suspension check caught both directions regardless, since "MLB_GAME"
remained a key in the suspension dict. Confirmed a real casualty: a
July 8 YES signal (KC @ NYM, edge=36.1c, confidence=92.6%) cleared
every entry gate and should have traded, but was silently skipped.
Fixed; recomputing with the corrected logic showed exactly 1 signal in
the last 30 days would have genuinely qualified -- the same trade,
confirming the fix is precise, not overly permissive.

At Rus's request, investigated why SOCCER_GAME and MLS_GAME are
suspended. SOCCER_GAME's suspension note was ~2 months stale (still
said "25% accuracy, 1/4 games," with reassessment explicitly promised
at 20+ games) -- real current data showed 45 unique resolved markets,
more than double that threshold, never actually re-checked. Real
Gate 1 verdict run for real: n=45 (pass), Brier=0.169 (pass), win rate
40.0% (fail) -- suspension conclusion holds, but for the real current
reason, not the stale one. Also found "SOCCER_GAME" in the data is
actually World Cup matchups, not MLS -- the integrity doc's own title
was misleading. MLS_GAME had ZERO scored signals since its June 1
addition -- traced to a genuine infrastructure gap (no backfill
mechanism ever built for it, unlike MLB) rather than "not enough time."
Built mls_outcome_backfill.py, tested each component against real data
before running for real, then ran it live: 16 of 20 pending signals
filled immediately. Wired into the daily pipeline going forward.

Scoped expanding the soccer model to 8 major European leagues (EPL,
La Liga, Serie A, Bundesliga, Ligue 1, Primeira Liga, Eredivisie,
Scottish Premiership) at Rus's request -- confirmed real feasibility
for all 8 (Kalshi lists real markets, ESPN's API supports all 8 via
the same pattern already used for MLS, ClubElo already covers them).
But ran a real diagnostic on the EXISTING shared architecture first
rather than recommend scaling blind, and found genuine structural
problems: draws specifically broken (22% correct vs 44% for team-win
picks), a severe YES/NO asymmetry (25% vs 52% correct -- the same
shape of overconfidence pattern found in the NFL Elo work), and
inverted calibration (higher stated confidence performing worse, not
better). Recommended fixing the existing model before scaling it to 8
more leagues on the same foundation. Deferred per Rus's direction.

Session closed with a full, careful multi-file Oracle/laptop
reconciliation (17 conflicting files from a day of real, divergent
activity on both machines). Caught and corrected a real mistake in my
own first merge attempt for track_b_log.csv -- used the model's own
computed outputs (Elo ratings, divergence) as row identity instead of
real-world game identity (date + teams), which would have silently
near-duplicated the file. Caught by checking whether the merged
row count made real-world sense, not by trusting a script that ran
without error -- redone correctly, verified clean before committing.

====================================================================
3. SUNDAY -- NFL ITEMS #12, #13, #14: THE ARC CLOSES
====================================================================

NFL Item #12 (context_confidence formula) ratified through two
rounds. Built from scratch per the design doc's own instruction
(inheriting MLB's formula means inheriting an unresolved question
from a PENDING model). Honest framing throughout: unlike items #9-11,
this is an internal scoring-design question with no external ground
truth, and said so plainly. Your pressure-test found a real conflation
(weather-severity's stated rationale claimed to measure game variance
but actually measured estimate-confidence) and a new double-counting
risk (wind+precipitation stacking). Round 2 corrected the rationale
honestly, then checked directly whether nflverse carries a
precipitation field the way it carries wind/temp -- confirmed it does
NOT, meaning this specific stacking question genuinely cannot be
verified with data at hand, unlike item #11's real wind+temperature
check. Reported that limitation plainly rather than invent a discount
cap; kept the two penalties independently stackable, since item #11's
own precedent showed combined effects can be LARGER than naive
summing, not smaller. Your final read called this the strongest part
of the whole close-out -- the honest "we can't verify this yet"
answer, not a confident-sounding invention.

NFL Item #13 (secondary arbitrage layer) deferred -- a quick, clean
call once traced properly. This item's own contingency was already
written into the design doc before this weekend: "if the MLB Track A
verdict validates documented arbitrage empirically, a secondary layer
may be added post-launch." That verdict was itself deferred to Aug
10-12 on Friday. Not a new research question; a direct logical
consequence of Friday's own decision.

NFL Item #14 (version bump) done -- held deliberately until item #12
actually cleared review, so the bump reflects genuine completion, not
five-of-six-items-done. integrity_nfl_game.md is now at Version 1.0.
All six items (9-14) settled. Weeks 4-6 signal logic build (items
15-20: Elo implementation, injury adjustment, weather adjustment,
context_confidence scoring, 2024 backtest, Q1-Q4 re-confirmation) is
now unblocked to proceed against real, ratified parameters.

====================================================================
4. CARRIED, UPDATED
====================================================================
- NFL items #9-14: ALL RATIFIED/DEFERRED. Arc complete.
- NFL Weeks 4-6 build (items #15-20): not started, now unblocked.
- MLB Track A/B: deferred to August 10-12, unchanged.
- NFL Item #13: revisit when the MLB Track A verdict lands.
- Rookie QB prior trigger: first 10 rookie/first-year-starter QB
  games in 2026, unchanged, tracked.
- Wind+precipitation stacking: flagged for the standing verification
  check once real Weeks 1-4+ data exists.
- SOCCER_GAME root-cause diagnostic (draws, YES/NO asymmetry,
  calibration): surfaced Saturday, real findings, not yet started.
- 8-league soccer expansion: feasibility confirmed, blocked on the
  diagnostic above per Rus's direction.
- SEDE Signals: both real bugs fixed (17-day silence, missed-trade
  gate bug) and verified live.
- MLS_GAME: backfill built and running, real data now accumulating
  toward its first genuine Gate 1 verdict.

====================================================================
5. HARD DEADLINE STANDING
====================================================================
NFL: entire six-item architecture ratification complete. Weeks 4-6
build window (through Aug 11) now open, Sept 3 hard deadline
unaffected. MLB: Aug 10-12 decision date, unchanged.

====================================================================
6. TRADE STATUS
====================================================================
Unchanged across the whole weekend -- 2 OPEN (#8 Fed cuts ~11c vs 21c
entry, #13 GDP >2.0% ~54c vs 60c entry). Market reopens Monday.

Archie | Papa Ralph standard. If it's worth doing it's worth doing
right -- including reporting "we genuinely can't verify this yet" as
often as reporting a confirmed number, and catching mistakes in our
own merge scripts before they became real data corruption.
