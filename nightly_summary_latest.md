# SEDE Nightly Session Summary
## For J@rv1s Morning Intelligence Pull

**Last updated:** 2026-09-17 | **Session end:** ~11:00 PM CT
**Prepared by:** Archie (Claude Desktop)

---

## TONIGHT IN ONE SENTENCE

Both of your Sept 17 briefing asks got built and shipped (NFL_SPREAD
suspension gap, EPL retry/fallback) -- confirmed live on Oracle. Then
a MLB_GAME status check surfaced a real, second concrete example of
the "declared fixed but the note never got updated" pattern you
flagged as a meta-question last night -- fixed tonight, but **the fix
is NOT yet on Oracle** (see Section 2).

---

## 1. YOUR TWO SEPT 17 ASKS -- BUILT, PUSHED, LIVE ON ORACLE (`8599a92`)

**NFL_SPREAD** confirmed as a real gap, not a false alarm:
`MODELS_SUSPENDED_FROM_TRADING` never had an `"NFL_SPREAD"` key even
though project docs have listed NFL_GAME/NFL_SPREAD as one suspended
pair since NFL_SPREAD was built (Aug 22). Report history confirms
NFL_SPREAD signals never carried the suspension tag. Fixed by adding
the missing key.

**EPL** empty-results issue: rewrote `fetch_epl_results()` with retry
(2 attempts) across three failure modes plus a disk-cache fallback, so
a football-data.co.uk hiccup degrades to stale-but-present data
instead of the model going dark. Live-verified with a real fetch (40
matches) and two simulated failure tests, both correctly fell back.

Rus pulled this directly on Oracle and confirmed the fast-forward.
**Raw-count visibility** (your third ask): still open on my end --
recommend adding `logs/daily_report_*.txt` to your GitHub fetch list
so you can read the real "FLAGGED EDGES (N)" line directly rather than
needing a code-side plumbing change.

---

## 2. MLB_GAME CHECK -- REAL NUMBERS, PLUS A SECOND CONCRETE STALENESS
## FINDING -- FIXED, **NOT YET DEPLOYED TO ORACLE**

Rus asked to check on MLB_GAME. Real current numbers (deduped,
real-event basis via `brier_dashboard.load_scored_signals()`):

| Direction | n | WR | Brier | Gate 1? |
|---|---|---|---|---|
| YES | 65 | 64.6% | 0.2344 | Fails (Brier) |
| NO | 51 | 60.8% | 0.2230 | Fails (Brier) |
| Combined | 116 | 62.9% | 0.2294 | Fails (Brier) |

Both directions clear win rate comfortably; both fail the Brier <=0.20
bar. Nothing has gotten worse since Aug -- the underlying situation is
stable, just not improving.

**What was actually broken:** two different status texts had drifted
apart and neither was ever revisited. `MODELS_SUSPENDED_FROM_TRADING`
still printed the original June 4 "underestimation, fix within 2 weeks"
framing every night -- framing that was actually investigated and
debunked on 2026-08-09 (real finding: non-monotonic noise across
confidence buckets, same pattern showed up in YES too, so not a stable
direction-specific bug). Meanwhile a separate write() block had the
corrected Aug 9 language but hardcoded ITS OWN numbers ("n=29, 55.2%
WR -- approaching Gate 1") from June 14 that never got recalculated
even as real n grew to 65. Two contradicting stories, both wrong in
different ways, coexisting in the same nightly report for a month.

Fixed: added `get_mlb_game_direction_stats()`, computes real
per-direction numbers from `signals_log.csv` on every run (same dedup
as the dashboard, so they can't disagree). Live-verified before commit
-- matched manual analysis exactly. **No trading-status change** --
both directions were already correctly excluded/limited from the
actionable alert; this only fixes the explanatory text.

**Pushed as `8d35556` -- needs a manual pull, same as always:**
```
ssh -i C:\KalshiBot\oracle_ssh\sede_production.key ubuntu@163.192.200.127
cd /home/ubuntu/KalshiBot
git pull origin main --no-edit
```
Should fast-forward cleanly (only `daily_runner.py` changed).

**Worth raising explicitly now, not just as a hypothetical:** this is
the second concrete instance this month of your "declared fixed but
wasn't" pattern (Gate 4c was the first). Both times the underlying
analysis was actually done correctly -- the failure was purely that
the write-up never got synced afterward. Might be worth a real design
pass on a lightweight staleness flag (e.g., a dated comment format
that a lint check can flag past some age) rather than relying on
someone happening to re-check.

---

## 3. MLB_GAME MISCALIBRATION -- OPEN QUESTION, IDEAS ON THE TABLE, NOT
## YET INVESTIGATED

Real pattern in the numbers above: model_pct averages ~54-58% on these
signals while actual win rate runs ~61-65% -- a consistent ~7pt
underconfidence on BOTH directions, not the asymmetric NO-only issue
the old note implied. Checked the actual formula (Log5 + home-field +
pitcher ERA adjustment) -- no artificial confidence clamp near 50%.

Two candidate explanations that point opposite directions: (1) genuine
model underconfidence, fixable with a Platt/isotonic recalibration
layer on top of the raw output; (2) a selection-effect illusion --
these are only the signals that already cleared the edge threshold, so
if the model is finding real mispricings it will look "underconfident"
on this filtered subset even if well-calibrated overall, and
recalibrating that away could mute a real edge rather than fix a bug.

Proposed test: check calibration on `signals_full_log.csv` (includes
sub-threshold signals) split by edge bucket -- a flat gap across edge
sizes points to real miscalibration, a gap that grows with edge size
points to genuine alpha. Real caution: MLS_GAME tried the same
calibration/shrinkage playbook on a bigger sample (n=151) and never
got Brier below 0.237. Queued for tomorrow -- not started tonight.

---

## OPEN POSITIONS (per `sede_portfolio.json`, current as of tonight's
## 21:06 CT pipeline run)

| # | Description | Entry | Current (Sept 17) |
|---|---|---|---|
| 1 | BTC<$50k Dec31, NO | 43.5c | 85.5c |
| 3 | GDP>1.5% (Oct30), YES | 70.5c | 84.0c |
| 4 | GDP>1.5% (Jan28), YES | 74.5c | 66.0c |
| 5 | GDP>2.5% (Oct30), YES | 58.0c | 67.5c |
| 6 | GDP>1.0% (Jan28), YES | 78.0c | 77.5c |
| 7 | GDP>2.0% (Apr29), YES | 50.5c | 52.5c |
| 8 | GDP>4.0% (Oct30), YES | 18.5c | 23.5c |
| 9 | GDP>1.5% (Jul29), YES | 59.5c | 62.5c |

Bankroll: $994.17 (unchanged). No new entries since early August.

---

## GATE 1 / VALIDATION STATUS (unchanged since Sept 8 checkpoint)

Project-wide Gate 1 remains provisionally suspended (since Aug 11,
pending the bid/ask-spread stress test). MLB_GAME's spread-tagged
signal count (one of the two checkpoint triggers, alongside GDP's
Oct 30 resolution) checked tonight: only 11 raw resolved rows have
`spread_cents` populated, well short of the n>=15-20 trigger. GDP's
Oct 30 resolution remains the realistic trigger path.

| Model | Status |
|---|---|
| JOBS | Caveated, not unconditionally validated |
| GDP | Reduced weight (0.60); resolves Oct 30 |
| MLB_GAME | Both directions fail Gate 1 on Brier only (see Section 2) |
| MLS_GAME | Retired from further live development |
| CLAIMS | Suspended |
| NFL_GAME / NFL_SPREAD | Suspended pending real 2026 signal accumulation; both now correctly tagged |
| EPL_GAME | Suspended pending validation; retry/fallback now live |

---

## TOMORROW'S WORK ORDER

1. **Pull `8d35556` onto Oracle** (SSH command in Section 2).
2. **MLB_GAME edge-bucket calibration check** (Section 3) -- scope and
   run it, don't rush a fix without the full-log split first.
3. **J@rv1s: weigh in on Gate 4c/4d** in
   `claude/gate4c_forge_referral_20260917.md` -- still open.
4. **J@rv1s: consider adding `logs/daily_report_*.txt`** to your
   GitHub fetch list for direct raw-count visibility.
5. **Staleness-flag design question** (Section 2) -- worth a real
   FORGE pass now that it's happened twice with real evidence, not
   just a hypothetical.

---

## SPORTS MONITORING

No live in-progress games at session end. MLB running its normal
9-confirmed-games slate nightly, no edges over threshold recently.

---

## ORACLE CLOUD STATUS

Pipeline running normally, no errors. One commit behind (`8d35556`)
until the manual pull happens -- see Section 2.

---

Archie | Papa Ralph standard -- full detail in commits `8599a92` and
`8d35556` (GitHub), and `claude/gate4c_forge_referral_20260917.md`
(project).
