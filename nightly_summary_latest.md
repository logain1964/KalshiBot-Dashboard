# Nightly Summary - Archie - Fri Nov 13 to Sun Nov 15, 2026 (closed Sun Nov 15, ~18:30 CST)

Covers three sessions with no separate close (Nov 13, Nov 14, Nov 15); replaces the Nov 12 version. Nothing in SEDE code, Oracle, or Phase 2 hashed files was touched. Oracle: nothing run by me; Rus ran one read-only check on request (see item 6). The sealed KXHIGHLAX outcome files have never been opened or hashed. Stage 1 has NOT been run.

## NEEDS ATTENTION

### Rus
1. **Stage 1 run day is Tue Nov 17** (you picked it Sun 15:41 CST; J@rv1s took it as picked). You must be at the machine. Window: after 15:00 CT, started no later than 18:30, finished well before 20:00. Order: I do pre-run steps 1-6 plus the selftest and send results to J@rv1s; he answers CLEAR TO RUN or NOT CLEAR; only then I invoke once. No git operation, no Oracle contact, no alert work on run day.
2. **Monday (optional, J@rv1s asked):** I send J@rv1s the invocation text and F3 header with the real dates filled in (2026-11-17 / 20261117) so he checks the actual string. Needs a session on Monday.
3. Nothing is owed from you on Oracle; the optional read-only check is done (item 6).

### J@rv1s
1. Answer owed in my next note, per his Nov 15 ruling section 2: was the Nov 5 `git pull --rebase --autostash` an exception or drift from the "merge, not rebase" rule. My honest answer: drift, not a documented exception. Protocol form from here on; on run day no git operation at all.
2. Alert-patch scoping waits for his ruling and Rus; not on run day.

## WORK DONE (all saved as project docs, claude/*_20261113 to *_20261115)
1. **FINAL KXHIGHLAX spec saved Sat Nov 14, 14:17:03 CST** (store timestamp), seed 20261114, SHA256 `52F7595782D53D3F83C37918C8027ABA93A8EED5F7E1FFB06DF0B21243B68B94`, 31,324 bytes, LF. The Nov 16 slot-reservation deadline was met. Lock-day SHA256SUMS list: 45 lines (hash `C164153E...4642`). Laptop copy verified against it.
2. **Calibration scoring committed and pushed: `d366fc4`** (Nov 14 20:21 CST), nine J@rv1s rows scored plus five Archie rows. Records for the Nov 13 summary: commit `131ea21` (Nov 13 21:05 CST) also carried the Nov 11 and Nov 12 calibration rows and the 45% score; the Nov 4 Archie calibration rows reached git only in `d366fc4`. `jarvls_calibration.csv` is J@rv1s's file (CRLF, append-only), edited by raw line.
3. **Run-day checklist** drafted (F1-F4 script findings, 13 steps) and ruled on by J@rv1s. Script `stage1_analysis.py` frozen: SHA256 `F56B788E9052C5ED56B3C23B6B58E3278AF7D6C5124A008D5E2E83B35C591755`, 32,781 bytes. F1: no fee PDF in the project. F2: no centicent flip row printed (done by hand after the output is saved). F3: `->` labels print on descriptive rows; only the `STAGE 1 LABEL:` line is the label. F4: `--spec-sha256` recorded, not verified.
4. **Command line CONFIRMED** (J@rv1s R1): `python -I`, panel `history_panel_20261104`, `--seed 20261114`, `--confirm-open-sealed YES`, `Start-Process` with embedded-quoted paths (the space in "Claude AI" broke the first rehearsal) and stdout/stderr redirects. Added run-day items: print the expanded argument list and `Test-Path` on the output (must be False); log exit code and sizes/hashes of the three files.
5. **Endpoint convention RULED** (R2): implied YES iff floor <= TMAX <= cap, whole degrees F, both ends inclusive. An inference from pool structure (2,655 of 2,655 brackets width 1.0; midpoint minus floor 0.5), confirmed against rules text of currently open markets (probe `rules_between_probe_20261115`, three templates, 12 open markets); not read from historical markets' rules text. 10 blank-strike primary dates take floor = midpoint - 0.5, cap = midpoint + 0.5. NCEI cross-check only after the label exists; it never changes the label. Environment recorded: Python 3.12.10, numpy 1.26.4, pytz 2026.2.
6. **Seeds-state report (alert-patch precondition), ACCEPTED and CLOSED.** `post_seeds_bridge.py` on the laptop (Task Scheduler, SEEDS_Live 09:00 and SEEDS_Pitchers 14:00) writes `data/seeds_bridge_state.json` only when it alerts a new signal. Laptop copy (Oct 28 content, hash `37EDE2EF...3391`) is the live runtime state; the committed copy is a stale Sept 24 snapshot. The Nov 5 23:02:00 file mtime is explained: my `git pull --rebase --autostash` at 23:01:55 popped the dirty file (commit `bdd1b64` at 23:01:59). Oracle read-only check (Rus, 16:25 CST): Oracle's copy is clean and identical to the Sept 24 commit, no seeds cron entry, so the bridge write path is laptop-only. Not checked: whether any Oracle script reads the file itself; Oracle's full crontab. Recorded risk: `data_freshness.py` line 364 lists the file as safe to discard on auto-pull.

## STATE AT CLOSE
- KalshiBot laptop repo: branch `mlb-backfill-fix-20261102`, 0 ahead / 0 behind origin/main, HEAD `9c2c36d` (Nov 15 10:15 CT auto-update). Only dirty file: `data/seeds_bridge_state.json` (runtime state; NOT to be committed or modified).
- Dashboard repo: fast-forwarded to `2384322` (10 Oracle auto-updates and J@rv1s's Nov 13 briefing rename), then this file committed.
- Phase 2 hashed files untouched. Nothing scheduled. No outcome statistic computed on any data.

## CARRIED / NOT DONE
- Stage 1 run (Nov 17); NCEI cross-check and F2 flip arithmetic only after the label is saved and in the project; calibration rows (P1 abort does not fire 93% / 95%, Stage 1 label rows, 15% decision-relevant row) scored only on J@rv1s's ruling.
- P1 abort or crash after the sealed file opens: report the line, no diagnosis, wait for J@rv1s; one repair shared between abort and crash, a second failure = NOT RUN, goes to Rus.
- `_safe_text` DATA-block fix: its own later commit. Alert patch: after the run, after rulings.
- Fri 10:15 and 20:00 CT Oracle run logs: unverified.
- Nov 19 Judge-line review belongs to J@rv1s.

Archie
