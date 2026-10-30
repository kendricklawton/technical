# AWS DevOps Engineer - Professional (DOP-C02) Practice Exams

Six advanced 15-question practice exams plus a 5-question calibration set, all written to one standard and calibrated to roughly twice the difficulty of the real DOP-C02 exam.

---

## Question Standard

Every question in this folder follows these rules:

- **Multi-paragraph scenario** that integrates three AWS services, with concrete numbers, an "already in place" fact the best answer must use, and at least one red herring.
- **Exactly one governing business constraint**, written in CAPITALS in the question line (for example, LEAST operational overhead, MOST cost-effective, FASTEST rollback, LEAST privilege).
- **Four options (A–D), single answer.** All four are working architectures that meet the functional requirements; the three distractors are wrong *only* because they violate the constraint. There are no throwaway options.
- **Balanced answer key and no length tell:** answer letters are evenly distributed, and the correct option is not systematically the longest (or shortest).
- **Answers hidden** in an answer key at the end of each file, with a Socratic walkthrough, the mechanism behind the correct answer, and the exact flaw of every distractor.
- **Current AWS only:** no EOL or closed-to-new-customer services (for example, OpsWorks, CodeStar, Incident Manager, CloudTrail Lake, CodeCatalyst), and current service names (Amazon Data Firehose, Amazon Q Developer in chat applications, AWS Security Hub CSPM).

---

## Exam Blueprint & Domain Weightings

| Domain | Domain Name | Exam Weight | Questions across the 6 exams |
|---|---|---|---|
| **Domain 1** | [SDLC Automation](../Domain_1_SDLC_Automation) | **22%** | 20 |
| **Domain 2** | [Configuration Management and IaC](../Domain_2_Configuration_Management_IaC) | **17%** | 15 |
| **Domain 3** | [Resilient Cloud Solutions](../Domain_3_Resilient_Cloud_Solutions) | **15%** | 14 |
| **Domain 4** | [Monitoring and Logging](../Domain_4_Monitoring_Logging) | **15%** | 13 |
| **Domain 5** | [Incident and Event Response](../Domain_5_Incident_Event_Response) | **14%** | 13 |
| **Domain 6** | [Security and Compliance](../Domain_6_Security_Compliance) | **17%** | 15 |

---

## Inventory

1. [`Practice_Exam_1.md`](Practice_Exam_1.md) — **SDLC & Deployment Pipeline Defense:** Lambda alias canary rollback, CloudFormation Guard hooks, least-privilege cross-account pipelines, ECS built-in blue/green, CodeArtifact domain design, Inspector CI/CD scanning, AppConfig rollback, IAM Roles Anywhere.
2. [`Practice_Exam_2.md`](Practice_Exam_2.md) — **Configuration Drift & Infrastructure Orchestration:** StackSets auto-deployment, CDK bootstrap least privilege, AWS Config daily recording, warm pools, CloudTrail-based drift alerts, Default Host Management Configuration, stack refactoring, zonal autoshift, CloudFormation Git sync.
3. [`Practice_Exam_3.md`](Practice_Exam_3.md) — **High Availability, DR & Traffic Failover:** headless Aurora global database pilot light, S3 Multi-Region Access Point failover, KMS multi-Region keys, CloudFront continuous deployment, ARC Region switch, EventBridge global endpoints, replica secrets.
4. [`Practice_Exam_4.md`](Practice_Exam_4.md) — **Enterprise Observability & Distributed Tracing:** OAM cross-account observability, version-scoped Lambda canary alarms, CloudWatch Application Signals, Logs data protection, CodeDeploy zonal deployments, Lambda destinations, Athena over CloudTrail, Flow Logs in Parquet, Lambda log-level controls.
5. [`Practice_Exam_5.md`](Practice_Exam_5.md) — **Event-Driven Incident Response & Chaos Engineering:** EC2 isolation and connection tracking, pipeline incident gates, conformance pack remediation, DLQ redrive, composite alarm suppression, EventBridge replay, FIS scenarios, exposed-key response, ECS deployment alarms.
6. [`Practice_Exam_6.md`](Practice_Exam_6.md) — **Organizational Governance & Security Guardrails:** resource control policies, GitHub OIDC least privilege, logically air-gapped backup vaults, declarative policies, Security Hub CSPM central configuration, centralized root access, Route 53 Profiles, tag policies, permissions-boundary delegation, CodeArtifact package groups.
7. [`Practice_Questions.md`](Practice_Questions.md) — **Calibration Set (5 questions):** one question per governing constraint; take it first.

---

## Instructions for Taking Practice Exams

- Set a timer for **75 minutes** per 15-question exam (≈5 minutes per question — about double the real exam's pace, matching the higher difficulty), or **25 minutes** for the calibration set.
- Answer every question before opening the answer key; don't consult documentation.
- Passing target: **12 / 15** per exam. Because these questions are harder than the real exam, 12+ indicates comfortable readiness.
- For every miss, read the full Socratic breakdown and note which constraint signal you overlooked.
