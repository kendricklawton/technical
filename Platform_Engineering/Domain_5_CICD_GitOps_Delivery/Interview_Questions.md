# PE Domain 5: CI/CD, GitOps & Delivery — 25 Core Interview Questions

Master these questions across Technical Screening, IaC/GitOps Deep Dive (Mock 4), and Staff Architecture rounds.

---

## Tier 1: Core / Screening Questions (Questions 1–10)

### Q1: What is the core difference between Push and Pull deployment models?
- **Short Answer:** In the Push model, external CI systems push changes to target clusters, requiring external cluster credentials. In the Pull model (GitOps), an in-cluster agent monitors Git and pulls changes inward, eliminating external credentials and automatically correcting drift.
- **Deep Dive:** Push models create security vulnerabilities: if CI is breached, all clusters are exposed. Pull models (Argo CD, Flux) operate behind the firewall, continuously reconciling desired Git state against live cluster state.
- **Trade-offs & Realities:** Pull models require running in-cluster controllers and can introduce minor reconciliation latency compared to direct push execution.
- **Follow-up Probes:** "What happens when someone runs `kubectl delete` in a GitOps-managed cluster?" (The GitOps agent detects drift and automatically recreates the resource).

### Q2: What is the difference between Blue/Green and Canary deployments?
- **Short Answer:** Blue/Green switches 100% of traffic instantly between two identical environments. Canary gradually shifts traffic (e.g., 5%, 25%, 100%) to a new version while evaluating real-time telemetry metrics.
- **Deep Dive:** Blue/Green provides instant rollback (re-switch traffic) but temporarily doubles compute costs. Canary minimizes blast radius by exposing only a tiny fraction of users to potential bugs.
- **Trade-offs & Realities:** Canary requires active traffic and robust observability (Prometheus error metrics) to make automated promote/abort decisions.
- **Follow-up Probes:** "What tool enables automated canary rollouts in Kubernetes?" (Argo Rollouts or Flagger).

### Q3: Why is Actions Runner Controller (ARC) preferred over static CI runner VMs?
- **Short Answer:** ARC provisions ephemeral runner pods on demand inside Kubernetes, ensuring every job executes in a clean, isolated environment that is destroyed upon completion.
- **Deep Dive:** Static runner VMs accumulate dirty state, uncleaned Docker layers, and leaked credentials. ARC scales runners to zero when idle and scales up during business hours, optimizing compute costs.
- **Trade-offs & Realities:** Ephemeral runners require cold-start time to pull images unless warm pools or cached base images are maintained.
- **Follow-up Probes:** "How does ARC scale runner pods?" (Using Kubernetes metrics or webhook event listeners via KEDA).

### Q4: Explain the difference between Helm and Kustomize.
- **Short Answer:** Helm is a templated package manager that parameterizes manifests using Go templates and `.Values`. Kustomize is a template-free overlay engine that patches raw declarative YAML without syntax modification.
- **Deep Dive:** Helm is ideal for distributing third-party off-the-shelf software across heterogeneous organizations. Kustomize is superior for internal microservice promotion pipelines across `dev`, `staging`, and `prod`.
- **Trade-offs & Realities:** Helm templates can turn into unreadable YAML soup. Kustomize overlays can duplicate patch boilerplate if directory hierarchies become too deep.
- **Follow-up Probes:** "Can Argo CD use Helm and Kustomize together?" (Yes, Argo CD supports Kustomize with Helm chart inflators).

### Q5: What is a Software Bill of Materials (SBOM)?
- **Short Answer:** A structured, machine-readable inventory of all libraries, dependencies, packages, and metadata embedded inside a software container or binary.
- **Deep Dive:** SBOMs (standardized in SPDX or CycloneDX formats) enable automated compliance and vulnerability analysis. When a new CVE is announced, platform teams query the SBOM database to instantly locate affected services.
- **Trade-offs & Realities:** An SBOM is a snapshot in time; it must be continuously scanned against updated CVE vulnerability databases.
- **Follow-up Probes:** "What tool generates SBOMs from container images?" (`syft` or `trivy`).

### Q6: What is Sigstore Cosign, and what does "keyless signing" mean?
- **Short Answer:** Cosign is a container image signing tool. "Keyless signing" uses ephemeral certificates tied to OIDC identity (e.g., GitHub Actions workflow) rather than long-lived private keys.
- **Deep Dive:** The signer exchanges an OIDC token with Fulcio (Certificate Authority) for a short-lived certificate, signs the image, and logs the transaction in Rekor (transparency log). Verification proves who built the image without managing key rotation.
- **Trade-offs & Realities:** Keyless signing requires outbound internet access to Sigstore public infrastructure (or deploying a self-hosted private Sigstore stack).
- **Follow-up Probes:** "Where is the image signature stored?" (In the container registry alongside the image tag as an OCI artifact).

### Q7: Why is "Branch-per-Environment" considered an anti-pattern in GitOps?
- **Short Answer:** Because branches naturally diverge over time through cherry-picks and missed backports, creating permanent Git drift and defeating environment reproducibility.
- **Deep Dive:** Merging between environment branches (`dev` -> `staging` -> `prod`) introduces high merge conflict risks. Best practice uses a single `main` trunk with **Directory-per-Environment** overlays.
- **Trade-offs & Realities:** Directory-per-environment requires automated tooling (like Renovate or Image Updater) to commit image bumps across directories.
- **Follow-up Probes:** "How do you promote an image in a directory-based GitOps repo?" (Update the image tag in `overlays/prod/kustomization.yaml` via a pull request).

### Q8: What does `selfHeal: true` mean in an Argo CD application sync policy?
- **Short Answer:** It instructs Argo CD to automatically overwrite any out-of-band manual changes made directly to live cluster resources to ensure they strictly match the Git repository.
- **Deep Dive:** If an engineer deletes a pod or manually edits a configmap using `kubectl`, Argo CD detects the divergence and immediately restores the state declared in Git.
- **Trade-offs & Realities:** Can interfere with emergency incident debugging if engineers need to temporarily patch a live pod; requires temporarily disabling sync during hotfix investigations.
- **Follow-up Probes:** "How do you temporarily pause self-healing during an outage?" (`argocd app set <app> --self-heal=false` or suspend in the UI).

### Q9: What is BuildKit in Docker, and why is it important for CI?
- **Short Answer:** The modern, high-performance container build subsystem in Docker featuring parallel stage compilation, secret mounts, and persistent cache mounts.
- **Deep Dive:** BuildKit constructs an execution DAG of Dockerfile stages, executing independent stages concurrently. It supports `--mount=type=cache`, allowing compiler caches (Go, Rust, npm) to persist between builds.
- **Trade-offs & Realities:** Requires enabling `DOCKER_BUILDKIT=1` or using `docker buildx`.
- **Follow-up Probes:** "How does BuildKit handle secrets during image builds?" (`--mount=type=secret` mounts secrets in memory without writing them into image layers).

### Q10: What is the SLSA framework?
- **Short Answer:** Supply-chain Levels for Software Artifacts (SLSA): a security framework defining standards (Levels 1–4) to prevent tampering, ensure integrity, and secure software build pipelines.
- **Deep Dive:** Enforces automated builds, non-falsifiable signed provenance, isolated ephemeral build environments, and two-party code reviews.
- **Trade-offs & Realities:** Reaching SLSA Level 3/4 requires sophisticated CI infrastructure and strict cryptographic signing automation.
- **Follow-up Probes:** "What is build provenance?" (Signed metadata detailing the source repo, commit SHA, build runner, and parameters used to build an artifact).

---

## Tier 2: Deep Dive & Mid-Senior Questions (Questions 11–20)

### Q11: Walk through the internal reconciliation loop of Argo CD.
- **Short Answer:** The repo-server fetches the target revision from Git; the application-controller queries the live cluster state, computes a deep diff, determines sync status (Synced/OutOfSync), and executes reconciliation if automated sync is enabled.
- **Deep Dive:** The controller maintains an in-memory cache of cluster resources populated by Informers. It evaluates manifests against live resources, identifies resource additions, updates, or prunes, applies changes in configured Sync Phases/Waves, and updates application health (`Healthy`, `Degraded`, `Progressing`).
- **Trade-offs & Realities:** High numbers of applications (thousands) can cause Git API rate-limiting or high CPU utilization on `argocd-repo-server`.
- **Follow-up Probes:** "How do you prevent Git rate-limiting in large Argo CD installations?" (Configure webhooks on Git push events rather than relying on aggressive polling).

### Q12: How do you design an automated, progressive canary deployment pipeline using Argo Rollouts and Prometheus?
- **Short Answer:** Replace the standard Kubernetes `Deployment` with a `Rollout` object referencing an `AnalysisTemplate` that queries Prometheus for HTTP error rate and latency at defined step intervals.
- **Deep Dive:** Steps:
  1. Rollout shifts 10% traffic to canary ReplicaSet.
  2. Pause for 5 minutes while `AnalysisTemplate` runs Prometheus queries every 30 seconds (`rate(http_requests_total{status=~"5.."}[1m]) / rate(...)`).
  3. If error rate is <0.5%, advance to 25%, then 50%, then 100%.
  4. If error rate exceeds threshold, controller aborts rollout, routes 100% traffic back to stable, and emits a notification.
- **Trade-offs & Realities:** Low-traffic services may not generate enough requests for statistical metric significance; requires synthetic smoke tests or longer pause windows.
- **Follow-up Probes:** "How does Argo Rollouts shift traffic at the network level?" (By integrating directly with Ingress, Gateway API, Envoy, or Service Mesh).

### Q13: What are Sync Waves and Sync Phases in Argo CD?
- **Short Answer:** Mechanisms to enforce explicit ordering of resource deployment during a GitOps sync (e.g., ensuring CRDs and namespaces exist before application pods).
- **Deep Dive:** Configured via annotations: `argocd.argoproj.io/sync-wave: "1"`. Argo CD executes waves in ascending numerical order (Wave -1 -> Wave 0 -> Wave 1). It waits for all resources in a wave to reach a `Healthy` state before initiating the next wave.
- **Trade-offs & Realities:** Overusing sync waves slows down deployment duration; use only where true initialization dependencies exist (like database schema migrations).
- **Follow-up Probes:** "What is a PreSync hook in Argo CD?" (A Kubernetes Job executed before main manifests are synced, commonly used for database migrations).

### Q14: How does Actions Runner Controller (ARC) autoscaling work under the hood?
- **Short Answer:** ARC deploys an `AutoScalingRunnerSet` that communicates with GitHub via HTTPS long-polling. When workflow jobs are queued, ARC scales Kubernetes runner pods to match queue depth.
- **Deep Dive:** Modern ARC uses the GitHub Actions Scale Set listener. A dedicated listener pod receives job allocation messages directly from GitHub's backend and updates the replica count of the runner pod statefulset/deployment, provisioning pods just in time.
- **Trade-offs & Realities:** Ephemeral pod startup latency (pulling images, initializing container runtime) adds 15–30 seconds to job start times.
- **Follow-up Probes:** "How do you minimize runner cold-start latency?" (Pre-pull runner container images onto worker nodes using a DaemonSet or pre-baked AMIs).

### Q15: How do you implement end-to-end container image verification in Kubernetes using Kyverno and Cosign?
- **Short Answer:** Deploy Kyverno in the cluster with a `ClusterPolicy` that intercepts Pod creation requests, verifies the cryptographic signature of the image against a public key or Fulcio OIDC issuer, and rejects unverified pods.
- **Deep Dive:** Kyverno policy:
  ```yaml
  apiVersion: kyverno.io/v1
  kind: ClusterPolicy
  metadata:
    name: check-image-signature
  spec:
    validationFailureAction: Enforce
    rules:
      - name: verify-signature
        match:
          resources:
            kinds: [Pod]
        verifyImages:
          - imageReferences: ["internal-registry.corp/*"]
            attestors:
              - entries:
                  - keyless:
                      issuer: "https://token.actions.githubusercontent.com"
                      subject: "https://github.com/my-org/*"
  ```
- **Trade-offs & Realities:** If the external transparency log (Rekor) or registry is unreachable, admission webhooks can block valid deployments unless caching is configured.
- **Follow-up Probes:** "What happens if an attacker pushes an unsigned image directly to the registry?" (The image is pushed, but Kyverno blocks it from ever running in the Kubernetes cluster).

### Q16: Compare Monorepo vs Polyrepo architectures from a Platform Engineering perspective.
- **Short Answer:** Monorepos simplify cross-service refactorings, atomic commits, and centralized tooling, but require sophisticated change detection and build caching (Bazel/Nx). Polyrepos provide isolated blast radiuses and independent CI pipelines, but make cross-service changes and dependency synchronization difficult.
- **Deep Dive:**
  - *Monorepo:* 1 repo, unified tooling, easier code discovery, shared CI configs. Weakness: massive Git clone sizes, high CI concurrency, complex permissions.
  - *Polyrepo:* Clear ownership boundaries per team, fast small clones, independent release cadences. Weakness: dependency hell, duplicated CI configs, fleet-wide migrations require opening hundreds of PRs.
- **Trade-offs & Realities:** Monorepos require dedicated platform investment in build acceleration tooling; unassisted monorepos quickly collapse under CI wait times.
- **Follow-up Probes:** "What is a Sparse Checkout in Git?" (A Git feature allowing a CI runner or developer to clone only a specific subdirectory of a massive monorepo).

### Q17: How do you design an automated image promotion pipeline between Dev, Staging, and Production GitOps environments?
- **Short Answer:** CI builds, tests, and signs the image, tagging it with the Git commit SHA. CI pushes a PR updating the image tag in `environments/dev/`. After automated smoke tests pass, a deployment bot automatically promotes the tag by opening PRs into `staging/` and subsequently `production/` following approval gates.
- **Deep Dive:** Avoid rebuilding the image between environments. The exact same cryptographic container image digest (`sha256:...`) validated in `dev` must be promoted to `staging` and `prod`, guaranteeing that what was tested is what runs in production.
- **Trade-offs & Realities:** Rebuilding images for each environment introduces non-deterministic variations and invalidates previous test verification.
- **Follow-up Probes:** "How do you handle environment-specific configuration without rebuilding the image?" (Inject config via ConfigMaps, Secrets, and environment variables).

### Q18: What is the purpose of Traffic Shadowing (Mirroring), and how is it implemented in modern delivery?
- **Short Answer:** Duplicating real production traffic and sending a copy asynchronously to a candidate service without impacting real user responses, allowing verification of performance and error rates under real-world load.
- **Deep Dive:** Configured at the Ingress / Service Mesh layer (Envoy, Istio, or Gateway API). The proxy forks incoming HTTP requests, sending the primary request to stable v1 and an asynchronous clone to candidate v2. Responses from v2 are discarded.
- **Trade-offs & Realities:** Dangerous for stateful mutating requests (e.g., POST `/v1/payments`); shadowing should be restricted to read-only or idempotent endpoints unless mock backends are configured.
- **Follow-up Probes:** "Why is shadowing better than synthetic load testing?" (Synthetic traffic rarely captures real-world edge cases, payload variations, and burst patterns).

### Q19: How do you implement remote build caching in GitHub Actions using Docker Buildx?
- **Short Answer:** Use `docker/build-push-action` configured with `cache-from: type=gha` and `cache-to: type=gha,mode=max`.
- **Deep Dive:** This utilizes GitHub's native caching API to store intermediate BuildKit layer caches. `mode=max` caches intermediate build stages, not just the final stage. When subsequent builds run, unchanged layers are downloaded from the cache, turning 10-minute builds into 30-second operations.
- **Trade-offs & Realities:** GitHub Actions enforces a 10GB cache limit per repository; for massive projects, use registry-backed cache (`type=registry`) pointing to an internal OCI registry.
- **Follow-up Probes:** "What happens when the GitHub Actions cache limit is reached?" (GitHub evicts the least recently used cache entries).

### Q20: Explain the difference between `Kustomize` strategic merge patches and JSON 6902 patches.
- **Short Answer:** Strategic merge patches resemble standard Kubernetes YAML and merge keys intelligently based on schema. JSON 6902 patches are explicit lists of operations (`op: replace`, `op: add`, `op: remove`) targeting exact JSON paths.
- **Deep Dive:** Strategic merge allows intuitive overriding of pod fields:
  ```yaml
  spec:
    template:
      spec:
        containers:
          - name: app
            image: new-image:tag
  ```
  JSON 6902 allows precise array manipulation:
  ```yaml
  - op: replace
    path: /spec/template/spec/containers/0/image
    value: new-image:tag
  ```
- **Trade-offs & Realities:** Strategic merge is more readable; JSON 6902 is required when modifying Custom Resources or deleting specific array elements.
- **Follow-up Probes:** "Why can't strategic merge patch all CRDs?" (Because CRDs lack in-tree schema patch strategies unless OpenAPI schemas define patch keys).

---

## Tier 3: Staff & Principal Architect Scenarios (Questions 21–25)

### Q21: Architect an enterprise CI/CD and delivery platform serving 3,000 developers, 800 microservices, and 50 Kubernetes clusters with strict SOC2 compliance.
- **Candidate Framework:**
  - *CI Architecture:* Deploy Actions Runner Controller (ARC) in dedicated Kubernetes runner clusters; scale ephemeral runners with RAM-backed build caches and OIDC AWS/GCP authentication.
  - *Supply Chain Security:* Mandate SLSA Level 3: enforce container image builds via certified reusable workflows, sign images with keyless Cosign, generate SBOMs via Syft, and verify signatures via Kyverno admission controllers on all clusters.
  - *Delivery Plane:* Deploy Argo CD in a hub-and-spoke architecture (central management control plane managing remote spoke clusters via Cluster API and ApplicationSets).
  - *Environment Promotion:* Directory-per-environment repository with automated image-updater bots.
  - *Progressive Delivery:* Standardize on Argo Rollouts with automated canary analysis based on Prometheus latency and error budgets.
  - *Compliance & Audit:* Git commit history acts as the immutable audit trail; automated branch protection rules enforce peer reviews and passing CI checks.
- **Red Flags:** Recommending shared static runner VMs or allowing manual `kubectl apply` access in production.

### Q22: A developer pushes a bad change that bypasses canary analysis and triggers an immediate production-wide outage. How do you lead incident recovery and platform prevention?
- **Candidate Framework:**
  - *Immediate Mitigation:* Execute instant GitOps rollback (`git revert <bad_commit>` or trigger Argo CD manual rollback to previous sync commit). Ensure traffic shifts back to the healthy ReplicaSet immediately.
  - *Root Cause Analysis:* Why did canary analysis fail to catch it? Was the error threshold misconfigured? Was the canary traffic duration too short? Did the bug affect an un-monitored asynchronous queue?
  - *Platform Hardening:*
    - Enhance `AnalysisTemplate` to evaluate both synchronous HTTP 5xx rates and asynchronous consumer error metrics.
    - Implement mandatory minimum pause windows (e.g., 10 minutes at 10% traffic).
    - Introduce synthetic automated smoke tests that execute active transactions against the canary pods during rollout.
- **Red Flags:** Blaming the developer instead of addressing the failure of the platform verification guardrails.

### Q23: How do you solve the "GitOps Secret Management" dilemma without committing plaintext secrets to Git?
- **Candidate Framework:**
  - *The Dilemma:* Git is the source of truth, but storing plaintext secrets in Git violates all security standards.
  - *Evaluated Patterns:*
    1. *Sealed Secrets (Bitnami):* Encrypts secrets with cluster public key; safe for Git, but re-keying across multiple clusters is painful.
    2. *Mozilla SOPS:* Encrypts YAML values using KMS/PGP; requires managing developer decryption keys.
    3. *External Secrets Operator (Recommended):* Commit only a declarative `ExternalSecret` manifest to Git that references an external secure secret store (HashiCorp Vault, AWS Secrets Manager). The in-cluster ESO controller fetches the secret dynamically and generates native Kubernetes Secrets in real time.
  - *Conclusion:* External Secrets Operator decouples secret storage from GitOps, preserving centralized secret rotation, audit logging, and principle of least privilege.
- **Red Flags:** Suggesting `.gitignore` on secret files in a GitOps repository (violates declarative reproducibility).

### Q24: What is the architectural role of ephemeral preview environments, and how do you design them at enterprise scale?
- **Candidate Framework:**
  - *Value:* Spin up a temporary, production-like environment for every open Pull Request, allowing product managers, QA, and automated integration tests to validate changes before merging.
  - *Architecture:*
    - *Compute Isolation:* Use vCluster or dedicated ephemeral namespaces dynamically provisioned by an Argo CD ApplicationSet watching PR branches.
    - *Routing:* Gateway API or Ingress controller dynamically assigns wildcard hostnames (`pr-123.preview.company.com`).
    - *Data Layer:* Lightweight mock databases or ephemeral DB clones created via containerized testcontainers or AWS RDS snapshots.
    - *Lifecycle & Teardown:* Automated TTL controller (e.g., Kube-downscaler) terminates environments after 4 hours of inactivity or when the PR is closed/merged to eliminate idle cloud waste.
- **Red Flags:** Spinning up full, heavyweight AWS VPCs and RDS clusters for every PR without auto-teardown controls.

### Q25: How do you design an automated migration strategy to transition 400 legacy Jenkins pipelines to a modern GitHub Actions and Argo CD GitOps model?
- **Candidate Framework:**
  - *Discovery & Taxonomy:* Catalog all 400 Jenkins pipelines; categorize into 3 standard archetypes (Java/Spring, Go microservice, Frontend web app).
  - *Paved Path Creation:* Author 3 standardized, reusable GitHub Actions workflows (`.github/workflows/standard-go.yml@v2`) encapsulating linting, testing, Docker builds, and SBOM generation.
  - *Lighthouse Migration:* Partner with 2 influential teams in *Collaboration* mode. Migrate their pipelines, validate the GitOps flow, and document time-savings.
  - *Automated Fleet PRs:* Write a migration script that scans Jenkinsfiles, generates equivalent 10-line GitHub Actions caller workflows, creates the Kustomize GitOps directory, and submits automated PRs to application repositories.
  - *Deprecation Campaign:* Announce Jenkins sunset window; conduct planned brownouts (disabling Jenkins for 2 hours during daytime) to surface un-migrated jobs; permanently decommission Jenkins.
- **Red Flags:** Attempting to manually rewrite 400 pipelines one by one without reusable workflows.
