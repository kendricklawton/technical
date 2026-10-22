# Domain 5: Security, Governance & SRE Operations — Study Guide

**Exam:** Google Cloud Certified: Professional Cloud DevOps Engineer  
**Weight:** ~15% of the exam curriculum

---

## 1. Identity & Access Management (IAM) for DevOps & CI/CD
- **Least Privilege Principle:** Never assign primitive roles (`Owner`, `Editor`, `Viewer`) to automated service accounts or CI/CD pipelines. Use granular, predefined roles (e.g. `roles/cloudbuild.builds.builder`, `roles/clouddeploy.releaser`) or custom IAM roles.
- **Service Account Keys vs Workload Identity Federation (WIF):**
  - Storing static service account private keys (`service-account-key.json`) in external CI systems (GitHub Actions, GitLab CI, Jenkins) is an anti-pattern and major security vulnerability.
  - Workload Identity Federation enables external workloads to exchange an OpenID Connect (OIDC) or SAML 2.0 token for short-lived Google Cloud OAuth credentials without long-lived keys.
- **GKE Workload Identity:**
  - Binds Kubernetes Service Accounts (KSA) to Google Service Accounts (GSA) using IAM annotations and Workload Identity Pools.
  - Eliminates the need to mount secret JSON keys inside pod containers.
  - Metadata server intercepts pod requests to `http://metadata.google.internal` and returns short-lived Google Cloud tokens scoped strictly to that KSA.

---

## 2. Supply Chain Security & Binary Authorization
- **Software Supply Chain Threats:** Source tampering, dependency poisoning, malicious build injection, unverified deployment artifacts.
- **SLSA Framework (Supply-chain Levels for Software Artifacts):**
  - Level 1: Scripted build, provenance available.
  - Level 2: Build service version control, authenticated provenance.
  - Level 3: Isolated ephemeral build environments, non-falsifiable provenance.
- **Binary Authorization:**
  - Deploy-time security control for GKE and Cloud Run.
  - Verifies cryptographic attestations (signatures) created by trusted authorities (Attestors) using Cloud KMS before allowing container execution.
  - Built-in dry-run mode (`DRYRUN_AUDIT_LOG_ONLY`) to evaluate policy impact without breaking active deployments.
  - Breakglass mechanism: Allows emergency deployment of unsigned containers during SEV-1 outages with mandatory audit logging in Cloud Audit Logs.

---

## 3. Secret Management & Data Protection
- **Google Secret Manager:**
  - Secure, centralized storage for API keys, passwords, certificates, and database credentials.
  - Versioned secrets with automatic payload encryption using Google-managed keys or CMEK (Customer-Managed Encryption Keys).
  - Integration with Cloud Run and GKE via volume mounts or environment variables.
  - Automatic rotation schedules triggered by Pub/Sub notifications and Cloud Functions.
- **External Secrets Operator (ESO) on GKE:**
  - Synchronizes secrets from Google Secret Manager into native Kubernetes `v1/Secret` resources, maintaining GitOps parity without committing secrets to Git.

---

## 4. Network Security & Perimeter Defense
- **VPC Service Controls (VPC-SC):**
  - Creates a security perimeter around Google Cloud APIs and storage (e.g., Cloud Storage, BigQuery, Secret Manager) to prevent data exfiltration.
  - Ingress and egress rules allow authorized cross-project or cross-perimeter data access based on identity and IP attributes.
- **Private Google Access & Private Service Connect (PSC):**
  - Allows VMs and GKE nodes without external public IPs to access Google Cloud APIs and managed services privately over internal Google backbone routing.
- **Cloud Armor & Edge Security:**
  - Enterprise DDoS protection and Web Application Firewall (WAF) integrated directly with Global External HTTP(S) Load Balancers.
  - OWASP Top 10 pre-configured rules, rate limiting, and geographic filtering.

---

## 5. Audit Logging & Compliance Monitoring
- **Cloud Audit Logs:**
  - **Admin Activity Logs:** Always enabled, cannot be disabled, retained for 400 days at zero cost. Records configuration changes.
  - **Data Access Logs:** Disabled by default (except for BigQuery) due to volume. Records read/write calls to user data (`ADMIN_READ`, `DATA_WRITE`, `DATA_READ`).
- **Log Sinks & Long-Term Archival:**
  - Export audit logs to BigQuery for SQL analysis and anomaly detection.
  - Route to Cloud Storage with Object Retention Lock for regulatory immutability (WORM compliance).
- **Security Command Center (SCC):**
  - Central vulnerability management, misconfiguration detection, and threat detection across GCP assets.
