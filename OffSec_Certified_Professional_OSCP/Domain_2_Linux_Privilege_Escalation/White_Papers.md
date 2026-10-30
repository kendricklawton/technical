# Domain 2 — Reference White Papers & Deep Dives

> Mechanism-level background for Linux privilege escalation. For each topic, aim to explain the *trust assumption that breaks*.

---

## 1. The Unix Privilege Model: UID, EUID, SUID

- A process has a **real UID** (who launched it) and an **effective UID** (whose privileges it acts with). The SUID bit sets EUID to the file *owner* on exec — so a root-owned SUID binary runs with EUID 0 for any caller.
- `bash` drops privileges by default when EUID ≠ RUID; `bash -p` preserves the elevated EUID, which is why escalation payloads use it.
- The design intent (let `passwd` edit `/etc/shadow` safely) becomes a vulnerability whenever a SUID binary exposes a shell escape, a relative command, or a writable library.

**Sources:** `credentials(7)`, `execve(2)`, `setuid(2)` man pages; Aleph One-era Unix security literature.

---

## 2. The Dynamic Linker and `LD_PRELOAD`

- `ld.so` resolves shared libraries at load time. `LD_PRELOAD` forces a library to load first, letting its symbols (and `_init`/constructor) run before the program's own code.
- For security, the loader **ignores** `LD_PRELOAD` for SUID binaries *unless* the environment is preserved — which is exactly what `sudo`'s `env_keep += LD_PRELOAD` does, re-opening the door.

**Sources:** `ld.so(8)`, `dlopen(3)`; `sudoers(5)` on `env_keep`/`env_reset`.

---

## 3. POSIX Capabilities

- Capabilities partition root into ~40 units (`CAP_SETUID`, `CAP_DAC_READ_SEARCH`, `CAP_NET_ADMIN`, …). Attached to a file via extended attributes (`setcap`), a binary wields only those powers — the least-privilege alternative to SUID-root.
- The abuse: a capability sufficient to *become* root (`CAP_SETUID`) or to *read any file* (`CAP_DAC_READ_SEARCH`) on an interpreter (python/perl/tar) is effectively full compromise, because the interpreter can then do anything.

**Sources:** `capabilities(7)`, `getcap(8)`/`setcap(8)`; Linux kernel capability documentation.

---

## 4. Cron, `PATH`, and Shell Glob Expansion

- Cron runs jobs with a minimal, often predictable `PATH`. A job invoking a bare command name resolves it through that `PATH`; if an attacker-writable directory precedes the real one, the attacker's binary wins.
- **Wildcard injection** exploits that the *shell* expands `*` into a sorted argument list **before** the target program sees it. A file literally named `--checkpoint-action=exec=sh x.sh` becomes a command-line option to `tar`. The program cannot distinguish an attacker filename from a legitimate flag.

**Sources:** `crontab(5)`, `glob(7)`; DefenseCode "Unix wildcard" advisory (the canonical tar/chown/rsync write-up).

---

## 5. Container Isolation and Its Failure Modes

- Containers are **not** VMs: they share the host kernel, isolated by namespaces (PID, mount, net, user) and limited by cgroups and capabilities. Weaken any of those and the boundary leaks.
- **Docker socket:** the daemon runs as root; anyone who can talk to `docker.sock` can request a container that mounts host `/` — full host root by design of the API.
- **`--privileged`:** disables most isolation (all caps, device access). The `release_agent` cgroup-v1 escape and raw host-disk mounts both exploit that the container can reach host-level mechanisms.
- **`docker`/`lxd` groups:** membership is a documented root-equivalent; vendors explicitly warn that adding a user to these groups grants root.

**Sources:** Docker security docs; `namespaces(7)`, `cgroups(7)`; Trail of Bits and NCC Group container-escape research; `release_agent` escape write-ups.

---

## 6. NFS Squashing

- NFSv3 authorises by the UID the client asserts. `root_squash` (default) remaps client UID 0 to `nobody`; `no_root_squash` trusts client root as server root — so a client-side root creates server-side root-owned files, including SUID binaries.

**Sources:** `exports(5)`; RFC 1813 (NFSv3).

---

## 7. Primary Reference Index

- **GTFOBins** (gtfobins.github.io) — the canonical sudo/SUID/capability abuse reference; learn to read *why* each entry works.
- **HackTricks — Linux Privilege Escalation** — broad checklist (verify, don't paste).
- **PEASS-ng / LinPEAS** source — understand what each check detects.
- **Linux man pages** — `capabilities(7)`, `exports(5)`, `sudoers(5)` are primary sources worth reading directly.

> **Study tip:** keep a personal "misconfig → mechanism → fix" table in your notes. On the exam you will recognise the misconfig faster if you have already written the mechanism in your own words.
