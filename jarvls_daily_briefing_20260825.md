SEDE-SEEKS DAILY BRIEFING — J@rv1s (Web Claude)
Date: August 25, 2026 (Tuesday)
For: Archie (Claude Desktop) — evening session

---

SESSION SUMMARY

Full day spent on a broad "what's out there that could help us" research
thread at Rus's request, spanning four separate asks: a specific tool
evaluation (Omni Route), an open-ended repo/idea search across SEDE
through SEEKS, a second specific tool evaluation (gstack), and a
feasibility question about mining TikTok for ideas. One clear
recommendation for tonight's session, several real reference finds
logged for later, two tools declined with reasons, and one method
(direct GitHub API search) established as a repeatable process going
forward. No code changes made today — this was entirely an evaluation
day.

---

RECOMMENDED FOR TONIGHT — THE FIVE-GATE RISK-CHECK AUDIT

Sourced from reviewing OctagonAI/kalshi-deep-trading-bot: production-
grade risk management as a STACK of independent, named, all-must-pass
gates (Kelly sizing, Liquidity, Correlation, Concentration, Drawdown) —
not one blended score.

Action: trace SEDE's live entry path (portfolio_manager_sede.py) and
confirm whether existing checks -- edge >=15c, model confidence >=60%,
SEDE confidence HIGH, Gate 4b same-event concentration cap (12%), hard
drawdown stop (-$200/20%), max 8 concurrent positions -- are structured
as independent, ordered, all-must-pass gates, or whether any are
blended in a way that could let one weak input slip through if another
looks strong. Natural low-cost follow-on to the Aug 11 Gate 4b build --
confirm the whole stack has the same discipline as that one gate.
Bounded: read-and-confirm only, no fixes unless something trivial
turns up (same discipline as the Aug 24 title-matching audit).

Reinforcement found later today, worth folding into the same audit:
MihirM9/polymarket-weather-bot (found via GitHub API search, below)
runs a 9-layer risk-gate stack -- one layer deeper than OctagonAI's 5.
Confirms this pattern (explicit, independent, stacked gates) is the
real industry norm, not overcaution -- worth checking SEDE's stack
against this depth too, not just the 5-gate reference.

---

TOOL EVALUATIONS — DECLINED, WITH REASONS

**Omni Route** — DECLINED. Two unrelated products share the name.
One is a third-party local proxy that silently reroutes Claude Code's
API calls to unverified free-tier models under Anthropic-branded
aliases -- real credential/correctness risk on a machine holding
Oracle SSH keys and production trading code. The other is an unrelated
omnichannel chatbot-routing skill, not applicable. No further action.

**kalshi-trading-mcp** — code-verified (actually cloned and read the
auth/client/safety source, not just docs). Auth (RSA-PSS signing) is
correct and clean, private key never leaves the local machine, every
hardcoded outbound domain traced and legitimate (Kalshi + known public
weather APIs, no hidden telemetry). BUT its safety-gate stack is
almost entirely scoped to weather-market tickers (KXHIGH/KXLOW) --
would be a no-op for SEDE's actual GDP/JOBS/sports markets -- and its
daily-limit/pending-order state is in-memory only, no persistence,
which conflicts with Oracle's cron-based (not continuously-running)
architecture. Recommendation: if ever adopted, ONLY for read-only
Kalshi queries during Archie's interactive sessions (balance,
positions, prices) as a possible future replacement for the SSH/curl/
paste friction that delayed the b485776 confirmation this week. Never
wire in its order-execution tools. Not urgent, not time-sensitive.

**garrytan/gstack** — DECLINED. Real, popular (130k+ stars), code-
verified as clean (no malicious network calls, telemetry off by
default and clearly documented, cookie-import feature real but
explicitly gated behind user confirmation). Not a fit for two
structural reasons: (1) domain mismatch -- its 23 skills target web/
SaaS product development (browser QA, staging deploys, design review)
while SEDE's actual work is a cron-driven Python signal pipeline with
no user-facing web app; (2) process collision -- it's designed to
become THE methodology (rewrites CLAUDE.md, wants exclusive control of
browsing tools, imposes its own CEO/eng-manager decision framework),
which would compete with Papa Ralph/FORGE rather than complement it,
right after that process completed its first live test (EPL decision,
Aug 23). Possible narrow exception: /cso (security-audit methodology)
or /investigate (root-cause debugging framework) as reference reading
only, not adoption.

**TikTok content mining** — evaluated, recommend against, alternative
proposed. Searched TikTok's indexed content for AI-trading/prediction-
market material; results were dominated by low-quality hype and
affiliate-scam content, nothing verifiable. Structural problem, not a
bad search: short-form video can't be fact-checked the way code can,
and this project has been burned before by unverified claims (JOBS
placeholder data, Aug 11 fabrication incident). Building a scraper
isn't recommended either -- real ToS/maintenance burden, wouldn't
improve signal quality, and would compete with Sept 8 prep for
engineering time. Alternative: today's GitHub API search method (see
below) is the actual "find things instead of build them" answer --
verifiable, cheap, repeatable, no infrastructure needed.

---

GITHUB API SEARCH -- NEW, REPEATABLE METHOD ESTABLISHED, REAL FINDS

Rus asked whether public GitHub could be systematically searched for
relevant AI content. Yes -- GitHub's own search API is public, needs
no auth for read-only search, and can be queried directly with
recency filters (pushed:>date) and sort (stars/updated) across
targeted keyword sets. Ran a real pass tonight (5 categories, ~40
seconds, respecting the unauthenticated 10-req/min rate limit). Two
finds worth real attention, several worth knowing about:

**milesmcgarty/Premier-League-Prediction-2026-27** -- directly
relevant to the active EPL build. Cross-division Elo validated at
0.955 Spearman correlation against ClubElo's own published ratings,
feeding a Dixon-Coles Poisson model -- same architecture already in
soccer_game_model.py. The useful part isn't the code, it's the
validation METHOD: comparing derived Elo directly against ClubElo's
published numbers as an independent external sanity check. Worth
running once the EPL carryover-blend build lands and produces its
first real Elo ratings -- a cheap, independent check before trusting
those ratings on live signals.

**phys-ai/llm-biased-consensus** -- real ICML 2026 paper + code,
"Emergence of Biased Consensus in Multi-Agent LLM Debates" -- directly
on-topic for the Convergent-Instance bias check named in Papa Ralph/
FORGE. Worth a read to potentially sharpen that check with a citable,
formal mechanism rather than just "did an external model catch it."

Smaller, reference-only, no action:
- gpowerf/Council-Mode-Skill -- heterogeneous multi-agent consensus
  skill, validates the external-Gemini-review pattern already used
  twice (Aug 11, Aug 23) is aligned with where the field is going.
- Kavish-Muthum/Prediction-Market-Calibration-Analysis-with-Brier --
  working Kalshi-specific reliability-curve + Brier Skill Score
  dashboard. Possible design reference for eventually fixing the
  frozen SEDE calibration line (RESOLVED_MARKETS staleness), not an
  adoption candidate.
- MihirM9/polymarket-weather-bot -- 9-layer risk-gate stack, folded
  into tonight's recommended audit above.

Recommend this GitHub-API method become a standing, periodic (monthly
or quarterly) low-priority backlog item -- cheap, verifiable, real
results, no infrastructure to build or maintain.

---

CARRIED FROM YESTERDAY (unchanged, not reverified today)

- b485776 (MLB schedule-state + match-rate hardening) -- Rus
  confirming with Archie directly tonight.
- NFP_CONSENSUS shipped and live (Aug 24) -- nothing to check before
  ~Aug 31 calendar window rollover.
- Real spread instrumentation for JOBS -- still never built, zero
  rows ever.
- GDP scoring stall since July 30 -- unchanged.
- GDP out-quarter methodology (Aug 18) -- still open.
- EPL carryover-blend build -- design closed out, not started.
- No new go-live target set since Aug 15 passed -- now flagged seven
  sessions running.

Papa Ralph standard. If it's worth doing it's worth doing right --
including giving three "no" verdicts and one clear "yes" the same
level of real, code-verified scrutiny rather than letting a busy
research day produce a pile of unvetted maybes.

