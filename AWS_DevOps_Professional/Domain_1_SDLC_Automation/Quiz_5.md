# Domain 1 (SDLC Automation) — Quiz 5: Version Control & CodeCommit

**Topic:** AWS CodeCommit Governance, Approval Rule Templates, Monorepos & Event Triggering  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: CodeCommit Approval Rule Templates for Compliance

A financial firm requires that any pull request targeting the `main` branch across 25 AWS CodeCommit repositories must receive at least 2 approvals from members of the `SecurityReviewers` IAM group before merging is permitted.

**How should this governance be configured with the LEAST administrative effort?**
- A. Manually configure approval rules on all 25 repositories individually in the console.
- B. Create an **Approval Rule Template** in AWS CodeCommit with `ApprovalsNeeded: 2` and `ApprovalPoolMembers` targeting the `SecurityReviewers` IAM group ARN. Associate the template with all 25 repositories.
- C. Use a pre-receive git hook on developer workstations.
- D. Disable pull requests and require direct SSH pushes.

---

### Question 2: Triggering CodePipeline from CodeCommit via EventBridge vs. Polling

In older AWS CodePipeline setups, pipelines periodically polled CodeCommit every few minutes to check for changes. What is the recommended modern mechanism for triggering pipelines on CodeCommit pushes, and why is it superior?

**Which mechanism and rationale is correct?**
- A. Polling is still recommended because it consumes zero API quotas.
- B. **Amazon EventBridge Rule** matching `source: aws.codecommit` and `detail-type: CodeCommit Repository State Change`. It provides sub-second event-driven triggering, eliminates polling API overhead, and supports fine-grained branch filtering.
- C. S3 bucket notifications.
- D. CodeCommit email triggers.

---

### Question 3: CodeCommit Branch Protection via IAM Policies

An enterprise wants to prohibit all developers from pushing code directly to the `main` branch in AWS CodeCommit. All changes to `main` must arrive exclusively through pull request merges.

**Which IAM policy statement enforces this branch protection rule?**
- A.
  ```json
  {
    "Effect": "Deny",
    "Action": [
      "codecommit:GitPush",
      "codecommit:DeleteBranch",
      "codecommit:PutFile",
      "codecommit:MergeBranchesByFastForward"
    ],
    "Resource": "arn:aws:codecommit:us-east-1:123456789012:MyRepo",
    "Condition": {
      "StringEqualsIfExists": {
        "codecommit:References": [
          "refs/heads/main"
        ]
      }
    }
  }
  ```
- B. Disable git in the developers' IAM policies.
- C. Delete the `main` branch.
- D. Use S3 Object Lock on CodeCommit.

---

### Question 4: Large Binary File Storage in CodeCommit

A game development studio uses AWS CodeCommit for source control. Developers frequently commit large 3D graphic models, textures, and video assets (500 MB to 2 GB per file). Developers report that `git clone` operations are taking over 45 minutes and failing intermittently.

**What is the recommended best practice for managing large binary files in AWS CodeCommit?**
- A. Compress all assets using zip files before pushing.
- B. Configure **Git Large File Storage (Git LFS)** in the local repository and track large binary extensions (`git lfs track "*.psd" "*.fbx"`). CodeCommit supports Git LFS natively and stores large file pointers in Git while storing large object data in specialized backend storage.
- C. Store assets in an open public S3 bucket and hardcode HTTP URLs in the source code.
- D. Split the files into 100 KB chunks manually.

---

### Question 5: CodeCommit Cross-Account Access via Role Chaining

A contractor working in AWS Account `222222222222` needs read and write access to an AWS CodeCommit repository located in Account `111111111111` without creating a dedicated IAM user in Account `111111111111`.

**How should this cross-account access be configured using Git credential helpers?**
- A. Share the root account password of Account `111111111111` with the contractor.
- B. In Account `111111111111`, create an IAM role with CodeCommit permissions and a trust policy permitting Account `222222222222`. The contractor configures their local `~/.gitconfig` to use the AWS CLI credential helper with `aws sts assume-role` profile configuration.
- C. Export the repository to GitHub.
- D. Create an S3 bucket in Account `222222222222` and sync git packs.

---

### Question 6: Pull Request Notifications via Amazon SNS and EventBridge

An engineering team wants developers to receive instant notifications in their team Slack channel whenever a comment is posted on a CodeCommit pull request or when an approval rule is overridden.

**Which event pattern in Amazon EventBridge captures these specific pull request activities?**
- A.
  ```json
  {
    "source": ["aws.codecommit"],
    "detail-type": [
      "CodeCommit Comment on Pull Request",
      "CodeCommit Pull Request State Change"
    ]
  }
  ```
- B.
  ```json
  {
    "source": ["aws.s3"],
    "detail-type": ["Object Created"]
  }
  ```
- C. CloudWatch Metric Alarm on GitPush.
- D. EC2 system logs.

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

### Question 1: CodeCommit Approval Rule Templates for Compliance

**Answer:** B  
**Explanation:** 
AWS CodeCommit **Approval Rule Templates** allow defining approval requirements once (e.g. 2 approvals from designated IAM users/roles) and associating the template across multiple repositories. Any new pull request created in any associated repository automatically enforces the approval requirements, eliminating repetitive manual configuration.

---

### Question 2: Triggering CodePipeline from CodeCommit via EventBridge vs. Polling

**Answer:** B  
**Explanation:** 
AWS recommends using **Amazon EventBridge** to detect CodeCommit repository state changes. When a commit is pushed, CodeCommit immediately emits an event to EventBridge, which triggers the target CodePipeline in near real-time (sub-second latency), completely eliminating polling delays and unnecessary CloudTrail/API costs.

---

### Question 3: CodeCommit Branch Protection via IAM Policies

**Answer:** A  
**Explanation:** 
AWS CodeCommit branch protection is enforced through IAM policies using the condition key `codecommit:References`. An explicit `Deny` on `codecommit:GitPush`, `codecommit:PutFile`, and `codecommit:DeleteBranch` conditioned on `"codecommit:References": ["refs/heads/main"]` prevents direct commits and pushes to `main`, while still allowing developers to push to feature branches and merge through the pull request console/API.

---

### Question 4: Large Binary File Storage in CodeCommit

**Answer:** B  
**Explanation:** 
AWS CodeCommit natively supports **Git Large File Storage (Git LFS)**. Git LFS replaces large binary files in the Git repository with lightweight text pointers, storing the actual binary payloads in a dedicated, high-speed object store. This drastically reduces repository clone and fetch times while preserving full version control history for multimedia and binary assets.

---

### Question 5: CodeCommit Cross-Account Access via Role Chaining

**Answer:** B  
**Explanation:** 
Cross-account CodeCommit access is cleanly achieved by having the external identity assume an IAM role in the repository-owning account. The developer configures their local `.gitconfig` with the `aws codecommit credential-helper`, specifying an AWS CLI named profile that automatically handles `sts:AssumeRole` token generation during `git fetch` and `git push` operations.

---

### Question 6: Pull Request Notifications via Amazon SNS and EventBridge

**Answer:** A  
**Explanation:** 
Amazon EventBridge receives native events from AWS CodeCommit with `detail-type` including `CodeCommit Comment on Pull Request`, `CodeCommit Pull Request State Change`, and `CodeCommit Repository State Change`. Targeting AWS Chatbot or an AWS Lambda function allows posting formatted notifications to Slack or Microsoft Teams.

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

