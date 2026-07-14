# SEDE NIGHTLY SUMMARY — 2026-07-13 (Mon)
_For J@rv1s morning briefing. Written by Archie (web, Opus 4.8), ~11:00 PM CST._

## TL;DR
Short session tonight: NFL-spec feasibility follow-up + full security rotation (done earlier) and one substantive MLB integrity triage (post-relaunch). **Two of your requested Elo checks are still OPEN.** One new finding materially reframes the July 25 MLB verdict — needs your adversarial pass.

---

## NEEDS YOUR EYES / ANSWERS

### 1. [OPEN — carried to tomorrow] Two Elo checks before NFL tier boundaries lock
Still owed from your feasibility review; NOT run tonight (triage took the session):
- Week 6 crossing point at **carry = 0.50**
- Convergence test extended back to **1999** (from the n=4-season window)

Tier structure (Wk 1–5 / 6–10 / 11+) stays UNLOCKED until both clear. Archie runs them first thing next session.

### 2. [DECISION NEEDED] The n=103 MLB Brier is measuring the WRONG model
Verified tonight in `models/mlb_model.py`: MLB **Track A** `model_prob` is assigned in fallback order **SharpAPI consensus → OddsAPI consensus → Pythagorean+ERA**. So the probability the Brier scores literally *is the sportsbook line*. Track A is empirically the "disguised arbitrage against sharp books" case we theorized — now confirmed from the code, not inferred.

Therefore the **n=103 / Brier 0.2611** figure is a Track-A market-follow metric, NOT the Track B Elo/FIP prospective model the go/no-go is actually about.

- **Question for you:** should the July 25 verdict hang on Track B's blind prospective record instead of this Brier? Archie's read: **yes.** Wants your FORGE/Papa-Ralph pass before it's treated as decided.
- **Supporting note:** 0.2611 (worse than random) is a *selection* artifact — signals only fire where consensus diverges ≥8c from Kalshi, i.e. the most contested games. It is NOT evidence the consensus is miscalibrated.

### 3. [INFO + small ask] Oracle was missing SHARPAPI_KEY until tonight
Added + verified on Oracle tonight. Impact is **bounded**, not catastrophic:
- SharpAPI only entered the code **Jun 27**; the 103 span **May 27–Jul 8**, so most of the sample predates it entirely.
- A missing key does NOT inject a bad line — the client returns `None` and the model falls cleanly to OddsAPI, then Pythagorean.
- Real blast radius: **Jun 27–Jul 8 Oracle runs.** Main casualty is Track B's `p_sharpapi_home` divergence reference for that window (compared against a fallback line, not SharpAPI).
- **`prob_source` was never persisted per signal** (all 103 `notes` are blank/trade-only), so the exact SharpAPI/OddsAPI/Pythagorean split can't be recovered from `signals_log.csv` — only from Oracle daily run logs.

**Archie recommends (needs Rus's go):** one-line fix to persist `prob_source` into the signal row. Measurement-from-day-one — same lesson as the NFL spec.
**Open for you:** is quantifying the Jun 27–Jul 8 source split (via Oracle run logs) worth doing before 7/25, or does moving the verdict to Track B make it moot?

---

## DONE TONIGHT (FYI — no action)
- **Security rotation:** exposed GitHub PAT + SharpAPI key both rotated, verified working on laptop AND Oracle.
- **NFL feasibility pass** answered your 3 questions → `model_integrity/nfl_spec_feasibility_pass_v1.md`: injury source clean of "Probable" (retired 2016), `game_id` flex-stable, Elo convergence ~Week 6 (not Wk 1–4).

## HARD DEADLINES
- **NFL Signal Contract spec — Sept 3 kickoff (hard).** Build window July–Aug. Not started. Highest-priority upcoming build.
- **July 25 MLB verdict** — foundation under review (see item 2).

## VERIFICATION (ABV)
- Verified by reading `models/mlb_model.py` + `brier_dashboard.py` and reproducing n=103 / 0.2611 from `signals_log.csv`.
- NOT verified (Rus/Oracle side): exact Jun 27–Jul 8 source split; whether the 103 were Oracle- vs laptop-generated (dedup is last-write-wins by run_date+market, so a mixed sample is possible).
