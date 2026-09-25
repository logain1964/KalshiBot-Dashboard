# Nightly Summary — 2026-09-25 (Archie)

**For:** J@rv1s (morning read) and Rus
**Session:** Evening build, Claude Desktop / Archie
**Commits tonight:** `9cf547c` (Kalshi fee fix), `80e6e39` (NFL_SPREAD suspension) — both pushed to `origin/main`.

---

## Carried in from before compaction

The merge-conflict resolution from the prior session is confirmed live —
Rus pasted a `git pull` output earlier tonight showing Oracle was synced
to `dbbb5c0` before tonight's two new commits went in. No open thread
there.

WO #1/#2 (JOBS n=6 reconciliation) — closed. Real accuracy 16.7% on a
small sample; nothing actionable, just closing the loop from a prior
work order.

WO #5 (repo research) — evaluated, findings written up, no code changes
warranted tonight.

---

## Fix 1 — Kalshi taker fee (SHIPPED, commit 9cf547c)

`brier_dashboard.py`'s `spread_stress_analysis()` was computing real-money
P&L off real bid/ask fills but never deducting Kalshi's actual taker fee.
Fixed: `fee_cents = 0.07 * real_cost * (100 - real_cost) / 100`, applied
to both the win and loss legs of real P&L, and to `realistic_edge`. Added
a `FEES` column to the report and a `fees_cents` field per model in the
JSON output.

Verified by full re-run against real data: NFL_SPREAD -389.1¢ across 324
rows, MLB_GAME -20.9¢/13 rows, NFL_GAME -17.6¢/12 rows — all matched the
formula's predicted magnitude. `data/brier_dashboard.json` regenerated
clean; only `brier_dashboard.py` and that JSON changed in git status.

## Fix 2 — WC_GAME / SOCCER_GAME gap-clustering (VERIFIED, no code change)

Checked the flagged 16:1 dedup ratio the same way the earlier confirmed
MLB_GAME bug was checked — pulled the real underlying rows and gap-day
distribution rather than guessing at a threshold. Conclusion: this one is
NOT the same failure pattern as MLB_GAME. No fix needed. The separate
WC_GAME→SOCCER_GAME relabel question is still open (see "Work for
tomorrow" below) — that's a naming/classification question, not a dedup
bug.

---

## Manual paper-trading "stall" — investigated, NOT actually stalled

Rus flagged that `paper_trades.json` looked stalled at Trade #25. Ran
every real signal since that trade through the actual entry gate
(`paper_trade_gates.py`). Finding: three separate, individually correct
reasons, not neglect —
1. Correctly avoided re-entering a position correlated with an existing
   open GDP bet.
2. A genuine dry spell — signals since #25 simply didn't clear the
   HIGH-confidence bar.
3. One CPI signal that was already deliberately deferred in an earlier
   session, not missed.

No bug, no fix. Closing this thread.

---

## System-wide calibration audit — BIGGEST FINDING TONIGHT

Ran a real, evidence-based audit across every model with a meaningful
sample: point-biserial correlation of edge size vs. actual win outcome,
plus per-model edge-bucket win-rate tables. This distinguished real
signal from noise:

- **NFL_SPREAD (n=430): genuinely broken.** Monotonic overconfidence —
  win rate declines as stated edge increases, exactly backwards from what
  a well-calibrated model should show:

| Edge bucket | Win rate |
|---|---|
| Lowest edge | 37.0% |
| … | declining |
| Highest edge | 21.1% |

  In other words: the more confident the model claims to be (the bigger
  the "edge" over Kalshi's price), the *worse* it actually does. That's
  the most dangerous shape a model can have, because it looks most
  fundable exactly where it's least trustworthy.

- **MLB_GAME, SOCCER_GAME:** well-calibrated. No action.
- **GDP, JOBS, CLAIMS, CPI:** n=5-6 each — statistically meaningless,
  can't conclude anything either way yet. Flagged for J@rv1s to keep an
  eye on as sample size grows, not to act on now.

### Root cause (traced into the actual code, not guessed)

`models/nfl_model.py`'s `margin_cover_probability()` treats the model's
point estimate of the game margin (`mu`, Elo-implied) as *certain* and
only models game-to-game variance around it (`sigma=13.65`, a real,
sourced academic figure). It never models the uncertainty *in mu itself*
— which is compounded from Elo + QB-tier + rest-day + weather + motivation
adjustments, each with its own error. That compounded uncertainty in mu
shows up as false confidence, worst exactly where the model disagrees
most with Kalshi's price — i.e. exactly where "edge" looks biggest.

Ruled out the obvious quick fix (blend with SharpAPI market consensus)
because SharpAPI's real single-book coverage is only ~91.5% of games
single-book (confirmed via the existing `nfl_sharpapi_coverage_retest`
doc) — too thin to gate on. This was a deliberate, previously-ratified
"Strong Node" architecture decision (July 2, 2026), not an oversight, so
the real fix has to be a proper parameter-uncertainty model, not a quick
market-blend patch.

## NFL_SPREAD suspended (SHIPPED, commit 80e6e39)

Per Rus's explicit direction ("figure out how to resolve it," then "go
ahead with your suggestions"): NFL_SPREAD is now suspended from paper
trade eligibility in `paper_trade_gates.py`'s `FULLY_SUSPENDED_MODELS`,
with the full finding documented inline as a comment. The real fix
(proper parameter-uncertainty modeling in `margin_cover_probability()`)
is scoped, not built — that's tonight's "Work for tomorrow" item, not
tonight's fix.

### Correction on the record

Earlier tonight, in the Week 3 NFL status check, I told Rus NFL_SPREAD
was not formally suspended and only NFL_GAME couldn't be traded. That was
**wrong**. While implementing the fix above I found `daily_runner.py`
maintains a second, separate, actually-authoritative suspension list
(`MODELS_SUSPENDED_FROM_TRADING`) that already had NFL_SPREAD suspended
since Sept 18 — for a different, now-superseded reason (zero real 2026
signals resolved yet).

Net effect: no practical harm — the fully autonomous, subscriber-facing
SEDE portfolio (`portfolio_manager_sede.py`, via `daily_runner.py`) was
never exposed to tonight's calibration bug, because that second list
already blocked it. Only the Tier 1/Tier 2 paper-trade surface (governed
by `paper_trade_gates.py`) had the gap, and that's now closed too. But my
earlier claim to Rus was factually inaccurate and belongs on the record
as a correction, not quietly dropped.

Also updated the stale Sept 18 reasoning note in `daily_runner.py` so a
future session doesn't reinstate NFL_SPREAD on "enough data now" grounds
without knowing there's a real diagnosed calibration defect underneath.

**Incidental find, not fixed (Surgical Changes — out of scope tonight):**
`paper_trade_gates.py` has a comment claiming it "mirrors
`portfolio_manager_sede.py` MODELS_SUSPENDED_FROM_TRADING keys" — that
name doesn't exist anywhere in `portfolio_manager_sede.py` (confirmed via
direct grep, zero matches). Stale comment, flagged for cleanup, not
touched.

---

## Oracle Cloud status

Oracle is **2 commits behind** origin/main as of tonight (`9cf547c`,
`80e6e39`). Per the standing Oracle Command Handoff rule, I don't run
commands on Oracle myself — Rus, paste this into your own Oracle SSH
session:

```
cd ~/KalshiBot && git pull origin main && git log --oneline -1
```

Server: 163.192.200.127, Ubuntu 22.04, VM.Standard.A1.Flex, $0/month.
No other Oracle-side changes needed tonight.

## Validation / Gate 1 status

No change from standing status — still project-wide provisionally
suspended pending the real bid/ask-spread stress test (paper fills were
mid-price, not real executable price). Checkpoint: September 8, 2026 (per
project instructions — flagging that this date has already passed;
worth confirming with Rus whether the checkpoint review actually happened
or needs scheduling).

Suspended: CLAIMS, MLB_GAME (NO direction), and now NFL_SPREAD (new
tonight, see above). Reduced weight: GDP (0.60, scoring stalled since
July 30). JOBS remains caveated, not unconditionally validated.

## Open positions

No new positions opened tonight — this session was investigation/fix
work, not trading. Existing GDP position and any others from prior
sessions are unchanged; refer to `paper_trades.json` for the live list.

## Sports monitoring

NFL Week 3 (Sunday) was checked earlier tonight via the real signal log
(`run_date == 2026-09-24`, filtered by `market` field since `event_date`
is blank for live pre-game NFL rows). NFL_GAME remains fully suspended on
principle; NFL_SPREAD is now also suspended as of tonight (see above) —
so no NFL trades of either type should be recommended Sunday regardless
of how attractive a signal looks. Re-check the slate fresh tomorrow since
this was checked before tonight's suspension landed.

---

## Work for tomorrow — J@rv1s

1. **CPI Part B** — pick up where the CPI backlog thread left off (see
   `cpi_backfill_and_accuracy_bug_20260924.md` and `cpi_mom_backlog_20260922.md`
   project docs for context).
2. **FORGE the NFL_SPREAD fix approach** — run the real parameter-
   uncertainty modeling approach through Papa Ralph / FORGE before
   Archie builds anything. This is a real architecture decision (how to
   model compounded uncertainty in `mu`), not a quick patch — worth the
   full Find/Oppose/Reframe/Ground/Evaluate treatment.

3. **Proactively run the calibration-audit method on other models** —
   tonight's point-biserial + edge-bucket method found a real, previously
   invisible bug. Worth running the same method against models not
   covered tonight (or re-running periodically) rather than waiting for
   another complaint to trigger it.
4. **Resolve the WC_GAME → SOCCER_GAME relabel question** — separate from
   tonight's gap-clustering check (which came back clean). This is a
   naming/classification question that's been sitting open.
5. **Refresh the NFL Week 3 slate check** — tonight's check ran before
   the NFL_SPREAD suspension landed. Re-verify no stale NFL_SPREAD
   signals are floating around anywhere before Sunday.

## Work for tomorrow — Rus

1. Run the Oracle pull command above so the server matches origin
   (`9cf547c`, `80e6e39`).
2. Decide on the manual-trading-cadence question you raised — the stall
   investigation found the process is working correctly, but you may
   still want to revisit how often you're checking in on it.
3. Sign off on timing for the real NFL_SPREAD fix — is this next
   session's priority, or does something else come first?
4. Optional: green-light cleanup of the stale `paper_trade_gates.py`
   comment referencing a `portfolio_manager_sede.py` name that doesn't
   exist (cosmetic, zero functional impact, purely a future-confusion
   risk).

---

## Carried forward, unchanged from before tonight

- MLB park-factor gap — still open, not touched tonight.
- `hypothesis` library trial — still on the list, not started.

---

Archie | Papa Ralph standard.
