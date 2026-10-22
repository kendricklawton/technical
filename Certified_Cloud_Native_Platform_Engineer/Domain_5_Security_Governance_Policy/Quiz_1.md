# Domain 5: Security, Governance & Policy Enforcement — Quiz 1

**Exam:** Certified Cloud Native Platform Engineer (CNPE)  
**Format:** 10 Hands-On Scenario & Performance Questions with Detailed Rationales

---

### Question 1: Admission Webhook Failure Policy Deadlock
A platform engineer installs a custom ValidatingAdmissionWebhook with `failurePolicy: Fail` targeting all namespaces including `kube-system`. When the webhook pod crashes due to a node network partition, the cluster administrator can no longer deploy any pods, update configurations, or even restart the webhook pod. What misconfiguration caused this control plane lockout?
- A. The API server ran out of disk space.
- B. The webhook intercepted its own namespace and system namespaces without an exclusion rule (`namespaceSelector` ignoring `kube-system` and webhook pods), combined with `failurePolicy: Fail`.
- C. Kyverno does not support Kubernetes v1.31.
- D. The worker nodes were not tainted.

---

### Question 2: Kyverno Validation vs Mutation vs Generation
Which Kyverno rule type is used to automatically create a default `NetworkPolicy` and default `ResourceQuota` inside any newly created namespace?
- A. `validate`
- B. `mutate`
- C. `generate`
- D. `verifyImages`

---

### Question 3: Pod Security Standards Restricted Profile Requirements
Under the Kubernetes Pod Security Standard `Restricted` profile, which configuration is mandatory in a pod container's `securityContext`?
- A. `privileged: true`
- B. `runAsNonRoot: true` and `allowPrivilegeEscalation: false`
- C. `capabilities.add: ["SYS_ADMIN"]`
- D. `hostNetwork: true`

---

### Question 4: External Secrets Operator (ESO) Creation Policy
In an ExternalSecret resource, setting `spec.target.creationPolicy: Owner` ensures what operational lifecycle behavior?
- A. The secret can only be read by the root user.
- B. The generated Kubernetes Secret is owned by the ExternalSecret CRD; deleting the ExternalSecret automatically garbage collects the underlying Secret.
- C. The secret is saved directly to Git.
- D. The cloud provider deletes the upstream Vault secret.

---

### Question 5: Sigstore / Cosign Keyless Signing Mechanics
How does Cosign perform "keyless" container image signing without requiring developers or CI systems to store long-lived private encryption keys?
- A. By generating an unencrypted symmetric key.
- B. By obtaining an ephemeral x509 certificate from the Fulcio CA based on an OIDC identity token, signing the image digest, and recording the signed entry in the Rekor transparency log.
- C. By hashing the Dockerfile with MD5.
- D. By storing keys in the Linux kernel keyring.

---

### Question 6: OPA Gatekeeper ConstraintTemplate Language
In what language are OPA Gatekeeper `ConstraintTemplate` validation policies written?
- A. Python
- B. Go
- C. Rego
- D. Rust

---

### Question 7: PSA Namespace Enforcement Labels
To enforce the `restricted` Pod Security Standard on a namespace while warning developers when resources violate the latest version, which namespace labels must be applied?
- A. `pod-security.kubernetes.io/enforce: restricted` and `pod-security.kubernetes.io/warn: restricted`
- B. `security.istio.io/enforce: true`
- C. `rbac.authorization.k8s.io/policy: restricted`
- D. `networkpolicy.cilium.io/mode: strict`

---

### Question 8: Kyverno PolicyException CRD Benefit
Why is utilizing a `PolicyException` custom resource superior to hardcoding namespace or pod exclusions directly within a Kyverno `ClusterPolicy` rule?
- A. It bypasses git approvals.
- B. It decouples the core security policy from individual workload exemptions, allowing platform teams to audit, track, and apply expiration times to exceptions independently.
- C. PolicyException makes pods run faster.
- D. It allows pods to run without Docker.

---

### Question 9: External Secrets Refresh Interval
An ExternalSecret has `refreshInterval: "1h"`. A developer rotates a database password directly in HashiCorp Vault. Under normal operation without manual intervention, when will the Kubernetes Secret be updated?
- A. Instantly in 0 milliseconds.
- B. Within 1 hour (when ESO performs its next reconciliation cycle).
- C. Only after the Kubernetes cluster is rebooted.
- D. Never, because ESO does not support HashiCorp Vault.

---

### Question 10: Cilium DNS-Based Egress Filtering
Why is DNS-based egress filtering with `CiliumNetworkPolicy` (`toFQDNs`) more secure and maintainable than IP CIDR egress filtering (`toCIDR`) for external cloud services (e.g. `api.github.com` or `api.stripe.com`)?
- A. IP addresses for modern cloud services change dynamically across thousands of CDNs and anycast IPs, making static CIDR whitelisting fragile and prone to breakage.
- B. CIDR blocks are deprecated in IPv6.
- C. Cilium cannot filter IP addresses.
- D. DNS filtering disables TLS encryption.

---

## Answer Key & Rationales

1. **B** — A webhook with `failurePolicy: Fail` that intercepts all namespaces will block admission requests if the webhook itself is down, causing a complete cluster deadlock unless system namespaces are bypassed via `namespaceSelector`.
2. **C** — Kyverno `generate` rules create auxiliary Kubernetes resources (such as NetworkPolicies, Roles, Quotas, or ConfigMaps) in response to trigger events like namespace creation.
3. **B** — The PSS `Restricted` profile strictly mandates `runAsNonRoot: true`, `allowPrivilegeEscalation: false`, dropping `ALL` capabilities, and disallowing host namespaces.
4. **B** — `creationPolicy: Owner` sets Kubernetes ownerReferences on the synced Secret, ensuring clean automated cascading deletion when the ExternalSecret is removed.
5. **B** — Sigstore keyless signing combines OIDC tokens (e.g., from GitHub Actions), short-lived certificates from Fulcio CA, and tamper-evident append-only logging in Rekor.
6. **C** — OPA and Gatekeeper use Rego, a declarative query language designed specifically for evaluating complex hierarchical JSON data structures.
7. **A** — Kubernetes PSA uses standard labels: `pod-security.kubernetes.io/enforce` (blocks non-compliant pods) and `pod-security.kubernetes.io/warn` (outputs CLI warnings).
8. **B** — The `PolicyException` CRD provides an auditable, separate resource for handling temporary or approved security exemptions without polluting production policy definitions.
9. **B** — ESO re-fetches external secret data from the remote provider at every `refreshInterval` cadence (e.g., 1 hour) and updates the in-cluster Secret.
10. **A** — Modern SaaS and cloud APIs utilize dynamic IP pools and CDNs; `toFQDNs` policies allow Cilium's eBPF datapath to track DNS lookups and dynamically permit only valid resolved IPs.
