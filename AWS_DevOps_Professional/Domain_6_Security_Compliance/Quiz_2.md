# Domain 6 (Security and Compliance) — Quiz 2: AWS Organizations & Service Control Policies (SCPs)

**Topic:** SCP Inheritance, Explicit Deny vs Allow, Management Account Guardrails & Region Restrictions  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: Service Control Policy (SCP) Evaluation and Inheritance

An AWS Organization has a root, an Organizational Unit named `ProductionOU`, and an AWS account `Account-Prod` inside `ProductionOU`.
- The Root has the default `FullAWSAccess` SCP attached.
- `ProductionOU` has an SCP attached that explicitly denies `s3:DeleteBucket`.
- `Account-Prod` has an SCP attached that explicitly allows `*` on `*`.

**Can an administrator with `AdministratorAccess` IAM permissions in `Account-Prod` delete an S3 bucket?**
- A. Yes, because `Account-Prod` has an SCP allowing all actions.
- B. No, because an **explicit Deny in any SCP in the inheritance hierarchy (from Root down to OU to Account) overrides all Allows**, and permissions must be allowed at every single level.
- C. Yes, because IAM permissions supersede SCP guardrails.
- D. Yes, if the administrator uses the AWS CLI with `--force`.

---

### Question 2: Limitations of Service Control Policies

Which of the following principals or actions is **EXEMPT** from restrictions imposed by Service Control Policies (SCPs)?
- A. The root user of a member AWS account.
- B. An IAM federated role with `AdministratorAccess` in a member account.
- C. The **management (root) account of the AWS Organization**, service-linked roles, and certain service-specific resource actions (like registering an S3 bucket with AWS Macie).
- D. Any user connecting via AWS CloudShell.

---

### Question 3: Enforcing Allowed AWS Regions via SCP Guardrails

A European banking institution must ensure that no workloads, data, or AWS resources can ever be deployed outside of `eu-west-1` (Ireland) and `eu-central-1` (Frankfurt) by any member account, while ensuring global services (IAM, Route 53, CloudFront) continue to function.

**Which SCP statement pattern achieves this compliance requirement?**
- A. 
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyNonEU",
      "Effect": "Deny",
      "NotAction": [
        "iam:*",
        "route53:*",
        "cloudfront:*",
        "wafv2:*",
        "support:*",
        "organizations:*"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": ["eu-west-1", "eu-central-1"]
        }
      }
    }
  ]
}
```
- B. Delete all VPCs in other regions using a shell script.
- C. Block all outbound internet traffic on port 443.
- D. Remove the `FullAWSAccess` SCP without attaching any replacement policy.

---

### Question 4: Disabling the Default FullAWSAccess SCP Safely

When implementing a strict "Allow-list" SCP governance strategy across an organization, what happens if an administrator detaches the default `FullAWSAccess` SCP from the Root before attaching an alternative allow policy?
- A. All accounts immediately lose all permissions and all AWS operations in all member accounts are completely blocked.
- B. The AWS Organization is automatically deleted.
- C. Nothing happens because AWS ignores empty SCP lists.
- D. Only the management account is affected.

---

### Question 5: Preventing Member Accounts from Disabling Security Services

An organization deploys Amazon GuardDuty, AWS Config, and AWS Security Hub in all accounts. A rouge developer in a member account with local `AdministratorAccess` attempts to turn off AWS Config recording to conceal unauthorized infrastructure.

**How can the enterprise mathematically prevent any member account administrator from disabling AWS Config?**
- A. Ask developers to promise not to run `aws configservice stop-configuration-recorder`.
- B. Attach an **SCP at the Organization Root** that explicitly denies `config:Delete*`, `config:Stop*`, and `config:PutConfigurationRecorder` with `"Resource": "*"`.
- C. Store the AWS Config recorder inside a private subnet.
- D. Use a CloudWatch Alarm that triggers after 24 hours.

---

### Question 6: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap

A centralized CI/CD pipeline in Account 111111111111 deploys an ECS service in Account 222222222222. The deployment stage fails with: 'The ciphertext refers to a customer master key that does not exist, does not allow access, or is invalid.' The S3 bucket is encrypted with the default AWS-managed key 'aws/s3'. How should the engineer resolve this?

- A. Update the S3 bucket policy to grant read access to Account 222222222222.
- B. Reconfigure the S3 artifact bucket to use a Customer Managed Key (CMK) and grant `kms:Decrypt` and `kms:DescribeKey` permissions to Account 222222222222's execution role in the KMS Key Policy.
- C. Create a VPC Endpoint for S3 in Account 222222222222.
- D. Enable S3 Object Ownership with Bucket Owner Preferred.

---

### Question 7: CodeDeploy In-Place ApplicationStop Lifecycle Failure

During an in-place deployment to an EC2 Auto Scaling Group, the deployment fails at the ApplicationStop lifecycle event hook because a previous buggy script had been deployed in the older revision. How can the engineer force CodeDeploy to deploy the new fixed revision without terminating the instances?

- A. Delete the CodeDeploy agent from all EC2 instances.
- B. Trigger a new deployment via the AWS CLI with the `--ignore-application-stop-failures` flag enabled.
- C. Manually stop the EC2 instances in the AWS Management Console.
- D. Reboot the instances and rerun the same deployment ID.

---

### Question 8: CloudFormation Custom Resource 60-Minute Stack Freeze

A CloudFormation stack creation hangs for exactly 1 hour in `CREATE_IN_PROGRESS` on a Lambda-backed custom resource before failing and rolling back. What was the root cause?

- A. The Lambda function exceeded its maximum execution timeout of 15 minutes.
- B. An unhandled exception crashed the Lambda function before it could send an HTTP PUT response with status SUCCESS or FAILED to the pre-signed Amazon S3 ResponseURL.
- C. CloudFormation lost IAM permissions to invoke Lambda.
- D. The S3 bucket holding the CloudFormation template was deleted.

---

## Answer Key & Detailed Explanations

### Question 1: Service Control Policy (SCP) Evaluation and Inheritance

**Answer:** B  
**Explanation:** 
Service Control Policies (SCPs) specify the maximum permissions for an account. For an action to be permitted:
1. It must NOT be explicitly denied by any SCP at any level in the hierarchy (Root -> OUs -> Account).
2. It must be explicitly allowed by an SCP at every single level from Root down to the Account.
Because `ProductionOU` has an explicit `Deny` on `s3:DeleteBucket`, that action is permanently blocked for all IAM principals (including account root) in `Account-Prod`, overriding any local Allow policies.

---

### Question 2: Limitations of Service Control Policies

**Answer:** C  
**Explanation:** 
SCPs do NOT apply to:
1. The **Management Account** of the organization (neither the management account's root user nor its IAM principals are restricted by SCPs).
2. **Service-Linked Roles** (SLRs cannot be blocked by SCPs).
3. Actions taken on emergency security resources (e.g., CloudFront key pairs).
Member account root users and member account administrators ARE strictly subject to SCPs.

---

### Question 3: Enforcing Allowed AWS Regions via SCP Guardrails

**Answer:** A  
**Explanation:** 
An explicit `Deny` with a `StringNotEquals` condition on `aws:RequestedRegion` blocks API calls targeting unauthorized regions. Crucially, the policy must use `NotAction` to exempt AWS global services (such as IAM, Route 53, CloudFront, Support, AWS Organizations) which do not operate in a specific regional endpoint (`aws:RequestedRegion` resolves to `us-east-1` or global for these services). Without exempting these services, basic IAM authentication and billing break entirely.

---

### Question 4: Disabling the Default FullAWSAccess SCP Safely

**Answer:** A  
**Explanation:** 
In AWS Organizations, access is governed by the intersection of allows. If `FullAWSAccess` is detached from an OU or Root and no other SCP granting allows exists at that level, the effective permission becomes an implicit deny for everything. To transition to a custom Allow-list strategy, you must attach the new custom SCP **before** removing `FullAWSAccess`.

---

### Question 5: Preventing Member Accounts from Disabling Security Services

**Answer:** B  
**Explanation:** 
Because Service Control Policies (SCPs) apply as an authoritative permission boundary over member accounts, an explicit `Deny` on `config:StopConfigurationRecorder` and related deletion APIs in an SCP makes it impossible for any IAM user, role, or even the member account root user to disable AWS Config, guaranteeing continuous compliance.

---


### Question 6: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap
**Answer:** B
**Explanation:** Default AWS-managed keys like `aws/s3` cannot have their key policies modified and CANNOT be accessed across AWS accounts. Cross-account CodePipeline deployments strictly require an S3 bucket encrypted with a Customer Managed Key (CMK) whose key policy explicitly grants cross-account decrypt rights.

---

### Question 7: CodeDeploy In-Place ApplicationStop Lifecycle Failure
**Answer:** B
**Explanation:** CodeDeploy executes the `ApplicationStop` hook from the PREVIOUS revision's AppSpec file. If that script fails, all future deployments fail at that step unless you specify `--ignore-application-stop-failures`.

---

### Question 8: CloudFormation Custom Resource 60-Minute Stack Freeze
**Answer:** B
**Explanation:** CloudFormation Custom Resources expect a signed JSON payload uploaded to the pre-signed S3 `ResponseURL`. If the Lambda function crashes or exits without sending this signal, CloudFormation waits 1 hour before timing out.

---

