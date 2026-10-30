# GCP Professional Cloud DevOps Engineer — Master Cheat Sheet

> [!IMPORTANT]
> **Exam Strategy:** The GCP DevOps exam tests your mastery of SRE principles (SLI/SLO/SLA math, multi-window burn rate alerts, toil reduction), continuous build and progressive delivery (Cloud Build private pools, Skaffold profiles, Cloud Deploy canaries and automated rollbacks), full-stack observability (GMP PromQL, MQL queries, structured JSON logging, distributed tracing), and supply chain security (Binary Authorization, Workload Identity Federation). Remember: **mitigate before investigating**, **eliminate static service account keys**, and **prefer blameless systemic solutions over human blame**.

---

## 1. SRE Mathematical Foundations & Formulas

### 1.1 SLI, SLO & Error Budget Formulas

* **Service Level Indicator (SLI):**
  $$\text{SLI} = \frac{\sum \text{Good Events}}{\sum \text{Total Valid Events}} \times 100\%$$
* **Service Level Objective (SLO):** Target reliability over a rolling compliance window (e.g., $99.9\%$ over 30 days).
* **Error Budget:**
  $$\text{Error Budget} = 100\% - \text{SLO} = 1 - \text{SLO}$$
* **Allowed Downtime per Rolling 30-Day Window ($43,200\text{ minutes}$):**
  * $99\%$ SLO: $1\% \text{ budget} = 432 \text{ minutes } (7.2 \text{ hours})$
  * $99.9\%$ SLO: $0.1\% \text{ budget} = 43.2 \text{ minutes}$
  * $99.99\%$ SLO: $0.01\% \text{ budget} = 4.32 \text{ minutes}$
  * $99.999\%$ SLO: $0.001\% \text{ budget} = 25.9 \text{ seconds}$

### 1.2 Multi-Window Multi-Burn-Rate Alerting

$$\text{Burn Rate} = \frac{\text{Observed Error Rate}}{\text{Allowed Budget Error Rate}} = \frac{1 - \text{Observed SLI}}{1 - \text{SLO}}$$

* **$14.4\times$ Burn Rate:** Consumes **$2\%$ of 30-day budget in 1 hour**. Requires immediate page. Confirmed with a 5-minute short window to eliminate transient false positives.
* **$6\times$ Burn Rate:** Consumes **$5\%$ of 30-day budget in 6 hours**. Requires immediate page. Confirmed with a 30-minute short window.
* **$1\times$ Burn Rate:** Consumes **$100\%$ of budget over exactly 30 days** ($720\text{ hours}$). Generates a non-paging ticket alert.

---

## 2. Essential gcloud CLI Reference

### 2.1 Cloud Build

```bash
# Submit build from local directory using private pool
gcloud builds submit --config=cloudbuild.yaml \
    --worker-pool=projects/PROJ/locations/us-central1/workerPools/prod-private-pool \
    --substitutions=_IMAGE_TAG="v2.1.0" .

# Stream build logs live in terminal
gcloud builds log <BUILD_ID> --stream

# List failed builds over the past 24 hours
gcloud builds list --filter="status=FAILURE AND createTime>-P1D"
```

### 2.2 Google Cloud Deploy

```bash
# Apply declarative delivery pipeline and target configurations
gcloud deploy apply --file=clouddeploy.yaml --region=us-central1

# Create an immutable release from a Skaffold source configuration
gcloud deploy releases create release-2026-10-21 \
    --delivery-pipeline=order-pipeline \
    --region=us-central1 \
    --images=app-image=us-central1-docker.pkg.dev/PROJ/app-repo/web:v2.1.0

# Promote release from staging to production
gcloud deploy rollouts promote \
    --release=release-2026-10-21 \
    --delivery-pipeline=order-pipeline \
    --region=us-central1

# Execute emergency rollback to previous stable release
gcloud deploy rollouts rollback \
    --delivery-pipeline=order-pipeline \
    --region=us-central1 \
    --release=release-2026-10-21
```

### 2.3 Artifact Registry

```bash
# Create Docker repository with immutable tags
gcloud artifacts repositories create app-repo \
    --repository-format=docker \
    --location=us-central1 \
    --description="Production OCI container images" \
    --immutable-tags

# Run on-demand vulnerability scan on a local container image
gcloud artifacts docker images scan us-central1-docker.pkg.dev/PROJ/app-repo/web:v2.1.0 \
    --format="value(response.scan)"
```

### 2.4 Cloud Logging & Monitoring

```bash
# Query error logs with HTTP status >= 500 in the past 1 hour
gcloud logging read 'resource.type="k8s_container" AND httpRequest.status>=500 AND timestamp>="2026-10-26T22:00:00Z"' \
    --limit=50 --format=json

# Create aggregated organization-level audit log sink
gcloud logging sinks create org-audit-sink \
    bigquery.googleapis.com/projects/security-lake/datasets/audit_logs \
    --organization=123456789012 \
    --include-children \
    --log-filter='logName:"cloudaudit.googleapis.com"'
```

---

## 3. Declarative CRD & Configuration Cheat Sheet

### 3.1 Cloud Build Parallelism & Secret Manager (`cloudbuild.yaml`)

```yaml
steps:
  - name: 'golang:1.27-alpine'
    id: 'unit-tests'
    entrypoint: 'go'
    args: ['test', '-v', './...']
    waitFor: ['-'] # Runs in parallel with db-dryrun

  - name: 'migrate/migrate'
    id: 'db-dryrun'
    entrypoint: 'sh'
    args: ['-c', '/migrate -database "$$DB_URI" version']
    secretEnv: ['DB_URI']
    waitFor: ['-']

  - name: 'gcr.io/kaniko-project/executor:latest'
    id: 'kaniko-build'
    args: ['--destination=us-central1-docker.pkg.dev/$PROJECT_ID/repo/app:$COMMIT_SHA', '--cache=true']
    waitFor: ['unit-tests', 'db-dryrun']

availableSecrets:
  secretManager:
    - versionName: projects/$PROJECT_ID/secrets/db-conn/versions/latest
      env: 'DB_URI'
```

### 3.2 Google Cloud Managed Service for Prometheus (`PodMonitoring`)

```yaml
apiVersion: monitoring.googleapis.com/v1
kind: PodMonitoring
metadata:
  name: order-api-monitor
  namespace: production
spec:
  selector:
    matchLabels:
      app: order-api
  endpoints:
    - port: metrics
      path: /metrics
      interval: 15s
```

### 3.3 Binary Authorization Enforcement Policy

```yaml
admissionWhitelistPatterns:
  - matchedPattern: "gcr.io/google-containers/*"
globalPolicyEvaluationMode: ENABLE
defaultAdmissionRule:
  evaluationMode: ALWAYS_DENY
  enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
clusterAdmissionRules:
  us-central1.production-cluster:
    evaluationMode: REQUIRE_ATTESTATION
    enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
    requireAttestationsBy:
      - "projects/prod-security/attestors/security-qa-attestor"
```

---

## 4. Top 15 High-Probability GCP DevOps Exam Traps

1. **Mitigate First, Investigate Later:** During an active SEV-1 incident, the immediate correct answer is **always** mitigation (roll back release, shed load, drain traffic), **never** debugging code or analyzing stack traces during user downtime.
2. **Static Service Account Keys:** Storing a downloaded service account JSON key inside GitHub Actions, GitLab CI, or a Kubernetes secret is an **immediate distractor**. The correct answer is **Workload Identity Federation (WIF)**.
3. **Cloud Build Private VPC Access:** When Cloud Build needs to reach private GKE control planes or on-premises Git repos, the answer is **Cloud Build Private Pools peered via Private Services Access (PSA)**.
4. **Log Storage Cost Optimization:** To prevent high-volume debug logs or health check polling from driving up bills, use **Log Exclusion Filters** on the `_Default` Log Router sink.
5. **Burn Rate Window Pairing:** A high burn rate (e.g., $14.4\times$) requires **two simultaneous windows**: a long window (1 hour) to measure budget impact, and a short window (5 minutes) to confirm the issue is ongoing and avoid false alerts on transient spikes.
6. **Binary Authorization Rollout:** When enabling Binary Authorization on an existing production cluster, start with `enforcementMode: DRYRUN_AUDIT_LOG_ONLY` to identify unsigned images without breaking active deployments.
7. **Breakglass in Binary Authorization:** When an emergency hotfix must deploy during a catastrophic outage where attestation pipelines are broken, use the **`breakglass: true` pod annotation**. It bypasses admission and emits a high-priority audit log.
8. **Cloud Deploy Release Immutability:** Once a Release is created in Cloud Deploy, its manifests and image digests are **completely immutable**. Any change to code or configuration requires creating a new Release.
9. **GKE Workload Identity Binding:** Requires binding the Kubernetes Service Account (KSA) to the Google Service Account (GSA) using `roles/iam.workloadIdentityUser`, AND annotating the KSA with `iam.gke.io/gcp-service-account`.
10. **Error Budget Policy Enforcement:** When a product team completely burns its error budget ($100\%$ spent), enforce a **feature release freeze** and redirect 100% of engineering capacity to reliability and technical debt reduction.
11. **Blameless Postmortems:** Postmortems must focus on missing systemic guardrails, inadequate testing, or poor tooling. Naming individual engineers or proposing punitive retraining is strictly an anti-pattern.
12. **Cloud Profiler Overhead:** Cloud Profiler runs continuously in production with **$<5\%$ overhead** using statistical sampling; it does not require stopping containers or running in staging environments.
13. **Artifact Registry Cleanup:** Use **Cleanup Policies** to automatically delete untagged container images older than $N$ days while keeping the most recent $N$ tagged release versions.
14. **Thundering Herd Retries:** When services retry failed database requests, standard exponential backoff causes synchronized waves of traffic. Always add **randomized jitter** to spread retry distributions.
15. **Dataplane V2 Network Policies:** GKE Dataplane V2 uses eBPF (Cilium) to enforce network policies directly in kernel sockets, eliminating legacy `iptables` performance degradation at scale.
