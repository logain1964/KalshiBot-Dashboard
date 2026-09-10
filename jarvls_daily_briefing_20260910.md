# J@rv1s Daily Briefing — 2026-09-10 (EOD)

## TIME-SENSITIVE — NFL_GAME OPTION C FIX, IDEALLY BEFORE TONIGHT'S KICKOFF

Real urgency here, not a routine priority item. Tonight's game and
Sunday Night Football both kick off ~7:15-7:20 PM ET/CT — the same
timing that caused the season-opener contamination (9PM CT scheduled
run firing mid-game, comparing a stale pregame number against an
already game-informed live market price). If Option C (suppress the
*alert* for in-progress games, still log the signal for later scoring)
doesn't ship before tonight, we should expect a second live example of
the same misleading-edge problem tonight, and a third Sunday night.
Sunday's early/afternoon games (most of the 15-game slate) should
mostly finish before 9PM CT regardless, so the real risk is
specifically the two prime-time-style games. If Option C ships in time,
tonight becomes a real, concrete "did it work" test rather than a
hypothetical.

## RESTART-VS-HARDEN — CLOSED, CORRECTION OWNED

Archie's independent read (ratify "harden, don't restart," widen scope
to the storage-format/header-drift bug class that actually caused the
worst incidents, rather than the tuple-arity issue the original
document focused on) is accepted. Real correction to my own Sept 1
analysis, not just an update — noted plainly rather than glossed over.
Good outcome; the independent-read requirement worked as intended.

## CHECK — log_all_signals() BLAST RADIUS (SKIP IF ALREADY ANSWERED)

Still open as of this morning, unless already resolved and I just
don't have visibility into it: does the `log_all_signals()`
header-drift fix touch the shared writer path used by every model, or
only the new MLS_GAME-specific path? If shared, every model's
signal-writing behavior changed the night of the rebuild, not just the
one suspended model described as "migrated as proof-of-concept" — a
materially bigger blast radius than the summary implied. Given this
project's real history with header-drift bugs specifically (three
prior incidents — the exact class this rebuild exists to kill), I'd
want confirmation this was verified with synthetic writes across
multiple model types before calling it fully trusted, same standard as
the Sept 6/7 corruption repair. If already checked and confirmed,
Archie should just note "already confirmed, see [wherever]" and move
on rather than redo settled work.

## NFL_GAME LIVE-STATE GAP — MY READ, FOR RELAY

Already given in full earlier today, restating for the record: Option
C first and fast (see time-sensitive item above), Option A (a real
game-already-started gate, MLS_GAME-style) as the proper permanent
fix once there's time to do it right, Option B (genuine live
win-probability updating) shelved outright — a fundamentally different
model than what NFL_GAME is scoped to be, deserving its own scope and
its own Gate 1 track if ever pursued, not a graft-on. Two follow-up
asks: (1) verify what data source `game_status.fetch_game_status`
actually queries before assuming the MLS_GAME pattern ports cleanly to
NFL, since the doc doesn't confirm NFL coverage/latency; (2) don't
exclude post-kickoff signals from Gate 1 sample outright — add an
explicit `generated_pregame: true/false` field so scoring can be run
both ways, turning any gap between "all signals" and "pregame-only"
into its own useful diagnostic instead of discarded data.

## NEW — MLB_GAME BRIER/WIN-RATE GAP, REAL DIAGNOSTIC ASK

Real, answerable question, not a permanent mystery: MLB_GAME's 62.4%
win rate (passes) alongside 0.2305 Brier (misses 0.20) is the classic
signature of overconfidence — being right often enough to clear the
win-rate bar, but confidently wrong when it misses, which Brier
punishes quadratically while win rate doesn't care about confidence at
all. This is directly diagnosable with the same method that already
worked once this month: the MLS_GAME confidence-bucket breakdown (n=49
at the time) that confirmed the Dixon-Coles overweighting hypothesis
with real numbers. Ask for tonight: run the same bucketed breakdown on
MLB_GAME's real n=112 sample (a notably better sample size than
MLS_GAME had) — check whether the miss concentrates in high-confidence
buckets (calibration drift), a specific scenario pattern like
MLS_GAME's favorite/underdog split (a nameable bug or bias), or spreads
evenly (pointing to something else, like a scoring bug). Cheap to run
against data already sitting in `load_scored_signals()`. Given MLB
season ends Sept 27 and no active work is underway to change the
model's calibration regardless, this won't change the Gate 1 outcome —
but it answers a real, currently-unresolved "why" before the season
closes the book on it, rather than leaving it as a permanent unknown.

## CARRIED FORWARD, UNCHANGED

- auto_monitor.py's backwards "loss" label (still reads negative for a
  real gain) — trivial, fix whenever there's a spare moment.
- Worth writing the checksummed-base64-chunk large-file-paste recovery
  pattern into the Archie Session Protocol doc explicitly, given the
  Sept 9/10 corrupted-paste incident — so it's the default next time
  rather than rediscovered.
- Whether/when to migrate any model besides MLS_GAME onto
  `SignalRecord` — deliberately undecided, foundation only, no action
  needed yet.
- `polymarket_monitor.py` disposition, 429-retry log visibility, MLB
  Track B unpack error, GDPNow anomaly verification, motivation-
  adjustment clinch feed, SharpAPI pagination live-boundary test — all
  still on record, untouched.

## VALIDATION TRACKER

MLB_GAME: n=112, 62.4%/0.2305 — real diagnostic ask above, but the
Gate 1 outcome itself (narrow Brier miss) is very unlikely to change
before Sept 27 regardless of the diagnostic's answer, since no active
recalibration work is underway. Season end remains the committed real
final checkpoint for this model — expect a closed verdict, almost
certainly a Brier fail, with real execution-cost data behind it,
rather than a pass. MLS_GAME: hard-exclusion fix deployed, doesn't
retroactively alter the Sept 3 suspended verdict (39.1% WR, n=151).
NFL_GAME: suspended, live-state gap being addressed per above. GDP's
one open paper trade (#25) still sitting at a real unrealized gain.
Gate 1 project-wide suspended, Oct 1 fallback date stands, untouched.

---
J@rv1s | Papa Ralph standard.
