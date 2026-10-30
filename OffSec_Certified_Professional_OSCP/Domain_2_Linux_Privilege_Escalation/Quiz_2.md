# Domain 2 — Quiz 2: Cron Jobs, PATH Hijacking & Wildcard Injection

> Scenario-driven, exam-difficulty. Answer before expanding.

---

**Q1.** You cannot read root's crontab, but you suspect a periodic root job. Which approach reveals it without any file-read access?

- A. Run pspy to watch processes and their UIDs as they spawn in real time.
- B. Read `/etc/shadow` to correlate login times with the job schedule.
- C. Run `crontab -l` as your user to list every account's scheduled jobs.
- D. Inspect `/proc/1/environ` to recover the cron daemon's job table.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** `pspy64` monitors `/proc` for new processes without root, exposing commands and UIDs of cron jobs as they fire — the standard way to discover unreadable root cron.

- **B** is wrong: `/etc/shadow` holds password hashes, not schedules, and is not readable anyway.
- **C** is wrong: `crontab -l` only lists *your* crontab, not root's or other users'.
- **D** is wrong: `/proc/1/environ` is init's environment, not a cron job table, and is not readable by you.
</details>

---

**Q2.** A root cron runs `/opt/clean.sh` every minute, and that script is writable by your user. What is the minimal change that reliably gives you root?

- A. Append a line copying bash to a SUID root binary, then run it with `-p` after the next tick.
- B. Replace the entire script with a reverse shell that connects back as your own user.
- C. Append a line that prints the root password to the script's standard output.
- D. Change the script's permissions to 777 so the scheduler runs it as your user.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** Appending `cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash` means the root cron creates a SUID-root bash; `/tmp/rootbash -p` then gives a root shell. Minimal and reliable.

- **B** is wrong: the cron runs as *root*, so a reverse shell would return root — but connecting "as your own user" misstates the outcome and a full rewrite risks breaking the job/detection; appending a SUID step is cleaner. (The key error is the stated "as your own user": the shell would be root.)
- **C** is wrong: the script does not know the root password; printing it yields nothing.
- **D** is wrong: `chmod 777` does not change *who* runs the job; cron still runs it as root, and this step alone grants nothing.
</details>

---

**Q3.** A root cron executes `backup.sh` whose contents are `tar czf /root/b.tgz *` run inside `/home/user/data` (writable by you). Which files create root execution via wildcard injection?

- A. Files named `--checkpoint=1` and `--checkpoint-action=exec=sh payload.sh`, plus the payload itself.
- B. A file named `*.sh` containing the payload, since tar executes matching scripts.
- C. A file named `.tar` containing option flags that tar auto-sources on startup.
- D. A symlink named `backup.sh` pointing to `/bin/bash` in the data directory.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** The shell expands `*` into filenames as arguments; tar reads `--checkpoint-action=exec=...` as an option and runs your script as root.

- **B** is wrong: tar does not "execute matching scripts"; a `*.sh` filename is just archived.
- **C** is wrong: tar does not auto-source a `.tar` file for options; there is no such startup behaviour here.
- **D** is wrong: the cron calls `backup.sh` by its own path elsewhere; a symlink in the *data* dir is just another file to the `*` glob, not the executed script.
</details>

---

**Q4.** A root cron sets `PATH=/home/user/bin:/usr/bin` and runs `backup` (bare name). `/home/user/bin` is writable by you. What is the escalation?

- A. Place an executable named `backup` in `/home/user/bin` so cron runs your version as root.
- B. Rename `/usr/bin/backup` so cron falls back to a shell when it fails to find it.
- C. Set your own `PATH` to include `/root/bin` so cron inherits the change.
- D. Create `/home/user/bin/PATH` containing a new search path for the scheduler.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** Cron searches `PATH` left to right; `/home/user/bin` precedes `/usr/bin`, so a malicious `backup` there is executed as root before the real one is ever reached.

- **B** is wrong: you cannot rename files in `/usr/bin`, and a missing command yields an error, not a shell.
- **C** is wrong: the cron job's `PATH` is set in the crontab, not inherited from your interactive shell.
- **D** is wrong: there is no `PATH` *file* mechanism; `PATH` is an environment variable set in the job definition.
</details>

---

**Q5.** Why does wildcard injection succeed even though the cron author never intended any file to be treated as an option?

- A. The shell expands the glob into arguments before the program runs, so filenames become indistinguishable from flags.
- B. The program re-reads its own source at runtime and executes any comment that looks like a flag.
- C. The kernel intercepts the glob and passes privileged filenames straight to the program as root.
- D. Cron pre-parses filenames and promotes any starting with a dash into environment variables.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** Glob expansion happens in the shell *before* exec; the program receives a plain argv it cannot trace back to a wildcard, so a file named `--flag` is received as the flag `--flag`.

- **B** is wrong: programs do not re-read and execute their own comments.
- **C** is wrong: the kernel does not expand globs or grant privilege to filenames.
- **D** is wrong: cron does not promote dash-prefixed filenames to environment variables; the shell simply passes them as arguments.
</details>

---

**Q6.** You have a writable systemd service unit that root starts on boot/restart. Which modification escalates, assuming you can trigger a restart?

- A. Set `ExecStart` to a command that creates a SUID root shell, then trigger the service restart.
- B. Set `User=root` in your own user service, since any unit may self-elevate to root.
- C. Add `Environment=LD_PRELOAD=/tmp/e.so` to a user-scope unit you fully control.
- D. Rename the unit to `root.service` so systemd runs it in the root slice automatically.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** If root runs the service and you can edit its `ExecStart` (and cause a restart), your command runs as root — e.g. `cp /bin/bash /tmp/rb; chmod +s /tmp/rb`.

- **B** is wrong: a *user* service runs as you; declaring `User=root` in a user unit does not grant root without a privileged manager.
- **C** is wrong: a user-scope unit you control runs as you; preloading into your own process gives nothing new.
- **D** is wrong: there is no "root slice by filename" behaviour; the unit's privilege comes from the manager that runs it, not its name.
</details>

---

**Q7.** LinPEAS highlights a writable `/etc/cron.d/backup` file in red. Before acting, what is the responsible verification step on a real engagement?

- A. Confirm the file is genuinely writable by you and that a root process consumes it, then document impact without breaking the job.
- B. Immediately overwrite it with a reverse shell, since red highlights are always exploitable.
- C. Delete the file to stop the job, then recreate it with your payload at your convenience.
- D. Edit it to disable logging first, so the escalation leaves no trace in the system logs.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** Verify the permission and that cron actually runs it as root, then exploit carefully, preserving the original so you can restore it — professional impact demonstration without collateral damage.

- **B** is wrong: red highlights are *leads*; blind overwrites can break services or be false positives.
- **C** is wrong: deleting a client's cron file causes unauthorised disruption.
- **D** is wrong: deliberately disabling logging to hide activity is out of scope for an authorised test and unprofessional; you document your actions, not conceal them.
</details>

---

**Q8.** For the report, which remediation set correctly addresses cron-based escalations you found?

- A. Make root cron scripts non-writable by users, set absolute PATHs, and avoid unquoted globs over user-writable directories.
- B. Disable cron entirely and run all scheduled work manually from an interactive root shell.
- C. Move all cron jobs into user crontabs so they no longer execute with root privileges.
- D. Rename sensitive scripts with random suffixes so attackers cannot locate them to modify.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** The real fixes: correct ownership/permissions on scripts, hardcode absolute `PATH`, and never run `cmd *` over directories users can write to.

- **B** is impractical and throws away legitimate automation; it is not a proportionate fix.
- **C** is wrong: many jobs legitimately need root; relocating them to user crontabs breaks functionality and dodges the real issue.
- **D** is security by obscurity; permissions, not secret names, are the control.
</details>
