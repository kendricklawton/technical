# Domain 5: Supply Chain Security — Quiz 2 (Dockerfile Hardening, Static Analysis & SBOMs)

**Exam:** Certified Kubernetes Security Specialist (CKS)  
**Format:** 10 Performance Scenario & Multiple Choice Security Questions with Explanations

---

### Question 1: Multi-Stage Dockerfile Secret Leakage Trap
An engineer writes a single-stage Dockerfile that copies a private SSH deploy key (`COPY id_rsa /root/.ssh/id_rsa`), clones a private repository, and then runs `RUN rm -f /root/.ssh/id_rsa` in a subsequent `RUN` instruction. Why is `id_rsa` still compromised if the image is pushed to a registry?
- A. OCI container images consist of immutable, additive tar layers; `rm -f` in a later `RUN` instruction only creates a `.wh.id_rsa` whiteout file in the upper layer while the plaintext `id_rsa` remains recoverable via `docker history` / `tar -xf` in the lower layer—which is why multi-stage builds (`FROM ... AS builder`) or BuildKit `--mount=type=secret` must be used.
- B. `rm -f` does not work on SSH keys.
- C. The key is stored in `etcd`.
- D. Docker automatically publishes `/root/.ssh` to GitHub.

---

### Question 2: Numeric UID vs Username in Dockerfile `USER` Instruction
Why is `USER 65532:65532` (numeric UID:GID) preferred over `USER appuser` (symbolic username) in a hardened Dockerfile destined for a Kubernetes cluster enforcing `runAsNonRoot: true`?
- A. The kubelet cannot parse `/etc/passwd` inside the container image before starting the container unless the UID is numeric; if `USER appuser` is used without numeric `securityContext.runAsUser` in the Pod YAML, the kubelet rejects the container with `Error: container has runAsNonRoot and image has non-numeric user (appuser), cannot verify user is non-root`.
- B. Numeric UIDs execute faster in the Linux kernel.
- C. `USER appuser` grants root permissions.
- D. `/etc/passwd` is deleted by `kube-proxy`.

---

### Question 3: Dockerfile `COPY` vs `ADD` Security Best Practice
Why do CIS Docker Benchmark and CKS best practices require using `COPY` instead of `ADD` when copying local files into an image?
- A. `ADD` implicitly fetches remote URLs and automatically extracts local compressed tar archives (risking Zip Bomb / arbitrary path traversal attacks), whereas `COPY` performs an explicit, predictable local file copy only.
- B. `ADD` runs containers as root.
- C. `COPY` encrypts files automatically.
- D. `ADD` is deprecated and removed from Docker.

---

### Question 4: Static Analysis of Kubernetes Manifests with `kubesec`
You run `kubesec scan pod.yaml` on a Pod manifest and receive a negative score (`-30`). Which specific Pod findings cause `kubesec` to assign a critical negative score?
- A. `privileged: true`, `hostNetwork: true`, `hostPID: true`, `hostIPC: true`, mounting `/var/run/docker.sock` or `/proc` via `hostPath`, or granting `CAP_SYS_ADMIN`.
- B. Setting `readOnlyRootFilesystem: true`.
- C. Setting `runAsNonRoot: true`.
- D. Using `emptyDir` volumes.

---

### Question 5: Policy-as-Code Static Analysis with `conftest` (OPA Rego)
In a CI pipeline before `kubectl apply`, you want to test local Kubernetes YAML manifests (`deployment.yaml`) and Dockerfiles against a directory of OPA Rego policies (`./policy/deployment.rego`). Which CLI tool executes Rego rules directly against structured configuration files (YAML, JSON, Dockerfile, HCL)?
- A. `conftest test deployment.yaml -p ./policy/`
- B. `kubeadm verify deployment.yaml`
- C. `etcdctl check deployment.yaml`
- D. `crictl lint deployment.yaml`

---

### Question 6: Software Bill of Materials (SBOM) Standards (`SPDX` vs `CycloneDX`)
What are the two primary industry-standard machine-readable formats for generating and exchanging a Software Bill of Materials (SBOM) on the CKS exam?
- A. **SPDX** (Linux Foundation ISO/IEC 5962 standard, e.g., `spdx-json`) and **CycloneDX** (OWASP standard, e.g., `cyclonedx-json`)
- B. `CSV` and `XML-RPC`
- C. `Helm` and `Kustomize`
- D. `Seccomp` and `AppArmor`

---

### Question 7: Generating an SBOM with `syft` or `trivy` or `bom`
Which commands generate an SPDX JSON Software Bill of Materials (`sbom.spdx.json`) for container image `registry.k8s.io/pause:3.9`?
- A. `trivy image --format spdx-json --output sbom.spdx.json registry.k8s.io/pause:3.9` OR `syft registry.k8s.io/pause:3.9 -o spdx-json=sbom.spdx.json` OR `bom generate --image registry.k8s.io/pause:3.9 --format json --output sbom.spdx.json`
- B. `kubectl sbom create registry.k8s.io/pause:3.9`
- C. `crictl sbom registry.k8s.io/pause:3.9`
- D. `docker inspect registry.k8s.io/pause:3.9 > sbom.spdx.json`

---

### Question 8: Scanning an Existing SBOM File for CVEs with `trivy sbom`
Why do enterprise security platforms generate an SBOM once at build time and run `trivy sbom sbom.spdx.json` daily instead of re-pulling and scanning multi-gigabyte container images every day?
- A. Scanning a lightweight JSON SBOM (`trivy sbom sbom.spdx.json --severity CRITICAL`) takes milliseconds and requires zero container image pulls while still detecting newly disclosed zero-day CVEs against the exact package versions recorded in the SBOM.
- B. `trivy image` cannot detect OS CVEs.
- C. SBOM files automatically patch running containers.
- D. `trivy sbom` modifies the container image in place.

---

### Question 9: Dockerfile `ENV` / `ARG` Credential Exposure Risk
A developer passes `docker build --build-arg AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI...` into a Dockerfile containing `ARG AWS_SECRET_ACCESS_KEY` and `ENV AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY`. Why is this insecure?
- A. Both `ARG` values used in `RUN` steps and `ENV` variables are baked permanently into the image's OCI configuration JSON (`docker inspect` / `crictl inspecti` / `docker history`), exposing the plaintext secret to anyone who can pull the image.
- B. `ARG` variables are deleted after build.
- C. `ENV` variables are encrypted with AES-256.
- D. Kubernetes blocks images with `ENV` variables.

---

### Question 10: SLSA (Supply-chain Levels for Software Artifacts) Build Provenance
What does a SLSA (Supply-chain Levels for Software Artifacts) Provenance attestation (`in-toto` JSON statement) prove about a container image?
- A. It cryptographically binds the artifact's SHA-256 digest to the exact source Git repository commit, build system identity, build parameters, and dependencies used to produce the binary, defending against build-pipeline tampering.
- B. It proves the container has zero open network ports.
- C. It replaces Kubernetes RBAC.
- D. It configures `kube-proxy` IPVS rules.

---

## Answer Key & Detailed Explanations

1. **A** — Every `COPY` or `RUN` creates an immutable filesystem layer; deleting a file in a subsequent layer only hides it via an overlayfs whiteout file while the secret remains in the lower layer tarball.
2. **A** — Kubelet's `runAsNonRoot` check inspects the numeric UID before starting the container; if the image uses a string username (`USER appuser`) without a numeric `runAsUser` in the Pod spec, verification fails.
3. **A** — `COPY` only copies local files, whereas `ADD` includes URL fetching and automatic archive extraction.
4. **A** — `kubesec` penalizes dangerous host-access configurations (`privileged`, `hostNetwork`, `hostPID`, `hostIPC`, `docker.sock`, `CAP_SYS_ADMIN`) with heavy negative points.
5. **A** — `conftest` (part of the Open Policy Agent project) evaluates Rego policies against Kubernetes manifests, Dockerfiles, and Terraform files.
6. **A** — SPDX and CycloneDX are the two recognized SBOM formats tested in CKS.
7. **A** — `trivy image --format spdx-json`, `syft -o spdx-json`, and the Kubernetes SIG Release `bom` CLI all generate compliant SPDX SBOMs.
8. **A** — `trivy sbom` matches the package inventory inside an existing SPDX/CycloneDX SBOM against the latest vulnerability database without pulling the image.
9. **A** — `docker inspect` / `crictl inspecti` reveals all `ENV` variables and `History` build commands in plaintext.
10. **A** — SLSA provenance attestations record verifiable metadata about how, where, and from what source commit an artifact was built.
