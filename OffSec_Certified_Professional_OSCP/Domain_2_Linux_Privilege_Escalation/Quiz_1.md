# Domain 2 — Quiz 1: SUID, Capabilities & Sudo Exploitation

> Scenario-driven, exam-difficulty. Answer before expanding.

---

**Q1.** `sudo -l` shows `(root) NOPASSWD: /usr/bin/awk`. Which invocation yields a root shell, and why does it work?

- A. `sudo awk 'BEGIN{system("/bin/sh")}'`, because awk's BEGIN block runs a shell as root.
- B. `sudo awk '/root/{print $0}' /etc/shadow`, because printing the file grants its privileges.
- C. `sudo awk -f /bin/sh`, because awk interprets the shell binary as a script.
- D. `sudo awk --exec /bin/sh`, because the exec flag launches a child shell.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** `awk`'s `BEGIN{system(...)}` executes a command before processing input; run under sudo it spawns `/bin/sh` as root. This is the GTFOBins awk escape.

- **B** only *reads* `/etc/shadow` (useful, but not a shell) and relies on the file being an argument; it does not escalate to an interactive root shell.
- **C** is wrong: `-f` expects an awk program file; `/bin/sh` is not valid awk and errors.
- **D** is wrong: awk has no `--exec` flag.
</details>

---

**Q2.** `sudo -l` output includes `env_keep+=LD_PRELOAD` and permission to run `/usr/bin/apache2ctl` as root. What is the correct escalation path?

- A. Compile a shared object with a constructor that spawns a root shell, then run the allowed binary with `LD_PRELOAD` set to it.
- B. Replace `/usr/bin/apache2ctl` with a shell script, since the binary itself is writable under sudo.
- C. Set `LD_LIBRARY_PATH` to `/tmp` and drop a library there, since sudo preserves that path too.
- D. Export `LD_PRELOAD` to point at `/bin/bash` directly, since bash loads as a library when preloaded.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** `env_keep+=LD_PRELOAD` means sudo preserves `LD_PRELOAD`. A `.so` whose `_init`/constructor calls `setuid(0);system("/bin/bash")`, preloaded while running any sudo-allowed binary, executes as root.

- **B** is wrong: the sudo rule permits *running* the binary, not writing to it; `/usr/bin` is not writable by you.
- **C** is wrong: the rule preserves `LD_PRELOAD`, not `LD_LIBRARY_PATH`; the loader ignores the latter for the elevated run here.
- **D** is wrong: `/bin/bash` is an executable, not a preloadable shared library with a constructor; preloading it does nothing useful.
</details>

---

**Q3.** A custom SUID-root binary, when run under `ltrace`, shows a call to `system("id")` (bare command, no path). Which escalation fits precisely?

- A. Place a malicious `id` earlier in `$PATH` so the SUID binary runs your version as root.
- B. Pass `id` a crafted argument that overflows its buffer and returns to your shellcode.
- C. Preload a shared object into `id` using `LD_PRELOAD` while the SUID binary runs.
- D. Replace `/usr/bin/id` on disk, since SUID binaries always call commands by absolute path.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** A bare command in `system()` is resolved via `$PATH`. Prepending a writable directory containing a malicious `id` makes the root-SUID binary execute your `id` as root — classic PATH hijack.

- **B** is wrong: nothing indicates a buffer overflow; the finding is a relative command call, a logic flaw.
- **C** is wrong: SUID binaries ignore `LD_PRELOAD` from the environment (loader security), so this fails unless sudo preserves it — not the case here.
- **D** is wrong: the premise is false (the binary used a *bare* name), and you cannot write to `/usr/bin` as a normal user anyway.
</details>

---

**Q4.** `getcap -r / 2>/dev/null` returns `/usr/bin/python3.8 = cap_setuid+ep`. What does this permit, and how do you abuse it?

- A. The binary may call `setuid(0)`, so run Python to set UID 0 and spawn a shell.
- B. The binary may read any file, so run Python to read `/etc/shadow` only.
- C. The binary may bind low ports, so run Python to start a privileged listener.
- D. The binary may load kernel modules, so run Python to insert a rootkit.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** `cap_setuid+ep` lets the binary change its UID. `python3.8 -c 'import os;os.setuid(0);os.system("/bin/bash")'` becomes root directly.

- **B** describes `cap_dac_read_search`, a different capability; `cap_setuid` is about changing UID, not bypassing read checks.
- **C** describes `cap_net_bind_service`; not what is set here.
- **D** describes `cap_sys_module`; not what is set here.
</details>

---

**Q5.** You find `/usr/bin/vim.basic = cap_dac_read_search+ep`. Which outcome is directly achievable, and what is the limitation?

- A. Read any file on the system regardless of permissions, but you cannot change UID with this capability alone.
- B. Write to any file on the system regardless of permissions, including `/etc/passwd` directly.
- C. Execute any command as root, because read access implies full control of the host.
- D. Bypass SELinux policy entirely, because the capability overrides mandatory access control.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** `cap_dac_read_search` bypasses *read/search* permission checks, so vim can open `/etc/shadow` and SSH keys — but it does **not** grant UID 0 or write bypass, so you leverage the *data* you read (crack a hash, reuse a key), not a direct shell.

- **B** confuses it with `cap_dac_override` (write bypass); read-search does not grant writes.
- **C** overstates: reading files is not arbitrary command execution.
- **D** is wrong: DAC capabilities concern discretionary access control, not SELinux/MAC.
</details>

---

**Q6.** On a sudo version older than 1.8.28, a rule reads `(ALL, !root) NOPASSWD: /bin/bash`. The intent is to allow running bash as any user *except* root. Which command defeats that intent?

- A. `sudo -u#-1 /bin/bash`, because UID -1 wraps to 0 and bypasses the `!root` restriction.
- B. `sudo -u root /bin/bash`, because the `!root` token is ignored by older sudo entirely.
- C. `sudo !root /bin/bash`, because the negation can be echoed back to re-enable root.
- D. `sudo -g root /bin/bash`, because specifying the group overrides the user restriction.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** CVE-2019-14287: sudo's UID parsing lets `-u#-1` (and `-u#4294967295`) resolve to 0, sidestepping the `!root` exclusion and running bash as root.

- **B** is wrong: `-u root` is exactly what `!root` blocks; it is denied.
- **C** is wrong: `!root` is sudoers syntax, not a command argument you can replay.
- **D** is wrong: `-g` sets the group, not the user; it does not grant UID 0 here.
</details>

---

**Q7.** You obtained a shell as `www-data` and want to stabilise it before escalating. After backgrounding your netcat shell, which sequence restores a fully interactive TTY?

- A. Run a Python pty spawn, background it, then `stty raw -echo; fg`, then export TERM.
- B. Run `sudo su -` immediately, since the TTY upgrades automatically once root.
- C. Run `ssh localhost` from inside the shell, since SSH always provides a full TTY.
- D. Run `script /dev/null` and then disconnect, since detaching allocates a new TTY.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** `python3 -c 'import pty;pty.spawn("/bin/bash")'`, Ctrl-Z, then `stty raw -echo; fg` and `export TERM=xterm` yields a working TTY (arrows, Ctrl-C, tab).

- **B** is wrong: you are not root yet and have no creds; `sudo su -` fails.
- **C** is wrong: you lack local SSH creds as `www-data`; and it would not reliably work on an exam foothold.
- **D** is wrong: `script /dev/null` can start an upgrade, but *disconnecting* does not allocate a usable interactive TTY for your current session.
</details>

---

**Q8.** For the exam report, you must give the root cause of a PATH-hijack escalation through a SUID binary. Which statement is correct and actionable?

- A. The binary invoked an external command by relative name; call external commands by absolute path and sanitise `PATH`.
- B. The binary had the SUID bit set; removing SUID from all system binaries prevents the attack.
- C. The user had a writable home directory; mounting `/home` read-only removes the vulnerability.
- D. The shell honoured `$PATH`; replacing bash with a restricted shell eliminates PATH resolution.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** The defect is calling a command by bare name so `$PATH` controls resolution. The fix: invoke with an absolute path (and reset `PATH` to a trusted value) inside the privileged binary.

- **B** overcorrects: many system binaries *must* be SUID (e.g. `passwd`); the problem is this binary's relative call, not SUID in general.
- **C** is wrong: a writable home is incidental; the attacker controls `PATH` and the hijack dir regardless.
- **D** is wrong: honouring `$PATH` is normal shell behaviour; the fix belongs in how the privileged binary calls commands, not in replacing the user shell.
</details>
