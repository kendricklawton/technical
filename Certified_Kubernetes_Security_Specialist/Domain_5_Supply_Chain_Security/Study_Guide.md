# Domain 5: Supply Chain Security — In-Depth Master Study Guide

**Exam Weight:** 20% of CKS  
**Target Mastery:** Minimizing base image attack surface, Multi-stage distroless builds, Vulnerability scanning with Trivy, Software Bill of Materials (SBOM), Cryptographic image signing with Cosign, and ImagePolicyWebhook admission control.

---

## 1. Minimizing Image Footprint & Multi-Stage Builds

### Secure Multi-Stage Dockerfile Pattern
```dockerfile
# Stage 1: Build & Compile
FROM golang:1.27-alpine AS builder
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -o /bin/api-server .

# Stage 2: Minimal Distroless Runtime
FROM gcr.io/distroless/static-debian12:nonroot
USER nonroot:nonroot
WORKDIR /app
COPY --from=builder /bin/api-server /app/api-server
EXPOSE 8080
ENTRYPOINT ["/app/api-server"]
```

### Security Benefits of Distroless:
- No package manager (`apt`, `apk`, `yum`).
- No shell (`sh`, `bash`).
- Eliminates 95%+ of typical OS-level CVEs.
- Attacker cannot spawn an interactive terminal if execution injection occurs.

---

## 2. Container Image Vulnerability Scanning with Trivy

Trivy scans container images, file systems, Git repositories, and Kubernetes configurations for known CVEs.

```bash
# Scan container image for High and Critical vulnerabilities only
trivy image --severity HIGH,CRITICAL nginx:1.21

# Fail the CI/CD pipeline or script if High/Critical vulnerabilities exist
trivy image --severity HIGH,CRITICAL --exit-code 1 --ignore-unfixed python:3.9-alpine

# Scan a local Dockerfile or Kubernetes manifest directory
trivy config /home/k-henry/manifests/

# Generate an SBOM in CycloneDX or SPDX format
trivy image --format spdx-json --output /tmp/app-sbom.json myapp:v1.0.0
```

---

## 3. Cryptographic Image Signing with Sigstore Cosign

Cosign provides keyless or key-based signing of container images in OCI registries.

```bash
# Generate key pair
cosign generate-key-pair

# Sign container image
cosign sign --key cosign.key myregistry.io/apps/secure-api:v1.0.0

# Verify signature
cosign verify --key cosign.pub myregistry.io/apps/secure-api:v1.0.0
```

---

## 4. `ImagePolicyWebhook` Admission Controller

Enforces that images are validated by an external webhook server before the API server accepts a Pod creation request.

### Control Plane Configuration (`/etc/kubernetes/admission/admission-config.yaml`)
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: ImagePolicyWebhook
  configuration:
    imagePolicy:
      kubeConfigFile: /etc/kubernetes/admission/kubeconfig.yaml
      allowTTL: 50
      denyTTL: 50
      retryBackoff: 500
      defaultAllow: false
```

> [!CAUTION]
> Setting `defaultAllow: false` enforces a **fail-closed** security model. If the external webhook server is offline or unreachable, ALL pod deployments are rejected!
