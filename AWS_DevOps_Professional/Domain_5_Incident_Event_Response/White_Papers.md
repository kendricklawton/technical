# Domain 5: Incident and Event Response — Essential White Papers & Documentation Guide

> **Blueprint Weight:** 14% of AWS Certified DevOps Engineer - Professional (DOP-C02)  
> **Core Objective:** Design and execute automated incident detection, event routing, real-time alerting, and automated remediation workflows, while verifying system resilience via chaos engineering drills.

---

## 1. Priority AWS White Papers & Architecture Guides

### 1.1 [Automating Incident Response in the Cloud (AWS Whitepaper)](https://docs.aws.amazon.com/whitepapers/latest/automating-incident-response/welcome.html)
* **Canonical Scope:** Architectural blueprint for converting security alerts and operational outages into self-remediating event-driven workflows.
* **Core Tenets:**
  * **Zero-Touch Mitigation:** Automatically isolating compromised resources (e.g., quarantining an EC2 instance by swapping Security Groups to a forensic SG, taking EBS snapshots, and terminating network connections).
  * **Event-Driven Architecture:** Decoupling detection systems (GuardDuty, Security Hub, Config) from remediation logic using Amazon EventBridge and AWS Step Functions.
  * **Immutable Evidence Collection:** Preserving volatile memory, system logs, and storage snapshots into dedicated, air-gapped forensic accounts.

### 1.2 [AWS Incident Response Guide](https://docs.aws.amazon.com/whitepapers/latest/aws-incident-response-guide/welcome.html)
* **Canonical Scope:** Operational playbook for preparing, detecting, containing, eradicating, and recovering from incidents in AWS environments.
* **Key Focus Areas:**
  * **Four Phases of Incident Response:** Preparation -> Detection & Analysis -> Containment, Eradication & Recovery -> Post-Incident Activity.
  * **Pre-Delegated Emergency Access:** Break-glass roles with temporary elevated IAM permissions, audited via AWS CloudTrail and AlertManager.

### 1.3 [Chaos Engineering on AWS & Fault Injection Service](https://docs.aws.amazon.com/fis/latest/userguide/)
* **Canonical Scope:** Principles of chaos engineering and automated resilience validation in AWS environments.
* **Core Practices:**
  * Formulating steady-state hypotheses before executing experiments.
  * Injecting realistic faults: network latency, AZ power failures, packet loss, CPU/memory spikes, and DB failovers using AWS FIS.
  * Automated stop conditions: configuring CloudWatch alarms that automatically abort FIS experiments if business SLAs are endangered.

---

## 2. Official AWS Documentation & User Guides to Master

### 2.1 [Amazon EventBridge User Guide](https://docs.aws.amazon.com/eventbridge/latest/userguide/)
* **Deep Dive:**
  * **Event Patterns:** Content filtering using JSON matching (prefix, wildcard, numeric range, IP address, null matching).
  * **Input Transformer:** Transforming incoming raw event schemas into clean, human-readable notifications for AWS SNS or Chatbot payloads.
  * **Archive and Replay:** Recording all events passing through an event bus to an EventBridge Archive; replaying historical events to test new microservices or recover from processing failures.
  * **Global Endpoints:** Multi-region automatic failover for event ingestion with Route 53 health check integration.
  * **EventBridge Pipes:** Point-to-point integrations connecting sources (DynamoDB Streams, Kinesis, SQS) to targets with optional filtering and enrichment (Lambda, Step Functions).

### 2.2 [AWS Systems Manager Incident Manager & Automation](https://docs.aws.amazon.com/incident-manager/latest/userguide/)
* **Deep Dive:**
  * **Response Plans:** Pre-configured playbooks defining chat channels (AWS Chatbot / Slack), escalation plans, and automated remediation runbooks.
  * **Automation Documents (SSM Runbooks):** Executing multi-step operational tasks with concurrency controls (`MaxConcurrency`, `MaxErrors`), approval gates, and multi-account execution (`AWS-RunPatchBaseline`, `AWS-CreateImage`).
  * **AWS Chatbot Integration:** Bi-directional ChatOps in Slack or Microsoft Teams allowing engineers to run SSM runbooks, approve CodePipeline actions, and query CloudWatch metrics directly from chat.

### 2.3 [AWS Health API & Health Dashboard](https://docs.aws.amazon.com/health/latest/ug/)
* **Deep Dive:**
  * Ingesting AWS personal health events (e.g., upcoming EC2 hardware retirement, certificate expiration) into EventBridge to trigger automated instance migration or notifications.

---

## 3. High-Frequency Exam Traps & Architectural Reflexes

1. **Compromised EC2 Instance Remediation:**
   * GuardDuty detects `UnauthorizedAccess:EC2/SSHBruteForce` or C2 beaconing.
   * *Automated Workflow:* EventBridge catches GuardDuty Finding -> Step Functions runs:
     1. Detaches instance from Auto Scaling Group (so ASG doesn't terminate it).
     2. Modifies Security Groups to an isolation SG (deny all inbound/outbound).
     3. Captures EBS snapshots for forensic analysis.
     4. Gathers memory dump via SSM Run Command.
2. **Dead-Letter Queue (DLQ) Redrive:**
   * Lambda or SQS processing failure sends poison-pill messages to DLQ.
   * *Resolution:* Use SQS DLQ Redrive to programmatically or console-driven route failed messages back to the source queue after the processing bug is fixed.
3. **EventBridge Event Bus Permissions:**
   * Cross-account event bus access requires an `AWS::EventBridge::EventBusPolicy` granting `events:PutEvents` to external accounts or an entire AWS Organization (`aws:PrincipalOrgID`).

---

## 4. Reading & Study Checklist

- [ ] Read *Automating Incident Response in the Cloud (AWS Whitepaper)*
- [ ] Read *AWS Incident Response Guide*
- [ ] Master EventBridge Event Patterns, Input Transformers, and Archive/Replay
- [ ] Review Systems Manager Incident Manager response plans and escalation policies
- [ ] Study AWS Fault Injection Service (FIS) experiment templates and stop conditions
- [ ] Master automated EC2 forensic isolation workflows via Step Functions
