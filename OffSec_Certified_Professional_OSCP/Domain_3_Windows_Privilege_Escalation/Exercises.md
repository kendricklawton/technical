# Domain 3 — Hands-On Lab Exercises

> **Lab safety & authorisation.** Use Windows VMs you own (an evaluation Windows 10/11 or Server ISO from Microsoft) or Proving Grounds / HackTheBox Windows boxes you are subscribed to. Snapshot before each exercise. Never target systems you are not authorised to test.

---

## Exercise 3.1 — `whoami /priv` Triage

**Objective:** map each dangerous privilege to its escalation.

**Setup:** a service account shell (simulate by running your foothold as an IIS apppool or MSSQL service) with `SeImpersonatePrivilege` enabled.

**Tasks:**
1. Run `whoami /priv` and identify every privilege marked Enabled.
2. For `SeImpersonatePrivilege`, use PrintSpoofer or GodPotato to obtain a SYSTEM shell.
3. Write a one-line explanation of *why* impersonation leads to SYSTEM.

**Success criteria:** `nt authority\system` confirmed, with the token-impersonation mechanism documented.

---

## Exercise 3.2 — SeBackupPrivilege to Hashes

**Objective:** extract local hashes using backup privilege.

**Setup:** a shell holding `SeBackupPrivilege`/`SeRestorePrivilege`.

**Tasks:**
1. `reg save HKLM\SAM` and `HKLM\SYSTEM` to a working directory.
2. Transfer both hives to Kali.
3. Run `impacket-secretsdump -sam sam.hiv -system system.hiv LOCAL` and recover the local Administrator hash.

**Success criteria:** the local admin NT hash extracted offline, ready for Pass-the-Hash.

---

## Exercise 3.3 — Weak Service ACL Reconfiguration

**Objective:** escalate by reconfiguring a service you can modify.

**Setup:** create a service whose ACL grants your user `SERVICE_CHANGE_CONFIG`.

**Tasks:**
1. Find it with `accesschk.exe -uwcqv user *`.
2. Generate a reverse-shell exe with `msfvenom` and host a listener.
3. `sc config` the binPath to your exe, restart the service, and catch SYSTEM.

**Success criteria:** SYSTEM shell via service reconfiguration, with the original binPath recorded for restoration.

---

## Exercise 3.4 — Unquoted Service Path

**Objective:** exploit an unquoted path with a writable intermediate directory.

**Setup:** install a service with `binPath=C:\Program Files\Vuln Svc\svc.exe` (unquoted) and make an intermediate directory writable.

**Tasks:**
1. Enumerate unquoted auto-start services with `wmic`.
2. Confirm which intermediate directory you can write to.
3. Drop a payload named to win the search order and restart the service.

**Success criteria:** code execution as the service account, plus an explanation of the space-and-search-order mechanic.

---

## Exercise 3.5 — DLL Hijacking with Procmon

**Objective:** find and exploit a missing DLL loaded by a privileged service.

**Setup:** a service/app that loads a DLL by name from a writable path (or fails to find one).

**Tasks:**
1. Run Procmon filtered on `Result is NAME NOT FOUND` and `Path ends with .dll`.
2. Identify a hijackable DLL in a writable location.
3. Compile a malicious DLL exporting the needed function and plant it; trigger the load.

**Success criteria:** code execution via the planted DLL, with the Procmon evidence saved.

---

## Exercise 3.6 — AlwaysInstallElevated

**Objective:** install an MSI as SYSTEM.

**Setup:** set both `AlwaysInstallElevated` registry values to 1.

**Tasks:**
1. Confirm both HKCU and HKLM values are 1.
2. Build an MSI reverse shell with `msfvenom -f msi`.
3. Install with `msiexec /quiet /qn /i` and catch SYSTEM.

**Success criteria:** SYSTEM shell via MSI, with a note that *both* keys must be 1.

---

## Exercise 3.7 — Credential Hunt

**Objective:** harvest credentials from five common locations.

**Setup:** seed a VM with an unattend.xml password, a GPP `cpassword`, a secret in PowerShell history, a `cmdkey` stored credential, and a KeePass database.

**Tasks:**
1. Recover each secret with the appropriate technique (`gpp-decrypt`, reading `ConsoleHost_history.txt`, `cmdkey /list` + `runas /savecred`, etc.).
2. Use one recovered credential to escalate or move laterally.

**Success criteria:** at least four of five secrets recovered and one used to gain higher privilege.

---

## Exercise 3.8 — UAC Bypass

**Objective:** go from medium-integrity admin to high integrity without a prompt.

**Setup:** a local admin account running at medium integrity.

**Tasks:**
1. Confirm your integrity level and admin membership.
2. Perform a `fodhelper.exe` registry-hijack UAC bypass to spawn a high-integrity shell.
3. Verify with `whoami /groups` (Mandatory Label High).

**Success criteria:** a high-integrity shell obtained without a UAC prompt.

---

## Consolidation Challenge — WinPEAS Blind Drill

Run `winPEAS.exe` once on a lab box, close it, and escalate from notes alone — manually confirming each service ACL, path, and credential lead. Then diff what you used against WinPEAS output. This trains you to not depend on the tool's colour-coding, which the exam's deliberately noisy boxes punish.
