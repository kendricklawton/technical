# Domain 3: System Hardening — Quiz 3 (Seccomp & AppArmor Mandatory Access Control)

**Exam:** Certified Kubernetes Security Specialist (CKS)  
**Format:** 10 Performance Scenario & Multiple Choice Security Questions with Explanations

---

### Question 1: AppArmor Profile Verification on Worker Nodes
A Pod fails to start with error `AppArmor profile k8s-deny-write not loaded`. Which command on the worker node verifies all currently loaded AppArmor profiles and their enforcement modes?
- A. `systemctl status apparmor`
- B. `aa-status` (or `apparmor_status`)
- C. `kubectl get apparmor`
- D. `crictl apparmor-list`

---

### Question 2: Loading a New AppArmor Profile into the Kernel (`apparmor_parser`)
You copied a custom AppArmor profile file to `/etc/apparmor.d/k8s-nginx-hardened` on `worker-1`. What command parses and loads the profile into the Linux kernel in **enforce** mode, and what flag would load it in **complain** mode?
- A. Enforce mode: `apparmor_parser -q /etc/apparmor.d/k8s-nginx-hardened` (or `-r` to reload); Complain mode: `apparmor_parser -C /etc/apparmor.d/k8s-nginx-hardened` (or `aa-complain`).
- B. `systemctl restart kubelet`
- C. `chmod +x /etc/apparmor.d/k8s-nginx-hardened`
- D. `modprobe apparmor`

---

### Question 3: AppArmor Profile Name vs File Path Trap
The file `/etc/apparmor.d/usr.sbin.nginx-custom` begins with the header:
```text
#include <tunables/global>
profile k8s-restricted-nginx flags=(attach_disconnected,mediate_deleted) {
  #include <abstractions/base>
  deny /etc/shadow r,
}
```
When binding this profile to a Kubernetes container, which string must be used as the `localhostProfile` name?
- A. `k8s-restricted-nginx` (the internal `profile <name>` identifier inside the file, NOT the filename `usr.sbin.nginx-custom`).
- B. `usr.sbin.nginx-custom`
- C. `/etc/apparmor.d/usr.sbin.nginx-custom`
- D. `default`

---

### Question 4: Modern `securityContext.appArmorProfile` Syntax (Kubernetes 1.30+ GA)
In Kubernetes 1.30+, AppArmor graduated to GA and moved from beta Pod annotations (`container.apparmor.security.beta.kubernetes.io/<container>`) to a native field in `securityContext`. What is the exact YAML structure to apply local profile `k8s-deny-write`?
- A.
  ```yaml
  securityContext:
    appArmorProfile:
      type: Localhost
      localhostProfile: k8s-deny-write
  ```
- B.
  ```yaml
  securityContext:
    appArmor: localhost/k8s-deny-write
  ```
- C.
  ```yaml
  securityContext:
    macProfile: k8s-deny-write
  ```
- D.
  ```yaml
  securityContext:
    seccompProfile:
      type: AppArmor
  ```

---

### Question 5: Seccomp Profile Types in Kubernetes (`Unconfined` vs `RuntimeDefault` vs `Localhost`)
By default, if `seccompProfile` is omitted on a Pod and `seccompDefault` is not enabled on the kubelet, what seccomp mode does Kubernetes apply, and how do you enable the container runtime's built-in default seccomp filter on a Pod?
- A. Default is `Unconfined` (zero syscall filtering); set `securityContext.seccompProfile.type: RuntimeDefault` to apply containerd/runc's default seccomp profile (which blocks ~44 dangerous syscalls like `kexec_load`, `reboot`, `mount`, `bpf`, `ptrace`).
- B. Default is `RuntimeDefault`.
- C. Default is `Localhost`.
- D. Default blocks all syscalls.

---

### Question 6: Defaulting All Pods on a Node to `RuntimeDefault` (`seccompDefault`)
How can a cluster administrator ensure that **every** Pod scheduled on a worker node uses `RuntimeDefault` seccomp automatically whenever the Pod spec omits `securityContext.seccompProfile`?
- A. Set `seccompDefault: true` in `/var/lib/kubelet/config.yaml` (or pass `--seccomp-default` to `kubelet`) and restart `kubelet`.
- B. Delete `/var/lib/kubelet/seccomp`.
- C. Set `privileged: false` in `etcd`.
- D. Disable `runc`.

---

### Question 7: Custom Seccomp Profile Root Directory on Worker Nodes
A Pod specifies:
```yaml
securityContext:
  seccompProfile:
    type: Localhost
    localhostProfile: profiles/audit.json
```
To which absolute filesystem path on the worker node does the kubelet resolve `profiles/audit.json`?
- A. `/var/lib/kubelet/seccomp/profiles/audit.json`
- B. `/etc/seccomp/profiles/audit.json`
- C. `/etc/kubernetes/seccomp/profiles/audit.json`
- D. `/usr/share/seccomp/profiles/audit.json`

---

### Question 8: Seccomp JSON Action Values (`SCMP_ACT_ERRNO`, `SCMP_ACT_LOG`, `SCMP_ACT_ALLOW`, `SCMP_ACT_KILL_THREAD`)
You want to write a custom seccomp profile `/var/lib/kubelet/seccomp/profiles/fine-grained.json` that allows all system calls by default **except** `chmod` and `fchmodat`, which must fail with `Operation not permitted` (`EPERM`). How is this structured in the JSON file?
- A. Set `"defaultAction": "SCMP_ACT_ALLOW"` and add an entry in `"syscalls"` with `"names": ["chmod", "fchmodat"]` and `"action": "SCMP_ACT_ERRNO"`.
- B. Set `"defaultAction": "SCMP_ACT_ERRNO"` and `"syscalls": []`.
- C. Set `"defaultAction": "SCMP_ACT_LOG"`.
- D. Use AppArmor syntax inside the JSON file.

---

### Question 9: Auditing All Syscalls Invoked by a Container Using `SCMP_ACT_LOG`
You are profiling a microservice to see which system calls it makes at runtime without blocking any of them. You apply a seccomp profile containing `{"defaultAction": "SCMP_ACT_LOG"}`. Where does the Linux kernel log the audited syscalls on the worker node?
- A. `/var/log/syslog` (or `/var/log/audit/audit.log` if `auditd` is running, and `dmesg` / `journalctl -k` with `type=SECCOMP` audit records containing the numeric `syscall=` ID).
- B. `kubectl logs <pod-name>`
- C. `/var/lib/kubelet/seccomp.log`
- D. `/etc/seccomp/audit.txt`

---

### Question 10: Translating Numeric Kernel Syscall IDs (`ausyscall` / `/usr/include/asm/unistd_64.h`)
In `/var/log/syslog`, a `type=SECCOMP` audit message reports `syscall=165` was blocked on an `x86_64` worker node. How can you quickly look up the human-readable system call name corresponding to number `165` on the Linux host?
- A. Run `ausyscall 165` or `grep -w 165 /usr/include/x86_64-linux-gnu/asm/unistd_64.h` (which reveals `#define __NR_mount 165`).
- B. Run `kubectl explain syscall 165`.
- C. Run `crictl syscall 165`.
- D. Check `/etc/services`.

---

## Answer Key & Detailed Explanations

1. **B** — `aa-status` lists all loaded AppArmor profiles categorized by enforce mode and complain mode, as well as confined PIDs.
2. **A** — `apparmor_parser` compiles and loads human-readable policy files from `/etc/apparmor.d/` into the kernel LSM.
3. **A** — Kubernetes references the profile by its declared `profile <name>` (or path if no `profile` keyword is used), not the host filename.
4. **A** — `securityContext.appArmorProfile` with `type: Localhost` (or `RuntimeDefault` / `Unconfined`) and `localhostProfile: <profile-name>` is the standard GA field.
5. **A** — Kubernetes defaults to `Unconfined` for backward compatibility unless `RuntimeDefault` or `Localhost` is specified.
6. **A** — `seccompDefault: true` in KubeletConfiguration changes the node's fallback seccomp profile from `Unconfined` to `RuntimeDefault`.
7. **A** — The kubelet resolves `localhostProfile` paths relative to `/var/lib/kubelet/seccomp/`.
8. **A** — `SCMP_ACT_ALLOW` as the default action combined with a `SCMP_ACT_ERRNO` rule for specific syscalls is a denylist profile; in high-security production environments, an allowlist (`"defaultAction": "SCMP_ACT_ERRNO"` with explicit `"SCMP_ACT_ALLOW"` syscalls) is preferred.
9. **A** — Kernel seccomp `SCMP_ACT_LOG` emits `audit` subsystem messages (`type=SECCOMP`) to `/var/log/syslog` or `/var/log/audit/audit.log`.
10. **A** — `unistd_64.h` (or `ausyscall`) maps `x86_64` syscall numbers to names.
