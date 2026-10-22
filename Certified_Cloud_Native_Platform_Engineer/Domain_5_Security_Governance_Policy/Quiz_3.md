# Domain 5: Security, Governance & Policy Enforcement — Quiz 3

**Exam:** Certified Cloud Native Platform Engineer (CNPE)  
**Format:** 10 Hands-On Scenario & Performance Questions with Detailed Rationales

---

### Question 1: Admission Webhook WebhookTimeoutSeconds
An admission webhook is taking 12 seconds to respond due to external database lookups. The API server has a client timeout of 10 seconds. What will happen to developer requests applying new pods if the webhook has `failurePolicy: Fail`?
- A. The requests will pass automatically.
- B. The API server will abort the request after the webhook timeout expires, returning an HTTP 500/Internal Server Error and blocking pod admission.
- C. The pods will be scheduled without validation.
- D. The nodes will be rebooted.

---

### Question 2: Pod Security Admission Enforcement Version
Why is it best practice to specify `pod-security.kubernetes.io/enforce-version: v1.31` (a pinned version) rather than `latest` in production namespace labels?
- A. Pinned versions run faster.
- B. Pinning the version prevents a future Kubernetes cluster upgrade from automatically applying new, stricter rules from a newer PSS version that could unexpectedly break existing running workloads.
- C. `latest` is not a valid Kubernetes string.
- D. Pinned versions disable audit logging.

---

### Question 3: ESO PushSecret CRD
While `ExternalSecret` pulls credentials from an external store into Kubernetes, which External Secrets Operator CRD allows pushing a Kubernetes Secret out to an external secret store (e.g. AWS Secrets Manager or Vault)?
- A. `PushSecret`
- B. `ExportSecret`
- C. `SecretUploader`
- D. `ClusterSecretStore`

---

### Question 4: Sigstore Rekor Transparency Log Purpose
What security guarantee does the Rekor transparency log provide in the Sigstore ecosystem?
- A. It provides an immutable, tamper-evident, append-only public ledger proving that a specific artifact signature existed at a specific point in time.
- B. It stores the full container filesystem layers.
- C. It encrypts Git commit messages.
- D. It hosts container images for Docker.

---

### Question 5: Kyverno Generating Sync Property
When configuring a Kyverno `generate` rule with `sync: true`, what happens when the source resource (e.g., a central ConfigMap in `default`) is updated?
- A. Nothing happens until the cluster restarts.
- B. Kyverno immediately propagates and synchronizes the updates into all generated target resources across all target namespaces.
- C. Kyverno deletes all target namespaces.
- D. The API server generates an error.

---

### Question 6: OPA Rego Set Comprehension vs Boolean Logic
In Rego policy definitions, when does an OPA rule rule evaluate to true?
- A. When at least one statement in the rule body evaluates to true (OR logic).
- B. When all expressions within the rule body evaluate to true (AND logic).
- C. When the rule is written in Python.
- D. When the user executes `exit 0`.

---

### Question 7: SPIFFE/SPIRE Workload Identity
How does SPIFFE/SPIRE provide cryptographic zero-trust identity to Kubernetes pods without relying on IP addresses?
- A. By issuing short-lived, cryptographically signed X.509 SVIDs (SPIFFE Verifiable Identity Documents) or JWT SVIDs based on pod attestation (namespace, service account, container UID).
- B. By hardcoding passwords in environment variables.
- C. By assigning static MAC addresses to pods.
- D. By requiring biometric authentication.

---

### Question 8: Trivy Misconfiguration Scanning (IAC)
In addition to container image vulnerability scanning, what other platform artifacts can Trivy scan for compliance and misconfigurations?
- A. Terraform code, Kubernetes YAML manifests, Dockerfiles, and Helm charts.
- B. Physical server motherboards.
- C. Optical fiber cables.
- D. Network router power supplies.

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
- B. Running as root (`runAsNonRoot: false` or unset) and using the default container capability set.
- C. Sharing the host PID namespace (`hostPID: true`)
- D. Sharing the host network (`hostNetwork: true`)

---

## Answer Key & Rationales

1. **B** — When a webhook with `failurePolicy: Fail` times out, the Kubernetes API server treats it as a validation failure and rejects the admission request.
2. **B** — Pinning the PSS enforce version ensures predictable evaluation; upgrading Kubernetes won't silently introduce new PSS checks that block deployments.
3. **A** — `PushSecret` is an ESO CRD that synchronizes Kubernetes Secret data upstream into external providers like AWS Secrets Manager or HashiCorp Vault.
4. **A** — Rekor provides cryptographic proof of signature timestamp and artifact provenance on a public, immutable Merkle tree ledger.
5. **B** — When `sync: true` is configured on a Kyverno generate rule, mutations to the source data are actively synchronized to all generated resources.
6. **B** — In Rego, statements within a rule body are conjunctions (AND); all statements must evaluate to true for the rule to trigger.
7. **A** — SPIFFE provides cryptographic SVIDs based on workload attestation, allowing mutual TLS authentication regardless of network topology.
8. **A** — Trivy performs static analysis across Infrastructure-as-Code (IaC) templates, checking Kubernetes YAML, Terraform, Helm, and Dockerfiles for misconfigurations.
9. **B** — Admission controllers on the critical path require multi-replica HA with pod anti-affinity and PDBs to withstand node failures and maintenance.
10. **B** — The `Baseline` profile prevents egregious privilege escalation (like `privileged: true` or `hostPID: true`) but permits default root execution; `Restricted` strictly mandates `runAsNonRoot: true`.
