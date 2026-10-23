# Domain 5: Supply Chain Security — Hands-On Exercises & Solutions

## Exercise 1: Automated Container Image Vulnerability Scanning with Trivy
### Scenario
Integrate Trivy into a CI/CD build script to audit base images, ignore unfixable CVEs, and fail the build pipeline if any `HIGH` or `CRITICAL` vulnerabilities are discovered.

```bash
# 1. Download and scan target image
trivy image \
  --severity HIGH,CRITICAL \
  --ignore-unfixed \
  --exit-code 1 \
  --format table \
  python:3.9-alpine

# 2. Export detailed JSON scan report for audit compliance
trivy image \
  --severity CRITICAL \
  --format json \
  --output /opt/security/trivy-critical-report.json \
  nginx:1.24.0
```

---

## Exercise 2: Software Bill of Materials (SBOM) Generation and CycloneDX Validation
### Scenario
Generate an enterprise Software Bill of Materials (SBOM) for a production microservice to comply with Executive Order 14028. Validate component package licenses and dependencies.

```bash
# 1. Generate SBOM in SPDX JSON format
trivy image --format spdx-json --output /opt/security/sbom-spdx.json node:18-bullseye-slim

# 2. Generate SBOM in CycloneDX format
trivy image --format cyclonedx-json --output /opt/security/sbom-cyclonedx.json node:18-bullseye-slim

# 3. Audit SBOM packages for GPL / AGPL copyleft license violations
cat /opt/security/sbom-cyclonedx.json | jq '.components[] | {name: .name, version: .version, licenses: .licenses}'
```

---

## Exercise 3: Cryptographic Image Signing & In-Cluster Verification with Sigstore Cosign
### Scenario
Generate cryptographic key pairs using Cosign, sign an application image in an OCI registry, and enforce signature validation before deployment.

```bash
# 1. Generate key pair
cosign generate-key-pair

# 2. Sign image digest (always sign by digest, never mutable tag)
IMAGE_DIGEST=$(crane digest myregistry.internal/apps/payment:v1.0.0)
cosign sign --key cosign.key myregistry.internal/apps/payment@${IMAGE_DIGEST}

# 3. Verify signature using public key
cosign verify --key cosign.pub myregistry.internal/apps/payment@${IMAGE_DIGEST}
```

---

## Exercise 4: Enforce Image Verification via OPA Gatekeeper
### Scenario
Deploy an OPA Gatekeeper constraint that validates all container images in namespace `production` originate strictly from approved enterprise registries (`myregistry.internal/`).

```yaml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8strustedregistries
spec:
  crd:
    spec:
      names:
        kind: K8sTrustedRegistries
      validation:
        openAPIV3Schema:
          type: object
          properties:
            allowedPrefixes:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8strustedregistries
        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          image := container.image
          not startswith_any(image, input.parameters.allowedPrefixes)
          msg := sprintf("Image '%v' is not from an approved registry.", [image])
        }
        startswith_any(str, prefixes) {
          prefix := prefixes[_]
          startswith(str, prefix)
        }
---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sTrustedRegistries
metadata:
  name: enforce-trusted-registries
spec:
  match:
    namespaces: ["production"]
  parameters:
    allowedPrefixes: ["myregistry.internal/"]
```

---

## Exercise 5: Hardened Minimal Multi-Stage Dockerfile (Distroless & Non-Root)
### Scenario
Refactor a bloated development Dockerfile containing compilers and package managers into a production-grade, zero-CVE distroless image running as a non-privileged user.

```dockerfile
# Stage 1: Build & Compile
FROM golang:1.27-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -o /app/server .

# Stage 2: Minimal Distroless Runtime
FROM gcr.io/distroless/static-debian12:nonroot
WORKDIR /
COPY --from=builder /app/server /server
USER 65532:65532
ENTRYPOINT ["/server"]
```

---

## Exercise 6: Static Kubernetes Manifest Analysis with Kubesec and Conftest
### Scenario
Scan Helm charts and Kubernetes manifests in CI/CD pipelines before deployment to detect insecure capabilities, missing securityContexts, and privileged execution.

```bash
# 1. Scan manifest with kubesec
kubesec scan /opt/manifests/deployment.yaml

# 2. Evaluate against Rego security policy with conftest
conftest test /opt/manifests/deployment.yaml --policy /opt/policies/
```
