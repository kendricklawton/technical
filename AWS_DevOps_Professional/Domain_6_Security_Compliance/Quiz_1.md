# Domain 6: Security and Compliance — Knowledge Assessment & Mastery Quiz

**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Weight:** 17% of Exam  
**Format:** 15 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: IAM Permission Boundaries for Delegated Role Administration

An enterprise grants senior developers the ability to create and manage new IAM roles for AWS Lambda functions and Amazon ECS tasks within an isolated development account. The security compliance team mandates that developers must **never** be able to escalate their own privileges or create IAM roles that possess permissions beyond an approved ceiling (e.g. they must never grant full `AdministratorAccess` or modify billing or IAM policies).

**Which IAM architecture enforces this maximum permission ceiling on newly created roles?**
- A. Create an **IAM Permission Boundary** policy defining the maximum allowable permissions, and attach an IAM policy to developers allowing `iam:CreateRole` and `iam:PutRolePolicy` **only** when the request includes a condition enforcing that permission boundary: `StringEquals: { "iam:PermissionsBoundary": "arn:aws:iam::123456789012:policy/DeveloperBoundary" }`.
- B. Apply a Service Control Policy (SCP) to the development account that denies `iam:CreateRole` unless the user authenticates with hardware multi-factor authentication (MFA).
- C. Deploy an AWS Config rule that runs every 30 minutes to evaluate newly created IAM roles, invoking an AWS Lambda function to delete roles whose attached policies exceed 5 managed policies.
- D. Attach an IAM session policy to the developers' login role that restricts all downstream API actions to read-only actions on the `iam:*` namespace.

---

### Question 2: Secrets Manager Automated Database Secret Rotation Mechanics

An enterprise uses AWS Secrets Manager to store database master credentials for an Amazon RDS PostgreSQL database. The application runs across dozens of Amazon ECS tasks maintaining active database connection pools. The security team enables automatic secret rotation every 30 days using the **Single-User** rotation strategy. During rotation, active application tasks experience intermittent `PasswordAuthenticationFailed` connection errors.

**How does Secrets Manager Single-User rotation work, and how does the Alternating Users (Dual-User) strategy eliminate application downtime?**
- A. Single-User rotation immediately alters the active user's password in the database engine before updating cached application tokens, breaking existing connection pools until tasks refresh; **Alternating Users (Dual-User)** alternates between two distinct database users (`UserA` and `UserB`), rotating the inactive user's password and verifying it before updating Secrets Manager, ensuring uninterrupted client traffic.
- B. Single-User rotation requires manual application restarts after rotation executes, whereas Alternating Users rotation uses AWS KMS Multi-Region Keys to decrypt passwords in memory without restarting containers.
- C. Single-User rotation stores passwords as unencrypted plaintext in the Lambda rotation function, whereas Alternating Users rotation encrypts payloads using client-side envelope encryption.
- D. Single-User rotation rotates passwords sequentially across Availability Zones, whereas Alternating Users rotation updates all database read replicas simultaneously via Aurora Global Database.

---

### Question 3: Enterprise Service Control Policy (SCP) Enforcing Regional Data Sovereignty

A healthcare enterprise mandates that all cloud infrastructure and patient data must reside exclusively within United States regions (`us-east-1` and `us-west-2`) to comply with legal data sovereignty regulations. All member accounts in the organization must be prevented from creating resources in unapproved regions, while allowing global AWS services (IAM, Route 53, CloudFront, AWS Health) to function normally without disruption.

**Which Service Control Policy (SCP) statement satisfies these regulatory requirements?**
- A.
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "DenyUnapprovedRegionsExceptGlobal",
        "Effect": "Deny",
        "NotAction": [
          "iam:*", "organizations:*", "route53:*", "cloudfront:*",
          "support:*", "health:*", "waf:*", "wafv2:*", "sts:*"
        ],
        "Resource": "*",
        "Condition": {
          "StringNotEquals": {
            "aws:RequestedRegion": ["us-east-1", "us-west-2"]
          }
        }
      }
    ]
  }
  ```
- B.
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "DenyAllOutsideApprovedRegions",
        "Effect": "Deny",
        "Action": "*",
        "Resource": "*",
        "Condition": {
          "StringEquals": {
            "aws:RequestedRegion": ["eu-west-1", "ap-southeast-1"]
          }
        }
      }
    ]
  }
  ```
- C.
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": "*",
        "Resource": "*",
        "Condition": {
          "StringEquals": {
            "aws:RequestedRegion": ["us-east-1", "us-west-2"]
          }
        }
      }
    ]
  }
  ```
- D.
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Deny",
        "Action": "ec2:*",
        "Resource": "*",
        "Condition": {
          "ArnNotEquals": {
            "aws:PrincipalARN": "arn:aws:iam::*:role/AdminRole"
          }
        }
      }
    ]
  }
  ```

---

### Question 4: KMS Key Policy vs. IAM Policy Evaluation Logic

An administrator in Account `111111111111` has an attached IAM identity policy granting full administrative permissions: `{"Effect": "Allow", "Action": "*", "Resource": "*"}`. However, when the administrator executes `aws kms decrypt` on an Amazon S3 object encrypted with a Customer Managed Key (CMK) located in the same account, the request fails with an `AccessDeniedException`.

**Why was the decryption request rejected despite the user possessing AdministratorAccess in IAM?**
- A. In AWS KMS, an IAM policy alone cannot grant access to a Customer Managed Key (CMK); the **KMS Key Policy** is the primary access control mechanism. If the Key Policy does not explicitly delegate control to the account root principal (`"Principal": {"AWS": "arn:aws:iam::111111111111:root"}`) or explicitly include the administrator role, IAM identity policies have zero effect.
- B. KMS Customer Managed Keys can only be accessed using temporary security credentials generated by AWS STS via hardware MFA tokens.
- C. S3 server-side encryption with KMS Customer Managed Keys requires the user to assume the AWS S3 service-linked role before issuing decrypt commands.
- D. Customer Managed Keys enforce an immutable 24-hour propagation delay before new IAM administrator roles are recognized by the KMS key manager.

---

### Question 5: Continuous Automated Vulnerability Assessment with Amazon Inspector v2

A financial services company operates Amazon EKS clusters, containerized workloads in Amazon ECR, and serverless workflows in AWS Lambda. The Chief Information Security Officer (CISO) requires an automated vulnerability management solution that provides:
1. Continuous scanning of operating system packages and application dependencies in Amazon EC2 instances and EKS worker nodes.
2. Continuous scanning of container images in Amazon ECR upon push and continuously thereafter as new CVEs are published.
3. Code vulnerability scanning and package dependency scanning for AWS Lambda functions.

**Which AWS service provides continuous, automated assessment across all three workload categories?**
- A. Amazon Inspector v2: Natively integrates with Amazon EC2 (via SSM Agent or agentless), Amazon ECR (enhanced automated scanning), and AWS Lambda (standard dependency and code scanning) to provide unified, continuous CVE vulnerability scoring.
- B. Amazon GuardDuty: Continuously monitors VPC Flow Logs, DNS logs, CloudTrail management events, and Kubernetes audit logs to detect compromised instances and unauthorized API activity.
- C. AWS Security Hub: Compiles compliance posture assessments against CIS AWS Foundations Benchmarks and aggregated findings from third-party security appliances.
- D. Amazon Macie: Uses machine learning and pattern matching to discover, classify, and protect sensitive Personally Identifiable Information (PII) stored in Amazon S3 buckets.

---

### Question 6: Automated Sensitive Data Classification in S3 with Amazon Macie

A health-tech platform maintains hundreds of Amazon S3 buckets containing patient diagnostic records, medical imaging, and application telemetry. The compliance officer must ensure that unencrypted Personally Identifiable Information (PII)—such as Social Security Numbers (SSNs), passport numbers, and bank account numbers—is automatically detected across all buckets, and alerts are dispatched when sensitive data is exposed.

**Which architecture satisfies this compliance requirement across the entire AWS Organization?**
- A. Enable **Amazon Macie** at the AWS Organizations management or delegated administrator level, configure automated sensitive data discovery across all S3 buckets, and configure an Amazon EventBridge rule to route high-severity Macie findings to an SNS alert topic.
- B. Deploy an AWS CloudFormation StackSet that configures Amazon S3 Event Notifications on all buckets to invoke an AWS Lambda function that parses object text using regex.
- C. Configure AWS Config managed rule `s3-bucket-ssl-requests-only` with automated remediation enabled to encrypt non-compliant S3 buckets.
- D. Enable Amazon GuardDuty S3 Protection to inspect object payload contents and redact patient financial identifiers before objects are written to disk.

---

### Question 7: Mitigating Distributed Bot & Credential Stuffing Attacks with AWS WAF

An online retail platform experiences distributed credential-stuffing and automated scraper attacks against its customer authentication endpoint (`/api/v1/auth/login`). Attackers utilize residential proxy networks spanning hundreds of thousands of rotating IP addresses, making standard rate-based IP rules ineffective. The security team needs to block automated headless browsers and identify stolen credentials before they hit backend authentication servers.

**Which AWS WAF capability should the DevOps Engineer deploy on the Application Load Balancer?**
- A. Deploy the **AWS WAF Bot Control** managed rule group (to identify automated headless browsers, scrapers, and bot signatures) and the **AWS WAF Fraud Control - Account Takeover Prevention (ATP)** managed rule group (to detect credential stuffing and compromised password usage).
- B. Configure a standard AWS WAF Rate-Based Rule with an evaluation window of 5 minutes and an aggregation threshold of 100 requests per IP address.
- C. Deploy an Amazon CloudFront distribution in front of the Application Load Balancer and enable CloudFront Geographic Restrictions to block non-domestic country traffic.
- D. Configure an AWS Network Firewall stateful rule group that evaluates TCP SYN packet timing to drop requests originating from proxy networks.

---

### Question 8: Cross-Region Cryptography with AWS KMS Multi-Region Keys

An international enterprise operates an active-active multi-region application deployed across `us-east-1` (Primary) and `eu-central-1` (Disaster Recovery). Application data is encrypted at rest using an AWS KMS Customer Managed Key. During an emergency disaster recovery failover to `eu-central-1`, the application must decrypt existing database records and S3 objects locally without making cross-region KMS network calls to `us-east-1` and without re-encrypting the data.

**Which KMS key architecture satisfies these high-resilience requirements?**
- A. Create an **AWS KMS Multi-Region Key (MRK)**: Create a primary multi-region key in `us-east-1` and replicate it to `eu-central-1`; both keys share the same key ID, key material, and ARN prefix (`mrk-...`), allowing data encrypted in `us-east-1` to be decrypted natively in `eu-central-1`.
- B. Create two independent single-region Customer Managed Keys in `us-east-1` and `eu-central-1` with identical key aliases (`alias/app-encryption-key`), and grant cross-region access in the key policy.
- C. Export the private key material from AWS KMS using the `kms:ExportKey` API, and import the raw key bytes into an AWS CloudHSM cluster in `eu-central-1`.
- D. Use client-side envelope encryption with a shared passphrase stored in AWS Systems Manager Parameter Store replicated via AWS DataSync.

---

### Question 9: Cross-Account Encrypted AMI Sharing with KMS Grants

A central shared-services account (`111111111111`) builds and encrypts hardened golden AMIs using a Customer Managed Key (CMK). The AMIs must be shared with an application account (`222222222222`), where an Auto Scaling Group launches EC2 instances from the shared encrypted AMI. During instance launch, the Auto Scaling service fails with `Client.InternalError: Client error on launch`.

**Which combination of KMS and IAM permissions is required to allow the Auto Scaling service in the target account to use the encrypted AMI? (Select TWO)**
- A. In the KMS key policy in Account `111111111111`, grant `kms:DescribeKey`, `kms:ReEncrypt*`, and `kms:CreateGrant` permissions with condition `Bool: {"kms:GrantIsForAWSResource": "true"}` to Account `222222222222`'s **Auto Scaling Service-Linked Role** (`AWSServiceRoleForAutoScaling`).
- B. Share the encrypted AMI and its underlying EBS snapshots with Account `222222222222` using `ec2:ModifyImageAttribute` and `ec2:ModifySnapshotAttribute`.
- C. In Account `222222222222`, copy the unencrypted AMI snapshot locally and re-encrypt it using the default `aws/ebs` AWS-managed key.
- D. Attach an IAM policy to the EC2 Instance Profile in Account `222222222222` granting `kms:Decrypt` on the CMK in Account `111111111111`.
- E. Change the KMS key policy in Account `111111111111` to grant `kms:*` to `*` without conditions.

---

### Question 10: Zero-Trust On-Premises Workload Authentication via AWS IAM Roles Anywhere

An enterprise operates containerized batch-processing applications running in an on-premises corporate data center. These on-premises containers require temporary credentials to upload financial datasets to Amazon S3. Corporate security policy strictly prohibits the creation of long-lived IAM user access keys (`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`) and prohibits maintaining persistent site-to-site VPN tunnels for AWS Systems Manager agents.

**Which AWS authentication architecture meets these security mandates?**
- A. Deploy **AWS IAM Roles Anywhere**: Establish a **Trust Anchor** referencing the corporate internal Public Key Infrastructure (PKI) X.509 Certificate Authority (or AWS Private CA), configure an IAM Roles Anywhere **Profile** mapping allowed IAM roles, and use the `aws_signing_helper` credential helper to obtain short-lived STS tokens.
- B. Deploy an AWS Storage Gateway File Gateway in the on-premises data center, and configure local NFS mounts to sync data to S3 without IAM credentials.
- C. Create an IAM User with programmatic access, store the access keys in an on-premises HashiCorp Vault cluster, and rotate the keys every 90 days via an automated script.
- D. Configure Amazon Cognito User Pools with machine-to-machine OAuth 2.0 client credentials, and exchange Cognito JWTs for temporary STS credentials via Amazon API Gateway.

---

### Question 11: Modern Amazon EKS Workload Identity: EKS Pod Identity vs. IRSA

A platform engineering team manages 40 Amazon EKS clusters across multiple AWS accounts. Currently, pods authenticate to AWS APIs using legacy IAM Roles for Service Accounts (IRSA). Managing IRSA introduces severe operational overhead because every cluster requires an individual IAM OIDC Identity Provider, and shared IAM role trust policies repeatedly exceed the 2,048-character policy size limit when trusting dozens of cluster OIDC URLs.

**Which architecture should the platform team adopt to streamline workload identity management?**
- A. Migrate to **Amazon EKS Pod Identity**: Install the EKS Pod Identity Agent add-on on clusters, create IAM roles trusting the service principal `pods.eks.amazonaws.com`, and associate the roles directly with Kubernetes service accounts via the EKS API, eliminating OIDC providers and enabling cross-cluster role reuse.
- B. Store AWS IAM User access keys in Kubernetes Secrets and mount them as environment variables into application pod definitions.
- C. Deploy an AWS Systems Manager Session Manager agent sidecar inside every Kubernetes pod to allow pods to inherit the EC2 node's instance profile permissions.
- D. Use HashiCorp Vault to dynamically generate temporary IAM user access keys for every pod invocation using the Vault AWS Secrets Engine.

---

### Question 12: AWS Control Tower Governance: Preventative vs. Detective vs. Proactive Guardrails

An enterprise platform architecture team is establishing a multi-account compliance baseline using AWS Control Tower. The compliance officer defines three distinct requirements:
1. Prevent member accounts from ever creating unencrypted S3 buckets, blocking the API call before creation.
2. Flag existing resources that deviate from approved security configurations and publish compliance alerts.
3. Validate CloudFormation templates pre-deployment in CI/CD pipelines to prevent non-compliant resources from entering CloudFormation change sets.

**Which Control Tower control types map to these three requirements?**
- A. Requirement 1: **Preventative Controls** (implemented via Service Control Policies); Requirement 2: **Detective Controls** (implemented via AWS Config rules); Requirement 3: **Proactive Controls** (implemented via AWS CloudFormation Hooks).
- B. Requirement 1: Detective Controls (AWS Config); Requirement 2: Preventative Controls (SCPs); Requirement 3: Responsive Controls (SSM Automation).
- C. Requirement 1: Proactive Controls (CloudFormation Hooks); Requirement 2: Preventative Controls (SCPs); Requirement 3: Detective Controls (AWS Inspector).
- D. Requirement 1: Preventative Controls (IAM Policies); Requirement 2: Responsive Controls (Lambda); Requirement 3: Detective Controls (CloudTrail).

---

### Question 13: Enforcing Least Privilege via AWS Service Catalog Launch Constraints

An enterprise platform team needs to empower non-technical business analysts to provision approved analytics compute clusters (Amazon EMR and Amazon EC2) on demand. The analysts must **NOT** be granted direct IAM permissions for `ec2:*`, `emr:*`, `iam:PassRole`, or `cloudformation:*`, because direct access would allow them to bypass architectural standards or launch unapproved expensive instance types.

**How should this self-service provisioning capability be architected?**
- A. Publish the approved infrastructure templates as **AWS Service Catalog Products** into a portfolio shared with the analysts. Configure a **Launch Constraint** on the portfolio that specifies an administrative IAM role with necessary provisioning permissions; grant analysts IAM permissions exclusively for `servicecatalog:*`.
- B. Create an Amazon API Gateway endpoint backed by an AWS Lambda function that executes `cloudformation:CreateStack`, granting analysts IAM permissions to invoke the API.
- C. Attach an IAM permissions boundary to the analysts' IAM roles that restricts EC2 instance creation to approved instance types.
- D. Grant analysts temporary `AdministratorAccess` IAM credentials using AWS STS `GetSessionToken` with a 1-hour expiration window.

---

### Question 14: Centralized Threat Remediation with AWS Security Hub & Automated Response

A multi-account organization enables AWS Security Hub to monitor compliance with the CIS AWS Foundations Benchmark. When a critical finding occurs (such as an S3 bucket with public read access enabled or an IAM user with an access key unused for 90 days), the security team needs the finding remediated automatically across all member accounts without human intervention.

**Which architecture provides automated multi-account remediation for Security Hub findings?**
- A. Configure Security Hub to send findings to **Amazon EventBridge**. Create an EventBridge rule in the central security account matching `Security Hub Findings - Custom Action` or specific finding compliance statuses, targeting an **AWS Systems Manager Automation runbook** that executes remediation actions across member accounts.
- B. Deploy an AWS Lambda function that polls the Security Hub API every 5 minutes using `GetFindings` and calls the AWS CLI to alter non-compliant resources.
- C. Configure AWS CloudTrail to stream security events directly to an Amazon OpenSearch Service cluster, and use OpenSearch Alerting to trigger shell scripts.
- D. Enable AWS Config Conformance Packs in each member account with manual ticketing integrations via Amazon SNS.

---

### Question 15: Automated Threat Containment with Amazon GuardDuty & AWS Step Functions

An Amazon GuardDuty detector identifies anomalous API activity indicating that an IAM user's access keys have been compromised and are being used from an unauthorized external IP address to enumerate AWS infrastructure (`UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration`). An automated incident response workflow must immediately contain the compromise, revoke active credentials, and alert the security operations center.

**Which automated incident response sequence satisfies these containment requirements?**
- A. Configure an Amazon EventBridge rule matching the specific GuardDuty finding type, targeting an **AWS Step Functions state machine** that: 1) calls `iam:UpdateAccessKey` to deactivate the compromised key; 2) attaches an inline IAM policy denying all actions (`Effect: Deny, Action: *, Resource: *`); 3) revokes active IAM sessions by attaching an `iam:PutUserPolicy` statement invalidating tokens issued before the current timestamp; and 4) publishes an urgent incident alert to an Amazon SNS topic.
- B. Configure GuardDuty to delete the compromised IAM user automatically using the native GuardDuty remediation console setting.
- C. Write an Amazon CloudWatch Alarm monitoring GuardDuty finding volume, and trigger an AWS Lambda function to reboot all running EC2 instances.
- D. Deploy an AWS WAF Web ACL to block the compromised IAM user's IP address from accessing the AWS Management Console login page.

---

## Answer Key & Detailed Explanations

### Question 1: IAM Permission Boundaries for Delegated Role Administration
**Answer:** A  
**Explanation:**  
An **IAM Permission Boundary** defines the absolute maximum permission ceiling that an identity-based policy can grant to an IAM entity. By attaching a condition `StringEquals: {"iam:PermissionsBoundary": "arn:aws:iam::...:policy/DeveloperBoundary"}` to developers' `iam:CreateRole` and `iam:PutRolePolicy` permissions, developers are physically prevented from creating any role unless the approved boundary is attached. If a developer attempts to create a role with `AdministratorAccess`, the effective permissions of the created role are strictly constrained to the intersection of the role policy and the boundary, preventing privilege escalation.

---

### Question 2: Secrets Manager Automated Database Secret Rotation Mechanics
**Answer:** A  
**Explanation:**  
In **Single-User rotation**, Secrets Manager updates the database user's password directly in the database engine before updating the stored secret value. Any application holding connections established with the old password (or using a connection pool) will fail to establish new connections until the application refreshes the secret. In **Alternating Users (Dual-User) rotation**, Secrets Manager alternates between two distinct users (`UserA` and `UserB`). While `UserA` remains active and serves live application traffic, Secrets Manager rotates and verifies `UserB`'s password. Once verified, Secrets Manager promotes `UserB` to current (`AWSCURRENT`), ensuring zero application connection dropouts.

---

### Question 3: Enterprise Service Control Policy (SCP) Enforcing Regional Data Sovereignty
**Answer:** A  
**Explanation:**  
Statement A correctly implements the AWS-recommended region-restriction SCP pattern. It uses an explicit `Deny` with a `NotAction` block that exempts global services whose control planes operate in `us-east-1` but serve global requests (such as `iam:*`, `organizations:*`, `route53:*`, `cloudfront:*`, `support:*`, `health:*`, `waf:*`, and `sts:*`). It applies a condition `StringNotEquals: {"aws:RequestedRegion": ["us-east-1", "us-west-2"]}`, blocking API actions directed to any other regional endpoint.

---

### Question 4: KMS Key Policy vs. IAM Policy Evaluation Logic
**Answer:** A  
**Explanation:**  
AWS KMS Customer Managed Keys (CMKs) use their **Key Policy** as the primary evaluation filter. IAM policies cannot grant access to a CMK unless the Key Policy explicitly includes a delegation statement granting access to the root account (`"Principal": {"AWS": "arn:aws:iam::111111111111:root"}`). This delegation statement allows the account to use IAM identity-based policies to control key access. If this statement is missing, or if the Key Policy contains an explicit `Deny`, even principals with `AdministratorAccess` in IAM are denied access to the key.

---

### Question 5: Continuous Automated Vulnerability Assessment with Amazon Inspector v2
**Answer:** A  
**Explanation:**  
**Amazon Inspector v2** is a consolidated vulnerability management service that continuously and automatically scans workloads for software vulnerabilities (CVEs) and unintended network exposure. It natively supports three distinct compute tiers: 1) Amazon EC2 instances (using the SSM Agent or agentless EBS volume scanning); 2) Amazon ECR container images (continuous enhanced scanning upon push and rescan upon new CVE publication); and 3) AWS Lambda functions (scanning application package dependencies and function code).

---

### Question 6: Automated Sensitive Data Classification in S3 with Amazon Macie
**Answer:** A  
**Explanation:**  
**Amazon Macie** is a fully managed data security and privacy service that uses machine learning and pattern-matching algorithms to discover, classify, and protect sensitive Personally Identifiable Information (PII) in Amazon S3. When enabled across an AWS Organization, Macie performs automated sensitive data discovery across all member account S3 buckets, evaluates data encryption and public accessibility, and emits detailed finding events to Amazon EventBridge.

---

### Question 7: Mitigating Distributed Bot & Credential Stuffing Attacks with AWS WAF
**Answer:** A  
**Explanation:**  
Standard rate-based WAF rules fail against distributed botnets because requests are spread across thousands of distinct residential IP addresses. **AWS WAF Bot Control** identifies and blocks automated bots, scrapers, and headless browsers by inspecting TLS fingerprints, browser heuristics, and client challenges. The **AWS WAF Fraud Control - Account Takeover Prevention (ATP)** managed rule group specifically protects login endpoints against credential-stuffing attacks by analyzing login attempts against a global database of stolen and compromised credentials.

---

### Question 8: Cross-Region Cryptography with AWS KMS Multi-Region Keys
**Answer:** A  
**Explanation:**  
**AWS KMS Multi-Region Keys (MRKs)** are designed for multi-region active-active architectures and disaster recovery. An MRK consists of a primary key in one region and replica keys in other regions. They share the exact same Key ID, key material, and ARN prefix (`mrk-...`). Because the cryptographic key material is identical, ciphertext generated by the key in `us-east-1` can be decrypted directly by the replica key in `eu-central-1` without transferring plaintext data across regions and without re-encrypting underlying databases during failover.

---

### Question 9: Cross-Account Encrypted AMI Sharing with KMS Grants
**Answer:** A, B  
**Explanation:**  
When an encrypted AMI is shared across accounts, the target account's Auto Scaling Group cannot launch instances unless: 1) The AMI and its snapshots are shared with the target account (Action B); and 2) The source KMS key policy grants `kms:DescribeKey`, `kms:ReEncrypt*`, and `kms:CreateGrant` (with condition `Bool: {"kms:GrantIsForAWSResource": "true"}`) directly to the target account's **Auto Scaling Service-Linked Role** (`AWSServiceRoleForAutoScaling`) (Action A). The ASG service-linked role in the target account uses `CreateGrant` to delegate cryptographic operations to the EC2 hypervisor during launch.

---

### Question 10: Zero-Trust On-Premises Workload Authentication via AWS IAM Roles Anywhere
**Answer:** A  
**Explanation:**  
**AWS IAM Roles Anywhere** allows on-premises workloads (such as servers, containers, or Kubernetes pods) to obtain temporary, short-lived AWS IAM credentials using X.509 digital certificates issued by an internal Public Key Infrastructure (PKI) or AWS Private CA. By establishing a Trust Anchor and Profile in IAM Roles Anywhere, the `aws_signing_helper` utility signs requests using the server's private key to call STS `CreateSession`, completely eliminating the need for long-lived IAM user access keys.

---

### Question 11: Modern Amazon EKS Workload Identity: EKS Pod Identity vs. IRSA
**Answer:** A  
**Explanation:**  
**Amazon EKS Pod Identity** is the modern replacement for IAM Roles for Service Accounts (IRSA). With EKS Pod Identity, cluster administrators install the EKS Pod Identity Agent add-on. IAM roles configure a trust policy with service principal `pods.eks.amazonaws.com` rather than federated OIDC provider ARNs. Roles are associated with Kubernetes ServiceAccounts directly through EKS API calls or console settings. The exact same IAM role can be reused across hundreds of EKS clusters simultaneously without hitting the 2,048-character IAM role trust policy limit.

---

### Question 12: AWS Control Tower Governance: Preventative vs. Detective vs. Proactive Guardrails
**Answer:** A  
**Explanation:**  
AWS Control Tower categorizes governance controls into three enforcement paradigms:
1. **Preventative Controls**: Enforced using AWS Organizations **Service Control Policies (SCPs)** that disallow API calls entirely, physically blocking non-compliant actions before execution.
2. **Detective Controls**: Enforced using **AWS Config Rules** that continuously evaluate deployed resources post-creation, flagging non-compliant configurations in dashboards.
3. **Proactive Controls**: Enforced using **AWS CloudFormation Hooks** that inspect templates during the pre-provisioning phase, halting non-compliant deployments before resources are provisioned.

---

### Question 13: Enforcing Least Privilege via AWS Service Catalog Launch Constraints
**Answer:** A  
**Explanation:**  
**AWS Service Catalog Launch Constraints** allow organizations to enforce strict least-privilege access. A portfolio containing approved infrastructure products (e.g. EC2/EMR clusters) is shared with end users. A Launch Constraint specifies a dedicated IAM role (the launch role) that possesses the permissions required to provision the underlying CloudFormation templates and resources. End users only need permissions for `servicecatalog:*` and require zero direct permissions for `ec2:*` or `cloudformation:*`, preventing privilege escalation and unapproved resource creation.

---

### Question 14: Centralized Threat Remediation with AWS Security Hub & Automated Response
**Answer:** A  
**Explanation:**  
AWS Security Hub integrates with **Amazon EventBridge** to enable automated incident response. When Security Hub generates or updates a finding, it publishes an event to EventBridge. By configuring EventBridge rules matching specific finding titles or compliance statuses (e.g. CIS Benchmark failures), EventBridge automatically invokes **AWS Systems Manager Automation runbooks** to remediate the non-compliant resource across accounts (e.g. disabling public S3 bucket policies or deactivating stale IAM access keys).

---

### Question 15: Automated Threat Containment with Amazon GuardDuty & AWS Step Functions
**Answer:** A  
**Explanation:**  
When Amazon GuardDuty flags compromised IAM credentials, rapid automated containment is critical. GuardDuty emits finding events to Amazon EventBridge. An EventBridge rule matching finding `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration` invokes an **AWS Step Functions state machine** that coordinates the containment runbook: 1) deactivating the compromised access key (`UpdateAccessKey`); 2) attaching an explicit inline `Deny` policy to the user; 3) revoking all active sessions by attaching an IAM policy with a `aws:CurrentTime` condition; and 4) publishing an emergency page to the security operations center.
