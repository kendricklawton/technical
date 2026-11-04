# Domain 5: Security, Governance & Policy Enforcement — Quiz 2

**Exam:** Certified Cloud Native Platform Engineer (CNPE)  
**Format:** 10 Hands-On Scenario & Performance Questions with Detailed Rationales

---

### Question 1: Kyverno Image Tag Mutating Policy
You need to ensure that every container image pulled from the internal registry `registry.corp.net/apps/*` is automatically rewritten at admission time to append the `:stable` tag if no tag is specified. Which Kyverno rule type achieves this?
- A. `verifyImages`
- B. `mutate`
- C. `generate`
- D. `validate`

---

### Question 2: OPA Gatekeeper Enforcement Action Dryrun
What is the effect of setting `enforcementAction: dryrun` on an OPA Gatekeeper `Constraint`?
- A. Violations are audited and exported as Prometheus metrics and logged in the constraint status, but non-compliant requests are NOT denied admission.
- B. Pods are scaled to zero replicas.
- C. The constraint is deleted from the cluster.
- D. The API server dry-runs pod deployments in memory.

---

### Question 3: ClusterSecretStore vs SecretStore in ESO
What is the structural scope difference between a `SecretStore` and a `ClusterSecretStore` in External Secrets Operator?
- A. `SecretStore` is namespace-scoped and can only be referenced by `ExternalSecrets` within that same namespace; `ClusterSecretStore` is cluster-scoped and can be referenced across all namespaces.
- B. There is no difference.
- C. `SecretStore` is stored in etcd; `ClusterSecretStore` is stored on local disk.
- D. `SecretStore` only works with AWS; `ClusterSecretStore` only works with GCP.

---

### Question 4: Software Bill of Materials (SBOM) Formats
Which two standard formats are widely recognized by CNCF tools (such as Syft, Trivy, and Cosign) for generating and attesting Software Bill of Materials?
- A. YAML and INI
- B. SPDX and CycloneDX
- C. Markdown and HTML
- D. XML and CSV

---

### Question 5: Kyverno Background Scanning Mechanics
By default, Kyverno sets `background: true` on validation rules. What does this background controller perform?
- A. It scans images on Docker Hub.
- B. It restarts non-compliant pods automatically.
- C. It periodically scans existing cluster resources against the policy and creates `PolicyReport` / `ClusterPolicyReport` CRDs identifying non-compliant resources already running in the cluster.
- D. It recompiles Linux kernel modules.

---

### Question 6: Supply Chain Attestation Verification
When verifying an in-toto attestation with Cosign, what does the attestation verify beyond a simple cryptographic signature?
- A. It checks the network latency between developer laptops and the registry.
- B. It verifies that the developer has paid their cloud bill.
- C. It verifies structured claims about the build process, such as SLSA provenance, test execution results, and vulnerability scan reports generated during CI.
- D. It verifies the size of the git commit.

---

### Question 7: Cilium L7 NetworkPolicy HTTP Rule Syntax
Which YAML snippet correctly configures a `CiliumNetworkPolicy` to allow only `GET` requests to the `/healthz` endpoint on port `8080`?
- A. ```yaml
egress:
  - allowHTTP: "GET /healthz"
```
- B. ```yaml
httpFilter:
  denyAllExcept: "GET /healthz"
```
- C. ```yaml
rules:
  http:
    - method: "GET"
      path: "/healthz"
```
- D. ```yaml
toPorts:
  - ports:
      - port: "8080"
        protocol: TCP
    rules:
      http:
        - method: "GET"
          path: "/healthz"
```

---

### Question 8: HashiCorp Vault Kubernetes Auth Engine
When External Secrets Operator authenticates to HashiCorp Vault using the Kubernetes authentication method, what token does ESO present to Vault?
- A. A hardcoded Vault root token.
- B. A basic auth username and password.
- C. An SSH private key.
- D. A short-lived JSON Web Token (JWT) from a Kubernetes ServiceAccount token projection.

---

### Question 9: Trivy Vulnerability Severity Gating
In a platform CI/CD pipeline, which Trivy CLI flag is used to exit with a non-zero code only when CRITICAL or HIGH vulnerabilities are detected?
- A. `trivy image --exit-code 1 --severity CRITICAL,HIGH my-image:tag`
- B. `trivy image --block-cve all my-image:tag`
- C. `trivy image --force-stop my-image:tag`
- D. `trivy run --strict my-image:tag`

---

### Question 10: Kyverno Auto-Gen Feature
When writing a Kyverno `ClusterPolicy` matching `Pod` resources, what does the Kyverno "auto-gen" feature automatically generate?
- A. Terraform code for AWS.
- B. Helm charts for the application.
- C. Documentation in HTML.
- D. Equivalent rules matching higher-level workload controllers (Deployments, StatefulSets, DaemonSets, Jobs, CronJobs) to catch violations before pods are spawned.

---

## Answer Key & Rationales

1. **B** — Kyverno `mutate` rules modify resource payloads during admission before validation and storage in etcd.
2. **A** — In Gatekeeper, `enforcementAction: dryrun` evaluates rules without blocking API requests, writing violations to the constraint status for auditing.
3. **A** — `SecretStore` is namespace-scoped; `ClusterSecretStore` is cluster-scoped and allows centralized credential definitions shared across multiple namespaces.
4. **B** — SPDX (Software Package Data Exchange) and CycloneDX are the two primary industry standards for machine-readable SBOM metadata.
5. **C** — Kyverno background scanning evaluates existing workloads against active policies, publishing results in `PolicyReport` CRDs without disrupting running pods.
6. **C** — Attestations encapsulate cryptographic statements (predicates) regarding provenance (SLSA), vulnerability scan results, and CI environment metadata.
7. **D** — Cilium enforces L7 application rules under `toPorts.rules.http` specifying allowed HTTP methods and path regexes.
8. **D** — Vault's Kubernetes auth engine validates the pod's projected ServiceAccount JWT against the Kubernetes API `TokenReview` API before returning a Vault token.
9. **A** — `trivy image --exit-code 1 --severity CRITICAL,HIGH` instructs Trivy to return exit code 1 to fail the pipeline step if matching CVEs are found.
10. **D** — Kyverno auto-gen inspects rules targeting `Pod` and automatically synthesizes validation rules for `Deployment`, `StatefulSet`, `DaemonSet`, `Job`, and `CronJob`.
