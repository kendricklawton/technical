# Domain 3 — Reference White Papers & Deep Dives

> Mechanism-level background for Windows privilege escalation. For each, articulate the *legitimate feature* being abused.

---

## 1. The Windows Access Token Model

- Every process/thread carries an **access token**: the user SID, group SIDs, and a **privilege** list. The SRM checks the token on each securable-object access.
- **Impersonation** lets a server thread temporarily adopt a client's token to act on its behalf — essential for services. `SeImpersonatePrivilege` authorises this. The Potato exploits weaponise it by coercing a *SYSTEM* service to authenticate to a local endpoint, yielding a SYSTEM token to impersonate.
- **Integrity levels** (Low/Medium/High/System) and UAC's split token explain why an admin often runs at Medium until elevated.

**Sources:** Microsoft *Access Tokens* and *Privilege Constants* docs; "Rotten/Juicy/Rogue/GodPotato" research write-ups; James Forshaw (Project Zero) token research.

---

## 2. Backup/Restore and Debug Privileges

- `SeBackupPrivilege`/`SeRestorePrivilege` exist so backup software can read/write every file regardless of ACL. That same bypass lets an attacker copy `HKLM\SAM`/`SYSTEM` (hashes) or overwrite protected files.
- `SeDebugPrivilege` grants a handle to any process — intended for debuggers — enabling LSASS dumping and token theft.
- `SeTakeOwnershipPrivilege` lets you seize ownership of any object and then rewrite its DACL.

**Sources:** Microsoft privilege-constant documentation; Impacket `secretsdump` internals; Sysinternals docs.

---

## 3. Windows Services and the SCM

- The Service Control Manager launches services, typically as `LocalSystem`. A service's security is only as strong as the DACL on the service object, the binary, its directory, and its registry key.
- **Unquoted paths:** `CreateProcess` tokenises an unquoted path on spaces, probing `C:\Program.exe` before `C:\Program Files\...`. A writable earlier segment = execution as the service account.
- **DLL search order:** documented order (app dir, then system dirs, then `PATH`). A writable directory earlier in the order, or a missing DLL, is hijackable. Procmon's `NAME NOT FOUND` filter is the discovery tool.

**Sources:** Microsoft *Dynamic-Link Library Search Order* and *Service Security and Access Rights*; Sysinternals `accesschk`/`procmon`; PowerUp/PrivEscCheck source.

---

## 4. Installer Elevation and `AlwaysInstallElevated`

- Windows Installer can run packages with elevated rights. The `AlwaysInstallElevated` policy (both HKLM *and* HKCU set to 1) makes *any* MSI install as SYSTEM — a policy intended for constrained software deployment but catastrophic if left on for all users.

**Sources:** Microsoft Windows Installer policy docs; MSI elevation advisories.

---

## 5. Credential Storage on Windows

- **DPAPI** encrypts per-user secrets (browser passwords, Credential Manager) with a key derived from the user's password, protected by a masterkey. Mimikatz decrypts blobs given the masterkey or the user context.
- **GPP `cpassword`:** legacy Group Policy Preferences stored an AES-encrypted password in SYSVOL using a *publicly documented key* (MS14-025). Any domain user could read and decrypt it — hence `gpp-decrypt`.
- **Unattend/sysprep** files frequently retain provisioning passwords; **PowerShell history** records command-line secrets.

**Sources:** Microsoft DPAPI docs; MS14-025 bulletin; Mimikatz wiki; Benjamin Delpy's DPAPI research.

---

## 6. UAC Internals

- UAC gives admins a *filtered* (medium-integrity) token for normal use and a full token on consent. It is explicitly **not a security boundary** (per Microsoft) — auto-elevating binaries (`fodhelper`, `eventvwr`) that read user-writable registry paths can be hijacked to elevate silently.

**Sources:** Microsoft UAC documentation; UACME project (technique catalogue); Microsoft's "UAC is not a security boundary" guidance.

---

## 7. Primary Reference Index

- **HackTricks — Windows Local Privilege Escalation** (broad checklist; verify).
- **PayloadsAllTheThings — Windows Privilege Escalation**.
- **Sysinternals Suite** (`accesschk`, `procmon`, `autoruns`) — primary tooling.
- **PowerUp / PrivEscCheck / WinPEAS** source — learn what each check detects.
- **Microsoft Docs** — access tokens, privilege constants, service security: authoritative primary sources.

> **Study tip:** write a privilege→technique map (`SeImpersonate`→Potato, `SeBackup`→hive dump, `SeDebug`→LSASS, `SeTakeOwnership`→DACL rewrite). Recall speed on `whoami /priv` is a direct exam-time advantage.
