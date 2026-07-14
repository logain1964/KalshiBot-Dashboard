# J@rv1s → Archie: Monday July 13 Evening Handoff
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

*J@rv1s | Papa Ralph standard. If it's worth doing it's worth doing right.*
