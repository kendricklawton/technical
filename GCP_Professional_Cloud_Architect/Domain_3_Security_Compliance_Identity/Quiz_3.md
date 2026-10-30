# Domain 3: Designing for Security, Identity, and Compliance — Quiz 3

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
An architect is designing an automated data pipeline that transfers customer billing records between two projects in different departments. Project A is inside Service Perimeter Alpha, and Project B is inside Service Perimeter Beta. The transfer fails with a VPC Service Controls perimeter violation error. How should the architect enable secure data sharing between these two projects without merging the perimeters?
- A. Delete both perimeters.
- B. Configure VPC Service Controls Perimeter Ingress and Egress rules allowing specific identities, services, and methods between Perimeter Alpha and Perimeter Beta.
- C. Download data to a developer laptop and re-upload.
- D. Make the Cloud Storage bucket public.

---

### Question 2
A retail e-commerce platform must achieve PCI-DSS Level 1 compliance. Which architectural design practice isolates credit card data and minimizes the scope of PCI audit compliance?
- A. Storing credit card numbers in the primary customer database alongside public profile information.
- B. Isolating the Cardholder Data Environment (CDE) into dedicated Google Cloud projects within a restricted VPC Service Controls perimeter, tokenizing card numbers using Cloud DLP, and processing tokens in general application services.
- C. Disabling all audit logging.
- D. Using unencrypted HTTP for internal microservices.

---

### Question 3
An enterprise requires that all Compute Engine VM disks created by developers be encrypted using a customer-managed key in Cloud KMS. If a developer attempts to create a VM with standard Google-managed encryption, the creation must be blocked automatically. What mechanism enforces this requirement?
- A. An Organization Policy constraint: `constraints/gcp.restrictNonCmekServices`
- B. A post-deployment bash script
- C. A weekly email audit
- D. Disabling Compute Engine

---

### Question 4
An architect needs to assign administrative permissions to a junior cloud engineer who will manage Compute Engine VM instances in the `staging` project only. The engineer should not be able to modify firewall rules, delete VPC networks, or modify IAM permissions. Which role should be assigned?
- A. `roles/owner`
- B. `roles/editor`
- C. `roles/compute.instanceAdmin.v1`
- D. `roles/viewer`

---

### Question 5
How does Google Cloud Shielded VM protect workloads against rootkits and malicious bootloader modifications?
- A. It automatically doubles RAM.
- B. It uses Secure Boot, Virtual Trusted Platform Module (vTPM), and integrity monitoring to ensure that the guest OS kernel and boot components match cryptographic baseline measurements.
- C. It blocks all outbound HTTP connections.
- D. It disables SSH access permanently.

---

### Question 6
A developer needs to query BigQuery data containing hashed customer passwords. The table has columns `customer_id`, `email`, and `password_hash`. The analyst must be able to query `customer_id` and `email` for marketing analytics, but must NOT be allowed to view `password_hash`. What BigQuery security feature enforces this without duplicating tables or creating separate views for each user?
- A. Column-Level Access Control (Policy Tags / Taxonomy)
- B. Deleting the column
- C. Converting the dataset to CSV
- D. Granting `roles/bigquery.admin`

---

### Question 7
An architect wants to implement automated threat detection that alerts the security operations center (SOC) if an unauthorized user creates an IAM service account key or if anomalous outbound network traffic indicates cryptocurrency mining. Which service provides these capabilities natively?
- A. Security Command Center Premium (Event Threat Detection & Container Threat Detection)
- B. Cloud DNS
- C. Cloud Trace
- D. Cloud Composer

---

### Question 8
What is the effect of granting a user the `roles/iam.securityAdmin` role at the Folder level?
- A. The user can create and modify IAM policies and grant roles on all projects and resources residing within that folder and its child folders.
- B. The user can delete the Google Cloud Organization.
- C. The user cannot view any resources.
- D. The user can access Google Cloud billing credit cards.

---

### Question 9
An enterprise wants to ensure that all data stored in Cloud Storage buckets in Europe is physically stored only within the geographic boundaries of the European Union to comply with GDPR data sovereignty requirements. How should the architect configure bucket provisioning?
- A. Set bucket location to the `EU` multi-region or European dual/regional locations (e.g. `europe-west1`), and enforce the `constraints/gcp.resourceLocations` organization policy constraint.
- B. Rely on DNS names.
- C. Store data in `us-central1` and encrypt with a European password.
- D. Use tape drives.

---

### Question 10
When configuring Workload Identity Federation for an on-premises Kubernetes cluster, what is the advantage of using OIDC token exchange over distributing service account private keys?
- A. Tokens are valid for 10 years.
- B. Short-lived federated tokens eliminate the risk of leaked static JSON credentials, require zero manual key rotation, and tie authentication to cryptographic workload identity proofs.
- C. Tokens do not require network connectivity.
- D. Tokens bypass Cloud IAM permissions.

---

## Answer Key & Rationales

1. **B** — VPC Service Controls Ingress and Egress rules establish audited, cryptographic bridges between distinct security perimeters without compromising perimeter isolation.
2. **B** — Segmenting the Cardholder Data Environment (CDE) into dedicated projects and tokenizing sensitive data with DLP reduces the PCI audit scope to the CDE boundary.
3. **A** — Organization Policy CMEK constraints mandate customer-managed keys, rejecting non-compliant resource creation requests at the API layer.
4. **C** — `roles/compute.instanceAdmin.v1` grants full control over instance lifecycle without network administration or IAM privilege escalation rights.
5. **B** — Shielded VMs leverage Secure Boot and vTPM to cryptographically measure boot integrity and detect firmware/kernel level tampering.
6. **A** — BigQuery Column-Level Access Control applies policy tags to restrict access to sensitive columns while allowing general queries on remaining fields.
7. **A** — Security Command Center Event Threat Detection parses audit logs and telemetry to identify compromised credentials, malware, and cryptocurrency mining.
8. **A** — `roles/iam.securityAdmin` grants permission to manage IAM policies across all child entities beneath the assigned resource node.
9. **A** — Selecting European bucket locations combined with `constraints/gcp.resourceLocations` guarantees physical storage within EU borders for GDPR compliance.
10. **B** — Workload Identity Federation replaces vulnerable static private keys with short-lived tokens derived from cryptographic OIDC proofs.
