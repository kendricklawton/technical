# Domain 5: Supply Chain Security — Quiz 1 (Trivy, CVE Scanning & Base Image Security)

**Exam:** Certified Kubernetes Security Specialist (CKS)  
**Format:** 10 Performance Scenario & Multiple Choice Security Questions with Explanations

---

### Question 1: Filtering Trivy Image Scans by Severity (`--severity`)
During a CKS task, you must scan all container images running in namespace `payments` and delete any Pod whose image contains `CRITICAL` or `HIGH` vulnerabilities. Which `trivy` command scans `nginx:1.19` for **only** `HIGH` and `CRITICAL` CVEs?
- A. `trivy scan --level high,critical nginx:1.19`
- B. `trivy inspect nginx:1.19`
- C. `trivy cve --filter HIGH nginx:1.19`
- D. `trivy image --severity HIGH,CRITICAL nginx:1.19`

---

### Question 2: Failing CI/CD Pipelines with `trivy image --exit-code`
By default, `trivy image` exits with return code `0` even when critical vulnerabilities are discovered. Which flag forces `trivy` to return a non-zero exit code (`1`) when vulnerabilities matching `--severity CRITICAL` are found?
- A. `trivy image --fail-on CRITICAL <image>`
- B. `trivy image --exit-code 1 --severity CRITICAL <image>`
- C. `trivy image --abort <image>`
- D. `trivy image --strict <image>`

---

### Question 3: Ignoring Unfixed Vulnerabilities (`--ignore-unfixed`)
An upstream Debian base image has 4 `HIGH` CVEs for which the Debian security team has not yet released a patched package (`Status: affected / will_not_fix`). Which Trivy flag filters out vulnerabilities that have no available fix so engineers can focus on actionable upgrades?
- A. `--only-fixed`
- B. `--skip-unpatched`
- C. `--ignore-unfixed`
- D. `--no-upstream`

---

### Question 4: Scanning Offline / Air-Gapped Local Containerd Images with Trivy
In the CKS exam environment, if an image exists only in the local `containerd` image store (not on Docker Hub) or you want to skip downloading a new vulnerability DB update (`--skip-db-update`), how can `trivy` scan it?
- A. `trivy local <image>`
- B. `trivy containerd <image>`
- C. `trivy离线 <image>`
- D. `trivy image --skip-db-update <image>` (or export via `crictl` / `ctr -n k8s.io images export img.tar <image>` and run `trivy image --input img.tar`).

---

### Question 5: Pinning Container Images by Immutable SHA-256 Digest
Why is referencing `image: nginx:1.27.1` in a production Kubernetes Deployment still vulnerable to supply-chain tag mutation, and how do you pin the exact immutable artifact?
- A. Set `imagePullPolicy: Never`.
- B. Registry tags (`:1.27.1`) are mutable pointers that can be overwritten if a registry account is compromised; pinning by cryptographic digest (`image: nginx@sha256:4a8b9c...`) guarantees the kubelet verifies the exact content-addressable SHA-256 hash of the image manifest.
- C. Add an annotation with the tag name.
- D. Use `:latest` with `AlwaysPullImages`.

---

### Question 6: Extracting the `repoDigests` SHA-256 of a Running Container
A CKS question asks you to update Deployment `web` so its container image is pinned to its current `@sha256:...` digest. Which command retrieves the exact `imageID` / `SHA-256` digest of the running container?
- A. `kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[0].imageID}'` (or `crictl inspecti <image-name> | grep -A 2 repoDigests`)
- B. `kubectl describe deploy web`
- C. `md5sum /usr/bin/nginx`
- D. `kubectl top pod <pod-name>`

---

### Question 7: Why Distroless (`gcr.io/distroless/*`) & Scratch Images Reduce Attack Surface
What is removed from `gcr.io/distroless/static` or `scratch` runtime images compared to `ubuntu:24.04` or `alpine:3.20`?
- A. Network sockets.
- B. Package managers (`apt`, `apk`), POSIX shells (`/bin/sh`, `/bin/bash`), and core utilities (`curl`, `wget`, `chmod`, `find`), eliminating dozens of OS library CVEs and preventing attackers from executing reverse shells or downloading post-exploitation payloads if an RCE vulnerability occurs in the application.
- C. The Linux kernel.
- D. The application binary itself.

---

### Question 8: Scanning Kubernetes Manifests & Live Clusters with `trivy k8s` / `trivy config`
Beyond scanning container image layers for CVEs, which `trivy` subcommands scan local Kubernetes YAML files (`deployment.yaml`) and live Kubernetes cluster namespaces for misconfigurations (such as missing `runAsNonRoot` or privileged containers)?
- A. `trivy yaml deployment.yaml`
- B. `trivy policy deployment.yaml`
- C. `trivy lint deployment.yaml`
- D. `trivy config deployment.yaml` (for local IaC/YAML files) and `trivy k8s -n default --report summary` (for live cluster resources).

---

### Question 9: Auditing Vulnerable OS Packages vs Language-Specific Dependencies in Trivy
When `trivy image` scans a Python or Node.js container image, which two categories of vulnerabilities does it inspect by default (`--scanners vuln,secret`)?
- A. Both OS-level packages (`dpkg`, `apk`, `rpm`) AND language-specific application dependency lockfiles/metadata (`pip` `requirements.txt`/`METADATA`, `npm` `package-lock.json`, `Go` binary buildinfo, `jar` `pom.xml`), plus embedded plaintext secrets.
- B. Only running processes.
- C. Only OS packages installed via `apt`.
- D. Only the Dockerfile text.

---

### Question 10: Securing Private Registry Credentials (`kubernetes.io/dockerconfigjson`)
How do you create a Kubernetes `Secret` named `regcred` for authenticating to a private registry `registry.corp.io` and configure a ServiceAccount `app-sa` so all Pods using `app-sa` automatically inherit the pull secret without adding `imagePullSecrets` to every Pod YAML?
- A. Put the password in `/etc/hosts`.
- B. Pass `--docker-password` on the container `command:` line.
- C. Run `kubectl create secret docker-registry regcred --docker-server=registry.corp.io --docker-username=... --docker-password=...` and patch the ServiceAccount: `kubectl patch sa app-sa -p '{"imagePullSecrets": [{"name": "regcred"}]}'`.
- D. Store the password in a `ConfigMap`.

---

## Answer Key & Detailed Explanations

1. **D** — `trivy image --severity HIGH,CRITICAL <image>` filters output strictly to High and Critical CVEs.
2. **B** — `--exit-code 1` returns exit status 1 when matching vulnerabilities exist, halting CI/CD pipelines.
3. **C** — `--ignore-unfixed` omits CVEs that do not yet have an upstream patch available.
4. **D** — `trivy image` supports `--skip-db-update` (when the DB is already cached on the CKS node), direct CRI socket scanning, or `--input archive.tar`.
5. **B** — Image tags are mutable; SHA-256 digests (`@sha256:...`) are cryptographically immutable.
6. **A** — `.status.containerStatuses[].imageID` on a running Pod or `crictl inspecti` exposes the exact `sha256` digest pulled from the registry.
7. **B** — Distroless images contain only the application and its direct runtime dependencies—no shell, no package manager, and minimal CVE surface.
8. **D** — `trivy config` (formerly `trivy conf`) scans static Kubernetes/Terraform/Dockerfile files, while `trivy k8s` audits live cluster objects.
9. **A** — Trivy inspects OS package databases, language dependency manifests/binaries (including Go binaries compiled into distroless images), and leaked secrets.
10. **C** — Attaching `imagePullSecrets` to a `ServiceAccount` causes the `ServiceAccount` admission controller to automatically inject `imagePullSecrets` into every Pod created with that ServiceAccount.
