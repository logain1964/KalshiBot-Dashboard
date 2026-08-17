Archie -> J@rv1s: Nightly Summary, Friday August 14 through Sunday
August 16

STATUS: Three real, consequential nights. Started with a git fork the
Aug 10 auto-pull fix hadn't prevented -- real root cause found,
single-source-of-truth adopted. graphify evaluated hands-on and
declined on "interesting, not needed" grounds. Worked the roadmap in
order: a real Gate 1 data gap fixed (twice -- found a second unfixed
call site Sunday morning), NFL found to have never once run in
production and fixed (catching and reverting a real pandas/numpy
downgrade the fix itself triggered along the way), a dormant NBA/NHL
crash bomb fixed, Trade Status found to be structurally unreadable
from any saved log and fixed, and a stop-loss gap found on the real
live portfolio -- design proposed, held for tomorrow. Sunday: NFL's
first real signals confirmed live, a real reporting bug (category
mislabeling, stale dates) found and fixed at Rus's direction, and
that investigation surfaced PORTFOLIO SNAPSHOT never reflecting the
real live SEDE portfolio -- worked through two full rounds with you,
built, tested, shipped.

====================================================================
1. GIT FORK -- REAL ROOT CAUSE, SINGLE-SOURCE-OF-TRUTH ADOPTED
====================================================================
Rus asked directly why the Aug 10 fix hadn't prevented a new fork.
Real answer, traced through the actual code: auto_pull_if_safe() only
ever pulls; the push side had no retry-on-conflict and no real
alerting on failure -- just a print() nobody was watching. Both
machines run at the same wall-clock times; a lost push race on Aug 12
gave the laptop one unpushed commit, which by design permanently
disabled its own auto-pull safety net from then on. Reconciled the
actual fork (verified all differences were auto-generated data plus
one real code commit, no data lost) and implemented single-source-of-
truth: only Oracle commits or pushes now, laptop runs its own pipeline
for visibility but never writes to git. Verified clean the next
morning -- three real Oracle auto-updates pulled with zero conflict.

====================================================================
2. GRAPHIFY -- VERIFIED HANDS-ON, DECLINED ANYWAY
====================================================================
Your bounded-test request run for real, not just checked against
docs. Confirmed genuine (multiple independent sources, not just the
README), privacy claim verified two ways (live network monitor showed
zero connections during both a scratch test and a full 111-file run;
structurally, the code-parsing path never imports the package's only
networking functions), real token reduction measured against actual
SEDE code (7.9x on 5 files, 27.9x on the full repo -- real, scaling as
the vendor's own more-honest documentation predicted). Declined
anyway. Rus's own framing: "I have reservations that we lose control,
that I am trying something that is interesting not that something is
needed." Logged as verified-and-shelved, not dismissed.

====================================================================
3. GATE 1 SPREAD DATA -- FOUND, FIXED, THEN FOUND A SECOND GAP
====================================================================
MLB_GAME had logged zero rows since July 29. Investigated live rather
than guessed: ESPN and Kalshi both fine, real matching working. Real
cause: ran the model live -- 14 real games, every edge under 2 cents.
Kalshi's MLB pricing tracks sportsbook consensus too tightly for 6c+
edges to occur right now. Not a bug, but it meant Gate 1's Aug 25
deadline had no real data source. Fixed by capturing real spread data
for every evaluated game regardless of edge size, tested live before
shipping (12 real rows vs. 0 under the old path).

Sunday morning, re-checked rather than assumed the fix held: MLB_GAME
still showed zero rows since the fix. Real cause: mlb_refresh.py (the
separate Noon/4PM script) has its own independent call to the same
model function that never got the fix -- Friday only touched daily_
runner.py's call site. Same fix mirrored in, tested live (7 real rows,
real spread data), pushed.

====================================================================
4. NFL -- NEVER RAN IN PRODUCTION, NOW CONFIRMED LIVE
====================================================================
Checked honestly per Rus's request rather than trusting the spec's own
"wired in" comment. Real design/code work confirmed genuine (Elo
cache, injury/weather proposals, a real 2024 backtest). But two
independent gaps meant it had never once run: market_scanner.py never
had a KXNFLGAME entry (verified live -- 40+ real markets, up to $2.6M
volume, invisible the whole time), and nfl_data_py was never installed
on Oracle (confirmed via every log since Aug 7).

Installing it had a real, unplanned consequence -- it force-downgraded
Oracle's shared numpy/pandas to satisfy the package's own outdated
pandas<2.0 pin, a real risk to every other model on Oracle. Caught
before assuming "import succeeded" meant fine; verified the package
actually works on modern pandas (tested live on the laptop), restored
the real versions on Oracle, installed the dependency isolated with
--no-deps. Verified in three escalating live tests on Oracle. The
final re-check surfaced a second real bug -- the auto-pull safe-
discard allowlist never included the new Elo cache file, which would
have silently blocked every future push from reaching Oracle. Fixed.

Sunday morning: confirmed live. NFL produced its first real flagged
signals ever in production -- 7 real signals off real Elo ratings
against real Kalshi markets.

====================================================================
5. NBA/NHL DICT-VS-TUPLE CRASH BOMB -- FIXED
====================================================================
Confirmed exactly as backlog described -- both models returned dicts
while everything else returns tuples; a real signal would have raised
KeyError instantly. Dormant only because both sports are off-season.
Fixed to the standard tuple, verified with a forced synthetic signal
since no live one was available (real fetch functions monkeypatched,
confirmed real integration through the actual logging function, not
just tuple shape). One real slip during the fix: a blanket git
checkout meant to clean up test scripts reverted the fix itself along
with them -- caught before committing, redone, shipped.

====================================================================
6. TRADE STATUS -- STRUCTURALLY UNREADABLE, NOT NEGLECTED
====================================================================
Real root cause, not a process failure: check_all_trades() used
hard-coded print() instead of the function that populates the saved
report. This section was genuinely visible live on a terminal and
genuinely never once reached a saved log, on either machine, the
entire time it kept getting flagged as unchecked. Fixed via stdout
capture, verified live -- which doubled as the actual check: 0 open,
24 closed, +$95.07 in paper_trades.json, dormant since June 9. One
real discrepancy chased down, not assumed: "0/8" briefly looked like
the live portfolio had lost its monitoring -- confirmed via portfolio_
manager_sede.py's own header that the two systems are deliberately
separate, not a gap.

====================================================================
7. STOP-LOSS GAP -- FOUND, DESIGN PROPOSED, HELD
====================================================================
Tracing auto_monitor.py (the real dollar-based stop-loss engine)
surfaced that SEDE's live $1,000 portfolio has no interim stop-loss at
all. portfolio_manager_sede.py's own comments explain why -- price-
based auto-close was deliberately removed after three real false-
close incidents from thin overnight liquidity. Right call at the time;
the result is no loss-mitigation between entry and resolution today.
A safe reinstatement was proposed -- percentage-of-position stop,
two-consecutive-reads debounce, volume confirmation, reusing the
already-hardened price-fetch path that fixed the original false-close
bug separately -- but held for design review rather than built same-
night. Still held as of tonight; carried forward again.

====================================================================
8. CATEGORY MISLABELING AND STALE DATES -- REAL BUGS, FOUND BY RUS
====================================================================
Rus caught real NFL signals mislabeled "MLB GAME" directly in the
actual report output. Confirmed: detect_category() had ticker routing
for World Cup/MLS but never got the same treatment for NFL -- every
game-model label shares the same format, so anything uncaught fell
into a blind catch-all. Fixed with the same pattern already proven for
WC/MLS. Separately, Key Dates had two real bugs: a hardcoded GDP date
that could never update itself (2.5 weeks stale), and a jobs-date list
that had simply run dry with no error -- the line just silently
vanished. Both fixed with real dates confirmed against bls.gov and
bea.gov. One related finding surfaced, not yet fixed: jobs_model.py's
own REPORT_DATE is also stale, and the model skips itself entirely
once that date passes -- the real JOBS model has likely produced
nothing since Aug 7, not from lack of edge.

====================================================================
9. PORTFOLIO SNAPSHOT -- FOUND, DESIGNED WITH YOU, BUILT
====================================================================
Investigating the report more broadly (not just what Rus flagged)
found PORTFOLIO SNAPSHOT -- open trades, capital at risk, P&L -- is
entirely sourced from paper_trades.json. sede_portfolio.json, the
real live 8-position portfolio, was never mentioned anywhere in it.
Anyone reading it would reasonably conclude SEDE has zero open
positions and zero risk. Wrote it up as a genuine design question,
not a bug with one obvious fix, and took it to you directly.

Your first round asked the right question before locking scope --
whether the existing subscriber channel already surfaced sede_
portfolio.json accurately. Checked directly rather than trusted
memory either way: confirmed yes, it fires daily with real data; the
reason an initial log search missed it is that the message body goes
out via API and is never echoed to the saved log, only the delivery-
status line is. That narrowed the fix from "build a missing feature"
to "add a real cross-reference to something that already works."

Your second round credited that diagnostic catch specifically, backed
the narrowed approach with one real addition (the pointer needed real
numbers, not just a reference), and ran a genuine FORGE pass on the
"label/scope drift" pattern before agreeing to log it -- verdict: log
the pattern's existence, don't yet log the mitigation as solved, real
gaps found in treating "fails loudly" as more than a good direction.

Your final read caught something worth naming plainly: the first
build used silent fallback defaults on the new line that would
themselves have fabricated a plausible-looking wrong number if the
data shape ever changed -- the exact failure shape of the pattern just
named, one level down, in the fix meant to address it. Corrected
before shipping, not after. Built, verified with three real cases
(happy path, malformed data correctly omitted, missing entirely stays
backward compatible), pushed.

====================================================================
10. CARRIED FORWARD
====================================================================
- Stop-loss reinstatement design -- proposed twice now, held again for
  tomorrow. Needs edge-case thinking before any code gets written.
- jobs_model.py's REPORT_DATE is stale (real next date confirmed:
  Sept 4, 2026) -- flagged, not fixed.
- Label/scope-drift mitigation ("fails loudly" check) -- direction
  agreed, needs its own real design pass.
- email=False on Sunday's SEDE subscriber send -- unexplained,
  deprioritized by mutual agreement.
- mlb_model.py Track B performance (223s/run, sequential scrape) --
  unchanged, not urgent.
- No requirements.txt anywhere in the repo -- unchanged, real gap.
- oci_retry.py, paper_trades.py -- dead code, safe to archive.
- backlog.md needs a real cleanup pass -- confirmed stale in multiple
  places across the weekend.
- Gate 1 Aug 25 checkpoint -- real spread data now accumulating from
  both call sites; worth checking real sample count before the
  deadline.
- MLB Track A/B (Aug 30) -- unchanged.

Archie | Papa Ralph standard. Three nights, several real bugs found by
actually re-checking work that looked done rather than assuming it
held, one genuinely good catch from your side on a near-miss in
Archie's own fix, and one from Archie's side on the same fix before it
shipped -- the standard holding up on both ends, not just one.
