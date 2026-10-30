# Practice Exam 3 — Web-Heavy Initial Access, ADCS Escalation & Container Escape Simulation

> Emphasises modern footholds: web-chained initial access, an ADCS escalation in the AD set, and a container-escape standalone. Attempt under exam conditions, then work the 20 scenario questions.

---

## Target Architecture Blueprint

### Active Directory Set — 40 points (`certco.local`)

```text
   [ PORTAL01 ]──►[ WORK02 ]──────────►[ DC03 + CA ]
   web upload     cred harvest /         ADCS ESC1
   → foothold     AS-REP roast           → cert → DCSync
```

- **PORTAL01 — web foothold.** Unrestricted file upload (magic-byte + extension bypass) to a webshell, then service-account context. Internal NIC to `10.10.40.0/24`. Skills: Domain 1 upload bypass, Domain 5 pivot.
- **WORK02 — lateral target.** Reached via pivot. DPAPI/stored creds and an AS-REP-roastable user. Skills: Domain 3 credential harvesting, Domain 4 AS-REP roasting.
- **DC03 (+ AD CS role) — domain controller.** A vulnerable certificate template (ESC1) lets a low-priv user obtain an Administrator certificate, authenticate via PKINIT, and DCSync. Skills: Domain 4 ADCS + DCSync.

### Standalone Machines — 20 points each

- **STANDALONE-G (Linux, container escape).** Web RCE drops you inside a container; an exposed Docker socket or `--privileged` flag escapes to host root. Skills: Domain 1 + Domain 2 containers.
- **STANDALONE-H (Windows, SeBackupPrivilege).** Weak-cred foothold; `SeBackupPrivilege` to dump SAM/SYSTEM and PtH the local admin. Skills: Domain 3.
- **STANDALONE-I (Linux, SUID reversing).** Foothold via log-poisoning LFI; a custom SUID binary reversed with `ltrace` reveals a PATH-hijackable relative call. Skills: Domain 1 + Domain 2.

---

## Intended Point Flow (one viable path)

1. PORTAL01: bypass upload validation for a webshell, pivot into `10.10.40.0/24`.
2. WORK02: harvest DPAPI/stored creds, AS-REP roast a user, crack, move laterally.
3. DC03: Certipy ESC1 to an Administrator certificate, PKINIT to a TGT/hash, DCSync.
4. Standalones G/H/I each: `local.txt` + `proof.txt`.

**Pass line:** 70.

---

## 20 Exam-Style Scenario Questions

**Q1.** PORTAL01 blocks `.php` and checks magic bytes, but executes files in the upload dir. Which single upload is most likely to yield a webshell?

- A. A `GIF89a;`-prefixed polyglot with a `.phtml` (or handler-mapped) extension.
- B. A pure PHP file renamed `shell.jpg` with no header changes.
- C. A `.php` file with only the Content-Type header forged.
- D. A plain text note instructing the admin to run a command.

<details><summary>Show Answer & Explanation</summary>

**A.** Defeating both the magic-byte check (GIF header) and the extension block (alternate executable extension) in one upload is the path. A `.jpg` body won't execute as PHP (B); `.php` is blocked outright (C); a text note (D) is not code execution.
</details>

---

**Q2.** After the webshell you stabilise and want to pivot. PORTAL01 reaches `10.10.40.0/24`. Preferred transparent pivot?

- A. Ligolo-ng TUN pivot with a Kali route for the subnet.
- B. Manual `socat` relays for each service you discover.
- C. Base64 copy-paste of tool output through the webshell.
- D. No pivot; exploit everything from the webshell directly.

<details><summary>Show Answer & Explanation</summary>

**A.** A TUN pivot gives native reachability for all your tools. Per-service `socat` (B) is tedious; base64 through a webshell (C) is painful; webshell-only (D) is severely limited for AD work.
</details>

---

**Q3.** On WORK02 you find a DPAPI-protected browser credential blob. What do you also need to decrypt it?

- A. The user's DPAPI masterkey (or their password/context) to decrypt the blob.
- B. The domain krbtgt hash, which decrypts all DPAPI blobs.
- C. Nothing; DPAPI blobs are plaintext once copied off-host.
- D. The machine's BitLocker recovery key.

<details><summary>Show Answer & Explanation</summary>

**A.** DPAPI blobs need the user's masterkey (recoverable via Mimikatz in the user context or with the password). The krbtgt hash (B) is unrelated; blobs are encrypted (C); BitLocker (D) is a different mechanism.
</details>

---

**Q4.** WORK02 has an AS-REP-roastable user. Which Hashcat mode, and what precondition made it roastable?

- A. `-m 18200`; the account has Kerberos pre-authentication disabled.
- B. `-m 13100`; the account has an SPN set.
- C. `-m 5600`; the account sent a NetNTLMv2 response.
- D. `-m 1000`; the account's NT hash was exposed.

<details><summary>Show Answer & Explanation</summary>

**A.** AS-REP roasting is `-m 18200`, enabled by pre-auth being disabled. `-m 13100` is Kerberoasting/SPN (B); `-m 5600` is NetNTLMv2 (C); `-m 1000` is raw NTLM (D).
</details>

---

**Q5.** `certipy find` on DC03 flags a template as ESC1. Why is this directly exploitable by your low-priv user?

- A. The template permits an enrollee-supplied SAN with client-auth EKU, so you request a cert as Administrator.
- B. The template leaks the CA private key for offline cert signing.
- C. The template disables revocation so old certs can be reused.
- D. The template requires manager approval you can self-grant.

<details><summary>Show Answer & Explanation</summary>

**A.** ESC1 = enrollee-specified SAN + client auth + low-priv enrollment, so you enroll as `administrator@certco.local`. The CA key is not exposed (B); revocation (C) is not the condition; ESC1 specifically lacks the approval requirement (D is inverted).
</details>

---

**Q6.** With the ESC1 Administrator certificate, how do you convert it into domain compromise?

- A. PKINIT-authenticate with the cert to obtain a TGT (and optionally the NT hash), then DCSync.
- B. Import the cert into your browser and log in to the DC web console.
- C. Crack the certificate's private key to recover the password.
- D. Use the cert as a Golden Ticket directly.

<details><summary>Show Answer & Explanation</summary>

**A.** The cert authenticates via PKINIT to yield a TGT (UnPAC-the-hash can give the NT hash), enabling DCSync. It is not a browser login (B); there is no password to crack (C); a cert is not a Golden Ticket (D).
</details>

---

**Q7.** STANDALONE-G: a web RCE drops you in a container. `ls -la /var/run/` shows `docker.sock`. Escape?

- A. `docker run -v /:/host -it <img> chroot /host sh` to become host root.
- B. `exit` the container, since the socket auto-mounts the host.
- C. Reboot the container to merge it with the host namespace.
- D. Read `docker.sock` with `cat` to extract the host password.

<details><summary>Show Answer & Explanation</summary>

**A.** A reachable Docker socket lets you launch a host-mounting container and chroot to host root. The socket does not auto-mount the host (B); rebooting does not merge namespaces (C); the socket is an API, not a file of passwords (D).
</details>

---

**Q8.** STANDALONE-H foothold holds `SeBackupPrivilege`. Escalation to local admin?

- A. `reg save` the SAM and SYSTEM hives, dump hashes offline, then PtH the local admin.
- B. Dump LSASS live, since backup privilege grants debug rights.
- C. Reconfigure a service binPath, since backup privilege edits services.
- D. Install an MSI, since backup privilege enables AlwaysInstallElevated.

<details><summary>Show Answer & Explanation</summary>

**A.** `SeBackupPrivilege` bypasses file ACLs: save SAM+SYSTEM, `secretsdump ... LOCAL`, then Pass-the-Hash. LSASS is a debug (B) not backup capability; it does not reconfigure services (C) or enable installer elevation (D).
</details>

---

**Q9.** STANDALONE-I: `ltrace` on a custom SUID binary shows `system("backup.sh")` (bare name). Escalation?

- A. Place a malicious `backup.sh` earlier in `$PATH` and run the SUID binary.
- B. Overwrite `/bin/sh`, since SUID binaries always call it.
- C. Use LD_PRELOAD against the SUID binary from the environment.
- D. Exploit a format string in the `system()` argument.

<details><summary>Show Answer & Explanation</summary>

**A.** A bare command name resolves via `$PATH`; a malicious `backup.sh` in a writable PATH entry runs as root. No LD_PRELOAD from env on SUID (C); `/bin/sh` overwrite (B) is not indicated and usually impossible; no format string is shown (D).
</details>

---

**Q10.** It is hour 2 and PORTAL01's upload filter is resisting you. The AD set depends on this foothold. Decision?

- A. Systematically test each bypass class (extension, content-type, magic byte, handler/.htaccess) rather than guessing.
- B. Abandon the AD set and focus only on standalones.
- C. Brute-force the admin login instead.
- D. Run an automated web scanner to find the bypass.

<details><summary>Show Answer & Explanation</summary>

**A.** Work the bypass matrix methodically against the specific validation. Abandoning 40 points at hour 2 (B) is premature; brute force (C) is unrelated; scanners (D) are banned.
</details>

---

**Q11.** You get host root on STANDALONE-G via the container escape. Where is `proof.txt` expected, and what proof do you capture?

- A. On the host filesystem as root; screenshot `proof.txt` with `id` and `ip a` showing host root.
- B. Inside the container only; the host flag does not count.
- C. In the Docker socket file itself.
- D. On your Kali box after exfiltration.

<details><summary>Show Answer & Explanation</summary>

**A.** The escalation goal is *host* root; capture `proof.txt` on the host with identity/network in frame. The container (B) is only the foothold; the socket (C) holds no flag; the flag lives on the target, not Kali (D).
</details>

---

**Q12.** WORK02 enumeration shows `cmdkey /list` has a stored Administrator credential. How do you use it without the password?

- A. `runas /savecred /user:Administrator` to launch your payload with the saved credential.
- B. `type` the Credential Manager vault to read the plaintext.
- C. Feed it to secretsdump for the NT hash.
- D. Export it and crack it offline.

<details><summary>Show Answer & Explanation</summary>

**A.** `runas /savecred` uses the stored credential directly. The vault is DPAPI-encrypted (B); secretsdump does not consume cmdkey entries (C); there is no crackable export (D).
</details>

---

**Q13.** You pivoted and BloodHound shows two paths to DA: a long delegation chain and the ESC1 template you can enroll against now. Choose?

- A. Take ESC1 — it is a short, well-understood path you can execute immediately.
- B. Take the delegation chain to demonstrate more techniques.
- C. Do both in parallel to hedge.
- D. Neither; re-run SharpHound to find a third path.

<details><summary>Show Answer & Explanation</summary>

**A.** Shortest reliable path to the objective wins; ESC1 is quick and deterministic here. The longer chain (B) spends time for no extra points; parallel work (C) splits focus; re-running collection (D) delays progress.
</details>

---

**Q14.** Your Certipy request fails with a time-skew Kerberos error. Most likely fix?

- A. Sync your clock to the DC (e.g. `ntpdate`/`faketime`) since Kerberos requires close time alignment.
- B. Re-run with a different template, since skew means the template is patched.
- C. Reboot the DC to reset its clock.
- D. Ignore it; skew errors are cosmetic.

<details><summary>Show Answer & Explanation</summary>

**A.** Kerberos (and PKINIT) require clock alignment; sync to the DC. Skew is unrelated to the template (B); you cannot reboot the DC (C); skew breaks auth, it is not cosmetic (D).
</details>

---

**Q15.** You have 50 points at hour 14: AD foothold+lateral (partial) and one standalone. What maximises your pass chance?

- A. Finish a second standalone (banking 20) while periodically retrying the AD DC step.
- B. Pour all remaining time into the DC step alone.
- C. Start a brand-new standalone and the AD DC simultaneously, splitting attention evenly.
- D. Stop and write the report with 50 points.

<details><summary>Show Answer & Explanation</summary>

**A.** A second standalone is a reliable 20 toward the 70 line, with opportunistic DC retries. All-in on the DC (B) risks stalling; even splitting (C) dilutes; stopping at 50 (D) fails.
</details>

---

**Q16.** PORTAL01's service account shell has `SeImpersonatePrivilege`. You need SYSTEM locally to read a protected config. Approach?

- A. Use a Potato tool to get SYSTEM, then read the config.
- B. Read the config as the service account; SYSTEM is unnecessary.
- C. Reboot into safe mode to bypass the file ACL.
- D. Use icacls to grant yourself rights without elevation.

<details><summary>Show Answer & Explanation</summary>

**A.** `SeImpersonate` to SYSTEM (Potato) grants access to the protected file. If the service account could already read it (B) you would not need SYSTEM; safe mode (C) is not available remotely; icacls without rights (D) fails on a protected file.
</details>

---

**Q17.** During STANDALONE-I log poisoning, your PHP payload in the User-Agent is not executing when you include the log. Likely cause to check first?

- A. The web user may lack read access to the log, or the log path/escaping differs — verify the exact log path and permissions.
- B. PHP automatically strips all payloads from logs, so log poisoning never works.
- C. The target must be rebooted for the poisoned log to take effect.
- D. You need root to include any log file via LFI.

<details><summary>Show Answer & Explanation</summary>

**A.** Confirm the correct log path and that the web user can read it; wrong path/permissions is the usual failure. PHP does not auto-strip payloads (B); no reboot is needed (C); LFI uses the web user's context, not root (D).
</details>

---

**Q18.** You realise at hour 23 that a screenshot for STANDALONE-H's `proof.txt` is missing the IP address in frame. Action?

- A. Re-take the screenshot now with `ipconfig`/`ip a` and `whoami`/`id` visible, while access remains.
- B. Submit it anyway; graders infer the host from context.
- C. Crop a previous screenshot to add the IP.
- D. Skip the box in the report to avoid the issue.

<details><summary>Show Answer & Explanation</summary>

**A.** Re-capture correct proof immediately while you still have access — incomplete proof risks the points. Assuming graders infer (B) is risky; fabricating/cropping (C) is dishonest and invalid; skipping (D) discards earned points.
</details>

---

**Q19.** For the ADCS finding in the report, which remediation is correct?

- A. Remove enrollee-supplied SAN / enforce manager approval and restrict enrollment on the template; audit all templates.
- B. Uninstall AD CS entirely and issue certificates manually.
- C. Shorten certificate lifetimes to one hour.
- D. Rename the template so Certipy cannot find it.

<details><summary>Show Answer & Explanation</summary>

**A.** Fix the template's SAN/approval/enrollment settings and audit the rest. Uninstalling AD CS (B) is disproportionate; short lifetimes (C) do not fix the enrollment flaw; renaming (D) is obscurity (Certipy enumerates via LDAP).
</details>

---

**Q20.** Across all three mocks, which habit most consistently predicted a pass?

- A. Full enumeration plus disciplined time-boxing and complete, reproducible notes.
- B. Memorising the most exploit payloads.
- C. Spending the longest time on the hardest box.
- D. Using automated tools to shortcut enumeration.

<details><summary>Show Answer & Explanation</summary>

**A.** The exam rewards methodology: enumerate fully, time-box, and keep reproducible notes. Payload memorisation (B) is brittle; over-investing in one box (C) is tunnel vision; banned/over-relied automation (D) fails you.
</details>

---

## Debrief

Compare your three mock scores and retros. Your lowest rubric dimension across all three is your exam-day risk — drill it deliberately before sitting the real OSCP.
