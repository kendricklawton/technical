# PE Domain 5: CI/CD, GitOps & Delivery — Hands-On Labs & Exercises

Six production-grade labs drilling Argo CD GitOps, Argo Rollouts canary analysis, Cosign keyless signing, SBOM generation, and Docker caching.

---

## Lab 1: Deploying the Argo CD App-of-Apps Pattern with ApplicationSets

### Objective
Implement the enterprise App-of-Apps pattern using an Argo CD `ApplicationSet` that dynamically detects directories in a GitOps repository and deploys them to target namespaces with automated drift self-healing.

### Step-by-Step Implementation

1. **Create Directory Structure:**
   ```bash
   mkdir -p /tmp/gitops-fleet/apps/guestbook
   mkdir -p /tmp/gitops-fleet/apps/redis
   ```

2. **Define the ApplicationSet (`applicationset.yaml`):**
   ```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: ApplicationSet
   metadata:
     name: platform-fleet-apps
     namespace: argocd
   spec:
     generators:
       - git:
           repoURL: https://github.com/argoproj/argocd-example-apps.git
           revision: HEAD
           directories:
             - path: guestbook
             - path: helm-guestbook
     template:
       metadata:
         name: "{{path.basename}}"
       spec:
         project: default
         source:
           repoURL: https://github.com/argoproj/argocd-example-apps.git
           targetRevision: HEAD
           path: "{{path}}"
         destination:
           server: https://kubernetes.default.svc
           namespace: "{{path.basename}}"
         syncPolicy:
           automated:
             prune: true
             selfHeal: true
           syncOptions:
             - CreateNamespace=true
   ```

3. **Apply and Verify Synchronization:**
   ```bash
   kubectl apply -f applicationset.yaml
   kubectl get applications -n argocd
   ```

---

## Lab 2: Progressive Canary Delivery with Argo Rollouts & Metric Analysis

### Objective
Configure an `Argo Rollouts` resource featuring step-based traffic shifting (20%, 40%, 60%, 100%) and an `AnalysisTemplate` simulating automatic rollback on synthetic HTTP errors.

### Implementation Manifest (`rollout-canary.yaml`)
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: payments-service-rollout
  namespace: default
spec:
  replicas: 5
  strategy:
    canary:
      analysis:
        templates:
          - templateName: synthetic-smoke-test
        args:
          - name: service-name
            value: payments-service
      steps:
        - setWeight: 20
        - pause: { duration: 30s }
        - setWeight: 40
        - pause: { duration: 30s }
        - setWeight: 60
        - pause: { duration: 15s }
  selector:
    matchLabels:
      app: payments-service
  template:
    metadata:
      labels:
        app: payments-service
    spec:
      containers:
        - name: app
          image: argoproj/rollouts-demo:blue
          ports:
            - containerPort: 8080
---
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: synthetic-smoke-test
  namespace: default
spec:
  metrics:
    - name: synthetic-ping
      interval: 10s
      count: 3
      successCondition: result == "ok"
      provider:
        job:
          spec:
            template:
              spec:
                containers:
                  - name: test-runner
                    image: alpine:latest
                    command: ["/bin/sh", "-c"]
                    args: ["echo 'ok'"]
                restartPolicy: Never
```

---

## Lab 3: Cryptographic Container Image Signing with Cosign

### Objective
Sign a container image using Sigstore Cosign keyless mode (or local key pair) and verify the cryptographic signature in a CI/CD pipeline.

### Step-by-Step Implementation

1. **Generate Local Key Pair for Testing:**
   ```bash
   cosign generate-key-pair
   # Creates cosign.key (private) and cosign.pub (public)
   ```

2. **Build and Tag a Test Image:**
   ```bash
   docker build -t ttl.sh/platform-lab-image:1h - <<EOF
   FROM alpine:latest
   CMD ["echo", "Verified supply chain"]
   EOF
   docker push ttl.sh/platform-lab-image:1h
   ```

3. **Sign the Image with Cosign:**
   ```bash
   cosign sign --key cosign.key ttl.sh/platform-lab-image:1h
   ```

4. **Verify the Signature:**
   ```bash
   cosign verify --key cosign.pub ttl.sh/platform-lab-image:1h
   ```
   Observe the verified JSON payload containing issuer, timestamp, and signature claims!

---

## Lab 4: Generating and Validating Software Bill of Materials (SBOM)

### Objective
Generate a complete Software Bill of Materials (SBOM) in SPDX and CycloneDX formats using `syft`, and scan the SBOM for known CVE vulnerabilities using `trivy`.

### Step-by-Step Implementation

1. **Generate SBOM using Syft:**
   ```bash
   syft packages alpine:3.19 -o spdx-json=sbom.spdx.json
   syft packages alpine:3.19 -o cyclonedx-json=sbom.cyclonedx.json
   ```

2. **Inspect SBOM Packages:**
   ```bash
   jq '.packages[] | {name: .name, version: .versionInfo}' sbom.spdx.json | head -n 20
   ```

3. **Scan the SBOM for Known CVEs:**
   ```bash
   trivy sbom sbom.spdx.json --severity HIGH,CRITICAL
   ```

---

## Lab 5: Building a High-Performance Ephemeral CI Pipeline with Docker Buildx Caching

### Objective
Author a multi-stage Dockerfile and a GitHub Actions workflow snippet utilizing BuildKit cache mounts and GitHub Actions Cache backend (`type=gha`) to reduce build times by 80%.

### Implementation (`.github/workflows/build.yml`)
```yaml
name: High-Performance Container Build

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build and Push with Cache Mounts
        uses: docker/build-push-action@v5
        with:
          context: .
          push: false
          cache-from: type=gha
          cache-to: type=gha,mode=max
          tags: internal-registry.corp/payments-service:latest
```

---

## Lab 6: Structuring a Multi-Environment Kustomize Base and Overlay Pipeline

### Objective
Create a clean directory-based promotion layout using Kustomize with a shared `base/` and distinct `overlays/` (`dev`, `staging`, `prod`) modifying replica counts, resource limits, and environment variables.

### Step-by-Step Implementation

1. **Create Directories:**
   ```bash
   mkdir -p /tmp/kustomize-demo/base
   mkdir -p /tmp/kustomize-demo/overlays/dev
   mkdir -p /tmp/kustomize-demo/overlays/prod
   ```

2. **Base Deployment (`base/deployment.yaml`):**
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: sample-app
   spec:
     replicas: 1
     template:
      spec:
        containers:
          - name: app
            image: nginx:1.25
   ```

3. **Production Overlay Patch (`overlays/prod/kustomization.yaml`):**
   ```yaml
   apiVersion: kustomize.config.k8s.io/v1beta1
   kind: Kustomization
   resources:
     - ../../base
   replicas:
     - name: sample-app
       count: 10
   images:
     - name: nginx
       newTag: 1.25-alpine
   ```

4. **Render Manifest:**
   ```bash
   kustomize build /tmp/kustomize-demo/overlays/prod
   ```
