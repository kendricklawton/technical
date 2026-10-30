# Domain 2 — Hands-On Lab Exercises

> **Lab safety & authorisation.** Build a disposable Linux target VM (or use Proving Grounds / HackTheBox boxes you are subscribed to). Snapshot before each exercise so you can reset. Never run these against systems you do not own or are not authorised to test.

---

## Exercise 2.1 — `sudo -l` Decision Tree

**Objective:** map each `sudo -l` finding to the right escalation.

**Setup:** a VM where your user has three sudo rules: one GTFOBins binary (e.g. `find` or `vim`), one with `env_keep+=LD_PRELOAD`, and a vulnerable sudo version for CVE-2019-14287.

**Tasks:**
1. For the GTFOBins binary, obtain a root shell and note the exact escape feature abused.
2. Compile an `LD_PRELOAD` shared object and escalate through the `env_keep` rule.
3. Trigger CVE-2019-14287 with `sudo -u#-1`.

**Success criteria:** three independent root shells, each with a one-line explanation of the mechanism for the report.

---

## Exercise 2.2 — Reversing a Custom SUID Binary

**Objective:** escalate through a custom SUID binary by analysis, not guesswork.

**Setup:** compile a small SUID-root C program that calls `system("service --status-all")` (bare command name).

**Tasks:**
1. Run `strings` and `ltrace` against it; identify the relative command call.
2. Hijack `$PATH` with a malicious `service` binary and gain root.
3. Rebuild the program calling the command by absolute path; confirm the hijack no longer works and explain why.

**Success criteria:** root via PATH hijack, plus a written before/after comparison proving the absolute-path fix.

---

## Exercise 2.3 — Capabilities Enumeration

**Objective:** escalate using a file capability.

**Setup:** set `cap_setuid+ep` on a copy of Python (`setcap cap_setuid+ep /opt/python`), and `cap_dac_read_search+ep` on another binary.

**Tasks:**
1. Discover both with `getcap -r / 2>/dev/null`.
2. Use the `cap_setuid` binary to `setuid(0)` and spawn a root shell.
3. Use the `cap_dac_read_search` binary to read `/etc/shadow` despite lacking permission.

**Success criteria:** a root shell from the first and the root hash from the second, each with the capability's meaning noted.

---

## Exercise 2.4 — Cron Discovery with pspy

**Objective:** find and abuse a root cron job you cannot see in config.

**Setup:** a root cron (in `/var/spool/cron/crontabs/root`, unreadable to your user) that runs a world-writable script every minute.

**Tasks:**
1. Run `pspy64` and observe the job firing as UID 0.
2. Identify the writable script and inject `chmod +s /bin/bash`.
3. Wait for the next run and confirm root via `bash -p`.

**Success criteria:** root obtained, with pspy output saved as evidence that the job was discovered dynamically.

---

## Exercise 2.5 — Wildcard (Tar) Injection

**Objective:** achieve RCE through `tar *` option-smuggling.

**Setup:** a root cron running `cd /opt/backups && tar czf /root/backup.tgz *` over a directory you can write to.

**Tasks:**
1. Create the `--checkpoint=1` and `--checkpoint-action=exec=...` filenames plus a payload script.
2. Wait for the cron and confirm the payload executed as root.
3. Repeat the concept against `chown`/`rsync` if time permits.

**Success criteria:** root via wildcard injection, with an explanation of how shell glob expansion turns filenames into options.

---

## Exercise 2.6 — Writable `/etc/passwd`

**Objective:** add a UID-0 user.

**Setup:** make `/etc/passwd` world-writable in a lab VM.

**Tasks:**
1. Generate a DES/MD5 hash with `openssl passwd`.
2. Append a `:0:0:` user line and `su` to it.
3. Separately, read a provided `/etc/shadow` and crack root's hash with `hashcat`.

**Success criteria:** UID-0 shell via the injected user, plus a cracked root password.

---

## Exercise 2.7 — Docker Socket Escape

**Objective:** escape to the host via an exposed Docker socket.

**Setup:** a container (or a user in the `docker` group) with `/var/run/docker.sock` accessible.

**Tasks:**
1. Confirm socket access (`docker ps`).
2. Launch a container mounting host `/` and `chroot` into it.
3. Read `/etc/shadow` on the host (or drop a SUID bash) to prove host root.

**Success criteria:** demonstrable root on the *host*, not just the container, with the root cause (daemon runs as root) articulated.

---

## Exercise 2.8 — LXD Group Escalation

**Objective:** escalate via `lxd` group membership.

**Setup:** a user in the `lxd` group with a minimal image available locally.

**Tasks:**
1. Initialise a privileged container and attach the host filesystem as a device.
2. Exec into it and read/modify host root-owned files.

**Success criteria:** host filesystem fully readable/writable from the container, with the least-privilege lesson noted.

---

## Exercise 2.9 — NFS `no_root_squash`

**Objective:** chain Domain-1 NFS enumeration into root.

**Setup:** an NFS server exporting a directory with `no_root_squash`, mountable by your attack box.

**Tasks:**
1. Mount the export as root on your attack box.
2. Drop a root-owned SUID `bash` into the export.
3. On the target, execute it with `-p` for a root shell.

**Success criteria:** root on the target obtained purely via the NFS misconfiguration.

---

## Consolidation Challenge — LinPEAS Triage Drill

On a fresh lab box, run `linpeas.sh` once, then **close it** and escalate from your notes alone, manually verifying each promising finding. Afterwards, compare what you used against what LinPEAS flagged in red/yellow. The goal is to stop depending on the script's highlighting and build your own triage instinct — exactly what the exam demands when a box has several red herrings.
