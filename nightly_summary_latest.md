# Nightly Summary — 2026-09-24 (Archie)

Rus skipped straight to item 6 from your Sept 23 briefing, then item 10
("see if it helps tomorrow's session"), explicitly deferring JOBS
reconciliation plus items 7/8/9 to tomorrow. Real bug found and fixed
along the way (direction/accuracy inversion, below). Session closed with
an unresolved 20-file git merge conflict left for tomorrow on purpose —
Rus's call, nothing lost, stash preserved. Local commit tonight: 642a6be
(NOT yet pushed to origin or Oracle — see Oracle status below).

## Item 6 — CPI backfill (Part A shipped, Part B referred to you)
CPI had zero resolved signals in its entire history before tonight.
Sourced the real May 2026 CPI YoY release from the BLS archive (4.2%
actual) and added the >3.9/>4.1/>4.2/>4.3/>4.4 MAY threshold ladder to
signal_scorer.py's RESOLVED_MARKETS, following the existing
exact-tie-resolves-NO convention. **Part B — the June 10-Sept 21
historical dead window — is explicitly referred to you** (Rus approved
the split rather than Archie building it): same shape of work as the
MLB/MLS backfills, needs sourcing decisions you're better positioned to
make.

## Real bug found: direction/accuracy inversion (4 files, 5 locations)
Honest process note: I misdiagnosed this once (thought signal_scorer.py
had the bug, backwards) and caught it myself before shipping anything, by
re-reading run_scorer()'s own comment — `actual_outcome` in
signals_log.csv is already direction-baked ("1 if the signal won"), not
raw "did YES win."

Real fix: brier_dashboard.py (model_breakdown, spread_stress_analysis,
edge_bucket_analysis, print_report) and daily_runner.py
(get_mlb_game_direction_stats) were all re-deriving "correct" as
direction+outcome, double-applying direction on an already-direction-baked
value — inverting accuracy and spread-stress P&L sign for every
NO-direction signal. Fixed to the simple `outcome == 1` check in all 5
places.

**Verified against real data, not syntax:** before the fix, only 1 of 9
models (CLAIMS — the one all-YES model) showed matching accuracy between
signal_scorer.py's console report and brier_dashboard.py. After the fix,
all 9 match exactly.

**Flag for you, unresolved:** JOBS's original Gate 1 pass (n=73, 58.9%
WR, Brier 0.134) needs reconciliation against this fix. I have not yet
checked whether JOBS's sample includes NO-direction signals — only those
would be affected. JOBS already carried other real, independent caveats
(84% of sample against placeholder NFP consensus; ~2 months blind to real
Kalshi markets) — this is an additional, separate question, not
necessarily compounding.

Also part of tonight's commit (built earlier this session, folded in
here): `dedup_by_real_event()` unifies what were two independently-
maintained dedup algorithms (signal_scorer.py + brier_dashboard.py) into
one shared function. Verified byte-identical against real data (786/786
events matching).

## Item 10 — multica-ai/andrej-karpathy-skills CLAUDE.md
Tested a real platform question rather than assuming: does Claude
Desktop's Chat surface auto-load a project's CLAUDE.md the way Claude
Code's Code tab does? Confirmed via `anthropics/claude-code#30530` (open
GitHub issue) — no. Built C:\KalshiBot\CLAUDE.md (76 lines, the 4
Karpathy-derived sections verbatim: Think Before Coding, Simplicity
First, Surgical Changes, Goal-Driven Execution) and updated the "Archie
Session Protocol" project doc to require reading it explicitly every
session opener. No verdict yet on whether it changes anything — an
experiment per Rus's own framing, not a claimed win.

## Oracle Cloud status — NOT yet updated tonight
**Oracle has not received any of tonight's fixes.** Local branch is
`main...origin/main [ahead 2]` (commit 642a6be + a merge commit), both
unpushed. A pre-pull stash pop conflicted in 20 data/log files (git
status confirms this right now — correction to my own earlier in-session
estimate of 19). Rus's explicit call: hold the conflict, don't resolve
tonight — nothing is lost, the stash is preserved
(`stash@{0}: "pre-pull stash before pushing dedup/CPI/accuracy-bug fixes
20260924"`). **Tomorrow's first priority has to be resolving this
conflict and pushing**, or Oracle keeps running the old dedup logic, the
old direction-inversion bug, and zero CPI resolved signals indefinitely.

## Validation / Gate 1 status
Still project-wide provisionally suspended since Aug 11 — unchanged.
CPI now has its first-ever resolved signal (see above), so its Gate 1
sample is no longer permanently empty, though still far short of n≥30.
JOBS's original n=73/58.9%/Brier 0.134 pass needs the reconciliation
flagged above before being cited as unaffected by tonight's fix.

## Open positions (verified against data/paper_trades.json — reliable;
data/sede_portfolio.json is currently mid-merge-conflict and should NOT
be cited for figures until tomorrow's resolution)
Exactly one open position: Trade #25, GDP > 3.5% (Q3 2026 advance
estimate), YES @ 21c, 119 contracts, model 82.2% vs Kalshi-implied 21%
(61.2c edge), max loss $24.99 / max profit $94.01, resolves 2026-10-30.

## Sports monitoring
None required — the one open position is a GDP macro release, not
sports-tied.

## Carried forward — ranked, most urgent first
1. Resolve the 20-file git merge conflict in C:\KalshiBot, then push
   642a6be + merge to origin and hand Oracle the pull command — blocks
   everything else below reaching Oracle.
2. Reconcile JOBS's n=73/58.9%/Brier 0.134 against tonight's
   direction/accuracy-inversion fix.
3. CPI Part B — JUN/JUL/AUG historical backfill (referred, above).
4. Item 7 — mlb_model.py park-factor gap (after MLB_GAME season ends
   Sept 27).
5. Item 8 — WC_GAME's 16:1 raw-to-event dedup ratio, still unverified.
6. Item 9 — try HypothesisWorks/hypothesis on real_event_id()/
   qualifies().

Archie | Papa Ralph standard.
