# Domain 2: Building & Implementing CI/CD Pipelines — Quiz 1

**Exam:** Google Cloud Certified: Professional Cloud DevOps Engineer  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
Your development team needs to build container images rapidly in Google Cloud Build. Currently, builds take over 20 minutes because Docker layers are downloaded from scratch on every run. How should you optimize build speed using Cloud Build's native features?
- A. Use the **Kaniko** executor step with caching enabled (`--cache=true` and `--cache-ttl=24h`), which stores cached layers in Artifact Registry.
- B. Run builds on larger Compute Engine VMs with custom shell scripts.
- C. Disable all automated tests in the pipeline.
- D. Build images locally on developer laptops and push to production manually.

---

### Question 2
In Google Cloud Deploy, what is the fundamental difference between a **Release** and a **Rollout**?
- A. They are synonyms and can be used interchangeably.
- B. A Rollout compiles source code; a Release applies Terraform.
- C. A Release is an immutable point-in-time snapshot of the pipeline, rendered Kubernetes manifests, and container image versions; a Rollout is the operational deployment of a specific Release to a specific Target environment.
- D. A Release is for GKE; a Rollout is only for Cloud Run.

---

### Question 3
An engineering organization wants to eliminate the need to maintain static, long-lived service account keys (`.json` credentials) in external GitHub Actions runners that trigger Google Cloud Build pipelines. What Google Cloud feature should they configure?
- A. Workload Identity Federation (OIDC) between GitHub Actions and Google Cloud IAM.
- B. Cloud Key Management Service (Cloud KMS)
- C. Run GitHub Actions on Google Compute Engine instances with public IP addresses.
- D. Commit the service account JSON key to a private Git repository.

---

### Question 4
You are designing a zero-downtime deployment strategy for a stateless microservice on Google Kubernetes Engine. The service receives 10,000 requests per second. You need to ensure that at no point during the deployment does the total available pod capacity drop below 100%, and you want to avoid spinning up more than 25% extra temporary pods. How should the Deployment `strategy` be configured?
- A. Set replicas to 1.
- B. `strategy: { type: Recreate }`
- C. `strategy: { type: RollingUpdate, rollingUpdate: { maxUnavailable: "50%", maxSurge: 0 } }`
- D. `strategy: { type: RollingUpdate, rollingUpdate: { maxUnavailable: 0, maxSurge: "25%" } }`

---

### Question 5
When using Google Cloud Deploy, which open-source tool is used as the underlying declarative manifest rendering and application engine?
- A. Skaffold
- B. Ansible
- C. Chef
- D. Puppet

---

### Question 6
You are configuring Google Cloud Deploy with a **Canary** deployment strategy targeting GKE. You want the pipeline to pause and route 10% of traffic to the new version, automatically running synthetic test requests to verify response codes before prompting an SRE for promotion approval. Which configuration achieves this?
- A. Create two separate Google Cloud projects and copy the database.
- B. Set `canaryDeployment: { percentages: [10], verify: true }` in the delivery pipeline stage.
- C. Manually edit iptables on worker nodes.
- D. Use Cloud DNS geolocation routing.

---

### Question 7
A security compliance mandate requires that all container images deployed to production GKE clusters must be cryptographically signed during the CI pipeline, and GKE must block any unsigned images from running. Which Google Cloud service enforces this at admission time?
- A. Secret Manager
- B. Security Command Center
- C. Binary Authorization
- D. Cloud Armor

---

### Question 8
What is the recommended storage backend for private Docker and OCI container images in Google Cloud, replacing deprecated Container Registry (GCR)?
- A. Google Cloud Storage public buckets
- B. Artifact Registry
- C. Cloud Filestore NFS share
- D. Compute Engine local SSDs

---

### Question 9
In a Blue-Green deployment strategy, what is the primary operational advantage over an in-place rolling update?
- A. Blue-Green does not require container images.
- B. It eliminates the need for database backups.
- C. Blue-Green consumes 50% less compute capacity.
- D. Near-instantaneous cutover and near-instantaneous rollback by switching traffic at the load balancer or router level without waiting for pods to terminate or start.

---

### Question 10
According to DORA metrics, which of the following metrics measures the percentage of deployments causing a degradation or outage that requires immediate remediation (e.g. rollback, patch, hotfix)?
- A. Lead Time for Changes
- B. Change Failure Rate
- C. Mean Time to Recover
- D. Deployment Frequency

---

## Answer Key & Rationales

1. **A** — Kaniko runs inside Cloud Build without requiring Docker daemon privileges and caches intermediate image layers in Artifact Registry, slashing build times.
2. **C** — In Google Cloud Deploy, a Release represents the immutable packaged configuration and artifacts, whereas a Rollout is the execution of applying that Release to a specific Target.
3. **A** — Workload Identity Federation allows external identity providers (like GitHub Actions) to exchange OIDC tokens for short-lived Google Cloud STS access tokens without static JSON keys.
4. **D** — Setting `maxUnavailable: 0` guarantees 100% baseline capacity is preserved at all times, while `maxSurge: 25%` limits temporary extra compute overhead to 25%.
5. **A** — Cloud Deploy utilizes Skaffold to parse, render (via Kustomize or Helm), and apply declarative Kubernetes and Cloud Run manifests.
6. **B** — Cloud Deploy native canary configuration specifies percentage steps (e.g. 10%) and enables automated `verify: true` jobs before manual or automated progression.
7. **C** — Binary Authorization is a deploy-time security control for GKE that ensures only trusted container images signed by designated attestors can be deployed.
8. **B** — Artifact Registry is Google Cloud's fully managed universal package and OCI container repository, providing granular IAM, regional replication, and vulnerability scanning.
9. **D** — Blue-Green maintains two independent environments, enabling instantaneous traffic redirection via load balancer route switching and immediate rollback if issues occur.
10. **B** — Change Failure Rate specifically quantifies the proportion of production releases that result in degraded service requiring rollbacks or hotfixes.
