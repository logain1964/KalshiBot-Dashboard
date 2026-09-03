# SEDE Nightly Session Summary
## For J@rv1s Morning Intelligence Pull

**Last updated:** 2026-09-02 | **Session end:** ~10:45 PM CT
**Prepared by:** Archie (Claude Desktop)

Note: the previous version of this file was stale since 2026-06-18 --
this session's summary was not being regenerated for months. Restarting
the habit tonight; flagging the gap rather than quietly overwriting it.

---

## TONIGHT IN ONE SENTENCE

Found and fixed a real gap in last night's NFL_SPREAD display fix (a
ground-truth-map miss was defeating it for at least one live signal,
and the same miss was corrupting SEDE Signal Confidence scoring, not
just display text); continued the still-unresolved cron data-loss
investigation with rigorous instrumentation and an external second
opinion; ran a critical, honest UX review of the subscriber alert
format with Gemini and marked what's genuinely useful vs. what
conflicts with the existing product spec; confirmed the MLB_GAME
historical-loss concern is very likely not a systemic issue.

---

## 1. NFL_SPREAD DISPLAY FIX -- RESIDUAL GAP FOUND AND CLOSED (commit 408b627)

Last night's fix (913f0b4) closed the "P(team win)" mislabeling for
NFL_SPREAD rows, verified against synthetic tuples before shipping.
Tonight Rus pasted a real line from the actual 21:00 CT report --
"LV >2.5 [THIN MARKET]" -- still showing the old wrong text an hour
after that fix was live. Confirmed genuine (not stale) against the
saved report file, then traced it: the fix only fires when the
ground-truth `label_to_model` lookup hits; for this label it missed
and fell back to `detect_signal_model()`, which predates NFL_SPREAD
and had never been taught its label shape, so it returned "OTHER"
and defeated the fix for that row specifically.

**Shipped:** a regex branch in `detect_signal_model()` recognizing
NFL_SPREAD's label shape directly (fixes the symptom regardless of
why the map misses), plus a `[ModelType] WARNING` log on every
ground-truth map miss to surface the underlying cause on the next
real run. Root cause of the map miss itself is still open.

**Bigger than display:** `compute_sede_confidence()` shares the exact
same fallback and the same map. Any NFL_SPREAD signal hitting this
miss since Aug 22 wasn't just mislabeled -- its SEDE confidence rating
was very likely scored under the wrong model's reliability stats.
Tonight's fix resolves this too, but past NFL_SPREAD confidence
ratings since Aug 22 should be treated as suspect until spot-checked.
Confirmed (from the code, not assumed) that the actual email --
`email_alerts.py`'s `fmt_signal()` -- uses a different, tuple-content
based mechanism and was never affected by this specific gap.

Full writeup: `claude/nfl_spread_display_fix_proposal_20260902.md`
(addendum).

---

## 2. CRON DATA-LOSS BUG -- STILL UNRESOLVED, REAL PROGRESS

signals_log.csv intermittently fails to commit on cron runs (not
manual runs) with zero exceptions anywhere -- multiple recurrences
through early Sept. Ruled out (with real evidence, not assumption):
gitignore, crash, header-migration path, concurrent process, silent
truncation, git push failure, swallowed exceptions.

Diagnostic checkpoints (start / pre_logging / pre_push) added this
week evolved twice tonight: added print breadcrumbs on entry/success
(commit 4cc666a) after the Sept 2 21:00 CT run showed only the
"start" checkpoint's write actually landing on disk -- pre_logging
and pre_push never wrote at all, despite proof execution passed both
points and zero exceptions anywhere. Then replaced buffered
`open().write()` with raw `os.open`/`write`/`fsync` plus an
independent re-read verification and inode/device identity logging
(commit 7fb9631), since buffered I/O couldn't be fully ruled out
as an explanation even though the timing already argued against it.

Also wrote up the investigation anonymized and had it reviewed by
Gemini as a second opinion -- most of the suggestions were generic
incident-response boilerplate, but a few were genuinely checkable
(git hooks, `os.chdir()` search) and were checked: Oracle's git hooks
directory is empty (confirmed directly), ruling that out.

**Not resolved tonight.** Next real data point needs the 07:00 CT
Sept 3 cron cycle -- specifically whether the new print breadcrumbs
show `pre_logging`/`pre_push` "starting" without "OK" (write itself
failing silently) or not printing "starting" at all (call site
somehow not reached, which would contradict everything confirmed so
far). Rus checking this directly when it lands.

---

## 3. SUBSCRIBER ALERT FORMAT -- HONEST UX REVIEW, NO CODE CHANGES

Rus asked directly: would a newcomer to prediction markets understand
tonight's alert output? Honest answer: no -- "BUY NO," cents pricing,
and "Edge" are all unexplained jargon, and (until fix #1 above) the
NFL_SPREAD row was actively telling even an experienced reader the
wrong thing. This is a known, already-spec'd gap -- `sede_product_spec_v1`
explicitly bans raw edge_cents/model probabilities from ever reaching
subscribers (P2, Subscriber Signal Formatter, BLOCKING, not yet built).
A partial plain-English fix ("Will HOU win? -- model favors LV")
shipped a while back but only for NFL_GAME/MLB_GAME, never extended to
NFL_SPREAD/GDP/JOBS -- which is why tonight's report is a mix of clear
and cryptic rows.

Rus then had Gemini review the same output and asked for a critical
read, not a rubber stamp. Marked up and saved
(`claude/subscriber_format_ideas_gemini_20260903.md`):

- Gemini independently re-derived the exact NFL_SPREAD bug from fix #1
  above, from the text alone -- a real outside confirmation.
- Genuinely new and worth keeping for P2: generic probability labels
  tied to the market's own question (would make this whole bug class
  structurally impossible, not just patched per model); a warning that
  the 6-7 GDP threshold positions are correlated, not independent bets
  (confirmed real -- see Open Positions below, this isn't hypothetical);
  tying "thin market" warnings to real bid/ask/size data, which lines
  up directly with the Sept 8 spread stress-test work already in
  flight.
- Real pushback: most of Gemini's mockups show exact model
  probabilities and raw cents-edge to the reader, which
  `sede_product_spec_v1` explicitly bans for subscribers. Flagged so
  P2 doesn't get built off those mockups by mistake.

---

## 4. MLB_GAME HISTORICAL LOSS CONCERN -- CHECKED, LIKELY NOT SYSTEMIC

Carried-forward ask from J@rv1s: was the MLB_GAME cron-loss pattern
longstanding? Git history shows 15 consecutive clean days before the
incident window started -- doesn't look like a longstanding systemic
issue, more consistent with something that started recently (see #2).

---

## OPEN POSITIONS (sede_portfolio.json, live-read tonight)

8 open positions, all pre-dating the Aug 11 Gate 1 suspension --
no new entries since (correct, trading is suspended pending Sept 8):

| Market | Model | Direction | Entry | Current |
|--------|-------|-----------|-------|---------|
| BTC<$50k Dec31 | BTC | NO | 43.5c | 81.5c |
| GDP >1.5% (Oct30) | GDP | YES | 70.5c | 76.0c |
| GDP >1.5% (Jan28) | GDP | YES | 74.5c | 65.5c |
| GDP >2.5% (Oct30) | GDP | YES | 58.0c | 54.0c |
| GDP >1.0% (Jan28) | GDP | YES | 78.0c | 77.5c |
| GDP >2.0% (Apr29) | GDP | YES | 50.5c | 53.5c |
| GDP >4.0% (Oct30) | GDP | YES | 18.5c | 11.5c |
| GDP >1.5% (Jul29) | GDP | YES | 59.5c | 61.5c |

Worth flagging plainly: this is exactly the correlated-exposure
pattern Gemini's review called out (item #3 above) -- 7 of 8 open
positions are GDP threshold bets on the same underlying quarterly
GDP outcome, not independent trades. Not a new problem tonight, but
real, and worth a look whenever GDP position sizing gets revisited.

---

## COMMIT LOG (tonight, 2026-09-02)

| Commit | Description |
|--------|-------------|
| 408b627 | Fix NFL_SPREAD residual gap: detect_signal_model() fallback + [ModelType] WARNING diagnostic |
| 7fb9631 | Diag checkpoint v3: raw os.open/write/fsync + independent re-read + inode/dev identity |
| 4cc666a | Diag checkpoint: print breadcrumbs on entry/success, not just failure |
| 8260acb | Auto-update 2026-09-02 21:00 CT (clean cycle -- first since the cron data-loss bug started) |
| 324910d | Add cron-vs-manual env diagnostic instrumentation (v1) |
| 3343958 | Merge branch 'main' |
| 913f0b4 | Fix NFL_SPREAD outcome_desc misprinting as team-win moneyline text |
| 91efc60 | Auto-update 2026-09-02 11:15 CT (data-loss recurrence) |
| 5f1ecd2 | Auto-update 2026-09-02 07:00 CT |

---

## SYSTEM STATUS

| Component | Status |
|-----------|--------|
| GATE 1 (project-wide) | SUSPENDED since Aug 11 -- pending real bid/ask-spread stress test, checkpoint Sept 8 |
| CLAIMS | SUSPENDED -- never trade |
| MLB_GAME NO | SUSPENDED -- YES direction experimental tier only |
| GDP | REDUCED WEIGHT (0.60) -- scoring stalled since Jul 30, root cause open |
| JOBS | CAVEATED -- statistically passed original Gate 1 but with material caveats, not unconditionally validated |
| NFL_SPREAD display bug | FIXED tonight (408b627), both display and confidence-scoring paths |
| Cron data-loss bug | UNRESOLVED -- v3 diagnostic live, next data point needs 07:00 CT Sept 3 |
| Subscriber format (P2) | Not built -- design ideas captured tonight for when it is |
| MLB_GAME historical concern | Checked -- likely not systemic (15 clean days pre-incident) |
| Oracle 21:00 CT cron | Clean -- first clean cycle since data-loss bug started |

---

## J@rv1s MORNING ACTIONS (ordered)

1. **Check for `[ModelType] WARNING` / `[Diag]` lines** in the next
   real report (07:00 CT Sept 3 or later) -- this is the actual
   evidence for both open investigations (#1 map-miss root cause,
   #2 cron data-loss). Don't let these get buried by other log noise.

2. **NFL_SPREAD rows** -- spot-check that today's live report shows
   correct spread-cover language, not "P(team win)," on any NFL_SPREAD
   signal. The fix should hold now regardless of ground-truth map
   misses, but verify against real output, not just trust the fix.

3. **Sept 8 checkpoint** -- GATE 1 stays project-wide suspended until
   then. Don't treat any model's prior Gate 1 pass as confirmed,
   JOBS included, per standing instructions.

4. **Subscriber format ideas** -- no action needed now, just aware:
   `claude/subscriber_format_ideas_gemini_20260903.md` has marked
   ideas for whenever P2 gets prioritized, including a real, live
   example of the GDP correlated-exposure problem (see Open Positions).

5. **This file** -- was stale since June 18. If it goes stale again,
   flag it rather than quietly working around a missing summary.

---

## VALIDATION TRACKER

| Model | Status |
|-------|--------|
| GATE 1 (project-wide) | SUSPENDED since Aug 11, checkpoint Sept 8 |
| CLAIMS | SUSPENDED |
| MLB_GAME | NO suspended; YES experimental tier only |
| GDP | REDUCED WEIGHT (0.60), scoring stalled since Jul 30 |
| JOBS | Caveated -- n=73, 58.9% WR, Brier 0.134, but real caveats (see project instructions) |
| NFL_SPREAD | Display + confidence-scoring bug fixed tonight |

---

Archie | Papa Ralph standard. If it's worth doing it's worth doing right.
