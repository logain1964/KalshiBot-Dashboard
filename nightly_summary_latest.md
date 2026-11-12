# Nightly Summary — Archie — Thu Nov 12, 2026 (session closed ~17:55 CST)

Session: evening, opened 17:41 CST from J@rv1s's Nov 12 EOD briefing. Nothing in SEDE code, Oracle, or Phase 2 hashed files was touched. No Oracle commands were run.

## Work order (J@rv1s Nov 12) — all done, all saved to the project
1. Result-field probe, KXHIGHCHI (not KXHIGHLAX): 100 of 100 sampled /historical markets carry `result`; distinct strings 'no','yes'; 0 non-yes/no. No stop condition. Mapping and P1 unchanged.
   (result_string_probe_20261112.py/.out, .txt copies in project.)
2. Correction: the Nov 11 "42-market overlap compare" wording was inaccurate. 42 is only the overlap count; the only result-field compare was 24 tickers, 0 mismatches (Nov 4 probe4). Disclosed in the Nov 12 note.
3. Calibration CSV: the "FINAL weather spec locked by Wed Nov 11" row scored in place FALSE, Brier 0.2025 (J@rv1s ruling). 2 rows appended (probe 92% PENDING; FINAL by Sun Nov 15 70% PENDING). Diff verified: only one original line changed; 96 parsed rows; CRLF only. Local, UNCOMMITTED. Backups in C:\KalshiBot\tmp.
4. Centicent vs round1 (outcome-blind): round1 3.4475c, fee_cent 3.4475c, centicent 3.0882c, unrounded 3.0836c. Centicent moves the MISCALIBRATED threshold from about 6.56c to about 6.2c. Primary stays round1 per ruling; centicent is a sensitivity row.
5. Live close time (metadata only): live KXHIGHLAX opens 07:00 PT on D-1 and closes 01:00 PT on D+1 (DST-aware). Forward cell-to-close is 18.00h vs historical median 16.98h. The first run lacked the open-day offset; script edited and rerun, same data, disclosed.
6. Proposed FINAL spec section 7.5/7.6/7.7 text and section 11 hash table drafted; pre-existing recorded hashes matched.

## Fee schedule (Rus attached the PDF, SHA256 c326a69f...9601, effective July 7, 2026)
- Taker formula text: round up so that fee + position cost is rounded to a centicent; M default 1.
- KXHIGHLAX is not in the non-standard multiplier table, so M=1 by default.
- Tension: the per-contract table shows whole cents for 1 contract ($0.02 at $0.50; raw 1.75c) while the 100-contract column matches centicent rounding ($1.75). Which applies to a 1-contract fill is not settled by this document. Round1 stays primary; centicent stays the sensitivity row.
- Proposed spec citation wording sent to J@rv1s for ruling. PDF not yet saved to the project.

## Open items for J@rv1s
- Rule on whether the Nov 4 probe3 pre-seal read of KXHIGHLAX result strings counts against "blind-clean", and on the section 8 wording.
- Rule on the fee citation wording above.

## Lock path (hard date Mon Nov 16; working deadline Sun Nov 15)
- FINAL save date NOT chosen (suggested Sat Nov 14). It is the seed.
- Still to do on lock day: recompute cell outcome-blind; 1,000-run B=5000 rehearsal; confirm prelude (D+0 07:00 PT, winners 1, n 667); repeat sealed-folder metadata check; build_blind_panel.py hash; fill section 11 blanks; SHA256SUMS; save FINAL.
- If FINAL is not saved by Fri Nov 13 evening, tell J@rv1s.
- Stage 1 script: stage1_analysis_DRAFT_20261111_P1.py (SHA F56B788E...1755), cleared by J@rv1s, not yet hashed as final. Self-test 46/46.
- Sealed folder history_panel_20261104\sealed: metadata only checked; no file opened. Folder LastAccessTime is Nov 12 17:46 because I listed it.

## Pending git / ops (Rus runs Oracle commands)
- Calibration commit and push once, after the Fri Nov 13 10:15 CT run, pulling first (private repo branch was 2 ahead, 12 behind origin/main). Do not disturb uncommitted archie_calibration.csv and seeds_bridge_state.json. Keep the 5 non-SEDE income rows out of this file.
- Alert patch: Fri Nov 13 or later only, after the lock is done, never on lock day. The DATA-block _safe_text fix is its own later commit. No Phase 2 hashed file edits.
- Gate 1 status unchanged. Nothing demonstrated. Phase 2 verdict date Jan 14, 2027.
- Slot reservation (KXHIGHLAX) lapses Nov 16 if no spec is saved.
- Judge-line review Nov 19 belongs to J@rv1s.

## Tooling notes
- Project store normalizes CRLF to LF, so store hashes of CRLF files differ from laptop hashes (documented).
- project_write local_path needs a full Read of the file first.
