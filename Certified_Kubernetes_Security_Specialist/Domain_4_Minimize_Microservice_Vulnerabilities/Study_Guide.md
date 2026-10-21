# Domain 4: Minimize Microservice Vulnerabilities — In-Depth Master Study Guide

**Exam Weight:** 20% of CKS  
**Target Mastery:** Pod Security Standards (PSS) & Admission (PSA), etcd Secret Encryption at Rest (`EncryptionConfiguration`), Container Sandboxing (gVisor `runsc` & Kata Containers), SecurityContext (Linux Capabilities, read-only rootfs, privilege escalation), OPA Gatekeeper & Kyverno, and Service Mesh Pod-to-Pod mTLS.

---

## 1. Pod Security Admission (PSA) & Pod Security Standards (PSS)

Kubernetes replaces legacy PodSecurityPolicies (PSP) with the native **Pod Security Admission (PSA)** controller.

### The 3 Pod Security Standards Levels

| PSS Profile | Description | Allowed Controls & Prohibitions |
|---|---|---|
| **Privileged** | Unrestricted policy | Allows all privilege escalation, root containers, host namespaces (`hostPID`, `hostNetwork`, `hostIPC`), and all Linux capabilities. Intended only for system components (CNI, kube-proxy). |
| **Baseline** | Minimally restrictive | Prevents known privilege escalations. Blocks `hostNetwork`, `hostPID`, `hostIPC`, `hostPath` volumes (except approved types), and adding capabilities beyond standard default set. |
| **Restricted** | Heavily hardened | Strictly enforces: `runAsNonRoot: true`, `readOnlyRootFilesystem: true`, `allowPrivilegeEscalation: false`, `capabilities: drop: ["ALL"]`, `seccompProfile: RuntimeDefault` or `Localhost`. |

### The 3 Modes of PSA Enforcement
Labels applied at the Namespace level:
1. **`enforce`**: Violations cause pod admission rejection (hard block).
2. **`audit`**: Violations are allowed, but an audit log event is recorded with an audit annotation.
3. **`warn`**: Violations trigger a warning message returned directly to the user in their terminal during `kubectl apply`.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: payment-processing
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: latest
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/audit-version: latest
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/warn-version: latest
```

### Complete Restricted-Compliant Pod Spec
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: compliant-microservice
  namespace: payment-processing
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 10001
    runAsGroup: 10001
    fsGroup: 10001
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: cgr.dev/chainguard/static:latest
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
    volumeMounts:
    - name: writable-tmp
      mountPath: /tmp
  volumes:
  - name: writable-tmp
    emptyDir: {}
```

---

## 2. Secrets Management & etcd Encryption at Rest

By default, Kubernetes Secrets stored in etcd are plaintext base64! Anyone with physical or API access to etcd can extract all database passwords and tokens.

```mermaid
flowchart TD
    API["kube-apiserver"] -->|Writes Secret| EncProv{"Encryption Provider"}
    EncProv -->|AES-CBC / KMS / Secretbox| Encrypted["Ciphertext in etcd<br/>k8s:enc:aescbc:v1:key1:..."]
    EncProv -->|identity (plaintext)| Plaintext["Plaintext base64 in etcd (Insecure)"]
```

### Complete `EncryptionConfiguration` Schema (`/etc/kubernetes/enc/enc.yaml`)
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: c2VjcmV0IGlzIDE2IGJ5dGVzIQ==
      - identity: {}
```

### The 4 Mandatory Steps for CKS Exam Day:
1. **Generate 32-byte Base64 Key:**
   ```bash
   head -c 32 /dev/urandom | base64
   ```
2. **Create the file on Control Plane:** `/etc/kubernetes/enc/enc.yaml`.
3. **Update `/etc/kubernetes/manifests/kube-apiserver.yaml`:**
   - Add parameter: `--encryption-provider-config=/etc/kubernetes/enc/enc.yaml`
   - Add hostPath volume for `/etc/kubernetes/enc` and matching `volumeMount`.
4. **Re-Encrypt All Existing Secrets (Do not forget this!):**
   ```bash
   kubectl get secrets -A -o json | kubectl replace -f -
   ```
5. **Verify Direct etcd Ciphertext:**
   ```bash
   ETCDCTL_API=3 etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt      --cert=/etc/kubernetes/pki/etcd/server.crt      --key=/etc/kubernetes/pki/etcd/server.key      get /registry/secrets/default/my-secret | hexdump -C
   ```
   *(Must show `k8s:enc:aescbc:v1:key1` prefix instead of raw base64 data).*

---

## 3. Container Sandboxing: gVisor (`runsc`) & Kata Containers

Standard containers share the host Linux kernel. A kernel exploit (privilege escalation, kernel vulnerability) allows a compromised container to compromise the entire host node.

### gVisor Architecture
- **gVisor (`runsc`)**: User-space kernel written in Go that intercepts all application system calls (Syscall virtualization). The container never communicates directly with the host Linux kernel!

```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: gvisor
handler: runsc
---
apiVersion: v1
kind: Pod
metadata:
  name: untrusted-sandboxed-workload
spec:
  runtimeClassName: gvisor
  containers:
  - name: untrusted-code-runner
    image: python:3.11-alpine
    command: ["python", "-c", "import os; print(os.uname())"]
```

Verify isolation: Inside a gVisor container, `dmesg` returns empty or virtualized messages, and `uname -r` returns the virtualized gVisor kernel version (`4.4.0-runsc`).
