# AWS Certified DevOps Engineer - Professional (DOP-C02)

Welcome to your comprehensive study, architecture, and hands-on lab workspace for the **AWS Certified DevOps Engineer - Professional (DOP-C02)** certification!

Each of the 6 domains is structured with **deep-dive master study guides**, **hands-on production architecture exercises with full reference solutions**, and **6 domain quizzes per domain** (36 domain quizzes total), supplemented by **six advanced 15-question practice exams (calibrated to ~2× real-exam difficulty) plus a 5-question calibration set** and a **[Master Cheat Sheet & Exam Battle Guide](DOP_C02_Master_Cheat_Sheet.md)**.

---

## 📁 Workspace Structure

| Domain / Folder | Exam Weight | Key Services & Topics | Study & Assessment Resources |
|---|---|---|---|
| **[`DOP_C02_Master_Cheat_Sheet.md`](DOP_C02_Master_Cheat_Sheet.md)** | **100%** | Lifecycle hooks, IAM/SCP/KMS evaluation matrices, DR RTO/RPO, Top 25 Exam Reflexes. | [`DOP_C02_Master_Cheat_Sheet.md`](DOP_C02_Master_Cheat_Sheet.md) |
| [`Domain_1_SDLC_Automation/`](Domain_1_SDLC_Automation) | **22%** | CodeCommit, CodeBuild, CodeDeploy, CodePipeline (V1/V2), CodeArtifact, EC2 Image Builder, SAM, Blue/Green ECS, Canary Lambda. | [`Study_Guide.md`](Domain_1_SDLC_Automation/Study_Guide.md) · [`Exercises.md`](Domain_1_SDLC_Automation/Exercises.md) · [Quizzes 1–6](Domain_1_SDLC_Automation) · [`White_Papers.md`](Domain_1_SDLC_Automation/White_Papers.md)
| [`Domain_2_Configuration_Management_IaC/`](Domain_2_Configuration_Management_IaC) | **17%** | CloudFormation (`CreationPolicy`, `UpdatePolicy`, `cfn-init`), StackSets, Custom Resources, `cfn-guard`, CDK, SSM (Parameter Store, Patch/State/Session Manager, AppConfig), Beanstalk. | [`Study_Guide.md`](Domain_2_Configuration_Management_IaC/Study_Guide.md) · [`Exercises.md`](Domain_2_Configuration_Management_IaC/Exercises.md) · [Quizzes 1–6](Domain_2_Configuration_Management_IaC) · [`White_Papers.md`](Domain_2_Configuration_Management_IaC/White_Papers.md)
| [`Domain_3_Resilient_Cloud_Solutions/`](Domain_3_Resilient_Cloud_Solutions) | **15%** | Auto Scaling Lifecycle Hooks, Warm Pools, Mixed Instances, ALB/NLB/GWLB, Route 53 & ARC, RDS Proxy, Aurora Global Databases, DynamoDB Global Tables, S3 CRR/RTC/MRAP, AWS DRS. | [`Study_Guide.md`](Domain_3_Resilient_Cloud_Solutions/Study_Guide.md) · [`Exercises.md`](Domain_3_Resilient_Cloud_Solutions/Exercises.md) · [Quizzes 1–6](Domain_3_Resilient_Cloud_Solutions) · [`White_Papers.md`](Domain_3_Resilient_Cloud_Solutions/White_Papers.md)
| [`Domain_4_Monitoring_Logging/`](Domain_4_Monitoring_Logging) | **15%** | CloudWatch Unified Agent, EMF, Composite Alarms, Metric Streams, Logs Subscription Filters, Data Protection, Contributor Insights, CloudTrail Lake, X-Ray, ADOT, VPC Flow Logs v5. | [`Study_Guide.md`](Domain_4_Monitoring_Logging/Study_Guide.md) · [`Exercises.md`](Domain_4_Monitoring_Logging/Exercises.md) · [Quizzes 1–6](Domain_4_Monitoring_Logging) · [`White_Papers.md`](Domain_4_Monitoring_Logging/White_Papers.md)
| [`Domain_5_Incident_Event_Response/`](Domain_5_Incident_Event_Response) | **14%** | EventBridge Patterns, Input Transformer, Archive/Replay, Global Endpoints, Pipes, SSM Incident Manager, AWS Chatbot, Automated Runbooks, Step Functions Forensics, FIS, Health API. | [`Study_Guide.md`](Domain_5_Incident_Event_Response/Study_Guide.md) · [`Exercises.md`](Domain_5_Incident_Event_Response/Exercises.md) · [Quizzes 1–6](Domain_5_Incident_Event_Response) · [`White_Papers.md`](Domain_5_Incident_Event_Response/White_Papers.md)
| [`Domain_6_Security_Compliance/`](Domain_6_Security_Compliance) | **17%** | IAM Evaluation Logic, Permission Boundaries, Roles Anywhere, SCPs, KMS Grants & Multi-Region Keys, Secrets Manager Rotation, AWS Config Conformance Packs, Service Catalog, GuardDuty, Inspector v2, WAF, Firewall Manager. | [`Study_Guide.md`](Domain_6_Security_Compliance/Study_Guide.md) · [`Exercises.md`](Domain_6_Security_Compliance/Exercises.md) · [Quizzes 1–6](Domain_6_Security_Compliance) · [`White_Papers.md`](Domain_6_Security_Compliance/White_Papers.md)
| [`Practice_Exams/`](Practice_Exams) | **100%** | Six advanced 15-question exams (~2× real-exam difficulty, constraint-driven distractors, Socratic answer keys) plus a calibration set, weighted to the official DOP-C02 blueprint. | [`README.md`](Practice_Exams/README.md) · [Exams 1–6](Practice_Exams) |

---

## 📝 Training Plan & Progress Tracker

### Step 0: Rapid Orientation
- [ ] Read [`DOP_C02_Master_Cheat_Sheet.md`](DOP_C02_Master_Cheat_Sheet.md) to internalize the top 25 DOP-C02 exam reflexes, lifecycle hook orders, and policy evaluation trees.

### Domain 1: SDLC Automation (22% — 6 Quizzes)
- [ ] [Domain 1 Master Study Guide](Domain_1_SDLC_Automation/Study_Guide.md)
- [ ] [Domain 1 Essential White Papers & Documentation](Domain_1_SDLC_Automation/White_Papers.md)
- [ ] [Domain 1 Hands-On Architecture Exercises & Solutions](Domain_1_SDLC_Automation/Exercises.md)
- [ ] [Quiz 1: CodePipeline, CodeBuild & CodeDeploy Foundations](Domain_1_SDLC_Automation/Quiz_1.md)
- [ ] [Quiz 2: Advanced Deployment Strategies (Canary, Blue/Green, Linear)](Domain_1_SDLC_Automation/Quiz_2.md)
- [ ] [Quiz 3: Container Pipeline Automation (ECS, ECR & AppSpec)](Domain_1_SDLC_Automation/Quiz_3.md)
- [ ] [Quiz 4: Cross-Account & Multi-Region CI/CD Pipelines](Domain_1_SDLC_Automation/Quiz_4.md)
- [ ] [Quiz 5: Automated Testing, Quality Gates & Secret Injection](Domain_1_SDLC_Automation/Quiz_5.md)
- [ ] [Quiz 6: Pipeline Failure Recovery & Rollback Automation](Domain_1_SDLC_Automation/Quiz_6.md)

### Domain 2: Configuration Management & Infrastructure as Code (17% — 6 Quizzes)
- [ ] [Domain 2 Master Study Guide](Domain_2_Configuration_Management_IaC/Study_Guide.md)
- [ ] [Domain 2 Essential White Papers & Documentation](Domain_2_Configuration_Management_IaC/White_Papers.md)
- [ ] [Domain 2 Hands-On Architecture Exercises & Solutions](Domain_2_Configuration_Management_IaC/Exercises.md)
- [ ] [Quiz 1: CloudFormation & SSM Foundations](Domain_2_Configuration_Management_IaC/Quiz_1.md)
- [ ] [Quiz 2: Advanced CloudFormation, Custom Resources & Macros](Domain_2_Configuration_Management_IaC/Quiz_2.md)
- [ ] [Quiz 3: CloudFormation StackSets & Multi-Account Deployment](Domain_2_Configuration_Management_IaC/Quiz_3.md)
- [ ] [Quiz 4: Systems Manager State Manager, Parameter Store & Session Manager](Domain_2_Configuration_Management_IaC/Quiz_4.md)
- [ ] [Quiz 5: Systems Manager Patch Manager & Maintenance Windows](Domain_2_Configuration_Management_IaC/Quiz_5.md)
- [ ] [Quiz 6: Elastic Beanstalk Deployment Policies & Configuration Drift](Domain_2_Configuration_Management_IaC/Quiz_6.md)

### Domain 3: Resilient Cloud Solutions (15% — 6 Quizzes)
- [ ] [Domain 3 Master Study Guide](Domain_3_Resilient_Cloud_Solutions/Study_Guide.md)
- [ ] [Domain 3 Essential White Papers & Documentation](Domain_3_Resilient_Cloud_Solutions/White_Papers.md)
- [ ] [Domain 3 Hands-On Architecture Exercises & Solutions](Domain_3_Resilient_Cloud_Solutions/Exercises.md)
- [ ] [Quiz 1: High Availability & Multi-AZ Foundations](Domain_3_Resilient_Cloud_Solutions/Quiz_1.md)
- [ ] [Quiz 2: Auto Scaling Lifecycle Deep Dive & AZ Balancing](Domain_3_Resilient_Cloud_Solutions/Quiz_2.md)
- [ ] [Quiz 3: Amazon Route 53 DNS Routing & Health Checks](Domain_3_Resilient_Cloud_Solutions/Quiz_3.md)
- [ ] [Quiz 4: Disaster Recovery Strategies & Routing Control (ARC)](Domain_3_Resilient_Cloud_Solutions/Quiz_4.md)
- [ ] [Quiz 5: High Availability & Distributed Databases](Domain_3_Resilient_Cloud_Solutions/Quiz_5.md)
- [ ] [Quiz 6: Resilient Storage, Messaging & Networking](Domain_3_Resilient_Cloud_Solutions/Quiz_6.md)

### Domain 4: Monitoring and Logging (15% — 6 Quizzes)
- [ ] [Domain 4 Master Study Guide](Domain_4_Monitoring_Logging/Study_Guide.md)
- [ ] [Domain 4 Essential White Papers & Documentation](Domain_4_Monitoring_Logging/White_Papers.md)
- [ ] [Domain 4 Hands-On Architecture Exercises & Solutions](Domain_4_Monitoring_Logging/Exercises.md)
- [ ] [Quiz 1: Core Monitoring & Metric Filters](Domain_4_Monitoring_Logging/Quiz_1.md)
- [ ] [Quiz 2: Advanced CloudWatch Alarms & Cross-Account Telemetry](Domain_4_Monitoring_Logging/Quiz_2.md)
- [ ] [Quiz 3: CloudWatch Logs Architecture & Streaming Pipelines](Domain_4_Monitoring_Logging/Quiz_3.md)
- [ ] [Quiz 4: Logs Insights, Contributor Insights & Data Protection](Domain_4_Monitoring_Logging/Quiz_4.md)
- [ ] [Quiz 5: CloudTrail Governance & Organization Trails](Domain_4_Monitoring_Logging/Quiz_5.md)
- [ ] [Quiz 6: Distributed Tracing with AWS X-Ray & ADOT](Domain_4_Monitoring_Logging/Quiz_6.md)

### Domain 5: Incident and Event Response (14% — 6 Quizzes)
- [ ] [Domain 5 Master Study Guide](Domain_5_Incident_Event_Response/Study_Guide.md)
- [ ] [Domain 5 Essential White Papers & Documentation](Domain_5_Incident_Event_Response/White_Papers.md)
- [ ] [Domain 5 Hands-On Architecture Exercises & Solutions](Domain_5_Incident_Event_Response/Exercises.md)
- [ ] [Quiz 1: Event-Driven Automation Foundations](Domain_5_Incident_Event_Response/Quiz_1.md)
- [ ] [Quiz 2: Enterprise EventBridge Patterns & Archive/Replay](Domain_5_Incident_Event_Response/Quiz_2.md)
- [ ] [Quiz 3: AWS Systems Manager Incident Manager & ChatOps](Domain_5_Incident_Event_Response/Quiz_3.md)
- [ ] [Quiz 4: Systems Manager Automation & Multi-Account Remediation](Domain_5_Incident_Event_Response/Quiz_4.md)
- [ ] [Quiz 5: Automated Security Incident Remediation](Domain_5_Incident_Event_Response/Quiz_5.md)
- [ ] [Quiz 6: AWS Fault Injection Service (FIS) & Disaster Drills](Domain_5_Incident_Event_Response/Quiz_6.md)

### Domain 6: Security and Compliance (17% — 6 Quizzes)
- [ ] [Domain 6 Master Study Guide](Domain_6_Security_Compliance/Study_Guide.md)
- [ ] [Domain 6 Essential White Papers & Documentation](Domain_6_Security_Compliance/White_Papers.md)
- [ ] [Domain 6 Hands-On Architecture Exercises & Solutions](Domain_6_Security_Compliance/Exercises.md)
- [ ] [Quiz 1: Identity & Permission Boundaries](Domain_6_Security_Compliance/Quiz_1.md)
- [ ] [Quiz 2: AWS Organizations & Service Control Policies (SCPs)](Domain_6_Security_Compliance/Quiz_2.md)
- [ ] [Quiz 3: AWS KMS Cryptography & Key Management](Domain_6_Security_Compliance/Quiz_3.md)
- [ ] [Quiz 4: Secrets Manager Lifecycle & Kubernetes CSI Driver](Domain_6_Security_Compliance/Quiz_4.md)
- [ ] [Quiz 5: AWS Config Governance & Conformance Packs](Domain_6_Security_Compliance/Quiz_5.md)
- [ ] [Quiz 6: Workload, Perimeter & Container Security](Domain_6_Security_Compliance/Quiz_6.md)

---

### Advanced Practice Exams (15 questions each, ~2× real-exam difficulty)
- [ ] [Practice Exam 1: SDLC & Deployment Pipeline Defense](Practice_Exams/Practice_Exam_1.md)
- [ ] [Practice Exam 2: Configuration Drift & Infrastructure Orchestration](Practice_Exams/Practice_Exam_2.md)
- [ ] [Practice Exam 3: High Availability, DR & Traffic Failover](Practice_Exams/Practice_Exam_3.md)
- [ ] [Practice Exam 4: Enterprise Observability & Distributed Tracing](Practice_Exams/Practice_Exam_4.md)
- [ ] [Practice Exam 5: Event-Driven Incident Response & Chaos Engineering](Practice_Exams/Practice_Exam_5.md)
- [ ] [Practice Exam 6: Organizational Governance & Security Guardrails](Practice_Exams/Practice_Exam_6.md)
- [ ] [Calibration Set — take this first](Practice_Exams/Practice_Questions.md)

---
*Pro-tip: When practicing in these folders, remember to run `terraform destroy` or delete your CloudFormation stacks after building to avoid unexpected AWS charges!*
