# Certified Kubernetes Security Specialist (CKS) — Full-Length Performance Practice Exams

Welcome to the CKS Practice Exam Simulation Hub. These three full-length, 120-minute performance-based mock exams replicate the exact difficulty, control-plane hardening, supply-chain security, and runtime threat detection tasks of the live CNCF Certified Kubernetes Security Specialist exam.

---

## 1. Exam Blueprint & Scoring Matrix

| Parameter | Official CNCF CKS Standard | This Practice Suite |
| :--- | :--- | :--- |
| **Duration** | 120 Minutes (2 Hours) | 120 Minutes per Exam |
| **Number of Tasks** | 15–20 Performance Tasks | **16 Tasks per Exam (48 Total Tasks)** |
| **Passing Score** | **67%** | Target **80%+ (80/100 points)** before booking |
| **Prerequisite** | Active CKA Certification | Assumes full CKA cluster administration fluency |

### Domain Weight Breakdown

| Domain | Weight | Points per Exam | Core Security Controls Tested |
| :--- | :---: | :---: | :--- |
| **1. Cluster Setup** | **10%** | 10 pts | Default-deny NetworkPolicies, CIS `kube-bench` remediation, TLS Ingress (`tls.crt`/`tls.key`), GUI/Dashboard lockdown |
| **2. Cluster Hardening** | **15%** | 15 pts | Least-privilege RBAC, `automountServiceAccountToken: false`, `kube-apiserver` flags (`Node,RBAC`, `--anonymous-auth=false`), Kubernetes upgrades |
| **3. System Hardening** | **15%** | 15 pts | AppArmor (`aa-status`/`apparmor_parser`), Seccomp (`RuntimeDefault`/`Localhost`), Linux capabilities (`drop: ["ALL"]`), `ufw`, kernel module blacklisting |
| **4. Minimize Microservice Vulnerabilities** | **20%** | 20 pts | Pod Security Admission (`restricted`), `etcd` Encryption at Rest (`aescbc`), `RuntimeClass` (`gVisor`/`Kata`), OPA Gatekeeper / Kyverno / CEL, mTLS (`Istio`/`Cilium`) |
| **5. Supply Chain Security** | **20%** | 20 pts | Multi-stage Dockerfile hardening, `trivy image` CVE scanning, SPDX/CycloneDX SBOMs (`syft`/`bom`/`trivy`), `cosign` signing/verification, `ImagePolicyWebhook` |
| **6. Monitoring, Logging & Runtime Security** | **20%** | 20 pts | Kubernetes API Audit Policy (`Metadata`/`Request`/`RequestResponse`), **Falco** custom rules, **Tetragon** eBPF `TracingPolicy` (`Sigkill`), Immutable containers, live `/proc/<pid>` forensics |

---

## 2. Practice Exam Suite

| Exam | Focus Profile | Tasks | Time Limit | Target Score |
| :--- | :--- | :---: | :---: | :---: |
| **[Practice Exam 1: Control-Plane Hardening, Encryption & Runtime Defense](Practice_Exam_1.md)** | CIS `kube-bench` fixes, AppArmor enforcement, `etcd` `aescbc` encryption, `ImagePolicyWebhook`, Audit Policy configuration, Falco shell detection, and `trivy` CVE pod eviction | 16 Tasks | 120 mins | $\ge 80 / 100$ |
| **[Practice Exam 2: Zero-Trust Networking, Sandboxing & Supply Chain Security](Practice_Exam_2.md)** | Pod Security Standards (`restricted`), gVisor `RuntimeClass`, Cosign signature verification, SPDX SBOM generation, Tetragon eBPF `Sigkill` enforcement, and Istio `STRICT` mTLS | 16 Tasks | 120 mins | $\ge 80 / 100$ |
| **[Practice Exam 3: Live Breach Forensics, Policy-as-Code & Host Lockdown](Practice_Exam_3.md)** | Compromised container `/proc/<pid>/exe` forensics, OPA Gatekeeper & CEL `ValidatingAdmissionPolicy`, custom Seccomp profiles, `ufw` & kernel module blacklisting, and RBAC privilege de-escalation | 16 Tasks | 120 mins | $\ge 80 / 100$ |
| **[Practice Exam 4: Droplet VPC Threat Detection & Kernel Isolation](Practice_Exam_4.md)** | Falco interactive shell alerts, AppArmor deny-write profiles, Seccomp chmod/chown filtering, gVisor sandboxing, etcd encryption key rotation, and Ingress mTLS | 16 Tasks | 120 mins | $\ge 80 / 100$ |
| **[Practice Exam 5: Advanced Supply Chain Integrity & eBPF Policy Enforcement](Practice_Exam_5.md)** | Falco /etc/shadow alerts, Tetragon Sigkill tracing, Cosign SBOM verification, Cilium FQDN egress, Distroless container hardening, and RBAC aggregation | 16 Tasks | 120 mins | $\ge 80 / 100$ |

---

## 3. Critical CKS Survival Rules (Read Before Every Exam)

> [!CAUTION]
> **Rule #1: Always Backup `/etc/kubernetes/manifests/kube-apiserver.yaml` Outside `/etc/kubernetes/manifests/`!**
> Many CKS tasks require modifying `kube-apiserver.yaml` (Audit Policy, `EncryptionConfiguration`, `ImagePolicyWebhook`, RBAC flags, anonymous auth).
> ```bash
> sudo cp /etc/kubernetes/manifests/kube-apiserver.yaml /root/kube-apiserver.yaml.bak
> ```
> **Never** place the `.bak` file inside `/etc/kubernetes/manifests/`, or the `kubelet` will parse both files and crash the API server due to port `6443` conflicts.

1. **Double-Check `volumes` and `volumeMounts` on `kube-apiserver`:**
   Whenever you add `--audit-policy-file`, `--audit-log-path`, `--encryption-provider-config`, or `--admission-control-config-file` to `kube-apiserver.yaml`, verify that **every referenced host directory/file is mounted into the `kube-apiserver` container** via `volumeMounts` and `volumes`.
2. **Validate Falco & AppArmor Before Moving On:**
   - Run `falco --validate /etc/falco/falco_rules.local.yaml` before restarting `falco`.
   - Run `sudo aa-status | grep <profile-name>` after running `sudo apparmor_parser -q /etc/apparmor.d/<file>` to verify the exact internal `profile <name>` matches your Pod's `localhostProfile`.
3. **Wait for `kube-apiserver` Restart Recovery:**
   After saving changes to `/etc/kubernetes/manifests/kube-apiserver.yaml`, allow 20–40 seconds for `kubelet` to recreate the container. Monitor progress with:
   ```bash
   sudo crictl ps --name kube-apiserver
   ```
   If it does not appear within 45 seconds, inspect `/var/log/pods/kube-system_kube-apiserver-*/kube-apiserver/*.log` or `sudo journalctl -u kubelet -n 30`.

---

## 4. Personal Score Tracker

| Attempt Date | Exam # | Score (/100) | Time Used | Missed Tasks / Weak Domains |
| :--- | :---: | :---: | :---: | :--- |
| `YYYY-MM-DD` | Exam 1 | `___ / 100` | `___ mins` | |
| `YYYY-MM-DD` | Exam 2 | `___ / 100` | `___ mins` | |
| `YYYY-MM-DD` | Exam 3 | `___ / 100` | `___ mins` | |
