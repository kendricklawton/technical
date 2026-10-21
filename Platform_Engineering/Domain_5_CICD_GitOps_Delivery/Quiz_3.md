# PE Domain 5: CI/CD, GitOps & Delivery — Scenario Quiz 3

Test your mastery of progressive delivery (Argo Rollouts), Sigstore Cosign, and SLSA supply chain security.

---

### Question 1
During an automated canary deployment using **Argo Rollouts**, 20% of traffic is shifted to the new version. The rollout's `AnalysisTemplate` queries Prometheus and discovers that the HTTP 500 error rate has spiked to 8% (threshold is 1%). What does Argo Rollouts do automatically?

A) It reboots the Kubernetes control plane.  
B) It pages the CEO.  
C) It routes 100% of traffic to the new version to stress-test it.  
D) It aborts the rollout immediately, restores 100% of traffic to the stable ReplicaSet, scales down the canary pods, and marks the rollout as Degraded.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Automated rollback is the core value of progressive delivery. If real-time metric analysis violates the defined SLO thresholds, Argo Rollouts aborts the canary step immediately, routing all user traffic back to the stable baseline without requiring human intervention.
</details>

---

### Question 2
How does **Sigstore Cosign keyless signing** authenticate the identity of the entity signing a container image in a GitHub Actions pipeline?

A) By scanning the developer's fingerprint.  
B) By storing private keys in a public S3 bucket.  
C) The CI runner obtains an ephemeral OpenID Connect (OIDC) token from GitHub, presents it to Sigstore's Fulcio Certificate Authority, which issues a short-lived x509 certificate bound to the GitHub repo URL and workflow identity, and records the signature in the Rekor transparency log.  
D) By requiring the developer to type a 64-character password into the terminal.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Keyless signing eliminates long-lived private keys. Fulcio verifies the GitHub OIDC token, issues a certificate valid for minutes tied to the repository and commit SHA, signs the container, and publishes an immutable cryptographic proof to the Rekor transparency log.
</details>

---

### Question 3
What is a **Software Bill of Materials (SBOM)**, and why is it essential for modern software supply chain security?

A) A backup copy of the source code.  
B) A comprehensive, machine-readable inventory of all third-party libraries, packages, dependencies, and licensing information embedded within a software artifact, enabling rapid vulnerability impact analysis when zero-day CVEs occur.  
C) A list of employees who contributed to a project.  
D) An invoice sent to customers for cloud usage.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** When major vulnerabilities (like Log4Shell) emerge, organizations without SBOMs spend weeks searching codebases manually. An SBOM (in SPDX or CycloneDX format) provides an immediate queryable database of all software components running across the enterprise.
</details>

---

### Question 4
Under the **SLSA (Supply-chain Levels for Software Artifacts)** framework, what is the primary achievement of reaching **SLSA Level 3**?

A) The software is 100% bug-free.  
B) The application runs at 100Gbps.  
C) The build platform is isolated and ephemeral, and generates non-falsifiable, cryptographically signed build provenance that proves the binary was produced from a specific Git commit on a trusted build system.  
D) All developers have passed background checks.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** SLSA Level 3 guarantees tamper-resistant build provenance. An attacker cannot modify the build script or forge provenance metadata; the signed attestation cryptographically binds the final artifact directly to its source repository and trusted build runner.
</details>

---

### Question 5
What is the difference between a **Canary Deployment** and a **Blue/Green Deployment**?

A) Blue/Green provisions an identical parallel environment and switches 100% of traffic instantly; Canary gradually shifts a small percentage of traffic (e.g., 5%, 20%, 50%) to the new version while monitoring error metrics before full promotion.  
B) Blue/Green is for cloud; Canary is for bare-metal.  
C) Blue/Green deployments do not support HTTP traffic.  
D) Canary deployments take 6 months.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Blue/Green is an all-or-nothing switch: you stand up "Green", verify it, and swap routing. Canary shifts traffic incrementally to a small cohort of real users, measuring live telemetry to catch subtle bugs before full blast radius exposure.
</details>

---

### Question 6
What admission control policy tool can be configured in a Kubernetes cluster to verify that container images deployed to production have a valid **Cosign signature** before admitting the pod?

A) CoreDNS  
B) Fluentbit  
C) Prometheus  
D) Kyverno (or Sigstore Policy Controller)  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Kyverno and Sigstore Policy Controller integrate natively with Cosign and Rekor. Admission rules inspect image references during pod creation, verifying cryptographic signatures and rejecting any un-signed or untrusted images.
</details>

---

### Question 7
In an Argo Rollouts `Rollout` specification, what is the purpose of the `pause: { duration: 10m }` step?

A) It reboots the pod containers.  
B) It holds the canary traffic weight steady for 10 minutes, allowing Prometheus to collect sufficient telemetry metrics and analysis templates to verify service health before advancing to the next weight step.  
C) It pauses all other deployments in the cluster.  
D) It gives the Kubernetes API server a 10-minute rest.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Pauses allow sufficient time for real users to exercise the canary pods. Analysis templates evaluate metrics over this window; if error rates or latencies violate thresholds during the pause, the rollout halts.
</details>

---

### Question 8
What is **Traffic Mirroring (Shadowing)** in advanced deployment architectures?

A) Copying HTML styling across websites.  
B) Running two identical load balancers.  
C) Duplicating real production incoming HTTP requests asynchronously and sending a copy to a candidate service without impacting the real response returned to end users, allowing performance testing against real-world traffic with zero business risk.  
D) Storing duplicate backups on optical discs.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Traffic shadowing (supported by Envoy, Istio, and Gateway API) duplicates production traffic and forwards it to a test version. Responses from the shadow service are discarded, allowing platform teams to observe performance and error rates under real production load with zero user impact.
</details>

---

### Question 9
What role does **in-toto** play in software supply chain security?

A) It replaces Git.  
B) It manages Linux user passwords.  
C) It compresses video files.  
D) It provides a framework for capturing and cryptographically verifying metadata across every step of the software supply chain (code review, test execution, compilation, packaging), ensuring no unauthorized steps occurred.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** in-toto creates a cryptographically verifiable chain of custody: Step A (unit tests passed) signs an attestation; Step B (build compiled) verifies Step A's signature and signs its own attestation, ensuring end-to-end provenance.
</details>

---

### Question 10
Why is generating an SBOM during the container build process more accurate than scanning a container image long after deployment?

A) Build-time generation captures exact compiler flags, source commit SHAs, package manager lockfiles, and intermediate build dependencies that are often stripped from minimal production distroless runtime images.  
B) Because SBOMs cannot be generated after a container is built.  
C) Build-time generation makes the container run 2x faster.  
D) It eliminates the need for container registries.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Minimal production images (like Distroless) deliberately strip build tools, headers, and package manager databases to reduce attack surface. Generating the SBOM during build time captures the complete build context before stripping occurs.
</details>
