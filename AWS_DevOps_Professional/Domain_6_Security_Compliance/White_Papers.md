# Domain 6: Security and Compliance — Essential White Papers & Documentation Guide

> **Blueprint Weight:** 17% of AWS Certified DevOps Engineer - Professional (DOP-C02)  
> **Core Objective:** Design and implement automated security guardrails, encryption lifecycle management, fine-grained access controls, and compliance governance across multi-account enterprise organizations.

---

## 1. Priority AWS White Papers & Architecture Guides

### 1.1 [Security Pillar — AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)
* **Canonical Scope:** Architectural blueprint for securing workloads in the cloud across identity, detective controls, infrastructure protection, data protection, and incident response.
* **Key Principles:**
  * **Apply security at all layers:** Defense-in-depth across VPC edges, subnets, instances, load balancers, and application code.
  * **Automate security best practices:** Codifying compliance rules (AWS Config) and automated drift remediation.
  * **Protect data in transit and at rest:** Enforcing TLS 1.3, KMS CMKs with automatic annual key rotation, and S3 Default Encryption.
  * **Keep people away from data:** Eliminating direct SSH/RDP and production database access; enforcing Systems Manager Session Manager with session logging to encrypted S3.

### 1.2 [Building a Scalable and Secure Multi-Account AWS Environment](https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/welcome.html)
* **Canonical Scope:** Architecture of AWS Organizations, Organizational Units (OUs), and centralized governance.
* **Key Architectural Guardrails:**
  * **Service Control Policies (SCPs):** Coarse-grained guardrails setting maximum permissions across member accounts; SCPs never grant permissions, only filter them.
  * **Resource Control Policies (RCPs):** Setting maximum perimeter permissions on resources (e.g., S3, KMS, SQS) to prevent external data exfiltration.
  * **Declarative Policies & Tag Policies:** Enforcing mandatory resource tagging standards across accounts.

### 1.3 [AWS KMS Cryptographic Details Whitepaper](https://docs.aws.amazon.com/whitepapers/latest/kms-cryptographic-details/welcome.html)
* **Canonical Scope:** In-depth cryptographic architecture of AWS Key Management Service (KMS), envelope encryption, and HSM isolation.
* **Key Concepts:**
  * **Envelope Encryption:** KMS generates a plaintext data key and an encrypted data key; application encrypts data locally with plaintext key and immediately zeros memory; encrypted data key stored alongside ciphertext.
  * **Multi-Region Keys (MRKs):** Primary and replica keys sharing the same key material, ID, and ARN across regions for seamless multi-region decryption without cross-region network calls.
  * **KMS Grants:** Temporary, fine-grained delegations of cryptographic permissions (e.g., used by Auto Scaling service-linked roles to attach encrypted EBS volumes).

---

## 2. Official AWS Documentation & User Guides to Master

### 2.1 [AWS IAM & Permissions Boundaries](https://docs.aws.amazon.com/IAM/latest/UserGuide/)
* **Deep Dive:**
  * **Complete Policy Evaluation Logic:** Explicit Deny > SCP / RCP > Identity-based Policies > Permissions Boundary > Session Policies > Resource-based Policies.
  * **Permissions Boundaries:** Maximum allowed permissions boundary attached to developer-created IAM roles to safely delegate role creation without privilege escalation.
  * **IAM Roles Anywhere:** Securely issuing temporary AWS credentials to on-premises servers, containers, or GitHub runners using X.509 PKI certificates without hardcoding long-lived IAM keys.

### 2.2 [AWS Secrets Manager & Parameter Store](https://docs.aws.amazon.com/secretsmanager/latest/userguide/)
* **Deep Dive:**
  * **Automatic Secret Rotation:** Pre-built Lambda rotation templates (RDS, Aurora, DocumentDB) operating via 4-step rotation model: `createSecret`, `setSecret`, `testSecret`, `finishSecret`.
  * **Cross-Account Secret Replication:** Replicating secrets across regions and sharing access across accounts using resource policies.
  * **Kubernetes Secrets Store CSI Driver:** Mounting AWS Secrets Manager and Parameter Store secrets directly as files into Kubernetes Pods.

### 2.3 [AWS Config & Conformance Packs](https://docs.aws.amazon.com/config/latest/developerguide/)
* **Deep Dive:**
  * **Managed vs. Custom Rules:** Evaluating resource compliance continually or upon configuration changes.
  * **Conformance Packs:** Collections of AWS Config rules and remediation actions packaged as a single YAML template deployed organization-wide.
  * **Automatic Remediation:** Triggering Systems Manager Automation documents upon non-compliant findings (e.g., auto-enabling S3 block public access).

### 2.4 [Security Hub CSPM & GuardDuty](https://docs.aws.amazon.com/securityhub/latest/userguide/)
* **Deep Dive:**
  * Centralized security posture management aggregating findings across GuardDuty, Inspector v2, Macie, IAM Access Analyzer, and Firewall Manager.
  * Organization-wide central configuration enabling automated security standard enforcement across all accounts.

---

## 3. High-Frequency Exam Traps & Architectural Reflexes

1. **SCP "Root User" Bypass Fallacy:**
   * SCPs affect ALL users and roles in member accounts, **including the account root user**!
   * *Exception:* SCPs do NOT affect the root user or roles inside the **Organizations Management Account** or service-linked roles.
2. **KMS Key Policy Delegation:**
   * IAM policies CANNOT grant access to a KMS key unless the key policy explicitly contains a statement delegating access to the root account (`"Principal": {"AWS": "arn:aws:iam::<ACCOUNT_ID>:root"}`).
3. **Secrets Manager Rotation Window:**
   * Secret rotation does NOT delete or invalidate the previous secret version immediately; it maintains `AWSCURRENT` and `AWSPREVIOUS` version stages to prevent downtime during database password updates.

---

## 4. Reading & Study Checklist

- [ ] Read *Security Pillar — AWS Well-Architected Framework*
- [ ] Read *Building a Scalable and Secure Multi-Account AWS Environment*
- [ ] Read *AWS KMS Cryptographic Details Whitepaper*
- [ ] Master the IAM policy evaluation hierarchy and Permissions Boundaries
- [ ] Review AWS Secrets Manager 4-step rotation Lambda lifecycle
- [ ] Master AWS Config Conformance Packs and automated remediation workflows
