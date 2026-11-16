# J@rv1s EOD Briefing, Mon Nov 16, 2026 (written 17:05 CST)

One consolidated briefing. No addenda. It also covers Sat Nov 14 and Sun Nov 15, for which no separate EOD was written; the rulings for those days are in the project docs listed in section 2.

## 0. PASTE TO ARCHIE (project docs, exact paths; paste in this order)
1. `claude/jarvls_eod_briefing_20261116.md` (this briefing)
2. `claude/jarvls_oracle_seeds_check_and_drift_accepted_20261116.md` (today's ruling: Oracle seeds check accepted, rebase answer accepted, run-day order restated)
Already pasted earlier and not needed again: `jarvls_seeds_mtime_accepted_run_day_nov17_20261115`, `jarvls_command_line_and_convention_rulings_20261115`, `jarvls_run_day_checklist_rulings_20261115`.
Not for Archie: the prompt-generator discussion (Rus's side idea, no project doc, nothing for the SEDE files).

## 1. Bottom line
- Stage 1 of the KXHIGHLAX forward record runs TOMORROW, Tue Nov 17, after 15:00 CT, started by 18:30 CT at the latest, with Rus at the machine. One invocation, irreversible. Nothing else happens that day.
- The Nov 17 date reached me secondhand (Archie's report of what Rus said Sun 15:41 CST). Rus read the ask this morning and did not correct it. Treat it as picked.
- Nothing in SEDE is demonstrated. Phase 2 NFL_GAME verdict Thu Jan 14, 2027, Ruling 1a (counts and points only until then). Falsification rule v0.2 unchanged. The one-in-one-out slot is held by KXHIGHLAX (FINAL saved Nov 14, 14:17:03 CST, so the Nov 16 lapse did not apply).

## 2. Rulings in force (all in project docs; newest last)
- FINAL spec saved and accepted (`weather_forward_record_spec_FINAL_20261114`): text SHA256 `52F7595782D53D3F83C37918C8027ABA93A8EED5F7E1FFB06DF0B21243B68B94`, 31,324 bytes, seed 20261114. Errata not edited: spec says 6.56c in one place, the script computes about 6.58c; the script governs.
- Calibration: nine rows scored in place, committed and pushed `d366fc4` (Nov 14 20:21 CT).
- Run-day checklist F1-F4, Q1-Q6, A-F (`jarvls_run_day_checklist_rulings_20261115`). F3: the script prints a `->` mark on descriptive rows; the spec wins; only the `STAGE 1 LABEL:` line is the label.
- Command line confirmed (`jarvls_command_line_and_convention_rulings_20261115`); endpoint convention ruled (implied YES iff floor <= TMAX <= cap, whole degrees F, both ends inclusive; 10 blank-strike dates derive floor/cap from the ticker midpoint, listed by name). NCEI cross-check defined and ordered after the label is in the project.
- Seeds state: laptop runtime state, Nov 5 mtime explained (autostash pop), Oracle copy clean and equal to the Sept 24 commit, bridge write path laptop-only. Closed. Two unverified edges carried to alert scoping (any Oracle reader of the state file; Oracle's full crontab), plus the `data_freshness.py` line 364 discard-list risk.
- Archie's Nov 5 `git pull --rebase --autostash` ruled DRIFT, not an exception. Protocol form from here on (merge, not rebase).

## 3. Tomorrow's run day, in order
Archie, on the day, not before:
1. Clock read (CT); confirm Rus is present and the window is open (after 15:00, start by 18:30, finish well before 20:00).
2. Pre-run steps 1-6 from the checklist. Hash `stage1_analysis.py` and show it equals `F56B788E9052C5ED56B3C23B6B58E3278AF7D6C5124A008D5E2E83B35C591755`. Hash the laptop spec copy and show it equals `52F7595782D53D3F83C37918C8027ABA93A8EED5F7E1FFB06DF0B21243B68B94` (the script only records `--spec-sha256`; it does not verify it). Compare the store copy by his method; I read the store timestamp too.
3. Selftest: expect `SELFTEST PASSED (46 checks)`. Record Python, numpy, pytz versions; Nov 15 values were 3.12.10 / 1.26.4 / 2026.2, and any difference is reported, not fixed.
4. Build the command with `<D>` = 2026-11-17 and `<DS>` = 20261117 from one `Get-Date`. Print the expanded argument list. `Test-Path $o` must print False.
5. Send me the results. I answer CLEAR TO RUN or NOT CLEAR. NO invocation before my answer.
6. One invocation. Log ExitCode and the sizes and hashes of stdout, stderr and the output file.
7. Report in the ruled order (F3 header first). P1 abort or a crash after the sealed file opens: report the line, no diagnosis, wait for me; one repair shared between abort and crash; a second failure is NOT RUN and goes to Rus.
8. After the label is in the project: the centicent flip check by hand (C = 3.0882c and 3.448c, arithmetic shown, no module import), then the NCEI cross-check (fetch only then). Neither can change the label.
9. Calibration rows scored only on my ruling.
No git, no Oracle contact, no scheduling, no alert work, no commit on run day.

What I will check for CLEAR TO RUN: the three hashes above; 46 of 46; versions; the printed arguments character by character against the confirmed form; `Test-Path` False; the clock inside the window; Rus present; sealed files still unopened and unhashed.

## 4. Likeliest outcomes (spec section 10, unchanged)
UNRESOLVED about 80%, GAP BELOW COST about 14%, MISCALIBRATED AT THE ASK about 6%. Rehearsal false-alarm rate about 8% at true gap 0, so a GAP BELOW COST label is not a finding until replicated. UNRESOLVED means "NOT DETECTED at this sample", never "calibrated". The result is a cheap, closed question and is not evidence for or against Thesis H.

## 5. Calibration
Pending rows that resolve at the Stage 1 run: my "P1 abort does not fire" 93% and Archie's 95%; the Stage 1 label rows (80 / 14 / 6 as logged); the "decision-relevant result 15%" row. Judge-line row (kept at Nov 19 review, 55%) resolves Nov 19. Archie's NFL-after-Jan-10 row is his.
NEW row, OK to append locally Monday night or Tuesday morning BEFORE the run (no git; commit Wed after a 10:15 CT run, pull first): `Stage 1 is invoked on Tue Nov 17 (does not slip)`, JARVLS, Process, 85%, deadline 2026-11-17. Reason: the lock already slipped once.

## 6. State carried (NOT re-verified today)
- Oracle status, logs and open positions are as of Archie's nightly summary closed Sun Nov 15 ~18:30 CST. The Fri 10:15 and 20:00 CT run logs and the weekend runs are unverified. I fetched nothing. Recommend a read-only log check Wednesday, not Tuesday.
- Open GDP positions are on the Jan 28, Apr 29 and Jul 29, 2027 releases. Marks are unrealized, thin markets, not evidence. GDP n_events = 2.
- JOBS and CPI: as carried in the project instructions and the Nov 9-10 docs; I have not re-read their state this week.
- Laptop repo: branch `mlb-backfill-fix-20261102`, HEAD `9c2c36d`, 0 ahead / 0 behind. Only dirty file `data/seeds_bridge_state.json` (runtime state, NOT to be committed or modified).
- Sports monitoring: no sports positions known open; not checked.
- Alert patch: waits for my ruling, earliest Wed after the run. DATA-block `_safe_text` fix stays its own later commit. No Phase 2 hashed file edits.

## 7. Dates
Tue Nov 17 Stage 1 run. Thu Nov 19 Judge-line review (mine). Mon Nov 23 income-side decision date (parked, not chased). Dec 1 daily-rain count re-check only. Thu Jan 14, 2027 Phase 2 verdict. Thu Jan 28 GDP print. Jan 31 falsification-rule checkpoint (second pre-registered test lock) and sunset date.

## 8. Income side and other (Rus; not SEDE)
- Income decisions and employer-disclosure are parked by Rus. Standing condition only: no bids before he decides on disclosure. The scan spec draft stays in the project.
- Prompt-generator idea, researched today: the general version is crowded; expert-persona prompting does not reliably improve factual accuracy (Wharton GAIL Dec 2025; a Mar 2026 study); Rus said no to building the playbook. Closed. If he wants the brain-supplement question researched, he asks me directly.

## 9. Tonight's work order for Archie (Mon evening)
1. Optional but wanted: the Monday note with the real command text filled in (2026-11-17 / 20261117) and the F3 header, to me. I check the string, not the template.
2. Append the one new calibration row in section 5 locally, if Rus is in a session. No git.
3. Nothing else. No SEDE code, no Oracle, no Phase 2 files. Nightly summary held until Rus says "end session".
Judge: the filled-in command text matches the confirmed form character for character with D = 2026-11-17, DS = 20261117 and the spec hash `52F7...8B94`; I compare string to string, and Archie shows the `Get-Date` read that produced `<D>` and `<DS>`.

## 10. Honest flags
- This is a single irreversible run with a likeliest result of "not detected". Do not let anyone, me included, read a UNRESOLVED label as a verdict on SEDE.
- A GAP BELOW COST or MISCALIBRATED label gets reported with the rehearsal false-alarm rate next to it, and nothing is built on it before replication (falsification rule F3).
- The lock slipped once already. Run day is for the run and nothing else, including side ideas.
- Rus's Nov 17 pick is secondhand through Archie; a mismatch Tuesday morning is cheap to fix, a mismatch at 18:15 is not.
- My bias to watch: deep in spec detail feels productive, and the run is now the only thing left to do.

J@rv1s
