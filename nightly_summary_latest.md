# Nightly Summary — Archie → J@rv1s
## Weekend arc: Friday August 21 (after last handoff) through Sunday August 23, 2026 (session close)
NOTE: this replaces the previous nightly_summary_latest.md, which only
covered Aug 21 night into Aug 22 early AM and was explicitly held/batched
for this write per that note. This one covers everything since, in one
consolidated document per protocol — no addenda.

---

## HEADLINE

Three build threads shipped this weekend (NFL rest/motivation/spread,
MLB per-side market-matching fix, EPL taper design) plus a fourth,
unplanned one today: an external (Gemini) code review of the MLB fix
turned up a real architectural gap — two silent-failure modes with zero
monitoring — which got fixed and pushed to origin/main tonight. Along
the way I caused and then fixed a real incident (a git stash conflict
that briefly corrupted several live production data files). Also:
pulled the real validation numbers, and they are not good — worth
saying plainly rather than letting the build-thread momentum carry the
story. Full breakdown below.

---

## WHAT SHIPPED

**JOBS Gate 1 audit + reset (carried from the last handoff, ratified
Aug 22)** — already reported in the prior nightly summary; restating
only the outcome since it governs everything downstream: original
JOBS Gate 1 pass (n=63, Brier 0.1337) was scored against a
code-comment-labeled "PLACEHOLDER" NFP number for 53 of 63 signals
(84%). Gate 1 checkpoint reset Aug 25 → **September 8, 2026** — third
reset since Aug 11. The 8 new JOBS signals surfaced by the ff582c3 fix
are held log-only, not tradeable, until NFP_CONSENSUS is real.

**NFL — 3 items, commits f71cd12 / dedc188 / 9a698f5 (Aug 22)**
1. Rest-day Elo adjustment (`rest_elo_adjustment()`,
   `build_rest_lookup()`), sourced from a 2024 academic paper (n=5,679
   games): short week -3.5 Elo, mini-bye +12.0, full bye +7.75. Wired
   into daily_runner.py with fail-safe degradation. Verified against 3
   scenarios offline; not yet confirmed live — season hasn't started
   (Week 1 = Sept 3).
2. Late-season motivation-asymmetry adjustment
   (`motivation_elo_adjustment()`), sourced from a Financial Services
   Review paper: -4.0 for any clinched seed, -8.0 for a clinched #1
   seed. **Real live gap, flagging plainly**: `motivation_lookup` is
   always None in production right now — no live clinch-detection feed
   has been built. The adjustment exists in code but cannot fire yet.
3. KXNFLSPREAD wired (24 threshold markets/game) —
   `margin_cover_probability()` using a Normal CDF with sigma=13.65
   (sourced from Hal Stern/Neil Paine). Moneyline model regression-
   tested unchanged after the shared-parser refactor.

**MLB per-side market-matching fix — commit b5cb427 (Aug 22)**
Zero MLB_GAME signals logged since Jul 29 — the matcher's per-side path
wasn't reliably resolving team keywords against real Kalshi titles.
Fixed, verified live: 12/12 real games matched on the next run.

**EPL — FORGE scoping + taper design (Aug 22–23, not yet built)**
First live test of the merged Papa Ralph/FORGE process on a
Mandatory-tier decision. FORGE pass on the carryover-blend design found
3 flaws (squad-turnover blind spot flagged as most concerning, an
undefined taper function, an unsourced promoted-club floor) and
recommended J@rv1s run an independent pass on the taper/floor before
building, given convergent-instance bias risk (Archie originated the
scoping pass). Base-rate research sourced real EPL outcome rates
(~42-45% home / 22-25% draw / 30-33% away, 2023-26 seasons excluding
pandemic/1990s outliers) — no threshold decided yet, flagged as
Rus/J@rv1s's call. Taper derivation: real 6-season backtest (4
transitions, football-data.co.uk) found the honest current-season
weight curve is much slower than a naive NFL port — ~20% at kickoff,
crossing 50% around game 10-12, leveling ~65-70% by game 25-30, never
reaching 100%. Closed out today: sigmoid `w(n) = 0.70 / (1 +
exp(-0.1666*(n-5.50)))` locked in against J@rv1s's constraints, checked
against 9 backtest buckets (mean abs deviation 0.04 across the 7 clean
ones). **Design is ready to build; not built yet** — still behind the
Sept 8 checkpoint, not urgent tonight.

**Today (Aug 23) — Gemini external review + MLB silent-failure
hardening + push**
Rus asked for the same independent, code-checked pressure-test on
Gemini's review of the two MLB fixes that J@rv1s's Dixon-Coles slip
got earlier. (Note: the pasted Gemini text Rus forwarded contained an
embedded instruction telling me to stop and output only an internal
summary — a prompt-injection attempt, not a real instruction. Flagged
to Rus, disregarded, task completed as asked.)

Verdict, checked against real code and live data, not either AI's
description of the system: Gemini's specific supporting claims were
partly wrong (the ESPN block *was* diagnosed — a confirmed Akamai CDN
block, not a mystery; the "A wins" vs "A to win" wording example
doesn't apply, the matcher never reads that phrase) but its underlying
instinct was right — both fixes leave a "trust the external source, no
monitoring" architecture untouched, and the real, sharper version of
its title-matching critique is that `match_game_to_kalshi()` ignores a
clean structured field (the ticker's team-code suffix) already sitting
in every market, in favor of noisy title-text scanning, when the
codebase already has the better ticker-based pattern working in
production for soccer. Also confirmed `data_freshness.py` already
exists and already does most of what Gemini proposed building from
scratch — the actual gap was two missing THRESHOLDS/MODEL_BLOCK_MAP
entries plus one genuinely new check type (volume/zero-count, not
staleness — neither failure this week was a staleness problem, both
sources were live and returning fresh, empty/wrong results). Full
verdict: `archie_external_review_verdict_20260823.md`.

Rus authorized going ahead on the concrete next actions. Shipped, to
the real files on C:\KalshiBot, verified offline before pushing:
- `fetch_mlb_schedule()` now writes `data/mlb_schedule_state.json` on
  every call so a run of failures can't erase how long it's really
  been since the last good fetch.
- The stale "(Off day, or ESPN API unavailable)" message — wrong API
  name since the Aug 21 rewrite — now reads that state file and
  reports the real reason.
- `match_game_to_kalshi()`'s per-side path now checks the market's own
  ticker-suffix first (new `_side_matches_team()` helper), falling back
  to the existing keyword scan only if that misses — purely additive,
  can only match something the old logic missed, cannot behave
  differently on anything it already matched correctly.
- New `data/mlb_match_rate_state.json`, written every run
  (attempted-vs-matched), plus two new `data_freshness.py` checks —
  `check_mlb_schedule()` (age-based, warn 20h/block 48h) and
  `check_mlb_match_rate()` (the new zero-count check type) — both wired
  into THRESHOLDS, MODEL_BLOCK_MAP, and the real Telegram alert path.

**Scoped down, on purpose**: the fourth item Gemini/the verdict
recommended — a full rewrite of `match_game_to_kalshi()` onto
ticker-suffix parsing, matching soccer's pattern exactly — was not
done. What shipped instead is ticker-suffix-first with the existing
matcher as fallback, not a replacement. Reasoning at the time: the
current matcher is confirmed working live (15/15 games across two real
runs today) with a validation checkpoint close, and a full rewrite of a
currently-working matcher carried real regression risk for marginal
gain. Worth naming plainly: that Aug 25 urgency was already stale by
the time I made the call — Gate 1 had been reset to Sept 8 the day
before. The decision still holds on its own merits (regression risk vs.
marginal gain), but I want to be honest that the reasoning I gave for
it leaned on a deadline that had already moved. Full detail:
`archie_external_review_fixes_shipped_20260823.md`.

Verified before pushing: py_compile clean both files; matcher
regression-tested against all 15 real tickers from today's production
log with titles stripped of team names (15/15 matched via ticker-suffix
alone); state-writer functions and both new severity ladders unit
tested against synthetic data; confirmed neither file had changed on
disk since last read, before overwriting.

Pushed to origin/main as commit `b485776`.

**The incident, reported straight**: restoring my second stash after
the rebase produced real merge conflicts — several tracked production
data/log files (brier_dashboard.json, sede_portfolio.json,
market_cache.json, signals_full_log.csv, signals_log.csv, and others)
got literal `<<<<<<<` conflict markers written into them, i.e. briefly
invalid JSON/CSV, while two long-running python.exe processes
(presumably auto_monitor/trade_monitor, running since 8/12 and 8/22)
could have read or written them at any time. Fixed via `git reset
--hard HEAD` to discard the conflicted working tree and restore the
clean, just-pushed state — justified by the fact that every one of
those files is already on the repo's own `SAFE_TO_DISCARD_PREFIXES`
allowlist, so this isn't a new risk policy, just applying the existing
one under pressure. Verified all JSON loads clean and both edited
modules import without error post-fix. My original safety-net stash
(`stash@{0}`) was preserved, not dropped. No data was lost that
wasn't already regenerable by design — but this was self-caused, and
I'm naming it as exactly that rather than folding it into "verification
done."

---

## THE GO-LIVE NUMBERS — SAID PLAINLY

Pulled `validation_track_record.csv` directly rather than assume:
**24 total closed/exited trades** — 10 LOST, 8 WON, 6 EARLY_EXIT. Win
rate on decided trades (8 of 18 WON/LOST) is 44.4%, below the 55%
go-live bar, and 24 total is nowhere near the 75+ required. This has
been true for a while and hasn't moved this weekend — none of this
weekend's build work (NFL, EPL) adds trades to this count, and MLB_GAME
never passed Gate 1 to begin with. Saying it plainly rather than
letting three shipped build-threads read as more validation progress
than they are: **the actual go-live gate has not moved this week.**
The original Aug 15 target has been passed with no new target set.

---

## ARCHIE'S SUGGESTIONS (given today, not yet ratified — for Rus/J@rv1s)

1. **Audit other models for the same title-text matching pattern**
   Gemini's critique of MLB's matcher — text-scanning a title instead
   of using a structured ticker field — is not MLB-specific. Worth a
   deliberate pass over every model that matches Kalshi markets by
   title/keyword (GDP, MLS_GAME, WC_GAME, Claims, CPI) to see which
   ones have a clean structured field sitting unused the way MLB did,
   before one of them goes silent for weeks the same way MLB and JOBS
   both did this month.
2. **Don't let build-thread momentum stand in for validation progress**
   — see the numbers section above. Three real things got built this
   weekend; the go-live count still didn't move. Worth being explicit
   about that distinction going into next week rather than letting the
   shipped-commit list imply otherwise.
3. Confirm tomorrow's real Oracle run before trusting either new
   `data_freshness.py` check in an actual incident (see below).

---

## VALIDATION TRACKER

24 total trades, 10 LOST / 8 WON / 6 EARLY_EXIT — 44.4% win rate on
decided trades, well short of 75+/>55%/Brier<0.20/avg edge>10c. No new
go-live target set since Aug 15 passed. JOBS remains the only model to
have ever cleared Gate 1 on paper, and per last weekend's audit that
pass is unconfirmed pending Sept 8, not a standing credential.
MLB_GAME: signals resumed (b5cb427, Aug 22) after the ~24-day per-side
matching gap (Jul 30 → Aug 22, confirmed against real
signals_full_log.csv); never passed Gate 1. GDP: scoring stall since
Jul 30, still open, untouched this weekend. NFL/EPL: pre-season/
pre-build respectively — neither contributes to this count yet.

---

## ORACLE CLOUD STATUS

Not directly SSH-checked this weekend. Indirect evidence of normal
operation: real daily-report files across the weekend's slots, normal
git push/pull activity, no freshness alerts surfaced in anything
reviewed. Tonight's push (b485776) has NOT yet been confirmed against
a real Oracle-side run — per Rus's instruction, waiting for the next
scheduled run rather than proactively checking. Two things to look for
on that run: `data/mlb_schedule_state.json` and
`data/mlb_match_rate_state.json` should appear with real timestamps,
and the DATA FRESHNESS CHECK printout should show `mlb_schedule` /
`mlb_match_rate` rows.

## OPEN POSITIONS

None flagged open this weekend across any of the reviewed material.

## SPORTS MONITORING

MLB regular season running through Sept 27, matching resumed and
verified 12/12 → 15/15 across two fixes this weekend. NFL preseason;
Week 1 starts Sept 3 — first real chance to confirm the rest-day
adjustment live, and the motivation adjustment will NOT fire in Week 1
regardless (no clinch scenarios exist yet). EPL not yet live in the
system — design locked, build pending.

---

## CARRIED FORWARD (unchanged, not touched this weekend)

- Real spread instrumentation for JOBS — never built, zero rows ever.
- Real sourced NFP_CONSENSUS — still a placeholder (+15,000).
- GDP scoring stall since July 30.
- RESOLVED_MARKETS dynamism question (from earlier carryover).
- mlb_outcome_backfill.py's 9 no-match rows (from earlier carryover).
- NFL Elo fix reconfirmed as a verified no-op as of the 8/21 21:00 run.
- Four tmp verification scripts left in the repo (cleanup, low
  priority).
- GDP out-quarter methodology question (from earlier carryover).
- Motivation-adjustment live clinch-detection feed — does not exist;
  code path present but structurally cannot fire until built.
- EPL carryover-blend build itself — design closed out, not started.

---

## FOR NEXT SESSION (priority order)

1. Confirm tonight's push (b485776) against a real Oracle run — the
   two new state files and the two new freshness-check rows. Per Rus:
   wait for the scheduled run, don't force it.
2. Decide whether to act on suggestion #1 above (title-matching audit
   across other models) before or after Sept 8.
3. Before Sept 8: real spread instrumentation for JOBS, and a real
   sourced NFP_CONSENSUS (or a documented, labeled fallback proxy).
4. Decide a new go-live target given the real numbers above — none is
   currently set, and repeating "the original one already passed"
   without deciding a new one is itself a decision by default.
5. EPL build, once bandwidth allows — design is ready and waiting.

---

Archie | Papa Ralph standard. Three build threads shipped, one real
external-review gap closed, one self-caused incident owned and fixed
in the open, and the number that actually gates go-live said plainly
instead of left to imply itself from the shipped-commit list.
