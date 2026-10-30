# Domain 1 (SDLC Automation) — Quiz 6: Multi-Region Pipelines, GitOps & Enterprise CI/CD Governance

**Topic:** Multi-Region Deployment Topologies, Container Signing, GitOps Boundaries & Pipeline Auditing  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: Cross-Region Multi-Site CodePipeline Artifact & Encryption Topologies

An enterprise operates a high-availability active-active application across `us-east-1` (Primary) and `ap-southeast-1` (Secondary). A single continuous delivery pipeline hosted in `us-east-1` must compile application container images once in `us-east-1` and deploy identical artifacts to Amazon ECS clusters running in both regions simultaneously.

**How must the artifact store and cryptographic keys be defined within the AWS CodePipeline declaration?**
- A. Define an `artifactStores` mapping in the pipeline definition containing two regional entries: an S3 bucket in `us-east-1` encrypted with a `us-east-1` Customer Managed Key (CMK), and an S3 bucket in `ap-southeast-1` encrypted with an `ap-southeast-1` regional CMK.
- B. Define a single `artifactStore` pointing to an S3 bucket in `us-east-1` encrypted with the default `aws/s3` key, and enable Amazon S3 Cross-Region Replication (CRR) to sync objects to `ap-southeast-1`.
- C. Configure a global Amazon S3 Multi-Region Access Point (MRAP) as the pipeline's single artifact store, and configure the deployment actions in both regions to query the MRAP hostname.
- D. Deploy an AWS Storage Gateway Volume Gateway in `ap-southeast-1` mapped to the `us-east-1` S3 artifact bucket, and mount the volume to ECS container instances.

---

### Question 2: Cryptographic Container Supply Chain Signing with AWS Signer & Amazon EKS

A regulatory compliance standard mandates that all container images deployed to production Amazon EKS clusters must be cryptographically signed by an approved CI/CD build authority. If an image is pushed to Amazon ECR without a valid signature, or if an unsigned image is referenced in an EKS pod manifest, the deployment must be rejected at admission time.

**Which architectural pattern enforces this cryptographic container supply chain gate?**
- A. In AWS CodeBuild, sign the compiled container image using **AWS Signer** with a container signing profile and Notation CLI; push the OCI signature to Amazon ECR, and enforce signature validation on the EKS cluster using an admission controller (e.g. AWS Signer Gatekeeper or Kyverno).
- B. Configure an Amazon ECR repository policy that enforces a condition on `kms:Decrypt`, rejecting Docker push commands unless the client presents an approved PGP key.
- C. Encrypt the container image filesystem layers using an AWS KMS Customer Managed Key (CMK), and configure EKS worker nodes to decrypt layers using KMS Grants.
- D. Deploy an Amazon EventBridge rule that detects ECR `ImagePush` events, invoking an AWS Lambda function that terminates pods running unverified image digests.

---

### Question 3: Security Boundary Trade-offs: GitOps Pull-Based vs. CI/CD Push-Based Delivery

An enterprise platform architecture team is evaluating whether to adopt a GitOps delivery model (using Argo CD or Flux running on Amazon EKS) or maintain a centralized push-based CI/CD pipeline (using AWS CodePipeline deploying into member accounts).

**From an IAM and network security boundary perspective, what is the primary architectural advantage of the GitOps model?**
- A. In GitOps, the deployment agent runs **inside the target Kubernetes cluster** and pulls manifests outbound over HTTPS; external CI/CD servers do not require administrative network access or privileged cluster credentials (`sts:AssumeRole`), eliminating central control plane compromise risks.
- B. GitOps models completely bypass AWS IAM policy evaluation because Kubernetes role-based access control (RBAC) supersedes AWS identity evaluation inside worker nodes.
- C. GitOps natively encrypts all underlying Amazon EBS persistent volume claims without requiring AWS KMS Customer Managed Keys or storage encryption configurations.
- D. GitOps eliminates the need for Git version control repositories by synthesizing manifests dynamically in container memory during cluster startup.

---

### Question 4: Immutable Pipeline Execution Auditing and Regulatory Compliance

A financial regulatory audit requires an immutable, tamper-proof record of every production software release executed across the company for the past 7 years. The audit trail must record who approved the deployment, the commit SHA deployed, the pipeline execution ID, and timestamps of all stage transitions.

**Where is this audit data captured and durably preserved?**
- A. Configure **AWS CloudTrail** to record all CodePipeline and IAM API events (`StartPipelineExecution`, `PutApprovalResult`), delivering log files to an Amazon S3 bucket in a dedicated log archive account protected by **S3 Object Lock** in Compliance Mode.
- B. Export the AWS CodePipeline execution history console table to a CSV file every 30 days and upload it to an Amazon S3 Standard-IA bucket.
- C. Rely on the default AWS CodePipeline console history dashboard, which automatically preserves execution records indefinitely across all deleted pipelines.
- D. Configure an Amazon EventBridge rule that intercepts CodePipeline execution state change events and appends them to a local text file on an EC2 instance.

---

### Question 5: CodePipeline V2 Execution Modes for Ordered Database Migrations

An enterprise application relies on strict sequential database schema migrations (`V1 -> V2 -> V3`). If multiple developers merge pull requests within minutes of each other, pipeline executions must run through all deployment stages in strict First-In-First-Out (FIFO) chronological sequence without newer executions skipping or canceling earlier executions.

**Which AWS CodePipeline V2 execution mode must be configured?**
- A. Configure `executionMode: QUEUED`: CodePipeline processes every execution through all stages in strict FIFO order without superseding or skipping intermediate executions.
- B. Configure `executionMode: SUPERSEDED`: CodePipeline allows newer executions to overtake older executions at stage boundaries, canceling waiting releases.
- C. Configure `executionMode: PARALLEL`: CodePipeline executes multiple pipeline runs simultaneously across the same stages without waiting for prior runs to finish.
- D. Configure `executionMode: SERIAL_ISOLATED`: CodePipeline creates dynamic clone pipelines for each execution and deletes them upon completion.

---

### Question 6: Multi-Region Wave Deployments with Automated Blast Radius Containment

An enterprise deploys a mission-critical global service across 12 AWS regions. The release engineering team mandates a staged wave deployment: Wave 1 deploys to 1 canary region and waits 2 hours while monitoring synthetic health alarms; Wave 2 deploys to 3 secondary regions and waits 2 hours; Wave 3 deploys to the remaining 8 regions. If any region reports elevated error rates, all deployed regions must immediately roll back.

**Which architecture orchestrates this complex multi-stage, multi-region rollout with the LEAST custom code?**
- A. Deploy an **AWS Step Functions state machine** that coordinates the regional CodePipeline executions in sequential waves, utilizes native `Wait` states to enforce 2-hour observation windows, checks CloudWatch alarms, and triggers automated rollback states on failure.
- B. Deploy a single AWS CodePipeline pipeline containing 12 parallel deploy actions within a single stage, configuring a 2-hour manual approval action.
- C. Configure Amazon Route 53 Traffic Flow to shift DNS weights gradually across regions over a 6-hour window without modifying backend deployment pipelines.
- D. Create an Amazon EventBridge Scheduler schedule that triggers AWS CodeDeploy in-place deployments across all 12 regions simultaneously.

---

### Question 7: Cross-Account IAM Role Chaining & Service Role Delegation in CodePipeline

A central tools account (`111111111111`) hosts a CodePipeline pipeline deploying an application into a production account (`222222222222`). The deploy action uses AWS CloudFormation. The security team mandates strict separation of duties: the cross-account role assumed by CodePipeline must not possess administrative permissions to create resources directly; it must only be permitted to pass a dedicated CloudFormation execution role in Account `222222222222`.

**Which combination of IAM policies enforces this separation of duties? (Select TWO)**
- A. The Cross-Account Action Role in Account `222222222222` is granted `cloudformation:*` and `iam:PassRole` restricted specifically to the ARN of the CloudFormation Execution Role in Account `222222222222`.
- B. The CloudFormation Execution Role in Account `222222222222` has a trust policy permitting `cloudformation.amazonaws.com` and permissions granting least-privilege creation of application resources.
- C. The CodePipeline Service Role in Account `111111111111` is attached the `AdministratorAccess` AWS-managed policy and assumes the root account principal of Account `222222222222`.
- D. The Cross-Account Action Role trust policy allows `s3.amazonaws.com` to assume the role and grants full `iam:*` permissions across all organization accounts.
- E. The CloudFormation Execution Role is configured with an IAM permission boundary requiring all created resources to attach public IPv4 addresses.

---

### Question 8: Dynamic Pipeline Variables & Version Promotion in CodePipeline V2

A continuous delivery pipeline built in CodePipeline V2 compiles a Docker container image in an AWS CodeBuild action. The generated container image tag (formatted as `git_commit_sha-build_number`) must be passed dynamically into the downstream AWS CloudFormation deploy action to update the ECS task definition parameter without committing the tag to source code.

**How should this dynamic variable passing be configured in CodePipeline V2?**
- A. In the CodeBuild action, declare an **action output variable** (e.g. `IMAGE_TAG`); in the downstream CloudFormation deploy action, reference the variable using the syntax `#{CodeBuildNamespace.IMAGE_TAG}` in the template parameter overrides.
- B. Write the image tag to a text file in the CodeBuild container, upload it to an unencrypted S3 bucket, and configure CloudFormation to read the S3 object via UserData.
- C. Execute the `aws cloudformation update-stack` CLI command directly within the CodeBuild `post_build` phase, bypassing the CodePipeline deploy stage entirely.
- D. Store the image tag in an environment variable inside `buildspec.yml` and use `export` to set it as a global CodePipeline system variable.

---

### Question 9: Cross-Account S3 Bucket Policies and CMK Permissions for Artifact Promotion

A centralized deployment pipeline in Account `A` compiles software artifacts and stores them in an S3 bucket encrypted with a Customer Managed Key (CMK). The pipeline promotes artifacts across Account `B` (Staging) and Account `C` (Production). An update to Account `C`'s deployment role causes the production deploy action to fail with `KMS.AccessDeniedException`.

**What is the minimum set of permissions required on the CMK Key Policy in Account `A` to allow Account `C` to decrypt the artifact?**
- A. The KMS Key Policy in Account `A` must include a statement granting `kms:Decrypt` and `kms:DescribeKey` permissions with the `Principal` specifying the specific deployment role ARN in Account `C` (`arn:aws:iam::ACCOUNT_C:role/DeploymentRole`).
- B. The KMS Key Policy in Account `A` must grant `kms:*` to the root principal of Account `A`, and Account `C` must configure a VPC endpoint for KMS in its private subnets.
- C. Account `C`'s deployment role must assume the CodePipeline service role in Account `A` using AWS STS `AssumeRoleWithSAML`.
- D. The artifact bucket in Account `A` must disable KMS encryption and enable default S3 server-side encryption with Amazon S3-managed keys (`SSE-S3`).

---

## Answer Key & Detailed Explanations

### Question 1: Cross-Region Multi-Site CodePipeline Artifact & Encryption Topologies
**Answer:** A  
**Explanation:**  
When an AWS CodePipeline executes deploy or invoke actions in a region different from the pipeline's host region, a single S3 bucket cannot be used. CodePipeline requires the **`artifactStores`** mapping in the pipeline declaration. This mapping must specify a regional S3 artifact bucket and regional Customer Managed Key (CMK) in every region where an action executes. CodePipeline automatically replicates input artifacts between the regional S3 buckets before triggering regional actions.

---

### Question 2: Cryptographic Container Supply Chain Signing with AWS Signer & Amazon EKS
**Answer:** A  
**Explanation:**  
**AWS Signer** integrates with container registries and orchestrators to provide end-to-end cryptographic supply chain security. In the build stage (CodeBuild), the container image is signed using an AWS Signer signing profile with the Notation CLI, and the OCI signature artifact is pushed to Amazon ECR. On the Amazon EKS cluster, an admission controller (such as Kyverno or Gatekeeper with AWS Signer plugin) intercepts pod creation requests, validates the cryptographic signature against the approved signing profile, and rejects unsigned or tampered images.

---

### Question 3: Security Boundary Trade-offs: GitOps Pull-Based vs. CI/CD Push-Based Delivery
**Answer:** A  
**Explanation:**  
In traditional push-based CI/CD pipelines, the central CI/CD server or pipeline execution role must possess elevated administrative permissions and direct network connectivity to assume roles and execute API calls against every target Kubernetes cluster. In the **GitOps model**, an in-cluster agent (such as Argo CD or Flux) runs inside the cluster and pulls manifests outbound over standard HTTPS. External CI/CD systems only require write permissions to the Git repository, drastically shrinking the attack surface.

---

### Question 4: Immutable Pipeline Execution Auditing and Regulatory Compliance
**Answer:** A  
**Explanation:**  
**AWS CloudTrail** captures complete, non-repudiable API logs for all CodePipeline events (such as `StartPipelineExecution`, `PutApprovalResult`, `UpdatePipeline`). Delivering these logs to an Amazon S3 bucket protected with **S3 Object Lock** in **Compliance Mode** ensures that records are immutable (WORM storage) and cannot be deleted or overwritten by any user, including the root account, satisfying long-term compliance requirements.

---

### Question 5: CodePipeline V2 Execution Modes for Ordered Database Migrations
**Answer:** A  
**Explanation:**  
CodePipeline V2 provides three distinct execution modes:  
1. **`QUEUED`**: Executions run through all stages in strict First-In-First-Out (FIFO) chronological sequence without superseding or canceling earlier executions. This is mandatory for sequential database migrations.  
2. **`SUPERSEDED`**: A newer waiting execution cancels older waiting executions at stage boundaries.  
3. **`PARALLEL`**: Multiple executions proceed simultaneously without waiting for one another.

---

### Question 6: Multi-Region Wave Deployments with Automated Blast Radius Containment
**Answer:** A  
**Explanation:**  
Orchestrating multi-region wave deployments (deploying to wave 1, waiting 2 hours, deploying to wave 2, etc.) is an ideal architectural use case for **AWS Step Functions**. Step Functions natively manages asynchronous task execution, long-duration `Wait` states, parallel branches, and automated error catching (`Catch` / `Retry`) to execute coordinated rollback states across multiple regions if health alarms breach.

---

### Question 7: Cross-Account IAM Role Chaining & Service Role Delegation in CodePipeline
**Answer:** A, B  
**Explanation:**  
To enforce strict least privilege and separation of duties in cross-account CloudFormation deployments:  
1. The **Cross-Account Action Role** in the target account is assumed by CodePipeline; it only requires `cloudformation:*` and `iam:PassRole` restricted specifically to the CloudFormation Execution Role ARN (Action A).  
2. The **CloudFormation Execution Role** has a trust policy trusting `cloudformation.amazonaws.com` and holds the actual permissions needed to provision the application infrastructure (Action B).

---

### Question 8: Dynamic Pipeline Variables & Version Promotion in CodePipeline V2
**Answer:** A  
**Explanation:**  
AWS CodePipeline supports **Action Output Variables**. When an action (such as CodeBuild) produces an output variable (e.g. `IMAGE_TAG`), downstream actions in subsequent stages can consume that variable using namespace syntax: `#{NamespaceName.VariableName}` within parameter overrides or configuration blocks, allowing dynamic artifact references without manual Git commits.

---

### Question 9: Cross-Account S3 Bucket Policies and CMK Permissions for Artifact Promotion
**Answer:** A  
**Explanation:**  
For an external account role (Account `C`) to decrypt artifacts stored in Account `A`'s S3 bucket, the Customer Managed Key (CMK) policy in Account `A` must explicitly grant `kms:Decrypt` and `kms:DescribeKey` permissions to Account `C`'s deployment IAM role ARN. Default AWS-managed keys (`aws/s3`) cannot be used because their key policies cannot be modified to trust external accounts.
