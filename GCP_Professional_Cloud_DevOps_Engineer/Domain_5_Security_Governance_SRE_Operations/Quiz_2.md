# Domain 5: Security, Governance & SRE Operations — Quiz 2

**Exam:** Google Cloud Certified: Professional Cloud DevOps Engineer  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
A security scan discovers that several microservices deployed on Compute Engine VMs are communicating with Cloud SQL over the public internet with authorized networks IP allowlists. What is the most secure architectural change to eliminate public internet exposure?
- A. Whitelist all corporate office IP addresses.
- B. Connect to Cloud SQL using Private Service Access (internal RFC 1918 VPC IP) combined with the Cloud SQL Auth Proxy for mutual TLS encryption and IAM authentication.
- C. Disable database passwords.
- D. Run the database directly on the application VM.

---

### Question 2
An SRE team wants to enforce automated vulnerability scanning on all container images pushed to Artifact Registry before deployment. Which Google Cloud service natively inspects images for known CVEs and package vulnerabilities?
- A. Container Analysis / Artifact Analysis
- B. Cloud Profiler
- C. Cloud Trace
- D. Compute Engine Autoscaler

---

### Question 3
A DevOps engineer needs to run database migrations against a private Cloud SQL instance from a Cloud Build pipeline running in a Google-managed worker pool. How can Cloud Build reach the private VPC network?
- A. Open the Cloud SQL database to `0.0.0.0/0`.
- B. Configure a Cloud Build Private Pool peered with the application VPC.
- C. Store database passwords in GitHub README.
- D. Run migrations manually from a developer laptop.

---

### Question 4
Under the Principle of Least Privilege, which service account should be attached to GKE worker nodes during cluster creation?
- A. The Compute Engine default service account (`PROJECT_NUMBER-compute@developer.gserviceaccount.com`) with Editor role.
- B. A dedicated, minimally privileged custom Service Account with only `roles/logging.logWriter`, `roles/monitoring.metricWriter`, and `roles/artifactregistry.reader`.
- C. The organization Administrator account.
- D. No service account at all.

---

### Question 5
How does Google Cloud Customer-Managed Encryption Keys (CMEK) differ from default Google encryption?
- A. Default encryption uses no keys at all.
- B. Default encryption uses keys generated and managed entirely by Google; CMEK allows the customer to control key lifecycle, rotation, and revocation in Cloud KMS.
- C. CMEK makes queries run faster.
- D. CMEK is only available for compute instances.

---

### Question 6
An organization must comply with PCI-DSS requirements. All production Cloud Storage buckets containing credit card transaction logs must prevent deletion or modification of logs for 7 years. What Cloud Storage feature guarantees this?
- A. Storage Class Coldline
- B. Bucket Retention Policy with Bucket Lock enabled (WORM compliance)
- C. Dual-region replication
- D. Object versioning without retention

---

### Question 7
In a microservices architecture on GKE, how can network traffic between Pod A and Pod B be restricted so that Pod A can only talk to Pod B on port 8080 and all other internal traffic is blocked?
- A. Cloud Armor
- B. Kubernetes NetworkPolicy (with network policy enforcement or Datapath V2 / Cilium enabled)
- C. VPC Firewall rules applied to node external IPs
- D. DNS TXT records

---

### Question 8
What is the purpose of Google Cloud IAM Deny Policies?
- A. To replace all Allow policies across the organization.
- B. To define explicit, overriding restrictions that prevent principals from exercising specific permissions, regardless of any roles granted in Allow policies.
- C. To block all incoming HTTP web requests.
- D. To prevent users from logging into the Google Cloud Console.

---

### Question 9
A team wants to automatically rotate database credentials stored in Secret Manager every 30 days. What combination of Google Cloud services facilitates this automation?
- A. Cloud Build and Cloud Run only.
- B. Secret Manager automatic rotation configured with a Pub/Sub topic and a Cloud Function / Cloud Run service that updates the database password and creates a new secret version.
- C. An on-call engineer setting a calendar reminder.
- D. Cloud Trace and Cloud Profiler.

---

### Question 10
When using Workload Identity Federation, what role does Google Cloud STS (Security Token Service) fulfill?
- A. It scans container images for malware.
- B. It validates the incoming third-party OIDC/SAML token, exchanges it for a federated Google access token, and authorizes the principal to impersonate a Google service account.
- C. It deploys GKE pods.
- D. It acts as an external DNS server.

---

## Answer Key & Rationales

1. **B** — Connecting via Private Service Access (internal RFC 1918) and Cloud SQL Auth Proxy provides mTLS encryption, IAM-based access, and removes exposure to the public internet.
2. **A** — Container Analysis / Artifact Analysis automatically scans container images in Artifact Registry for vulnerability CVEs.
3. **B** — Cloud Build Private Pools run inside a managed tenant peered directly with your VPC, enabling private, non-public IP access to databases.
4. **B** — The Compute Engine default service account has overly broad `Editor` rights. Hardened clusters use dedicated custom service accounts with minimum required roles.
5. **B** — CMEK allows customers to manage encryption keys in Cloud KMS, enabling immediate data access revocation by disabling the key.
6. **B** — A locked Bucket Retention Policy enforces Write-Once-Read-Many (WORM) storage that cannot be bypassed or shortened even by project owners.
7. **B** — Kubernetes NetworkPolicies enforce East-West zero-trust pod communication within the cluster.
8. **B** — IAM Deny policies create absolute guardrails that override any conflicting Allow policies.
9. **B** — Secret Manager rotation schedules publish events to Pub/Sub, triggering a serverless function that updates the target database and writes the new secret version.
10. **B** — STS is the security bridge that verifies external IdP tokens and returns short-lived Google Cloud authentication tokens.
