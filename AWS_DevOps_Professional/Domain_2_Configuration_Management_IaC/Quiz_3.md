# Domain 2 (Configuration Management & IaC) — Quiz 3: Parameter Store Deep Dive

**Topic:** AWS Systems Manager Parameter Store: Tiers, Hierarchies, Dynamic Resolution & RAM Sharing  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: Standard vs. Advanced Parameter Store Tiers

An enterprise needs to store a 6.5 KB configuration payload and enable automated expiration for temporary access tokens in AWS Systems Manager Parameter Store. When attempting to create the parameter in the Standard tier, the API returns `ValidationException`.

**Why does this error occur, and how is it resolved?**
- A. Parameter Store only supports strings up to 1 KB; Amazon DynamoDB must be used.
- B. The **Standard tier** of Parameter Store enforces a maximum parameter size limit of **4 KB** and does not support parameter policies (such as expiration). The parameter must be created using the **Advanced tier**, which supports values up to **8 KB**, parameter policies, and larger total parameter counts (up to 100,000 parameters).
- C. The parameter must be compressed with base64.
- D. AWS Secrets Manager is required for any value over 4 KB.

---

### Question 2: CloudFormation Dynamic SSM References

A CloudFormation template needs to retrieve a database password stored as an encrypted `SecureString` in AWS Systems Manager Parameter Store.

**Which CloudFormation dynamic reference syntax retrieves the decrypted password securely during stack execution?**
- A. `!Ref MyPasswordParameter`
- B. `{{resolve:ssm-secure:my-db-password:1}}`
- C. `!GetAtt SSM::Parameter.Value`
- D. `{{resolve:secretsmanager:my-db-password}}`

---

### Question 3: Cross-Account Sharing of SSM Parameters with AWS RAM

A central shared services account (`111111111111`) maintains standardized AMI IDs, corporate DNS endpoints, and proxy configurations in Systems Manager Parameter Store. These parameters must be accessible to 50 member accounts in the AWS Organization without duplicating parameter values across accounts.

**How should cross-account sharing of Parameter Store parameters be implemented?**
- A. Write a Lambda function in Account `111111111111` to replicate parameters into all member accounts daily.
- B. Upgrade the parameters to the **Advanced tier** in Account `111111111111`. Create a resource share in **AWS Resource Access Manager (AWS RAM)**, specify the parameter ARNs, and share them with the entire AWS Organization.
- C. Grant public read access to the parameters.
- D. Parameter Store does not support cross-account sharing; S3 must be used.

---

### Question 4: Parameter Store Hierarchical Path Traversal

A microservice needs to fetch all configuration parameters associated with the `billing` service in the `production` environment with a single API call. The parameters are organized under `/app/prod/billing/db_host`, `/app/prod/billing/db_port`, and `/app/prod/billing/api_key`.

**Which AWS Systems Manager API call and parameter achieves this?**
- A. `aws ssm get-parameters-by-path --path "/app/prod/billing/" --recursive --with-decryption`
- B. `aws ssm get-parameter --name "/app/prod/billing/*"`
- C. `aws ssm describe-parameters --filter "billing"`
- D. Running three separate `get-parameter` commands sequentially.

---

### Question 5: Parameter Store Change Notifications via EventBridge

A compliance requirement mandates that whenever an application configuration parameter under `/production/compliance/*` is updated or deleted in Systems Manager Parameter Store, a security alert must be triggered, and an automated verification job must execute.

**How should this change detection be architected?**
- A. Enable S3 access logging on Parameter Store.
- B. Create an **Amazon EventBridge Rule** with the event pattern:
  ```json
  {
    "source": ["aws.ssm"],
    "detail-type": ["Parameter Store Change"],
    "detail": {
      "name": [{"prefix": "/production/compliance/"}],
      "operation": ["Create", "Update", "Delete"]
    }
  }
  ```
  Target an Amazon SNS topic and an AWS Lambda function.
- C. Write a cron job in EC2 that polls Parameter Store every 30 seconds.
- D. CloudWatch Metrics only.

---

### Question 6: Parameter Store Versioning and Rollback

A junior engineer updates the database connection pool limit in Parameter Store from `50` to `500`, causing application connection starvation. The operations team needs to revert the parameter to its previous working value immediately.

**How does Parameter Store support version history and rollback?**
- A. Parameter Store does not store past versions; a backup must be restored from S3.
- B. Parameter Store **automatically increments the version number** on every update (e.g. Version 1 $\rightarrow$ Version 2). The team can inspect past versions using `GetParameterHistory` and update the parameter value back to Version 1's value (creating Version 3 with the restored content).
- C. Run `git revert` on Parameter Store.
- D. The database must be restarted.

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

### Question 1: Standard vs. Advanced Parameter Store Tiers

**Answer:** B  
**Explanation:** 
AWS Systems Manager Parameter Store has two tiers:
- **Standard:** Free, supports up to 10,000 parameters, maximum value size of 4 KB, no parameter policies.
- **Advanced:** Billed per parameter-month and per API interaction, supports up to 100,000 parameters, maximum value size of 8 KB, parameter policies (e.g. expiration or notification thresholds), and cross-account sharing via AWS RAM.

---

### Question 2: CloudFormation Dynamic SSM References

**Answer:** B  
**Explanation:** 
AWS CloudFormation supports dynamic references to retrieve parameters at stack execution time. For `SecureString` parameters in Systems Manager Parameter Store, the syntax is `{{resolve:ssm-secure:parameter-name:version}}`. Specifying `ssm-secure` tells CloudFormation to decrypt the parameter using the associated KMS key and inject it directly into the resource property without storing the plaintext in template files or CloudFormation event history.

---

### Question 3: Cross-Account Sharing of SSM Parameters with AWS RAM

**Answer:** B  
**Explanation:** 
AWS Systems Manager Parameter Store supports sharing **Advanced tier** parameters across AWS accounts using **AWS Resource Access Manager (AWS RAM)**. The owning account shares the parameters with specific accounts or the entire organization. Shared parameters can be read directly by member accounts using standard `GetParameter` APIs using the parameter's ARN, eliminating synchronization scripts and redundant storage costs.

---

### Question 4: Parameter Store Hierarchical Path Traversal

**Answer:** A  
**Explanation:** 
The AWS CLI command `aws ssm get-parameters-by-path` retrieves all parameters within a specified hierarchy. Specifying `--recursive` searches all child paths under `/app/prod/billing/`, and `--with-decryption` automatically unwraps any `SecureString` parameters using the caller's authorized KMS key in a single, efficient API transaction.

---

### Question 5: Parameter Store Change Notifications via EventBridge

**Answer:** B  
**Explanation:** 
AWS Systems Manager Parameter Store automatically emits events to Amazon EventBridge whenever parameters are created, updated, or deleted (`detail-type: Parameter Store Change`). EventBridge rules can match on parameter name prefixes (e.g. `/production/compliance/`) and operations, providing immediate event-driven notifications and automated compliance workflows.

---

### Question 6: Parameter Store Versioning and Rollback

**Answer:** B  
**Explanation:** 
Every modification to an SSM parameter creates a new immutable version number (1, 2, 3...). The API `GetParameterHistory` allows inspecting historical values, modification timestamps, and user identities. Reverting is accomplished by reading the historical version and saving it as the new latest version.

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

