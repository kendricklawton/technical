# Domain 1 (SDLC Automation) — Quiz 6: Multi-Region Pipelines & Governance

**Topic:** Multi-Region Deployment Pipelines, Container Security Gates & Pipeline Governance  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: Cross-Region CodePipeline with AWS KMS CMK Configuration

An enterprise maintains an active-active web application deployed across `us-east-1` and `ap-southeast-1`. The deployment pipeline is hosted in `us-east-1`. The pipeline must build container images once in `us-east-1` and deploy identical artifacts to Amazon ECS clusters in both regions.

**How must the S3 artifact store and KMS keys be defined in the AWS CodePipeline template?**
- A. A single S3 bucket in `us-east-1` encrypted with default `aws/s3`.
- B. Define `artifactStores` in the pipeline definition specifying an S3 bucket in `us-east-1` encrypted with a `us-east-1` Customer Managed Key (CMK), and a second S3 bucket in `ap-southeast-1` encrypted with an `ap-southeast-1` Customer Managed Key (CMK).
- C. Use Amazon EFS instead of Amazon S3.
- D. Cross-region pipeline actions are not supported; two independent pipelines must be created.

---

### Question 2: Container Image Signing with AWS Signer in CI/CD

A financial compliance standard requires that container images deployed to Amazon EKS must be cryptographically signed by an approved build authority. Any container image whose signature is invalid or unsigned must be rejected by the Kubernetes admission controller at deploy time.

**Which AWS service integration implements automated container image signing in the CI/CD pipeline?**
- A. PGP signatures stored in GitHub.
- B. **AWS Signer** with container image signing profiles integrated into AWS CodeBuild. In CodeBuild, sign the container image using the AWS Signer signing profile and store the signature in the Amazon ECR OCI registry. Deploy Kyverno or AWS Signer Gatekeeper on EKS to enforce signature verification.
- C. Encrypt the container image with AWS KMS.
- D. Base64 encode the Dockerfile.

---

### Question 3: GitOps Pull-Based Delivery vs. Pipeline Push-Based Delivery

An engineering organization is migrating from a traditional push-based CI/CD pipeline to a GitOps delivery model using **Argo CD** on Amazon EKS.

**What is the primary security advantage of the GitOps model regarding Kubernetes cluster credentials?**
- A. GitOps makes Kubernetes clusters run faster.
- B. In GitOps, the deployment agent runs **inside** the Kubernetes cluster and pulls manifests from Git. External CI/CD tools (like GitHub Actions or Jenkins) do **not** require administrative cluster access or long-lived Kubernetes API credentials.
- C. GitOps eliminates the need for Git version control.
- D. GitOps automatically encrypts all application data at rest.

---

### Question 4: Pipeline Execution History & CloudTrail Auditing

An enterprise auditor requires an immutable audit trail showing every execution of the production deployment pipeline for the past 12 months, including who triggered the deployment, which commit was deployed, and whether approvals were granted.

**Where is this audit data captured and queried?**
- A. CodePipeline console history only (stored for 30 days).
- B. **AWS CloudTrail** logs capturing all CodePipeline API calls (`StartPipelineExecution`, `PutApprovalResult`), delivered to an encrypted S3 bucket in a dedicated log archive account with S3 Object Lock enabled.
- C. Local developer terminal logs.
- D. AWS Trusted Advisor.

---

### Question 5: CodePipeline Source Action Polling vs. Webhook Latency

A development team switches their CodePipeline source provider from AWS CodeCommit to GitHub Enterprise. The team notices that new commits pushed to GitHub take between 5 and 15 minutes before the pipeline detects the change and starts building.

**Why is there a delay, and how should it be resolved?**
- A. The pipeline is using periodic polling (`PollForSourceChanges: true`). Reconfigure the source action to use **AWS CodeStar Connections** (or GitHub Webhooks) to enable instantaneous event-driven pipeline execution.
- B. GitHub Enterprise restricts API calls to once every 15 minutes.
- C. Increase CodeBuild compute size.
- D. The developer must manually click "Release Change" in the console.

---

### Question 6: Automated Rollback on Canary Pipeline Stages with Step Functions

A complex enterprise deployment requires deploying an application to 10 AWS regions sequentially in waves (Wave 1: 1 region, Wave 2: 3 regions, Wave 3: 6 regions), waiting 2 hours between waves to monitor synthetic error rates. If error rates exceed threshold in any wave, all deployed regions must automatically roll back.

**Which AWS service is best suited to orchestrate this multi-wave, multi-region deployment workflow?**
- A. A single AWS CodePipeline pipeline with 50 parallel actions.
- B. **AWS Step Functions** state machine orchestrating regional deployments, embedding wait states, monitoring CloudWatch alarms, and executing rollback states if errors are detected.
- C. A bash script running on an EC2 instance.
- D. Elastic Beanstalk rolling deployments.

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

### Question 1: Cross-Region CodePipeline with AWS KMS CMK Configuration

**Answer:** B  
**Explanation:** 
When an AWS CodePipeline contains actions in a region different from the pipeline's host region, CodePipeline requires `artifactStores` to be configured as a dictionary of regions to S3 buckets. Each region where an action runs must have its own S3 bucket located physically in that region, encrypted with a Customer Managed Key (CMK) native to that region. CodePipeline handles cross-region artifact transfer and re-encryption transparently.

---

### Question 2: Container Image Signing with AWS Signer in CI/CD

**Answer:** B  
**Explanation:** 
**AWS Signer** is a fully managed code-signing service that supports signing container images stored in OCI-compliant registries like Amazon ECR. In AWS CodeBuild, images are signed using an AWS Signer signing profile. The resulting signature is stored in ECR alongside the image. An admission controller (e.g. AWS Signer Gatekeeper or Kyverno) intercepts pod creation on EKS and verifies the signature before allowing the pod to launch.

---

### Question 3: GitOps Pull-Based Delivery vs. Pipeline Push-Based Delivery

**Answer:** B  
**Explanation:** 
In traditional push-based CI/CD, external build servers must possess administrative credentials (e.g. `kubeconfig`, IAM roles) to communicate directly with the Kubernetes API server, creating a high-risk security target. In GitOps pull-based delivery, the operator runs inside the cluster and pulls configuration from Git, completely closing the cluster's inbound API attack surface to external CI tools.

---

### Question 4: Pipeline Execution History & CloudTrail Auditing

**Answer:** B  
**Explanation:** 
While the AWS CodePipeline console displays recent execution history, long-term regulatory compliance requires immutable audit logging via **AWS CloudTrail**. CloudTrail captures every API invocation, including the IAM principal who triggered the pipeline, stage transitions, and manual approval decisions (`PutApprovalResult`), archived securely in Amazon S3 with Object Lock.

---

### Question 5: CodePipeline Source Action Polling vs. Webhook Latency

**Answer:** A  
**Explanation:** 
When CodePipeline source actions are configured with polling (`PollForSourceChanges: true`), CodePipeline checks for changes on a periodic schedule (every 5–15 minutes). Switching to **AWS CodeStar Connections** (AWS CodeConnections) establishes an active webhook integration with GitHub, triggering pipeline executions within seconds of a `git push`.

---

### Question 6: Automated Rollback on Canary Pipeline Stages with Step Functions

**Answer:** B  
**Explanation:** 
While AWS CodePipeline excels at single-pipeline staging, complex multi-wave, multi-region orchestrations with long wait periods (e.g. 2 hours between regional waves), dynamic error evaluation, and multi-region parallel rollbacks are ideally managed by **AWS Step Functions**. Step Functions natively handles long-running workflows (up to 1 year), retries, and branching error-recovery logic.

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

