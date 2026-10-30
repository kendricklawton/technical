# Domain 3 — Quiz 2: Service Misconfigurations, Unquoted Paths & DLL Hijacking

> Scenario-driven, exam-difficulty. Answer before expanding.

---

**Q1.** `accesschk.exe -uwcqv youruser *` shows you have `SERVICE_CHANGE_CONFIG` on a SYSTEM service. Which action escalates, and what must you preserve?

- A. Reconfigure the service binPath to your payload and restart it, recording the original binPath to restore later.
- B. Delete the service and recreate it under your own user account to inherit SYSTEM.
- C. Change the service start type to disabled so it reloads your payload on boot.
- D. Rename the service so the SCM launches a fallback shell in its place.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** `SERVICE_CHANGE_CONFIG` lets you `sc config <svc> binPath= "...payload"`; restarting runs it as the service account (SYSTEM). Preserve the original binPath so you can restore the service for the client.

- **B** is wrong: recreating "under your own user" would run as *you*, not SYSTEM, and deletion is destructive.
- **C** is wrong: disabling prevents it from starting at all; nothing of yours runs.
- **D** is wrong: renaming does not spawn a fallback shell; the SCM just fails to find it.
</details>

---

**Q2.** A service has `binPath = C:\Program Files\Acme Agent\agent.exe` (unquoted). Which path does Windows try *first*, and when is it exploitable?

- A. `C:\Program.exe` first; exploitable if you can write to a probed earlier location such as `C:\`.
- B. `C:\Program Files\Acme Agent\agent.exe` first; exploitable only if that exact file is writable.
- C. `C:\Program Files\Acme.exe` first; exploitable only if `Program Files` is writable.
- D. The last segment first; exploitable only if the final directory is writable.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** With an unquoted path, `CreateProcess` tries `C:\Program.exe`, then `C:\Program Files\Acme.exe`, then the real target. It is exploitable wherever you can write an executable in a probed-earlier location (often an intermediate dir; `C:\` root is usually not writable by non-admins but the mechanic is the same).

- **B** describes the *writable-binary* case, not the unquoted-path mechanic.
- **C** names the wrong first probe (the first is `C:\Program.exe`).
- **D** inverts the search order; probing goes left to right, not from the last segment.
</details>

---

**Q3.** You want to confirm which DLL a SYSTEM service fails to find so you can hijack it. Which Procmon filter isolates the opportunity?

- A. Result is `NAME NOT FOUND` and Path ends with `.dll`, scoped to the service process.
- B. Operation is `RegOpenKey` and Result is `SUCCESS`, scoped to all processes.
- C. Path contains `System32` and Result is `SUCCESS`, scoped to the service process.
- D. Operation is `Process Create` and Result is `ACCESS DENIED`, scoped to all processes.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** A `NAME NOT FOUND` on a `.dll` the service tries to load — in a directory you can write to — is exactly the hijack opportunity; scoping to the process cuts noise.

- **B** finds successful registry reads, irrelevant to a missing-DLL hijack.
- **C** shows DLLs it *did* find in System32 (not writable, not missed).
- **D** targets process creation denials, not DLL load failures.
</details>

---

**Q4.** The service's own `.exe` is writable by your user, and you can restart the service. What is the cleanest escalation?

- A. Replace the binary with your payload, restart the service, then restore the original after proving SYSTEM.
- B. Attach a debugger to the running binary and patch its memory to spawn a shell.
- C. Symlink the binary to `cmd.exe` so the SCM launches a console as SYSTEM.
- D. Set the binary read-only so the SCM loads a cached elevated copy instead.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** A writable service binary is directly replaceable; restart runs your code as the service account. Keep a copy of the original to restore.

- **B** is overcomplex and unnecessary when you can simply replace the file; it also may require privileges you lack.
- **C** is wrong: a symlink to `cmd.exe` as a service binary will not yield an interactive SYSTEM shell cleanly and may fail SCM's control handshake.
- **D** is wrong: there is no "cached elevated copy" behaviour.
</details>

---

**Q5.** DLL hijacking relies on the Windows DLL search order. Which statement about that order is correct for a default (safe-search-enabled) system?

- A. The application's own directory is searched before the system directories.
- B. The current working directory is always searched first, before the application directory.
- C. The `PATH` directories are searched before the application directory.
- D. Only `System32` is ever searched, so hijacking requires write access there.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** With SafeDllSearchMode (default), the executable's directory is searched first, then system dirs, then `PATH`. A writable app directory (or a missing DLL resolved from a writable location) enables hijacking.

- **B** is wrong: safe search mode moves the CWD *after* the system directories precisely to reduce hijacking.
- **C** is wrong: `PATH` is searched *after* the application and system directories.
- **D** is wrong: multiple locations are searched, not only System32.
</details>

---

**Q6.** You find a weak ACL on `HKLM\SYSTEM\CurrentControlSet\Services\Vuln`. Which modification is equivalent to reconfiguring the service binary?

- A. Set the `ImagePath` value to your payload, so the SCM launches it as the service account.
- B. Set the `Start` value to 4, so the service disables and reloads your payload.
- C. Delete the `ObjectName` value, so the service defaults to running as your user.
- D. Set the `Type` value to 0x10, so the service runs in an elevated session.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** `ImagePath` is the on-disk command the SCM runs; writable access to the service key lets you point it at your payload — the registry-side equivalent of `sc config binPath=`.

- **B** is wrong: `Start=4` disables the service; it does not run your payload.
- **C** is wrong: deleting `ObjectName` does not make it run as *you*; it defaults to LocalSystem, and this does not set your binary.
- **D** is wrong: `Type` controls service kind (own/shared/driver), not an "elevated session," and does not set your binary.
</details>

---

**Q7.** Before changing a production service's binPath on a real engagement, what is the professional sequence?

- A. Record the current configuration, make the change to demonstrate impact, then restore the original configuration.
- B. Change it permanently and leave your payload installed as proof for the client to find.
- C. Stop all other services first so your change cannot be interrupted by dependencies.
- D. Disable Windows Defender so your payload binary is not quarantined during the test.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** Capture the original (`sc qc`), demonstrate the escalation, then restore — proving impact without leaving the service broken.

- **B** is wrong: leaving a payload installed is unprofessional and risky for the client.
- **C** is wrong: stopping unrelated services causes unnecessary disruption.
- **D** is wrong: disabling security controls is destructive and out of scope unless explicitly authorised.
</details>

---

**Q8.** For the report, which remediation set addresses the service vulnerabilities in this quiz?

- A. Correct service and registry-key ACLs, quote all service paths, and protect service binaries and their directories.
- B. Set all services to manual start so attackers cannot restart them to trigger payloads.
- C. Run every service as a dedicated low-privilege user instead of applying ACL fixes.
- D. Rename service binaries with GUIDs so their paths cannot be predicted by attackers.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** The fixes map to the flaws: restrict who can reconfigure services/keys, quote paths with spaces, and lock down binary/directory permissions (also validating DLL load paths).

- **B** is wrong: manual start does not fix the underlying ACL/path flaws and breaks legitimate auto-start needs.
- **C** is partial and sometimes impossible (some services need SYSTEM); it does not replace fixing ACLs/paths.
- **D** is security by obscurity; permissions, not secret names, are the control.
</details>
