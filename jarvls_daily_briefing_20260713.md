Jarvis handoff 2026 07 13 · MDJ@rv1s → Archie: Monday July 13 Handoff

STATUS: NFL Signal Contract Spec v1 DRAFT ready for feasibility pass.

Daily report verified — win-rate fix confirmed LIVE. One Trade #13

observation flagged, not urgent. Nothing ratified, nothing built.


1. NFL SIGNAL CONTRACT SPEC v1 — DRAFT COMPLETE, NEEDS YOUR FEASIBILITY PASS

Full draft attached/pasted below. Built per Rus's instruction: measurement
infrastructure spec FIRST, before any model code, directly applying the
MLB Track A/B lesson (infra retrofitted under deadline pressure vs. locked
in from day one).

Process used: Ran FORGE/BIAS/Papa Ralph on the original 5-point list
before drafting. Two of five points were revised as a result (dedup key
changed from date-based to game_id-based; independence verification
promoted from an implied assumption to its own explicit section), and one
new point added (injury-report cadence as a first-class capture
requirement) based on structural differences between NFL and MLB that
have no direct precedent in the MLB build.

Verified via web search (7/13/2026) before locking into the spec:


Injury report cadence: Practice Reports (DNP/Limited/Full) filed
Wed/Thu/Fri for Sunday games; final Game Status Report (Out/Doubtful/
Questionable) due 4PM ET Friday for Sunday games — two days before
kickoff, compressed for Thursday games.
Correction to verify against your data source: "Probable" was
retired as a designation in 2016. Current designations are ONLY Out/
Doubtful/Questionable. If any historical dataset you're planning to
pull from still carries "Probable" as a current field, that's stale —
please confirm your intended source (Pro-Football-Reference or
similar) doesn't carry it before we build against it.
Historical participation-rate context: Questionable ~50% play rate,
Doubtful ~25% play rate (informal historical tracking, NOT an official
league-published conversion — team discretion governs the actual
designation). Useful as a model prior, not ground truth.


What I need from you specifically (Section 9 of the spec):


Confirm your intended injury-report/practice-participation data
source is clean of the stale "Probable" field and has usable
historical designation data.
Run the Elo-convergence testing to set the actual minimum-games
threshold in Section 2 — I proposed Weeks 1-4 as provisional but
that's an estimate, not a tested number.
Feasibility check on the Independence Verification section (Section
3) — specifically whether a game_id-based schedule source is
available that survives flex-scheduling changes cleanly.


Full spec content:


NFL Signal Contract — Specification v1 (DRAFT)

0. Why this exists: MLB_GAME's Track A/B validation couldn't run
because measurement infra was assumed to exist and wasn't built. Lesson:
measurement infrastructure is a day-one requirement, not a phase-2 add.
This spec is the frozen contract to build NFL_GAME against — nothing
implemented yet.

1. Signal-time capture: Every signal logs Kalshi price at signal
time (baseline) PLUS which injury-report stage was current at that
moment (not just a raw timestamp — a Tuesday signal and a Friday-4:05PM
signal for the same game carry fundamentally different information) PLUS
days-of-rest for both teams.

2. Independent model probability: Every signal logs a non-sportsbook-
derived model probability alongside the SharpAPI-derived one, from
signal #1 — no "add Track B later" deferral. NFL-specific sample-size
caveat: ~17 games/season per team vs MLB's 162 means an Elo rating needs
an explicit minimum-games-before-trusted threshold (proposed default:
Weeks 1-4 provisional/wide-uncertainty — needs your convergence testing
to set the real number).

3. Independence Verification (new section, core anti-arbitrage check):
Log divergence = model_prob - sharpbook_prob as a first-class field for
every game from game one. Pre-commit BEFORE seeing results to: (a) what
fraction of games should show >X-point divergence to count as genuinely
independent, (b) whether the model is more accurate than the sportsbook
specifically on divergent games. No sportsbook-derived data as a model
input, anywhere — a "predictive" model with line-derived features is
arbitrage wearing a prediction costume.

4. Injury report system (verified structure, see corrections above):
Practice Report cadence Wed/Thu/Fri for Sunday games; Game Status Report
(final designation) due Friday 4PM ET. Designations: Out/Doubtful/
Questionable only (Probable retired 2016). Historical participation:
Questionable ~50%, Doubtful ~25% — treat as three distinct probability
bands, not a binary, and as a prior not ground truth (no official
practice-to-designation formula exists; team discretion).

5. Schedule-type handling: Tag every game by schedule type. Thursday
games: compressed cycle, final report Wednesday not Friday. Monday games:
Friday's report is NOT final for Monday games — do not treat it as
complete information, a Saturday/Sunday update can still move the line
(same risk shape as the UTC/CST bug — an intermediate report mistaken for
final, via a different mechanism). Log rest-days-since-last-game for both
teams on every signal.

6. Uniqueness/dedup key: Use a stable game_id (not a
date+matchup reconstruction) — flex scheduling can move a game's date/
time, and a date-based key would break the same way MLB's market-only
key did, just via a different cause.

7. Timezone handling: CST-aware datetime from line one
(datetime.now(CENTRAL) pattern) — build it in from the start rather
than retrofitting after a live pipeline exists, given the cost of the
MLB UTC/CST bug.

8. Win/loss/early-exit schema: Use the already-ratified hybrid from
day one (pnl>0 gate, resolution-only companion, DISCRETIONARY_EXIT/
BUG_INDUCED_EXIT split) — no separate reconciliation project needed in
September.

9. Open items needing your feasibility pass: (see "what I need from
you" above — data source cleanliness, Elo convergence threshold,
game_id availability).

10. Explicitly out of scope for this doc: model architecture,
feature engineering, bet-sizing, market selection. This is the
measurement-and-independence contract only. Model-building shouldn't
start until this is ratified.


2. TODAY'S DAILY REPORT — VERIFIED, WIN-RATE FIX CONFIRMED LIVE

Cross-checked Monday July 13, 7AM CT report against the weekend's fixes:


Win-rate schema fix confirmed deployed correctly, not just
committed: report now shows 22 closed / 10 wins (was 18/10 before the
fix). Total P&L moved from +$139.14 to +$135.14 — exactly a $4 drop,
consistent with including trades #17-20 (four $1 real-money MLB losses
previously excluded by the schema bug). Math reconciles cleanly. Good
confirmation the fix produces correct output in a live report, not just
in your test run.
Trade #13 — small divergence worth noting, not urgent: GDPNow
unchanged at 1.2603% (expected, next real update July 16), and the
signal-confidence table still shows GDP >2.0% at only 27% model
probability — thesis still reads as decayed. But market price on the
position moved TOWARD entry (54c Jul 8 → 58c today, vs 60c entry) —
getting less bad, not more, despite no improvement in the model's own
read. Could be short-term noise or something the market's pricing in
ahead of Wednesday. No action needed given "hold to resolution" is
already ratified — flagging so it's not a surprise if it reverses after
the July 16 GDPNow update.
Trade #8 continuing its slow drift (-7c today, -6c Sunday) — still
nothing urgent, 171 days left.



J@rv1s | Monday July 13 session complete
Papa Ralph standard. If it's worth doing it's worth doing right.
