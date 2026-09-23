# Nightly Summary — 2026-09-22 (Archie)

Rus opened with "start work orders 1 -> 7" plus your full Sept 22 briefing,
with explicit instructions to work the 7 numbered items first and only
engage the rest of the briefing afterward. All 7 closed tonight, plus a
review of the briefing's remaining content. Commits: 56dfd2a/79da7dc (WO3),
bf05dbb (WO7).

## WO1 — Oracle pull confirmation
Already closed by your own briefing's independent confirmation. No work
needed.

## WO2 — Gate 1 bid/ask-spread stress test
Built and ran the first real bid/ask-spread stress test against 349 real
resolved, spread-tagged contracts — the exact test the Aug 11 project-wide
Gate 1 suspension has been waiting on. Real finding: roughly a third of
paper-reported P&L evaporates once priced at the real ask instead of mid,
and only 22% of apparent 15c-edge signals survive real execution.
Recommendation: **keep the suspension in place.** Not yet done: building
this into the permanent pipeline (currently a one-off script) and applying
it to GDP once its Q3 2026 markets resolve Oct 30.

## WO3 — single-source-of-truth (n_events counting)
Your pushback was correct and acted on — signal_scorer.py now imports and
reuses brier_dashboard.py's real_event_id(), verified against a live run
(801 real events from 1795 rows, cross-checked against
brier_dashboard.model_breakdown() at the same moment — matched exactly
across all 9 models). daily_runner.py's MLB-specific function was left
alone deliberately (it already imports brier_dashboard's numbers directly,
low practical risk today — 134 vs 135). **Real remaining gap, your point 2
stands only partially resolved:** the full architectural unification (one
shared function instead of two independently-verified-to-agree ones) is
still not built. Confirmed live on Oracle via `git log --oneline -5`.

**Your point 1 pushback ("n_events=30 sits exactly on the floor")** —
tonight's live re-verification run showed NFL_SPREAD at n=424/events=31,
one clear of the 30 floor, not the knife-edge number from last night.

## WO4 — MLB_GAME/WC_GAME ticket-gap investigation
MLB_GAME's ticket gap is resolved/legacy. WC_GAME's is real, and bigger than
originally asked: daily_runner.py logs WC_GAME's signal stream under the
model name "SOCCER_GAME" (line ~2714). Rus didn't remember why, so did the
git archaeology directly: commit ad9dac9, June 16 2026, "SOCCER_GAME rename
-- WC_GAME -> SOCCER_GAME in daily_runner" — a fully deliberate, documented
rename of every user-facing label, internal variable names left alone on
purpose. **Confirmed intentional, not drift.** Live risk that remains:
signal_scorer.py and brier_dashboard.py still track "WC_GAME" and
"SOCCER_GAME" as two separate models today (49 vs 70 real events per
tonight's run) even though they're the same continuous model before/after
the rename. Real, well-scoped follow-up, not done tonight: merge the two
labels for Gate 1 counting.

## WO5 — CPI MoM
No new work — confirmed last night's cut-from-feed decision still stands.

## WO6 — MIA@SF anomaly
**Not a bug.** Pulled the real 26 matching rows. The 24 NFL_GAME rows
matching your cited 37.0%/21.0c figures are NOT simultaneous — YES/-MIA
rows run Aug 29-31 only, NO/-SF rows run Sept 1-6 only, zero timestamp
overlap. They're also mathematically identical regardless: both directions
independently work out to the model believing P(MIA wins) ≈ 37% the whole
time, just expressed through the ticker's mirror side. Kalshi's own
mid-price stayed continuous through the switch too — no real market
discontinuity. Already correctly collapsed to one event by the WO3 fix.
One open, non-urgent curiosity: why the tracked ticker switched from -MIA
to -SF on Sept 1 with a ~23-hour gap. Not chased further.

## WO7 — Stage 1 monitoring, phase 1 (external heartbeat)
Honest caveat up front: there's no standalone FORGE spec doc anywhere in the
project for "Stage 1 monitoring" — built from breadcrumbs scattered across
your briefings (external heartbeat, whole-pipeline state alerting, short
daily digest; Healthchecks.io "already chosen"). Confirmed via code read
that the pipeline body under `if __name__ == "__main__":` had NO outer
try/except and run_kalshibot_full.bat never checked its own exit code — a
crash was previously invisible to anything outside the process. Confirmed
via grep that no heartbeat mechanism existed anywhere before tonight.

**Built and fully tested live, both machines:** tools/heartbeat_ping.py
pings Healthchecks.io on success/fail (no-ops safely if unconfigured, never
raises); wired into run_kalshibot_full.bat (laptop) and a new
tools/run_daily_with_heartbeat.sh (Oracle, matching Oracle's real crontab
exactly). Walked Rus through live setup end to end tonight — Healthchecks.io
account created, both checks configured (12h Period / 2h Grace, matching),
both `.env`s updated, Oracle's crontab repointed, both machines manually
test-run and confirmed green with real successful pings on record (not just
clean exit codes — verified Oracle's actual daily_runner.log tail showed a
genuine complete run, GitHub push OK, dashboard push OK). This is now real,
live, and doing its job — not just declared done.

**On your `sentry`-vs-Healthchecks.io question:** resolved by building —
Healthchecks.io is what's live now. Not treating them as true substitutes
either way (cron liveness check vs. app error tracking are different jobs).

## Briefing pushback — the 4 points
Points 1 and 2 covered above under WO3. Points 3 and 4:

**Point 3 (97% NFL_SPREAD correlation as a live risk-concentration finding,
not just accounting) — you're right, and it's a real, confirmed, unfixed
gap.** Checked paper_trade_gates.py's actual `qualifies()` function directly:
it takes only model_type/direction/edge_cents/model_pct — zero ticker or
game-identity awareness. There is currently no same-game/same-event dedup
anywhere in the entry criteria. Same disease as Gate 4c/4d, different model.
NOT fixed tonight — flagged as a real design decision deserving its own
session, not a rushed end-of-night patch. Good news: we already have the
tool to fix it (real_event_id(), built Sept 21) — applying it as a same-event
cap in the entry gate is a well-scoped, probably small future build.

**Point 4 (was CPI's June 10–Sept 21 dead window ever backfilled) — no,
straight answer.** That ~3.5-month gap in CPI's Gate 1 sample is real and
currently permanent. A retroactive backfill would need the same shape of
work as the MLB/MLS outcome backfills (sourcing historical consensus +
ticker data for those months specifically). Not built, not scoped tonight —
flagged as an open decision (build it, or explicitly accept the gap) rather
than left silently unaddressed.

## Tool/plugin referrals — reviewed with Rus, none adopted tonight
Went through the ~10 referrals from today's briefing with Rus directly, with
real pushback rather than rubber-stamping the rankings:

- `remember` — actually the one I'd have prioritized highest (targets this
  project's own repeated real context-loss incidents), but checked it via
  SearchPlugins and it does NOT exist in this account's installable
  catalog — likely a Claude Code CLI-specific plugin, a different surface
  than this Archie session runs on. Worth re-checking from Rus's actual
  local Claude Code install if this comes up again.
- `pmxt` — pushed back: no real need without Polymarket actually on the
  roadmap, and real risk trading a battle-tested, incident-hardened Kalshi
  integration for an unproven abstraction just for developer convenience.
- quantitative-trading's risk-manager plugin — pushed back: the actual fix
  (same-event dedup, see Point 3 above) is ~20-30 lines using
  real_event_id(), which we already have. Didn't see the case for adopting
  an external agent to do something we're already equipped to do directly.
- `code-review` — questioned whether it adds anything beyond the
  Archie/J@rv1s nightly review loop that's already demonstrably working
  (tonight's own 4 pushback points are the proof it functions).
- `repomix`, Perplexity, `superpowers`, `sentry` (moot, see WO7),
  `frontend-design`/`eli5`, `awesome-quant`, `/doctor`, the CloddsBot
  security flag, the TikTok workflow list — reviewed, no strong pushback
  either way, but Rus's final call across the board: **none get adopted
  right now.** Not revisiting unless something changes (e.g. Polymarket
  actually enters the roadmap, reviving the pmxt question).

## Gate 1 / validation status — unchanged
Still project-wide provisionally suspended since Aug 11, pending the
spread stress test (WO2 above reinforces: keep it suspended). Real triggers
unchanged: GDP's Q3 2026 markets resolving Oct 30, or MLB_GAME reaching
n≥15-20 spread-tagged signals.

## Carried forward — ranked, most urgent first
1. Same-game/same-event dedup missing from paper_trade_gates.py's entry
   criteria (Point 3, new tonight) — real concentration risk, tool to fix
   it already exists.
2. WC_GAME/SOCCER_GAME label merge for Gate 1 counting (WO4, new tonight).
3. Gate 1 spread-check needs to move from one-off script to permanent
   pipeline step (WO2 follow-up).
4. Full signal_scorer.py/brier_dashboard.py architectural unification
   (WO3 follow-up, your point 2 only partially resolved).
5. Apply the spread check to GDP once Q3 2026 markets resolve Oct 30.
6. CPI backfill decision (Point 4) — needs an explicit call, not silent
   drift.
7. mlb_model.py park-factor gap (carried) — natural pickup after MLB_GAME's
   season ends Sept 27.
8. WC_GAME's 16:1 raw-to-event dedup collapse ratio (WO4, carried) — still
   unverified as legitimate vs. a dedup bug.

## Oracle Cloud status
Both WO3 (56dfd2a) and WO7 (bf05dbb) confirmed live on Oracle tonight —
verified directly via `git log --oneline -5` on Oracle showing both commits
in history, and via a real manual test run of the new heartbeat wrapper
(tail of daily_runner.log showed a genuine complete pipeline run, GitHub
push OK, dashboard push OK). Both Healthchecks.io checks (kalshibot-laptop,
kalshibot-oracle) green with real recent pings, 12h Period / 2h Grace.

## Sports monitoring
No open positions requiring in-session monitoring tonight.

Archie | Papa Ralph standard.
