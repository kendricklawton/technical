# Domain 5: Security, Governance & SRE Operations — Quiz 3

**Exam:** Google Cloud Certified: Professional Cloud DevOps Engineer  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
An automated security review discovers that a developer committed a service account private key file (`sa-key.json`) to a public GitHub repository. What should the DevOps engineer do immediately?
- A. Delete the commit from git history and force push.
- B. Delete the compromised service account key immediately in Google Cloud IAM, revoke any active tokens, review Cloud Audit Logs for unauthorized activity, and rotate affected credentials.
- C. Wait for the weekly security meeting.
- D. Change the service account display name.

---

### Question 2
What is the primary security advantage of using Google-managed GKE Autopilot clusters compared to standard GKE clusters?
- A. Autopilot clusters are free of charge.
- B. Autopilot enforces hardened node configurations, automatic node OS patching, disables privileged containers by default, and manages master/node security hardening automatically according to Google best practices.
- C. Autopilot eliminates the need for container images.
- D. Autopilot automatically makes all services public.

---

### Question 3
An enterprise requires that all developer access to production Google Cloud resources pass through context-aware access controls (evaluating user identity, device health, corporate network location, and MFA). Which Google Cloud solution implements this zero-trust architecture?
- A. Cloud VPN
- B. BeyondCorp Enterprise (Context-Aware Access / IAP)
- C. Bastion host with static SSH keys
- D. Open public load balancer

---

### Question 4
In Cloud Deploy, an engineer wants to ensure that releases to the `prod` target require manual approval by an authorized security lead before deployment proceeds. How is this configured?
- A. By editing the Kubernetes deployment YAML manually.
- B. Setting `requireApproval: true` on the `prod` target definition in the Cloud Deploy pipeline configuration.
- C. Disabling Cloud Deploy in production.
- D. Sending an email to the on-call engineer.

---

### Question 5
What is the recommended practice for managing sensitive environment variables in Cloud Run services?
- A. Hardcode them in the container source code.
- B. Pass them as plain-text build arguments in Cloud Build.
- C. Reference them directly from Google Secret Manager using Cloud Run's native Secret Manager integration (`--set-secrets`).
- D. Write them to an unencrypted text file on local disk.

---

### Question 6
A company wants to restrict Google Cloud resource creation so that compute resources can only be deployed in European regions (`europe-west1`, `europe-west3`). Which service provides this enforcement?
- A. Organization Policy Constraint (`constraints/gcp.resourceLocations`)
- B. Cloud NAT
- C. Cloud DNS
- D. IAM Role `Viewer`

---

### Question 7
Which security log type records every time an authorized user reads an object from a private Cloud Storage bucket?
- A. Admin Activity Log
- B. Data Access Audit Log (`DATA_READ`)
- C. Cloud Monitoring metric
- D. System Event Log

---

### Question 8
Why is it dangerous to grant the `roles/iam.serviceAccountUser` role to a developer at the project level without restrictions?
- A. It gives them read-only access to logs.
- B. It allows the developer to attach ANY service account in the project (including highly privileged administrative service accounts) to compute resources they create, effectively elevating their own privileges.
- C. It deletes all existing service accounts.
- D. It prevents developers from deploying code.

---

### Question 9
How can an SRE team enforce that all container images deployed to production are scanned for vulnerabilities and have an attestation generated automatically in Cloud Build?
- A. By running a manual script on developer laptops.
- B. Implementing an automated Cloud Build pipeline step that checks Container Analysis vulnerability scan results and uses the Cloud KMS API to sign a Binary Authorization attestation note.
- C. By trusting developers to only push clean images.
- D. By disabling Container Analysis.

---

### Question 10
What is the effect of enabling Shielded GKE Nodes?
- A. It increases network bandwidth by 200%.
- B. It provides cryptographic identity and integrity verification for nodes using Secure Boot, vTPM (virtual Trusted Platform Module), and integrity monitoring to defend against rootkits and kernel tampering.
- C. It encrypts all user traffic with a static password.
- D. It disables all node logging.

---

## Answer Key & Rationales

1. **B** — Once a key is exposed publicly, it must be considered compromised. Immediately revoke the key in IAM and audit Cloud Audit Logs for malicious usage.
2. **B** — GKE Autopilot applies security baselines, prevents privileged containers, restricts Linux capabilities, and manages automated node upgrades.
3. **B** — BeyondCorp Enterprise enforces Zero Trust context-aware access policies evaluating device state, location, and user identity.
4. **B** — Cloud Deploy targets support `requireApproval: true`, holding the rollout in a pending state until an authorized approver reviews it.
5. **C** — Cloud Run integrates natively with Secret Manager, mounting secrets as environment variables or files in-memory without exposing values in configuration files.
6. **A** — `constraints/gcp.resourceLocations` enforces geographical boundary constraints organization-wide.
7. **B** — Data Access `DATA_READ` logs track read operations on user data objects; Admin Activity only tracks metadata and configuration changes.
8. **B** — Project-level `serviceAccountUser` allows a principal to impersonate any service account by binding it to a VM or Cloud Run service, resulting in privilege escalation.
9. **B** — Automated CI/CD steps inspect Container Analysis results and sign Binary Authorization attestations via KMS before deployment.
10. **B** — Shielded Nodes leverage Secure Boot and vTPMs to ensure host kernels and boot components have not been modified or compromised.
