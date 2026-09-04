# Nightly Summary — 2026-09-02 / 2026-09-03 (Archie → J@rv1s)

Covers two sessions in one -- Sept 2's summary got written to the wrong
file (nightly_summary.md instead of this one, now corrected) and never
reached you. Combining both nights here rather than leaving Sept 2
undocumented. This is the full, final Sept 3 close-out -- everything
below happened across one long evening session, not spread across
multiple documents.

## ORACLE CLOUD STATUS
Current through commit 7629052 (pushed tonight). Tonight's 21:00 CT
cron cycle fired clean and confirmed both fixes shipped earlier
tonight are holding: signals_log.csv gained 109 real rows (no wipe),
sede_portfolio.json shows a genuine 43-line position update, and
neither nightly_summary.md nor nightly_summary_latest.md appear
anywhere in that cycle's commit diff -- the docs_to_push fix is
working. A new diagnostic probe (/proc/self/stat starttime,
commit 00714f2) shipped after tonight's run, so it hasn't produced
data yet -- the decisive read on the double-start anomaly (see below)
comes from tomorrow's 07:00 CT cycle.

## OPEN POSITIONS
sede_portfolio.json: 8 open (7 GDP, 1 BTC), unchanged composition
after tonight's run -- correct, trading still suspended, no new
entries. 7 of 8 are GDP threshold bets on the same underlying
quarterly outcome, correlated exposure, not new risk but real and
still unaddressed (Gate 4c concentration cap proposed Aug 30, not yet
built).
paper_trades.json: 1 open, unchanged.

## VALIDATION TRACKER
- Gate 1 (project-wide) still SUSPENDED, Sept 8 checkpoint now 5 days out.
- **CLAIMS -- real status change tonight.** Found and fixed a genuine
  bug (not just a stale note): claims_model.py's own suspension gate
  returned an empty list before any signal computation ran at all, so
  CLAIMS logged literally zero rows -- not even track-only ones --
  from the 2026-06-04 suspension date through today, 91 days straight.
  This despite the actual reinstatement fixes (holiday-week detection,
  aftermath widening, confidence tagging) having genuinely shipped
  2026-06-15 -- the suspension note just never got updated to say so.
  Fixed (commit c19d563): gate is now informational only; real
  trade-blocking already happens correctly in two other places
  (MODELS_SUSPENDED_FROM_TRADING, FULLY_SUSPENDED_MODELS) that both
  already list CLAIMS. Verified via isolated test against synthetic
  markets -- confirmed it now reaches real signal-generation logic
  instead of short-circuiting. Next Thursday's release should be the
  first real CLAIMS row logged since June 4.
- **MLB_GAME NO-direction -- flagged, not fixed.** Its "fix within 2
  weeks or NO direction permanently suspended" deadline was set June
  14; that deadline was June 28. It's now September 3 -- 11 weeks past
  its own stated shelf life, still just sitting in "pending diagnostic"
  limbo. Nobody has diagnosed the ~8-point underestimation bug yet.
  Real investigation needed, not a quick patch.
- **MLS_GAME -- real Gate 1 verdict computed tonight, FAILS.** n=151
  resolved signals (well past the 30 threshold its own note was
  waiting on), win rate 39.1%, Brier 0.2693 -- fails both remaining
  criteria outright, not a near-miss. Updated the authoritative
  suspension note in code (commit 7629052) and logged full method to
  the project (mls_game_gate1_verdict_20260903.md). A fresh
  calibration-bucket check at this larger sample size (0-45%: 0/19,
  45-55%: 0/28, 65%+: 47.7%/86 -- worse than a coin flip even at high
  confidence) reinforces the real, research-backed Aug 9 hypothesis
  (Poisson/Dixon-Coles models documented to over-weight weaker
  opposition) rather than pointing to a new mechanism. Disposition
  (permanent suspension vs. further investigation) not decided --
  flagged for Rus/J@rv1s.
- SOCCER_GAME (World Cup) -- unchanged, already failed Gate 1 July 25
  (WR 40.0%), dataset closed, nothing new.
- GDP reduced weight (0.60, scoring stalled since Jul 30) -- unchanged,
  root cause still open, nothing moved on this tonight.
- JOBS caveated (not unconditionally validated) -- unchanged.
- **NFL_GAME -- suspended pending real 2026 signals, season opens
  Sept 9 (6 days out).** Ran the ratified NFL doc's own overdue
  SharpAPI 2-book coverage re-test live tonight (it was scheduled for
  "~Sept 3," which is today, and hadn't been touched since July 18):
  8.5% 2-book coverage (16/189 games), up 5x from July's 1.6% but
  still under the 50% preferred bar. Per the doc's own pre-agreed
  fallback, this is non-blocking -- SharpAPI just stays a single-book
  comparison check, never the primary probability source anyway.
  Logged to the project (nfl_sharpapi_coverage_retest_20260903.md).
  Separately confirmed the NFL_SPREAD ground-truth-map-miss fix from
  earlier this week is holding clean -- zero "[ModelType] WARNING"
  hits across every recent report.
- **Project's own custom instructions found stale.** The standing
  SUSPENDED MODELS list only names CLAIMS and MLB_GAME; the live code
  has six (also SOCCER_GAME, MLS_GAME, MLB_CHAMP, WC_WINNER). This is
  a claude.ai project-settings edit, not something writable through
  any tool available here -- needs Rus to update it directly.

## SEPT 2 WORK ORDER
1. **NFL_SPREAD residual gap found and fixed (commit 408b627).** Last
   night's display fix had a gap: a ground-truth model-lookup miss for
   one live label fell back to a guesser that didn't know NFL_SPREAD's
   format, defeating the fix for that row. Fixed the guesser directly
   and added a warning log for the underlying miss. Bigger than
   display: compute_sede_confidence() shares the same fallback, so
   affected signals were likely scored under the wrong model's
   reliability stats too, not just mislabeled. Confirmed the actual
   subscriber email (fmt_signal) uses a different mechanism and was
   never affected.
2. **Cron data-loss diagnostic hardened twice** (breadcrumbs, then raw
   os-level write+fsync+independent-reread+inode identity) after the
   Sept 2 21:00 CT run showed pre_logging/pre_push checkpoints never
   landing on disk despite zero exceptions.
3. **Subscriber alert format -- honest UX review, no code changes.**
   Confirmed a newcomer would be confused by the current report. Had
   Gemini review it too -- marked genuinely useful ideas (generic
   probability labels, GDP correlated-exposure warning, tying
   thin-market flags to real spread data) vs. ideas that conflict with
   the existing subscriber spec. Saved for whenever P2 gets built.
4. **MLB_GAME historical loss concern checked** -- 15 consecutive
   clean days before the incident window started, likely not a
   longstanding systemic issue.

## SEPT 3 WORK ORDER
1. **Likely root cause of the cron data-loss bug found and fixed
   (commit 6c1a191).** data_freshness.py's auto_pull_if_safe() was
   silently discarding live trading state (signals_log.csv,
   sede_portfolio.json, paper_trades.json among them) as if it were
   routine regeneratable dirt. Removed 7 append-only/live-state files
   from that discard list. Confirmed holding clean on tonight's 21:00
   CT run.
2. **The nightly_summary.md stale-revert bug found and fixed (commit
   7850247).** push_to_public_dashboard() was unconditionally copying
   a frozen source file (nightly_summary.md, untouched since June 18)
   over the dashboard repo's real copy every single cron cycle --
   confirmed directly via git history showing a same-night real update
   getting reverted within hours. Fixed by emptying docs_to_push;
   confirmed neither nightly_summary.md nor this file were touched by
   tonight's 21:00 CT cycle.
3. **Untracked WIP files investigated and committed.** paper_trade_
   candidates.py / paper_trade_gates.py / paper_trade_shadow_logger.py
   (Tier 1/Tier 2 paper-trade tooling, Aug 30 sign-off, standalone
   manual/dry-run tools, never wired into daily_runner.py or cron) plus
   4 model_integrity docs from Aug 30-31 -- all had been sitting
   uncommitted on the laptop for up to 5 days with no backup anywhere.
   Committed and pushed (commits 9226b4a, 65f9160).
4. **Double-start diagnostic anomaly -- caught live, still genuinely
   unresolved.** Tonight's 21:00 CT run showed "start" checkpoint
   firing twice under the identical pid/ppid, once without the new
   probe fields and once with them -- ruled out a full process
   relaunch (no second pre_logging appeared), ruled out any internal
   or external retry/loop (read the full function body directly, none
   exists), confirmed exactly one textual call site per checkpoint
   label. Added a /proc/self/stat starttime probe (kernel-assigned at
   process creation, immune to anything the code does) that will
   settle definitively whether this is boundary-case pid reuse or
   something live-patching a running interpreter -- watch for the
   result on tomorrow's first cron cycle.
5. **"SEDE restart vs harden FORGE outcome" carry-forward item --
   investigated, found to be a phantom.** No trace anywhere across
   filesystem, project, git history, or past briefings; Rus doesn't
   recall it either. Dropped from future carry-forward lists.
6. **NFL sweep ahead of the Sept 9 season opener** (see Validation
   Tracker above for the SharpAPI re-test and NFL_SPREAD confirmation).
   Also catalogued real, disclosed, non-blocking gaps already on
   record in the ratified NFL doc: rest-day adjustment and late-season
   motivation asymmetry were both planned and never built. Confirmed
   the Gate 1 tracking infrastructure (brier_dashboard.py,
   validation_dashboard.py) is generic across models, so NFL_GAME will
   be picked up automatically once Week 1 resolves -- no NFL-specific
   dashboard work needed.
7. **Full "what's sitting" sweep across every model**, at Rus's
   request, not just NFL. Results folded into the Validation Tracker
   above: CLAIMS bug found and fixed, MLS_GAME real verdict computed,
   MLB_GAME's blown deadline flagged, GDP's stall confirmed unchanged,
   project instructions confirmed stale.
8. **Soccer/EPL strategy discussion**, at Rus's request ("crack the
   soccer code"). Found EPL already has a fully-designed, real-data-
   backtested, ready-to-build spec in the project (FORGE-closed Aug
   23) that is NOT blocked by MLS's problem -- it was specifically
   designed around MLS's known flaw (current-season-only team-strength
   stats, which is genuine noise at a fresh season's 0-1-game start).
   Flagged a real risk nobody had connected before tonight: EPL's
   build would inherit the same core Dixon-Coles probability engine
   MLS and World Cup already share. Tonight's fresh MLS calibration
   check at n=151 (5x August's sample) reinforces the existing,
   research-backed "over-weights weaker opposition" hypothesis rather
   than surfacing a new bug. Decision deferred to tomorrow.

## PENDING, RANKED FOR TOMORROW
Rus's explicit instruction tonight: normal startup, then NFL/MLB
first; soccer/EPL work only if nothing pressing there.
1. NFL/MLB check first (Rus's priority) -- season opener 6 days out,
   MLB Gate 1 checkpoint 5 days out.
2. Read tomorrow's 07:00 CT cron diagnostic -- does the /proc/self/stat
   probe show identical tick counts across a double "start" (proves
   live-patching) or different ones (proves plain pid reuse)? This is
   the decisive test.
3. MLB_GAME NO-direction diagnostic -- genuinely overdue (11 weeks
   past its own deadline), needs real investigation into the ~8pt
   underestimation, not just documentation.
4. GDP reliability weight still stuck at 0.60 since July 30 -- root
   cause still open, untouched tonight.
5. If NFL/MLB is clear: soccer/EPL decision -- pursue a real historical
   soccer data source to properly fix MLS's underdog-weighting bias,
   and/or start the EPL build (spec is ready) with a conservative
   underdog-weight adjustment added on top.
6. Project's custom instructions -- suspended-models list is stale (2
   of 6 named); Rus-side edit, not writable through any tool here.
7. "Positional-tuple type inference" pattern naming -- still
   unaddressed, carried forward again.

## SPORTS MONITORING
Nothing open in either ledger depends on live game monitoring tonight.
NFL season opens Sept 9 (6 days), MLB Gate 1 checkpoint Sept 8 (5
days) -- both close enough now to have front of mind starting
tomorrow.

---
Archie | Papa Ralph standard.
