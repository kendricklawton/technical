# Domain 2: Building & Implementing CI/CD Pipelines — Quiz 3

**Exam:** Google Cloud Certified: Professional Cloud DevOps Engineer  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
Your team deploys microservices to GKE using Google Cloud Deploy. A new release has entered a Canary phase with 25% traffic. During automated verification, synthetic requests detect an increase in latency beyond the acceptable SLO. Which command allows the team to terminate the rollout immediately and restore 100% of traffic to the stable version?
- A. `gcloud deploy rollouts terminate`
- B. `gcloud deploy rollouts rollback`
- C. `kubectl delete pod --all`
- D. `gcloud container clusters restart`

---

### Question 2
In Cloud Build, what is the advantage of using **Cloud Build Private Pools** over the default shared worker pool?
- A. Private Pools run on free hardware.
- B. Private Pools allow builds to access resources inside a private VPC network (such as internal Artifact Registry, private GKE master endpoints, or on-premises systems via Interconnect) with custom machine types and isolated network perimeters.
- C. Private Pools do not require IAM permissions.
- D. Private Pools only support Windows containers.

---

### Question 3
An application team uses GitOps with Google Cloud **Config Sync**. An engineer directly edits a production Kubernetes Service manifest using `kubectl edit svc payment-service` to change a port mapping. What will Config Sync do within minutes?
- A. Update the remote Git repository to match the manual change.
- B. Detect the drift between the live cluster state and the Git source of truth, and automatically revert the live Service back to the configuration declared in Git.
- C. Crash the Kubernetes cluster.
- D. Send an email to Google Cloud support.

---

### Question 4
You need to deploy a containerized application to **Cloud Run** using Google Cloud Deploy. In addition to GKE, what runtime target configuration must be declared in `clouddeploy.yaml`?
- A. `run: { location: "projects/my-proj/locations/us-central1" }`
- B. `computeEngine: { zone: "us-central1-a" }`
- C. `appEngine: { env: "standard" }`
- D. `cloudFunctions: { runtime: "nodejs20" }`

---

### Question 5
How does **Binary Authorization** interact with Cloud Build in a secure software delivery lifecycle?
- A. Cloud Build converts code into binary assembly.
- B. During the build pipeline, Cloud Build generates an image digest, signs it with an asymmetric Cloud KMS key via an Attestor, and writes an attestation to Grafeas/Artifact Analysis; GKE then verifies this attestation before admitting the pod.
- C. Cloud Build encrypts all source code files.
- D. Binary Authorization deletes old images.

---

### Question 6
You have a multi-tier microservice application where the database schema must be migrated before deploying new API pods. How can you orchestrate this ordering in Cloud Deploy?
- A. Run the database migration manually from a developer's workstation.
- B. Use Cloud Deploy **pre-deploy hooks** (or Kubernetes Job hooks defined in Skaffold) that execute the schema migration job and verify its exit status before triggering the main workload rollout.
- C. Deploy both simultaneously and hope for the best.
- D. Cloud Deploy does not support pre-deployment tasks.

---

### Question 7
In Artifact Registry, what is the effect of setting an **Immutable tags** policy on a Docker repository?
- A. New images cannot be pushed.
- B. Once an image is pushed with a specific tag (e.g. `:v1.2.0`), that tag cannot be overwritten or pointed to a different image digest, preventing accidental tag hijacking and ensuring release determinism.
- C. All images are deleted after 24 hours.
- D. Docker images can only be pulled over HTTP.

---

### Question 8
Which DORA metric measures the time elapsed from a developer pushing code to the Git repository to that code successfully executing in production?
- A. Time to Restore Service
- B. Change Failure Rate
- C. Lead Time for Changes
- D. Deployment Frequency

---

### Question 9
In Skaffold, what is the role of the `render` phase?
- A. It renders 3D graphics.
- B. It evaluates template manifests (such as Kustomize overlays or Helm charts) using provided environment profiles and outputs a single, fully rendered, multi-resource Kubernetes YAML stream.
- C. It compiles Go binaries.
- D. It starts the local Docker daemon.

---

### Question 10
When configuring Cloud Deploy delivery pipeline targets, how can an SRE require manual approval before any release can be rolled out to the production cluster?
- A. By deleting the pipeline after staging finishes.
- B. By setting `requireApproval: true` on the Target definition for the production environment.
- C. By changing the cluster password.
- D. By configuring Slack notifications only.

---

## Answer Key & Rationales

1. **B** — `gcloud deploy rollouts rollback` halts the canary progression and restores the last successful stable release.
2. **B** — Cloud Build Private Pools provide private VPC network peering, custom CPU/RAM sizing, and dedicated tenancy for security and internal connectivity.
3. **B** — GitOps engines like Config Sync treat Git as the single source of truth; any out-of-band manual mutation (`kubectl edit`) is actively auto-healed and reverted.
4. **A** — Cloud Deploy natively supports Cloud Run as a first-class Target type, specified with `run: { location: ... }`.
5. **B** — In a secure supply chain, Cloud Build acts as a trusted signer, generating attestations stored in Artifact Analysis that Binary Authorization validates on GKE.
6. **B** — Pre-deploy hooks in Cloud Deploy allow executing prerequisite tasks (such as database migrations) with strict exit-code verification before deploying pods.
7. **B** — Tag immutability in Artifact Registry guarantees that tags cannot be overwritten, eliminating non-deterministic deployments.
8. **C** — **Lead Time for Changes** quantifies the velocity of the delivery pipeline from initial commit to running in production.
9. **B** — Skaffold's render phase processes templated manifests (Helm/Kustomize) into hydrated raw Kubernetes YAML ready for deployment.
10. **B** — Adding `requireApproval: true` to a Target mandates that a user with `clouddeploy.approver` permissions must explicitly approve the rollout before it executes.
