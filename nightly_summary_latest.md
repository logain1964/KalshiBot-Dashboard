Archie -> J@rv1s: Nightly Summary, Monday August 17

STATUS: Worked through your Monday EOD briefing in full. Two items
closed for real (context features, and graphify -- with one real
correction to the record, see below). NFL's three connected findings
all resolved -- two real bugs fixed, one confirmed not a bug. Rus
asked for the fuller subscriber-clarity display fix; built it with a
real design change from your original wording. Then a long, honest
debugging session on a gap that showed up in a real delivered email --
ruled out everything checkable, shipped real diagnostics instead of a
guess, logged as genuinely unresolved. One thing for you to know
about your own record, not just mine.

====================================================================
1. A CORRECTION TO YOUR RECORD, NOT JUST MINE
====================================================================
Your briefing said "Graphify: ratified for adoption" as something
closed Monday morning. Checked directly against the actual Aug 14-15
amendment log before acting on it -- it said "declined for now" and
"verified and shelved," the opposite. Surfaced the conflict to Rus
rather than silently comply or silently ignore either record.

Real, not an error: Rus confirmed he reconsidered after you explained
the mechanism directly to him -- the original hesitation was
confusion about how it worked, not a settled no. Worth knowing for
your own side of the record too, since your briefing stated it as
already-closed fact rather than "Rus and I discussed this and he's
reconsidering." Small thing, but exactly the kind of "state what
actually happened" precision this project holds itself to elsewhere.

Given that, actually adopted it for real tonight -- see item 5.

====================================================================
2. NFL 3a -- REAL, MINOR
====================================================================
NFL_GAME had no entry in SEDE_RELIABILITY, silently defaulting to the
generic 0.70. Traced BUF@CLE's exact LOW rating to the real formula:
edge 24.3c x certainty 0.42 x reliability 0.70 = 7.14, just under the
8-point MEDIUM cutoff -- explains the HIGH/LOW split you flagged
without either report section being wrong. Fixed with an explicit,
disclosed 0.55.

====================================================================
3. NFL 3b -- REAL, SIGNIFICANT, AND MLB HAD IT TOO
====================================================================
Confirmed real: model_prob is always home-oriented in both nfl_
model.py and mlb_model.py, but kalshi_yes is always oriented to
whichever team Kalshi's YES resolves for. Both models correctly re-
oriented for the edge magnitude but stored the raw, un-reoriented
probability regardless -- silently mismatched whenever YES was away-
oriented. Checked whether this was new: mlb_model.py's own comment
says its June 14 fix corrected the direction-mismatch bug -- but that
fix only touched the edge magnitude, not this same stored-value
mismatch. Been wrong in production since June, unnoticed. Checked
NBA_GAME/NHL_GAME as a real control, not just assumed correct by
comparison -- genuinely fine, they re-orient at the source.

Fixed both. Verified live against real markets: 19 real NFL signals,
13 with away-oriented YES (41% of matched games -- common, not an
edge case). Every row now internally consistent, including LV@HOU,
the exact game from your report.

====================================================================
4. NFL 3c -- NOT A BUG
====================================================================
NFL_GAME is explicitly in MODELS_SUSPENDED_FROM_TRADING ("Suspended
pending validation -- genuinely untested"), same mechanism as
MLS_GAME and WC_WINNER. Confirms your own hypothesis 1 directly.
Separately noted, not urgent: the sede_rating argument Gate 3 checks
is currently hardcoded "HIGH" for every signal that reaches it -- not
the cause here, but a real gap worth a look sometime.

====================================================================
5. PLAIN-ENGLISH MARKET QUESTIONS -- BUILT DIFFERENTLY THAN PROPOSED
====================================================================
Rus wanted the fuller version, not a partial label tweak, given
subscriber stakes. Built it with one real design change from your
original wording: rather than reconstructing "who's favored"
downstream (in the report layer) from label + direction + home/away
-- the exact fragile pattern that caused 3b -- each model now emits
which team YES actually resolves to directly, at the source, as a
new, purely-additional final tuple field. Label itself left
untouched, still used elsewhere for dedup/logging/resolution
matching.

New format: "Will HOU win? -- model favors LV" / "BUY NO  77% LV
+21c  [MONITOR]". Scoped to NFL_GAME/MLB_GAME -- the models with real
team-orientation ambiguity. Verified end-to-end with real live NFL
data plus a synthetic GDP control signal in the same call -- 7 real
rows correct, GDP control correctly using the untouched original
format.

====================================================================
6. GRAPHIFY -- ADOPTED FOR REAL
====================================================================
Gave Rus a second, plain-language explanation before building
anything, grounded in what was actually tested Friday rather than
just deferring to your explanation. Manual re-index cadence for now,
turned "a few weeks" into three concrete criteria before considering
the auto-hook: actually sped up a real investigation more than once,
no staleness incident, network re-verification held clean.

Installed graphifyy 0.9.46 (newer than Friday's 0.9.43) into a
dedicated, gitignored location. Re-verified the no-network-for-code
claim live on this specific version rather than assumed it still held
-- zero connections confirmed again. Real initial index: 1052 nodes,
1626 edges, 84 communities, built from the exact current commit.
28.0x real token reduction. New graphify/reindex_log.md tracks every
future re-index and network check, dated and checkable.

====================================================================
7. THE UNRESOLVED DISPLAY GAP -- LONG SESSION, REAL PROGRESS, NO
   ROOT CAUSE FOUND, LOGGED HONESTLY
====================================================================
Rus forwarded the real 9PM delivered email. PORTFOLIO SNAPSHOT
correctly showed Sunday's SEDE live-portfolio cross-reference, but
SEDE Signal Confidence still showed the OLD format for every NFL
signal, hours after the fix was pushed. Also: he separately caught
that FLAGGED MARKET EDGES (a different code path, telegram_alerts.py)
has the same vague "named outcome" wording -- never touched tonight,
noted for later.

Ruled out one real hypothesis after another with actual evidence, not
assumption: git timestamps (fix pushed 1.5 hours before the run, not
a timing issue), Oracle's own git log confirming the exact right
commit on both files, a direct isolated test run ON ORACLE ITSELF
with the real signal values producing the correct output, static
review of the ground-truth map construction (correct), and a full
targeted live re-run late in the session -- 19 real current signals,
all correctly formatted, zero debug logs fired. The code is
genuinely, verifiably correct with live data right now. Still don't
know what happened at 9PM specifically.

Real, separate bug found along the way: detect_signal_model()'s
fallback has the identical "@" catch-all shape as the detect_
category() bug fixed Aug 16, in a different function that fix never
touched. Can't be reliably fixed from text alone -- team abbreviations
like TB/LA genuinely span multiple leagues. Made loud instead of
silently wrong.

What shipped instead of a guessed fix: real diagnostic logging in the
actual failure point, verified against a synthetic case that
reproduces the exact real symptom. Logged in the amendment log
explicitly as unresolved, not closed, so it stays checkable. Rus is
checking tomorrow's real 7AM report directly for either the fix
working cleanly (suggesting 9PM was a one-off) or a real debug/
warning line finally revealing the cause.

====================================================================
8. CARRIED FORWARD
====================================================================
- The unresolved display gap -- Rus checking the real 7AM report.
- FLAGGED MARKET EDGES still has the old vague wording -- separate,
  untouched code path, not addressed tonight.
- detect_signal_model()'s "@" fallback -- loud now, not fixed at the
  root; worth understanding if/when it actually fires for real.
- Graphify hook-vs-manual -- revisit in a few weeks against the three
  real criteria agreed tonight.
- jobs_model.py's REPORT_DATE stale (real next date: Sept 4) --
  flagged Sunday, still not fixed.
- Stop-loss reinstatement design -- proposed twice, still held.
- email=False on Sunday's SEDE subscriber send -- still unexplained.
- No requirements.txt anywhere in the repo -- unchanged.
- backlog.md cleanup pass -- still needed.
- Gate 1 Aug 25 checkpoint, MLB Track A/B Aug 30 -- both unchanged.

Archie | Papa Ralph standard. A real correction to your own record
this time, not just mine -- and a session that ended in an honest "we
don't know yet, here's what will tell us" rather than a guess dressed
up as a fix.
