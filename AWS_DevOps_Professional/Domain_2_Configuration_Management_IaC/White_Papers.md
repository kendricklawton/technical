# Domain 2: Configuration Management and IaC — Essential White Papers & Documentation Guide

> **Blueprint Weight:** 17% of AWS Certified DevOps Engineer - Professional (DOP-C02)  
> **Core Objective:** Design, provision, and maintain compliant, deterministic cloud infrastructure at enterprise scale using CloudFormation, AWS CDK, and AWS Systems Manager, while detecting and mitigating configuration drift.

---

## 1. Priority AWS White Papers & Architecture Guides

### 1.1 [Infrastructure as Code (AWS Whitepaper)](https://docs.aws.amazon.com/whitepapers/latest/infrastructure-as-code/welcome.html)
* **Canonical Scope:** Architectural principles of treating cloud infrastructure with software engineering rigor.
* **Key Focus Areas:**
  * **Declarative vs. Imperative IaC:** CloudFormation/Terraform vs. CLI scripts; idempotency and eventual consistency reconciliation.
  * **Modularity & Layering:** Decoupling foundational networking (VPCs, Subnets), shared platform services (EKS, RDS), and application workloads into isolated stack lifecycles.
  * **Immutable Infrastructure:** Eliminating in-place server modification in favor of automated instance replacement via Auto Scaling and golden AMIs.

### 1.2 [AWS CloudFormation Best Practices](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html)
* **Canonical Scope:** Official enterprise engineering guidelines for writing robust CloudFormation templates.
* **Key Architectural Rules:**
  * **Reuse Templates via Nested Stacks:** Use nested stacks (`AWS::CloudFormation::Stack`) for components with shared lifecycles within an account/region.
  * **Cross-Stack References vs. Parameter Store:** Avoid hard-coupling stacks with `Export` / `Fn::ImportValue` when stack updates need to decouple outputs; use SSM Parameter Store dynamic references (`{{resolve:ssm:/path/param}}`) instead.
  * **DeletionPolicy and UpdateReplacePolicy:** Always configure `DeletionPolicy: Retain` or `Snapshot` on stateful resources (RDS, S3, EBS, DynamoDB) to prevent accidental data destruction on stack deletion or property replacement.

### 1.3 [AWS Systems Manager Best Practices Guide](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-best-practices.html)
* **Canonical Scope:** Enterprise systems management across hybrid architectures.
* **Key Capabilities:**
  * **State Manager:** Enforcing baseline OS configuration, software installation, and antivirus compliance with automated drift remediation.
  * **Patch Manager:** Automated patch baselines, maintenance windows, and compliance scanning without SSH/RDP.
  * **Default Host Management Configuration (DHMC):** Managing core EC2 instances without attaching custom instance profiles.

---

## 2. Official AWS Documentation & User Guides to Master

### 2.1 [AWS CloudFormation Advanced Features Guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/)
* **Deep Dive:**
  * **StackSets:** Multi-account, multi-region deployments managed centrally from AWS Organizations management account or delegated administrator. Service-managed permissions enable auto-deployment upon new account creation.
  * **Custom Resources:** Backed by AWS Lambda to perform custom logic (e.g., seeding database, configuring third-party APIs). Lambda MUST send response JSON to pre-signed S3 `ResponseURL` within 60 minutes or stack update hangs.
  * **CloudFormation Macros:** Pre-processing templates using Lambda before CloudFormation parses them (e.g., global string replacement, automated tag injection).
  * **CloudFormation Git Sync:** Native Git repository synchronization to trigger CloudFormation stack deployments upon pull request merge.

### 2.2 [AWS Systems Manager Parameter Store & AppConfig](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)
* **Deep Dive:**
  * **Parameter Store vs. Secrets Manager:** Parameter Store for hierarchical application configurations with Standard/Advanced tiers; Secrets Manager for automatic credential rotation and cross-account secret replication.
  * **AWS AppConfig:** Runtime configuration management with syntactic/semantic JSON schema validators, fractional deployment strategies (e.g., `AppConfig.Linear50PercentEvery30Seconds`), and automated rollback integration with CloudWatch alarms.

### 2.3 [AWS Elastic Beanstalk Configuration & Deployment](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/)
* **Deep Dive:**
  * `.ebextensions` configuration files executed in alphabetical order.
  * `container_commands` with `leader_only: true` for running schema migrations on exactly one instance during deployment.

---

## 3. High-Frequency Exam Traps & Architectural Reflexes

1. **CloudFormation Circular Dependency:**
   * Stack A exports VPC ID to Stack B; Stack B exports Security Group ID to Stack A.
   * *Resolution:* Break circular dependencies using SSM Parameter Store or refactoring shared resources into a common base stack.
2. **Custom Resource Timeout:**
   * Lambda throws an uncaught exception or VPC networking blocks access to S3 `ResponseURL`.
   * *Resolution:* Wrap handler in `try/except/finally`, ensuring `cfn-response` sends `FAILED` to S3 so stack rolls back cleanly instead of hanging for 1 hour.
3. **CloudFormation Guard (`cfn-guard`):**
   * Policy-as-code CLI and CloudFormation hook used to validate templates against security/compliance rules prior to resource creation.

---

## 4. Reading & Study Checklist

- [ ] Read *Infrastructure as Code (AWS Whitepaper)*
- [ ] Read *AWS CloudFormation Best Practices*
- [ ] Review CloudFormation StackSets architecture with AWS Organizations
- [ ] Review Custom Resource Lambda integration and `ResponseURL` protocol
- [ ] Master Systems Manager Patch Manager and State Manager association models
- [ ] Review AWS AppConfig deployment strategies and validators
