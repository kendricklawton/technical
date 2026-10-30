# Domain 5 (Incident and Event Response) — Quiz 5: Automated Security Incident Remediation

**Topic:** GuardDuty Alerts, Compromised EC2 Isolation, STS Credential Revocation & Security Hub Custom Actions  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: Automated Forensics Isolation of Compromised EC2 Instances

Amazon GuardDuty generates a high-severity finding `Backdoor:EC2/C&CActivity.B!DNS` indicating that a production EC2 instance is communicating with a known command-and-control server. Corporate security incident response protocols require an automated response pipeline that executes within 30 seconds to:
1. Isolate the instance from all network communication to halt data exfiltration.
2. Prevent the instance from being terminated by Auto Scaling scale-in policies.
3. Preserve the volatile RAM memory state and snapshot attached EBS volumes for legal forensic analysis.

**Which automated remediation workflow satisfies all three requirements?**
- A. An EventBridge rule matches the GuardDuty finding, triggering an SSM Automation runbook that: invokes `aws autoscaling set-instance-protection` to block termination, replaces the instance's Security Groups with a Quarantine Security Group (denying all inbound and outbound traffic), and triggers an EBS snapshot of all attached volumes without rebooting the instance.
- B. An EventBridge rule invokes an AWS Lambda function that immediately calls `aws ec2 stop-instances` and creates an AMI from the stopped instance.
- C. An AWS Config rule detects non-compliance and calls `aws ec2 terminate-instances` to eradicate the malware instantly.
- D. A CloudWatch Alarm monitoring network egress triggers an Auto Scaling Lifecycle Hook that transitions the instance into `Terminating:Wait`.

---

### Question 2: Instant Revocation of Compromised IAM Long-Term & Temporary STS Credentials

Amazon GuardDuty flags an IAM finding `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.OutsideAWS`, indicating that temporary security credentials issued by the EC2 instance metadata service (IMDS) were stolen and are being utilized from an external rogue IP address.

**How must the security remediation pipeline immediately neutralize the stolen credentials without deleting the IAM role required by legitimate application instances?**
- A. Attach an inline policy to the compromised IAM Role containing an explicit `Deny` on all actions (`Action: "*"`, `Resource: "*"`) with a condition `DateLessThan: { "aws:TokenIssueTime": "<current-utc-timestamp>" }`, which invalidates all temporary credentials issued prior to that moment.
- B. Delete the IAM Role and recreate it with a new name using AWS CloudFormation StackSets.
- C. Call `aws iam deactivate-mfa-device` and change the AWS account root user password.
- D. Execute `aws sts get-session-token` with a duration of 0 seconds to expire the global STS token cache.

---

### Question 3: Security Hub Custom Actions for One-Click SRE Remediation

A Security Operations Center (SOC) analyst reviewing findings in the AWS Security Hub console notices an Amazon S3 bucket flagged with `S3.1: S3 Block Public Access setting should be enabled`. Rather than automating instant remediation for every bucket across the enterprise (which might disrupt public legacy portals), the platform team wants analysts to review findings manually and click an action button inside the Security Hub console that triggers an automated remediation workflow.

**Which AWS architecture provides this one-click human-in-the-loop remediation?**
- A. Configure a **Security Hub Custom Action**: In Security Hub, create a custom action (e.g., `RemediateS3PublicAccess`); when an analyst selects a finding and clicks the action, Security Hub emits an event to **Amazon EventBridge**, which matches the custom action ARN and triggers an **SSM Automation runbook** to apply the fix.
- B. Write a Python tampermonkey browser script that executes AWS CLI commands from the analyst's local workstation.
- C. Configure an AWS Config manual remediation rule with a 15-minute polling delay.
- D. Deploy an Amazon SNS topic that sends an email to the AWS account root user requesting confirmation.

---

### Question 4: Preventing Lateral Movement via Subnet NACL Quarantine Automation

A distributed ransomware attack compromises multiple EC2 instances within a private subnet. The infected instances attempt lateral network scanning and SSH brute-force attacks against adjacent instances in the same VPC. Security groups allow internal VPC communication on port 22.

**How can the automated remediation system isolate the entire compromised subnet instantly at the network boundary without modifying hundreds of individual instance security groups?**
- A. An automated Lambda function modifies the subnet's **Network Access Control List (NACL)**: Inserts an explicit Rule number `1` with `RuleAction: DENY`, `Protocol: ALL`, `CidrBlock: 0.0.0.0/0`, immediately dropping all incoming and outgoing network packets at the subnet boundary.
- B. The Lambda function deletes the VPC Internet Gateway and detaches the Virtual Private Gateway.
- C. The Lambda function executes an SSM Run Command to update the `iptables` configuration inside the Linux kernels of all running instances.
- D. The Lambda function modifies the route table to point the `0.0.0.0/0` default route to a blackhole dummy interface.

---

### Question 5: Automated WAF IP Blocking via CloudWatch Logs & Athena / Lambda Integration

A public-facing Application Load Balancer is targeted by a distributed HTTP flood attack sending 50,000 requests per second from dynamically changing IP addresses. AWS WAF rate-limiting rules are overwhelmed. The security team needs an automated pipeline that analyzes ALB access logs or WAF logs in real time, identifies attacking IPs generating $> 2,000$ requests within 1 minute, and automatically adds those IP addresses to an **AWS WAF IP Set** configured with a `BLOCK` action.

**Which architecture satisfies this real-time dynamic mitigation requirement with MINIMAL latency?**
- A. Stream WAF or ALB logs to an **Amazon Kinesis Data Firehose** delivery stream, process records in real time using an **AWS Lambda data transformation function** that tracks request counts per IP in Amazon ElastiCache for Redis, and invoke `wafv2:UpdateIPSet` via the AWS SDK to block offenders dynamically.
- B. Run an Amazon Athena query against S3 logs every 10 minutes and manually export the results to an Excel spreadsheet.
- C. Configure a CloudWatch Metric Filter on ALB 4xx errors and attach an alarm that executes `aws ec2 stop-instances`.
- D. Deploy AWS Shield Advanced and open an emergency support ticket with the AWS Shield Response Team (SRT).

---

### Question 6: Automated Secrets Invalidation Upon Git Leakage via EventBridge

A software developer accidentally commits an AWS access key and secret key associated with a production IAM user to a public GitHub repository. GitHub's Secret Scanning detects the leak and notifies AWS immediately via the AWS Secret Partner program.

**How does AWS automatically remediate the compromised credential, and how can the DevOps team automate incident notification?**
- A. AWS immediately places an explicit deny policy on the exposed IAM user to quarantine the key and publishes an event to **Amazon EventBridge** from source `aws.health`; an EventBridge rule catches this event, triggers an automated runbook to delete the access key, and alerts the SRE team via Amazon SNS.
- B. AWS automatically deletes the entire AWS account within 5 minutes of detecting public credential leakage.
- C. GitHub executes a remote SSH command into the AWS management plane to terminate the user session.
- D. CloudTrail logs the event to Amazon S3, where an AWS Glue crawler indexes the key for administrative review.

---

### Question 7: Cross-Account Automated Remediation via Centralized Security Hub

An enterprise with 60 AWS member accounts operates a delegated Security Hub administrator account. When high-severity findings (such as unencrypted EBS volumes or open security groups) occur in any spoke workload account, the remediation must execute in the specific spoke account where the resource resides, orchestrated centrally from the Security Hub administrator account.

**How must the EventBridge and IAM architecture be structured to achieve centralized cross-account remediation?**
- A. Findings from all member accounts automatically consolidate into the Security Hub administrator account; an EventBridge rule in the central account triggers an SSM Automation runbook that assumes a pre-deployed `AWS-SystemsManager-AutomationExecutionRole` in the target spoke account to execute remediation locally.
- B. Each member account must run its own independent EventBridge rules and Lambda functions without central coordination.
- C. Member accounts export logs to an S3 bucket in the central account, which triggers a central Lambda function using root credentials.
- D. The central account shares its default EventBridge bus with member accounts via AWS Resource Access Manager (RAM).

---

### Question 8: Preserving Network Packet Forensics via VPC Traffic Mirroring

During an active security breach investigation on an Amazon EC2 instance hosting an e-commerce database, the forensics investigator needs to capture and analyze full payload network traffic (PCAP) traversing the instance's Elastic Network Interface (ENI) without installing third-party agent software on the operating system or rebooting the server.

**Which AWS native feature facilitates non-intrusive network packet capture?**
- A. Configure **VPC Traffic Mirroring**: Define the compromised instance's ENI as the **Mirror Source**, configure a dedicated network monitoring tool or Network Load Balancer as the **Mirror Target**, define a **Mirror Filter** capturing all inbound/outbound TCP/UDP protocols, and stream raw encapsulated VXLAN packets for deep packet inspection.
- B. Enable VPC Flow Logs on the instance's network interface and set the log aggregation interval to 1 minute.
- C. Install Wireshark on the instance over SSH and write pcap files directly to the root EBS volume.
- D. Configure an Application Load Balancer listener rule with a custom HTTP header logging policy.

---

### Question 9: Automated GuardDuty Suppression Rules for Testing vs Automated Remediation

An enterprise DevOps team frequently executes scheduled penetration testing and chaos engineering drills against a designated staging environment. These tests generate thousands of predictable GuardDuty findings that trigger automated remediation pipelines, inadvertently isolating test instances and creating alert fatigue.

**How should the security team prevent automated remediation from firing during legitimate testing without disabling GuardDuty detectors?**
- A. Configure **GuardDuty Suppression Rules**: Define filter criteria matching staging account IDs or resource tags (e.g., `Environment = Staging`), automatically archiving matching findings so they do not generate finding events on EventBridge or trigger automated remediation workflows.
- B. Disable the GuardDuty detector in the staging account during testing hours via EventBridge Scheduler.
- C. Modify the IAM execution role of the remediation Lambda function to ignore staging account findings using an `if/else` check in Python.
- D. Turn off CloudTrail logging in the staging account to prevent GuardDuty from analyzing API calls.

---

## Answer Key & Explanations

### Question 1: Automated Forensics Isolation of Compromised EC2 Instances
**Answer:** A  
**Explanation:** 
When isolating a compromised EC2 instance for forensic investigation:
1. **Prevent Termination:** Auto Scaling groups might terminate unhealthy or anomalous instances. Invoking `aws autoscaling set-instance-protection --instance-ids <id> --auto-scaling-group-name <asg> --protected-from-scale-in` prevents scale-in termination.
2. **Quarantine Security Group:** Replacing all existing security groups on the instance's ENI with a pre-created "Quarantine" security group (with **zero inbound and zero outbound rules**) immediately cuts off command-and-control communication and stops data exfiltration.
3. **Preserve Volatile Memory:** Stopping or rebooting the instance (Option B) destroys the volatile RAM memory where malware code, encryption keys, and active network sockets reside. The instance must be left running while EBS snapshots are taken, allowing investigators to extract forensic artifacts without data corruption.

---

### Question 2: Instant Revocation of Compromised IAM Long-Term & Temporary STS Credentials
**Answer:** A  
**Explanation:** 
When temporary credentials issued by the AWS Security Token Service (STS) or EC2 Instance Metadata Service (IMDS) are exfiltrated:
- Simply removing permissions from the IAM role or rotating instance metadata does **not** revoke existing active STS tokens; they remain valid until their expiration time (up to 1 hour for instance profiles).
- The AWS standard remediation mechanism is to attach an inline policy to the compromised IAM Role with an explicit **Deny** on all actions (`Action: "*"`, `Resource: "*"`) paired with the condition:
  `"DateLessThan": { "aws:TokenIssueTime": "2026-10-27T12:00:00Z" }`
- Because an explicit Deny always overrides any Allow, this invalidates every temporary session token issued prior to that timestamp instantly, while allowing newly issued tokens to function normally once the instance is secured.

---

### Question 3: Security Hub Custom Actions for One-Click SRE Remediation
**Answer:** A  
**Explanation:** 
**AWS Security Hub Custom Actions** provide human-in-the-loop operational remediation:
- In Security Hub, administrators define a Custom Action (e.g., `RemediateNonCompliantS3`).
- When an analyst reviews findings in the console, they select the target finding and choose the custom action from the dropdown menu.
- Security Hub emits an event to **Amazon EventBridge** containing the finding payload and the custom action ARN.
- An EventBridge rule matches this event and triggers an automated remediation target (such as an AWS Systems Manager Automation runbook or AWS Lambda function) to execute the fix, combining human judgment with automated execution.

---

### Question 4: Preventing Lateral Movement via Subnet NACL Quarantine Automation
**Answer:** A  
**Explanation:** 
Network Access Control Lists (NACLs) operate at the subnet boundary and evaluate rules in numerical order before security groups:
- When lateral movement within a subnet or between subnets must be halted immediately, modifying the subnet's NACL to insert Rule `1` with `RuleAction: DENY` for all protocols and all traffic (`0.0.0.0/0`) drops all incoming and outgoing packets at the hypervisor boundary instantly.
- NACLs are stateless and immediate. This cuts off all network connectivity to and from the entire subnet in milliseconds without waiting to modify individual instance security groups or running in-guest commands.

---

### Question 5: Automated WAF IP Blocking via CloudWatch Logs & Athena / Lambda Integration
**Answer:** A  
**Explanation:** 
For high-rate real-time threat mitigation:
- Streaming WAF or ALB logs directly into **Amazon Kinesis Data Firehose** provides continuous, sub-minute telemetry ingestion.
- An inline AWS Lambda function or a Kinesis stream consumer inspects incoming request records in real time, tracking client IP request frequencies using an in-memory key-value store (like Amazon ElastiCache for Redis).
- When an IP exceeds 2,000 requests/minute, the Lambda function invokes the AWS WAF API `UpdateIPSet` to add the malicious IP address to an IP Set associated with a WAF `Block` rule.
- Batch polling tools like Amazon Athena (Option B) introduce 10–15 minute delays that are far too slow for real-time flood defense.

---

### Question 6: Automated Secrets Invalidation Upon Git Leakage via EventBridge
**Answer:** A  
**Explanation:** 
Under the AWS Compromised Key Notification program:
- When a public git repository detects committed AWS credentials, it notifies AWS.
- AWS immediately attaches an AWS-managed policy (`AWSCompromisedKeyQuarantine`) to the IAM user, which explicitly denies sensitive mutating actions to prevent attacker abuse.
- AWS publishes an event to the account's **Amazon EventBridge** default bus (from service `aws.health`).
- DevOps teams configure an EventBridge rule to intercept this event, invoke an SSM Automation runbook to permanently delete the compromised access key, and notify the security team via Amazon SNS.

---

### Question 7: Cross-Account Automated Remediation via Centralized Security Hub
**Answer:** A  
**Explanation:** 
In enterprise multi-account architectures:
- AWS Security Hub findings from all member accounts are aggregated centrally into the designated Security Hub administrator account.
- The administrator account's EventBridge service catches finding events.
- An EventBridge rule initiates an AWS Systems Manager Automation runbook.
- The runbook assumes the pre-deployed cross-account IAM role (`AWS-SystemsManager-AutomationExecutionRole`) in the target member account where the non-compliant resource lives.
- The remediation actions (e.g., enabling encryption, closing ports) execute locally within the member account, maintaining strict audit separation and least privilege.

---

### Question 8: Preserving Network Packet Forensics via VPC Traffic Mirroring
**Answer:** A  
**Explanation:** 
**VPC Traffic Mirroring** provides native packet capture (PCAP) capabilities:
- It copies raw network traffic from an Elastic Network Interface (ENI) of an EC2 instance.
- Traffic is encapsulated in VXLAN packets and forwarded out-of-band to a **Mirror Target** (another ENI or Network Load Balancer hosting an intrusion detection tool like Suricata or Zeek).
- It operates at the nitro hypervisor level, meaning it captures 100% of packets (including payload) without consuming CPU/memory on the monitored instance and without installing software that an attacker could tamper with.

---

### Question 9: Automated GuardDuty Suppression Rules for Testing vs Automated Remediation
**Answer:** A  
**Explanation:** 
**GuardDuty Suppression Rules** allow organizations to filter out expected or benign findings automatically:
- You define filter criteria based on finding attributes (e.g., Account ID, tag `Environment = Staging`, finding type).
- When a matching finding occurs, GuardDuty automatically **archives** the finding.
- Archived findings **do not generate events on Amazon EventBridge** and do not increment CloudWatch metrics.
- This ensures that automated remediation pipelines and security on-call pagers are never triggered during legitimate staging drills, while preserving the full finding history in GuardDuty for compliance audits.
