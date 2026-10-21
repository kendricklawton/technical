# Domain 2 (Configuration Management & IaC) — Quiz 2: CloudFormation Advanced Patterns

**Topic:** CloudFormation Custom Resources, StackSets, Drift Detection & Change Sets  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: CloudFormation Custom Resource Response Signal Timeout

An engineer creates an AWS CloudFormation Custom Resource backed by an AWS Lambda function (`Custom::DatabaseSeed`). During stack creation, the Lambda function executes successfully and inserts records into a database, but the CloudFormation stack remains in `CREATE_IN_PROGRESS` for exactly 1 hour before failing with the error:
`Custom Resource failed to stabilize in expected time.`

**What caused this failure, and how should it be prevented?**
- A. The Lambda function ran out of memory.
- B. The Lambda function code executed its business logic but **failed to send an HTTP PUT request with status `SUCCESS` to the pre-signed Amazon S3 ResponseURL** provided in the CloudFormation event payload. CloudFormation waits up to 60 minutes for this signal before timing out.
- C. CloudFormation custom resources require an attached NAT Gateway.
- D. The database was in a private subnet.

---

### Question 2: CloudFormation StackSets Self-Managed vs. Service-Managed Permissions

An enterprise uses AWS CloudFormation StackSets to deploy security tooling across 100 AWS accounts. The DevOps team needs to choose between **Self-Managed Permissions** and **Service-Managed Permissions**.

**Which statement accurately describes the advantages of Service-Managed Permissions with AWS Organizations?**
- A. Service-managed permissions require manual creation of `AWSCloudFormationStackSetAdministrationRole` in every member account.
- B. With **Service-Managed Permissions**, CloudFormation manages the necessary cross-account IAM roles automatically. StackSets can target AWS Organizations Organizational Units (OUs) directly, and **Automatic Deployment** can be enabled to automatically deploy stacks into new accounts created in the OU.
- C. Self-managed permissions support automatic deployment, whereas service-managed permissions do not.
- D. Service-managed permissions can only be used with root user accounts.

---

### Question 3: CloudFormation Change Sets and Replacement Warnings

A developer modifies a CloudFormation template to change the `Port` property of an Amazon RDS DB Instance from 5432 to 5433. The developer runs `aws cloudformation create-change-set`.

**How does CloudFormation Change Sets alert the engineer to the potential impact of this modification?**
- A. The change set will be rejected with an error.
- B. The change set lists the RDS DB Instance resource with `Action: Modify` and **`Replacement: True`**, warning the engineer that applying this change set will delete the existing database and create a brand-new instance, causing total data loss unless retained.
- C. The change set automatically takes an EBS snapshot.
- D. CloudFormation will automatically update the port in-place with zero downtime.

---

### Question 4: Resolving CloudFormation Drift

An AWS Config rule detects configuration drift on an Amazon S3 bucket managed by CloudFormation: a system administrator manually added a bucket tag `CostCenter=9999` directly in the S3 console. CloudFormation Drift Detection reports the resource as `MODIFIED`.

**How can the DevOps Engineer resolve this drift so the template and live resource match?**
- A. Delete the CloudFormation stack.
- B. Update the CloudFormation template in Git to include the `CostCenter: 9999` tag in the `AWS::S3::Bucket` resource properties and execute a CloudFormation stack update (or manually remove the tag from the S3 bucket to revert live drift).
- C. Run `cfn-hup` in the terminal.
- D. S3 drift cannot be remediated.

---

### Question 5: StackSet Concurrency and Failure Tolerance

An engineer deploys an updated IAM policy to 50 AWS accounts using CloudFormation StackSets. The update must proceed quickly, but if more than 3 accounts encounter errors, the deployment must immediately abort to prevent widespread configuration issues.

**Which parameters on `create-stack-instances` or `update-stack-set` should be set?**
- A. `MaxConcurrentCount: 10`, `FailureToleranceCount: 3`
- B. `MaxConcurrentCount: 50`, `FailureToleranceCount: 50`
- C. `MaxConcurrentCount: 1`, `FailureToleranceCount: 0`
- D. `RetainStacks: true`

---

### Question 6: CloudFormation Template Transforms & Includes

A company maintains standardized networking components (Security Groups, Subnet route associations) that are reused across 30 different CloudFormation templates. The company wants to store these standard snippets in an Amazon S3 bucket and insert them dynamically into other templates.

**Which CloudFormation macro or transform achieves this?**
- A. `Transform: AWS::Serverless-2016-10-31`
- B. `Transform: AWS::Include` with `Location: s3://my-templates/security-groups.yaml`
- C. Hardcoding the YAML in each file.
- D. AWS OpsWorks Chef recipes.

---

### Question 7: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap

A centralized CI/CD pipeline in Account 111111111111 deploys an ECS service in Account 222222222222. The deployment stage fails with: 'The ciphertext refers to a customer master key that does not exist, does not allow access, or is invalid.' The S3 bucket is encrypted with the default AWS-managed key 'aws/s3'. How should the engineer resolve this?

- A. Update the S3 bucket policy to grant read access to Account 222222222222.
- B. Reconfigure the S3 artifact bucket to use a Customer Managed Key (CMK) and grant `kms:Decrypt` and `kms:DescribeKey` permissions to Account 222222222222's execution role in the KMS Key Policy.
- C. Create a VPC Endpoint for S3 in Account 222222222222.
- D. Enable S3 Object Ownership with Bucket Owner Preferred.

---

### Question 8: CodeDeploy In-Place ApplicationStop Lifecycle Failure

During an in-place deployment to an EC2 Auto Scaling Group, the deployment fails at the ApplicationStop lifecycle event hook because a previous buggy script had been deployed in the older revision. How can the engineer force CodeDeploy to deploy the new fixed revision without terminating the instances?

- A. Delete the CodeDeploy agent from all EC2 instances.
- B. Trigger a new deployment via the AWS CLI with the `--ignore-application-stop-failures` flag enabled.
- C. Manually stop the EC2 instances in the AWS Management Console.
- D. Reboot the instances and rerun the same deployment ID.

---

### Question 9: CloudFormation Custom Resource 60-Minute Stack Freeze

A CloudFormation stack creation hangs for exactly 1 hour in `CREATE_IN_PROGRESS` on a Lambda-backed custom resource before failing and rolling back. What was the root cause?

- A. The Lambda function exceeded its maximum execution timeout of 15 minutes.
- B. An unhandled exception crashed the Lambda function before it could send an HTTP PUT response with status SUCCESS or FAILED to the pre-signed Amazon S3 ResponseURL.
- C. CloudFormation lost IAM permissions to invoke Lambda.
- D. The S3 bucket holding the CloudFormation template was deleted.

---

## Answer Key & Detailed Explanations

### Question 1: CloudFormation Custom Resource Response Signal Timeout

**Answer:** B  
**Explanation:** 
When CloudFormation invokes a custom resource Lambda function, it provides a pre-signed S3 URL in the event payload (`ResponseURL`). The Lambda function code must explicitly construct and send an HTTP `PUT` request containing a JSON payload with `"Status": "SUCCESS"` to this `ResponseURL`. If the Lambda function terminates without sending this response (e.g. missing `try/catch` or network failure reaching S3), CloudFormation will wait for up to 60 minutes before timing out and rolling back the stack.

---

### Question 2: CloudFormation StackSets Self-Managed vs. Service-Managed Permissions

**Answer:** B  
**Explanation:** 
When integrated with AWS Organizations, **Service-Managed StackSets** eliminate the need to pre-create manual administration and execution IAM roles in target accounts. StackSets can target specific OUs, and by enabling `AutoDeployment: { Enabled: true }`, any new account created in or moved into that OU automatically receives stack deployments, and stacks are automatically deleted when an account leaves the OU.

---

### Question 3: CloudFormation Change Sets and Replacement Warnings

**Answer:** B  
**Explanation:** 
In Amazon RDS, changing the `Port` property requires physical resource replacement. CloudFormation Change Sets explicitly flag this by setting `Replacement: True` on the modified resource. Reviewing change sets before execution is a critical operational safety gate that prevents accidental, catastrophic deletion of stateful data stores.

---

### Question 4: Resolving CloudFormation Drift

**Answer:** B  
**Explanation:** 
CloudFormation Drift Detection identifies divergence between the live infrastructure and the template. To resolve drift, operators can either:
1. Revert the live resource manually to match the CloudFormation template.
2. Update the CloudFormation template to reflect the intentional change and execute a stack update, bringing the template and live infrastructure into synchronization.

---

### Question 5: StackSet Concurrency and Failure Tolerance

**Answer:** A  
**Explanation:** 
- `MaxConcurrentCount: 10`: Executes the stack operation across up to 10 accounts simultaneously.
- `FailureToleranceCount: 3`: Allows up to 3 stack instance operations to fail before CloudFormation automatically halts the entire operation, preventing systemic failures across the organization.

---

### Question 6: CloudFormation Template Transforms & Includes

**Answer:** B  
**Explanation:** 
The AWS-provided transform **`AWS::Include`** allows inserting template snippets from Amazon S3 into CloudFormation templates. It works like an `#include` directive in C/C++, allowing reusable blocks of YAML/JSON (such as shared security groups or IAM roles) to be stored once in S3 and pulled dynamically into multiple templates during stack creation or updates.

---


### Question 7: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap
**Answer:** B
**Explanation:** Default AWS-managed keys like `aws/s3` cannot have their key policies modified and CANNOT be accessed across AWS accounts. Cross-account CodePipeline deployments strictly require an S3 bucket encrypted with a Customer Managed Key (CMK) whose key policy explicitly grants cross-account decrypt rights.

---

### Question 8: CodeDeploy In-Place ApplicationStop Lifecycle Failure
**Answer:** B
**Explanation:** CodeDeploy executes the `ApplicationStop` hook from the PREVIOUS revision's AppSpec file. If that script fails, all future deployments fail at that step unless you specify `--ignore-application-stop-failures`.

---

### Question 9: CloudFormation Custom Resource 60-Minute Stack Freeze
**Answer:** B
**Explanation:** CloudFormation Custom Resources expect a signed JSON payload uploaded to the pre-signed S3 `ResponseURL`. If the Lambda function crashes or exits without sending this signal, CloudFormation waits 1 hour before timing out.

---

