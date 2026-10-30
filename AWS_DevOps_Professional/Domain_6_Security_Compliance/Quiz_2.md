# Domain 6 (Security and Compliance) — Quiz 2: AWS Organizations & Service Control Policies (SCPs)

**Topic:** SCP Inheritance, Explicit Deny vs Allow, Management Account Guardrails & Region Restrictions  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: Region Restriction SCP with Mandatory Global Service Exemptions

A security architect is authoring an enterprise Service Control Policy (SCP) to enforce data sovereignty. Workloads across all member accounts must be restricted strictly to operating within `us-east-1` and `us-west-2`. The policy must deny all API requests targeting unauthorized regions, but must NOT break global AWS services or core infrastructure functionality.

**Which condition structure and action specification correctly enforces this regional boundary?**
- A. Apply a `Deny` statement on `Action: "*"` with condition `StringNotEquals: { "aws:RequestedRegion": ["us-east-1", "us-west-2"] }`, combined with condition `StringNotEquals: { "aws:PrincipalArn": "arn:aws:iam::*:role/aws-service-role/*" }`, and exempting global service prefixes (`iam:*`, `route53:*`, `cloudfront:*`, `wafv2:*`, `support:*`) using `NotAction`.
- B. Apply a `Deny` statement on `Action: "*"` with condition `StringNotEquals: { "aws:RequestedRegion": ["us-east-1", "us-west-2"] }` attached directly to the Organizations root without exceptions.
- C. Apply an `Allow` statement listing only the approved regions in `Condition: { "aws:RequestedRegion": ["us-east-1", "us-west-2"] }` and detach `FullAWSAccess` from all accounts.
- D. Attach an IAM permissions boundary to all IAM roles denying API calls where the client IP address originates outside North America.

---

### Question 2: Exempting Incident Response Break-Glass Roles from Restrictive SCPs

An enterprise enforces an organization-wide SCP that explicitly denies all mutating actions on networking infrastructure (`ec2:DeleteVpc`, `ec2:DetachInternetGateway`, `ec2:DeleteRouteTable`). During severe operational outages, an emergency Site Reliability Engineering Break-Glass role (`arn:aws:iam::*:role/EmergencyIncidentCommander`) must be able to modify networking components to restore connectivity.

**How should the SCP be architected to exempt the break-glass role while denying all other principals?**
- A. In the SCP's `Deny` statement, add a condition: `"ArnNotLike": { "aws:PrincipalArn": ["arn:aws:iam::*:role/EmergencyIncidentCommander"] }`, ensuring that the explicit deny evaluates for every identity except the emergency responder role.
- B. In the SCP, use the `"NotPrincipal": { "AWS": "arn:aws:iam::*:role/EmergencyIncidentCommander" }` block to exclude the role.
- C. Grant the `EmergencyIncidentCommander` role the `AdministratorAccess` policy; administrator privileges automatically override SCP deny rules.
- D. Move the account experiencing the incident into an unmanaged OU where all SCPs are permanently detached.

---

### Question 3: SCP Inheritance, Effective Permissions & Detaching `FullAWSAccess`

An AWS Organization root contains an Organizational Unit (OU) named `CoreServices`, which contains a child OU named `DatabaseTier`. An engineer creates a custom SCP named `AllowOnlyDynamoDB` with an `Allow` statement granting `dynamodb:*` and attaches it to `DatabaseTier`. However, administrators in the `DatabaseTier` member accounts report they can still create Amazon S3 buckets and EC2 instances.

**Why are the administrators still able to create non-DynamoDB resources, and how are effective permissions evaluated in SCP inheritance?**
- A. The default AWS-managed policy **`FullAWSAccess`** is still attached at the Root and parent OU levels; an action is permitted only if allowed by an SCP at **every level** of the hierarchy (an intersection of allows); to restrict the OU to DynamoDB, `FullAWSAccess` must be removed from `DatabaseTier` or an explicit `Deny` must be applied.
- B. SCPs do not apply to child OUs; policies must be attached directly to individual member accounts to take effect.
- C. The administrators are using root user credentials, which automatically bypass all SCP restrictions.
- D. SCP `Allow` statements take precedence over `Deny` statements in child OUs.

---

### Question 4: The Management Account Security Exception Boundary

A security auditor logs into the Organizations Management Account (`111122223333`) using an IAM administrator role and deletes an Amazon S3 compliance bucket protected by an organization-wide SCP that explicitly denies `s3:DeleteBucket`. The auditor is shocked that the deletion succeeded despite the SCP being attached to the Organization Root.

**Why did the SCP fail to prevent the bucket deletion in this scenario?**
- A. SCPs NEVER apply to the Organizations Management Account: Service Control Policies govern member accounts only; IAM principals (including root and administrators) in the management account are completely exempt from all SCP evaluations.
- B. The S3 bucket did not have MFA Delete enabled in the management console.
- C. The SCP was written using invalid JSON syntax that caused the policy evaluation engine to fail open.
- D. S3 bucket policies always override Service Control Policies.

---

### Question 5: Enforcing Resource Tagging Standards at Creation Time via SCPs

Corporate governance mandates that no Amazon EC2 instance or EBS volume can be created in any member account unless it is tagged with an `Environment` tag containing either `Production`, `Staging`, or `Development`. Resources created without this tag must be blocked immediately at the API level.

**Which SCP statement enforces this pre-provisioning tagging standard?**
- A. Apply a `Deny` on `Action: ["ec2:RunInstances"]` targeting resources `["arn:aws:ec2:*:*:instance/*", "arn:aws:ec2:*:*:volume/*"]` with conditions: `Null: { "aws:RequestTag/Environment": "true" }` and `StringNotEquals: { "aws:RequestTag/Environment": ["Production", "Staging", "Development"] }`.
- B. Deploy an AWS Config rule `required-tags` with an automatic remediation Lambda function that tags instances after launch.
- C. Configure an AWS Systems Manager State Manager association that scans EC2 instances every 10 minutes and applies tags.
- D. Attach an IAM permissions boundary denying `ec2:DescribeInstances` if tags are missing.

---

### Question 6: Protecting Security and Governance Tooling Against Rogue Member Administrators

A rogue administrator possessing `AdministratorAccess` inside a member account attempts to evade security monitoring by disabling AWS CloudTrail logging and deleting the AWS Config recorder.

**Which SCP statement prevents all IAM principals in member accounts from tampering with security baselines?**
- A.
```json
{
  "Effect": "Deny",
  "Action": [
    "cloudtrail:StopLogging",
    "cloudtrail:DeleteTrail",
    "cloudtrail:UpdateTrail",
    "config:DeleteDeliveryChannel",
    "config:StopConfigurationRecorder"
  ],
  "Resource": "*"
}
```
- B.
```json
{
  "Effect": "Deny",
  "Action": ["cloudtrail:*", "config:*"],
  "Resource": "*",
  "Condition": { "StringEquals": { "iam:Role": "Admin" } }
}
```
- C.
```json
{
  "Effect": "Allow",
  "NotAction": ["cloudtrail:StopLogging", "config:DeleteDeliveryChannel"],
  "Resource": "*"
}
```
- D. Detach the IAM policy `AdministratorAccess` from all member accounts and replace it with `ReadOnlyAccess`.

---

### Question 7: Preventing Tampering with AWS Backup Recovery Vaults

An enterprise backup policy mandates that no user in any member account—including account administrators—can delete AWS Backup recovery points, shorten backup retention periods, or delete backup vaults.

**Which SCP statement enforces this immutable data protection standard?**
- A. Apply an explicit `Deny` on `Action: ["backup:DeleteBackupVault", "backup:DeleteBackupVaultAccessPolicy", "backup:DeleteRecoveryPoint", "backup:PutBackupVaultAccessPolicy", "backup:UpdateRecoveryPointLifecycle"]` on `Resource: "*"`.
- B. Enable S3 Object Lock in Governance Mode on the primary application EC2 instances.
- C. Remove `kms:Decrypt` permissions from the AWS Backup service-linked role.
- D. Configure an Amazon EventBridge rule that alerts on `DeleteBackupVault` API calls in CloudTrail.

---

### Question 8: Delegated Administrator Architecture for Centralized Security Services

Corporate security requires that the central Security Team manages AWS GuardDuty, AWS Security Hub, and AWS IAM Access Analyzer organization-wide. However, the CISO mandates that the Security Team must NOT be granted access to the Organizations Management Account to minimize administrative blast radius.

**Which AWS Organizations governance pattern fulfills this requirement?**
- A. Designate the dedicated Security Account as a **Delegated Administrator** for AWS GuardDuty, Security Hub, and IAM Access Analyzer via the Organizations Management Account; the Security Team can then manage member account security settings organization-wide directly from the delegated security account.
- B. Create IAM users for the security team inside the Management Account and attach an IAM policy restricting them to security services.
- C. Write an AWS Step Functions state machine in the management account that executes API calls triggered via webhooks from the security account.
- D. Deploy AWS RAM to share the management account's root credentials with the security account.

---

### Question 9: Preventing Disruption to Service-Linked Roles in Restrictive SCPs

An enterprise implements an aggressive zero-trust SCP that explicitly denies all IAM actions (`iam:*`) across member accounts to prevent developer privilege escalation. Shortly after applying the policy, Auto Scaling Groups begin failing to scale out, and Elastic Beanstalk environments crash because AWS services cannot create their required **Service-Linked Roles (SLRs)**.

**How should the SCP be adjusted to allow AWS services to provision service-linked roles while continuing to block member users?**
- A. In the SCP's `Deny` statement for IAM actions, add a `Condition` or `NotAction` exempting `iam:CreateServiceLinkedRole`, `iam:DeleteServiceLinkedRole`, `iam:GetServiceLinkedRoleDeletionStatus`, and restrict the grant to service principals using `StringEquals: { "iam:AWSServiceName": ["autoscaling.amazonaws.com", "elasticbeanstalk.amazonaws.com"] }`.
- B. Grant all developer IAM users the `iam:PassRole` permission globally across all resources.
- C. Temporarily move the affected member accounts into the Organizations root whenever Auto Scaling events trigger.
- D. Change the SCP effect from `Deny` to `Allow` and delete all IAM groups in the member accounts.

---

## Answer Key & Explanations

### Question 1: Region Restriction SCP with Mandatory Global Service Exemptions
**Answer:** A  
**Explanation:** 
When enforcing region restrictions via Service Control Policies:
- An explicit `Deny` targeting `Action: "*"` with condition `StringNotEquals: { "aws:RequestedRegion": ["us-east-1", "us-west-2"] }` blocks unauthorized regional endpoints.
- **Global Services Trap:** Global AWS services (IAM, Route 53, CloudFront, WAFv2, AWS Organizations, AWS Support) operate out of global control planes (typically hosted in `us-east-1` or with no explicit region token `aws-global`). If an SCP denies actions outside target regions without exempting global services, users cannot modify IAM roles, update Route 53 DNS records, or open AWS Support tickets.
- Using `NotAction` to exempt global service prefixes (`iam:*`, `route53:*`, `cloudfront:*`, `wafv2:*`, `support:*`) or exempting service-linked roles (`arn:aws:iam::*:role/aws-service-role/*`) prevents operational self-denial.

---

### Question 2: Exempting Incident Response Break-Glass Roles from Restrictive SCPs
**Answer:** A  
**Explanation:** 
In Service Control Policies:
- `NotPrincipal` is **not supported** in SCPs (it is valid only in resource-based policies like S3 bucket policies). Attempting to use `NotPrincipal` in an SCP generates a policy validation syntax error.
- The standard AWS architectural pattern to exempt specific roles (like an emergency break-glass SRE role) from a broad `Deny` SCP is to add a condition:
  `"ArnNotLike": { "aws:PrincipalArn": ["arn:aws:iam::*:role/EmergencyIncidentCommander"] }`
- Because the condition evaluates to false when the emergency role assumes the identity, the `Deny` statement is not enforced against that principal, allowing them to remediate critical infrastructure outages while maintaining the restriction for all other identities.

---

### Question 3: SCP Inheritance, Effective Permissions & Detaching `FullAWSAccess`
**Answer:** A  
**Explanation:** 
Service Control Policies operate as guardrails (filters) rather than permission grants:
- For an action to be permitted, it must be explicitly allowed by an SCP at **every level** of the hierarchy (Root $\rightarrow$ Parent OU $\rightarrow$ Child OU $\rightarrow$ Account). This is the intersection of allows.
- If the AWS-managed policy **`FullAWSAccess`** remains attached at all levels, every action is allowed by default at every level.
- Attaching an `AllowOnlyDynamoDB` policy to a child OU without removing `FullAWSAccess` has zero restrictive effect because `FullAWSAccess` is still present on that OU, granting full access.
- To restrict the OU to DynamoDB, you must either detach `FullAWSAccess` from that OU (leaving only `AllowOnlyDynamoDB`) or attach an explicit `Deny` policy (since explicit deny overrides any allow).

---

### Question 4: The Management Account Security Exception Boundary
**Answer:** A  
**Explanation:** 
A critical architectural boundary of AWS Organizations:
- **SCPs DO NOT apply to the Organizations Management Account.**
- No matter where an SCP is attached (even the Organizations Root), it has zero effect on IAM users, roles, or the root user within the management account.
- Because of this, AWS strongly recommends against running workloads in the management account. The management account should be reserved exclusively for organization billing, OU hierarchy governance, and delegated administration. Workloads must reside in member accounts where SCPs are fully enforced.

---

### Question 5: Enforcing Resource Tagging Standards at Creation Time via SCPs
**Answer:** A  
**Explanation:** 
To prevent resources from being created without mandatory tags:
- Use an explicit `Deny` on creation actions (e.g., `ec2:RunInstances`) scoped to the specific resource types (`instance/*`, `volume/*`).
- Condition 1: `Null: { "aws:RequestTag/Environment": "true" }` denies creation if the caller did not pass the tag at all.
- Condition 2: `StringNotEquals: { "aws:RequestTag/Environment": ["Production", "Staging", "Development"] }` denies creation if the caller passed an unauthorized tag value.
This blocks non-compliant resources synchronously at API invocation time before any resource is provisioned.

---

### Question 6: Protecting Security and Governance Tooling Against Rogue Member Administrators
**Answer:** A  
**Explanation:** 
Even users with `AdministratorAccess` inside member accounts are subject to SCPs:
- An SCP that explicitly denies destructive actions against governance tools (`cloudtrail:StopLogging`, `cloudtrail:DeleteTrail`, `cloudtrail:UpdateTrail`, `config:DeleteDeliveryChannel`, `config:StopConfigurationRecorder`) protects the audit trail.
- Because an explicit Deny in an SCP overrides any IAM policy attached to member roles, member account administrators cannot disable monitoring, cover their tracks, or blind security teams.

---

### Question 7: Preventing Tampering with AWS Backup Recovery Vaults
**Answer:** A  
**Explanation:** 
Ransomware attackers who compromise member account administrative credentials often attempt to delete backups and snapshots before detonating ransomware payloads.
- Applying an organization-wide SCP with an explicit `Deny` on `backup:DeleteBackupVault`, `backup:DeleteRecoveryPoint`, and `backup:UpdateRecoveryPointLifecycle` prevents any member account principal from deleting backups or shortening retention schedules.
- Paired with AWS Backup Vault Lock in Compliance Mode, this guarantees absolute immutability of enterprise recovery points.

---

### Question 8: Delegated Administrator Architecture for Centralized Security Services
**Answer:** A  
**Explanation:** 
**Delegated Administrator** in AWS Organizations enables least-privilege operations:
- Many AWS security services (Amazon GuardDuty, AWS Security Hub, Amazon Macie, IAM Access Analyzer, AWS Firewall Manager) support designating a member account as the Delegated Administrator.
- This allows security personnel to configure organization-wide security detectors, view finding dashboards, and manage compliance across all member accounts directly from the dedicated Security Account.
- Security engineers do not require access to the sensitive Organizations Management Account, isolating management plane risks.

---

### Question 9: Preventing Disruption to Service-Linked Roles in Restrictive SCPs
**Answer:** A  
**Explanation:** 
Many AWS managed services (such as Auto Scaling, Elastic Beanstalk, and Elastic Load Balancing) require creating **Service-Linked Roles (SLRs)** in member accounts to orchestrate resources on behalf of the customer.
- If an SCP enforces a blanket `Deny` on `iam:*`, AWS services attempting to invoke `iam:CreateServiceLinkedRole` fail, causing deployments and scaling events to halt.
- Best practice requires adding a condition or `NotAction` exempting `iam:CreateServiceLinkedRole` and `iam:DeleteServiceLinkedRole`, optionally restricting the exemption via the `iam:AWSServiceName` condition to approved AWS service principals.
