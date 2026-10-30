# Domain 2: Cluster Hardening — Essential Security Benchmarks & Technical Documentation Guide

> **Blueprint Weight:** 15% of Certified Kubernetes Security Specialist (CKS)  
> **Core Objective:** Restrict access to cloud provider metadata APIs, secure the Kubelet API, implement modern CEL-based ValidatingAdmissionPolicies, and manage secure ServiceAccount tokens.

---

## 1. Authoritative Security Standards & Documentation

### 1.1 [Restricting Cloud Metadata API Access](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
* **Canonical Scope:** Mitigating Server-Side Request Forgery (SSRF) and metadata credential theft.
* **Hardening Techniques:**
  * **NetworkPolicy Block:** Denying egress traffic to the link-local metadata address `169.254.169.254/32`.
    ```yaml
    egress:
    - to:
      - ipBlock:
          cidr: 0.0.0.0/0
          except:
          - 169.254.169.254/32
    ```
  * **Cloud IMDSv2 Enforcement:** Requiring hop-limit = 1 (preventing container bridged network traversal) and PUT session token negotiation.

### 1.2 [Hardening the Kubelet API](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)
* **Canonical Scope:** Securing port 10250 from unauthenticated remote execution.
* **Configuration (`/var/lib/kubelet/config.yaml`):**
  ```yaml
  authentication:
    anonymous:
      enabled: false
    webhook:
      enabled: true
  authorization:
    mode: Webhook
  readOnlyPort: 0 # Disable unauthenticated metrics port 10255
  ```

### 1.3 [Kubernetes ValidatingAdmissionPolicy (CEL-Based)](https://kubernetes.io/docs/reference/access-authn-authz/validating-admission-policy/)
* **Canonical Scope:** Declarative, in-tree admission control using Common Expression Language (CEL) without external webhook web services.
* **Core Components:**
  * `ValidatingAdmissionPolicy`: Defines the validation rule (e.g., `object.spec.replicas <= 5`).
  * `ValidatingAdmissionPolicyBinding`: Binds the policy to specific namespaces or resources with enforcement actions (`Deny` or `Warn`).

### 1.4 [ServiceAccount Token Projection & AutoMount](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
* **Canonical Scope:** Bound ServiceAccount tokens with audience restriction and time-to-live expiration.
* **Best Practice:** Always set `automountServiceAccountToken: false` on pods and ServiceAccounts that do not interact with the Kubernetes API.

---

## 2. High-Frequency Exam Traps & Practical Reflexes

1. **Kubelet Port 10250 Unauthenticated Exploit:**
   * By default, kubelet may allow anonymous curl requests to `https://<node-ip>:10250/run/<ns>/<pod>/<container>`.
   * *Resolution:* Set `anonymous.enabled: false` and `authorization.mode: Webhook`, then restart kubelet.
2. **ValidatingAdmissionPolicy vs Webhook:**
   * CEL policies run entirely in-process within `kube-apiserver`, eliminating network timeouts and admission controller crashes that take down the control plane.
3. **Restricting API Server Profiling:**
   * Flag `--profiling=false` prevents malicious actors from extracting sensitive memory artifacts via `/debug/pprof/`.

---

## 3. Reading & Study Checklist

- [ ] Read *Restricting Cloud Metadata API Access*
- [ ] Read *Hardening the Kubelet API*
- [ ] Read *ValidatingAdmissionPolicy Documentation & CEL Expressions*
- [ ] Practice disabling `automountServiceAccountToken` across manifests
- [ ] Configure egress NetworkPolicy blocking `169.254.169.254/32`
