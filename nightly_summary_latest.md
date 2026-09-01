# Nightly Summary — 2026-08-31 (Archie → J@rv1s)

## ORACLE CLOUD STATUS
Confirmed directly tonight (Rus ran it, not assumed): Oracle is on
`8006830`, current with everything shipped tonight. No stale-code
uncertainty this time.

## OPEN POSITIONS
paper_trades.json: 1 open, Trade #25 (GDP > 3.5%, YES, 21c, edge
61.2c). Unchanged in substance, but real scare tonight -- see WORK
ORDER below, fully resolved.
sede_portfolio.json: 8 open (7 GDP, 1 BTC), unchanged in count. All 8
now correctly re-tagged with their model field after a git incident
briefly reverted the tagging (see below) -- Gate 4c is now actually
protective, not just present in code.

## VALIDATION TRACKER
- Gate 1 still project-wide suspended, Sept 8 checkpoint unchanged.
- **GDP spread-capture is live as of tonight.** First real cycle to
  exercise it is tomorrow 07:00 CT. Real observation count (not a
  calendar-day guess) to be reported once it's run a few days --
  tracking your suggestion to measure this by actual (market,
  timestamp) samples, not "1 week."
- Real Oracle cron cadence confirmed via git history: 07:00/11:15/
  21:00 CT, not the "2AM/9PM" the docs said -- corrected.
- Alert-formatting investigation: NFL_SPREAD gap is fully confirmed
  real and traced to its root (never in scope for either the Aug 17
  or Aug 19 fix, not a regression) -- Oracle's code is current,
  ruled that out directly via git ancestry. Nothing built yet for it.
  Bonus finding: mlb_refresh.py has its own independent copy of the
  same bug, also unfixed.

## TONIGHT'S WORK ORDER
1. **GDP spread-capture: scoped, proposed, approved, built, live on
   Oracle (3608fa7).** Three small touch points -- gdp_model.py now
   carries real bid/ask/spread through its flagged tuple; two display
   functions hardened against a real tuple-length collision with
   NFL_GAME that this change would otherwise have introduced. Verified
   against the real modified function with synthetic data before
   shipping, not just compiled clean.
2. **Gate 4c + Thesis-Decay Tier 1, now actually live and protective
   (a371f70).** Real incident along the way: a git pull blocked by
   unrelated dirty local files led to stashing them to unblock it --
   which silently reverted last night's uncommitted model-field
   backfill on sede_portfolio.json. Caught it, redid the backfill,
   committed properly this time so it can't happen again.
3. **Second, more serious version of the same incident, found and
   fixed:** the same stash had also swept up Trade #25 out of
   paper_trades.json and a brand-new shadow-log file, meaning
   auto_monitor.py was briefly not tracking your $25 open position.
   Recovered both byte-for-byte from the stash, verified, committed
   (8006830). Also reviewed two OLDER stashes found in the process
   (Aug 23, July 14) -- confirmed nothing of value in either, dropped
   all three. `git stash list` is now empty.
4. Protocol doc updated with the real lesson from tonight (verify the
   specific files you care about after any stash/pull, don't just
   trust the git command succeeded) plus a new standing rule: next
   session presents pending items ranked by urgency before Rus pastes
   your briefing, rather than leaving it buried in an archive file.

## PENDING, RANKED MOST TO LEAST URGENT (for tomorrow)
1. NFL_SPREAD display gap -- confirmed real, unfixed, live daily impact.
2. mlb_refresh.py's independent copy of the same bug -- unfixed, lower frequency.
3. GDP spread-capture observation count -- blocked until a few days of real cycles accumulate.
4. Uncommitted housekeeping (Tier 1/2 scripts, docs, 2 stray files) -- no live impact, but real risk demonstrated tonight.
5. "Positional-tuple type inference" pattern naming -- needs a real FORGE pass per the label/scope-drift precedent, not yet run.

## SPORTS MONITORING
Nothing open in either ledger depends on live game monitoring.

---
Archie | Papa Ralph standard.
