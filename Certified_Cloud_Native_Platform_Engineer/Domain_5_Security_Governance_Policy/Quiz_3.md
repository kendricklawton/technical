# Domain 5: Security, Governance & Policy Enforcement — Quiz 3

**Exam:** Certified Cloud Native Platform Engineer (CNPE)  
**Format:** 10 Hands-On Scenario & Performance Questions with Detailed Rationales

---

### Question 1: Admission Webhook WebhookTimeoutSeconds
An admission webhook is taking 12 seconds to respond due to external database lookups. The API server has a client timeout of 10 seconds. What will happen to developer requests applying new pods if the webhook has `failurePolicy: Fail`?
- A. The API server will abort the request after the webhook timeout expires, returning an HTTP 500/Internal Server Error and blocking pod admission.
- B. The requests will pass automatically.
- C. The pods will be scheduled without validation.
- D. The nodes will be rebooted.

---

### Question 2: Pod Security Admission Enforcement Version
Why is it best practice to specify `pod-security.kubernetes.io/enforce-version: v1.31` (a pinned version) rather than `latest` in production namespace labels?
- A. `latest` is not a valid Kubernetes string.
- B. Pinning the version prevents a future Kubernetes cluster upgrade from automatically applying new, stricter rules from a newer PSS version that could unexpectedly break existing running workloads.
- C. Pinned versions disable audit logging.
- D. Pinned versions run faster.

---

### Question 3: ESO PushSecret CRD
While `ExternalSecret` pulls credentials from an external store into Kubernetes, which External Secrets Operator CRD allows pushing a Kubernetes Secret out to an external secret store (e.g. AWS Secrets Manager or Vault)?
- A. `PushSecret`
- B. `ClusterSecretStore`
- C. `SecretUploader`
- D. `ExportSecret`

---

### Question 4: Sigstore Rekor Transparency Log Purpose
What security guarantee does the Rekor transparency log provide in the Sigstore ecosystem?
- A. It provides an immutable, tamper-evident, append-only public ledger proving that a specific artifact signature existed at a specific point in time.
- B. It hosts container images for Docker.
- C. It stores the full container filesystem layers.
- D. It encrypts Git commit messages.

---

### Question 5: Kyverno Generating Sync Property
When configuring a Kyverno `generate` rule with `sync: true`, what happens when the source resource (e.g., a central ConfigMap in `default`) is updated?
- A. Kyverno deletes all target namespaces.
- B. Kyverno immediately propagates and synchronizes the updates into all generated target resources across all target namespaces.
- C. The API server generates an error.
- D. Nothing happens until the cluster restarts.

---

### Question 6: OPA Rego Set Comprehension vs Boolean Logic
In Rego policy definitions, when does an OPA rule rule evaluate to true?
- A. When the rule is written in Python.
- B. When the user executes `exit 0`.
- C. When all expressions within the rule body evaluate to true (AND logic).
- D. When at least one statement in the rule body evaluates to true (OR logic).

---

### Question 7: SPIFFE/SPIRE Workload Identity
How does SPIFFE/SPIRE provide cryptographic zero-trust identity to Kubernetes pods without relying on IP addresses?
- A. By requiring biometric authentication.
- B. By hardcoding passwords in environment variables.
- C. By assigning static MAC addresses to pods.
- D. By issuing short-lived, cryptographically signed X.509 SVIDs (SPIFFE Verifiable Identity Documents) or JWT SVIDs based on pod attestation (namespace, service account, container UID).

---

### Question 8: Trivy Misconfiguration Scanning (IAC)
In addition to container image vulnerability scanning, what other platform artifacts can Trivy scan for compliance and misconfigurations?
- A. Optical fiber cables.
- B. Network router power supplies.
- C. Terraform code, Kubernetes YAML manifests, Dockerfiles, and Helm charts.
- D. Physical server motherboards.

---

### Question 9: Kyverno Webhook High Availability
To prevent admission controller downtime from blocking cluster operations, what is the recommended deployment configuration for the Kyverno admission controller pods?
- A. Run a single pod with no replicas.
- B. Run at least 3 replicas spread across different nodes using `podAntiAffinity`, backed by a PodDisruptionBudget (`minAvailable: 2`).
- C. Run Kyverno only on the master node.
- D. Run Kyverno as a CronJob every midnight.

---

### Question 10: Pod Security Standards Baseline Profile
Which of the following configurations is permitted under the Pod Security Standard `Baseline` profile, but strictly prohibited under the `Restricted` profile?
- A. Running with `privileged: true`
- B. Sharing the host PID namespace (`hostPID: true`)
- C. Sharing the host network (`hostNetwork: true`)
- D. Running as root (`runAsNonRoot: false` or unset) and using the default container capability set.

---

## Answer Key & Rationales

1. **A** — When a webhook with `failurePolicy: Fail` times out, the Kubernetes API server treats it as a validation failure and rejects the admission request.
2. **B** — Pinning the PSS enforce version ensures predictable evaluation; upgrading Kubernetes won't silently introduce new PSS checks that block deployments.
3. **A** — `PushSecret` is an ESO CRD that synchronizes Kubernetes Secret data upstream into external providers like AWS Secrets Manager or HashiCorp Vault.
4. **A** — Rekor provides cryptographic proof of signature timestamp and artifact provenance on a public, immutable Merkle tree ledger.
5. **B** — When `sync: true` is configured on a Kyverno generate rule, mutations to the source data are actively synchronized to all generated resources.
6. **C** — In Rego, statements within a rule body are conjunctions (AND); all statements must evaluate to true for the rule to trigger.
7. **D** — SPIFFE provides cryptographic SVIDs based on workload attestation, allowing mutual TLS authentication regardless of network topology.
8. **C** — Trivy performs static analysis across Infrastructure-as-Code (IaC) templates, checking Kubernetes YAML, Terraform, Helm, and Dockerfiles for misconfigurations.
9. **B** — Admission controllers on the critical path require multi-replica HA with pod anti-affinity and PDBs to withstand node failures and maintenance.
10. **D** — The `Baseline` profile prevents egregious privilege escalation (like `privileged: true` or `hostPID: true`) but permits default root execution; `Restricted` strictly mandates `runAsNonRoot: true`.
