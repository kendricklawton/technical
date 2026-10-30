# Domain 2: Building & Implementing CI/CD Pipelines — Quiz 1

**Exam:** Google Cloud Certified: Professional Cloud DevOps Engineer  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
Your development team needs to build container images rapidly in Google Cloud Build. Currently, builds take over 20 minutes because Docker layers are downloaded from scratch on every run. How should you optimize build speed using Cloud Build's native features?
- A. Run builds on larger Compute Engine VMs with custom shell scripts.
- B. Use the **Kaniko** executor step with caching enabled (`--cache=true` and `--cache-ttl=24h`), which stores cached layers in Artifact Registry.
- C. Disable all automated tests in the pipeline.
- D. Build images locally on developer laptops and push to production manually.

---

### Question 2
In Google Cloud Deploy, what is the fundamental difference between a **Release** and a **Rollout**?
- A. A Release is an immutable point-in-time snapshot of the pipeline, rendered Kubernetes manifests, and container image versions; a Rollout is the operational deployment of a specific Release to a specific Target environment.
- B. A Release is for GKE; a Rollout is only for Cloud Run.
- C. A Rollout compiles source code; a Release applies Terraform.
- D. They are synonyms and can be used interchangeably.

---

### Question 3
An engineering organization wants to eliminate the need to maintain static, long-lived service account keys (`.json` credentials) in external GitHub Actions runners that trigger Google Cloud Build pipelines. What Google Cloud feature should they configure?
- A. Cloud Key Management Service (Cloud KMS)
- B. Workload Identity Federation (OIDC) between GitHub Actions and Google Cloud IAM.
- C. Commit the service account JSON key to a private Git repository.
- D. Run GitHub Actions on Google Compute Engine instances with public IP addresses.

---

### Question 4
You are designing a zero-downtime deployment strategy for a stateless microservice on Google Kubernetes Engine. The service receives 10,000 requests per second. You need to ensure that at no point during the deployment does the total available pod capacity drop below 100%, and you want to avoid spinning up more than 25% extra temporary pods. How should the Deployment `strategy` be configured?
- A. `strategy: { type: Recreate }`
- B. `strategy: { type: RollingUpdate, rollingUpdate: { maxUnavailable: 0, maxSurge: "25%" } }`
- C. `strategy: { type: RollingUpdate, rollingUpdate: { maxUnavailable: "50%", maxSurge: 0 } }`
- D. Set replicas to 1.

---

### Question 5
When using Google Cloud Deploy, which open-source tool is used as the underlying declarative manifest rendering and application engine?
- A. Ansible
- B. Skaffold
- C. Puppet
- D. Chef

---

### Question 6
You are configuring Google Cloud Deploy with a **Canary** deployment strategy targeting GKE. You want the pipeline to pause and route 10% of traffic to the new version, automatically running synthetic test requests to verify response codes before prompting an SRE for promotion approval. Which configuration achieves this?
- A. Set `canaryDeployment: { percentages: [10], verify: true }` in the delivery pipeline stage.
- B. Manually edit iptables on worker nodes.
- C. Create two separate Google Cloud projects and copy the database.
- D. Use Cloud DNS geolocation routing.

---

### Question 7
A security compliance mandate requires that all container images deployed to production GKE clusters must be cryptographically signed during the CI pipeline, and GKE must block any unsigned images from running. Which Google Cloud service enforces this at admission time?
- A. Cloud Armor
- B. Binary Authorization
- C. Security Command Center
- D. Secret Manager

---

### Question 8
What is the recommended storage backend for private Docker and OCI container images in Google Cloud, replacing deprecated Container Registry (GCR)?
- A. Google Cloud Storage public buckets
- B. Artifact Registry
- C. Compute Engine local SSDs
- D. Cloud Filestore NFS share

---

### Question 9
In a Blue-Green deployment strategy, what is the primary operational advantage over an in-place rolling update?
- A. Blue-Green consumes 50% less compute capacity.
- B. Near-instantaneous cutover and near-instantaneous rollback by switching traffic at the load balancer or router level without waiting for pods to terminate or start.
- C. Blue-Green does not require container images.
- D. It eliminates the need for database backups.

---

### Question 10
According to DORA metrics, which of the following metrics measures the percentage of deployments causing a degradation or outage that requires immediate remediation (e.g. rollback, patch, hotfix)?
- A. Deployment Frequency
- B. Lead Time for Changes
- C. Change Failure Rate
- D. Mean Time to Recover

---

## Answer Key & Rationales

1. **B** — Kaniko runs inside Cloud Build without requiring Docker daemon privileges and caches intermediate image layers in Artifact Registry, slashing build times.
2. **A** — In Google Cloud Deploy, a Release represents the immutable packaged configuration and artifacts, whereas a Rollout is the execution of applying that Release to a specific Target.
3. **B** — Workload Identity Federation allows external identity providers (like GitHub Actions) to exchange OIDC tokens for short-lived Google Cloud STS access tokens without static JSON keys.
4. **B** — Setting `maxUnavailable: 0` guarantees 100% baseline capacity is preserved at all times, while `maxSurge: 25%` limits temporary extra compute overhead to 25%.
5. **B** — Cloud Deploy utilizes Skaffold to parse, render (via Kustomize or Helm), and apply declarative Kubernetes and Cloud Run manifests.
6. **A** — Cloud Deploy native canary configuration specifies percentage steps (e.g. 10%) and enables automated `verify: true` jobs before manual or automated progression.
7. **B** — Binary Authorization is a deploy-time security control for GKE that ensures only trusted container images signed by designated attestors can be deployed.
8. **B** — Artifact Registry is Google Cloud's fully managed universal package and OCI container repository, providing granular IAM, regional replication, and vulnerability scanning.
9. **B** — Blue-Green maintains two independent environments, enabling instantaneous traffic redirection via load balancer route switching and immediate rollback if issues occur.
10. **C** — Change Failure Rate specifically quantifies the proportion of production releases that result in degraded service requiring rollbacks or hotfixes.
