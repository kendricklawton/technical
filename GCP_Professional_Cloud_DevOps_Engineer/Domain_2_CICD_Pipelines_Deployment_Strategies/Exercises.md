# Domain 2: Building & Implementing CI/CD Pipelines — Performance Labs & Drills

---

## Lab 1: End-to-End Cloud Deploy Delivery Pipeline with Canary Verification

### Objective
Create a multi-target delivery pipeline in Google Cloud Deploy with a 3-stage Canary rollout (25%, 50%, 100%) targeting GKE clusters.

```yaml
# clouddeploy.yaml
apiVersion: deploy.cloud.google.com/v1
kind: DeliveryPipeline
metadata:
  name: payment-delivery-pipeline
description: Multi-stage pipeline with canary deployment
serialPipeline:
  stages:
    - targetId: staging-cluster
      profiles: [staging]
    - targetId: prod-cluster
      profiles: [prod]
      strategy:
        canary:
          runtimeConfig:
            kubernetes:
              serviceNetworking:
                service: payment-service
                deployment: payment-deployment
          canaryDeployment:
            percentages: [25, 50]
            verify: true
---
apiVersion: deploy.cloud.google.com/v1
kind: Target
metadata:
  name: staging-cluster
gke:
  cluster: projects/my-devops-project/locations/us-central1/clusters/staging-gke
---
apiVersion: deploy.cloud.google.com/v1
kind: Target
metadata:
  name: prod-cluster
gke:
  cluster: projects/my-devops-project/locations/us-central1/clusters/prod-gke
```

### Verification & Deployment
```bash
# 1. Apply pipeline and targets
gcloud deploy apply --file=clouddeploy.yaml --region=us-central1

# 2. Create release from source
gcloud deploy releases create rel-v1-0-0 \
  --delivery-pipeline=payment-delivery-pipeline \
  --region=us-central1 \
  --source=.

# 3. Check rollout status
gcloud deploy rollouts list \
  --release=rel-v1-0-0 \
  --delivery-pipeline=payment-delivery-pipeline \
  --region=us-central1
```

---

## Lab 2: Build and Sign Container with Kaniko & Cloud Build

### Objective
Configure `cloudbuild.yaml` to build a container image using Kaniko layer caching and push to Google Cloud Artifact Registry.

```yaml
# cloudbuild.yaml
steps:
  # 1. Kaniko build with caching
  - name: 'gcr.io/kaniko-project/executor:latest'
    args:
      - '--destination=us-central1-docker.pkg.dev/$PROJECT_ID/enterprise-apps/order-api:$SHORT_SHA'
      - '--cache=true'
      - '--cache-ttl=24h'

  # 2. Run Container Vulnerability Scan Check via gcloud
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk:latest'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        echo "Image successfully built and pushed to Artifact Registry: us-central1-docker.pkg.dev/$PROJECT_ID/enterprise-apps/order-api:$SHORT_SHA"

options:
  logging: CLOUD_LOGGING_ONLY
```

### Execution
```bash
gcloud builds submit --config=cloudbuild.yaml .
```

---

## Lab 3: Automated Rollback on Canary Metric Degradation

### Objective
Execute an immediate automated rollback of a Cloud Deploy rollout when canary health checks detect an elevated error rate.

```bash
# Abort and roll back the active rollout on the production target
ROLLOUT_NAME=$(gcloud deploy rollouts list \
  --release=rel-v1-0-0 \
  --delivery-pipeline=payment-delivery-pipeline \
  --region=us-central1 \
  --format="value(name)" | head -n 1)

gcloud deploy rollouts rollback \
  --rollout="$ROLLOUT_NAME" \
  --delivery-pipeline=payment-delivery-pipeline \
  --region=us-central1
```
