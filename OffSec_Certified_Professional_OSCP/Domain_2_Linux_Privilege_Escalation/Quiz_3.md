# Domain 2 — Quiz 3: Containers, NFS & Kernel Escalation

> Scenario-driven, exam-difficulty. Answer before expanding.

---

**Q1.** Inside a container you find `/var/run/docker.sock` mounted and the `docker` CLI available. What is the most direct route to host root?

- A. Launch a container that bind-mounts host `/` and chroot into it to act as host root.
- B. Restart the Docker daemon from inside the container to inherit its root context.
- C. Edit `/etc/passwd` inside the container to add a UID-0 user on the host.
- D. Read the Docker socket file contents to extract the host root password hash.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** Access to `docker.sock` is control of the root-owned daemon. `docker run -v /:/host -it alpine chroot /host sh` mounts the host filesystem and gives a root shell on the host.

- **B** is wrong: you cannot "restart the daemon to inherit root"; the leverage is issuing API calls via the socket, which A does.
- **C** is wrong: editing the *container's* `/etc/passwd` affects the container, not the host.
- **D** is wrong: the socket is an API endpoint, not a file containing password hashes.
</details>

---

**Q2.** You are root inside a `--privileged` container. Which statement explains why a host escape is possible?

- A. A privileged container keeps all capabilities and host device access, so host-level mechanisms like cgroup release_agent or raw disks are reachable.
- B. A privileged container runs a separate kernel, so compromising it automatically compromises the host kernel too.
- C. A privileged container disables namespaces, so the container and host share the same PID 1 process directly.
- D. A privileged container mounts the host root filesystem at `/` by default, so no further action is needed.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** `--privileged` grants all capabilities and access to host devices, enabling escapes such as the cgroup-v1 `release_agent` trick or mounting the raw host disk (`/dev/sda*`).

- **B** is wrong: containers share the *host* kernel; there is no separate kernel.
- **C** is wrong: privileged mode relaxes capabilities/devices but does not merge PID namespaces so that container and host share PID 1.
- **D** is wrong: the host root is not auto-mounted at `/`; you must mount a device or abuse a mechanism.
</details>

---

**Q3.** Your user is in the `lxd` group but not `sudo`. Why is this effectively root, and what is the escalation shape?

- A. The group lets you launch a privileged container with the host disk attached, giving root access to host files.
- B. The group grants passwordless sudo implicitly, so `sudo -i` returns a root shell.
- C. The group allows editing `/etc/sudoers`, so you add your user and escalate.
- D. The group permits loading kernel modules, so you insert a module that grants UID 0.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** `lxd` membership lets you create a privileged container and attach the host filesystem as a device, then read/write host root-owned files — documented root-equivalence.

- **B** is wrong: `lxd` membership does not grant sudo.
- **C** is wrong: it does not grant write access to `/etc/sudoers`.
- **D** is wrong: the escalation is via container+host-mount, not kernel module loading.
</details>

---

**Q4.** Domain-1 enumeration found an NFS export with `no_root_squash`, mountable from your Kali box. Which sequence produces root on the *target*?

- A. Mount the export as root on Kali, drop a SUID-root bash into it, then execute that bash with `-p` on the target.
- B. Mount the export on the target as your low-priv user, then chmod bash to SUID from there.
- C. Export your own filesystem back to the target so it trusts your root user over NFS.
- D. Copy `/etc/shadow` from the export and crack it, since no_root_squash exposes password hashes.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** `no_root_squash` trusts client root, so a root-owned SUID bash you create on Kali (as root, into the mounted export) appears SUID-root on the target; running it `-p` gives root there.

- **B** is wrong: as a low-priv user on the target you cannot create a *root-owned* SUID file; the trick needs client-side root via the Kali mount.
- **C** is wrong: re-exporting your filesystem does not make the target trust you; the exploit uses the *target's* export settings.
- **D** is wrong: the export is a shared directory, not necessarily `/etc/shadow`, and the whole point is write-as-root, not hash theft.
</details>

---

**Q5.** A kernel-exploit privesc (e.g. a Dirty-style local exploit) is a *last resort* on the OSCP. Why?

- A. Kernel exploits can crash the target, and a hung box wastes exam time and may lose your foothold.
- B. Kernel exploits are strictly forbidden by the exam rules under all circumstances.
- C. Kernel exploits require Metasploit, which is limited to a single machine on the exam.
- D. Kernel exploits only work on Windows targets, so they never apply to Linux boxes.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** Kernel exploits are often unstable and can panic the host; a crashed target is unrecoverable without a revert and burns time. Exhaust configuration-based escalations first.

- **B** is wrong: kernel exploits are not banned (automated *frameworks* like autopwn are); a manual local exploit is allowed.
- **C** is wrong: kernel exploits do not require Metasploit; many are standalone C.
- **D** is wrong: kernel exploits very much apply to Linux; the claim is false.
</details>

---

**Q6.** Before running a compiled local kernel exploit on a target, which precaution best reflects professional practice?

- A. Confirm the exact kernel version and architecture, match the exploit to it, and prefer a revert-able target.
- B. Run every kernel exploit in your collection in sequence until one happens to work.
- C. Disable the target's logging and monitoring first so the exploit is not recorded.
- D. Recompile the kernel on the target with debugging symbols to improve reliability.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** Match `uname -r`/architecture precisely to a known-good exploit; mismatched exploits crash boxes. Having a revert available (lab/exam) is the safety net.

- **B** is wrong: spraying random exploits reliably crashes the host and wastes time.
- **C** is wrong: tampering with logging to hide activity is out of scope for an authorised test.
- **D** is wrong: recompiling the kernel is absurd on a target and would likely break it.
</details>

---

**Q7.** You escaped a container to the host. For the report, which root-cause/remediation pairing is correct?

- A. The container was over-privileged (mounted socket / privileged flag); apply least privilege and never expose docker.sock to untrusted containers.
- B. The container image was outdated; rebuilding from the latest base image prevents all escapes.
- C. The host firewall allowed container traffic; blocking egress from containers stops escapes.
- D. The container lacked a resource limit; setting CPU and memory quotas prevents host access.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** The escape stemmed from excess privilege (socket exposure or `--privileged`). The fix is least privilege: drop capabilities, avoid `--privileged`, and never mount the Docker socket into untrusted containers.

- **B** is wrong: image freshness does not address a privileged/socket-exposed configuration.
- **C** is wrong: egress filtering does not stop a host-filesystem escape.
- **D** is wrong: CPU/memory quotas are resource controls, not isolation boundaries against escape.
</details>

---

**Q8.** Across this domain, which single habit most reliably prevents you from missing a Linux privesc on the exam?

- A. Enumerate systematically (sudo, SUID, caps, cron, writable files, internal ports) and manually verify every automated finding.
- B. Run a kernel exploit first, since kernel bugs are the most common escalation path.
- C. Attempt password reuse exclusively, since credentials always escalate faster than misconfigurations.
- D. Focus only on the findings LinPEAS marks red, ignoring yellow and uncoloured output.

<details><summary>Show Answer & Explanation</summary>

**Correct: A.** Breadth-first, systematic enumeration with manual verification catches the one misconfiguration that matters and lets you reproduce it for the report.

- **B** is wrong: kernel exploits are a risky last resort, not a first move.
- **C** is wrong: reuse is worth trying but is not a substitute for enumeration; many boxes escalate via config, not creds.
- **D** is wrong: LinPEAS highlighting is imperfect; real findings often sit in yellow/uncoloured output, so you must read all of it.
</details>
