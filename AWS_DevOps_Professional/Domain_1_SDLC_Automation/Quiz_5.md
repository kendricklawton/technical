# Domain 1 (SDLC Automation) — Quiz 5: Version Control, Source Governance & Event-Driven Triggers

**Topic:** AWS CodeCommit, AWS CodeConnections, Branch Protection Policies & Event-Driven Triggering  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: Organization-Wide Multi-Repository Approval Governance in CodeCommit

A financial institution requires that any pull request targeting the `main` branch across 40 AWS CodeCommit repositories must receive at least 2 approvals from senior engineers in the `SecurityAuditors` IAM role before merging is permitted. The security governance policy must automatically apply to any new CodeCommit repositories created in the future.

**How should this governance control be enforced with the LEAST administrative overhead?**
- A. Create an **Approval Rule Template** in AWS CodeCommit with `ApprovalsNeeded: 2` and `ApprovalPoolMembers` targeting the `SecurityAuditors` role ARN, and associate the template with repository naming rules or all 40 repositories simultaneously.
- B. Deploy an AWS Lambda function triggered by an Amazon EventBridge rule on `CreateRepository` events that calls `codecommit:CreatePullRequestApprovalRule` for each repository individually.
- C. Write a custom pre-receive Git hook script that parses commit signatures and enforces review sign-offs on developers' local workstations before git pushes succeed.
- D. Attach a Service Control Policy (SCP) to the organization root that denies `codecommit:MergePullRequestBy*` unless the requesting session contains two distinct MFA claims.

---

### Question 2: Sub-Second Event-Driven Pipeline Triggering vs. Repository Polling

A DevOps team manages high-velocity deployment pipelines. Previously, AWS CodePipeline polled AWS CodeCommit every 5 minutes to detect commits, creating unwanted release delays and consuming unnecessary AWS KMS API request quotas. The team needs instant, sub-second pipeline execution upon git push to the `main` branch, while ensuring pushes to feature branches do not trigger the production pipeline.

**Which architecture satisfies this trigger requirement?**
- A. Configure an **Amazon EventBridge Rule** matching `source: aws.codecommit`, `detail-type: CodeCommit Repository State Change`, and filtering `referenceType: branch` with `referenceName: main`, targeting the CodePipeline pipeline ARN with an IAM service role.
- B. Enable S3 Event Notifications on the central pipeline artifact store to invoke an AWS Lambda function that queries the CodeCommit commit history API.
- C. Configure a CodeCommit Repository Trigger that publishes commit metadata directly to an Amazon SNS topic subscribed to an Amazon SQS queue polled by CodePipeline.
- D. Decrease the CodePipeline polling interval parameter to 5 seconds within the CodePipeline source stage action configuration settings.

---

### Question 3: Granular IAM Branch Protection & Direct Push Prevention

An enterprise security policy strictly prohibits developers from pushing commits directly to the `main` branch in an AWS CodeCommit repository. All modifications to `main` must arrive exclusively through reviewed and approved pull request merges. Developers must retain the ability to push commits to feature branches and create pull requests.

**Which IAM policy statement enforces this branch protection boundary?**
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
    "Resource": "arn:aws:codecommit:us-east-1:123456789012:CoreRepo",
    "Condition": {
      "StringEqualsIfExists": {
        "codecommit:References": [
          "refs/heads/main"
        ]
      }
    }
  }
  ```
- B.
  ```json
  {
    "Effect": "Deny",
    "Action": "codecommit:*",
    "Resource": "arn:aws:codecommit:us-east-1:123456789012:CoreRepo/branches/main"
  }
  ```
- C.
  ```json
  {
    "Effect": "Allow",
    "Action": "codecommit:GitPush",
    "Resource": "arn:aws:codecommit:us-east-1:123456789012:CoreRepo",
    "Condition": {
      "StringNotEquals": {
        "codecommit:Branch": "main"
      }
    }
  }
  ```
- D.
  ```json
  {
    "Effect": "Deny",
    "Action": "codecommit:CreatePullRequest",
    "Resource": "arn:aws:codecommit:us-east-1:123456789012:CoreRepo"
  }
  ```

---

### Question 4: Optimizing Massive Binary Asset Ingestion with Git Large File Storage (LFS)

A game development studio hosts its source code in AWS CodeCommit. Artists and designers frequently commit large 3D graphic models, textures, and video cutscenes (ranging from 500 MB to 1.8 GB per file). Developers report that standard `git clone` and `git checkout` operations take over 50 minutes, exhaust local memory, and frequently encounter network termination errors.

**What is the architectural best practice to manage these large binary files in AWS CodeCommit?**
- A. Configure **Git Large File Storage (Git LFS)** in the local repository and track binary extensions (`git lfs track "*.fbx" "*.psd"`); CodeCommit natively supports Git LFS, replacing large binaries in Git with lightweight text pointers while storing binary payloads in dedicated backend object storage.
- B. Instruct developers to compress large assets with multi-part gzip archives before committing and write a pre-build CodeBuild script to decompress files during CI.
- C. Store all binary assets in a public Amazon S3 bucket, and configure an Amazon CloudFront distribution with signed URLs embedded within application configuration code.
- D. Split large binary files into 50 MB byte chunks using shell utilities and assemble them dynamically using an EC2 UserData bootstrap script.

---

### Question 5: Frictionless Cross-Account CodeCommit Access via `git-remote-codecommit`

A contractor working within AWS Account `222222222222` needs to clone, commit, and push changes to an AWS CodeCommit repository hosted in corporate Account `111111111111`. Corporate security policy prohibits provisioning long-lived IAM Users or static SSH keys in Account `111111111111`, and mandates the use of short-lived AWS STS credentials.

**How should the contractor configure their local workstation to authenticate securely?**
- A. Install the **`git-remote-codecommit`** utility, configure an AWS CLI profile in Account `222222222222` that assumes an authorized IAM role in Account `111111111111`, and clone using the syntax: `git clone codecommit://ProfileName@CoreRepo`.
- B. Generate HTTPS Git credentials for an IAM User in Account `111111111111`, and configure the contractor's local Git credential helper to store the credentials in plaintext in `~/.git-credentials`.
- C. Deploy an AWS Storage Gateway File Gateway in Account `222222222222` mapped to Account `111111111111`'s repository S3 bucket, and mount the repository locally via NFS.
- D. Establish an AWS Site-to-Site VPN between Account `222222222222` and Account `111111111111`, allowing direct unauthenticated Git daemon access over port 9418.

---

### Question 6: Automated Pull Request State Monitoring & ChatOps Integration

An engineering organization wants immediate visibility into CodeCommit pull request workflows within a dedicated Slack team channel. Whenever a pull request is created, an existing pull request status changes to `MERGED` or `CLOSED`, or a team member posts a review comment, a formatted message must appear in Slack containing the pull request title, author, and direct console link.

**Which EventBridge event pattern accurately matches these CodeCommit pull request events?**
- A.
  ```json
  {
    "source": ["aws.codecommit"],
    "detail-type": [
      "CodeCommit Pull Request State Change",
      "CodeCommit Comment on Pull Request"
    ]
  }
  ```
- B.
  ```json
  {
    "source": ["aws.codecommit"],
    "detail-type": [
      "CodeCommit Repository State Change"
    ],
    "detail": {
      "event": ["pullRequestCreated", "commentAdded"]
    }
  }
  ```
- C.
  ```json
  {
    "source": ["aws.codepipeline"],
    "detail-type": [
      "CodePipeline Stage Execution State Change"
    ]
  }
  ```
- D.
  ```json
  {
    "source": ["aws.sns"],
    "detail-type": [
      "SNS Message Published"
    ]
  }
  ```

---

### Question 7: Secure Self-Managed GitLab Integration via AWS CodeConnections

An enterprise operates a self-managed GitLab Enterprise Server deployed inside an on-premises data center with no inbound internet access. The company is migrating its CI/CD pipelines to AWS CodePipeline and needs CodePipeline to trigger automatically upon merge requests and push events in the on-premises GitLab server.

**Which architecture connects AWS CodePipeline to this on-premises source repository securely?**
- A. Configure an **AWS CodeConnections Host** specifying provider type `GitLabEnterpriseServer`, configure an AWS Site-to-Site VPN or Direct Connect with a VPC endpoint, and register the connection to establish secure webhook communication with the private host.
- B. Deploy an Amazon EC2 instance running open-source Jenkins in a public subnet to poll GitLab via SSH and trigger CodePipeline executions using the AWS CLI.
- C. Configure a publicly accessible Application Load Balancer in AWS that proxies incoming GitLab webhook requests over the public internet to an Amazon SQS queue.
- D. Configure an S3 bucket in AWS and use a nightly cron job on the on-premises GitLab server that pushes git bundle archives to S3 using the AWS CLI.

---

### Question 8: Microservice Monorepo CI/CD Filtering: CodePipeline V2 vs. EventBridge

An enterprise maintains a mono-repository containing four decoupled microservices (`/billing`, `/catalog`, `/orders`, `/auth`). Each service has a dedicated pipeline. Pushing changes to the `/billing` service must trigger only the billing pipeline, while ignoring changes to documentation (`/docs/**`) and other microservices.

**What is the most maintainable and modern architectural approach to achieve path-filtered triggering?**
- A. Upgrade to **CodePipeline V2** and configure declarative **Git Triggers** with `FilePaths.Includes: ["billing/**"]` and `FilePaths.Excludes: ["docs/**", "**/*.md"]` directly on the `CodeStarSourceConnection` source action.
- B. Configure an Amazon EventBridge rule that captures repository state changes, invoking an AWS Lambda function that clones the repository, runs `git diff`, and starts pipelines via the AWS SDK.
- C. Add a pre-build shell script to each CodeBuild project that checks the commit diff and calls `aws codebuild stop-build` with status `SUCCESS` if no changes occurred in its directory.
- D. Split the monorepo into four distinct repositories, managing shared code using Git submodules.

---

### Question 9: Enforcing Cryptographic Commit Verification in CodeCommit CI/CD

A regulated banking institution mandates that all code pushed to production branches in AWS CodeCommit must be cryptographically signed using approved GPG keys to guarantee author non-repudiation and prevent unauthorized code injection. CodeCommit does not natively block unsigned git pushes.

**How can the DevOps team enforce signed commits as an automated compliance gate?**
- A. In the CodeBuild `install` or `pre_build` phase, configure the build script to inspect commit metadata using `git verify-commit HEAD` or verify the GPG signature against a central keyring stored in AWS Secrets Manager; exit with code `1` if the commit is unsigned or untrusted.
- B. Attach an IAM policy to developers denying `codecommit:GitPush` unless the condition `StringEquals: {"codecommit:GpgSignature": "valid"}` evaluates to true.
- C. Enable Amazon GuardDuty S3 Protection to scan git pack files in S3 and alert when GPG headers are absent.
- D. Configure an AWS WAF Web ACL on the CodeCommit endpoint to inspect HTTP POST payloads for PGP signature headers.

---

## Answer Key & Detailed Explanations

### Question 1: Organization-Wide Multi-Repository Approval Governance in CodeCommit
**Answer:** A  
**Explanation:**  
AWS CodeCommit **Approval Rule Templates** allow organizations to enforce consistent pull request approval workflows across multiple repositories simultaneously. By specifying `ApprovalsNeeded: 2` and defining `ApprovalPoolMembers` targeting the `SecurityAuditors` IAM role ARN, the rule is applied automatically to all associated repositories and can be configured to automatically attach to newly created repositories matching specific naming conventions.

---

### Question 2: Sub-Second Event-Driven Pipeline Triggering vs. Repository Polling
**Answer:** A  
**Explanation:**  
The AWS-recommended pattern for triggering CodePipeline from CodeCommit is **Amazon EventBridge**. When a developer pushes code, CodeCommit emits an event with `detail-type: CodeCommit Repository State Change`. An EventBridge rule filters for `referenceType: branch` and `referenceName: main`, targeting the CodePipeline execution ARN. This provides sub-second event-driven triggering, avoids polling latency, and consumes zero KMS API polling calls.

---

### Question 3: Granular IAM Branch Protection & Direct Push Prevention
**Answer:** A  
**Explanation:**  
In AWS CodeCommit, branch protection is enforced via IAM policies using the condition key `codecommit:References`. An explicit `Deny` statement on actions `codecommit:GitPush`, `codecommit:DeleteBranch`, `codecommit:PutFile`, and `codecommit:MergeBranchesByFastForward` with the condition `StringEqualsIfExists: {"codecommit:References": ["refs/heads/main"]}` prevents developers from pushing directly to `main`. Because pull request merges use distinct API actions (`MergePullRequestBySquash`, etc.), developers can still merge reviewed pull requests.

---

### Question 4: Optimizing Massive Binary Asset Ingestion with Git Large File Storage (LFS)
**Answer:** A  
**Explanation:**  
AWS CodeCommit provides native support for **Git Large File Storage (Git LFS)**. Git LFS replaces large files (up to 2 GB per file) with small text pointers inside the Git tree, storing the actual binary payloads in specialized scalable backend storage. When cloning or fetching, developers only download the specific binary versions needed, drastically reducing clone duration and preventing repository bloat.

---

### Question 5: Frictionless Cross-Account CodeCommit Access via `git-remote-codecommit`
**Answer:** A  
**Explanation:**  
The **`git-remote-codecommit`** extension provides the standard mechanism for interacting with CodeCommit repositories across accounts without long-lived credentials. By configuring an AWS CLI profile with `role_arn` targeting an IAM role in the destination account, the utility uses AWS STS to dynamically generate temporary credentials and sign Git requests using SigV4 over HTTPS, using syntax `codecommit://ProfileName@RepositoryName`.

---

### Question 6: Automated Pull Request State Monitoring & ChatOps Integration
**Answer:** A  
**Explanation:**  
AWS CodeCommit publishes distinct event types to Amazon EventBridge for pull request lifecycle events:  
- `CodeCommit Pull Request State Change`: Fires when a pull request is created, updated, merged, or closed.  
- `CodeCommit Comment on Pull Request`: Fires when an inline or general review comment is submitted.  
Matching these two event types in an EventBridge rule allows routing notifications to Amazon SNS and AWS Chatbot for Slack integration.

---

### Question 7: Secure Self-Managed GitLab Integration via AWS CodeConnections
**Answer:** A  
**Explanation:**  
**AWS CodeConnections** (formerly CodeStar Connections) supports **Hosts** for self-managed source providers (including GitLab Enterprise Server and GitHub Enterprise Server). By configuring a Connection Host associated with VPC configuration, CodeConnections securely tunnels over AWS Direct Connect or VPN to communicate with the on-premises GitLab server without exposing it to the public internet, providing native webhook triggers and branch synchronization.

---

### Question 8: Microservice Monorepo CI/CD Filtering: CodePipeline V2 vs. EventBridge
**Answer:** A  
**Explanation:**  
**CodePipeline V2** introduces native declarative **Git Triggers** supporting branch, tag, and file path filtering directly on the source connection. By defining `FilePaths.Includes` and `FilePaths.Excludes`, CodePipeline automatically evaluates commit diffs at the source stage and initiates execution *only* if files in the designated directory path were altered, completely eliminating custom Lambda poller functions.

---

### Question 9: Enforcing Cryptographic Commit Verification in CodeCommit CI/CD
**Answer:** A  
**Explanation:**  
AWS CodeCommit does not provide native server-side rejection of unsigned commits. To enforce cryptographic commit verification as a compliance gate, the verification must be executed within the CI/CD pipeline. In the CodeBuild `install` or `pre_build` phase, a script imports approved corporate GPG public keys and runs `git verify-commit HEAD`. If the commit signature is missing, invalid, or signed by an untrusted key, the build process exits with a non-zero exit code (`1`), aborting the pipeline before deployment.
