# Domain 5: Security, Governance & SRE Operations — Quiz 1

**Exam:** Google Cloud Certified: Professional Cloud DevOps Engineer  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
A DevOps engineer is configuring a GitHub Actions CI/CD pipeline to deploy container images to Google Artifact Registry and Google Kubernetes Engine. The company security policy strictly forbids storing long-lived service account keys in third-party CI tools. How should the engineer configure authentication?
- A. Export a service account JSON key, base64-encode it, and store it in GitHub repository secrets.
- B. Configure Google Cloud Workload Identity Federation with GitHub's OIDC provider, allowing GitHub Actions workflows to exchange short-lived OIDC tokens for temporary GCP credentials.
- C. Open firewall rules to allow all traffic from GitHub IP ranges without authentication.
- D. Run an unauthenticated public Cloud Build trigger.

---

### Question 2
A security audit reveals that developer containers running in production GKE clusters are fetching database passwords from environment variables defined directly in plain text in Kubernetes ConfigMaps committed to Git. What is the recommended Google Cloud native solution to secure these credentials?
- A. Encrypt the Git repository with a password.
- B. Store credentials in Google Secret Manager, and mount them into GKE pods using the Secrets Store CSI Driver or External Secrets Operator with Workload Identity.
- C. Hardcode credentials into the container Dockerfile.
- D. Base64 encode the values in the ConfigMap.

---

### Question 3
An enterprise requires that only container images that have been signed by an automated build system and verified to be free of high-severity vulnerabilities can be scheduled on GKE production clusters. Which Google Cloud service enforces this requirement at deployment time?
- A. Cloud Armor
- B. Binary Authorization
- C. VPC Service Controls
- D. Cloud NAT

---

### Question 4
During a critical SEV-1 production outage, an emergency patch must be deployed immediately to GKE, but the Binary Authorization attestation pipeline is offline due to a third-party KMS issue. How can the incident responder deploy the patch without disabling Binary Authorization cluster-wide?
- A. Delete the GKE cluster.
- B. Use the Binary Authorization "Breakglass" emergency override by adding a specific annotation to the pod template, which allows deployment while generating a security audit log.
- C. Grant the GKE service account `Owner` permissions.
- D. Wait for business hours.

---

### Question 5
Which Google Cloud IAM role should be granted to a service account that needs to upload container images to Artifact Registry, following the principle of least privilege?
- A. `roles/owner`
- B. `roles/editor`
- C. `roles/artifactregistry.writer`
- D. `roles/storage.admin`

---

### Question 6
An organization wants to prevent engineers from accidentally making Cloud Storage buckets public or creating VM instances with external public IP addresses. What mechanism provides declarative, organization-wide governance guardrails in Google Cloud?
- A. Cloud Build triggers
- B. Organization Policy Service (Org Policies / Constraints)
- C. Cloud DNS
- D. Cloud Monitoring alerts

---

### Question 7
To protect sensitive analytics data in BigQuery from data exfiltration caused by compromised service account keys or malicious insider network calls, which security perimeter technology should be implemented?
- A. VPC Service Controls (VPC-SC)
- B. Cloud NAT
- C. Cloud Trace
- D. Cloud Logging

---

### Question 8
In GKE Workload Identity, what role does the GKE Metadata Server play on worker nodes?
- A. It routes customer web traffic to ingress pods.
- B. It intercepts calls to the Google Cloud metadata API (`169.254.169.254`) from within pods, translating Kubernetes Service Account tokens into Google Cloud OAuth tokens based on IAM bindings.
- C. It manages DNS name resolution for external domains.
- D. It compresses log files on disk.

---

### Question 9
A compliance regulation requires that all administrative actions and configuration changes performed on Google Cloud resources be retained immutably for at least 400 days. Which audit log type fulfills this requirement automatically without additional configuration?
- A. Data Access Logs (Data Read)
- B. Admin Activity Audit Logs
- C. Network Flow Logs
- D. Cloud Build build logs

---

### Question 10
When configuring a Workload Identity Pool Provider for an external CI tool, why is it critical to configure an `attribute-condition` (such as `assertion.repository == 'org/repo'`)?
- A. To make builds run 50% faster.
- B. To prevent ANY workflow from ANY repository on that external CI provider from exchanging tokens and assuming the privileged deployment service account.
- C. Because Google Cloud charges per repository.
- D. To enable debug logging.

---

## Answer Key & Rationales

1. **B** — Workload Identity Federation replaces brittle, long-lived JSON keys with short-lived cryptographic OIDC token exchange.
2. **B** — Secret Manager with Secrets Store CSI Driver and Workload Identity ensures credentials remain encrypted at rest and in transit, decoupled from source code.
3. **B** — Binary Authorization acts as an admission controller enforcing cryptographic attestations before images can be deployed.
4. **B** — Breakglass allows emergency deployments to bypass attestation checks while producing an immutable audit log for post-incident review.
5. **C** — `roles/artifactregistry.writer` grants precise push permissions without granting unnecessary administrative or broad storage privileges.
6. **B** — Organization Policies enforce centralized, preventive compliance guardrails (e.g. `constraints/compute.vmExternalIpAccess`).
7. **A** — VPC-SC establishes cryptographic network and API perimeters around managed services to stop data exfiltration across project boundaries.
8. **B** — The GKE Metadata Server intercepts pod metadata requests, validating pod identity and returning scoped short-lived GCP tokens.
9. **B** — Admin Activity logs are permanently enabled, free of charge, and retained for 400 days across all Google Cloud services.
10. **B** — Without an attribute condition restricting repository and branch, any actor capable of obtaining a valid JWT from the provider could assume the service account.
