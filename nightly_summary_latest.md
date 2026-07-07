# SEDE Nightly Summary — 2026-07-06 (Evening Session)

**For:** J@rv1s
**From:** Archie (web interface, this session)
**Session focus:** WC_GAME lambda "bug" was actually a display bug (fixed, verified), laptop duplicate cron found and disabled

---

## TL;DR FOR J@RV1S

The WC_GAME lambda compression fix you scoped tonight wasn't needed --
the underlying probability math was never broken. The real bug was a
universal display mislabeling that made most NO-signals across every
model look far less confident than they actually were. Fixed the
display, NOT the math. Also found and disabled a 2+ month duplicate
cron job on Rus's laptop that explains most of this week's merge
conflicts. Read the correction below carefully -- it affects how you
should read every future report with NO-direction signals.

---

## THE LAMBDA "BUG" WAS A DISPLAY BUG, NOT A MATH BUG

Your briefing cited five "confirmed lambda compression" cases (France
vs Paraguay 37.7%, USA vs Bosnia 53.3%, England vs Ghana 36.3%, Germany
vs Paraguay 42.9%, France vs Morocco 55.6%). Diagnostic step (per your
own instructions -- pull actual values before touching code) found every
single one of these numbers was the COMPLEMENT of the model's real
output, not the real output itself.

**Why:** every model's flagged.append() call for NO-direction signals
stores (1-kalshi_yes, 1-model_prob) -- correct internally, since that's
the probability of the side actually being bought. But the report/email/
Telegram formatters printed that complement labeled just "Model: X%",
which reads naturally as "the model's probability the named team wins"
when for NO signals it's actually 1 minus that.

Decoded correctly: France vs Paraguay was actually 62.3% (not 37.7%).
USA vs Bosnia was actually 46.7% (not 53.3% -- this one is genuinely
WORSE than you thought, a real underdog call on a team that won 2-0).
England vs Ghana was 63.7% (not 36.3%). Germany vs Paraguay was 57.1%
(not 42.9%). France vs Morocco was 44.4% (not 55.6%).

**Confirmed via independent fresh computation from world_cup_model.py's
own WC_TEAMS strength values and soccer_game_model.py's actual Poisson
math** -- not just reading the log differently, actually recomputing
from source and cross-checking against the logged values, which matched
exactly once the flip was accounted for.

**What this means for your future reviews:** don't trust "Model: X%" on
any NO-direction signal at face value going forward until you confirm
the fix (below) is live in whatever report you're reading. If you're
reviewing an OLDER report from before tonight, the NO-signal numbers in
it are the flipped complement, not the model's real confidence.

**Confirmed NOT affected:** the actual SEDE confidence scoring/tiering
(HIGH/MEDIUM/LOW) was never wrong -- the certainty calculation is
symmetric around 0.5, so this bug never caused a trade to be
mis-prioritized. It was purely a human-readability problem, but a
significant one that likely shaped how multiple past sessions (including
yours tonight) read these alerts.

---

## THE FIX -- AND A REAL MISTAKE CAUGHT MID-STREAM (worth reading)

Fixed the display layer in three files: `daily_runner.py`
(write_summary + compute_sede_confidence), `email_alerts.py` (fmt_signal),
`telegram_alerts.py` (send_telegram_signal + the live digest loop). All
now show the TRUE probability of the named outcome, explicitly labeled
as such, regardless of direction.

**First attempt at this fix (v1) was wrong and got caught before it went
live.** v1 assumed every model flips for NO signals the same way. Testing
against real GDP data showed `gdp_model.py` does NOT flip -- it stores
raw kalshi_yes/model_prob directly regardless of direction, confirmed by
reading its actual flagged.append() call. v1 was reverted (clean git
revert) same night, before Oracle's next scheduled run could have sent
out wrong GDP numbers. Audited every model file individually before
reapplying -- btc, claims, cpi, fed, jobs, mlb, nba (both game and
championship sections), nhl (both sections), soccer_game (MLS + WC_GAME),
world_cup_model (winner) all confirmed flip. GDP is the ONE exception.
v2 fix applied with an explicit `MODELS_THAT_DONT_FLIP` set, verified
against real signals for both cases before committing.

**Verified end-to-end with a real test send** -- actual email and
Telegram message, clearly labeled as a test, using tonight's real 6
flagged signals re-rendered through the fixed logic. Both delivered
correctly on inspection.

**Worth internalizing for your own future checks:** if a new model gets
added later, check its actual flagged.append() call directly before
assuming it follows the majority convention. Don't generalize from a
sample of "most models do X" -- verify each one.

---

## LAPTOP DUPLICATE CRON -- FOUND AND DISABLED (explains this week's conflicts)

Found while chasing an unrelated false alarm (my own mistake -- checked
whether the 7:45 AM cron fired by looking at my own un-pulled laptop
directory instead of pulling first; it had fired correctly all along).
That investigation led to discovering: **Rus's laptop has been running
a full duplicate `daily_runner.py` pipeline at 7:00 AM and 9:00 PM CT
every single day since April 30, 2026** -- parallel to Oracle's identical
schedule, via Windows Task Scheduler tasks "Kalshibot Full AM" and
"Kalshibot Full PM".

Confirmed via git commit author names (`logain1964` = laptop,
`Archie-OracleCloud` = Oracle) showing both machines committing at the
same labeled times every day going back at least to June 30 (likely
April 30). This is almost certainly the actual root cause behind every
merge conflict resolved this week, and very likely means Rus has been
getting two emails and two Telegram digests daily for 2+ months.

**Fixed:** both tasks disabled on the laptop, verified via
Get-ScheduledTask. Oracle is now the sole source of truth. If merge
conflicts recur, check whether something re-enabled these tasks before
assuming a new cause.

---

## OTHER FINDING -- DORMANT BUG, NOT FIXED TONIGHT

`nba_game_model.py` and `nhl_game_model.py` return dicts, not the
5-tuple format every other model uses. They get merged directly into
the same `all_flagged` list, which would likely crash write_summary()/
compute_sede_confidence()'s tuple-unpacking if either model ever
actually flags a signal. Apparently never triggered in practice --
every report reviewed shows both models skipping with "No markets
found." Logged to `docs/backlog.md` as medium priority: dormant but
will cause a real crash the day it fires. Worth fixing proactively.

---

## OPEN POSITIONS (unchanged)

### SEDE Portfolio
| ID | Market | Dir | Entry | Status |
|---|---|---|---|---|
| 1 | BTC<$50k Dec31 | NO | 43.5c | OPEN |

Bankroll: $994.17

### paper_trades.json
| # | Description | Entry | Status |
|---|---|---|---|
| 8 | Fed 1x cut YES | 21c | Documented loss, HOLD |
| 13 | GDP>2.0% YES | 60c | HELD -- calibration value reasoning |

Record unchanged: 8W-5L-5EE, +$139.14.

---

## STILL OPEN / CARRYING FORWARD

| Priority | Item |
|----------|------|
| 🟡 | nba/nhl_game_model dict-vs-tuple bug -- dormant, logged, not fixed. |
| 🟡 | UNEMP_CONSENSUS still 4.2% vs Dow Jones 4.3% -- flagged repeatedly, still undecided. |
| 🟡 | Trades #23/#24 close_note/pnl mismatch -- flagged repeatedly, still unfixed. |
| 🔴 | June 30 session archive file still missing -- flagged repeatedly. |

---

## KEY DATES

| Date | Event |
|---|---|
| Jul 7 | GDPNow next update |
| Jul 9 | Morocco vs France QF (Boston) -- now correctly read as a real 44.4%/55.6%/26.8% three-way, not a "37.7%" underestimate |
| Jul 10 | Spain/Portugal vs USA/Belgium QF (LA) |
| Jul 11 | Norway vs England QF (Miami) |
| Jul 14 | CPI release -- real first test of the 7:45 AM CT cron fix from July 2 |
| Jul 19 | World Cup Final |
| Jul 25 | MLB Track A/B verdict + NFL architecture parameters finalized |
| Jul 30 | GDP advance estimate -- Trade #13 resolves |
| Aug 2 | Vegas -- confirmed SOFT, self-imposed |
| Sep 3 | NFL season opens -- sole hard deadline |

---

*Session | Identity: Archie | Web interface*
*Lambda "bug" was a display bug -- underlying model math was never broken.*
*v1 of the display fix was wrong (GDP exception missed); caught and corrected same night before going live.*
*Laptop's 2+ month duplicate cron job found and disabled -- likely explains months of duplicate alerts.*
*Papa Ralph standard. If it's worth doing it's worth doing right.*
