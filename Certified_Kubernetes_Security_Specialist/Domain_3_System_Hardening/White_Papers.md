# Domain 3: System Hardening — Essential Security Benchmarks & Technical Documentation Guide

> **Blueprint Weight:** 15% of Certified Kubernetes Security Specialist (CKS)  
> **Core Objective:** Harden Linux host operating systems, restrict container syscalls using Seccomp profiles, enforce mandatory access controls via AppArmor, and audit host attack surfaces.

---

## 1. Authoritative Security Standards & Documentation

### 1.1 [Restrict a Container's Syscalls with Seccomp](https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/seccomp/)
* **Canonical Scope:** Linux kernel Secure Computing Mode (seccomp) syscall filtering.
* **Seccomp Profile Configuration:**
  * Default profile directory: `/var/lib/kubelet/seccomp/`.
  * Setting default or custom profile in Pod `securityContext`:
    ```yaml
    securityContext:
      seccompProfile:
        type: RuntimeDefault
    ```
  * Custom localhost profile:
    ```yaml
    securityContext:
      seccompProfile:
        type: Localhost
        localhostProfile: profiles/audit.json
    ```

### 1.2 [Restrict Container Access with AppArmor](https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/apparmor/)
* **Canonical Scope:** Mandatory Access Control (MAC) restricting file access, networking capabilities, and process execution.
* **Workflow:**
  1. Profile file located at `/etc/apparmor.d/<profile-name>`.
  2. Load profile into Linux kernel: `apparmor_parser -q /etc/apparmor.d/<profile-name>`.
  3. Verify loaded profile: `aa-status`.
  4. Attach to Pod (Kubernetes 1.30+):
     ```yaml
     securityContext:
       appArmorProfile:
         type: Localhost
         localhostProfile: <profile-name>
     ```

### 1.3 [Linux Host Hardening & Kernel Security](https://www.cisecurity.org/benchmark/ubuntu_linux)
* **Canonical Scope:** Minimizing the host attack surface.
* **Key Practices:**
  * Disabling legacy network protocols (DCCP, SCTP, RDS).
  * Enforcing SSH key-only access with non-root logins.
  * Removing unneeded compilers and utilities (`gcc`, `curl`, `wget`) from production worker nodes.

---

## 2. High-Frequency Exam Traps & Practical Reflexes

1. **AppArmor Profile Loading Trap:**
   * Specifying an AppArmor profile in a Pod manifest when that profile has NOT been loaded into the kernel via `apparmor_parser` causes the Pod to remain in `Blocked` or `CreateContainerConfigError`.
   * *Resolution:* Run `apparmor_parser -r /etc/apparmor.d/<profile>` on all nodes where the pod could be scheduled.
2. **Seccomp Path Resolution:**
   * The `localhostProfile` path in `seccompProfile` is **strictly relative** to the kubelet seccomp root directory (`/var/lib/kubelet/seccomp/`). Writing an absolute path causes container launch failure.

---

## 3. Reading & Study Checklist

- [ ] Read *Restrict a Container's Syscalls with Seccomp*
- [ ] Read *Restrict Container Access with AppArmor*
- [ ] Practice loading and validating AppArmor profiles (`apparmor_parser`, `aa-status`)
- [ ] Practice configuring `RuntimeDefault` and `Localhost` seccomp profiles
- [ ] Review CIS Linux benchmark rules for kernel parameters (`sysctl`)
