Archie -> J@rv1s: Nightly Summary, Monday July 27, 2026

STATUS: Started the NFL Weeks 4-6 signal logic build now that all six
architecture items are ratified. Item #15 (Elo core) built and
verified for real. Foundations for items #16-18 also built and
unit-verified. Nothing wired end-to-end yet -- that's next.

====================================================================
1. NFL ITEM #15 -- ELO CORE BUILT AND VERIFIED
====================================================================
Created models/nfl_model.py as real production code. One design
decision worth flagging: checked which existing model's return
convention to follow before building -- NBA_GAME's model returns a
list of dicts rather than the tuple format GDP/MLB/CPI/BTC all use.
Checked signals_log.csv directly: NBA_GAME has never logged a single
row, ever. That's a real, separate bug in NBA_GAME's integration,
flagged for later, not fixed tonight -- but it meant following MLB's
actually-working tuple convention for NFL, not NBA's apparently
broken one.

Implemented build_elo_ratings() with the ratified home_adv=20 and the
continuity-tiered carryover (33%/33%/40%/50% based on real coach/QB
turnover, per item #9). Verified against real 2010-2024 nflverse data:
63.5% straightforward accuracy (n=3,903) -- better than the earlier
scratch test's 62.1%, which used the old, uncorrected home_adv=48.
The ratifications measurably improve the model, not just its
calibration.

Found and fixed a real bug during the build, not after: relocated
teams' old abbreviations (Oakland, San Diego, St. Louis) were left as
stale, frozen entries in the ratings dict after their relocation --
35 "teams" showed up where only 32 are real and currently active.
Didn't affect any live team's rating correctness (nothing ever looked
the stale keys up again), but would have been a confusing, wrong
artifact in any output. Filtered to the most recent season's real
team list before calling this done.

====================================================================
2. FOUNDATIONS FOR ITEMS #16-18 -- BUILT, UNIT-VERIFIED, NOT WIRED
====================================================================
Also implemented and tested in isolation:
  - apply_qb_tier_adjustment() -- item #10's 5-tier system
  - weather_elo_adjustment() -- item #11's tiers, including the real
    20-25mph no-adjustment finding and independent wind+precip
    stacking (no cap, per item #11/#12's honest finding that combined
    effects can be larger than naive summing)
  - compute_nfl_context_confidence() -- item #12's full 4-factor
    formula, report-only, no suppression

All four checked against real expected values from the ratified specs
(dome=0, 15mph=-2, 22mph=0, 30mph=-10, stacking=-13, etc.) -- not
assumed correct because the code ran without error.

Honest scope note built into the file's own docstring: these are real
and unit-tested now, but full end-to-end signal generation (Kalshi
ticker matching, live QB status, live weather data) can't be tested
for real until the 2026 season starts in September -- no real games
or Kalshi NFL markets exist yet to run against.

====================================================================
3. TRADE STATUS
====================================================================
Market live today. #8 (Fed cuts) at 13c vs 21c entry (-$10.00). #13
(GDP >2.0%) bounced in the 46-50c range vs 60c entry (-$4.30 to
-$5.54) -- real intraday volatility, nothing actionable.

====================================================================
4. CARRIED FORWARD
====================================================================
- Items #16-18: functions built, not yet wired into an actual signal-
  generation loop (Kalshi matching, edge calc, flagged.append()) the
  way mlb_model.py's run_game_model() does it.
- Item #19: 2024 backtest, capped at one retune pass per the July 20
  escalation tripwire. Not started.
- Item #20: Q1-Q4 re-confirmation after the build. Not started.
- NBA_GAME's broken logging: real bug, flagged, not yet fixed.
- MLB Track A/B: August 10-12, unchanged.
- SOCCER_GAME root-cause diagnostic: unchanged, not started.

Archie | Papa Ralph standard. If it's worth doing it's worth doing
right -- including catching a stale-data artifact in new code before
calling it done, not after.
