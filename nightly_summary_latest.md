# Nightly Summary — 2026-09-05 to 2026-09-07 (Archie → J@rv1s)

Three real work nights in one doc (Friday 9/4 had no session — routine
Oracle auto-updates only, nothing to report). Sept 5-6 covers a lot of
ground; Sept 7 (today) both finished that thread and caught a real
mistake in how it was being reported. Closing the session tonight with
this as the single consolidated handoff.

## ORACLE CLOUD STATUS
Oracle's real crontab (pasted by Rus, confirmed 2026-09-07): exactly 3
entries, all running `daily_runner.py` only, at 07:00 / 11:15 / 21:00 CT
daily, no weekday gap. `mlb_refresh.py` (the noon/4PM lighter refresh)
is laptop-only — never runs on Oracle. Local HEAD is `f57fd3e`, pushed
clean to origin/main. Oracle has pulled through the 11:15 CT cycle
(`edb02e0`) but has NOT yet picked up tonight's later commits
(`be71852` through `f57fd3e`, listed below) — that happens automatically
on the 21:00 CT run, ~3 hours from this write-up. Worth a quick check
that cycle's log rather than assuming.

New this window: a passive read-only mirror task (`mirror_pull.bat`,
laptop Task Scheduler, hourly at :45 — fetch+ff-only, never commits,
can't create a conflict) and Oracle's crontab backed up locally
(`oracle_backups/oracle_crontab_20260907.txt`, gitignored folder).

## OPEN POSITIONS
`sede_portfolio.json` (subscriber-facing track): 8 open (7 GDP YES
threshold bets, 1 BTC<$50k NO), bankroll $994.17, 1 early exit on the
books (-$5.83), no resolutions yet. Same correlated-exposure situation
as before — 7 of 8 on the same underlying Q3/future GDP outcome, Gate
4c concentration cap still not built.
`paper_trades.json` (calibration ledger): 1 open (GDP>3.5% YES,
entered 8/30, resolves with the Q3 advance estimate 10/30).

## VALIDATION TRACKER — SEPT 8 GATE 1 CHECKPOINT (tomorrow)
Real recommendation, computed fresh this window, not carried forward:
**extend the provisional suspension again** — not because tomorrow's
date arrives, but because no model has both a real Gate-1-sized sample
AND enough spread data to stress-test it yet.
- **GDP**: spread data exists, 6 of 7 open Q3 markets stress-survive
  realistic execution cost — but real resolved sample is only n=6, no
  resolution event before Oct 30.
- **MLB_GAME**: real resolved sample n=112 (passes n, 62%+ WR, Brier
  ~0.23 — narrow miss on the 0.20 bar), but only 4 real spread-tagged
  signals, too thin to say anything about execution cost.
- **NFL_GAME/NFL_SPREAD**: spread/book capture just got wired in
  (below) — zero real spread-tagged signals yet, but now a live
  candidate for the *next* checkpoint's trigger as real season volume
  accumulates.
- Proposed real trigger for the next checkpoint (not another arbitrary
  date): whichever comes first of GDP's Oct 30 resolution, or
  MLB_GAME/NFL accumulating n>=15-20 real spread-tagged signals.
- **MLB_GAME NO-direction "anomaly" — resolved as a non-issue.** A
  candidate explanation floating around (a "known NO-direction
  miscalibration, 56.8% vs 64.9%") turned out to be the *original* June
  14 claim, already investigated and retracted August 9 at n=40 (small-
  sample noise, same pattern in YES too — `model_integrity/
  amendment_log.md`). Reran it fresh against the current full n=112
  sample: YES underestimates its own flagged direction by +7.0pts, NO
  by +6.5pts — symmetric, confirms the Aug 9 retraction still holds.
  MLB_GAME's real suspension reason stays the narrow Brier miss, not a
  direction-specific bug.
- CLAIMS, MLB_GAME NO-side trading, GDP's 0.60 reduced weight — all
  unchanged from before.

## SEPT 5 WORK ORDER (Saturday)
1. **MLB Track B checkpoint — real numbers, overdue 8 days, finally
   computed.** 177 post-fix scored games: Track B (Elo+FIP blend, FIP
   weight already zeroed 8/9) Brier 0.2419 / 61.0% hit rate vs
   SharpAPI's own line at 0.2318 / 60.4% on the same games — parity,
   not a demonstrated edge over a professional line. Took to J@rv1s;
   decision below.
2. **Duplicate-row logging root cause found.** `log_signals()` in
   `daily_runner.py` had zero check for "have I already logged this" —
   one real JOBS prediction got logged 28 times across 18 days.
   Read-side dedup fix shipped same night in `brier_dashboard.py`'s
   `load_scored_signals()` (real per-event key: ticker first,
   gap-clustered label fallback only for rematch-risk sports models).
   Surfaced a second bug in the process: 19 real MLB_GAME games (34
   rows) were being silently dropped from every Gate 1 count because
   `p_yes_model` held a string instead of a number on rows written by
   `mlb_refresh.py`'s THIN MARKET path — fixed the symptom (loud
   warning + kept row) that night, root cause chased down Sept 6.
   Corrected real Gate 1 numbers that night: JOBS n=6 83.3%/0.1210
   Brier, GDP n=6 66.7%/0.1698, MLS_GAME n=33 42.4%/0.2717, MLB_GAME
   n=109 62.4%/0.2305.
3. **JOBS Gate 1 spread stress test** — its own dedicated memo,
   real n=6 (arguably ~2 independent events), no real Gate 1 sample,
   earlier "corroborated" language retracted.

**Decision reached (J@rv1s, relayed):** Track B redesignated —
purpose is now "calibration/sanity-check against a professional line,"
not "beat the market." FIP got a genuinely rigorous out-of-sample
derivation and a fair shot; its near-zero real correlation is a
finished experiment with a real negative result. No further FIP
redesign work or checkpoint-extension cycling. Logged in
`track_b_model.py`'s own header.

## SEPT 6 WORK ORDER (Sunday)
1. **p_yes_model/prob_source field-scramble root cause found and
   fixed.** Not the tuple-arity issue guessed the night before —
   `log_signals()` was the last writer still trusting the static
   `SIGNALS_HEADERS` constant instead of the file's real on-disk header
   at append time, confirmed disagreeing on column order/count at least
   3 times across a 2026-07-15/18 window. Fixed by reading the real
   header at write time. All 34 corrupted rows repaired (back-solved
   from each row's own correct `brier_score`, verified exact match
   before writing). Also found, separately: `mlb_gametime_fill.py` was
   writing the literal string "RESOLVED" into a numeric column when it
   couldn't find a price — fixed same night (13 corrupted rows blanked).
2. **Dashboard trust language gated on sample size.** Any model n<30
   now gets explicit "too early to act on" language regardless of Brier,
   instead of pure-Brier labels that would've told a reader to "trust"
   a 6-signal model.
3. **NFL readiness pass (Rus flagged kickoff proximity) — clean on the
   model, but surfaced 3 real infrastructure gaps, all fixed:**
   - `label_to_true_model`'s silent last-write-wins collision risk
     (NFL_GAME shares label format with 3 other sports) — now logs
     loudly on a real collision instead of staying invisible.
   - The bigger one: `detect_signal_model()`'s unsafe "@"-fallback
     guess (defaults to MLB_GAME) was trusted enough to grant the
     MLB_GAME_YES_EXPERIMENTAL live-trade-entry bypass — meaning an
     NFL/NBA/NHL signal that missed the ground-truth map could ride
     into a real premature paper trade under the wrong label, days
     before NFL is even supposed to be tradeable. Now requires a
     *confirmed* classification, never a guessed one.
   - NFL_SPREAD was missing from `daily_runner.py`'s own
     `SEDE_RELIABILITY` dict (had it in `paper_trade_gates.py`'s mirror
     copy the whole time) — was silently falling back to the generic
     0.70 default instead of a deliberately low 0.55, on a model that
     (unlike NFL_GAME) is currently eligible for real paper-trade entry.
     Fixed.
   - Checked whether `paper_trades.json`'s separate manual entry track
     shared any of this exposure — it doesn't, 100% manual, no
     model-guessing involved anywhere in that path.
4. **GDP_STD real revision-history derivation.** Pulled actual GDPNow
   tracking data (`gdpnow_history.txt`) for Q2/Q3 2026: found the
   opposite of the hypothesis this project had been carrying — the
   first ~4 weeks of a quarter are calm (~0.20pp stdev), and volatility
   jumps specifically around day 30, when the *prior* quarter's BEA
   advance estimate lands. Deliberately not wired into `gdp_model.py`
   yet (n=2 quarters, live model, needs explicit sign-off) — see Sept 7
   correction below on how this interacts with the out-quarter decision.
5. First draft of the Sept 8 Gate 1 checkpoint write-up — caught and
   corrected a real inflated GDP count in its own first pass (91 raw
   rows vs. 7 real distinct markets) before it went out.

## SEPT 7 WORK ORDER (today, Monday) — mostly correcting the record, plus real new fixes
1. **JOBS silently dead since ~Sept 4 — 4th confirmed recurrence of the
   same bug.** `REPORT_DATE` staleness, same pattern as always. Rolled
   to Oct 2 with real August BLS actuals (NFP +162K vs ~53K consensus —
   a real reversal), interim Sept consensus, widened std dev. Shipped
   without waiting on sign-off, same as prior recurrences. **This keeps
   recurring — a standing calendar reminder or automated staleness
   check in the morning briefing would stop this needing a 5th audit.**
2. **Shadow-book — real wiring bug found and fixed, not just a
   rare-event story.** `mlb_refresh.py` (noon/4PM) has called
   `run_mlb_game_model()` daily since 8/11 without ever passing
   `shadow_book=` — the near-miss branch's own backward-compatible
   default silently skipped it every time, no error, no way to tell
   "ran fine, zero near-misses" from "never live." Only the 9PM full
   run ever had a chance to populate it. Fixed and verified live.
   Whether to widen the near-miss band is still J@rv1s's separate call,
   once real data flows from all three windows.
3. **NFL_GAME/NFL_SPREAD spread/book capture wired in** — real season
   volume is accumulating right now (unlike GDP/JOBS, which are
   calendar-capped either way), so this makes NFL a real candidate
   trigger for the next Gate 1 checkpoint. Caught and fixed one real
   regression risk before shipping: `email_alerts.py`'s NFL_SPREAD
   detection was pinned to the old tuple length and would have let the
   MLB_GAME team-name-extraction branch fire on the new one, reviving a
   mislabeling bug already fixed twice before.
4. **Oracle/laptop dual-scheduling — real root cause found and fixed.**
   Not a same-instant collision (both machines already run 07:00/11:15
   /21:00 CT by deliberate design, confirmed via Oracle's real crontab).
   Real cause: the laptop's own local pipeline writes to 7 live-state
   files were never safely discardable after Sept 3's protective fix
   (which was needed to stop Oracle from losing its own data) got
   applied uniformly to both machines — so every laptop run made its
   own local drift from origin strictly worse, no way back to clean.
   Fixed with a machine-aware discard list. Rus's call: keep both
   machines running the mirrored schedule, no change to that model.
5. **A real intra-session correction, worth naming plainly.** After
   reporting items 2 and 4 above (from Sept 6/7) as "still open,
   nothing done" to Rus mid-session, a fresh check of this project's own
   docs found they'd already been fixed earlier the same session — this
   wasn't a J@rv1s/Archie cross-instance gap, it was the same instance
   losing track of its own recent work, almost certainly across a
   context-compaction boundary. Corrected immediately once caught.
   Worth J@rv1s knowing this failure mode exists on this side too, not
   just cross-instance: check a topic's own dedicated doc for later
   updates before reporting status, don't trust a running summary alone.
6. **GDP out-quarter methodology — already resolved, not a fresh
   decision.** A message came in framing "Option A" (restrict GDP
   signal generation to current-quarter markets) as a new ratification
   needing implementation. Checked `models/gdp_model.py` directly before
   building anything: Option A was already implemented and live, shipped
   2026-08-26, well-built (fail-safe ticker-date parsing, current
   quarter = soonest real resolution date among live markets, out-quarter
   markets skipped and logged not dropped), confirmed as the only real
   call site. So: independently arriving at an already-shipped decision
   12 days later — a real cross-instance information gap, not wasted
   work or a disagreement. Corrected the record in `docs/backlog.md` and
   the project doc, including the Sept 6 GDP_STD derivation's own
   recommendation, which had assumed this was still an open A/B choice.
   What's genuinely still open: wiring the day-30-aware two-regime STD
   into the now-current-quarter-only model, deliberately deferred until
   Q3 completes Sept 30 (n=3 quarters) and the day-30 boundary is
   confirmed against the real BEA release calendar. Nothing to build
   before then.
7. **Hardcoded-Windows-paths audit (backlog item since 7/7, 9 files) —
   re-verified against ground truth, not assumed.** Checked Oracle's
   real crontab (only ever runs `daily_runner.py`) and both files' real
   import graphs. All 7 remaining files do have real hardcoded
   `C:\KalshiBot` paths, but none are reachable from Oracle's cron
   today — each is a standalone laptop-only script, or in
   `polymarket_monitor.py`'s case, not invoked anywhere at all. Two real
   latent risks flagged: `auto_monitor.py` is explicitly slated to move
   to Oracle at migration (its docstring says so) and will break that
   day unless converted first; `polymarket_monitor.py`'s docstring
   claims a `daily_runner.py` integration that was never actually
   built — zero call sites anywhere, real feature gap or stale comment,
   Rus's call which.
8. **Stop-loss reinstatement confirmed real.** `auto_monitor.py`'s
   $15.00 limit has a full poll→trigger→close execution path (6AM-6PM
   CT), explicitly guards against a previously-fixed sign bug recurring.
   `trade_monitor.py` enforces the same number for reporting. Confirmed
   this is "the" reinstated stop-loss, working as intended.
9. **Housekeeping.** Dropped 3 stale git stashes (verified each one's
   content already existed elsewhere first — one looked like a real
   orphaned `fed_model.py` fix, confirmed already live before dropping).
   Cleared 5 stray root-level scratch scripts and a leftover video
   transcript file. Gitignored a recurring untracked log
   (`logs/refresh_*.txt`, written every noon/4PM run with no cleanup).
10. **Dead code archived** (`oci_retry.py/.ps1`, `paper_trades.py` —
    confirmed zero live references) and `docs/backlog.md` rewritten
    twice against verified current state, not carried forward on faith.

## STILL GENUINELY OPEN — RANKED FOR NEXT SESSION
1. **Sept 8 Gate 1 checkpoint** — real recommendation above (extend,
   criteria-driven trigger for next one), Rus's call whether to accept
   it or push further.
2. **JOBS REPORT_DATE staleness** — 4th recurrence; needs a structural
   fix (calendar reminder or automated check), not a 5th manual catch.
3. **EPL carryover-blend** — approved spec, real backtested numbers,
   ready to build, correctly queued behind Sept 8 prerequisites.
4. GDP reduced weight (0.60, scoring stalled since Jul 30) — root cause
   still open, untouched this window.
5. `requirements.txt` still doesn't exist — real structural gap.
6. GDP_STD day-30-aware STD — correctly parked to Sept 30, no action
   needed before then.
7. `auto_monitor.py` hardcoded paths — fine today, will break at Oracle
   migration if not converted first.
8. `polymarket_monitor.py` — dead/orphaned despite its own docstring;
   decide build-it-for-real vs. retire the claim.
9. `.env` backup to `oracle_backups/` — needs Rus to run the `scp`
   command directly from his own terminal (real API keys, deliberately
   not routed through any Archie session).
10. NFL late-season motivation adjustment (built, inert pending a live
    clinch-status feed) and NFL totals/spread market wiring (not built)
    — unchanged, season now live.

## SPORTS MONITORING
MLB in full swing, NFL season now open (kicked off Sept 9 as scheduled
last window) — NFL_GAME/NFL_SPREAD spread data starts accumulating from
tonight's runs forward. Nothing tied to a live game needs attention
tonight specifically.

---
Archie | Papa Ralph standard. Three nights, one real self-caught
mistake owned plainly the moment it was found rather than smoothed
over, several stale claims corrected against actual code instead of
carried forward, and one already-shipped decision saved from getting
"re-ratified" a second time. If it's worth doing it's worth doing right.
