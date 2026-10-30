# Domain 5: Supply Chain Security — Essential Security Benchmarks & Technical Documentation Guide

> **Blueprint Weight:** 20% of Certified Kubernetes Security Specialist (CKS)  
> **Core Objective:** Secure the software supply chain: scan container images for known CVEs using vulnerability scanners (Trivy), verify cryptographic signatures via Sigstore / Cosign, analyze Software Bills of Materials (SBOMs), and enforce image admission policies.

---

## 1. Authoritative Security Standards & Documentation

### 1.1 [CNCF Software Supply Chain Best Practices White Paper](https://github.com/cncf/tag-security/blob/main/supply-chain-security/supply-chain-security-paper/sscs-whitepaper.pdf)
* **Canonical Scope:** Architectural framework for securing the entire software lifecycle from source code to production deployment.
* **Core Pillars:**
  * **Source Code Integrity:** Branch protection, commit signing, two-person code reviews.
  * **Hermetic & Reproducible Builds:** Isolated build environments with pinned dependencies.
  * **Artifact Integrity & Provenance:** Cryptographically signing artifacts, generating SLSA provenance, and storing signatures in OCI registries.

### 1.2 [Trivy Vulnerability & SBOM Scanner](https://aquasecurity.github.io/trivy/)
* **Canonical Scope:** Scanning container images, filesystems, and Git repositories for vulnerabilities, misconfigurations, and licenses.
* **Essential Commands:**
  * Scan image for High and Critical vulnerabilities:
    ```bash
    trivy image --severity HIGH,CRITICAL --ignore-unfixed <image-name>
    ```
  * Generate Software Bill of Materials (SBOM) in SPDX format:
    ```bash
    trivy image --format spdx-json --output sbom.spdx.json <image-name>
    ```

### 1.3 [Sigstore & Cosign Documentation](https://docs.sigstore.dev/cosign/overview/)
* **Canonical Scope:** Standard for signing, verifying, and storing container signatures in OCI-compliant registries.
* **Core Commands:**
  * Sign container image: `cosign sign --key cosign.key <image-uri>`
  * Verify signature: `cosign verify --key cosign.pub <image-uri>`

### 1.4 [ImagePolicyWebhook Admission Controller](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#imagepolicywebhook)
* **Canonical Scope:** Legacy in-tree webhook mechanism intercepting Pod creation to validate container image digests and signatures against an external policy engine.

---

## 2. High-Frequency Exam Traps & Practical Reflexes

1. **Trivy Severity Filtering:**
   * In exam tasks requiring you to identify and delete failing pods, use `trivy image --severity CRITICAL <image>` to quickly find offending images without getting bogged down in low-severity noise.
2. **Immutable Image Tags & Digests:**
   * Always pin production container images by cryptographic digest (`image@sha256:45b23dee...`) rather than mutable tags like `:latest` or `:v1.0`.
3. **ImagePolicyWebhook Backend Down:**
   * If the external webhook server is down and `defaultAllow: false` is configured in `/etc/kubernetes/admission/admission-config.yaml`, all Pod creations across the cluster will fail.

---

## 3. Reading & Study Checklist

- [ ] Read *CNCF Software Supply Chain Best Practices White Paper*
- [ ] Read *Trivy CLI Documentation*
- [ ] Read *Sigstore / Cosign Documentation*
- [ ] Practice running Trivy scans and filtering by severity
- [ ] Configure and test ImagePolicyWebhook admission controller
