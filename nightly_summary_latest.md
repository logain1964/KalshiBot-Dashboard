# Nightly Summary — 2026-09-01 (Archie → J@rv1s)

## ORACLE CLOUD STATUS
Oracle is current with tonight's two code fixes (6f8d230) and the new
stash-guard tool (66d1765) -- both pushed, Rus can pull whenever
convenient (no urgency, code-only, no data dependency). Separately,
see VALIDATION TRACKER below for a real, serious finding about
Oracle's own automated pipeline.

## OPEN POSITIONS
paper_trades.json: 1 open, Trade #25 (GDP > 3.5%, YES, 21c, edge
61.2c). Unchanged, verified intact tonight.
sede_portfolio.json: 8 open (7 GDP, 1 BTC), unchanged, verified intact
tonight (including through a real git pull -- no incident this time).

## VALIDATION TRACKER
- Gate 1 still project-wide suspended, Sept 8 checkpoint now 7 days
  out.
- **Real, serious finding tonight: Oracle's automated pipeline
  silently loses signals_log.csv's fresh data on every scheduled cron
  run, confirmed reproducible 3-for-3 today (07:00, 11:15, 21:00 CT).**
  The GDP spread-capture data generates correctly on Oracle's disk
  every cycle (confirmed via Oracle's own real report text) but never
  reaches anywhere retrievable -- ruled out a crash, a migration path,
  a concurrent process, gitignore, and a push failure, one at a time,
  with real commands against the real system. A manual, by-hand
  invocation of the identical script worked perfectly and committed
  everything correctly -- meaning this is a cron-specific environment
  difference, not an application bug in the fix itself. Root cause not
  yet found; next step (tomorrow) is an environment dump to diff a
  real cron cycle against a manual run.
  **Real number, not a calendar estimate:** exactly ONE cycle's worth
  of real GDP spread data has permanently reached GitHub since the
  feature shipped last night -- 7 observations, from tonight's manual
  run. Everything from today's three real scheduled cycles was
  generated and lost. This is now the top blocker for Sept 8 having
  any real GDP spread data at all.
- Real Oracle cron cadence reconfirmed via crontab -l: 07:00/11:15/
  21:00 CT exactly, no discrepancy.
- Alert-formatting: both of today's two flagged discrepancies traced
  to root and fixed (see WORK ORDER below).

## TONIGHT'S WORK ORDER
1. **Fixed the stale "1/5" position-limit display.** Root cause:
   email_alerts.py's live daily-digest function hardcoded "/5" while
   every real entry gate (daily_runner.py, trade_monitor.py,
   position_manager.py) was already correctly using 8 since June 14.
   Not a trading-impact bug -- display only. Now reads the real
   constant. Verified via a live import test and py_compile.
2. **Fixed "named outcome" printing on every GDP row.** Confirmed
   pre-existing (predates last night's spread-capture build by
   months, not a side effect of it) -- GDP's tuple was always too
   short/type-mismatched to reach the team-name branches, before or
   after last night's change. GDP's own label (e.g. "GDP > 4.0%")
   now substitutes in directly instead of the placeholder. Verified
   against synthetic GDP/NFL_GAME/MLB_GAME/other-model tuples --
   no regression to team-name display for the other models.
3. **Built and tested a real stash hard-exclusion guard**
   (tools\safe_stash.ps1), replacing last night's documented-reminder
   fix per your correct pushback that a reminder doesn't survive time
   pressure. Refuses by construction to stash paper_trades.json or
   sede_portfolio.json unless explicitly overridden. Tested against a
   throwaway repo and the real one before shipping.
4. **NFL_SPREAD + mlb_refresh.py: real commitment, not just "ranked
   #1."** Not tonight -- proposal + build scheduled for tomorrow
   evening, same process as the GDP spread-capture doc.
5. **The Oracle cron data-loss discovery above** -- the largest single
   finding tonight, found while tracing your sample-count ask.

## PENDING, RANKED MOST TO LEAST URGENT (for tomorrow)
1. Oracle cron silently loses signals_log.csv on every scheduled run
   -- confirmed reproducible, root cause not yet found, directly
   threatens Sept 8's real GDP spread data.
2. NFL_SPREAD display gap + mlb_refresh.py's copy -- scoped and built
   tomorrow evening.
3. "Positional-tuple type inference" pattern naming -- still needs a
   real FORGE pass, unchanged.
4. Uncommitted housekeeping (Tier 1/2 scripts, docs, 2 stray files) --
   untouched tonight.
5. SEDE Signal Confidence showing zero NFL rows despite qualifying
   edges -- your own read (composite score math) sounds plausible but
   wasn't independently traced against real code tonight.

## SPORTS MONITORING
Nothing open in either ledger depends on live game monitoring.

---
Archie | Papa Ralph standard.
