# J@rv1s Daily Briefing — 2026-09-22

**Prepared by:** J@rv1s (Web Claude)
**Session basis:** Review of Archie's Sept 21 nightly summary; a large batch of plugin/tool referrals evaluated through the day

---

## 1. LAST NIGHT'S FIXES — REVIEWED, PULLED TO ORACLE

Archie's Sept 21 session: CPI YoY zero-signal-since-June-10 bug root-caused and fixed (a backwards month-mapping, verified against live Kalshi ticker data), CPI MoM cut from the feed as a dormant landmine, and the correlated/duplicate signal-counting problem diagnosed with a new `n_events` metric added to `brier_dashboard.py`. Rus pulled all three commits (`eac66e4`, `986ad15`, `d1d46d1`) to Oracle this morning — confirmed live.

**Pushback on tonight's numbers, not a rubber stamp:**

1. **"NFL_SPREAD clears Gate 1's count floor" is shaky, not a clean result.** `n_events=30` against a floor of `≥30` lands exactly on the line, the first day this metric has ever existed. Before this gets cited as a real pass, it should be re-verified against fresh data independent of tonight's excitement about finally having an honest number.

2. **The single-source-of-truth gap got more dangerous, not less.** Only `brier_dashboard.py` got the `n_events` fix — `signal_scorer.py` and `daily_runner.py`'s MLB-specific function still count the old, inflated way. Different parts of the pipeline can now disagree about whether a model clears Gate 1. Recommend moving the refactor decision up in priority rather than leaving it at "if/when."

3. **The 97% correlation on NFL_SPREAD (403 rows → 30 real events) is a live risk-concentration finding, not just a sample-size accounting fix.** If one bad game-read produces 13+ correlated "signals" that all look independently tradeable under the 4-gate entry criteria, that's the same disease as the Gate 4c/4d concentration question in a different model. Worth a real look at whether entry criteria need a same-game dedup.

4. **Open question for Archie, not yet answered:** CPI was silently dead for ~3.5 months (June 10–Sept 21). Was any attempt made to backfill what CPI would have flagged over that window, or is that period a permanent gap in CPI's Gate 1 sample?

---

## 2. CPI MoM — DECISION CONFIRMED, LIQUIDITY QUESTION ALREADY ANSWERED

Cutting `KXECONSTATCPI` from the feed last night was the right call — confirmed again today: every real MoM threshold (-0.2% to 0.9%) sits under the model's 1.0% "trivially true" filter regardless of the YoY fix, so the MoM path has never produced a live signal under the current build. Not a real capability lost by cutting it.

On whether MoM deserves its own build: Archie's own check last night already answered the liquidity half of that question directly against the live API — confirmed real volume (~$14K on some thresholds), so MoM CPI contracts are genuinely traded, not dead markets. That closes out the open liquidity question from last night. Remaining open question is purely a prioritization call — a proper MoM build needs its own consensus table and its own threshold logic, real lift, not urgent while it's safely cut from the feed.

---

## 3. TOOL/PLUGIN REFERRALS — A LOT OF THESE TODAY, RANKED BY ACTUAL RELEVANCE

Went through roughly two dozen plugins, skills, and MCP servers today. Most were dropped after real verification found no fit or real problems. What's worth carrying forward, ranked:

**Real, worth testing:**
- **`pmxt`** — a unified API for Kalshi/Polymarket ("the CCXT for prediction markets"). Could genuinely simplify SEDE's bespoke Kalshi API glue. Worth Archie evaluating directly.
- **`remember`** (official Anthropic session-memory plugin) — try this before any third-party memory tool for Archie's session-to-session context gap. `claude-mem` (third-party, real, but defaults to a third-party cloud sync — use its local-only flag if tried) is the fallback if `remember` doesn't do enough.
- **Perplexity** (MCP) — real-time search with citations, a specific fit for the JOBS model's still-open gap: 84% of its original Gate 1 sample ran against placeholder NFP consensus data because a live search for the real Dow Jones/Bloomberg survey figure came up empty back in August. Worth a real test against that specific problem.
- **`quantitative-trading` plugin** (wshobson/agents) — its `risk-manager` agent's focus (correlation matrix, concentration monitoring, position sizing) maps directly onto the Gate 4c/4d question and tonight's NFL_SPREAD correlation finding. Worth a one-off test on that design question specifically, output verified against real data same as any other referral.

**Real, lower priority:**
- **`superpowers`** — the single pick if Archie wants to try one "change how I code" tool (spec-first, subagent-driven TDD, code review built in). It's in Anthropic's own official marketplace, which is why it beat out ECC, Ponytail, gstack, and GSD/Buildomator — all real, all overlapping, not worth piling up.
- **`sentry`** — real option for the still-unbuilt Stage 1 monitoring, worth weighing against the from-scratch Healthchecks.io heartbeat design already chosen.
- **`code-review`** (official Anthropic plugin) — conceptually strong (independent, confidence-scored review, exactly the discipline that would've caught Gate 4c's false premise or the EPL/MLS deploy gap earlier), but it needs a PR-based git workflow. Archie currently pushes straight to main. Worth raising as a "would this workflow change be worth it" question, not a drop-in install.
- **`repomix`** — flagged for me specifically, not Archie: if Rus runs it on KalshiBot and shares the packed output, I can verify things against real code directly instead of secondhand write-ups, same discipline this whole project already runs on.
- **`awesome-quant`**'s Prediction Markets section — a real, actively-curated reference list. Worth a periodic check back, not a one-time referral.

**Flagged for later, not now:**
- **`frontend-design`** and **`eli5`** (both official Anthropic) — save these for when the SEDE subscriber-facing web dashboard/signal product actually starts. `frontend-design` for making the dashboard look intentional instead of templated; `eli5` for turning things like Brier scores and Gate 1 criteria into something a non-technical subscriber can actually read.

**Standing habit, not a decision item:**
- **`/doctor`** — built into Claude Code already. Worth running first whenever Archie's session is behaving oddly, before assuming it's a real code bug — several of this month's "declared fixed but wasn't" incidents traced back to environment-level causes, not logic errors.

**One real security flag, worth remembering as a standing caution:** a trending "AI trading agent" repo (CloddsBot) turned out to bundle its own speculative memecoin token launch and encourage connecting real exchange/wallet credentials to unaudited code, with the identical marketing copy duplicated across a pile of unrelated GitHub accounts — a known pattern for repos farming trending status rather than organic adoption. Don't run unaudited "autonomous trading agent" repos with real API keys, ever, regardless of star count or trending badges.

**A verified TikTok workflow list, mostly legitimate:** 16 of 19 items checked out as real Claude Code token-hygiene practices (grep-before-read, read-with-offsets, cap-bash-output, sub-agents on cheaper models, plan-mode-before-edit, skills-over-CLAUDE.md, status-line context%, set-max-turns-headless — worth `ccusage` as a before/after baseline so any change is actually measured, not assumed). Two items worth calling out directly: "resume, don't re-explain" is the same problem `remember`/claude-mem is meant to solve — independent confirmation it's a real gap, not just my read on it; and the "deny reading node_modules" tip needs translating for a Python repo — the equivalent for KalshiBot is denying reads on `venv/` and `__pycache__`.

---

## STATUS UNCHANGED

- Gate 1: still provisionally suspended project-wide since Aug 11, pending the bid/ask-spread stress test. Real triggers unchanged: GDP's Q3 2026 markets resolving Oct 30, or MLB_GAME reaching n≥15-20 spread-tagged signals.
- Open positions: 8 (7 GDP thresholds + BTC<$50k), bankroll $994.17. No new entries.
- Gate 4c/4d: Sept 17 correction stands — Gate 4c never broke, structurally can't reach the 7 pre-existing GDP positions. Now has real added weight from tonight's NFL_SPREAD correlation finding — worth treating Gate 4d's design question as touching more than one model.

---

## SPORTS MONITORING

No open positions requiring in-session monitoring today.

---

J@rv1s | Papa Ralph standard.
