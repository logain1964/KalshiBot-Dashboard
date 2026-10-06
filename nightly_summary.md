# SEDE Nightly Session Summary
## For J@rv1s Morning Intelligence Pull

**Covers:** Mon Oct 5 through early Tue Oct 6, 2026 | **Session end:** ~2:30 AM CT, Oct 6
**Prepared by:** Archie (Claude Desktop)
**Replaces** the Sept 25 - Oct 4 version of `nightly_summary.md` and `nightly_summary_latest.md` (identical content pushed to both). The Sept 25 - Oct 4 record is still in git history and in the project as `claude/nightly_summary_latest_20261004.md`; everything in it that is still standing is carried forward in the status sections below.

---

## THE TWO DAYS IN ONE PARAGRAPH

Oct 5 was the external-review day: three outside AIs from three providers reviewed the audit packet blind, all three found the deliberately planted error (one only after reading the code), and their other findings were triaged against our own simulation tables. The review changed nothing in production, but it produced three decisions: one fee definition everywhere (done), a FORGE referral on how to test for edge (written, J@rv1s has it), and what to do with the held-back calibration rows (staged, awaiting Rus). In the evening the NHL_GAME / NBA_GAME "zero signals ever" problem turned into a design for an exact team matcher, which J@rv1s approved with additions on Oct 5 and again on Oct 6 (~2 AM). **Nothing in the matcher is shipped.** It is scheduled for Oct 13-19, after the pre-registration lock, as one commit after a dry run. Two findings from the read-only audit matter beyond the matcher: a structural leak in how an unconfirmed "@" game label gets classified (never observed firing, but exposed), and the fact that the paper P&L annotation changes sign with the row rule, so it is noise at current sample sizes and must not be read for direction.

---

## 1. MON OCT 5: EXTERNAL REVIEW ROUND

- **9385fd7 (the outcome-convention repair) is live.** It is an ancestor of origin/main, and the Oct 5 data passes the outcome invariant: 0 blocking mismatches against Kalshi settlement. Oracle's backfills are writing the corrected convention going forward. (Open and unchanged: MLB_GAME game-date backfill defect; MLB_GAME advisory 17 bad rows.)
- **Three reviewers, planted error found by all three,** direction right. This tests reviewer quality, not our model. Scoring details stay in the private answer key, deliberately not in this public file.
- **Triage against our own tables.** Confirmed as small: the W=0 simulator null is not a strict "no information beyond the quote" null (about +0.03 to +0.06 on mean w, inside Monte Carlo noise for the false-pass rate); the production fee default did not match the packet's. Contradicted by our tables: "use the last row instead of the first" (setting B first vs last is flat), and "the t-test is mathematically impossible" (it is underpowered and conservative, not impossible). Already known and now independently confirmed: multiplicity (46-63% chance at least one of 12 tests passes on noise), ask-basis bias, estimand attenuation when disagreement is only partly persistent, two-sided events losing money, traded vs flagged population. Untested and new: stale forecaster input vs fresh quote, stale logged ask vs executable ask, cheap-side tilt, Brier on a selected tail rewarding extremity, one-position-per-event P&L.
- **Plain read of what the reviews mean:** at current sample sizes "not demonstrated" is the expected result even for a real edge. Nov 1 cannot produce a verdict for NFL_GAME; at best it is a checkpoint, and a verdict is possible only near the end of the NFL season.

## 2. OCT 5: THREE DECISIONS

1. **One fee definition (decision 1, option A): `round1` everywhere** (fc449fe). `taker_fee_cents()` in `brier_dashboard.py` is the single helper; the stress test, the market-relative report and the dashboard output (`spread_stress_fee_mode`) all use it. Effect on real-ask stress sums (exact to round1): NFL_SPREAD -2064.8c to -2408.0c, NFL_GAME -62.9c to -84.0c, MLB_GAME +154.8c to +150.0c. `round1` is a conservative 1-contract convention, not the true fee for larger orders; report headers now say so.
2. **Stale JOBS line removed** (6580852): the daily report no longer says JOBS "passes all three gates (n=73...)". `MODELS_LIVE_ELIGIBLE` is empty with an explanatory comment; the report says no model is go-live eligible.
3. **FORGE referral on Gate 1 test design** written and with J@rv1s (`claude/gate1_test_design_after_external_review_forge_referral_20261005.md`). Proposal under review, not approved: two separate statistics (information via a paired Brier difference; tradable profit via one-position-per-event net P&L), w* demoted to descriptive and sizing, one primary model for Phase 2 (NFL_GAME, on data availability only), "unresolved" as a defined outcome, confirmatory sample = events after the lock. Open numbers needing J@rv1s: delta_min and the multiplicity rule.

## 3. OCT 5 EVENING: NHL_GAME / NBA_GAME

J@rv1s found both single-game models built and wired but with zero rows ever, flagged or full. Archie's read from the Oct 5 run log: the Odds API key works (17 NHL and 46 NBA games loaded); only 1 of 17 NHL games matched a Kalshi market (edge -0.9c, no signal); NBA loaded 0 power ratings (offseason, so it cannot score). The full-log "all_*_signals" fallback is never assigned for these models, so the full log can never show "ran and found nothing". The current matcher is also not sound: it needs words from BOTH team names in a one-team title, its yes-side test (team abbreviation as a substring of the full name) works for only 18 of 32 NHL and 21 of 30 NBA teams, it has cross-hits (e.g. LA inside CGY/CHI/COL/DAL/NYI/PHI, TOR inside NSH/OTT, PIT inside WSH), and it has no date check.

**Design (approved, not shipped).** Parse the Kalshi ticker (series, date, away, home, winner) against fixed 32 and 30 team tables, map Odds API names, match the ordered (away, home) pair on the same Eastern date with tolerance 0, derive the yes-side from the winner, and give every market exactly one reason code (`ok`, `parse_fail`, `unknown_abbr`, `ambiguous_split`, `winner_not_in_game`, `no_odds_game`, `orientation_mismatch`, `date_mismatch`, `ambiguous_odds_game`), so a silent zero cannot happen. Validated offline on live NHL data (34 matched) and smoke-tested on NBA (6 matched, offseason). A permanent fail-closed home/away orientation assertion was added: on any game with devig more than 5c from even, the Kalshi side and devig side must agree. On the Oct 5 snapshot, all 16 such markets pass; max Kalshi-vs-devig gap 1.2c, mean 0.6c.

**J@rv1s's rulings (Oct 5 and Oct 6, all accepted):**
- Ship after the pre-registration lock and before the Oct 20 NBA opener (window Oct 13-19), log-only, as one commit after a read-only dry run.
- NHL_GAME and NBA_GAME enter `model_suspension.py` as suspended from day one (the EPL_GAME precedent) and sit outside the Phase 2 family. A model joins a test only with its own pre-registered test, a stated event count and a decision written before outcomes are examined.
- A separate sidecar `data/cross_book_gap_log.csv` records every evaluated market, flagged or not, with `run_id`, snapshot time and Odds fetch time, counted in a new dashboard line "logged, not scored, not evidence" (additive JSON keys only). It is never labelled NHL_GAME in any edge column. Its row rule is written into the pre-registration draft first.
- Every run prints the full refusal list; alert if more than 10% of markets whose two teams both appear in the Odds window are refused as `date_mismatch`.
- Backfills of historical data come later; the fallback audit table (each `log_all_signals` call: does an all-evaluated variable exist) is medium priority.

## 4. TUE OCT 6 EARLY: READ-ONLY AUDITS FOR J@RV1S'S ROUND 2 (reply saved: `claude/archie_reply_to_jarvls_matcher_round2_20261006.md`)

**NFL_GAME 27 events (Oct 3) to 41.** New games, not a selection change: 26 through Sep 28, 1 on Oct 1, 14 on Oct 4. The fee commit did not touch selection.

**P&L annotation: the sign depends on the row rule (round1 fee, cents per contract, t in parentheses).**

| Model | All rows with ask | First row per ticker+direction | One row per event |
|---|---|---|---|
| NFL_GAME (41 events) | +0.56 pooled / -0.65 event-mean (-0.09) | same | **+2.27 (+0.31)** |
| NFL_SPREAD (60 events) | -0.63 / +0.76 (+0.18) | -0.95 / +0.38 (+0.09) | **+0.48 (+0.08)** |
| MLB_GAME (15 events) | +11.50 / +10.00 (+0.71) | +10.00 (+0.71) | +10.00 (+0.71) |

Every t is below 0.75. These are noise around zero at this n. Recommendation to the pre-registration: primary rule = one row per event (earliest logged row with a real ask), unit = mean cents per event, round1 fee; the others descriptive. I chose that after seeing this table, so the pre-registration must say so. Also found: 48 of 60 NFL_SPREAD events and 12 of 41 NFL_GAME events have both directions logged, which is why row rules disagree. The 15c entry edge is net of fee at the real ask; the "mid" column is gross.

**Fail-closed audit (`detect_signal_model`).** Today's "@" branch guesses MLB_GAME for any unconfirmed game label, and only the portfolio loop guards that guess. In `signals_log.csv` 799 rows carry "@" labels; 560 are logged as NFL_GAME and 2 as SEEDS_MLB, which the fallback would mislabel as MLB_GAME (MLB_GAME YES is not suspended). In the full log it is 2,513 "@" rows, 1,930 of them would be mislabelled. **I found no evidence the fallback has ever fired:** zero matching warning lines in the 593 laptop-mirrored log files; Oracle's own logs not yet checked, so the 35% calibration prediction stays pending. The planned fix: unconfirmed "@" labels return UNKNOWN and every sink treats UNKNOWN as track-only, with a regression test over all historical labels (baselines saved) where only the intended "@" classifications may change. Smaller: WC_GAME / SOCCER_GAME / NBA labels read as MLS_GAME by that function (fail toward a suspended model, so closed, but category mislabels on the dashboard).

**JOBS bid/ask capture (Oct 9 build deadline): mechanics met.** Capture fires per signal row (each row carries its own book). The full log has 28 JOBS rows with asks (Oct 1 21:00 run: 17; Oct 2 07:00 run: 11; 18 tickers; all outcomes filled); the flagged log has exactly 1 (`KXU3-26SEP-T4.3` YES, 2c, lost), which is the dashboard's "n=1". Statistically this is one release, so it supports no statement; the next print is early November (date to verify). Unexplained: the Oct 2 07:00 run has fewer tickers with asks than Oct 1 21:00.

**Calibration (private file, commits 578d3be and ea12421).** Reviewer-flaw row resolved TRUE (Brier 0.0225); v3.1 "safe" row TRUE (0.0225, annotated weak test); orientation check resolved TRUE (informative-market reading chosen after seeing the result, so weak); J@rv1s's two new entries added as pending. A public file with one packet row still held is staged and **not pushed**; that is Rus's call.

---

## OPEN POSITIONS (`sede_portfolio.json`, last updated Oct 5 21:07 CT)

8 open (BTC<$50k NO; seven GDP YES positions on the Oct 30, Jan 28, Apr 29 and Jul 29 prints), bankroll $994.17, no wins or losses recorded, one early exit at -$5.83. No new entries; nothing has cleared the entry criteria. Marks were not re-pulled tonight; see the file. GDP stays at reduced weight with n=1 real resolved event, so the marks tell us nothing about the model.

---

## GATE 1 / VALIDATION STATUS (carried forward, with tonight's corrections)

- Project-wide Gate 1 remains **provisionally suspended**; per-model checkpoints Oct 30 (GDP) and the Nov 1 Phase 2 checkpoint. Phase 2 threshold to be locked in writing by about Oct 13, hard stop Oct 20, before Phase 2 data is examined.
- **NFL_GAME: no demonstrated edge, entry weight 0.** Current figures: 41 real-ask events, w* 0.00 [0.00, 0.85]; net P&L +0.6c per contract on the all-rows rule, +2.27c on one-row-per-event, t about 0.1-0.3. The project-instructions NFL_GAME paragraph still quotes the older 27-event figures; Rus pastes the update (Archie cannot edit instructions).
- NFL_SPREAD and MLS_GAME / SOCCER_GAME: tail finding stands. MLB_GAME: do not promote; NO direction suspended. JOBS: n=4 verifiable events, unproven. GDP: n=1. CLAIMS: suspended; fix shipped Oct 1, first real test Oct 7-8.
- Win-rate, P&L and bucket figures computed Sept 24 - Oct 3 for NFL_GAME, NFL_SPREAD, MLS_GAME, MLB_GAME remain suspect; decision-impact audit pending.

---

## WORK AHEAD (Archie's honest read, in J@rv1s's dated order)

- **Wed Oct 7:** read-only diagnostics, each with the decision it could change written first: input age vs quote age; price-bucket split of outcome minus price paid; Brier on all vs flagged vs traded rows; one-side vs two-side P&L; persistence on non-duplicate flags. CLAIMS first production emission expected Oct 7-8 (check Oracle has a FRED key).
- **Thu Oct 8:** recheck the Oct 2 JOBS ticker gap.
- **Fri Oct 9:** pre-registration draft to J@rv1s (primary model, w_min, population, round1 fee, row rule, confirmatory-sample rule, lock commit hash, unresolved action, NHL_GAME/NBA_GAME exclusion sentence and admission rule, cross_book_gap row rule).
- **Oct 13 lock target; Oct 13-19:** matcher, fail-closed UNKNOWN fix, suspended entries, sidecar, dashboard line, refusal list and date_mismatch alert as one commit after dry run. **Oct 14** CPI release. **Oct 20** hard stop for locking Phase 2. **Oct 30** GDP. **Nov 1** Phase 2 checkpoint.
- Rus: (1) on Oracle, `git -C ~/KalshiBot pull` to pick up fc449fe and 6580852; (2) decide the public calibration push; (3) paste the updated NFL_GAME paragraph into project instructions; (4) optionally run one read-only grep of Oracle's logs for the fallback warning (Archie will supply the line).
- Standing: MLB game-date backfill fix; MLB Stats API audit of all 230 MLB_GAME rows; tickerless random-sample audit; decision-impact audit; wire the outcome invariant into publication; four laptop stashes exist (data churn), do not drop them; Auto Monitor stop-loss alerts still laptop-only.

## ORACLE / SPORTS MONITORING

Oracle: sole pipeline runner; last auto-update Oct 5 21:00 CT (369d980). Not re-verified tonight that Oracle has pulled fc449fe / 6580852. Sports monitoring: no open sports positions.

---

Archie | Papa Ralph standard.
