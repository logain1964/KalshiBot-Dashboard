# J@rv1s EOD Briefing, Tue Nov 24, 2026 (written about 17:00 CST)

One consolidated briefing. No addenda. SEDE-SEEKS only; non-SEDE work is excluded, as Rus asked. This replaces the Nov 23 EOD for Archie's purposes: Archie did not run last night, so everything in the Nov 23 work order is carried here in full.

## 0. PASTE TO ARCHIE (project docs, exact paths, in this order)
Paste from the saved files, with the filename line; chat paste strips underscores.
1. `claude/jarvls_eod_briefing_20261124.md` (this briefing)
2. `claude/jarvls_ssh_key_ruling_20261123.md` (the ruling; section 3 is the method)
3. `claude/jarvls_ssh_key_rus_confirmation_20261123.md` (adds the open-session step)
4. `claude/jarvls_token_scope_finding_20261123.md` and `claude/jarvls_github_security_log_read_20261123.md` (for the read-only credential checks in section 7)

**Rus: say "end session" to Archie at the end of tonight's session.** The nightly summary is held until that phrase, and Archie posts it to the project the same night.

## DATA LIMITS (read first)
- The newest Archie material I can see is the **Nov 22 nightly summary**. There is no Nov 23 summary and no Archie report dated Nov 23 or Nov 24. Rus said this morning he could not get on an Archie session last night.
- I have not seen Oracle's Nov 23 20:00 CT run or today's 06:00 and 10:15 CT runs. The 20:00 CT run is still ahead tonight. GitHub raw fetches stay unreliable from the web chat; I did not try them.
- Rus answered some of my security questions in chat this morning (the Chicago addresses, a second laptop, "Wren"). My working record of that exchange is compressed, so I do **not** mark Rus-only items 1 and 2 in section 9 closed. If they are settled, Rus restates each in one line and I close them.
- Nothing below is a validation claim about any model.

## 1. Bottom line
- **No new verified SEDE movement today.** No new data, no new run evidence reached me. No data in a day is not a result.
- **The Oracle SSH key swap is still NOT done.** It is tonight's job (section 7), if Rus opens a session. Until then the only key that opens the box that runs the pipeline is still in `KalshiBot` git history, where it has sat since Jun 6. Nothing found so far shows misuse (section 9).
- **Unchanged:**
  - Nothing in SEDE is demonstrated.
  - Phase 2 NFL_GAME verdict is Thu Jan 14, 2027, and checkpoints report counts and points only until then.
  - Falsification rule v0.2 stands.
  - Gate 1 has been project-wide provisionally suspended since Aug 11.

## 2. Oracle Cloud status
Last verified from Rus's read after the Nov 22 20:00 CT run: `HEAD` `9c2ab0f` ("Auto-update 2026-11-22 20:00 CT", on top of `cb4c562`), tree clean, `models/nfl_model.py` full 64-character hash match, the five Phase 2 hashed files equal to the lock-time manifest, 5 open positions, 0 entries since Nov 15. **Runs since then are unverified to me.** Rus runs every Oracle command, and nothing is pushed or pulled around 06:00, 10:15 and 20:00 CT. Laptop `C:\KalshiBot` was at `cb4c562`, one commit behind origin, as of Nov 22; not pulled; merge, not rebase, when needed.

## 3. Open positions
Last confirmed Nov 22: 5 open in `sede_portfolio.json`, 1 BTC NO and 4 GDP YES, none entered after Aug 1. The GDP positions sit on the Jan 28, Apr 29 and Jul 29, 2027 releases (unrealized marks, thin markets, not evidence). No new entries; nothing has cleared the entry criteria. Criterion 4 caveat: `auto_monitor.py` covers only `paper_trades.json`.

## 4. Validation tracker (unchanged; none re-read today)
- **NFL_GAME:** no demonstrated edge, entry weight 0, accumulating, verdict Jan 14, 2027.
- **GDP:** n_events 2, Gate 1 not met; next event Jan 28, 2027.
- **JOBS:** unproven. **CPI:** state not re-read by me.
- **MLB_GAME:** fully suspended; report only the verified tickered subset (61 rows, 24 wins 39.3%; 44 games, first row per game 19 of 44, 43.2%), descriptive; offseason.
- **CLAIMS:** suspended. **NFL_SPREAD, MLS_GAME, SOCCER_GAME:** tail finding stands (SOCCER_GAME rests on the unverified hand table). **NHL_GAME / NBA_GAME:** log-only, suspended.
- **Weather:** KXHIGHLAX Stage 1 UNRESOLVED, series closed NOT DETECTED at the stated n, slot released. Daily rain: count re-check Dec 1 only.

## 5. Calibration
- **Owed to the file, from my Nov 23 ruling (Archie appends at the next calibration commit; one named path, exactly the inserted lines, nothing else staged, five-file manifest check after):**
  - "The GitHub token that sat in the `C:\Claude AI` remote URL is dead", 90%, TRUE, Brier 0.01.
  - "The Oracle login key was never committed to any repo", 90%, FALSE (`7be4731`, `KalshiBot`, Jun 6), Brier 0.81. The broad claim goes in the prediction text; the scope note (my Nov 22 ruling was scoped to `C:\Claude AI`) goes in the notes column.
- **Open predictions to log, unresolved:**
  - Nobody but Rus has used the old key: 93%.
  - Every action in the GitHub security log is Rus's own: 85%. This waits on his answers on the two Chicago addresses and the Copilot agent.
  - The key swap goes cleanly with the two-window method: 85%.
  - Archie puts them in the file's column format.
- I have still not read the published `jarvls_calibration.csv` myself.

## 6. Sports monitoring
No sports positions known open; not checked.

## 7. Work order for Archie
**Tonight, only if Rus opens a session.** The key swap is the one change; everything else is read-only. No SEDE code, no Phase 2 files.
1. **Before the swap, read-only, file paths only:** find anything that uses the old key (`sede_production`) in scripts, scheduled tasks, config and notes under the three repo roots and Archie's own session files. Nothing may break silently when the old line goes.
2. **Key swap, per the ruling (section 3 of `jarvls_ssh_key_ruling_20261123`):**
   - New ed25519 pair on the laptop with a passphrase (Rus's call), stored in the user `.ssh` folder and outside every repo.
   - Keep the current Oracle session open.
   - Add the new public key as a NEW line in `authorized_keys`; do not touch the old line.
   - Second window: log in with the new key. Only if that works, remove the old line, then log in again from a third window.
   - Same sitting, read-only: ownership and mode of `~/.ssh` and `authorized_keys`, and `sshd`'s password-login setting.
3. **Close the old session.** Removing the old line does not end a session already open. The Nov 15 22:24 session has been open about nine days. After the new key works, end it (Rus closes it, or Archie supplies one command) and confirm with `last` / `who` that nothing is attached under the old key.
4. **Timing:** after 16:00 CT and before 19:30 CT, or after 20:30 CT. Not across the 20:00 CT run. If Rus starts after 19:30, wait until the 20:00 run has finished.
5. **After the swap, Rus:** delete the old key files, both copies (`C:\KalshiBot\oracle_ssh\` and `C:\Claude AI`), once the new login is confirmed. The tracked public key file and the `.gitignore` key pattern stay.
6. **Persistence sweep on Oracle, read-only, same night, after the swap:** accounts with a login shell; `crontab -l` for `ubuntu` and `root`; `/etc/cron.d`; systemd timers; every `authorized_keys` file on the box (paths and sizes). Anything unfamiliar comes to me before anyone touches it.
7. **Read-only credential checks:**
   - `stat` of `/home/ubuntu/.git-credentials` (mtime only, never the content). If the mtime is before Oct 12, Oracle holds a token other than the regenerated `Kalshibot-SEDE`, so another live fine-grained token exists.
   - Confirm `cb4c562` is an ancestor of `9c2ab0f`.
8. Open on Archie, low priority: does a failed pull show loudly in the report (my question 4)? Answer from the update script and one saved report.
9. Calibration rows (section 5). Nightly summary after "end session", posted to the project the same night.
10. **Judge-line rule:** every Judge line is tagged INDEPENDENT or SELF-CHECKED.
11. **Not tonight, own task, not in the same session as the swap:** give Oracle a read-only credential for `KalshiBot` only (a second fine-grained token with Contents read-only on that one repo, or a read-only deploy key). The laptop keeps the write token. Away from the 06:00, 10:15 and 20:00 CT pulls. Archie writes the commands; Rus runs them.
12. **Untouched by order:** alert patch, Phase 2 hashed files, scheduling, SEDE code, new models, the empty slot, a history scrub of `KalshiBot` (not approved; it does not undo fetched copies and puts shared `main` at risk).

**Judge (INDEPENDENT):** `last`, `who` and `authorized_keys` as printed on Oracle and pasted unedited by Rus; the new-key login working from a separate window; the `stat` output. None is an Archie test.

## 8. Dates
- **Tonight:** SSH key swap window, 16:00 to 19:30 CT or after 20:30 CT.
- **Tue Dec 1:** daily-rain count re-check only.
- **Thu Dec 3:** Judge-line tag count.
- **Thu Jan 14, 2027:** Phase 2 verdict.
- **Thu Jan 28:** GDP print.
- **Sun Jan 31:** falsification-rule checkpoint and sunset date.
- **Tue Sep 14, 2027:** rotate `Kalshibot-SEDE` (expires Oct 12, 2027), with "update Oracle's store file" as a named step.

## 9. Honest flags
- **Rus-only items still open** (carried from Nov 23; see the data limit above on items 1 and 2):
  1. Whose are `198.14.83.51` and `198.14.80.209` (Chicago, not his Wichita home)? They cover the Oct 5 Claude app repo grant, the Oct 12 token regeneration, the Copilot agent setup and the Nov 15 login. VPN, phone, work, second laptop?
  2. Did he set up the Copilot SWE Agent?
  3. The Claude GitHub App has access to `KalshiBot` (private) and `KalshiBot-Dashboard`. That path does not show on the collaborator list. Check its permissions (Settings, Applications, Installed GitHub Apps, Claude, Configure).
  4. How many tokens are on the fine-grained tokens list?
  5. The Deploy keys and Webhooks pages on `KalshiBot` (should be empty or recognizable).
  6. Optional: the Applications page and the Aug 25 to Sep 30 stretch of the security log.
  7. Delete the extra key copy in `C:\Claude AI` after the swap (J@rv1s: yes).
- **Confidence, stated:** 93% nobody else used the old key since Jun 8; 85% every logged GitHub action is Rus's own. Still not covered: Jun 6 to Jun 8 (about 37 hours with no login log), failed attempts, sessions without a terminal, and GitHub's roughly 90-day window on the security log. An address match shows the connection came from his network, not whose hands were on it. A second machine on his own network does not change that; it needs to be named so I can stop counting it as unknown.
- **The key has now sat open one more day.** Not an emergency, no evidence of misuse. But the swap has slipped twice, and "tonight" keeps meaning "if a session happens". If Rus cannot open Archie tonight, name the next window now so it is a date and not a hope.
- **My miss, on the record:** on Nov 22 I ordered the key-history check on `C:\Claude AI` only and put 90% on "never committed". That was right for that repo and irrelevant to the question. Archie widened the check and caught it. Test the premise before writing the flag.
- **Not re-read by me today:** Oracle runs since Nov 22, JOBS, CPI, the published calibration CSV, laptop repo state beyond Archie's Nov 22 report.
- **The slot is empty and nobody is entitled to fill it by reflex.** One-in, one-out stands; Rus's hold on the crypto-threshold and book-vs-Kalshi poller stands.
- **Decision owed to Rus before Jan 14:** the sunset date is Jan 31 unless a second pre-registered test is locked by then (which moves it to Jun 30, 2027). Stage 1 does not count as that test. Name it early so it is a choice and not a surprise.
- **Income-side decision (due Mon Nov 23):** parked, not chased. Rus can re-park it.
- **My bias to watch:** a second day of security rulings with no SEDE evidence feels like progress while SEDE's real test, Jan 14, sits untouched. The key exposure is a finding about hygiene, not about the models; do not let it displace the validation work.

J@rv1s | Papa Ralph standard.
