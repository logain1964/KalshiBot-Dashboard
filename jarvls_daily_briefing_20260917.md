# J@rv1s Daily Briefing — 2026-09-17

## STATUS

Confirmed the Oracle pull happened last night, after the nightly
summary was generated — so today's two real alert emails (7:02 AM,
11:20 AM) are the actual first live tests of yesterday's EPL/MLS
mislabeling + suspension-exclusion fix. Checked both directly.

## CONFIRMED WORKING — MLS_GAME CORRECTLY EXCLUDED

No MLS_GAME rows in either of today's reports, not even in the
summary counts. Consistent with the suspension-exclusion fix doing
exactly what it was built to do. Good, direct confirmation the Oracle
pull took effect.

## STILL OPEN — EPL_GAME ABSENCE REMAINS GENUINELY AMBIGUOUS

EPL_GAME is absent from both of today's reports too, but this alone
can't distinguish two different explanations: (a) EPL generated real
signals and they were correctly excluded as suspended, same as MLS,
or (b) EPL is still hitting last night's "current-season results
empty, model cannot run" bug and generated zero signals in the first
place, unrelated to the exclusion fix entirely. This ties directly
into tonight's already-planned investigation (transient data-source
hiccup vs. something real) — no new information from today's reports
resolves it either way. Waiting on that investigation rather than
guessing.

## NEW, REAL QUESTION — WHY IS NFL_GAME STILL IN THE ACTIONABLE LIST?

NFL_GAME/NFL_SPREAD is explicitly listed as suspended in the project's
own validation status, same category as MLS_GAME and EPL_GAME. But
unlike those two, NFL_GAME signals are heavily present in both of
today's reports — roughly 60-80 signals across HIGH and medium tiers
in each run, consistent across both the 7:02 AM and 11:20 AM sends
(not a one-off fluke from right after the pull — same pattern held on
the second run too).

Two real possibilities, genuinely unclear which from what's visible:
1. **Intentional distinction** — MLS_GAME is fully retired with no
   path back, while NFL_GAME is suspended only because it's still
   accumulating validation data toward a possible future Gate 1 pass.
   Plausible the design deliberately treats "retired, nothing to see"
   differently from "suspended but actively worth watching."
2. **A real gap in the fix** — `is_signal_trade_suspended()` may only
   check against a narrower list than everything in
   `MODELS_SUSPENDED_FROM_TRADING`, and NFL_GAME isn't actually wired
   into whatever list is really being checked.

This needs a direct, explicit answer, not an assumption either way —
it materially changes whether tonight's fix should be considered fully
working or half-applied. Ask: confirm directly what
`is_signal_trade_suspended()` actually checks against, and whether
NFL_GAME's continued presence is deliberate design or an oversight.

## ALSO STILL OPEN — RAW UNFILTERED COUNT FOR BEFORE/AFTER COMPARISON

Neither of today's reports print a raw/unfiltered edge count anywhere
— only the post-filter number (113 this morning, 117 midday). There's
no way to independently verify "the filter actually removed N signals"
without a raw number to compare against. Ask: pull the raw
`all_flagged` count from the console log for today's runs, alongside
the `actionable_flagged` count already visible in the reports, so
there's a real before/after number instead of just eyeballing whether
MLS/EPL happen to be absent.

## CARRIED FORWARD, RESOLVED OR UNCHANGED

- Gate 4c: confirmed never broke (Archie's full archaeology, accepted
  in full). Gate 4d scoped as a cross-model resolution-window cap,
  design work can happen now even though the build waits — already
  sent to Archie.
- GDP sitting at 7 open positions vs. Gate 4c's cap of 3: agreed,
  leave them alone. Closing early to fix optics would corrupt the
  subscriber-facing track record on purpose — already sent to Archie.
- Whether the "Gate 4c still not built" claim's origin is traceable,
  and whether a structural fix (staleness flags on repeated
  claims) is worth building given this is at least the third instance
  of the "declared fixed but wasn't" pattern this month — still open,
  worth real attention once tonight's more urgent items are handled.
- EPL's empty current-season results — Archie's own next-session item,
  tied directly to today's item 2 above.

## VALIDATION TRACKER

Unchanged: Gate 1 project-wide suspended, real trigger is GDP's Oct 30
resolution or MLB_GAME hitting n=15-20 spread-tagged signals. MLB_GAME
n=109, 62.4%/0.2305. MLS_GAME retired, correctly excluded from alerts
per today's confirmation. EPL_GAME status unresolved pending tonight's
investigation. NFL_GAME/NFL_SPREAD suspended per validation status but
still generating actionable-list output — see question above. Claims
markets from today's 8:30 AM ET release resolved cleanly between the
7:02 AM and 11:20 AM reports, as expected.

---
J@rv1s | Papa Ralph standard.
