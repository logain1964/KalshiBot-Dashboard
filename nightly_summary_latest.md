# Nightly Summary - Archie - Tue Nov 17, 2026 (session 17:42 to ~18:10 CST)

Stage 1 RUN DAY. The KXHIGHLAX Stage 1 test ran once and finished. Nothing in SEDE code, Oracle, or Phase 2 hashed files was touched. No Oracle commands run, no git on the main KalshiBot repo. Replaces the Nov 16 version.

## NEEDS ATTENTION

### Rus - WEDNESDAY Nov 18
1. After the 10:15 CT auto-update: I commit the two calibration files (`data/jarvls_calibration.csv`, `data/archie_calibration.csv`), pull first with a MERGE (not rebase), `data/seeds_bridge_state.json` stays out, nothing else swept in. Then J@rv1s checks the published copy.
2. One read-only Oracle log check (weekend and Nov 13-17 runs), which you run.

### J@rv1s
1. Tomorrow I propose the method for the g-with-and-without-2026-10-22 descriptive row. You rule on method and on a second read-only exception to the sealed outcomes file before anything runs. The driver must first reproduce the printed Stage 1 digits (n 667, g +2.80c, 90% [-0.19, +5.83], yes-rate 48.3%, mean m 45.5%, C 3.448c, seed 20261114, B 5000).
2. Nothing else is pending on me from your 18:00 ruling.

## RESULT
- **STAGE 1 LABEL: UNRESOLVED.** g +2.80c, 90% interval [-0.19, +5.83]c, C 3.45c, n 667 of 667, 1 of 668 dates excluded (0.1%), no COVERAGE LOW. The `->` marks on descriptive rows are not labels.
- Per spec section 6: the series closes as NOT DETECTED at this sample, the slot is released, no forward collector is built. NOT DETECTED does not mean "calibrated". **The slot is released with NO admission decided;** nothing is admitted into it without Rus under the one-in, one-out rule.
- J@rv1s's reading (accepted): the point estimate is not a trend; the lower bound -0.19c is a near-miss, not a finding; descriptive rows are noise-compatible and nothing is built on them.

## RUN RECORD
- Invoked once: 17:48:03 CST, finished 17:48:06, exit 0, stderr 0 bytes, no P1 abort line, script startup hash check OK on 5 files incl. both sealed files. I never listed, opened or hashed the sealed files for the run.
- Script `stage1_analysis.py` SHA256 `F56B788E9052C5ED56B3C23B6B58E3278AF7D6C5124A008D5E2E83B35C591755`; spec hash passed `52F7595782D53D3F83C37918C8027ABA93A8EED5F7E1FFB06DF0B21243B68B94`.
- Output `stage1_output_20261117.out` 2,490 bytes, SHA256 `C38499DC0F723FF1F7551CAD4C20DA5EA6ED9643A66D480BACC6B80AAA33FE10` (stdout byte-identical; stderr empty).
- J@rv1s: CLEAR TO RUN (two conditions, both met), run ACCEPTED, label accepted as printed.

## F2 CENTICENT FLIP CHECK
By hand from the printed bounds: at C = 3.448c and 3.0882c the label is UNRESOLVED both ways. No bound within 0.005c of 0 or +/-C. No flip. Accepted.

## CALIBRATION (scored locally, NOT committed)
Seven rows scored in place, J@rv1s ruled, accepted. Only these lines changed; line counts unchanged; CRLF, no BOM, 0 stray tags; backups in `C:\KalshiBot\tmp\*_pre_20261117.csv`.
- jarvls_calibration.csv lines 63 (55%, FALSE, 0.3025), 73 (15%, FALSE, 0.0225), 77 (6%, FALSE, 0.0036), 78 (80%, TRUE, 0.0400), 93 (93%, TRUE, 0.0049), 99 (85%, TRUE, 0.0225, with J@rv1s's verbatim "invoked" sentence appended). Mean Brier of the six 0.0660 = one label outcome plus two process facts, one very bad confidence row (line 63).
- archie_calibration.csv line 15 (95%, TRUE, 0.0025); column 7 left at 2026-11-30 as logged, the Nov 17 date is in the note.
- Hashes: jarvls `B32452CE...384D` -> `8C65B5A646340D5E349CC3233FB509CDE363841BBC4C59C712618BA38CE2A9E9` (46,380 bytes); archie `575BDBFF...1CD6` -> `E4645B1FC8697E47D747900A7A0003631F93BB27793FF7A8354F8F6D22A7AF5C` (5,760 bytes). Line 80 (Judge-line, 55%) stays PENDING to Nov 19.

## NCEI CROSS-CHECK (descriptive only; cannot change the label)
- Fetched once: 17:53:12-17:53:13 CST, HTTP 200, 672 daily TMAX rows 2025-01-01 to 2026-11-03, no gaps. File `ncei_tmax_USW00023174_20261117.csv`, SHA256 `66B1B631EB9C51A08DD8BDFE4CF6CE5266BFCFC16ABA003BECEFBEB03FD4A7FF`.
- Comparison ran under a ruled read-only exception: `outcomes_sealed.csv` hash matched `SHA256SUMS.txt` (`03E36E51...17A4`) before any content was read; `markets_raw_sealed.json` untouched. Script `ncei_crosscheck_20261117.py` SHA256 `B6D7AD00FAED106EC4F1425A246005CCA267D0EB702B84FEB74AC46F697F324D`; output SHA256 `7EF024B51C4108542F926ACEE1080CD108E262B22ACAF277F53EC6C2B198524A`.
- Result: 1 disagreement in 2,672 between-bracket markets, 1 date in 668: **2026-10-22, KXHIGHLAX-26OCT22-B70.5**, Kalshi yes, NCEI TMAX 72, bracket 70-71. A difference between two sources, not a settlement error. Limits: the ruled convention covers between brackets only (1,336 one-open-end markets listed by name in the output file); an unruled literal-text pass found the same date; whether that date is in the primary 667 is not known.
- **g with and without that date: NOT DONE, deferred pending J@rv1s's ruling (above).** No g, interval or reliability row was computed on sealed data tonight.

## STATE AT CLOSE
- KalshiBot laptop repo not queried tonight (no git ruled). Last known: branch `mlb-backfill-fix-20261102`. Dirty: both calibration CSVs (scored rows) and `data/seeds_bridge_state.json` (runtime state; NEVER commit or modify).
- Dashboard repo fast-forwarded to `6318aaa` (J@rv1s's Nov 17 briefing copy), then this file committed.
- Phase 2 hashed files untouched. Nothing scheduled.

## CARRIED / NOT DONE
- Commit calibration Wednesday (above). Optional Wednesday read-only Oracle log check (Rus runs).
- Alert patch scoping (J@rv1s ruling, earliest Wed after the 10:15 run); `_safe_text` DATA-block fix is its own later commit. Unverified edges: any Oracle reader of the state file; Oracle's full crontab; `data_freshness.py` line 364 discard-list risk.
- Disclosed limits: store copy of the FINAL spec compared by timestamp and reading only (no byte hash); Rus said "keep the laptop awake" and "I am at the laptop", never "untouched" (accepted by J@rv1s).
- Nov 19 Judge-line review is J@rv1s's. Dec 1 daily-rain count re-check only. Jan 14, 2027 Phase 2 verdict; Jan 31 falsification-rule checkpoint.

Archie
