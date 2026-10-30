# Domain 3 — Quiz 1: Token Privileges (SeImpersonate, SeBackup)

> Scenario-driven, exam-difficulty. Answer before expanding.

---

**Q1.** Your foothold runs as an IIS application pool identity. `whoami /priv` lists `SeImpersonatePrivilege` as Enabled. Which escalation fits, and what does it exploit?

- A. A Potato-style tool that coerces a SYSTEM service to authenticate locally, then impersonates its token.
- B. A kernel exploit that overwrites the process token directly in kernel memory.
- C. A Pass-the-Hash attack using the application pool's cached NTLM credentials.
- D. A DLL hijack against the IIS worker process to load a SYSTEM-level library.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** `SeImpersonatePrivilege` is exactly what PrintSpoofer/GodPotato need: they trick a SYSTEM service into authenticating to a local endpoint and then impersonate the returned SYSTEM token.

- **B** is wrong: no kernel bug is indicated; the privilege itself is the intended lever, no memory corruption needed.
- **C** is wrong: the apppool identity is a virtual account without a reusable NTLM hash for PtH here.
- **D** is unrelated to the `SeImpersonate` finding and not the direct path the privilege enables.
</details>

---

**Q2.** You hold `SeBackupPrivilege`. Which action most directly leads to credential recovery, and why is it permitted despite ACLs?

- A. Save the SAM and SYSTEM registry hives, since backup privilege bypasses file ACLs, then dump hashes offline.
- B. Read LSASS memory live, since backup privilege grants debug rights to running processes.
- C. Overwrite the Administrator's NTDS database, since backup privilege allows arbitrary writes to AD.
- D. Enable `xp_cmdshell` on the local SQL instance, since backup privilege includes database roles.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** `SeBackupPrivilege` bypasses file ACLs for backup, so `reg save HKLM\SAM` and `HKLM\SYSTEM` succeed; `secretsdump ... LOCAL` then extracts local hashes.

- **B** is wrong: reading LSASS is a *debug* capability (`SeDebugPrivilege`), not backup.
- **C** is wrong: this is a local workstation hive scenario; backup privilege is read-oriented here and NTDS lives on a DC, not addressed by this privilege alone.
- **D** is wrong: a Windows privilege does not grant SQL Server roles.
</details>

---

**Q3.** After `reg save HKLM\SAM sam.hiv` and `reg save HKLM\SYSTEM system.hiv`, which offline command extracts the local hashes, and why is the SYSTEM hive required?

- A. `secretsdump.py -sam sam.hiv -system system.hiv LOCAL`, because SYSTEM holds the boot key that decrypts SAM.
- B. `secretsdump.py -sam sam.hiv LOCAL`, because the SAM hive alone contains the plaintext passwords.
- C. `hashcat sam.hiv`, because the SAM hive is already a crackable hash list once saved.
- D. `john system.hiv`, because the SYSTEM hive stores the NT hashes directly.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** The SAM hashes are encrypted with the **boot key (SYSKEY)** stored in the SYSTEM hive; `secretsdump` needs both to decrypt and output NT hashes.

- **B** is wrong: without the SYSTEM hive's boot key the SAM cannot be decrypted; SAM does not store plaintext.
- **C** is wrong: a raw hive is not a hashcat-ready list; it must be parsed/decrypted first.
- **D** is wrong: the SYSTEM hive holds the boot key, not the NT hashes.
</details>

---

**Q4.** `whoami /priv` shows `SeDebugPrivilege` Enabled on your shell. Which escalation-relevant capability does this most directly grant?

- A. A handle to any process, enabling LSASS dumping or injection into a SYSTEM process.
- B. The ability to reconfigure any Windows service's binary path to your payload.
- C. The ability to install MSI packages with SYSTEM privileges silently.
- D. The ability to bypass file ACLs to copy the SAM and SYSTEM hives.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** `SeDebugPrivilege` opens a handle to any process; you can dump LSASS for credentials or inject into a SYSTEM process to inherit its context.

- **B** describes weak service ACLs, a separate vector.
- **C** describes `AlwaysInstallElevated`, unrelated to this privilege.
- **D** describes `SeBackupPrivilege`, a different privilege.
</details>

---

**Q5.** PrintSpoofer fails on a hardened host, but `whoami /priv` still shows `SeImpersonatePrivilege`. What is the sound next step?

- A. Try an alternative Potato variant (GodPotato/RoguePotato) matched to the OS build and conditions.
- B. Conclude the privilege is a false positive and pivot to kernel exploits immediately.
- C. Disable the privilege with `whoami /priv /disable` and retry PrintSpoofer.
- D. Request domain admin credentials, since local impersonation is clearly patched.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** The Potato family has several implementations tuned to different Windows builds/DCOM states; if one fails, another (GodPotato, RoguePotato, JuicyPotatoNG) often works with the same privilege.

- **B** is wrong: the privilege is real; jumping to risky kernel exploits skips the reliable alternative.
- **C** is wrong: there is no `/disable` escalation step; disabling the privilege removes your lever.
- **D** is wrong: you escalate locally with the privilege you already hold; nothing suggests asking for creds.
</details>

---

**Q6.** Why is the Potato technique best described as *abuse of a legitimate feature* rather than a software bug?

- A. Token impersonation is a designed service capability; the exploit merely feeds the service a SYSTEM token to impersonate.
- B. The technique patches a buffer overflow in the print spooler to redirect execution.
- C. The technique exploits a cryptographic weakness in NTLM to forge a SYSTEM token offline.
- D. The technique relies on a race condition in the kernel scheduler to swap tokens.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** Impersonation is how services act for clients. The exploit coerces a SYSTEM-context authentication to a local listener and impersonates that token — abusing the feature, not corrupting memory.

- **B** is wrong: there is no buffer overflow; spooler/DCOM coercion is a protocol interaction, not memory corruption.
- **C** is wrong: no offline NTLM forgery is involved; a live token is impersonated.
- **D** is wrong: it is not a scheduler race.
</details>

---

**Q7.** You have `SeTakeOwnershipPrivilege`. Conceptually, how does it lead to SYSTEM?

- A. Take ownership of a SYSTEM-run binary or service object, grant yourself write/control, then replace or reconfigure it.
- B. Take ownership of the user's own files to read cached credentials stored there.
- C. Take ownership of the network stack to intercept SYSTEM authentication traffic.
- D. Take ownership of the page file to read SYSTEM process memory from disk.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** Ownership lets you rewrite an object's DACL; take ownership of a file/service that SYSTEM executes, grant yourself write, replace it, and gain SYSTEM on next run.

- **B** understates it: taking ownership of your *own* files gains nothing; the power is seizing *protected* objects.
- **C** is wrong: ownership applies to securable objects, not the network stack.
- **D** is wrong: this is not a page-file memory-read primitive.
</details>

---

**Q8.** For the report, which remediation correctly addresses a `SeImpersonatePrivilege`-based SYSTEM escalation from a service account?

- A. Apply least privilege to service accounts and keep the OS patched so coercion vectors are closed.
- B. Rename the print spooler service so the Potato tools cannot locate it by name.
- C. Block outbound internet from the host so the exploit cannot download its payload.
- D. Set a longer, more complex password on the service account to prevent impersonation.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** Reduce which accounts hold `SeImpersonate`, run services with minimal rights, and patch (recent OS updates closed several local coercion paths). Defence in depth targets the privilege and the coercion.

- **B** is security by obscurity; renaming does not remove the privilege or the technique.
- **C** is wrong: the escalation is entirely local; no internet is required.
- **D** is wrong: impersonation does not depend on the account password's strength.
</details>
