# Domain 3: Designing for Security, Identity, and Compliance — Study Guide

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Weight:** ~21% of the certification curriculum

---

## 1. Resource Hierarchy & Identity Architecture
- **Google Cloud Resource Hierarchy:**
  - `Organization` (Root node, tied 1:1 to Google Workspace or Cloud Identity domain).
  - `Folders` (Nested up to 300 levels deep; represent business units, departments, or environments like Prod, Staging, Dev).
  - `Projects` (Billing and resource management boundary; all resources live in a project).
  - `Resources` (Compute Engine VMs, GKE clusters, Cloud Storage buckets).
- **Policy Inheritance:** IAM roles and Organization Policies flow downward down the hierarchy. An Allow permission granted at the Folder level cannot be revoked at the Project level (union model), but IAM Deny Policies can explicitly override Allow permissions.
- **Identity Federation:**
  - **Cloud Identity / Directory Sync:** Synchronizes identities and groups from Active Directory (AD) or Azure AD / Okta into Cloud Identity via SAML 2.0 / OIDC.
  - **Workload Identity Federation (WIF):** Allows non-GCP workloads (AWS, Azure, GitHub Actions, on-premises Kubernetes) to authenticate to GCP APIs using short-lived federated tokens without managing static service account keys.

---

## 2. Data Protection & Cryptographic Key Management
- **Encryption Tiers:**
  - **Default Encryption:** Google automatically encrypts all customer data at rest using AES-256 with Google-managed encryption keys. Zero configuration required.
  - **Customer-Managed Encryption Keys (CMEK):** Customer creates and controls encryption keys using Cloud KMS. Keys can be rotated automatically (e.g. every 90 days) or destroyed instantly to revoke access. Supported natively by BigQuery, Cloud Storage, Compute Engine, Persistent Disks.
  - **Customer-Supplied Encryption Keys (CSEK):** Customer supplies raw 256-bit AES keys in API headers. Google never stores the key in persistent memory. If the key is lost, data is permanently irretrievable.
  - **Cloud External Key Manager (Cloud EKM):** Keys reside entirely outside Google Cloud within a third-party Hardware Security Module (HSM) such as Thales or Fortanix.
- **Sensitive Data Protection (Cloud DLP):**
  - Discovers, classifies, and de-identifies PII, credit card numbers (PCI), Social Security numbers, and HIPAA PHI data using masking, tokenization, or bucketing.

---

## 3. Perimeter Security & Zero Trust Architecture
- **VPC Service Controls (VPC-SC):**
  - Defines fine-grained security perimeters around multi-tenant Google Cloud services (Cloud Storage, BigQuery, Secret Manager).
  - Blocks API requests originating outside the perimeter (even with valid IAM credentials) to prevent data exfiltration.
  - Ingress/Egress rules and Access Levels (Context-Aware Access) permit authorized data exchange between perimeters.
- **BeyondCorp Enterprise & Identity-Aware Proxy (IAP):**
  - Zero Trust security model replacing traditional corporate VPNs.
  - Enforces context-aware access (user identity, device health, location, MFA) to web applications and VM administrative interfaces (SSH/RDP) without public IP exposure.

---

## 4. Compliance, Auditability & Governance
- **Security Command Center (SCC):**
  - Enterprise security and risk management platform for Google Cloud.
  - Asset inventory, misconfiguration detection (Security Health Analytics), container vulnerability threats, and anomaly detection (Event Threat Detection).
- **Access Transparency & Access Approval:**
  - **Access Transparency:** Generates near-real-time audit logs whenever Google Cloud support engineers or site reliability engineers access customer data during support ticket resolution.
  - **Access Approval:** Requires customer cryptographic approval before Google personnel can access customer content.
- **Regulatory Frameworks:**
  - HIPAA (Business Associate Agreement [BAA] required before storing Protected Health Information).
  - PCI-DSS (Cardholder Data Environment [CDE] segmentation, tokenization via DLP, locked audit logs).
  - FedRAMP High / Moderate and SOC 1/2/3 certifications.
