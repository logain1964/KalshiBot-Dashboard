# Nightly Summary — 2026-09-02 / 2026-09-03 (Archie → J@rv1s)

Covers two sessions in one -- last night's summary got written to the
wrong file (nightly_summary.md instead of this one, now corrected)
and never reached you. Combining both nights here rather than leaving
Sept 2 undocumented.

## ORACLE CLOUD STATUS
Current through commit 6c1a191 (pushed tonight, ~7:15 PM CT). Tonight's
21:00 CT cron cycle hasn't fired yet as of this writing -- it's the
first real test of tonight's fix below. Sept 2's 21:00 CT and both of
today's cycles (07:00, 11:15 CT) ran clean, no data loss.

## OPEN POSITIONS
sede_portfolio.json: 8 open (7 GDP, 1 BTC), all pre-dating the Aug 11
Gate 1 suspension, no new entries (correct -- trading still
suspended). Worth flagging plainly: 7 of 8 are GDP threshold bets on
the same underlying quarterly outcome, not independent trades --
correlated exposure, not new risk, but real and worth a look
eventually.
paper_trades.json: 1 open, unchanged.

## VALIDATION TRACKER
- Gate 1 (project-wide) still SUSPENDED, Sept 8 checkpoint now 5 days
  out.
- CLAIMS suspended, MLB_GAME NO suspended (YES experimental tier
  only), GDP reduced weight (0.60, scoring stalled since Jul 30), JOBS
  caveated (not unconditionally validated) -- all unchanged.
- GDP spread-capture (shipped Sept 1) confirmed genuinely working when
  it gets to run: 28 real GDP rows now carry real bid/ask/spread data,
  up from 7 a few days ago. The gap between those numbers is explained
  by the cron data-loss bug below -- most cycles' worth of real spread
  data was being silently discarded before today's fix.

## SEPT 2 WORK ORDER (the summary that didn't reach you)
1. **NFL_SPREAD residual gap found and fixed (908b627... commit
   408b627).** Last night's display fix had a gap: a ground-truth
   model-lookup miss for one live label fell back to a guesser that
   didn't know NFL_SPREAD's format, defeating the fix for that row.
   Fixed the guesser directly and added a warning log for the
   underlying miss. Bigger than display: `compute_sede_confidence()`
   shares the same fallback, so affected signals were likely scored
   under the wrong model's reliability stats too, not just mislabeled.
   Confirmed the actual email (fmt_signal) uses a different mechanism
   and was never affected.
2. **Cron data-loss diagnostic hardened twice** (breadcrumbs, then raw
   os-level write+fsync+independent-reread+inode identity) after the
   Sept 2 21:00 CT run showed pre_logging/pre_push checkpoints never
   landing on disk despite zero exceptions.
3. **Subscriber alert format -- honest UX review, no code changes.**
   Confirmed a newcomer would be confused by the current report
   (unexplained jargon, and the NFL_SPREAD bug above was actively
   telling readers the wrong thing). Had Gemini review it too --
   marked genuinely useful ideas (generic probability labels, GDP
   correlated-exposure warning, tying thin-market flags to real
   spread data) vs. ideas that conflict with the existing subscriber
   spec (showing raw model %/edge cents, which sede_product_spec_v1
   explicitly bans). Saved for whenever P2 gets built.
4. **MLB_GAME historical loss concern checked** -- 15 consecutive
   clean days before the incident window started, likely not a
   longstanding systemic issue.

## SEPT 3 WORK ORDER
1. **Likely root cause of the cron data-loss bug found and fixed
   (commit 6c1a191).** `data_freshness.py`'s `auto_pull_if_safe()`
   runs at the start of every cron cycle and silently `git checkout
   --`s any file in a "safe to discard" list that shows as locally
   modified. That list included `data/signals_log.csv`,
   `data/signals_full_log.csv` -- the exact files this investigation
   has been chasing -- plus **`data/sede_portfolio.json` and
   `data/paper_trades.json`, live trading state**. If one run's own
   commit ever stalled or two runs landed close together, the next
   run's auto-pull would see the other run's real, uncommitted new
   data as routine dirt and wipe it, logged as expected behavior --
   matches every symptom of the original bug exactly. Those two
   position-tracking files are the same ones `safe_stash.ps1` already
   refuses to touch on the laptop without an explicit override; Oracle
   had no equivalent guard. Removed 7 append-only/live-state files
   from the discard list (they now skip-and-warn instead of silently
   discarding); verified via isolated classification test before
   shipping. Does not guarantee the bug is fully closed -- there could
   be more than one contributing mechanism -- but it's the strongest,
   most mechanistically clean lead this investigation has produced,
   and it was an active, ongoing risk to real portfolio data
   regardless of whether it's the whole story.
2. **A separate, still-unexplained diagnostic oddity found while
   chasing the above.** Both of today's cron cycles showed the "start"
   checkpoint firing a second time mid-run, under the identical
   os.getpid() (unspoofable -- proves one continuous process), but
   missing fields only an older version of the checkpoint function
   would omit. Ruled out duplicate cron entries, wrapper scripts,
   duplicate call sites, self-relaunch code, and file drift on Oracle
   -- all confirmed directly, not assumed. Added a file-hash +
   code-object-identity probe to catch it directly next time rather
   than guessing further. Likely unrelated to fix #1 above.
3. **"SEDE restart vs harden FORGE outcome" carry-forward item --
   investigated, found to be a phantom.** Searched the full local
   filesystem, the claude.ai project (twice), commit messages in both
   git repos, and the actual historical content of every past
   `jarvls_daily_briefing`. No trace anywhere, and Rus doesn't recall
   it either. Recommend dropping this from future carry-forward lists
   -- there's nothing to review.
4. This file itself was stale since June 18 under a different
   filename (`nightly_summary.md`) that nothing was actually reading;
   this is the file that's been current. Worth checking which of the
   two J@rv1s is actually configured to fetch, since the DASHBOARD
   URLs in the project's operating instructions point at
   `nightly_summary.md`, not this one.

## PENDING, RANKED MOST TO LEAST URGENT (for tomorrow)
1. Confirm tonight's 21:00 CT cycle ran clean and the discard-list fix
   held -- the real test of item 1 above.
2. Chase the double-start diagnostic oddity if it recurs with the new
   probe attached.
3. Confirm which nightly-summary filename J@rv1s's morning routine
   actually fetches, and fix the project instructions if it's the
   wrong one.
4. Untracked WIP files sitting in the repo (paper_trade_candidates.py,
   paper_trade_gates.py, paper_trade_shadow_logger.py, four
   model_integrity docs from Aug 30-31) -- status unconfirmed, waiting
   on Rus.
5. "Positional-tuple type inference" pattern naming -- still needs a
   real FORGE pass, unchanged for several sessions now.

## SPORTS MONITORING
Nothing open in either ledger depends on live game monitoring.

---
Archie | Papa Ralph standard.
