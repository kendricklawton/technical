# Domain 3: Designing for Security, Identity, and Compliance — Quiz 2

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
A financial institution requires that encryption keys used for encrypting credit card numbers in BigQuery must be stored in physical Hardware Security Modules (HSMs) located inside the bank's private on-premises datacenter. The bank must retain absolute physical custody of the keys at all times. Which encryption solution satisfies this requirement?
- A. Google default encryption
- B. Cloud KMS default software keys
- C. Cloud External Key Manager (Cloud EKM)
- D. Cloud Storage Nearline

---

### Question 2
An enterprise security team wants to continuously monitor their Google Cloud organization for security misconfigurations, such as public Cloud Storage buckets, open firewall ports (`0.0.0.0/0:22`), and outdated GKE cluster versions. Which centralized service provides this posture management?
- A. Cloud Profiler
- B. Security Command Center (SCC) with Security Health Analytics
- C. Cloud Trace
- D. Compute Engine Autoscaler

---

### Question 3
Under Google Cloud's Shared Responsibility Model, which of the following security aspects is the sole responsibility of the customer when running workloads on Compute Engine (IaaS)?
- A. Server hardware decommissioning and disk destruction
- B. Physical network fiber maintenance
- C. Physical data center access security
- D. Guest operating system patching, application security, and firewall rule configurations

---

### Question 4
An architect is reviewing service account usage in a production environment. Several developers have been granted `roles/iam.serviceAccountKeyAdmin`, and dozens of static JSON keys have been downloaded to local laptops. What is the primary security risk of this practice, and how should it be mitigated?
- A. Unmanaged static JSON keys risk credential leakage, have no automatic expiration, and are difficult to rotate; keys should be deleted, and developers should use short-lived credentials via IAM user authentication or Workload Identity Federation.
- B. Google Cloud does not allow service account keys.
- C. JSON keys cause billing spikes.
- D. Static keys slow down network connections; use Cloud NAT.

---

### Question 5
A company is designing a multi-tenant SaaS application on GKE where each customer's pods run in separate Kubernetes namespaces. The architect must ensure that pods in Customer A's namespace cannot initiate network connections to pods in Customer B's namespace. What Kubernetes resource enforces this network isolation?
- A. Cloud DNS
- B. Cloud Armor
- C. Compute Engine Firewall rules
- D. Kubernetes NetworkPolicy

---

### Question 6
An architect needs to configure Access Approval for an enterprise organization. What is the operational purpose of Access Approval?
- A. It approves cloud billing invoices.
- B. It speeds up VM boot times.
- C. It allows developers to approve their own pull requests.
- D. It requires designated corporate security administrators to explicitly approve or deny access requests from Google personnel before Google can access customer data during support interventions.

---

### Question 7
A healthcare application handles Protected Health Information (PHI) under HIPAA regulations. What legal agreement must be executed with Google Cloud before the application can legally store or process PHI in Google Cloud services?
- A. Master Services Agreement (MSA) only
- B. Business Associate Agreement (BAA)
- C. Non-Disclosure Agreement (NDA)
- D. End User License Agreement (EULA)

---

### Question 8
An organization wants to restrict access to their internal web portal so that only employees using corporate-managed laptops with disk encryption enabled can access the portal, even if personal employee credentials are valid. Which technology enforces this device-state access verification?
- A. VPC Peering
- B. BeyondCorp Enterprise with Context-Aware Access and Endpoint Verification
- C. Cloud NAT
- D. Basic authentication

---

### Question 9
A company wants to prevent any user in the organization from creating new Google Cloud projects unless the request has gone through an automated IT approval workflow. Where should project creation permissions be restricted?
- A. Block internet access in the corporate office.
- B. Remove all Compute Engine instances.
- C. Remove the `roles/resourcemanager.projectCreator` role from `default` domain users at the root Organization level.
- D. Delete all billing accounts.

---

### Question 10
When using Cloud KMS, what occurs when a customer "destroys" a CryptoKey version?
- A. The key material is scheduled for destruction (default 24-hour waiting period), after which it is permanently deleted from the KMS, rendering any data encrypted with that key version irreversibly unrecoverable.
- B. The data encrypted with the key is instantly deleted from Cloud Storage.
- C. Google sends a replacement key via email.
- D. The key is archived to tape for 50 years.

---

## Answer Key & Rationales

1. **C** — Cloud External Key Manager (Cloud EKM) allows GCP services (BigQuery, Compute Engine) to use keys stored directly in customer-owned on-premises HSMs.
2. **B** — Security Command Center with Security Health Analytics automatically scans for common misconfigurations and compliance violations.
3. **D** — In IaaS (Compute Engine), the customer owns guest OS updates, application configuration, firewall rules, and IAM policies.
4. **A** — Downloaded service account keys represent high-risk long-lived credentials. Best practice dictates eliminating keys in favor of short-lived federated credentials.
5. **D** — Kubernetes NetworkPolicies enforce East-West zero-trust pod isolation between namespaces at the cluster network layer.
6. **D** — Access Approval gives customers cryptographic control over whether Google personnel can access data during customer support operations.
7. **B** — Under HIPAA, a Business Associate Agreement (BAA) must be signed between the covered entity and cloud provider before storing PHI.
8. **B** — BeyondCorp Enterprise Context-Aware Access evaluates device posture (Endpoint Verification, disk encryption, OS patch level) before granting access.
9. **C** — Revoking `roles/resourcemanager.projectCreator` at the Organization level prevents unauthorized project proliferation and centralizes project provisioning.
10. **A** — Key destruction permanently erases the key material, achieving cryptographic erasure of any datasets encrypted with that key.
