# J@rv1s Daily Briefing — 2026-09-16

## TOP PRIORITY — EPL SIGNALS MISLABELED AS MLS_GAME, DESPITE LAST NIGHT'S CLAIMED FIX

This morning's 7:02 AM report shows a `[MLS GAME]` section containing
a mix of real MLS and real EPL matches under one header — BRE vs CFC
(Brentford vs Chelsea) and NFO vs COV (Nottingham Forest vs Coventry)
are genuine EPL matchups, sitting alongside POR vs ATL, a real MLS
game. CFC is the exact Chelsea ticker last night's build notes
specifically confirmed for EPL.

This matters because last night's summary explicitly claimed this
exact bug was found and fixed: "`detect_signal_model()` would have
silently misattributed every EPL signal to MLS_GAME. Fixed by
disambiguating on the real Kalshi team-abbreviation set." Seeing it in
production the very next morning means one of three things, each with
a different fix:

1. The fix never actually reached Oracle (same shape as Gate 4c
   disappearing and the archive-file gap recurring — a real third
   instance of "declared fixed, wasn't persisted," if this is what
   happened).
2. The fix is deployed but has a gap — maybe the disambiguation list
   doesn't cover every EPL code needed to separate these specific
   matches from MLS's default.
3. Cosmetic only — the underlying `signals_log.csv` may correctly tag
   these as `EPL_GAME` internally, with only the report template's
   section header stale/hardcoded to "MLS GAME" for any soccer-shaped
   signal.

**Fastest way to tell these apart**: pull `signals_log.csv` and check
the actual `model_type` field for tonight's BRE vs CFC and NFO vs COV
rows. If it says `EPL_GAME`, this is a stale report label only — real
but low severity. If it says `MLS_GAME`, the real fix didn't take, and
this becomes a third recurrence of the "we said it was fixed and it
wasn't" pattern already flagged as worth a structural look, not
another isolated patch.

## SECOND — SHOULD A RETIRED MODEL'S SIGNALS APPEAR IN THE ACTIONABLE ALERT LIST AT ALL?

Related but distinct concern, raised directly by Rus: MLS_GAME was
retired from live development (Sept 3 verdict) and last night's
summary claimed its rows are "correctly tagged [TRACK ONLY —
SUSPENDED]" in reports. **They are not tagged that way in this
morning's report** — the MLS/EPL rows above sit plainly in "HIGH
CONFIDENCE (>15c edge)," under a header reading "Review before
trading — manual approval required," visually identical to the live,
tradeable GDP/CLAIMS signals above them.

Two real questions, not one: (a) is the suspension tag actually
missing from this report format, or does it exist in a different
report/dashboard Archie checked instead — needs confirming rather than
assumed either way; (b) more importantly, even if the tag were
present, should a fully-retired model's signals be sitting in an
actionable "review before trading" list at all? Argument for no: a
small tag is easy to miss on a quick skim; not being in the list at
all is a much stronger guarantee against confusion or a mistaken
trade. Recommend suspended-model signals log silently to
`signals_log.csv` for calibration purposes only, invisible to anyone
scanning the alert for real opportunities, rather than appearing
in the human-facing actionable section under any tag.

Given this report already has one confirmed mislabeling problem this
morning, treat both issues together tonight as one "alert-formatting
integrity" pass rather than two unrelated nitpicks.

## THIRD — CHECK WHETHER DAL/NYG ARE STILL MISSING FROM NFL_GAME, OR JUST HAVE NO QUALIFYING EDGE TODAY

Last night's Elo cache fix claimed all four previously-dropped teams
(DEN, KC, DAL, NYG) were restored, confirmed "32/32 teams." This
morning's report shows DEN and KC back (real, positive confirmation
the fix is holding for those two) — but Dallas and the Giants don't
appear anywhere in the report, in any tier. Possible innocent
explanation: no current DAL/NYG market is mispriced enough to clear
the edge threshold today, which wouldn't be a bug. Given these are the
exact two remaining teams from an already-known, just-fixed issue, I
wouldn't assume that without checking. Ask: confirm directly in
`signals_log.csv` whether DAL/NYG signals are being generated
internally and just not clearing the edge bar, versus still being
silently filtered somewhere downstream of the Elo layer.

Lower priority than items 1 and 2 above, but same session, same
diagnostic effort (checking `signals_log.csv` directly) — worth
bundling into one pass tonight.

## CARRIED FORWARD FROM LAST NIGHT'S SUMMARY, UNCHANGED

- **Named-outcome placeholder bug, re-scoped upward.** Confirmed
  broader than originally thought — GDP is the only model that ever
  got real-label treatment; NFP, Unemployment, CLAIMS, NFL_GAME,
  MLS_GAME, and EPL_GAME all still show the generic placeholder. This
  is now a single, general, high-value fix (fixes six things at once)
  rather than a narrow CLAIMS-specific cleanup — recommend prioritizing
  above the NFL Elo doc-writing decision on tonight's list.
- **The recurring "declared fixed but wasn't" pattern** (Gate 4c, the
  local archive-file gap, and now possibly this EPL/MLS mislabeling)
  is worth naming directly rather than treating each as an isolated
  one-off — something about how "fixed" gets confirmed in this
  workflow may not be actually verifying persistence, just confirming
  intent at the moment of the fix. Worth a real structural conversation
  once tonight's diagnostic on item 1 above comes back, especially if
  it turns out the fix genuinely didn't persist a third time.
- **Gate 4c diagnosis (FORGE handoff)** stays tomorrow's top priority
  per Rus's explicit direction — real git archaeology on why the
  original concentration cap didn't survive two sessions, starting
  from the Aug 31 stash-incident lead, before any rebuild.
- **Local session archive gap** (recurred for the second time after
  being declared fixed two nights ago) — flagged as worth a real fix
  if it happens a third time; given tonight's pattern discussion above,
  may be worth addressing now rather than waiting for that third
  occurrence.

## VALIDATION TRACKER

Unchanged: Gate 1 project-wide suspended, real trigger is GDP's Oct 30
resolution or MLB_GAME hitting n=15-20 spread-tagged signals. MLB_GAME
n=109, 62.4%/0.2305. MLS_GAME retired from further development,
still generating track-only signals (see items 1-2 above for real
concerns about how those signals are surfacing). EPL_GAME built and
live, signal-generation confirmed working, formatting/attribution
issue open per item 1. JOBS n=6, not validated. GDP has real spread
data, no resolved sample before Oct 30.

---
J@rv1s | Papa Ralph standard.
