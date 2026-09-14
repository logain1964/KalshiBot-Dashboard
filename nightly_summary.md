# SEDE Nightly Session Summary
## For J@rv1s Morning Intelligence Pull

**Last updated:** 2026-09-14 | **Session end:** ~10:45 PM CT
**Prepared by:** Archie (Claude Desktop)

---

## TONIGHT IN ONE SENTENCE

EPL_GAME model built end-to-end (Poisson + carryover blend, football-data.co.uk, since ESPN's eng.1 is confirmed dead too) and shipped track-only; both open design questions sent to J@rv1s, answered, and closed out for real tonight — the taper formula independently re-derived and confirmed, and the Gate 1 "beats_random" question resolved with a correction to J@rv1s's own cited base rate along the way. One real housekeeping gap found and flagged below: this exact file has been serving J@rv1s stale June content for months, now fixed.

---

## 1. EPL_GAME MODEL — BUILT, TESTED, SHIPPED (commit 88f377c)

Rus explicitly authorized building EPL now rather than waiting on the stalled Sept 8 Gate 1 checkpoint, and explicitly chose football-data.co.uk over football-data.org as the data source.

**Before writing any code**, re-verified live (not assumed from the Aug 22/23 design docs) whether ESPN's `eng.1/standings` and `eng.1/scoreboard` still worked. They don't — 403, same Akamai block that's hit every other sport since Aug 9. So unlike MLS, EPL has **zero ESPN fallback at all**, live or standings.

**What got built:**
- `game_status.py`: SharpAPI league code `"EPL"` confirmed live (108 real rows). Added `epl` to `SHARPAPI_LEAGUE`/`BLACKOUT_BUFFER_HOURS` (2.5h, same as MLS).
- `models/soccer_game_model.py`: `FD_TO_KALSHI_EPL` alias table, verified against real live `KXEPLGAME` tickers (all 20 current clubs) — caught two real mismatches: Chelsea is `CFC` not `CHE`, Liverpool is `LFC` not `LIV`. Carryover-blend taper implemented per the locked Aug 23 FORGE spec. Promoted-club floor applied (loudly logged, not silent) for Coventry/Hull/Ipswich — all three genuinely newly promoted for 2026-27, confirmed against real data, not an error. `run_epl_game_model()` mirrors `run_mls_game_model()`'s structure.
- **Deliberately did NOT port** MLS's `MLS_DISAGREEMENT_PRICE_FLOOR`/`MLS_EXTREME_CONFIDENCE_GATE` hard-exclusion filter — that was earned from a real n=49 MLS backtest; EPL has zero signal history to justify an equivalent yet.
- `market_scanner.py`: `KXEPLGAME` wired into `TRACKED_SERIES`.
- `daily_runner.py`: full pipeline wiring, `SEDE_RELIABILITY["EPL_GAME"]=0.55`, added to `MODELS_SUSPENDED_FROM_TRADING` (track-only). **Found and fixed a real bug along the way**: EPL signal labels are textually identical to MLS_GAME's (`"X vs Y -- Z wins"`) — `detect_signal_model()` would have silently misattributed every EPL signal to MLS_GAME. Fixed by disambiguating on the real Kalshi team-abbreviation set (verified zero overlap with MLS's live codes).

**Verified live**, not synthetic: full run against real `KXEPLGAME` markets — 10 games, 7 signals flagged. Full writeup: `claude/epl_build_20260914.md`.

**Real caution, not a green light**: several flagged edges were 15-22c on a model with zero backtesting history (e.g. Brentford over Chelsea at 54.2% model vs 32c market). That's more likely a sign of model error than confirmed market inefficiency at this stage. Logged directly in the `MODELS_SUSPENDED_FROM_TRADING["EPL_GAME"]` entry so it can't get quietly forgotten before real Gate 1 data exists. **Suspended from trading. Track-only.**

---

## 2. EPL TWO OPEN ITEMS — SENT TO J@RV1S, ANSWERED, CLOSED

Full referral: `claude/epl_open_items_referral_20260914.md`. J@rv1s's full FORGE response is in the conversation record; both items now have real, grounded resolutions (`claude/epl_open_items_resolution_20260914.md`, `claude/epl_taper_derivation_20260914.md`).

**Item 1 — the missing taper-derivation doc.** `claude/epl_taper_derivation_20260823.md` (meant to hold the taper formula's full derivation) doesn't exist — confirmed missing. Per J@rv1s's recommendation, re-derived it for real tonight: pulled 7 real football-data.co.uk seasons (2019-20 through 2025-26), walked matches chronologically with no lookahead, grid-searched against real outcomes. **The locked asymptote (0.70) came back exact in every configuration**, with or without the 2020-21 no-fans season. Slope/crossing differ slightly from locked but the real performance difference is 0.2% — noise for the size of grid searched. **No code change made.** The formula is now independently verified, not just trusted.

**Item 2 — the Gate 1 "beats_random" bar.** Turned out simpler than either of us first thought. Read the actual scoring code (not just the one verdict doc): `beats_random` is a fixed, uniform `Brier < 0.25` check (coin-flip baseline) applied identically to every model — and since 0.25 > 0.20, **it's mathematically implied by passing Gate 1's own Brier<=0.20 criterion**. There's no 3-way-vs-binary adjustment to make because it was never a 3-way check at all. The real place a base-rate question lives is the 55% win-rate threshold — same mechanism as every other binary-leg model in this project, and the same class of risk `MLS_DISAGREEMENT_PRICE_FLOOR` already guards against for MLS_GAME. **No EPL-specific bar adjustment recommended.**

**One correction on the way through, flagged plainly rather than let stand**: J@rv1s cited a "real current" home/draw/away rate of 42/27/31. Pulled it directly from the same football-data.co.uk seasons everyone's using (last 4 complete seasons, n=1520 real matches): **44.5% home / 24.1% draw / 31.4% away** — closer to my original estimate than to J@rv1s's correction. Noted in the resolution doc so it doesn't quietly stand uncontested.

---

## 3. ONE REAL HOUSEKEEPING GAP FOUND — FLAGGED AND FIXED

**This exact file has been serving J@rv1s stale content.** `nightly_summary.md` (the literal filename J@rv1s's morning routine fetches from `raw.githubusercontent.com/.../main/nightly_summary.md`) was last meaningfully updated June 18 — every real nightly summary since (Sept 2 through tonight) has actually been going into `nightly_summary_latest.md` instead, committed by Rus by hand. Tonight's push updates **both** files with identical current content so J@rv1s's actual automated fetch target stops serving June data. Worth deciding going forward whether `nightly_summary.md` or `nightly_summary_latest.md` is the one true target — right now there are two files and only one was being kept current by habit.

(A second suspected gap — `sede_portfolio.json` looking 5 days stale — turned out to be my own mistake: I'd read it before pulling latest from origin. After `git pull`, `last_updated` is `2026-09-14T11:20:04` — today, confirmed. The daily pipeline is actively running; no real gap there. Correcting this here rather than letting a false alarm stand, since the whole point of flagging things is that the flags be real.)

---

## OPEN POSITIONS (per `sede_portfolio.json`, confirmed current as of today's 11:20 CT pipeline run)

| # | Description | Entry | Current (Sept 14) |
|---|---|---|---|
| 1 | BTC<$50k Dec31, NO | 43.5c | 83.5c |
| 3 | GDP>1.5% (Oct30), YES | 70.5c | 77.5c |
| 4 | GDP>1.5% (Jan28), YES | 74.5c | 66.0c |
| 5 | GDP>2.5% (Oct30), YES | 58.0c | 50.5c |
| 6 | GDP>1.0% (Jan28), YES | 78.0c | 77.5c |
| 7 | GDP>2.0% (Apr29), YES | 50.5c | 52.5c |
| 8 | GDP>4.0% (Oct30), YES | 18.5c | 17.0c |
| 9 | GDP>1.5% (Jul29), YES | 59.5c | 62.5c |

Bankroll: $994.17 (started $1,000, one early-exit close at -$5.83). No wins/losses recorded yet — all 8 still open, no new entries since early Aug (nothing new has cleared the 6-gate entry criteria since).

---

## GATE 1 / VALIDATION STATUS (unchanged since Sept 8 checkpoint — no new resolution tonight)

Project-wide Gate 1 remains **provisionally suspended** (since Aug 11, pending the bid/ask-spread stress test). The Sept 8 checkpoint's own real recommendation was to **extend** the suspension, not resolve it — no model had both a real deduplicated sample near the n>=30 floor and enough spread data to stress-test it. Real trigger set for the next checkpoint: GDP's Q3 2026 markets resolving Oct 30, or MLB_GAME accumulating n>=15-20 spread-tagged signals — whichever comes first. Nothing in tonight's session changes this; flagging that the project instructions' "Checkpoint: September 8, 2026" line is now stale by a week and should probably be updated to reflect "extended, next real trigger Oct 30 or MLB_GAME spread accumulation" rather than a passed date.

| Model | Status |
|---|---|
| JOBS | NOT validated (real n=6, ~2 independent events — reverses the earlier "corroborated" call) |
| GDP | Has real spread data, no real resolved sample yet before Oct 30 |
| MLB_GAME | n=109 resolved, 62.4% WR, Brier 0.2305 (fails 0.20 bar narrowly); only 4 real spread-tagged signals |
| MLS_GAME | RETIRED from further live development (Sept 3 verdict, confirmed fail) |
| CLAIMS | Suspended, gate bug fixed Sept 3 |
| NFL_GAME / NFL_SPREAD | Suspended pending real 2026 signal accumulation |
| **EPL_GAME** | **NEW tonight — suspended pending validation, zero signal history** |

---

## TONIGHT'S WORK ORDER FOR J@RV1S / NEXT SESSION

1. Decide `nightly_summary.md` vs `nightly_summary_latest.md` as the one real target going forward (gap above) — cheap fix, just needs a decision.
2. EPL_GAME's first real signals will start accumulating now that it's wired into the daily pipeline — worth a look at whether `KXEPLGAME` markets are actually showing up in tomorrow's run.
3. The Sept 8 Gate 1 checkpoint text in the project's own standing instructions is stale (says "Checkpoint: September 8, 2026" with no note that it was extended) — worth a cleanup pass whenever convenient, not urgent.
4. No open action from tonight's EPL FORGE exchange — both items are genuinely closed, not deferred.

---

## SPORTS MONITORING

No live in-progress games at session end. EPL's first tracked fixture (Leeds vs Newcastle, KXEPLGAME-26SEP14LEENEW) kicks off 19:00 UTC today — first real test of the SharpAPI-only schedule-blackout check (no ESPN live-state fallback for EPL, see Section 1). Nothing else currently open in-game per the portfolio snapshot above (all 8 open positions are GDP/BTC, not live-game markets).

---

## ORACLE CLOUD STATUS

Not directly checked tonight (Archie doesn't run commands on Oracle). Indirect signal is positive: `sede_portfolio.json`, `brier_dashboard.json`, `signals_log.csv`, and `signal_genealogy.json` all show fresh commits pulled from origin tonight (most recent `sede_portfolio.json` update: 2026-09-14 11:20 CT) — the daily pipeline is actively running and pushing data. No confirmed issue.

---

Archie | Papa Ralph standard — full session record in `claude/epl_build_20260914.md`, `claude/epl_open_items_referral_20260914.md`, `claude/epl_open_items_resolution_20260914.md`, `claude/epl_taper_derivation_20260914.md`.
