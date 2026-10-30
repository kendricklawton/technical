# Practice Exam 2 — Multi-Pivot Hybrid AD Forest & Hardened Standalone Simulation

> Harder than Exam 1: a two-subnet AD set requiring a double pivot, and standalones with red herrings that punish shallow enumeration. Attempt under exam conditions, then work the 20 scenario questions.

---

## Target Architecture Blueprint

### Active Directory Set — 40 points (`hybrid.corp`)

```text
   [ JUMP01 ]──►[ APP02 ]──────────────►[ SQL03 ]──►[ DC02 ]
   DMZ foothold  pivot A: 10.10.20.0/24   pivot B:     delegation/
   (web)         (constrained deleg.)     10.10.30.0   RBCD → DCSync
```

- **JUMP01 — DMZ foothold.** Web app with an authentication bypass or SSTI to a shell. One internal NIC. Skills: Domain 1, Domain 5 pivoting.
- **APP02 — first internal hop (`10.10.20.0/24`).** Reached via pivot A. Holds a constrained-delegation account (`msDS-AllowedToDelegateTo`). A second NIC reaches `10.10.30.0/24`. Skills: Domain 4 delegation, Domain 5 double pivot.
- **SQL03 — second internal hop (`10.10.30.0/24`).** Reached via pivot B (double pivot). MSSQL with `xp_dirtree` NTLM capture and an RBCD-writable computer object. Skills: Domain 1 MSSQL, Domain 4 RBCD.
- **DC02 — domain controller.** Reached via delegation/RBCD impersonation leading to DCSync. Skills: Domain 4.

### Standalone Machines — 20 points each

- **STANDALONE-D (Windows, DLL hijack).** Foothold via weak creds; Procmon-discovered DLL hijack of a SYSTEM service. Decoy: an unexploitable unquoted path. Skills: Domain 3.
- **STANDALONE-E (Linux, capabilities).** Foothold via a vulnerable web upload; `getcap` reveals `cap_setuid` on a copied interpreter. Decoy: a non-writable cron. Skills: Domain 1 + Domain 2.
- **STANDALONE-F (Windows, AlwaysInstallElevated).** Foothold via an adapted public exploit; MSI elevation to SYSTEM. Decoy: a dead-end service with a strong ACL. Skills: Domain 5 + Domain 3.

---

## Intended Point Flow (one viable path)

1. JUMP01 foothold (auth bypass/SSTI), pivot A into `10.10.20.0/24`.
2. APP02: abuse constrained delegation (`getST -impersonate`) and double-pivot into `10.10.30.0/24`.
3. SQL03: capture/relay or RBCD to impersonate Administrator; reach DC02; DCSync.
4. Standalones D/E/F each: `local.txt` + `proof.txt`, ignoring the planted decoys.

**Pass line:** 70. Note the decoys are deliberate — distinguishing exploitable from non-exploitable findings is the skill under test.

---

## 20 Exam-Style Scenario Questions

**Q1.** JUMP01's login form lets `admin'--` log you in as admin. What class of flaw is this, and the immediate follow-up?

- A. SQL injection in the auth query; pivot to UNION/stacked extraction or a file-write primitive for a shell.
- B. Cross-site scripting; steal the admin cookie from another user.
- C. Broken session fixation; replay a known session ID.
- D. A timing side channel; measure response deltas to extract the password.

<details><summary>Show Answer & Explanation</summary>

**A.** `admin'--` commenting out the password check is classic auth-bypass SQLi; escalate to data extraction or a webshell write. It is server-side (not XSS, B), not session fixation (C), and not a timing channel (D).
</details>

---

**Q2.** To reach APP02 you set up pivot A. Later you must reach `10.10.30.0/24` which only APP02 can see. What do you build?

- A. A second (chained) pivot through APP02, adding a route for 10.10.30.0/24.
- B. A direct Kali route to 10.10.30.0/24 over the first pivot only.
- C. A UDP tunnel, since second-hop subnets require connectionless transport.
- D. Nothing; the first pivot already routes all internal subnets.

<details><summary>Show Answer & Explanation</summary>

**A.** A double pivot chains a second agent/forward through APP02 and routes the deeper subnet. The first pivot cannot reach a subnet only APP02 sees (B, D), and UDP (C) is irrelevant to the topology.
</details>

---

**Q3.** APP02 holds an account with `msDS-AllowedToDelegateTo` set to a CIFS SPN on DC02. How do you abuse it?

- A. `getST -spn cifs/DC02 -impersonate Administrator` to obtain a ticket as Administrator to that service.
- B. Kerberoast the account and crack its password to log in to DC02.
- C. AS-REP roast the account since delegation disables pre-auth.
- D. DCSync directly, since delegation grants replication rights.

<details><summary>Show Answer & Explanation</summary>

**A.** Constrained delegation lets the account request service tickets as any user to the allowed SPN; `getST -impersonate Administrator` yields admin access to CIFS on DC02. Roasting (B, C) and DCSync (D) are unrelated primitives here.
</details>

---

**Q4.** On SQL03 you have a low-priv MSSQL login. `xp_cmdshell` is disabled and cannot be re-enabled. You want a crackable credential. Best move?

- A. `xp_dirtree \\KALI\share` to coerce the service account's NetNTLMv2 hash to responder.
- B. `INTO OUTFILE` a webshell into the MSSQL data directory.
- C. `COPY ... FROM PROGRAM` to run a command through MSSQL.
- D. `LOAD_FILE` to read the SAM over the SQL connection.

<details><summary>Show Answer & Explanation</summary>

**A.** `xp_dirtree` forces the service account to authenticate to your SMB listener (NetNTLMv2 capture), independent of `xp_cmdshell`. `INTO OUTFILE`/`LOAD_FILE` are MySQL (B, D); `COPY ... PROGRAM` is PostgreSQL (C).
</details>

---

**Q5.** STANDALONE-D shows an unquoted service path, but every intermediate directory is non-writable. Procmon also shows the service failing to load `winhelp.dll` from its own writable folder. Correct conclusion?

- A. The unquoted path is a decoy; pursue the writable-directory DLL hijack instead.
- B. Force the unquoted path by making `C:\` writable with icacls.
- C. Both are dead ends; move to another box.
- D. The DLL miss is irrelevant; only unquoted paths escalate on Windows.

<details><summary>Show Answer & Explanation</summary>

**A.** A non-writable unquoted path is unexploitable (a planted decoy); the `NAME NOT FOUND` DLL in a writable folder is the real vector. You cannot arbitrarily make `C:\` writable (B); it is not a dead end (C); DLL hijacking is a valid escalation (D is false).
</details>

---

**Q6.** STANDALONE-E: `getcap -r / 2>/dev/null` shows `/opt/py3 = cap_setuid+ep`, and there is a non-writable root cron. Escalation?

- A. Use `/opt/py3` to call `os.setuid(0)` and spawn a root shell; ignore the cron decoy.
- B. Wait for the cron and inject into it despite the permissions.
- C. Combine the capability and cron, since both are required together.
- D. Report no escalation path exists on this host.

<details><summary>Show Answer & Explanation</summary>

**A.** `cap_setuid` on an interpreter is a direct root path; the non-writable cron is the decoy. You cannot inject into a non-writable cron (B); the two are unrelated (C); a path clearly exists (D).
</details>

---

**Q7.** You double-pivoted successfully but `proxychains nmap -sS` through it returns nothing. Why?

- A. SYN scans use raw sockets that SOCKS cannot proxy; use `-sT -Pn`.
- B. The second pivot dropped; restart both agents.
- C. The target subnet blocks all TCP; switch to UDP.
- D. nmap requires root through proxychains; re-run with sudo.

<details><summary>Show Answer & Explanation</summary>

**A.** proxychains only carries full TCP connects, so `-sS` fails silently — use `-sT -Pn`. The pivot is likely fine (B); nothing indicates a TCP block (C); sudo does not fix the SOCKS/raw-socket limitation (D).
</details>

---

**Q8.** On SQL03 you can write `msDS-AllowedToActOnBehalfOfOtherIdentity` on a computer object. Which sequence achieves impersonation?

- A. addcomputer to control a machine account, rbcd.py to set the attribute, then getST -impersonate Administrator.
- B. Kerberoast the computer account, crack it, then log in as Administrator.
- C. Set an SPN on the computer and AS-REP roast it for the admin hash.
- D. DCSync the computer object to replicate the admin credentials.

<details><summary>Show Answer & Explanation</summary>

**A.** RBCD: own a machine account, set the delegation attribute on the target, then `getST -impersonate`. Machine-account passwords are uncrackable (B); roasting (C) does not apply; DCSync (D) needs replication rights.
</details>

---

**Q9.** It is hour 5 and the AD set is slow going. Standalones are untouched. Strategically, what is often wiser?

- A. Bank points on at least one standalone to guarantee a baseline before the AD set consumes the day.
- B. Commit fully to the AD set and ignore standalones until it is done.
- C. Work all six machines one hour at a time in rotation.
- D. Report the AD set as too hard and request different machines.

<details><summary>Show Answer & Explanation</summary>

**A.** Banking a standalone (20) early de-risks the exam and relieves pressure. All-in on AD (B) risks zero if it stalls; strict rotation (C) prevents deep progress; machines are fixed (D).
</details>

---

**Q10.** STANDALONE-F: you adapted a public exploit and got a user shell. `reg query` shows both AlwaysInstallElevated keys set to 1. Escalation?

- A. Build an MSI with msfvenom and install it with `msiexec /quiet /qn /i` for SYSTEM.
- B. Build an EXE and run it through msiexec to elevate.
- C. Only the HKLM key matters, so install regardless of HKCU.
- D. The keys are a decoy; AlwaysInstallElevated was removed years ago.

<details><summary>Show Answer & Explanation</summary>

**A.** Both keys at 1 means any MSI installs as SYSTEM. It must be an MSI, not an EXE (B); both keys are required (C); the feature still exists (D).
</details>

---

**Q11.** Your captured NetNTLMv2 from SQL03 will not crack against rockyou. Alternative use (assuming a relay target exists and signing is off)?

- A. Relay the authentication with ntlmrelayx to a host without SMB signing.
- B. Convert the NetNTLMv2 to an NT hash for Pass-the-Hash.
- C. Replay it directly as a Kerberos ticket.
- D. Use it as the krbtgt key for a Golden Ticket.

<details><summary>Show Answer & Explanation</summary>

**A.** An uncrackable NetNTLMv2 can still be *relayed* to a signing-disabled target. NetNTLMv2 cannot be converted to an NT hash (B) or replayed as Kerberos (C); it is not the krbtgt key (D).
</details>

---

**Q12.** You find credentials in STANDALONE-E's `ConsoleHost_history.txt`? No — it is Linux. Where is the Linux analogue of command-line credential leakage you should check?

- A. Shell history files (`.bash_history`, `.zsh_history`) and process args via `ps`/`/proc`.
- B. The Windows registry hive copied to the Linux host.
- C. `C:\Users\*\AppData` mounted under `/mnt`.
- D. The DPAPI masterkey store in the user's home.

<details><summary>Show Answer & Explanation</summary>

**A.** On Linux, check shell history and process arguments (`ps aux`, `/proc/*/cmdline`). The registry (B), Windows AppData (C), and DPAPI (D) are Windows constructs.
</details>

---

**Q13.** JUMP01's SSTI returns `49` for `{{7*7}}`. You identify Jinja2. The exam expects a known-good RCE payload. What is the conceptual escalation?

- A. Traverse Python's object graph to reach os.system via subclasses/globals.
- B. Overflow the template buffer to redirect execution.
- C. Inject SQL through the template into the backend database.
- D. Decrypt the Flask secret key to forge an admin session.

<details><summary>Show Answer & Explanation</summary>

**A.** Jinja2 RCE walks `__class__`/`__mro__`/`__subclasses__`/`__globals__` to reach OS functions. It is not a buffer overflow (B), SQL (C), or secret-key decryption (D).
</details>

---

**Q14.** You accidentally crash SQL03 with an unstable action and lose your shell. Best recovery?

- A. Request a revert of only SQL03, then re-establish via your documented steps.
- B. Revert the entire AD set to be safe.
- C. Abandon the AD set, since a crash forfeits its points.
- D. Keep reconnecting to the dead host until it responds.

<details><summary>Show Answer & Explanation</summary>

**A.** Revert the single affected machine and replay your notes. A full-set revert (B) is wasteful; a crash does not forfeit points (C); a dead host will not respond (D).
</details>

---

**Q15.** Across the exam you keep finding "interesting" files that lead nowhere. How should you treat planted decoys?

- A. Verify exploitability quickly (permissions, versions, reachability) and move on if it fails a precondition.
- B. Pursue every finding exhaustively regardless of preconditions.
- C. Assume all unusual files are decoys and ignore them.
- D. Report decoys to the proctor for clarification.

<details><summary>Show Answer & Explanation</summary>

**A.** Fast precondition checks separate real vectors from decoys without sinking hours. Exhaustive pursuit (B) is tunnel vision; ignoring everything (C) misses real vulns; the proctor (D) will not clarify.
</details>

---

**Q16.** You reach APP02 but have no credentials yet — only a foothold shell. Best immediate enumeration for lateral movement?

- A. Loot local creds/config, run BloodHound from the context, and check `whoami /priv` and delegation attributes.
- B. Immediately DCSync the domain from APP02.
- C. Password-spray the whole domain from APP02 at full speed.
- D. Shut down APP02 to force failover to the DC.

<details><summary>Show Answer & Explanation</summary>

**A.** Enumerate the new context (creds, privileges, delegation, BloodHound) to find the next edge. DCSync (B) needs rights you lack; fast spraying (C) locks accounts; shutting down a host (D) is destructive and pointless.
</details>

---

**Q17.** Your single Metasploit exploit use is still available at hour 20. STANDALONE-F already fell manually; the AD set is done; STANDALONE-D's DLL hijack is proving fiddly. Use it where?

- A. On STANDALONE-D if a suitable Metasploit module reliably achieves what your manual attempt cannot.
- B. Re-exploit STANDALONE-F to double-confirm.
- C. On the DC, even though the AD set is complete.
- D. Do not use it; unused Metasploit earns bonus points.

<details><summary>Show Answer & Explanation</summary>

**A.** Spend it on the box still resisting you if a module fits. Re-exploiting a solved box (B) or the finished DC (C) is wasteful; there is no bonus for unused Metasploit (D).
</details>

---

**Q18.** You have 70 points at hour 21 (pass line reached). What is the highest-value use of remaining time?

- A. Verify and screenshot every proof, and shore up notes so the report is airtight.
- B. Attempt a fourth standalone for extra credit beyond 100.
- C. Stop immediately and log off to rest.
- D. Re-exploit solved boxes to collect duplicate proof.

<details><summary>Show Answer & Explanation</summary>

**A.** At the pass line, protect the win: complete, verified proof and notes prevent losing points to documentation gaps. There is no credit beyond the objectives (B); resting is fine but verification comes first (C); duplicates add nothing (D).
</details>

---

**Q19.** STANDALONE-D's DLL hijack needs a malicious DLL exporting a specific function. If the exports are wrong, what happens?

- A. The service may fail to load the DLL or crash; match the expected exports so it loads cleanly.
- B. Windows auto-generates the missing exports at load time.
- C. The DLL runs regardless, since exports are optional.
- D. The service elevates you even on a failed load.

<details><summary>Show Answer & Explanation</summary>

**A.** A hijack DLL must satisfy the expected exports (often via a proxy/forwarder) or the load fails/crashes. Windows does not synthesise exports (B); exports are not optional for the sought symbol (C); a failed load does not elevate (D).
</details>

---

**Q20.** In the report, how should you present the double-pivot AD compromise so it is reproducible?

- A. Include the full tunnel topology, every route/forward command, and each host's chain in order.
- B. Summarise it as "pivoted to the DC and got domain admin" without commands.
- C. Provide only the final DCSync output as evidence.
- D. Describe the techniques generically without host-specific steps.

<details><summary>Show Answer & Explanation</summary>

**A.** A reproducible AD write-up documents the topology, exact forwarding commands, and the ordered per-host chain. Summaries (B), a lone DCSync dump (C), and generic descriptions (D) are not reproducible and lose points.
</details>

---

## Debrief

Did the decoys cost you time? Did the double pivot slow you down? Score against the [README](./README.md) rubric and target your weakest area in Practice Exam 3.
