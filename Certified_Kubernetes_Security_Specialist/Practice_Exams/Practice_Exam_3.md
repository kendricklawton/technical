# CKS Full-Length Performance Practice Exam 3

**Time Limit:** 120 Minutes  
**Passing Score:** 67% (Score Target: $\ge 85\%$)  
**Topology:** Multi-Cluster Simulation (`controlplane`, `node01`)  
**Format:** 16 Hard/Challenge Mode Performance Tasks Featuring Broken Control-Plane Recovery, Keyless Cosign Verification, In-Memory Vault Secrets, and Forensic Incident Drills

---

### Task 1: Broken Control Plane Recovery after Malformed Kubelet Config (Weight: 7%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** A worker node was rebooted and is stuck in `NotReady`. Control plane static pods are failing. Triage systemd logs (`journalctl -u kubelet -e`), repair the cgroup driver mismatch (`cgroupDriver: systemd`) in `/var/lib/kubelet/config.yaml`, and restore node readiness.

---

### Task 2: Advanced etcd Snapshot Restoration with Corrupted Database (Weight: 7%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** The etcd cluster database is corrupted. Restore from the verified snapshot at `/opt/etcd-backup.db` into a new data directory `/var/lib/etcd-restored`. Update `/etc/kubernetes/manifests/etcd.yaml` volume mounts and verify all deployments recover cleanly.

---

### Task 3: Zero Trust Namespace Isolation with Multi-Selector NetPols (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Write a NetworkPolicy in namespace `internal` that allows ingress traffic ONLY from pods labeled `tier: frontend` located inside namespaces labeled `environment: production`, while blocking all other ingress and external egress.

**Requirements:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: strict-production-only
  namespace: internal
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          environment: production
      podSelector:
        matchLabels:
          tier: frontend
```

---

### Task 4: In-Memory Secrets Management with External Secrets Operator (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Configure a `SecretStore` and `ExternalSecret` resource connecting to an external secret provider to synchronize production database credentials into a native Kubernetes Secret without committing secrets to Git.

---

### Task 5: ImagePolicyWebhook Admission Configuration (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Configure `kube-apiserver` to enable the `ImagePolicyWebhook` admission plugin using configuration `/etc/kubernetes/admission/admission-config.yaml`.

---

### Task 6: Audit Pod Capabilities & Drop Insecure Capabilities (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Audit namespace `utilities`. Any pod retaining `CAP_SYS_ADMIN` or `CAP_NET_ADMIN` must have those capabilities dropped, retaining only `NET_BIND_SERVICE`.

---

### Task 7: Kube-Bench Automation as a CronJob (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Deploy `kube-bench` as a daily Kubernetes `CronJob` running at midnight on all worker nodes, outputting JSON findings to a shared storage volume.

---

### Task 8: Restrict Direct API Server Port Access via NetworkPolicy (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Prevent general worker node pods from querying the Kubernetes API Server on port 6443 directly, unless specifically labeled `role: api-consumer`.

---

### Task 9: Detect In-Container Reverse Shells with Falco (Weight: 6%)
**Context:** `ssh node01`  
**Scenario:** Add a Falco rule to detect outbound network connections initiated by common shells (`bash`, `sh`, `zsh`) to external IP addresses. Set alert priority to `CRITICAL`.

---

### Task 10: Kyverno Policy for Disallowing Root Users (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Deploy a Kyverno `ClusterPolicy` named `disallow-root-execution` that validates that all pods enforce `runAsNonRoot: true` in their pod or container securityContext.

---

### Task 11: Linux PAM & User Account Hardening on Worker Nodes (Weight: 6%)
**Context:** `ssh node01`  
**Scenario:** Audit `/etc/passwd` on `node01`. Lock all expired or inactive developer accounts (`usermod -L`) and set the default shell to `/usr/sbin/nologin`.

---

### Task 12: In-Cluster TLS Certificate Expiration Audit (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Run `kubeadm certs check-expiration`. Identify any certificate expiring within 30 days and execute `kubeadm certs renew apiserver` to rotate the certificate without dropping active API clients.

---

### Task 13: Container Security: Read-Only Root Filesystem with Writable Volatile Storage (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Deploy a containerized microservice that requires a read-only root filesystem, but needs to write logs to `/var/log` and cache to `/tmp`. Mount dedicated in-memory `emptyDir` volumes with medium `Memory` and size limits.

---

### Task 14: Restrict Access to Kubelet Credentials (Weight: 6%)
**Context:** `ssh node01`  
**Scenario:** Verify and enforce that `/var/lib/kubelet/pki/kubelet-client-current.pem` has file permissions `600` and is owned exclusively by `root:root`.

---

### Task 15: Cosign Keyless OIDC Verification (Weight: 7%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Verify an image signed with Cosign keyless signatures using an OIDC issuer (`https://token.actions.githubusercontent.com`) and identity (`https://github.com/myorg/myrepo/.github/workflows/build.yml@refs/heads/main`).

---

### Task 16: Live Incident Forensics: Memory Dump & Disk Preservation (Weight: 7%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** An active rootkit is suspected on pod `suspicious-worker`. Execute non-destructive live forensics: isolate the pod, capture running processes via `/proc/<pid>/status`, archive `/proc/<pid>/root`, and save container state before termination.
