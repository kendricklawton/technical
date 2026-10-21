# Domain 1: SDLC Automation — Essential White Papers & Documentation Guide

> **Blueprint Weight:** 22% of AWS Certified DevOps Engineer - Professional (DOP-C02)  
> **Core Objective:** Design, implement, and secure end-to-end CI/CD delivery pipelines across multi-account topologies, enforcing automated testing, artifact governance, immutable deployments, and zero-downtime release strategies.

---

## 1. Priority AWS White Papers & Architecture Guides

### 1.1 [Practicing Continuous Integration and Continuous Delivery on AWS](https://docs.aws.amazon.com/whitepapers/latest/practicing-continuous-integration-continuous-delivery/welcome.html)
* **Canonical Scope:** Architectural blueprint for enterprise CI/CD workflows utilizing AWS native developer tooling.
* **Key Focus Areas:**
  * **Pipelines as Code:** Transitioning from console-driven orchestration to codified pipeline definitions via AWS CloudFormation and AWS CDK.
  * **Phase Transitions & Isolation:** Separating Source, Build, Test, Staging, and Production stages with distinct blast radiuses and permission gates.
  * **Deployment Verification:** Automating synthetic transactions and smoke tests between deployment stages to trigger automatic rollbacks.
* **Critical Exam Takeaway:** Pipeline execution must prevent downstream contamination through automated gating; manual approval steps are reserved for cross-account boundary promotion or compliance sign-offs.

### 1.2 [Overview of Deployment Options on AWS](https://docs.aws.amazon.com/whitepapers/latest/overview-deployment-options/welcome.html)
* **Canonical Scope:** Comparative evaluation of all cloud deployment patterns (All-at-once, Rolling, Rolling with additional batch, Immutable, Blue/Green, Canary).
* **Architectural Matrix:**
  | Strategy | Downtime | Rollback Speed | Cost Impact | Production Capacity Risk |
  | :--- | :--- | :--- | :--- | :--- |
  | **All-at-Once** | High | Slow (re-deploy) | Neutral | High (100% outage during deploy) |
  | **Rolling** | None | Medium | Neutral | Degraded capacity during deployment |
  | **Rolling with Batch** | None | Medium | Minimal (1 batch cost) | Full capacity maintained |
  | **Immutable** | None | Fast (terminate new ASG) | High (temporarily doubles) | Full capacity maintained |
  | **Blue/Green** | None | Instantaneous (DNS / ALB switch)| High (100% duplicate fleet) | Zero capacity impact |
  | **Canary** | None | Instantaneous (shift traffic back)| Minimal | Sub-5% blast radius during test |
* **Critical Exam Takeaway:** ECS and Lambda blue/green deployments utilize AWS CodeDeploy deployment configurations (`CodeDeployDefault.ECSLinear10PercentEvery1Minute`, `CodeDeployDefault.LambdaCanary10Percent5Minutes`).

### 1.3 [Automating Safe Hands-Off Deployments (Amazon Builders' Library)](https://aws.amazon.com/builders-library/automating-safe-hands-off-deployments/)
* **Canonical Scope:** Internal Amazon operational practices for safe software deployment pipelines without human intervention.
* **Key Focus Areas:**
  * **Fractional Traffic Shifting:** Shifting small increments of real production traffic (e.g., 1% -> 10% -> 100%) combined with automated CloudWatch alarm rollbacks.
  * **Bake Times:** Mandatory soaking periods between pipeline stages to detect slow memory leaks, cache corruption, or thread exhaustion.
  * **Pipeline Safety Invariants:** Automatically blocking deployment pipelines during global holidays or active high-severity incidents.

---

## 2. Official AWS Documentation & User Guides to Master

### 2.1 [AWS CodePipeline V2 User Guide](https://docs.aws.amazon.com/codepipeline/latest/userguide/)
* **Deep Dive:**
  * **V2 Pipeline Features:** Dynamic stage execution, stage-level conditions, parameterized inputs, and execution rules.
  * **Cross-Account Deployments:** Central Tools Account orchestrating target Workload Accounts (Dev, Staging, Prod). Requires Customer Managed Key (CMK) in Tools account, cross-account IAM assume-role policies, and explicit S3 bucket policy delegation.
  * **Trigger Filters:** Native Git branch, pull request, and file path filters (avoiding unwanted builds on docs/README modifications).

### 2.2 [AWS CodeBuild User Guide](https://docs.aws.amazon.com/codebuild/latest/userguide/)
* **Deep Dive:**
  * **VPC Networking:** CodeBuild ENIs in VPC never receive public IP addresses. Must run in **Private Subnets** with NAT Gateway for outbound internet access.
  * **Caching Mechanisms:** Amazon S3 caching vs. Local Docker layer / custom directory caching.
  * **Buildspec Hierarchy:** Pre-build, build, and post-build phases. Note that `finally:` blocks run regardless of command exit status.

### 2.3 [AWS CodeDeploy User Guide](https://docs.aws.amazon.com/codedeploy/latest/userguide/)
* **Deep Dive:**
  * **AppSpec Lifecycle Hooks:**
    * *EC2/On-Premises:* `ApplicationStop` -> `DownloadBundle`* -> `BeforeInstall` -> `Install`* -> `AfterInstall` -> `ApplicationStart` -> `ValidateService` -> `BeforeAllowTraffic` -> `AllowTraffic`* -> `AfterAllowTraffic`. (*Reserved for agent).
    * *ECS:* `BeforeInstall` -> `Install`* -> `AfterInstall` -> `AllowTestTraffic`* -> `AfterAllowTestTraffic` -> `BeforeAllowTraffic` -> `AllowTraffic`* -> `AfterAllowTraffic`. Validation tests run on ALB Test Port during `AfterAllowTestTraffic`.
    * *Lambda:* `BeforeAllowTraffic` -> `AllowTraffic`* -> `AfterAllowTraffic`.

### 2.4 [AWS CodeArtifact User Guide](https://docs.aws.amazon.com/codeartifact/latest/ug/)
* **Deep Dive:** Domain vs. Repository hierarchies, external connections (npm, PyPI, Maven), cross-account repository sharing, and package group origin controls.

---

## 3. High-Frequency Exam Traps & Architectural Reflexes

1. **Cross-Account CodePipeline Access Denied:**
   * Default AWS-managed KMS key (`aws/s3`) CANNOT be shared across accounts.
   * *Resolution:* Use a Customer Managed Key (CMK) with a key policy permitting `kms:Decrypt`, `kms:GenerateDataKey*` to target account roles, and set S3 bucket ACL/policy granting cross-account write/read permissions.
2. **Lambda Canary Rollback:**
   * Pre-traffic and post-traffic test hooks run via Lambda functions specified in `appspec.yaml`.
   * If the test Lambda fails or a CloudWatch alarm associated with the deployment triggers, CodeDeploy immediately shifts all traffic back to the original alias version.
3. **EC2 Image Builder Pipelines:**
   * Automated golden AMI creation using recipe components (Build and Validate), infrastructure configurations (instance types, VPC, SNS), and distribution settings (cross-region, cross-account AMI sharing and KMS re-encryption).

---

## 4. Reading & Study Checklist

- [ ] Read *Practicing Continuous Integration and Continuous Delivery on AWS*
- [ ] Read *Overview of Deployment Options on AWS*
- [ ] Read *Automating Safe Hands-Off Deployments (Builders' Library)*
- [ ] Review CodePipeline cross-account architecture guide and IAM trust policies
- [ ] Memorize CodeDeploy AppSpec hook sequences for EC2, ECS, and Lambda
