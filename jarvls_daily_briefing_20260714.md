# J@rv1s → Archie: Monday July 14 Evening Handoff
## STATUS: Deep pass on tonight's MLB Track A finding. One cheap diagnostic
## requested BEFORE any classification rewrite. Elo predictions logged for
## fast evaluation once your extended test lands. Nothing ratified, nothing
## built. For Rus's call.

---

## 1. PRIORITY ACTION — 10-MINUTE SPOT CHECK, BEFORE ANYTHING ELSE IN THIS DOC

**Do this first, before the governance discussion in Section 4 goes
anywhere:** manually pull 5-10 of the 103 signals and hand-check Kalshi's
actual settlement against what `model_prob` said AND which side got
scored. This is the cheapest possible diagnostic and it determines how
much of the rest of tonight's finding is real.

**Why this jumps the queue:** 0.2611 Brier (worse than random) on a
consensus-derived probability is a genuinely strange result — sharp
consensus lines are usually well-calibrated even on contested games, so
"scores like noise" doesn't sit comfortably with "the number being
scored is literally the sportsbook's own line." Given this exact
pipeline has already had two scoring-direction bugs this month (inverted
NO-branch sign; date-boundary mis-scoring), a flipped home/away or
YES/NO sign is now a live hypothesis, not a remote one, and it's cheaper
to rule out by hand than to reason about further.

If the spot-check finds a scoring bug: this becomes a bug fix, not a
provenance crisis, and Section 4's proposal below shrinks considerably.
If it finds clean scoring: Section 4's finding stands and is worth the
weight given to it. **Sequence matters — please run this before treating
the classification question as settled either way.**

---

## 2. SELECTION-ARTIFACT STRESS TEST — worked the math, three-way fork

Ran the actual numbers rather than accepting "selection artifact" as
asserted. Baseline coin-flip Brier is 0.25; observed is 0.2611 — worse
by 0.0111. At n=103, the standard error on a Brier score is roughly
0.03-0.04 depending on the prediction distribution, so **0.2611 is
statistically indistinguishable from pure noise around 0.25** — on
magnitude alone, "selection toward contested games, no real signal" is
defensible, not a stretch.

**But there's a real tension worth naming, not glossing over:** if
model_prob is genuinely SharpAPI/OddsAPI consensus, and sharp consensus
is usually well-calibrated even on close games, you'd expect Brier
meaningfully BETTER than 0.25 on a selected-for-contested subsample, not
statistically identical to a coin flip. That's the actual puzzle:
if Track A is just relaying sharp consensus, why does it score like
noise instead of scoring like a good sportsbook line should?

**Three-way fork, in order of how much weight I'd currently put on each:**
- (a) Confidence distribution IS clustered near 50% — selection worked
  as expected, even sharp books lack strong edges on Kalshi's specific
  contested-game selection. Boring, plausible explanation.
- (b) Scoring-direction bug (home/away or YES/NO reversal) making a
  genuinely good consensus line look bad. Weighted higher than I
  initially would, given this pipeline's track record this month.
- (c) The consensus data pulled for this subsample isn't actually
  reliable (stale lines, wrong market matched).

**The Section 1 spot-check is the fastest way to distinguish (b) from
(a)/(c).** Recommend running that before further analysis of what
0.2611 "means."

---

## 3. ELO CARRY-CONSTANT / EXTENDED-RANGE PREDICTIONS (for fast evaluation of your results)

Logging predictions now so we can evaluate your extended test quickly
against expectations rather than starting cold:

- **Shape:** the 1999-2024 correlation curve should still be
  monotonically increasing and roughly similar in shape to the 2021-2024
  curve. If it looks *substantially* different in shape (not just
  noisier), that's itself a flag — could mean era-level changes (16→17
  game season in 2021, rule changes affecting scoring variance) make
  recent-era-only data more relevant than the larger sample, not less.
- **Point estimate vs. confidence:** more seasons should tighten the
  confidence interval around the Week 6 crossing point, not necessarily
  move the point estimate much. Please report both the new point
  estimate AND how much it moved from the 4-season estimate — a crossing
  point that jumps to Week 3 or Week 9 is more surprising than one that
  holds ~Week 6 with a tighter band.
- **On WHY carry=0.50 gives lower Week 1 correlation than carry=0.33**
  (counter to naive expectation that more carried-forward info = more
  predictive): tentative mechanism — higher carry means clinging more to
  LAST season's final rating, which is less predictive of THIS season if
  there's been meaningful roster turnover (draft/free agency/coaching
  changes). Higher carry may be "more stale information," not "more
  information." If this mechanism is right, the gap between carry values
  should matter most early and shrink as the season's own results start
  dominating regardless of starting point — consistent with the ~Week 6
  crossing already holding across both carry values you tested. Flagging
  this so if the extended test shows something that contradicts this
  story, we know to ask why rather than just accepting a surprising
  number.

---

## 4. GOVERNANCE PROPOSAL — MLB_GAME classification (CONTINGENT ON SECTION 1)

**This section only applies if the Section 1 spot-check comes back
clean (i.e., confirms genuine market-relay, not a scoring bug).** If it
finds a flipped-sign bug instead, this becomes a much smaller finding —
fix the bug, re-score, move on — and the rest of this section can be
set aside.

**If confirmed clean, the issue is bigger than "wrong verdict metric for
July 25" — it's a provenance problem, not a performance problem, and
those should be classified differently even when they produce similar-
looking numbers.**

Proposed changes to the Foundation Document, as a standing rule (not
just an MLB-specific patch — same move as the win-rate definition and
NFL day-one-capture lesson: fix the category of mistake, not just the
instance):

1. **Re-label the existing 103-signal Track A history as "market-relay,
   not model output"** — kept as data, not deleted, but labeled honestly
   so it's never later read as evidence of genuine model performance.

2. **New standing Foundation Document requirement, applicable to any
   current or future model:** a model cannot be classified above
   "Pending" (cannot become Acceptable or Strong Node) until a
   provenance field (`prob_source` or equivalent) confirms a supermajor-
   ity of signals came from genuine model output, not a market-derived
   fallback. Turns tonight's specific discovery into a durable rule.

3. **MLB_GAME's honest current status is closer to "Unvalidated" than
   "Pending."** "Pending" implies a real test is underway with a known
   resolution date. What's actually true: the test meant to resolve
   Pending→decided (Track A) wasn't measuring the right thing, and the
   test that would (Track B) has ~no accumulated data yet. That's a
   materially more honest status than what the document currently shows.

---

## 5. CARRIED FROM YESTERDAY — STILL OPEN, UNCHANGED PRIORITY

- Two Elo checks (Section 3 above) — carried, first thing next session
  per your note, tier structure stays unlocked until both clear.
- `prob_source` persistence fix — endorsed, unblocks Section 1/2/4
  entirely. Also worth confirming whether the 103 signals are
  Oracle-generated, laptop-generated, or mixed (last-write-wins dedup
  makes a mixed-environment sample possible, which could itself
  introduce inconsistency on top of the source-fallback question).

---

## 6. CONFIRMED DONE, NO ACTION NEEDED

- Security rotation (GitHub PAT + SharpAPI key) — confirmed done,
  verified on both laptop and Oracle. Good, matches this morning's ask.
- NFL feasibility pass write-up — received, Q1/Q3 ratified separately.

---

## NET FOR RUS

1. Run the Section 1 spot-check first — cheapest, most load-bearing
   action available, determines how much of tonight's finding is real.
2. Section 2's three-way fork and Section 3's predictions are ready for
   fast evaluation whenever Archie's next results land.
3. Section 4's governance proposal is drafted and ready, but explicitly
   contingent on Section 1 — don't ratify classification changes before
   the spot-check runs.

Nothing built. Nothing ratified. For Rus's call.

---

J@rv1s → Archie: NFL Spec — Two New Feasibility Questions (Q4, Q5)
Monday July 14, 2026 (late session)
STATUS: Two additions surfaced from a broader architecture conversation with Rus tonight (MLOps/quant-finance pattern matching). Same treatment as Q1-Q3 — feasibility questions for you to test against real data, not decisions. Nothing built, nothing ratified.
CONTEXT (so you're not starting cold)
Rus and I spent part of tonight's session on a longer-term architecture idea — a shared "check engine" / inspection layer sitting between any model's output and eventual SEEKS integration. In working through it, we looked at what existing fields have already solved pieces of this problem: MLOps champion/challenger patterns, shadow deployment, and quant-finance backtesting methodology (specifically purged/embargoed cross-validation) and forecasting-calibration research (superforecasting/Good Judgment Project work).
Two of those map directly onto open pieces of the NFL spec and are worth testing before the spec locks, same as Q1-Q3. Framing them the same way: feasibility questions for you to check against real data, not conclusions already reached.
Q4 — REFRAME THE ELO CONFIDENCE TIERS AS AN EXPLICIT EMBARGO PERIOD
The pattern: in quant-finance backtesting (purged/embargoed cross-validation), when a model's inputs include information that summarizes a full prior period, an explicit "embargo" is required before trusting predictions in the new period — because the carried-over prior already encodes outcomes that haven't happened yet relative to the new test window.
Why this applies directly to what you already found: the Elo carry-constant (0.33 or 0.50, carrying forward last season's FINAL rating) is structurally an information-leakage source relative to the new season — last season's final rating already encodes that whole season's results. The tiered confidence structure you proposed (Wk 1-5 low, Wk 6-10 moderate, Wk 11+ high) is functionally an embargo period, just derived empirically rather than from the existing methodology.
Feasibility question: does treating this explicitly as an embargo-period problem (rather than a pure empirical convergence curve) change or sharpen where the boundary should sit? Specifically:

Does the purged-CV literature have a standard method for computing embargo length from a carry constant + season length, that could be compared against your empirical ~Week 6 finding as a cross-check?
Would explicit embargo framing suggest testing carry values BELOW 0.33 (less leakage, shorter earned embargo) as part of the same sweep you're already doing for the 1999-extension test?

This doesn't replace the empirical test already in progress (Week 6 crossing point, carry=0.50, 1999 extension) — it's a second lens to cross-check the same number against, not a different number to produce.
Q5 — PROBABILITY-BUCKET CALIBRATION, NOT JUST AGGREGATE BRIER
The pattern: superforecasting/forecasting-tournament methodology separates calibration (are stated probabilities honest) from resolution (how often you're right when confident), and checks calibration by BINNING predictions into probability ranges (e.g., all "~70% confident" signals) and testing whether reality matches the bucket (did the predicted side win ~70% of the time within that bucket specifically) — rather than relying on one aggregate Brier score across all signals.
Why this applies directly to a risk already surfaced this week: the MLB date-boundary Claim-2 investigation found that exposed signals skewed 2x more confident than clean ones (0.366 vs 0.166) — a subgroup-level miscalibration that an aggregate Brier score alone would not have surfaced without the specific confidence-distribution check we ran. A model can be well-calibrated on average while being badly miscalibrated within specific confidence bands or specific subgroups (e.g., systematically overconfident specifically on Questionable-tagged players) — and an aggregate score would hide exactly that.
Feasibility question: can probability-bucket calibration logging be built into Track B's day-one logging requirement (Section 2/3 of the spec) at reasonable cost? Specifically:

Is there a natural, clean bucket scheme given NFL's game count (fewer data points per season than MLB — buckets may need to be coarser, or accumulate across multiple seasons before being meaningful)?
Does this interact with the Section 4 Questionable/Doubtful handling — i.e., would bucketing by injury-designation subgroup (in addition to by confidence level) be worth logging from day one, given that's a plausible source of hidden miscalibration specific to NFL?

WHAT THIS IS NOT
Not a request to redesign the spec tonight. Not a claim that either of these changes the Elo Week-6 estimate or requires new infrastructure beyond what's already planned. These are two lenses borrowed from established fields that map onto problems already in front of us — worth testing for fit before the spec locks, same treatment as Q1-Q3.
REQUESTED FROM ARCHIE

Q4: sanity-check whether embargo-period framing changes or confirms the ~Week 6 empirical finding, using whatever cross-check is cheap given the 1999-extension work already planned.
Q5: assess whether probability-bucket + subgroup calibration logging is a reasonable day-one addition to Section 2/3, or adds meaningful build cost that should wait for a later revision.
Flag anything in either of these that doesn't fit NFL's actual data shape (sample size, season length, etc.) — these are patterns borrowed from other fields, not guaranteed to transfer cleanly.

Nothing built. Nothing ratified. For Rus's call once you've had a look.
J@rv1s | Papa Ralph standard. If it's worth doing it's worth doing right.

*J@rv1s | Papa Ralph standard. If it's worth doing it's worth doing right.*
