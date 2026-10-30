# Domain 3: Designing for Security, Identity, and Compliance — Quiz 2

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
A financial institution requires that encryption keys used for encrypting credit card numbers in BigQuery must be stored in physical Hardware Security Modules (HSMs) located inside the bank's private on-premises datacenter. The bank must retain absolute physical custody of the keys at all times. Which encryption solution satisfies this requirement?
- A. Cloud KMS default software keys
- B. Cloud External Key Manager (Cloud EKM)
- C. Google default encryption
- D. Cloud Storage Nearline

---

### Question 2
An enterprise security team wants to continuously monitor their Google Cloud organization for security misconfigurations, such as public Cloud Storage buckets, open firewall ports (`0.0.0.0/0:22`), and outdated GKE cluster versions. Which centralized service provides this posture management?
- A. Security Command Center (SCC) with Security Health Analytics
- B. Cloud Trace
- C. Cloud Profiler
- D. Compute Engine Autoscaler

---

### Question 3
Under Google Cloud's Shared Responsibility Model, which of the following security aspects is the sole responsibility of the customer when running workloads on Compute Engine (IaaS)?
- A. Physical data center access security
- B. Guest operating system patching, application security, and firewall rule configurations
- C. Server hardware decommissioning and disk destruction
- D. Physical network fiber maintenance

---

### Question 4
An architect is reviewing service account usage in a production environment. Several developers have been granted `roles/iam.serviceAccountKeyAdmin`, and dozens of static JSON keys have been downloaded to local laptops. What is the primary security risk of this practice, and how should it be mitigated?
- A. Static keys slow down network connections; use Cloud NAT.
- B. Unmanaged static JSON keys risk credential leakage, have no automatic expiration, and are difficult to rotate; keys should be deleted, and developers should use short-lived credentials via IAM user authentication or Workload Identity Federation.
- C. JSON keys cause billing spikes.
- D. Google Cloud does not allow service account keys.

---

### Question 5
A company is designing a multi-tenant SaaS application on GKE where each customer's pods run in separate Kubernetes namespaces. The architect must ensure that pods in Customer A's namespace cannot initiate network connections to pods in Customer B's namespace. What Kubernetes resource enforces this network isolation?
- A. Kubernetes NetworkPolicy
- B. Compute Engine Firewall rules
- C. Cloud Armor
- D. Cloud DNS

---

### Question 6
An architect needs to configure Access Approval for an enterprise organization. What is the operational purpose of Access Approval?
- A. It allows developers to approve their own pull requests.
- B. It requires designated corporate security administrators to explicitly approve or deny access requests from Google personnel before Google can access customer data during support interventions.
- C. It approves cloud billing invoices.
- D. It speeds up VM boot times.

---

### Question 7
A healthcare application handles Protected Health Information (PHI) under HIPAA regulations. What legal agreement must be executed with Google Cloud before the application can legally store or process PHI in Google Cloud services?
- A. Non-Disclosure Agreement (NDA)
- B. Business Associate Agreement (BAA)
- C. Master Services Agreement (MSA) only
- D. End User License Agreement (EULA)

---

### Question 8
An organization wants to restrict access to their internal web portal so that only employees using corporate-managed laptops with disk encryption enabled can access the portal, even if personal employee credentials are valid. Which technology enforces this device-state access verification?
- A. BeyondCorp Enterprise with Context-Aware Access and Endpoint Verification
- B. Cloud NAT
- C. Basic authentication
- D. VPC Peering

---

### Question 9
A company wants to prevent any user in the organization from creating new Google Cloud projects unless the request has gone through an automated IT approval workflow. Where should project creation permissions be restricted?
- A. Remove the `roles/resourcemanager.projectCreator` role from `default` domain users at the root Organization level.
- B. Delete all billing accounts.
- C. Block internet access in the corporate office.
- D. Remove all Compute Engine instances.

---

### Question 10
When using Cloud KMS, what occurs when a customer "destroys" a CryptoKey version?
- A. The data encrypted with the key is instantly deleted from Cloud Storage.
- B. The key material is scheduled for destruction (default 24-hour waiting period), after which it is permanently deleted from the KMS, rendering any data encrypted with that key version irreversibly unrecoverable.
- C. The key is archived to tape for 50 years.
- D. Google sends a replacement key via email.

---

## Answer Key & Rationales

1. **B** — Cloud External Key Manager (Cloud EKM) allows GCP services (BigQuery, Compute Engine) to use keys stored directly in customer-owned on-premises HSMs.
2. **A** — Security Command Center with Security Health Analytics automatically scans for common misconfigurations and compliance violations.
3. **B** — In IaaS (Compute Engine), the customer owns guest OS updates, application configuration, firewall rules, and IAM policies.
4. **B** — Downloaded service account keys represent high-risk long-lived credentials. Best practice dictates eliminating keys in favor of short-lived federated credentials.
5. **A** — Kubernetes NetworkPolicies enforce East-West zero-trust pod isolation between namespaces at the cluster network layer.
6. **B** — Access Approval gives customers cryptographic control over whether Google personnel can access data during customer support operations.
7. **B** — Under HIPAA, a Business Associate Agreement (BAA) must be signed between the covered entity and cloud provider before storing PHI.
8. **A** — BeyondCorp Enterprise Context-Aware Access evaluates device posture (Endpoint Verification, disk encryption, OS patch level) before granting access.
9. **A** — Revoking `roles/resourcemanager.projectCreator` at the Organization level prevents unauthorized project proliferation and centralizes project provisioning.
10. **B** — Key destruction permanently erases the key material, achieving cryptographic erasure of any datasets encrypted with that key.
