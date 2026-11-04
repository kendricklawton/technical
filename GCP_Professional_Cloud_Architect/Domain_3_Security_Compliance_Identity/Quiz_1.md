# Domain 3: Designing for Security, Identity, and Compliance — Quiz 1

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
A pharmaceutical corporation must store clinical research data in Cloud Storage. Regulatory compliance requires that encryption keys be generated, rotated, and revoked by the internal security team in a dedicated HSM, and that revoking the key immediately renders all encrypted data unreadable. The team does not want to maintain on-premises key servers. Which encryption strategy should be chosen?
- A. Google default encryption
- B. Plaintext storage with password protection
- C. Customer-Managed Encryption Keys (CMEK) using Cloud KMS / Cloud HSM
- D. Customer-Supplied Encryption Keys (CSEK)

---

### Question 2
An architect is designing an enterprise organization hierarchy with three departments: Retail, Logistics, and Finance. The corporate security policy dictates that no project in the entire company may ever create a public external IP address on any VM instance. Where should this Organization Policy constraint be applied?
- A. Individually on every Compute Engine VM
- B. At the subnet level
- C. At the root Organization level (`constraints/compute.vmExternalIpAccess`), allowing inheritance down through all folders and projects
- D. In developer `.bashrc` scripts

---

### Question 3
A rogue developer obtains valid service account credentials that have `roles/storage.admin` permissions on a sensitive production project. The developer attempts to copy multi-terabyte customer datasets from the production bucket to their personal Cloud Storage bucket hosted in an external, unmanaged Google Cloud project. What security feature prevents this data exfiltration?
- A. VPC Service Controls (VPC-SC)
- B. Cloud Armor
- C. Cloud DNS
- D. Cloud NAT

---

### Question 4
An enterprise needs to provide remote developers with secure SSH access to private Compute Engine VMs without assigning external public IP addresses to the VMs or managing a fleet of bastion hosts and VPN software. What Google Cloud solution meets this requirement?
- A. Cloud Interconnect
- B. Cloud NAT
- C. Identity-Aware Proxy (IAP) TCP forwarding
- D. Open port 22 to `0.0.0.0/0`

---

### Question 5
A healthcare platform ingests patient diagnostic reports into BigQuery. Before the records can be made available to data scientists for machine learning research, all Protected Health Information (PHI) such as patient names, phone numbers, and Social Security numbers must be identified, masked, or tokenized automatically. Which Google Cloud service should be integrated into the ETL pipeline?
- A. Google Cloud Deploy
- B. Sensitive Data Protection (Cloud DLP / Data Loss Prevention)
- C. Cloud Trace
- D. Cloud Armor

---

### Question 6
An organization wants to authenticate its 10,000 corporate employees in Google Cloud using their existing corporate Microsoft Entra ID (formerly Azure AD) credentials, including Single Sign-On (SSO) and Multi-Factor Authentication (MFA). What service enables this integration?
- A. Creating 10,000 individual Gmail accounts
- B. Cloud Identity configured with SAML 2.0 / OIDC Single Sign-On to Microsoft Entra ID
- C. Cloud Interconnect
- D. Hardcoding passwords in Cloud IAM

---

### Question 7
In Google Cloud IAM, what is the fundamental difference between an IAM Allow Policy and an IAM Deny Policy?
- A. Deny policies only apply to service accounts.
- B. Allow policies can only be applied at the project level.
- C. Allow policies are free; Deny policies have an hourly cost.
- D. Allow policies grant permissions to principals; Deny policies specify explicit restrictions that take precedence over any Allow policies, preventing access even if granted elsewhere in the hierarchy.

---

### Question 8
A banking enterprise in Europe is subject to strict data sovereignty regulations. When Google Cloud support engineers investigate a high-severity infrastructure incident involving the bank's project, the bank requires an immutable log of every time Google personnel accesses their project data. Which feature fulfills this requirement?
- A. Access Transparency
- B. Security Command Center
- C. Cloud Audit Logs (Admin Activity)
- D. VPC Flow Logs

---

### Question 9
An architect is designing an authentication architecture for a containerized microservice running on GKE that needs to read secrets from Google Secret Manager. How should the application pod authenticate to Secret Manager without storing static credentials in the container image or ConfigMaps?
- A. Use GKE Workload Identity to bind the Kubernetes Service Account directly to a Google Service Account granted `roles/secretmanager.secretAccessor`.
- B. Run the container in privileged mode.
- C. Disable authentication on the Secret Manager API.
- D. Embed a service account private key JSON file in the container image.

---

### Question 10
A security auditor requires that all administrative actions, project modifications, and IAM policy changes across an organization be streamed in real-time to a centralized third-party SIEM (Splunk) in an on-premises datacenter. What architecture delivers this telemetry reliably?
- A. Write a daily bash script to curl the Cloud Logging API.
- B. Email audit logs weekly.
- C. Take manual screenshots of the IAM console.
- D. Configure an Organization-level Log Sink in Cloud Logging that exports Admin Activity audit logs to a Cloud Pub/Sub topic, which is consumed by the SIEM collector over Cloud Interconnect.

---

## Answer Key & Rationales

1. **C** — Customer-Managed Encryption Keys (CMEK) in Cloud KMS allow internal security teams to control key generation, rotation, and cryptographic shredding without on-prem hardware.
2. **C** — Applying Organization Policy constraints at the root Organization node ensures consistent, non-bypassable inheritance across all existing and future projects.
3. **A** — VPC Service Controls establish a secure boundary around GCP APIs, blocking unauthorized cross-project egress even with valid IAM credentials.
4. **C** — IAP TCP forwarding provides secure, authenticated, zero-trust SSH access to private instances without public IPs or bastion hosts.
5. **B** — Sensitive Data Protection (Cloud DLP) automatically inspects, classifies, and redacts PII/PHI in streaming or batch pipelines.
6. **B** — Cloud Identity supports SAML 2.0 / OIDC federation with enterprise IdPs (Entra ID, Okta), centralizing SSO and MFA lifecycle management.
7. **D** — Deny policies act as absolute organizational guardrails, overriding all existing and inherited Allow policies.
8. **A** — Access Transparency generates audit logs whenever Google engineers or technicians access customer resources during support workflows.
9. **A** — GKE Workload Identity provides short-lived pod-scoped IAM credentials mapped from Kubernetes Service Accounts without static keys.
10. **D** — Organization-level Log Sinks export all project audit events to Pub/Sub in real time, enabling reliable streaming to enterprise SIEM platforms.
