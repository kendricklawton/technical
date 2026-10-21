# Domain 4: Minimize Microservice Vulnerabilities — Essential Security Benchmarks & Technical Documentation Guide

> **Blueprint Weight:** 20% of Certified Kubernetes Security Specialist (CKS)  
> **Core Objective:** Enforce Pod Security Standards (PSS) via Pod Security Admission (PSA), configure container SecurityContexts, manage Secrets securely, and enable KMS-backed Secrets encryption at rest.

---

## 1. Authoritative Security Standards & Documentation

### 1.1 [Pod Security Standards (PSS) & Pod Security Admission (PSA)](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
* **Canonical Scope:** Built-in admission control replacing deprecated PodSecurityPolicies.
* **Three Security Levels:**
  * **Privileged:** Unrestricted permissions (intended for system agents, CNIs, storage drivers).
  * **Baseline:** Minimally restrictive; prevents known privilege escalations (disallows `hostNetwork`, `hostPID`, `hostIPC`, privileged mode).
  * **Restricted:** Hardened best practices (must run as non-root, read-only root filesystem, drop all capabilities except `NET_BIND_SERVICE`, require seccomp).
* **Namespace Labels:**
  * `pod-security.kubernetes.io/enforce: restricted`
  * `pod-security.kubernetes.io/enforce-version: latest`
  * `pod-security.kubernetes.io/warn: restricted`

### 1.2 [Hardening Container SecurityContext](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
* **Canonical Scope:** Defense-in-depth container process isolation.
* **Restricted SecurityContext Template:**
  ```yaml
  securityContext:
    runAsNonRoot: true
    runAsUser: 10001
    runAsGroup: 10001
    allowPrivilegeEscalation: false
    readOnlyRootFilesystem: true
    capabilities:
      drop:
      - ALL
  ```

### 1.3 [Encrypting Confidential Data at Rest (KMS v2)](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)
* **Canonical Scope:** Encrypting Kubernetes Secrets stored inside etcd.
* **EncryptionConfiguration (`/etc/kubernetes/enc/enc.yaml`):**
  ```yaml
  apiVersion: apiserver.config.k8s.io/v1
  kind: EncryptionConfiguration
  resources:
    - resources:
        - secrets
      providers:
        - kms:
            apiVersion: v2
            name: myKmsPlugin
            endpoint: unix:///var/run/kmsplugin.sock
        - aescbc:
            keys:
              - name: key1
                secret: <base64-encoded-secret>
        - identity: {} # Fallback for plaintext reads
  ```
* **API Server Configuration:** Add `--encryption-provider-config=/etc/kubernetes/enc/enc.yaml` and mount the file into the `kube-apiserver` static pod.
* **Re-encrypt Existing Secrets:** Run `kubectl get secrets --all-namespaces -o json | kubectl replace -f -`.

---

## 2. High-Frequency Exam Traps & Practical Reflexes

1. **`identity: {}` Placement in EncryptionConfiguration:**
   * When enabling encryption, the encryption provider (e.g. `kms` or `aescbc`) MUST be listed FIRST under `providers:`.
   * If `identity: {}` is listed first, newly created secrets will remain unencrypted!
2. **Re-encryption Verification:**
   * After enabling encryption, existing secrets in etcd remain unencrypted until rewritten. Run `kubectl get secrets --all-namespaces -o json | kubectl replace -f -`.
   * Verify by querying etcd directly via `etcdctl get /registry/secrets/...` and checking for `k8s:enc:` prefix.

---

## 3. Reading & Study Checklist

- [ ] Read *Pod Security Standards & Pod Security Admission*
- [ ] Read *Hardening Container SecurityContext*
- [ ] Read *Encrypting Confidential Data at Rest*
- [ ] Practice applying PSA namespace labels (`enforce`, `warn`, `audit`)
- [ ] Practice end-to-end etcd encryption configuration and secret re-encryption
