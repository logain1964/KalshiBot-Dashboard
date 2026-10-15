# SEDE Nightly Session Summary
## For J@rv1s Morning Intelligence Pull

**Covers:** Wed Oct 14, 2026, evening session (laptop clock, about 17:30 CT to about 19:30 CT; Rus fell asleep before close-out, written Thu Oct 15 about 05:35 CT). Oracle clock and commit timestamps are UTC.
**Prepared by:** Archie (Claude Desktop)
**Replaces** the Oct 13 version of `nightly_summary_latest.md` (in git history).

---

## THE SESSION IN ONE PARAGRAPH

**The Phase 2 pre-registration is LOCKED.** Locked document committed at **18:32:39 CT Wed Oct 14** (commit **c22d79b**, private repo), public timestamp line committed by Rus at 18:52:31 CT (commit **42689b4**, dashboard repo). That beat your 20:00 CT target. The locked file is the signed v1.2 sections 1 to 12 unchanged, plus a filled section 13. The second item, the subscriber-email outage, is **partly diagnosed and NOT fixed**: Rus changed the SEDE.signals Gmail password for a reason he has not yet stated, which revoked the stored app password; a new one is still to be created and installed on Oracle (tonight). Nothing changed in any signal, model weight or trading rule.

---

## 1. THE LOCK: FACTS TO CHECK

- **File:** `model_integrity/preregistration_nfl_game_phase2_LOCKED_20261014.md`, 59,637 bytes, SHA-256 `70c11c7b59df7f16f804c5613760e0d65e9ef4cb3e27b2f0261548c2eaf323b9`. The stored git blob hashes identically.
- **Lock commit:** c22d79b, 2026-10-14 18:32:39 CT, pushed to origin/main. It also carries the two one-off count scripts (below). Oracle's 21:00 CT run (ed08fff) sits on top of c22d79b in origin/main's history, so Oracle held the lock commit by then.
- **Public timestamp:** dashboard repo `phase2_lock_timestamp.txt`, commit 42689b4, 18:52:31 CT, content `2026-10-14 18:32:39 CT | Phase 2 pre-registration (NFL_GAME market-relative test, v1.2) locked | SHA-256 70c11c7b...f323b9`. Verified against the raw URL and by git fetch.
- **Feed-status CSV:** `docs/phase2_lock_20261014/feed_status_20261014.csv`, 10,716 bytes, SHA-256 `b6eef7091a0334d2220db16b115d48c55095c357f67fb958ef64a47c64bc2ac4` (laptop copy), 358 runs, guard passed. Committed by Oracle as 90457d3 (18:16:45 CT). Rus pushed that commit from Oracle by hand; the lock record says so.
- **Confirmatory sample at lock time:** 19 events, counted with the frozen script's `select_events`. Of these, 4 have a pre-manifest earliest row (Oct 6 runs) by the ruled sentence, strict count 6 if the one-second write-order artifact is included. Your ruling (4 primary, both numbers) is in the document.

## 2. DISCLOSURES WRITTEN INTO SECTION 13 (read these; they are not flattering)

1. **The paper-trade policy count cannot be produced from the logs.** What I could produce: **8 of 19 events clear the 15c edge on their earliest row** (edges 7.7 to 28.2c). The full four-condition policy (edge, confidence, SEDE rating, monitor running) is not reconstructible. Do not read the 8 as a policy count.
2. **Mechanical diff against the signed text was not possible at writing time.** I ran it afterwards: word-for-word comparison of sections 1 to 12 against the signed v1.2, **0 differences** after stricter normalization (my first pass showed 90 tokenization artifacts, punctuation and list numbers, not content). I did **not** edit the file afterwards, since that would change the hash; the clean result is stated here, not in the locked file. The comparison was word-level because my copy of the signed text was the rendered view without markdown markers, so a byte diff was useless.
3. The one-off scripts `docs/phase2_lock_20261014/premanifest_count_20261014.py` (SHA-256 `fd82af1e...a628a2`) and `edge15_count_20261014.py` (SHA-256 `adc4dc8a...e418d`) are committed with the lock.
4. **The settlement-cache copies live on Oracle's disk only**, not in git.
5. **The outcome-invariant summary counts were not captured** (the command exited 0 but the counts were not recorded).
6. **The target verdict date, Thu Jan 14, 2027, is my derivation**: last regular-season games Sun Jan 10, 2027 (CBS Sports) plus the 3-day STALE rule. It is not a ruled date. Rule on it or replace it.
7. `MIN_EDGE_CENTS` has a **second use at line 1815 of `nfl_model.py` that I did not examine.** It may not matter (the frozen flag rule is at lines 1557 to 1563) but I have not shown that.

## 3. CAVEATS YOU SHOULD KNOW

- **CRLF:** the laptop has `core.autocrlf=true`. A fresh Windows checkout of the lock file converts LF to CRLF and **changes its hash.** Verify committed bytes with `git show c22d79b:model_integrity/preregistration_nfl_game_phase2_LOCKED_20261014.md`, or on Oracle.
- Git warned "LF will be replaced by CRLF" at `git add`; I verified the stored blob is still LF-only.
- **Rus's own date in "Rulings recorded":** not verified by me. The public timestamp is done. Check section 13 if you need that line.
- Your Gap 1 item was recorded per Rus's "confirming 1-->3": option (a), token renewal reminder Sept 13, 2027, ruling date Oct 14. The token's expiry date itself is Rus's report (Oct 11 or Oct 12, 2027 across two messages; his screenshot showed Oct 12, 2027). Either is after the verdict.

## 4. SUBSCRIBER EMAIL (your item 3): PARTLY DIAGNOSED, NOT FIXED

**Correction to the premise first.** Your briefing treated the Sep 21 email as proof that sends worked. The logs do not support that. **Oracle has never logged a successful subscriber send.** The Sep 21 email was probably the internal digest (a different sender path); that still needs confirming from the email's From address and subject. I earlier accepted your inference too quickly. Corrected framing: *Oracle never logged a successful subscriber send; whether Oct 1 to 14 messages were delivered is unknown.* Also, I briefly said the channel was "dead since at least Jun 23"; that overstated what the logs show.

What the evidence says:
- **Code path** (`email_alerts.send_subscriber_email`): SMTP_SSL to smtp.gmail.com:465 as SEDE.signals@gmail.com with `SEDE_EMAIL_APP_PASSWORD` from `.env`. It catches every exception and prints one line. **It does not write to `email_send_log.csv`** (the internal `send_email` does), and the run summary's "Email: [OK] Sent" line reflects the internal digest only, so subscriber failures are invisible in the summary. That is a design gap, independent of the password.
- **Error history on Oracle:** the sender was created Jun 22; failures began Jun 23. Failures were **535 login rejected** until Sep 30, then switched to **"Connection unexpectedly closed"** from Sep 30. The switch is unexplained. Telegram worked throughout. 101 failures logged since Jun 23. `.env` on Oracle last modified 2026-09-22 about 23:20 CT (laptop: Sep 22 23:17); all expected variable names present. Oracle's `email_send_log.csv`: 198 records, all OK (internal digest).
- **Open causes:** a bad or never-valid app password on Oracle; the Sep 30 change; a QUIT-time disconnect (a clean message accepted, then an exception on `with` exit, which the code logs as "failed"; my probe did not support this on a bare connection but did not rule it out).
- **New tonight:** Rus changed the SEDE.signals Google account password (about 19:15 CT; reason not yet stated, "I had to change"). Per Google's help page, a password change **revokes app passwords**. So whatever is on Oracle now is dead for certain. Steps given: sign in as SEDE.signals, https://myaccount.google.com/apppasswords (needs 2-Step Verification), create a new app password, install it with the `read -rs` / `sed -i` command (backs up `.env`, strips spaces, prints a count of 1), then a test send **only if** `SUBSCRIBER_EMAIL` is Rus's own address. The laptop `.env` still holds the old password (laptop pipeline tasks are disabled, so it only matters if it ever sends).

**Still to get from Rus (tonight):** the SEDE.signals Sent folder for Oct 1 to 14 (the decisive check on whether anything was delivered); the From address and subject of the Sep 21 email; why the password had to change (a forced change could mean a security event, and that matters); whether Recent security activity shows a blocked sign-in from Oracle (163.192.200.127); the printed result of the test send and whether it arrived.

**Proposed fix, awaiting Rus's go:** in `email_alerts.py` (outside the frozen NFL_GAME set; the change would show up in `pipeline_commit`): log every subscriber send to `email_send_log.csv`; count a QUIT-time disconnect as sent if the message was accepted; surface subscriber failures in the run summary.

## 5. OTHER ITEMS

- **brier_dashboard.json suspension-flag gap** (MLB_GAME shows clean stats with no suspension flag): not started; low priority, as you said.
- **Dashboard repo** had moved on while I was out: Oracle auto-updates through 21:00 CT Oct 14, your briefing as `jarvls_daily_briefing_20261014.md`, the timestamp file. Laptop pulled fast-forward this morning (05:35 CT). Private repo: origin/main at ed08fff (Oracle 21:00 CT run).
- **Not run tonight:** the session-opener live-trade health check; the Oct 8 to 13 session-archive catch-up (an archive gap: last archive is Oct 7).

## 6. DATES

Thu Oct 15: first normal CLAIMS week under the corrected holiday table; **Seattle at Denver is the first game after the lock (counts, lock CT date was Oct 14).** Oct 15 to 17: **CPI source read must happen before Oct 17**, when the CPI staleness clock expires (needs Rus's go-ahead to move any date). Oct 14 to 19: NHL/NBA matcher ship window (log-only, after the lock; regression must show NFL_GAME classification unchanged). Oct 22: first CLAIMS holiday-week skip. Oct 30: GDP print. Nov 1: Phase 2 checkpoint. About Nov 2 to 3: real JOBS survey; MLB_GAME audit week of Nov 2. Jan 14, 2027: derived verdict date (needs your ruling).

## 7. OPEN POSITIONS AND VALIDATION (carried, not re-pulled)

Unchanged from the Oct 13 summary: no live or paper trades placed or monitored by me; every model that matters is suspended or at entry weight 0. NFL_GAME: no demonstrated edge, unresolved, entry weight 0 until the pre-registered test demonstrates. Last verified Trade #25 KXGDP-26OCT30-T3.5 YES (Oct 12). Gate 1 suspended project-wide.

## 8. PENDING (owner Archie unless stated)

- Finish the sede.signals fix tonight (Rus's inputs above), then the code change if he says go.
- CPI source read before Oct 17; cache-bypass script and Elo determinism test before the verdict; post-lock list from Oct 13 (aftermath log column, fee-basis note, sync denylist gate, D4 join, tickerless audit, decision-impact audit, matcher).
- Rus: finish the app password; send the four email facts above; Sept 13, 2027 token reminder; still unanswered since Oct 12: what "looks Russian" referred to (my only guess is garbled emoji in old source comments).
- Standing: laptop `data/seeds_bridge_state.json` has an uncommitted auto-written change (left alone); four laptop stashes (do not drop); `tmp/` drafts untracked on purpose (including the `v12_signed.md` working copy and the email-probe scripts).

---

*Session | Identity: Archie | Claude Desktop*
*Papa Ralph standard. If it's worth doing it's worth doing right.*
