# Domain 3 — Quiz 3: Registry Abuse, Credential Harvesting & UAC Bypass

> Scenario-driven, exam-difficulty. Answer before expanding.

---

**Q1.** Both `HKCU` and `HKLM` `AlwaysInstallElevated` values read `0x1`. Which exploitation is correct, and why both keys?

- A. Build an MSI payload and run `msiexec /quiet /qn /i`, because installation elevates only when both keys are 1.
- B. Build an MSI payload and run it as your user, because the HKCU key alone grants elevation.
- C. Build an EXE payload and run `msiexec /i`, because msiexec elevates any executable it launches.
- D. Build an MSI payload and schedule it, because AlwaysInstallElevated applies only to scheduled installs.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** The policy elevates MSI installs only when **both** HKCU and HKLM are set to 1; an `msiexec /quiet /qn /i payload.msi` then runs as SYSTEM.

- **B** is wrong: HKCU alone is insufficient; both keys are required.
- **C** is wrong: the mechanism elevates MSI packages, not arbitrary EXEs passed to `/i`.
- **D** is wrong: it is not limited to scheduled installs; any MSI install elevates when both keys are 1.
</details>

---

**Q2.** `cmdkey /list` shows a stored credential for `Administrator`. You do not know the password. How do you use it?

- A. Launch your payload with `runas /savecred /user:Administrator`, since the saved credential authenticates for you.
- B. Read the plaintext password from the Credential Manager vault file directly with `type`.
- C. Pass the stored credential to `secretsdump` to extract the NT hash for Pass-the-Hash.
- D. Export the credential with `cmdkey /export` and crack the resulting hash offline.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** `runas /savecred /user:Administrator "payload"` uses the stored credential without you knowing the password — the whole point of a saved credential.

- **B** is wrong: the vault is DPAPI-encrypted; `type` yields ciphertext, not plaintext.
- **C** is wrong: `secretsdump` does not consume `cmdkey` entries this way; the usable primitive is `runas /savecred`.
- **D** is wrong: there is no `cmdkey /export` that yields a crackable hash.
</details>

---

**Q3.** You recover `Groups.xml` from SYSVOL containing a `cpassword` attribute. Why is this immediately exploitable?

- A. The value is AES-encrypted with a key Microsoft published, so `gpp-decrypt` recovers the plaintext.
- B. The value is a reversible base64 string that decodes directly to the password.
- C. The value is an NT hash usable as-is for Pass-the-Hash without cracking.
- D. The value is a plaintext password obfuscated only by XML entity encoding.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** GPP `cpassword` uses AES with a key Microsoft documented (MS14-025). `gpp-decrypt` reverses it to plaintext; any domain user who could read SYSVOL could do this.

- **B** is wrong: it is AES-encrypted, not mere base64 (though base64 wraps the ciphertext).
- **C** is wrong: it decrypts to a plaintext password, not an NT hash.
- **D** is wrong: it is encrypted, not just entity-encoded.
</details>

---

**Q4.** Where do you look first for credentials that an administrator typed into PowerShell, and what is the caveat?

- A. `ConsoleHost_history.txt` in the user's PSReadLine AppData path, which records command lines but only for that user profile.
- B. The Windows Event Log Security channel, which stores all typed passwords in event 4624.
- C. The registry key `HKCU\Console\History`, which caches the last 50 commands in plaintext.
- D. The pagefile, which always contains a full plaintext transcript of every shell session.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** PSReadLine writes `ConsoleHost_history.txt` under `AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine`; it captures command lines (often secrets) but is per-user, so you need that user's profile access.

- **B** is wrong: 4624 is a logon event and does not store typed passwords.
- **C** is wrong: there is no such plaintext command-history registry key.
- **D** is wrong: the pagefile may contain fragments but is not a reliable full transcript, and "always" is false.
</details>

---

**Q5.** You are a local administrator but running at medium integrity, and you need a high-integrity shell to dump hashes. Which technique fits and why does it avoid a prompt?

- A. A `fodhelper.exe` registry hijack, because the auto-elevating binary reads a user-writable key you can poison.
- B. A Potato exploit, because `SeImpersonatePrivilege` always elevates medium-integrity admins silently.
- C. A `runas` with your own password, because re-authenticating promotes you to high integrity.
- D. A kernel exploit, because UAC can only be bypassed through memory corruption.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** `fodhelper.exe` auto-elevates and reads a user-writable registry path; planting your command there yields a high-integrity shell with no UAC prompt — a classic UAC bypass for admins.

- **B** is wrong: Potato targets `SeImpersonate` service contexts for SYSTEM, not UAC elevation of an interactive admin; it is not the fitting technique here.
- **C** is wrong: `runas` with your own creds does not cross the UAC boundary to high integrity.
- **D** is wrong: UAC bypasses are typically logic/registry hijacks, not memory corruption.
</details>

---

**Q6.** Why does Microsoft state that UAC is "not a security boundary," and how does that inform your exam expectations?

- A. Auto-elevating binaries can be hijacked, so UAC bypasses are expected techniques rather than vulnerabilities Microsoft will always patch.
- B. UAC is encrypted weakly, so any attacker can decrypt the consent prompt token offline.
- C. UAC only applies to domain accounts, so local administrators are never prompted.
- D. UAC is disabled by default on all server editions, so it never applies on the exam.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** Because numerous auto-elevating paths exist and are not treated as servicing-priority vulnerabilities, UAC is defence-in-depth; bypasses are a normal part of admin-to-high-integrity escalation.

- **B** is wrong: there is no "consent prompt token" to decrypt; this misdescribes UAC.
- **C** is wrong: UAC applies to local admins too (that is its main case).
- **D** is wrong: UAC is present on servers; it is not uniformly disabled.
</details>

---

**Q7.** You find a KeePass `.kdbx` database on a user's desktop. What is the realistic path to using it, and the limitation?

- A. Exfiltrate it and attempt to crack the master password offline, which fails if the password is strong and a keyfile is required.
- B. Open it directly with any text editor, since `.kdbx` stores entries in plaintext XML.
- C. Pass the `.kdbx` to `secretsdump`, which extracts all entries without the master password.
- D. Rename it to `.xml` and import it into your browser's password manager to reveal entries.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** You crack the master password offline (`keepass2john` → hashcat/john). It only works if the master password is weak and you also have any required keyfile — strong master + keyfile defeats it.

- **B** is wrong: `.kdbx` is encrypted, not plaintext XML.
- **C** is wrong: `secretsdump` targets Windows secrets, not KeePass vaults.
- **D** is wrong: renaming does not decrypt it; browsers cannot import an encrypted KeePass DB as plaintext.
</details>

---

**Q8.** For the report, which credential-hygiene remediation set is correct for the findings in this quiz?

- A. Remove GPP cpassword from SYSVOL, delete unattend files post-provisioning, avoid secrets on command lines, and limit local admins.
- B. Rotate the domain functional level and disable NTLM entirely across the forest.
- C. Encrypt the pagefile and disable PowerShell on every host to stop history capture.
- D. Require smart cards for all logons so no password-based credential can ever be stored.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** The findings are stored/leaked secrets: purge `cpassword` (MS14-025), remove provisioning files, discourage command-line secrets (they land in history/SNMP/process lists), and reduce local-admin sprawl.

- **B** is disproportionate and unrelated to most of these leak sites; disabling NTLM has broad consequences and does not remove stored secrets.
- **C** is impractical (disabling PowerShell everywhere) and does not address GPP/unattend/stored creds.
- **D** is a large architectural change that still leaves service accounts/stored secrets; it does not map to these specific findings.
</details>
