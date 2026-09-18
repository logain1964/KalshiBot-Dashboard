# J@rv1s Daily Briefing — 2026-09-18

## ACTION NEEDED — PULL `8d35556` ONTO ORACLE

Same as always: MLB_GAME's status-text fix (correct real per-direction
numbers computed live from `signals_log.csv` instead of two stale,
contradicting hardcoded stories) does nothing until this runs:
```
ssh -i C:\KalshiBot\oracle_ssh\sede_production.key ubuntu@163.192.200.127
cd /home/ubuntu/KalshiBot
git pull origin main --no-edit
```

## CONFIRMED WORKING — NFL_GAME/NFL_SPREAD SUSPENSION TAG, TODAY'S 7:03 AM REPORT

Today's report shows zero NFL rows anywhere — first report since the
Sept 17 fix (`8599a92`, adding the missing `NFL_SPREAD` key to
`MODELS_SUSPENDED_FROM_TRADING`) where NFL_GAME is actually absent.
Direct, positive confirmation that fix is live and working, distinct
from and not to be confused with `8d35556` above, which is a separate
commit still awaiting its own pull.

**Small, low-priority inconsistency worth a quick look, not urgent:**
today's SEDE Signal Confidence section still showed one NFL_SPREAD-
shaped signal (`TB >13.5 [THIN MARKET]`) even though the main FLAGGED
MARKET EDGES section had zero NFL content. Possibly that section pulls
from a slightly different, not-yet-filtered source, or just residual
timing from an early-morning report cycle. Worth a quick confirmation
whenever convenient.

## MLB_GAME MISCALIBRATION — REAL PUSHBACK ON THE SELECTION-EFFECT FRAMING BEFORE THE EDGE-BUCKET TEST RUNS

Real, consistent ~7pt underconfidence on both directions (model_pct
~54-58% vs. actual win rate ~61-65%) — genuinely interesting pattern,
and the proposed edge-bucket test against `signals_full_log.csv` is
the right move. One thing worth flagging before it runs: a pure
selection-effect explanation (filtering for cases where the model's
estimate diverges most from the market) should, if anything, produce
*overconfidence* on the selected subset via a winner's-curse-style
mechanism — not underconfidence. The observed direction argues against
the selection-illusion story as originally framed, though a different,
less obvious selection mechanism could still be at play.

Watch for a specific shape in the edge-bucket results, since each
implies a different real conclusion:
- **Flat underconfidence gap across all edge sizes** → genuine,
  uniform miscalibration — a real recalibration layer may be
  warranted.
- **Gap that grows with edge size** → the model may be correctly
  finding real, bigger mispricings while understating its own
  confidence proportionally — recalibrating this away could mute
  genuine alpha rather than fix a bug.
- **Gap that shrinks or flips at the largest edges** → would actually
  support the original selection-illusion story, specifically at the
  extremes.

Real, relevant caution carried forward from precedent: MLS_GAME tried
the same calibration/shrinkage playbook on a bigger sample (n=151) and
never got Brier below 0.237 — go in with tempered expectations, not an
assumption that recalibration is a clean fix once the pattern's
identified.

## MONITORING/ALERTING SYSTEM — FULL FORGE COMPLETE, REAL WORK ORDER BELOW

Ran a proper FORGE pass on a lightweight pipeline monitoring/alerting
system, using an independently-sourced design (verified: Healthchecks.io's
free tier is real, current, stable pricing history, sufficient at 20
monitors) as the base material, then pressure-tested it directly.

**The core finding, worth stating plainly:** the base design treated
the pipeline as one thing that either succeeds or fails — one exit
code, one state machine. But every real incident this month that
actually cost time (ESPN's block, EPL's empty results, the cron
file-discard bug) was a case where the overall script kept running
and exiting cleanly while one specific component silently degraded
inside it. A whole-pipeline-only design would have caught none of
those. The build needs per-component tracking for the specific
components with a demonstrated history of this failure mode, not just
whole-pipeline liveness.

**Staged build plan, in sequence:**

**Stage 1 — build first:**
1. External heartbeat (Healthchecks.io free tier), pinged only at the
   very end of a fully successful pipeline run.
2. Whole-pipeline state-transition alerting (alert on OK→FAILING and
   FAILING→OK only, silent in between) via the existing Telegram bot
   integration.
3. A genuinely short daily digest (a handful of lines, not a wall of
   text) — run counts, row volumes, nothing that becomes its own
   ignored noise.

**Stage 2 — scoped now, built once Stage 1 is live:**
4. Per-component execution manifest, starting specifically with
   NFL_GAME, EPL_GAME, and MLS_GAME — the three components with an
   actual, demonstrated history of silent degradation this month —
   not uniform tracking across every model including stable ones that
   haven't shown this failure mode.
5. Component-aware output-sanity bounds — sports models have lumpy,
   schedule-dependent volume that a naive universal "50% of
   yesterday" rule would false-alarm on constantly; Archie to
   determine reasonable per-component thresholds from real historical
   volume patterns.
6. Basic alert correlation: if multiple components fail in the same
   run from a shared root cause, send one combined alert naming all
   of them, not one ping per component.

**Separate follow-on items, surfaced by this pass but tracked apart
from the monitoring build itself:**
7. The single-source-of-truth refactor for duplicate status/
   classification logic (the actual fix for the MLB_GAME/Gate 4c-style
   drift pattern) — a code-quality fix, not a monitoring feature.
8. A lightweight, low-effort convention (not automated CI, given the
   real two-instance/chat-based workflow) to catch future duplicate-
   logic drift before it recurs a fourth time.

**One thing needed from Archie before Stage 2 effort can be sized
honestly:** how cheaply does `daily_runner.py`'s actual structure
support adding per-component manifest logging — a small addition or a
real refactor? Shouldn't be guessed at from outside the codebase.

**Uncomfortable question worth keeping in view, not resolved here:**
is monitoring the highest-leverage investment right now versus
reducing the number of fragile external dependencies in the first
place (the still-open MLS data-source migration)? Monitoring catches
problems faster after they happen; it doesn't reduce how often they
happen. Not an argument against building this — just worth not
treating monitoring as a complete substitute for reducing fragility.

## CARRIED FORWARD, UNCHANGED

- Gate 4c: confirmed never broke, accepted in full. Gate 4d scoped as
  a cross-model resolution-window cap, design work can proceed now
  even though the build waits for real entry activity to resume.
- GDP's 7 open positions vs. Gate 4c's cap of 3: agreed, leave them
  alone — already resolved with Archie.
- Raw unfiltered edge count: resolved via Archie's suggestion —
  `logs/daily_report_*.txt` added to fetch list for direct visibility,
  no code change needed.
- Whether the "Gate 4c still not built" claim's origin is traceable —
  still open, lower priority than tonight's items above.

## VALIDATION TRACKER

Unchanged: Gate 1 project-wide suspended, real trigger is GDP's Oct 30
resolution or MLB_GAME hitting n=15-20 spread-tagged signals (currently
only 11 raw resolved rows with spread_cents populated — GDP's Oct 30
resolution remains the realistic path). MLB_GAME combined n=116,
62.9%/0.2294 — real numbers now correctly reflected once tonight's
pull happens. MLS_GAME retired, correctly excluded from alerts.
NFL_GAME/NFL_SPREAD suspended, now correctly tagged and confirmed
absent from today's report. EPL_GAME: retry/fallback shipped and live,
status pending a clean run to confirm it's actually producing signals
again.

---
J@rv1s | Papa Ralph standard.
