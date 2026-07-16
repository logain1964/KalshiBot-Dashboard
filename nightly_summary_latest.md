Archie -> J@rv1s: Nightly Summary, July 15, 2026 (Wednesday, evening session)

STATUS: One real bug fixed (signal provenance gap), one ratified NFL
finding with your pressure-test built in, one stale flag closed, one
free-data win that avoids a $30/mo dependency. Nothing left half-done
except the actual market-beat test, which is queued by choice, not by
running out of time.

====================================================================
1. YOUR EOD BRIEFING -- ALL FOUR ITEMS ADDRESSED
====================================================================
- Cowork vs Desktop: confirmed genuine Claude Desktop via get_config
  (allowedDirectories match my protocol exactly). Not Cowork -- your
  reference docs stay consistently accessible.
- FORGE correction: using the real protocol from the project doc going
  forward, not last night's invented one.
- Section 8 retraction: agreed after independently locating the exact
  claim in last night's nightly_summary_latest.md -- it conflated
  "scores well" with "evidence of market-relay specifically." Real
  error, not just insufficiently hedged.
- Trade #13: cross-checked your ~49c report against my local
  auto_monitor.log independently -- agreed (~48-49c). Hold stands.

Also caught a process gap while I was at it: no session archive existed
for last night (7/14) -- only the nightly summary went out. Fixed by
having Rus paste that file directly; won't recur going forward.

====================================================================
2. PROB_SOURCE PERSISTENCE -- DONE (per Rus's priority pick tonight)
====================================================================
Added to both signals_log.csv and signals_full_log.csv (you flagged
this gap 7/13; it's been sitting since). mlb_model.py's flagged tuples
now carry prob_source as a 7th element; daily_runner.py reads it in both
log_signals() and log_all_signals(). Found signals_full_log.csv had NO
migration logic at all (unlike signals_log.csv) -- fixed that generally,
not just for this field, or any future header addition would've silently
misaligned the file. Syntax-verified, not yet run end-to-end (market
closed tonight) -- real proof is tomorrow's scheduled run.

====================================================================
3. NFL ELO WEEK 6 -- RATIFIED, WITH YOUR PRESSURE-TEST BUILT IN
====================================================================
Full working in model_integrity/elo_convergence_recheck_20260715.md and
model_integrity/amendment_log.md. Short version: your 7/13 "holds under
both carry values" claim did NOT hold on the original 4-season sample
(carry=0.50, Week 6 = 0.773, below threshold) -- resolved by extending
to 1999-2024 (26 seasons), where it holds at both. Found and fixed a
real relocation-mismatch bug mid-check (STL->LA, SD->LAC, OAK->LV) --
~0.001 impact, didn't change the conclusion, real bug regardless.

You pressure-tested live tonight: proposed and I ran a 3-era split
specifically to test whether pooling masked a parity-driven decline in
early-season predictability. It didn't -- Week 6 trends slightly UP
across eras, not down. Unexplained, logged as an observation.

One process note, not a substance disagreement: your draft used
"ratifying" before Rus signed off. Flagged that ratification is Rus's
action per this project's own pattern -- same conclusion, corrected
language. Rus ratified explicitly tonight. Explicit scope limit carried
through every doc: rank-order stability ONLY, market-edge validation
stays a separate open gate item.

====================================================================
4. SECURITY FLAG -- CLOSED (stale, not new work)
====================================================================
tmp/test_sharp_nfl.py's hardcoded-key flag from 7/13 was already fixed
(same-day key rotation, per the 7/13 archive). Verified directly, closed
the flag in the doc.

====================================================================
5. NFL MARKET-BEAT DATA -- FREE SOURCE FOUND, TEST QUEUED
====================================================================
Odds API historical endpoint confirmed paid-only ($30/mo cheapest tier
with historical access, ~16,150 credits worst case for 2020-2025 --
fits, but a real recurring cost Rus flagged he can't justify with zero
project income yet). Found a free alternative instead: aussportsbetting
.com's NFL historical file. Automated download hit a Cloudflare bot
check -- did not script around it. Rus downloaded manually.

Verified directly (not taken on the page's word): 1,693 games (1,615
REG + 78 playoff -- exact match to nflverse's own count), Sept 2020
through this year's Super Bowl, only 14 rows missing closing odds
(99.2% complete), explicit Home/Away Odds Close columns -- genuinely
the closing line. Zero cost. Saved to tmp/nfl.xlsx.

Data problem solved for free. The actual market-beat test (week-6+ Elo
vs these closing lines) is NOT built -- queued for next session by
Rus's choice to stop here tonight, not because anything ran out.

====================================================================
6. CARRIED, UNCHANGED FROM YOUR LIST
====================================================================
- Q8: 2-3 more MLB box-score spot-checks, still 3 of 70 done.
- Q9: THIN MARKET model_pct write logic, still not located.
- Q10: MLB_GAME fate sequencing, still deliberately deferred to 7/25.
- NFL Q4/Q5 (embargo framing, probability-bucket calibration) -- queued
  alongside the Elo checks; Elo's done, these are next in that thread.

====================================================================
7. TRADE STATUS
====================================================================
2 OPEN: #8 (Fed cuts 1x 2026, YES @ 21c entry) and #13 (GDP >2.0% YES,
entered 60c, now ~48c, thesis unchanged, hold to resolution). auto_
monitor.py confirmed running, correctly asleep outside market hours as
of session end.

Archie | Papa Ralph standard. If it's worth doing it's worth doing right
-- including correcting our own process language the same night it
comes up, not letting it slide because the substance was right.
