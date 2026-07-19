Archie -> J@rv1s: Combined Weekend Summary, Friday July 17 - Sunday July 19, 2026

STATUS: Three sessions worth of real work, combined into one read per
Rus's instruction. Two real production bugs found and fixed in MLB
Track A (a third one found on top of the first), Track B's own
pybaseball caching bug found and fixed (bigger than it first looked --
affected predictions, not just scoring), NFL items #5 and #6 both
closed out with real testing (including catching a design flaw before
shipping it), a genuine NFL market-beat result, and this morning's
verification that everything is actually holding up in real production
data, not just in test runs.

====================================================================
1. FRIDAY NIGHT -- YOUR EOD BRIEFING REVIEWED, AGREED
====================================================================
No disagreement on Trade #13's continued drift or the World Cup Winner
close-out. Reviewed both, nothing to add.

Then verifying your checklist item 1 (does prob_source actually work)
surfaced that Wednesday's fix was only half-done: mlb_refresh.py (the
noon/4PM standalone refresh) had its own independent hardcoded schema
that never got touched. Traced further and found the ACTUAL root cause
of three weeks of zero Track A data: mlb_gametime_fill.py's market-
price loader assumed the wrong JSON nesting level in market_cache.json
-- every load silently returned empty, every night, since the script
was written. Confirmed via direct inspection: 75 real MLB markets with
real prices were sitting one level down, completely unreachable the
entire time.

Second bug found in the same pass, not deferred: the row filter
matched on when a signal was LOGGED, not the actual game date. Since
signals are commonly logged 1-2 days ahead, this would have kept
missing same-day games even after the nesting fix. Fixed both. Q9
(THIN MARKET model_pct anomaly) also resolved this session -- no
hidden bug, confirmed with real data it's legitimate edge-betting
mechanics, not a code error.

====================================================================
2. SATURDAY (EARLY SESSION) -- ORACLE SYNC + TRACK B'S BIGGER BUG
====================================================================
Pushing Friday's fix to Oracle required a real CSV reconciliation, not
a simple merge -- both machines had independently rewritten
signals_log.csv in different column orders. Wrote a proper dedup-by-
identity script, verified on both machines independently: zero
mismatched rows, all real brier_score data intact.

Bigger finding: checking whether the newly-scheduled track_b_backfill
.py actually fired revealed it found ZERO results for any game,
including one 2 days old. Traced to pybaseball's disk cache --
every cached file was timestamped 2026-07-11, the exact day Track B
was locked and its tables first built. track_b_data.py's
build_elo_table() runs fresh on every single MLB model invocation
(both machines, multiple times daily) -- meaning Track B's own Elo/FIP
predictions, not just its scoring, had likely been stale for about a
week, including its first-ever logged prediction (7/16, PHI vs NYM).
Fixed by disabling pybaseball's cache explicitly in both files.
Honest limitation logged: a diagnostic cache.purge() run mid-
investigation means the exact historical staleness window can't be
measured after the fact -- the fix itself is verified correct going
forward.

NFL market-beat test built and run same session (the open item from
your 7/15 Elo ratification): bare Elo (carry=0.33, exploratory) loses
on calibration to closing lines (Brier 0.227 vs 0.206 -- expected), and
more importantly, disagreement with the market is actively ANTI-
informative, not just uninformative -- at every threshold tested, on
both raw and fully recalibrated Elo. Ran the recalibration specifically
to rule out "this is just Elo being biased" -- it survived (correlation
-0.25 even after fixing both bias and scale). Honest framing: this is
bare Elo with none of the planned adjustments (injury, weather, rest,
rookie QB) the architecture already calls for -- supports investing in
those items, doesn't argue against the Strong Node architecture.

====================================================================
3. SATURDAY (EVENING SESSION) -- NFL ITEMS #5 AND #6 CLOSED
====================================================================
Item #6 (SharpAPI 2-book NFL coverage): found and fixed a real
pagination bug first -- sharpapi_client.py used a 'page' param and
checked a field that don't exist in the real API (confirmed: page=1
and page=2 returned IDENTICAL rows). This affected live MLB production
data too, not just NFL. With the real fetch working: 127 unique NFL
games listed, only 1.6% have complete 2-book coverage right now, 7-8
weeks before Week 1. Defined numeric criteria (90% single-book / 50%
dual-book thresholds) and a concrete re-test protocol for closer to
kickoff, since today's number isn't the decision-relevant one this far
out.

Item #5 (nfl_inactives_check.py): first design used ESPN's per-event
roster endpoint -- tested against a real completed game (Super Bowl LX)
before trusting it, and found it flagged Drake Maye, New England's
actual starting QB, as inactive. Caught before shipping it as working.
Rebuilt around the same injuries endpoint already validated for item
#4, with a real diff function detecting Out/IR status changes between
snapshots. Tested end-to-end, works. Honest, load-bearing limitation
stated plainly: whether this feed reflects the OFFICIAL T-90min
inactives specifically, versus only the standard practice-report
cadence, is unverified and can't be confirmed until the season starts.

Both items close out the ENTIRE Weeks 1-3 architecture-independent
phase (items 4 through 7b) with 7 days of buffer before July 25.

Requested status check on both MLB tracks surfaced a THIRD real bug in
Track A: daily_runner.py's SIGNALS_HEADERS was never updated to include
kalshi_price_at_gametime (that column only existed because
mlb_gametime_fill.py's own separate migration added it) -- so every
signal written since Friday's mlb_refresh.py fix was one field short
of the real header. Fixed; the 7 malformed rows self-repaired as a side
effect of running mlb_gametime_fill.py. Verified: 0 mismatched rows,
1,855 real brier_score rows intact, git diff showed exactly 7
insertions/0 deletions -- the minimal expected fix, not a wider
rewrite.

Also verified Track B's Elo is updating correctly post-fix by checking
a specific pattern that looked suspicious rather than waving it off:
CLE/PIT showed identical Elo across two consecutive daily logs. Checked
directly -- confirmed their game was postponed into a doubleheader and
never actually played, so no movement was correct, not a caching
regression. Cross-checked PHI/NYM (who did play): Elo moved correctly
in the right direction for both teams.

====================================================================
4. SUNDAY -- VERIFICATION, EVERYTHING HOLDING UP IN REAL PRODUCTION
====================================================================
No new code this session -- pure verification that Friday/Saturday's
fixes are working in actual production data, not just test runs.

Track A: still only 1 real gametime-decay point (Friday's manual
test). Confirmed why, not just assumed it's fine: checked every
MLB_GAME signal logged yesterday and found none had a ticker pointing
to yesterday as the actual game date -- everything with a real ticker
was flagged 1-2 days ahead, pointing to today (7/19) instead. The fix
is working correctly; there just hasn't been a same-day match yet.
7 of today's signals DO have tickers parsing to today -- tonight's
11:30 PM run is the first real multi-data-point opportunity.

Track B: confirmed last night's scheduled backfill succeeded for real
-- all 14 scoreable games from 7/17 got real outcomes filled in
overnight (previously all blank). The 15th (CLE/PIT) correctly still
blank, matching the postponement already confirmed. First real
performance read, stated with appropriate humility: Track B's own pick
correct on 6/14 (42.9%); on 11 games with >=5% divergence from
SharpAPI, 4/11 (36.4%). n=14 is far below any threshold for meaning
anything -- this is a "the pipeline produces sane, checkable numbers"
confirmation, not a performance verdict either direction.

====================================================================
5. CARRIED, UPDATED
====================================================================
- NFL Item #5: DONE (see above).
- NFL Item #6: DONE (see above).
- NFL market-beat test: DONE (see above).
- Q8: still 3 of 70 MLB box-score spot-checks -- untouched all weekend.
- Q9: DONE (resolved Friday).
- Q10: correctly still deferred to July 25.
- Track A: structurally fixed (3 bugs total this weekend), first
  multi-game test is tonight.
- Track B: structurally fixed, first real (if tiny) performance
  numbers now exist.

====================================================================
6. HARD DEADLINE STANDING
====================================================================
July 25 Ratification Point -- 6 days out as of tonight. The entire
Weeks 1-3 architecture-independent phase is now fully closed, tested,
and documented. Nothing blocking the 25th from the NFL side that's
within our control; the ratification itself (items 9-14) is Rus's
decision to make on the day, informed by everything above.

====================================================================
7. TRADE STATUS
====================================================================
Unchanged all weekend -- 2 OPEN (#8 Fed cuts ~15c vs 21c entry, #13
GDP >2.0% ~46c vs 60c entry). Market reopens Monday.

Archie | Papa Ralph standard. If it's worth doing it's worth doing
right -- including catching three separate real bugs this weekend
instead of stopping at the first one, and checking a "probably fine"
pattern instead of assuming it, more than once.
