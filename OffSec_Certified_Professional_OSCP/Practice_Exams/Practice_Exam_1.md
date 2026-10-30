# Practice Exam 1 — Complete 24-Hour Enterprise AD + 3 Standalone Simulation

> A full-scope mock. Build/source equivalents (Proving Grounds, HTB, GOAD) and attempt under exam conditions, then use the 20 scenario questions to pressure-test your decision-making.

---

## Target Architecture Blueprint

### Active Directory Set — 40 points (`corp.local`)

```text
   [ WEB01  (foothold) ]  ──►  [ FILE01 (lateral) ]  ──►  [ DC01 (domain controller) ]
     web app + service          creds reuse +              Kerberos/ACL/DCSync
     account foothold           internal services          domain dominance
```

- **WEB01 — foothold (dual-homed).** Public web app with a manual-SQLi or file-upload path to a service-account shell. Second NIC on the internal `10.10.10.0/24`. Skills: Domain 1 web exploitation, Domain 5 pivoting, Domain 3 service-account to SYSTEM.
- **FILE01 — lateral target.** Reachable only after pivoting. Credentials recovered on WEB01 reuse here; an internal share leaks a Kerberoastable SPN. Skills: Domain 4 Kerberoasting, Domain 4 lateral movement.
- **DC01 — domain controller.** Final objective. An ACL edge or delegation path from the cracked service account leads to DCSync. Skills: Domain 4 ACL abuse / delegation / DCSync.

### Standalone Machines — 20 points each

- **STANDALONE-A (Linux, web to sudo).** LFI/log-poisoning foothold; `sudo -l` GTFOBins escalation. Skills: Domain 1 + Domain 2.
- **STANDALONE-B (Windows, service misconfig).** Public-exploit or weak-cred foothold; unquoted service path or weak service ACL to SYSTEM. Skills: Domain 5 exploit adaptation + Domain 3.
- **STANDALONE-C (Linux, cron/container).** Credential-reuse foothold; `pspy`-discovered cron or Docker-socket escape. Skills: Domain 2.

---

## Intended Point Flow (one viable path, for self-marking)

1. WEB01 foothold via web vuln, stabilise, loot creds (partial AD-foothold credit).
2. Pivot (Ligolo) into `10.10.10.0/24`; reach FILE01; Kerberoast + crack to move laterally.
3. From FILE01, abuse an ACL/delegation edge, DCSync DC01 for the full 40.
4. Each standalone: foothold (`local.txt`, 10) + privesc (`proof.txt`, 10).

**Pass line:** any 70 points. Example: full AD set (40) + 1.5 standalones (30) = 70.

---

## 20 Exam-Style Scenario Questions

**Q1.** You land a shell on WEB01 as `iis apppool\web`. `whoami /priv` shows `SeImpersonatePrivilege`. Fastest route to SYSTEM?

- A. Use a Potato-family tool to impersonate a coerced SYSTEM token.
- B. Run a kernel exploit matched to the build number.
- C. Dump LSASS for cached domain credentials.
- D. Brute-force the local Administrator password.

<details><summary>Show Answer & Explanation</summary>

**A.** `SeImpersonate` on a service account is the textbook Potato path to SYSTEM — fast and reliable. A kernel exploit (B) is riskier; LSASS (C) needs SYSTEM first; brute force (D) is slow and noisy.
</details>

---

**Q2.** WEB01 has a second interface on `10.10.10.0/24` that Kali cannot reach. You want all tools to work against internal hosts transparently. Best choice?

- A. Establish a Ligolo-ng TUN pivot and add a Kali route for the subnet.
- B. Open an SSH local forward for each internal port as you need it.
- C. Copy every tool to WEB01 and run them locally there.
- D. Ask the proctor to add a route to the internal subnet.

<details><summary>Show Answer & Explanation</summary>

**A.** A TUN pivot makes the subnet routable so tools run natively. Per-port forwards (B) are tedious; running everything on WEB01 (C) is limited; the proctor (D) does not alter the exam network.
</details>

---

**Q3.** On FILE01 you crack a Kerberoasted hash for `svc_sql`. BloodHound shows `svc_sql` has `GenericAll` over a user in "Server Admins". Sound next move?

- A. Use the ACL edge to take over that user, then follow the path toward the DC.
- B. Immediately attempt a Golden Ticket with the svc_sql hash.
- C. Re-Kerberoast using svc_sql to collect more tickets.
- D. Password-spray svc_sql's password across the whole domain at once.

<details><summary>Show Answer & Explanation</summary>

**A.** `GenericAll` over a privileged-group user is a concrete takeover edge; follow BloodHound's path. A Golden Ticket (B) needs the `krbtgt` hash, not a service hash; re-roasting (C) adds nothing; a full-domain spray (D) risks lockouts.
</details>

---

**Q4.** STANDALONE-A shows an LFI. You read `/etc/passwd` but need code execution. Most likely to work first?

- A. Poison a readable web or SSH log and include it.
- B. Switch to a time-based blind SQLi extractor.
- C. Upload a webshell via an unrestricted upload form.
- D. Request `php://filter` to run commands directly.

<details><summary>Show Answer & Explanation</summary>

**A.** LFI plus a writable/readable log equals log poisoning to RCE. SQLi (B) is a different vuln; no upload is mentioned (C); `php://filter` (D) discloses source, it does not execute commands.
</details>

---

**Q5.** You have spent three hours on STANDALONE-B's web service with no foothold. The box also exposes SMB (445) and a nonstandard TCP port. Best decision?

- A. Stop tunnel-visioning the web app and enumerate SMB and the other port thoroughly.
- B. Keep attacking the web app, since persistence always pays off.
- C. Move permanently to another box and abandon STANDALONE-B.
- D. Run an automated vulnerability scanner against the web app.

<details><summary>Show Answer & Explanation</summary>

**A.** Re-enumerate under-explored surfaces; the foothold is usually something walked past. Persisting blindly (B) is tunnel vision; abandoning entirely (C) is premature; automated scanners (D) are banned.
</details>

---

**Q6.** STANDALONE-C: you have a user shell and `pspy64` shows root running `/usr/bin/python3 /opt/jobs/collect.py` every minute, with `collect.py` world-writable. Escalation?

- A. Append a root-shell payload to `collect.py` and wait for the next run.
- B. Replace `/usr/bin/python3` with a shell script.
- C. Hijack `$PATH` so `python3` resolves to your binary.
- D. Edit `/etc/crontab` to add your own root job.

<details><summary>Show Answer & Explanation</summary>

**A.** A writable script run by root cron is a direct escalation. You cannot overwrite `/usr/bin/python3` (B); the cron uses an absolute interpreter path so PATH hijack (C) fails; `/etc/crontab` is typically not user-writable (D).
</details>

---

**Q7.** You must choose where to spend your single allowed Metasploit `exploit` use. Wisest use?

- A. The one standalone whose public exploit you cannot get working manually.
- B. The Domain Controller, to guarantee the AD set.
- C. The easiest box, to save time early.
- D. Every box, since the limit is only a guideline.

<details><summary>Show Answer & Explanation</summary>

**A.** Reserve the single Metasploit exploit for the box that resists manual exploitation — maximum marginal value. Spending it on the DC (B) or an easy box (C) wastes it; using it everywhere (D) violates the rule and risks disqualification.
</details>

---

**Q8.** You recover `svc_backup:Winter2024!` from SNMP process arguments on a standalone. First thing to try?

- A. Reuse the credential against SSH, SMB, and the web login on that host and others.
- B. Crack the credential offline even though you already have the plaintext.
- C. Change the service account's password to lock out the service.
- D. Ignore it, since SNMP-sourced credentials are usually decoys.

<details><summary>Show Answer & Explanation</summary>

**A.** A plaintext credential's value is in reuse across services/hosts. Cracking a known plaintext (B) is pointless; changing it (C) is destructive; dismissing it (D) discards a likely foothold.
</details>

---

**Q9.** On the AD foothold you can either chase a complex ADCS ESC1 path or a simpler cracked-credential lateral move that BloodHound shows reaches the DC. Time is tight. Choose?

- A. Take the simpler credential path BloodHound already validated to the DC.
- B. Pursue ADCS ESC1 first because certificate attacks are more impressive.
- C. Attempt both simultaneously to double your chances.
- D. Neither; restart enumeration from scratch to be safe.

<details><summary>Show Answer & Explanation</summary>

**A.** Take the shortest validated path to the objective; points, not elegance, pass the exam. ESC1 (B) is longer here; splitting focus (C) wastes time; restarting (D) discards progress.
</details>

---

**Q10.** Your reverse shell on STANDALONE-A keeps dying when you press Ctrl-C. Before escalating, you should?

- A. Upgrade to a full TTY (pty spawn, stty raw -echo, export TERM).
- B. Reconnect repeatedly and avoid pressing Ctrl-C entirely.
- C. Switch to a bind shell, which never dies.
- D. Run all commands as one long semicolon-chained line.

<details><summary>Show Answer & Explanation</summary>

**A.** Stabilise the shell first; a fragile shell wastes time and loses sessions mid-escalation. Avoiding Ctrl-C (B) is a workaround, not a fix; bind shells (C) are not inherently stable and may be firewalled; chaining commands (D) is painful and error-prone.
</details>

---

**Q11.** You get `local.txt` on STANDALONE-B but cannot escalate after two hours. Scoring-wise, what should you ensure right now?

- A. Screenshot `local.txt` with `ipconfig` and `whoami` in frame to bank the 10 points.
- B. Delete your notes for the box to avoid confusion later.
- C. Skip documentation until you also have `proof.txt`.
- D. Submit the hash of `local.txt` instead of a screenshot.

<details><summary>Show Answer & Explanation</summary>

**A.** Capture proof of the foothold immediately to secure the 10 points regardless of escalation. Deleting notes (B) loses reproduction; delaying docs (C) risks forgetting steps; a hash (D) is not the required proof format.
</details>

---

**Q12.** FILE01 rejects NTLM but accepts Kerberos. You hold a local NT hash for a reused account. How do you authenticate?

- A. Overpass-the-Hash: get a TGT with the hash, export KRB5CCNAME, use Kerberos tools.
- B. Pass-the-Hash the NT hash straight to the KDC as a ticket.
- C. Crack the NT hash to plaintext first, since Kerberos needs a password.
- D. Use DCSync to convert the hash into a Kerberos ticket.

<details><summary>Show Answer & Explanation</summary>

**A.** Overpass-the-Hash converts the NT key to a Kerberos TGT for Kerberos-only targets. PtH to the KDC (B) is not a thing; cracking (C) is unnecessary; DCSync (D) needs replication rights and is unrelated.
</details>

---

**Q13.** AutoRecon output for STANDALONE-A lists an Apache version with a known RCE PoC on Exploit-DB. Correct handling?

- A. Mirror the PoC, read it fully, fix hardcoded values, test locally, then run it.
- B. Run the PoC directly against the target with its default settings.
- C. Discard it because Exploit-DB code is never exam-relevant.
- D. Feed the target to sqlmap to confirm the RCE first.

<details><summary>Show Answer & Explanation</summary>

**A.** Adapt-and-test is the professional workflow; default-configured PoCs rarely work and may crash the box (B). Exploit-DB is highly relevant (C); sqlmap (D) is banned and irrelevant to Apache RCE.
</details>

---

**Q14.** You reach a Domain Admin-equivalent on DC01. Which action best proves full domain compromise for the report?

- A. DCSync the `krbtgt` and Administrator hashes with secretsdump.
- B. Change the Administrator password to one you choose.
- C. Create a new Domain Admin account named after yourself.
- D. Reboot the DC to confirm you control it.

<details><summary>Show Answer & Explanation</summary>

**A.** DCSync of `krbtgt`/Administrator is the standard, non-destructive proof of domain dominance. Changing passwords (B) or adding accounts (C) is unnecessarily destructive; rebooting (D) proves nothing and disrupts.
</details>

---

**Q15.** It is hour 22. You have 65 points and are stuck on STANDALONE-C's privesc. Best use of remaining time?

- A. Re-enumerate C methodically for a missed vector while verifying all existing proofs.
- B. Start a brand-new box you have not touched.
- C. Stop working entirely and begin the report now.
- D. Retry the same failed escalation repeatedly.

<details><summary>Show Answer & Explanation</summary>

**A.** You need 5 more points; a missed vector on C (worth 10) is the highest-probability source, and verifying proofs protects what you have. A new box (B) is unrealistic late; stopping (C) forfeits a reachable pass; repeating a failed method (D) is futile.
</details>

---

**Q16.** STANDALONE-A `sudo -l` shows `(root) NOPASSWD: /usr/bin/tar`. Escalation?

- A. Use tar's checkpoint-action to execute a command as root (GTFOBins).
- B. Use tar to overwrite `/etc/shadow` with a blank root entry only.
- C. Compile an LD_PRELOAD library and run tar with it.
- D. Exploit an unquoted path in the tar binary.

<details><summary>Show Answer & Explanation</summary>

**A.** `sudo tar` with `--checkpoint-action=exec=...` runs a root command (GTFOBins). No `env_keep` is shown for LD_PRELOAD (C); unquoted paths (D) are a Windows-service concept; the shadow approach (B) is not tar's escape.
</details>

---

**Q17.** You discover an internal service on FILE01 bound to `127.0.0.1:8080` only. To reach it from Kali you should?

- A. Forward the port through your pivot (Ligolo listener/route or SSH -L chain).
- B. Rebind the service to 0.0.0.0 by editing its config as a low-priv user.
- C. Exploit it over UDP since localhost services accept UDP.
- D. Conclude it is unreachable and move on.

<details><summary>Show Answer & Explanation</summary>

**A.** Localhost-only services are reached by forwarding through your existing pivot. Rebinding (B) usually needs privileges you lack and is intrusive; UDP (C) is irrelevant; abandoning (D) skips a likely vector.
</details>

---

**Q18.** During the AD set you accidentally lock two accounts while spraying. What went wrong and what do you do?

- A. You exceeded the lockout threshold; stop, wait the observation window, and spray one attempt per account per window.
- B. The DC is faulty; request a revert of the entire AD set.
- C. Nothing is wrong; locked accounts still authenticate for PtH.
- D. Continue spraying faster to outrun the lockout counter.

<details><summary>Show Answer & Explanation</summary>

**A.** Lockouts come from too many attempts per window; slow to one-per-window after reading the policy. A revert (B) is drastic/unnecessary; locked accounts do not help (C); spraying faster (D) worsens it.
</details>

---

**Q19.** Your manual UNION SQLi on WEB01 returns data but you need a shell. The DB is MySQL with FILE privilege and a writable webroot. Next step?

- A. `INTO OUTFILE` a PHP webshell into the webroot and browse to it.
- B. `xp_cmdshell` to run commands through the engine.
- C. `COPY ... FROM PROGRAM` to execute a shell command.
- D. `LOAD_FILE` the kernel image to extract credentials.

<details><summary>Show Answer & Explanation</summary>

**A.** MySQL + FILE + writable webroot = `INTO OUTFILE` webshell to RCE. `xp_cmdshell` (B) is MSSQL; `COPY ... PROGRAM` (C) is PostgreSQL; `LOAD_FILE` on a kernel image (D) yields nothing useful.
</details>

---

**Q20.** Report-writing time. One standalone fell to a chain you only partly documented. Best course?

- A. Reconstruct and re-verify the chain on the box (if still available) so every step is reproducible.
- B. Submit the report with "exploited via web vuln" and no steps.
- C. Omit that box entirely to avoid an incomplete section.
- D. Guess the missing commands from memory and submit without checking.

<details><summary>Show Answer & Explanation</summary>

**A.** Reproducibility is scored; reconstruct and verify while you still can. Vague descriptions (B) and guessed commands (D) risk losing the points; omitting the box (C) discards points you earned.
</details>

---

## Debrief

Score each objective, then write a one-page retro against the rubric in the [README](./README.md): enumeration completeness, time-boxing discipline, reproducibility, proof hygiene, and report-readiness. Carry the weakest area forward into Practice Exam 2.
