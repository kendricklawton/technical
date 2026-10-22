# Domain 5: Security, Governance & Policy Enforcement — Authoritative White Papers & CNCF Standards

> **CNPE Exam Weight:** 15% of Certified Cloud Native Platform Engineer

---

## 1. Authoritative References & Standards

- **[CNCF Cloud Native Security Whitepaper (v2)](https://github.com/cncf/tag-security):** The lifecycle framework covering Develop, Distribute, Deploy, and Runtime cloud-native security.
- **[Kubernetes Pod Security Standards (PSS)](https://kubernetes.io/docs/concepts/security/pod-security-standards/):** Privileged, Baseline, and Restricted control specifications.
- **[Kyverno Policy Architecture Guide](https://kyverno.io/docs/):** Validation, mutation, generation, image verification, and cleanup policies.
- **[Open Policy Agent (OPA) & Gatekeeper Architecture](https://open-policy-agent.github.io/gatekeeper/website/docs/):** Rego constraint frameworks, audit versus admission mechanics, and external data integration.
- **[External Secrets Operator (ESO) Architecture](https://external-secrets.io/latest/):** Multi-tenant secret synchronization, provider abstractions, and automated secret rotation.
- **[SLSA (Supply-chain Levels for Software Artifacts) Framework](https://slsa.dev/):** Build integrity levels, in-toto provenance attestations, and cryptographic verification.

---

## 2. Staff SRE Architecture Patterns

### 2.1 Policy-As-Code Rollout Strategy (Audit $\rightarrow$ Warn $\rightarrow$ Enforce)
Applying a new restrictive admission policy directly in `Enforce` mode can inadvertently break critical cluster components or disrupt rolling deployments:
1. **Phase 1: Audit Mode (`validationFailureAction: Audit` / `enforcementAction: dryrun`).** Run the policy for 14 days. Inspect policy violation reports (`PolicyReport` CRDs in Kyverno or Gatekeeper audit metrics) to identify all non-compliant workloads.
2. **Phase 2: Warning Mode (`warn`).** Deliver immediate feedback to developers on `kubectl apply` or in CI pull request checks without blocking admission.
3. **Phase 3: Enforce Mode (`Enforce` / `deny`).** Block non-compliant resources at admission time.
4. **Policy Exceptions (`PolicyException` CRD):** Never add ad-hoc exclusions into the main policy YAML. Maintain an explicit exception registry with mandatory expiration dates and ticket tracking.

### 2.2 Zero-Trust Secrets Lifecycle with ESO
- **Decoupled RBAC:** Application developers require `get`, `list`, `watch` on namespace-local `ExternalSecret` and generated `Secret` objects. They are strictly prohibited from accessing cloud provider IAM keys or HashiCorp Vault root tokens.
- **`creationPolicy: Owner`:** Ensures that if the `ExternalSecret` custom resource is deleted, the generated Kubernetes `Secret` is automatically garbage collected, preventing orphaned credentials.
- **Rotational Hygiene:** Set `refreshInterval: 1h`. When database passwords or API keys are rotated upstream in Vault or AWS Secrets Manager, ESO updates the Kubernetes Secret within 60 minutes. Workloads configured with Reloader automatically trigger rolling updates.

### 2.3 Software Supply Chain Defense In Depth
Cryptographic image signing alone does not guarantee vulnerability-free software:
- **Build Stage:** Scan container images for CVEs using Trivy or Grype. Gate CI pipeline if CRITICAL CVEs with available fixes exist.
- **Signing Stage:** Cosign signs image digest (`sha256:...`) using GitHub Actions OIDC keyless signing, recording the signature in Sigstore Rekor.
- **Attestation Stage:** Generate an SBOM using Syft in SPDX or CycloneDX format, attest the SBOM using `cosign attest`, and upload it to the OCI registry.
- **Admission Stage:** Kyverno validates both the cryptographic signature and the SBOM attestation before allowing pods to be scheduled.

---

## 3. Production Failure Modes & Anti-Patterns

| Anti-Pattern | Operational Risk | Production Resolution |
| :--- | :--- | :--- |
| **Admission Webhook FailurePolicy: Fail Deadlock** | A webhook configured with `failurePolicy: Fail` whose pods fail or become unready will block all API server mutations, preventing even administrative recovery pods from starting. | Set `failurePolicy: Ignore` for non-critical webhooks, or use `namespaceSelector` to strictly exclude `kube-system`, `kube-node-lease`, and the webhook's own namespace. |
| **Storing SealedSecrets in Git without Key Backup** | Bitnami SealedSecrets encrypts secrets with a cluster private key. If the cluster is lost without a backup of the private key, all Git-tracked secrets become permanently irrecoverable. | Use External Secrets Operator (ESO) backed by an enterprise secrets store (Vault, AWS Secrets Manager) rather than in-cluster encrypted secrets. |
| **Using Image Tags instead of Digests** | Deploying `:latest` or mutable semantic tags (e.g., `:v1.2.0`) allows compromised registries to serve malicious images under legitimate tags. | Enforce image digest pinning (`image@sha256:...`) via Kyverno mutation/validation policies. |
| **Unbounded Kyverno Memory Consumption** | Running Kyverno background scans across hundreds of thousands of cluster objects causes controller OOM crashes. | Tune `backgroundScanInterval`, disable background scanning on high-churn resources, and increase memory limits. |
