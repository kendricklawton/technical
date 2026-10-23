# GCP Professional Cloud DevOps Engineer — Master Cheat Sheet

A rapid-fire, production-grade reference covering Google Cloud DevOps services, SRE mathematical formulas, CLI one-liners, and declarative manifests.

---

## 1. SRE Mathematical Foundations & Formulas

### 1.1 SLI, SLO & Error Budget
- **Service Level Indicator (SLI):**
  $$\text{SLI} = \frac{\text{Good Events}}{\text{Total Valid Events}} \times 100\%$$
- **Service Level Objective (SLO):** Target reliability over a rolling compliance window (e.g., $99.9\%$ over 30 days).
- **Error Budget:**
  $$\text{Error Budget} = 100\% - \text{SLO} = 1 - \text{SLO}$$
  - For $99.9\%$ SLO: Error Budget is $0.1\% = 0.001$.
  - In a 30-day window ($43,200$ minutes), budget $= 43,200 \times 0.001 = 43.2 \text{ minutes}$ of total downtime.

### 1.2 Multi-Window Multi-Burn-Rate Alerting
$$\text{Burn Rate} = \frac{\text{Observed Error Rate}}{\text{Allowed Budget Error Rate}} = \frac{1 - \text{Observed SLI}}{1 - \text{SLO}}$$
- **$14.4\times$ Burn Rate:** Consumes $2\%$ of error budget in 1 hour ($14.4 / 720 \approx 0.02$). Requires urgent page (confirmed by 5m short window).
- **$6\times$ Burn Rate:** Consumes $5\%$ of budget in 6 hours ($36 / 720 = 0.05$). Page within 6 hours (confirmed by 30m short window).
- **$1\times$ Burn Rate:** Consumes $100\%$ of budget over the full 30 days ($720\text{ hours}$). Ticket alert.

---

## 2. Essential gcloud CLI Reference

### 2.1 Cloud Build
```bash
# Submit build from local source using cloudbuild.yaml
gcloud builds submit --config=cloudbuild.yaml --substitutions=_IMAGE_TAG="v1.2.0" .

# Create a build trigger linked to GitHub repository
gcloud builds triggers create github \
  --repo-name="core-service" \
  --repo-owner="enterprise" \
  --branch-pattern="^main$" \
  --build-config="cloudbuild.yaml"

# Inspect build logs
gcloud builds log <BUILD_ID> --stream
```

### 2.2 Google Cloud Deploy (Continuous Delivery)
```bash
# Apply delivery pipeline and target configurations
gcloud deploy apply --file=clouddeploy.yaml --region=us-central1

# Create a new release and initiate deployment to the first target (dev)
gcloud deploy releases create release-2026-10-21 \
  --delivery-pipeline=payment-pipeline \
  --region=us-central1 \
  --source=.

# Promote rollout to the next stage (staging -> prod)
gcloud deploy rollouts promote \
  --release=release-2026-10-21 \
  --delivery-pipeline=payment-pipeline \
  --region=us-central1
```

### 2.3 GKE & Workload Identity
```bash
# Configure kubectl credentials for GKE cluster
gcloud container clusters get-credentials prod-cluster --region=us-central1

# Bind Kubernetes ServiceAccount to GCP Service Account via Workload Identity
gcloud iam service-accounts add-iam-policy-binding backend-sa@my-proj.iam.gserviceaccount.com \
  --role="roles/iam.workloadIdentityUser" \
  --member="serviceAccount:my-proj.svc.id.goog[production/backend-ksa]"

# Annotate the Kubernetes ServiceAccount
kubectl annotate serviceaccount backend-ksa -n production \
  iam.gke.io/gcp-service-account=backend-sa@my-proj.iam.gserviceaccount.com
```

### 2.4 Cloud Logging & Monitoring
```bash
# Read error logs from Cloud Run service in the last 15 minutes
gcloud logging read 'resource.type="cloud_run_revision" AND severity>=ERROR AND timestamp >= "2026-10-21T20:00:00Z"' --limit=20

# Create a log-based metric counting HTTP 500 errors
gcloud logging metrics create http_500_count \
  --description="Counts HTTP 500 server errors" \
  --log-filter='resource.type="k8s_container" AND httpRequest.status>=500'
```

---

## 3. Declarative Configuration Blueprints

### 3.1 Cloud Build Manifest (`cloudbuild.yaml`)
```yaml
steps:
  # 1. Run unit tests
  - name: 'golang:1.27-alpine'
    entrypoint: 'go'
    args: ['test', '-v', './...']

  # 2. Build container image using Kaniko with caching
  - name: 'gcr.io/kaniko-project/executor:latest'
    args:
      - '--destination=us-central1-docker.pkg.dev/$PROJECT_ID/apps/service:$COMMIT_SHA'
      - '--cache=true'
      - '--cache-ttl=24h'

  # 3. Create Cloud Deploy release
  - name: 'google/cloud-sdk:latest'
    entrypoint: 'gcloud'
    args:
      - 'deploy'
      - 'releases'
      - 'create'
      - 'rel-$COMMIT_SHA'
      - '--delivery-pipeline=microservice-pipeline'
      - '--region=us-central1'
      - '--images=service=us-central1-docker.pkg.dev/$PROJECT_ID/apps/service:$COMMIT_SHA'

options:
  logging: CLOUD_LOGGING_ONLY
```

### 3.2 Cloud Deploy Pipeline Manifest (`clouddeploy.yaml`)
```yaml
apiVersion: deploy.cloud.google.com/v1
kind: DeliveryPipeline
metadata:
  name: microservice-pipeline
description: Production delivery pipeline with canary verification
serialPipeline:
  stages:
    - targetId: dev
      profiles: [dev]
    - targetId: staging
      profiles: [staging]
    - targetId: prod
      profiles: [prod]
      strategy:
        canary:
          runtimeConfig:
            kubernetes:
              gatewayServiceMesh:
                httpRoute: prod-route
                service: prod-svc
                deployment: prod-deployment
          canaryDeployment:
            percentages: [10, 25, 50]
            verify: true
---
apiVersion: deploy.cloud.google.com/v1
kind: Target
metadata:
  name: prod
gke:
  cluster: projects/my-proj/locations/us-central1/clusters/prod-cluster
```

### 3.3 Skaffold Configuration (`skaffold.yaml`)
```yaml
apiVersion: skaffold/v4beta7
kind: Config
metadata:
  name: microservice
build:
  artifacts:
    - image: service
      context: .
manifests:
  rawYaml:
    - k8s/*.yaml
deploy:
  kubectl: {}
profiles:
  - name: dev
    manifests:
      kustomize:
        paths: [overlays/dev]
  - name: prod
    manifests:
      kustomize:
        paths: [overlays/prod]
```
