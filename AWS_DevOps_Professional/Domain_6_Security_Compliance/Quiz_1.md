# Domain 6: Security and Compliance — Knowledge Assessment & Mastery Quiz

**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Weight:** 17% of Exam  
**Format:** 15 Deep Scenario-Based Professional Practice Questions with Detailed Explanations

---

### Question 1: IAM Permission Boundaries for Delegated Role Administration

An enterprise grants senior developers permissions to create new IAM roles for AWS Lambda functions and Amazon ECS tasks in a sandbox account. The security compliance team mandates that developers must **never** be able to escalate their own privileges or create IAM roles that possess permissions beyond an approved ceiling (e.g. they must never grant full `AdministratorAccess` or modify billing).

**Which IAM feature enforces this maximum permission ceiling on newly created roles?**
- A. Service Control Policies (SCPs) applied to individual IAM users.
- B. **IAM Permission Boundaries**. An IAM policy defining the maximum allowable permissions is created. The developers' IAM policy allows `iam:CreateRole` and `iam:PutRolePolicy` **only** if the request includes a condition requiring the permission boundary to be attached: `StringEquals: { "iam:PermissionsBoundary": "arn:aws:iam::123456789012:policy/DeveloperBoundary" }`.
- C. AWS Config rule running every hour.
- D. Requiring MFA for all developer actions.

---

### Question 2: Secrets Manager Automated Database Secret Rotation Mechanics

An organization uses AWS Secrets Manager to store database master credentials for an Amazon RDS MySQL database. The security team enables automatic secret rotation every 30 days using the **Single-User** rotation strategy. During rotation, an application pod attempts to connect to the database using the cached secret and fails with `Access Denied`.

**How does Secrets Manager Single-User rotation work, and how does the Dual-User strategy eliminate application downtime during rotation?**
- A. Single-user rotation changes the password in the database and Secrets Manager at the exact same instant, so downtime cannot occur.
- B. In **Single-User rotation**, Secrets Manager changes the database user's password directly, rendering any application holding the old password invalid until it refreshes the secret. In **Dual-User rotation**, Secrets Manager alternates between two database users (`UserA` and `UserB`): one user is active while the other user's password is rotated and verified, allowing applications to continue connecting without interruption.
- C. Dual-user rotation requires two separate AWS accounts.
- D. Single-user rotation uses AWS KMS, whereas dual-user rotation uses asymmetric RSA keys.

---

### Question 3: Service Control Policy (SCP) Restricting AWS Regions

An enterprise requires that workloads and data must reside exclusively within the United States regions (`us-east-1` and `us-west-2`) to comply with legal data sovereignty requirements. All member accounts in the organization must be prevented from creating resources in any other AWS region, while allowing global services (IAM, Route 53, CloudFront) to function normally.

**Which SCP statement enforces this geographic guardrail?**
- A.
  ```json
  {
    "Effect": "Deny",
    "NotAction": [
      "iam:*",
      "organizations:*",
      "route53:*",
      "cloudfront:*",
      "support:*"
    ],
    "Resource": "*",
    "Condition": {
      "StringNotEquals": {
        "aws:RequestedRegion": [
          "us-east-1",
          "us-west-2"
        ]
      }
    }
  }
  ```
- B.
  ```json
  {
    "Effect": "Deny",
    "Action": "*",
    "Resource": "*",
    "Condition": {
      "StringEquals": {
        "aws:RequestedRegion": "eu-west-1"
      }
    }
  }
  ```
- C. Delete the VPCs in all other AWS regions.
- D. Disable the AWS accounts in other regions.

---

### Question 4: KMS Key Policy vs. IAM Policy for Access Control

An IAM user in Account `111111111111` has an attached IAM policy granting `AdministratorAccess` (`Action: "*", Resource: "*"`). However, when the user attempts to run `aws kms decrypt` on an Amazon S3 object encrypted with a Customer Managed Key (CMK) in the same account, the operation is rejected with `AccessDeniedException`.

**Why was the decryption request rejected despite the user having AdministratorAccess?**
- A. The IAM policy is cached and takes 4 hours to propagate.
- B. In AWS KMS, an IAM policy alone is insufficient; access to a Customer Managed Key is governed primarily by the **KMS Key Policy**. If the Key Policy does not explicitly grant access to the root account (`"Principal": {"AWS": "arn:aws:iam::111111111111:root"}`) to delegate access to IAM policies, or if the Key Policy lacks a statement allowing the user, IAM policies have zero effect.
- C. S3 objects can only be decrypted by the S3 service principal.
- D. KMS Customer Managed Keys can only be accessed using the root account credentials.

---

### Question 5: AWS Inspector v2 Automated Vulnerability Management

A company runs containerized microservices on Amazon EKS. The security team mandates automated, continuous vulnerability assessments for:
1. Operating system package vulnerabilities in Amazon EC2 instances.
2. Container images pushed to Amazon Elastic Container Registry (Amazon ECR).
3. Software dependencies inside AWS Lambda function application code.

**Which AWS service provides continuous, automated scanning across all three resource types?**
- A. Amazon Macie
- B. **Amazon Inspector v2**
- C. AWS Trusted Advisor
- D. AWS WAF

---

### Question 6: Amazon Macie for Sensitive Data Discovery in S3

A financial services firm maintains hundreds of Amazon S3 buckets. The compliance officer needs to verify whether any S3 buckets contain unencrypted Personally Identifiable Information (PII), such as Social Security Numbers (SSNs), credit card numbers, or passport numbers, and receive automated alerts when sensitive data is discovered.

**Which service should be deployed across the AWS Organization?**
- A. Amazon GuardDuty
- B. **Amazon Macie**
- C. AWS Config
- D. Amazon CloudWatch Logs Insights

---

### Question 7: AWS WAF Bot Control and Account Takeover Prevention

A ticketing website experiences automated bot attacks that submit thousands of login requests per minute to steal user credentials and scrape ticket availability. The attackers cycle through thousands of rotating residential proxy IP addresses, bypassing standard IP rate-limiting rules.

**Which AWS WAF feature mitigates these distributed automated bot attacks?**
- A. Standard IP Set match rules.
- B. **AWS WAF Bot Control** and **AWS WAF Fraud Control - Account Takeover Prevention (ATP)** managed rule groups.
- C. CloudFront Geo-blocking.
- D. Setting ALB idle timeout to 10 seconds.

---

### Question 8: KMS Multi-Region Keys for Disaster Recovery

An enterprise deploys a multi-region active-active application across `us-east-1` and `eu-west-1`. Data encrypted with AWS KMS in `us-east-1` must be decrypted in `eu-west-1` without transferring plaintext data across regions or re-encrypting existing databases during regional failover.

**Which KMS key architecture satisfies this requirement?**
- A. Single-region Customer Managed Keys in each region with the same key alias.
- B. **AWS KMS Multi-Region Keys** (a Primary Key in `us-east-1` and a Replica Key in `eu-west-1`), which share the same Key ID, key material, and key ARN across regions.
- C. Storing plaintext data in S3 and using client-side encryption with a shared password.
- D. Exporting the private key material from KMS to an on-premises HSM.

---

### Question 9: AWS Config Custom Rules with Lambda for Specific Compliance Logic

A corporate compliance rule mandates that any Amazon EC2 instance with the tag `Environment=Production` must have the tag `CostCenter` present, and the value of `CostCenter` must match an approved 4-digit code (e.g. `1001`, `1002`). AWS Config does not provide a managed rule for this specific regular expression validation.

**How should this compliance check be implemented in AWS Config?**
- A. Create an **AWS Config Custom Rule** backed by an **AWS Lambda function**. The Lambda function is triggered by configuration changes (`ConfigurationItemChangeNotification`), inspects the resource tags, and calls the `config:PutEvaluations` API to report compliance status (`COMPLIANT` or `NON_COMPLIANT`).
- B. Use an IAM policy to enforce the tag during `RunInstances`.
- C. Run a cron job on EC2 instances that checks tags locally.
- D. Configure an S3 lifecycle rule.

---

### Question 10: Cross-Account IAM Role Chaining Session Duration Limits

A developer in Account `A` assumes Role `B` in Account `B`, and from Role `B` assumes Role `C` in Account `C` (role chaining). Role `C` is configured with a maximum session duration of 12 hours. However, when the developer assumes Role `C` via role chaining, the temporary session expires after only 1 hour.

**Why did the temporary session expire after 1 hour despite Role C's 12-hour configuration?**
- A. The STS endpoint in Account C was unreachable.
- B. In AWS IAM, when you use **Role Chaining** (assuming a role from another role using `sts:AssumeRole`), the maximum session duration is hard-limited by AWS to **1 hour**, regardless of the role's configured `MaxSessionDuration` attribute.
- C. The developer's initial IAM user session expired.
- D. Cross-account roles are limited to 15 minutes by default.

---

### Question 11: GuardDuty Malware Protection for Amazon EBS

A company needs automated malware scanning for Amazon EC2 instances. If an instance behaves suspiciously (e.g. communicates with a cryptocurrency mining pool), an automated deep scan of the underlying Amazon EBS volume must be executed without installing third-party antivirus agents on the operating system.

**Which AWS service feature provides agentless malware scanning of EBS volumes?**
- A. Amazon Inspector v1
- B. **Amazon GuardDuty Malware Protection**, which automatically creates a snapshot of the EBS volume, attaches it to an isolated scanner instance in a service account, scans the volume for trojans and malware, and outputs detailed findings to GuardDuty.
- C. AWS Systems Manager Inventory
- D. AWS Trusted Advisor

---

### Question 12: Restricting IAM Roles to Specific AWS Services (`aws:SourceArn`)

A security auditor identifies a confused deputy risk in an IAM role assumed by AWS services. An IAM role trust policy allows `Principal: { "Service": "codebuild.amazonaws.com" }` without any condition keys, allowing any CodeBuild project in any AWS account to potentially assume the role.

**Which condition keys should be added to the trust policy to eliminate this vulnerability?**
- A. Add `Condition: { "StringEquals": { "aws:SourceAccount": "111111111111" }, "ArnLike": { "aws:SourceArn": "arn:aws:codebuild:us-east-1:111111111111:project/*" } }`.
- B. Add `Condition: { "Bool": { "aws:SecureTransport": "true" } }`.
- C. Remove the trust policy and use access keys instead.
- D. Add an MFA requirement to the service role.

---

### Question 13: Enforcing HTTPS and Secure TLS Ciphers on ALB

A banking regulation mandates that an enterprise web application must disable deprecated SSL/TLS protocols (SSL 3.0, TLS 1.0, TLS 1.1) and support only TLS 1.2 and TLS 1.3 with forward-secret cipher suites on its public endpoints.

**Where should this cryptographic policy be configured?**
- A. In the EC2 operating system `/etc/ssl/openssl.cnf` file.
- B. On the **Application Load Balancer HTTPS Listener**, select a modern **Security Policy** (e.g. `ELBSecurityPolicy-TLS13-1-2-2021-06`).
- C. In the Route 53 DNS record configuration.
- D. In the VPC Network ACL inbound rules.

---

### Question 14: Centralized Secrets Management with IAM and KMS Separation of Duties

A compliance regulation requires strict **Separation of Duties** between the cloud infrastructure engineers who deploy application resources and the security officers who manage cryptographic keys. Cloud engineers must be able to deploy application stacks that consume secrets from AWS Secrets Manager, but they must **never** be able to view or decrypt the underlying plaintext secrets in the console or CLI.

**How can this separation of duties be enforced?**
- A. Put all engineers in the same IAM group.
- B. Encrypt the secret in AWS Secrets Manager using a **Customer Managed Key (CMK)**. In the KMS Key Policy, grant `kms:Decrypt` permissions **only** to the specific application execution IAM roles (e.g. the ECS task execution role), and explicitly exclude the infrastructure engineers' IAM roles from the KMS key policy.
- C. Delete the KMS key after encrypting the secret.
- D. Store the secrets in plaintext in S3.

---

### Question 15: AWS Network Firewall Stateful Egress Filtering

A company needs to implement deep packet inspection (DPI) on all outbound internet traffic from a VPC. Outbound traffic must be restricted to an approved allowlist of fully qualified domain names (e.g. `*.github.com`, `*.amazonaws.com`). IP-based security groups are insufficient because the target services use dynamic, shared IP addresses.

**Which AWS network service enforces stateful domain-name filtering on egress traffic?**
- A. AWS Network Access Control Lists (NACLs)
- B. **AWS Network Firewall** configured with **Stateful Rule Groups** using Suricata-compatible domain list inspection rules.
- C. Amazon GuardDuty
- D. Amazon Route 53 Resolver query logging

---

## Answer Key & Detailed Explanations

### Question 1: IAM Permission Boundaries for Delegated Role Administration

**Answer:** B  
**Explanation:** 
**IAM Permission Boundaries** are advanced IAM policies that define the maximum permissions an identity-based policy can grant to an IAM entity. By requiring developers to attach an approved Permission Boundary whenever they call `iam:CreateRole` (enforced via an IAM condition `StringEquals: { "iam:PermissionsBoundary": ... }`), the newly created role's effective permissions can never exceed the boundary, preventing privilege escalation.

---

### Question 2: Secrets Manager Automated Database Secret Rotation Mechanics

**Answer:** B  
**Explanation:** 
- In **Single-User rotation**, Secrets Manager generates a new password, updates the database password for that single user, and updates Secrets Manager. If an application holds the old password in memory, it experiences authentication failures until it re-fetches the secret.
- In **Dual-User rotation** (recommended for production), Secrets Manager alternates between two users (e.g. `app_user_a` and `app_user_b`). When user A is active, user B is updated with a new password and verified. The secret is updated to point to user B. Applications pick up user B without any downtime.

---

### Question 3: Service Control Policy (SCP) Restricting AWS Regions

**Answer:** A  
**Explanation:** 
The standard AWS Region restriction SCP uses an explicit `Deny` on all actions *except* global services (`NotAction`: `iam:*`, `route53:*`, `cloudfront:*`, `support:*`, etc.) conditioned on `"StringNotEquals": { "aws:RequestedRegion": ["us-east-1", "us-west-2"] }`. This prevents any API calls that create or manage resources in unapproved regions while ensuring that global control-plane services continue operating seamlessly.

---

### Question 4: KMS Key Policy vs. IAM Policy for Access Control

**Answer:** B  
**Explanation:** 
In AWS KMS, the **Key Policy** is the primary access control mechanism. Unlike other AWS resources where an identity policy alone can grant access within the same account, KMS requires that the Key Policy contain the default statement enabling IAM user permissions:
`"Principal": {"AWS": "arn:aws:iam::account-id:root"}`.
If this statement is removed or altered, the KMS key is isolated, and even an IAM user with `AdministratorAccess` cannot use or administer the key.

---

### Question 5: AWS Inspector v2 Automated Vulnerability Management

**Answer:** B  
**Explanation:** 
**Amazon Inspector v2** is a vulnerability management service that continuously scans workloads for software vulnerabilities and unintended network exposure. Inspector v2 automatically discovers and continuously scans Amazon EC2 instances (via SSM Agent), Amazon ECR container images (both at push time and continuously when new CVEs are published), and AWS Lambda functions (including application dependencies in Python, Java, Node.js, and Go).

---

### Question 6: Amazon Macie for Sensitive Data Discovery in S3

**Answer:** B  
**Explanation:** 
**Amazon Macie** is a fully managed data security and privacy service that uses machine learning and pattern matching to discover, classify, and protect sensitive data in Amazon S3. Macie automatically scans S3 buckets for managed sensitive data identifiers (PII, financial data, credentials) and emits detailed findings to Amazon EventBridge and AWS Security Hub for automated triage.

---

### Question 7: AWS WAF Bot Control and Account Takeover Prevention

**Answer:** B  
**Explanation:** 
When attackers use distributed residential proxy networks, traditional IP rate limiting fails because each IP sends only a few requests. **AWS WAF Bot Control** and **Account Takeover Prevention (ATP)** managed rule groups inspect client behavioral signals, browser fingerprints, HTTP request headers, and known credential stuffing patterns to detect and block malicious bots and credential-stuffing attacks in real-time, regardless of IP rotation.

---

### Question 8: KMS Multi-Region Keys for Disaster Recovery

**Answer:** B  
**Explanation:** 
**AWS KMS Multi-Region Keys** allow creating a Primary KMS key in one region and replicating it to other regions. Multi-Region keys share the exact same Key ID and key material across regions. This enables ciphertext encrypted in `us-east-1` to be decrypted seamlessly by the replica key in `eu-west-1` without making cross-region network calls or re-encrypting data, providing optimal performance for active-active multi-region systems.

---

### Question 9: AWS Config Custom Rules with Lambda for Specific Compliance Logic

**Answer:** A  
**Explanation:** 
When AWS Config managed rules do not meet specific enterprise validation logic, developers can create **AWS Config Custom Rules** powered by AWS Lambda. AWS Config invokes the Lambda function whenever a target resource changes. The Lambda function evaluates custom logic (e.g. verifying that `CostCenter` matches a 4-digit regex) and reports the evaluation result back to AWS Config using `PutEvaluations`.

---

### Question 10: Cross-Account IAM Role Chaining Session Duration Limits

**Answer:** B  
**Explanation:** 
AWS IAM enforces a strict security limit on **Role Chaining**: when an identity assumes a role and then uses those temporary credentials to assume another role, the resulting session duration is capped at a maximum of **1 hour** (3,600 seconds). The configured `MaxSessionDuration` attribute (which can be up to 12 hours) applies only when the role is assumed by an IAM user or federated identity directly, not through role chaining.

---

### Question 11: GuardDuty Malware Protection for Amazon EBS

**Answer:** B  
**Explanation:** 
**Amazon GuardDuty Malware Protection** provides agentless malware detection for Amazon EBS volumes attached to EC2 instances. When GuardDuty detects suspicious activity, or upon on-demand invocation, it takes an EBS snapshot, creates a replica volume in an isolated AWS service environment, scans for malicious files, and generates findings without impacting the running instance's CPU or disk performance.

---

### Question 12: Restricting IAM Roles to Specific AWS Services (`aws:SourceArn`)

**Answer:** A  
**Explanation:** 
The **Confused Deputy** problem occurs when a service entity in one account is coerced into performing actions against another account's resources. To prevent this, AWS recommends using the `aws:SourceArn` and `aws:SourceAccount` global condition keys in service role trust policies. This restricts role assumption exclusively to requests originating from the specific project ARN and account ID.

---

### Question 13: Enforcing HTTPS and Secure TLS Ciphers on ALB

**Answer:** B  
**Explanation:** 
On an Application Load Balancer, SSL/TLS termination is controlled by the **Security Policy** assigned to the HTTPS listener. AWS provides predefined security policies (e.g. `ELBSecurityPolicy-TLS13-1-2-2021-06`) that restrict accepted protocols to TLS 1.2 and TLS 1.3 and specify allowed cryptographic ciphers, offloading compliance enforcement from backend compute instances.

---

### Question 14: Centralized Secrets Management with IAM and KMS Separation of Duties

**Answer:** B  
**Explanation:** 
AWS Secrets Manager uses envelope encryption with AWS KMS. By using a Customer Managed Key (CMK) whose key policy grants `kms:Decrypt` exclusively to the application service roles (and not to human engineers), human engineers can have full permissions on Secrets Manager (`secretsmanager:*`), but any attempt by an engineer to call `GetSecretValue` will fail at the KMS layer with `AccessDeniedException`, successfully enforcing separation of duties.

---

### Question 15: AWS Network Firewall Stateful Egress Filtering

**Answer:** B  
**Explanation:** 
**AWS Network Firewall** is a stateful, managed network firewall and intrusion detection/prevention system. It supports stateful domain list filtering using Server Name Indication (SNI) inspection on TLS handshakes and HTTP Host headers, allowing organizations to restrict outbound internet traffic to specific FQDNs and wildcard domain patterns (e.g. `*.amazonaws.com`) without maintaining static IP lists.

---

### Scoring Guide
- **14–15 Correct:** Mastery (Ready for DOP-C02 Security & Compliance section)
- **12–13 Correct:** Solid Proficiency (Review specific IAM role chaining limits and Secrets Manager dual-user rotation)
- **$\le 11$ Correct:** Needs Reinforcement (Review `Domain_6_Security_Compliance/Study_Guide.md` and `Exercises.md`)
