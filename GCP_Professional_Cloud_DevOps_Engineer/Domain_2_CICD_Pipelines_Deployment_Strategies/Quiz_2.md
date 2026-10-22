# Domain 2: Building & Implementing CI/CD Pipelines — Quiz 2

**Exam:** Google Cloud Certified: Professional Cloud DevOps Engineer  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
In Google Cloud Build, how should you store and pass database passwords or API tokens to build steps without exposing them in plaintext inside `cloudbuild.yaml` or build logs?
- A. Pass them as plain command-line arguments in `args: []`.
- B. Store the credentials in **Secret Manager**, grant the Cloud Build Service Account the `Secret Manager Secret Accessor` role, and reference them in `cloudbuild.yaml` using the `availableSecrets` block.
- C. Store credentials in a public GitHub repository.
- D. Base64-encode the password in the Git commit message.

---

### Question 2
You are using Google Cloud Deploy with **Skaffold** to deploy an application across multiple environments (`dev`, `staging`, `prod`). The production environment requires 10 replicas and 4 CPU requests per pod, while the dev environment requires only 1 replica and 250m CPU. How should this configuration be managed cleanly without duplicating entire Kubernetes manifest files?
- A. Maintain separate Git repositories for every environment with duplicate YAML files.
- B. Use **Kustomize overlays** organized into `base/` and environment-specific `overlays/` directories, referenced by Skaffold profiles (`profiles: [dev, prod]`).
- C. Write a custom Python script to edit YAML files using string replacement during CI.
- D. Manually edit running pods in production with `kubectl edit`.

---

### Question 3
An SRE team wants to test a brand new version of a payment processing engine using real production user traffic, but they cannot risk real customers experiencing failures or duplicate credit card charges from experimental code. Which deployment strategy should be implemented?
- A. Canary deployment with 1% traffic.
- B. **Shadow Deployment (Traffic Mirroring)** using Google Cloud External HTTP(S) Load Balancing or Istio/Cilium service mesh to mirror a copy of live production requests to the new service without returning its responses to clients.
- C. Blue-Green deployment with immediate cutover.
- D. Recreate deployment at midnight.

---

### Question 4
In Google Cloud Deploy, an automated rollout to production fails because the target GKE cluster lacks sufficient compute capacity to schedule new pods. What command immediately rolls back the production target to the previous successful release?
- A. `gcloud compute instances reset --all`
- B. `gcloud deploy rollouts rollback --rollout=<FAILED_ROLLOUT> --delivery-pipeline=<PIPELINE> --region=<REGION>`
- C. `git revert HEAD`
- D. `kubectl delete namespace default`

---

### Question 5
Which Cloud Build substitution variable provides the commit SHA that triggered the build?
- A. `$PROJECT_ID`
- B. `$COMMIT_SHA`
- C. `$BUILD_ID`
- D. `$BRANCH_NAME`

---

### Question 6
You need to prevent an unauthorized developer from bypassing the CI/CD pipeline and directly modifying live production workloads on a GKE cluster using `kubectl`. How should IAM permissions be structured?
- A. Delete all Google Cloud IAM roles.
- B. Grant developers `Kubernetes Engine Developer` only in the development project; grant the Google Cloud Deploy Service Account `Kubernetes Engine Admin` in the production project, enforcing that all production changes originate strictly from approved pipeline releases.
- C. Change the cluster root password daily.
- D. Disable the Kubernetes API server.

---

### Question 7
What is the function of **Google Cloud Deploy execution environments** (Private Pools)?
- A. They allow Cloud Deploy to run rendering and deployment jobs within a dedicated, customer-managed VPC network, enabling private network connectivity to GKE private clusters without exposing control planes to the public internet.
- B. They run local Docker containers for developers.
- C. They provide free GPU compute.
- D. They host DNS servers.

---

### Question 8
In GitOps, what is the core architectural principle that distinguishes it from traditional push-based CI/CD scripts?
- A. GitOps relies on manual approvals for every step.
- B. An in-cluster operator (such as **Config Sync** or Argo CD) continuously pulls declarative desired state from a Git repository and automatically reconciles any live cluster drift, rather than an external CI tool pushing changes imperatively.
- C. GitOps requires using GitHub Enterprise only.
- D. GitOps eliminates the need for Kubernetes.

---

### Question 9
How does **Artifact Analysis** in Google Cloud improve container supply chain security?
- A. It automatically decompiles Java bytecode.
- B. It scans container images in Artifact Registry for known Common Vulnerabilities and Exposures (CVEs) upon push, and continuously monitors them as new vulnerability advisories are published in the CVE database.
- C. It increases network download speeds by 50%.
- D. It compresses images using zip.

---

### Question 10
In Google Cloud Build, how can you configure a build step to execute only when previous steps succeed, and another step to run only if a failure occurs (cleanup step)?
- A. Using bash `if` statements inside container scripts or using the `waitFor` and `status` attributes.
- B. Cloud Build cannot handle step conditions.
- C. By submitting two separate builds.
- D. By restarting the runner.

---

## Answer Key & Rationales

1. **B** — Secret Manager securely stores credentials, and Cloud Build natively integrates via `availableSecrets` to inject secrets as environment variables into specific build steps.
2. **B** — Kustomize overlays provide declarative, DRY (Don't Repeat Yourself) environment customization, cleanly referenced by Skaffold profiles without code duplication.
3. **B** — Traffic mirroring (shadow deployment) copies real incoming production traffic to an experimental backend without affecting live customer transactions.
4. **B** — `gcloud deploy rollouts rollback` initiates an automated 1-click rollback, re-deploying the previous known-good release to the target.
5. **B** — `$COMMIT_SHA` is a built-in Cloud Build default substitution containing the complete git commit hash.
6. **B** — Enforcing least privilege via IAM ensures only automated CI/CD service accounts have deploy permissions on production clusters, eliminating manual developer tampering.
7. **A** — Private pools in Cloud Deploy allow deployment runners to communicate securely with private GKE clusters over internal VPC IPs without traversing public endpoints.
8. **B** — GitOps uses an in-cluster reconciliation loop (e.g. Google Cloud Config Sync) that continuously compares live state against declarative Git manifests, auto-healing drift.
9. **B** — Artifact Analysis provides automated vulnerability scanning at image push time and continuously updates vulnerability status when new CVEs are cataloged.
10. **A** — Cloud Build `waitFor` allows defining directed acyclic graphs (DAGs) of build steps, controlling step execution ordering and failure handling.
