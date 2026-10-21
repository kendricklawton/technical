# Domain 5 (Incident and Event Response) — Quiz 6: AWS Fault Injection Service & Disaster Drills

**Topic:** AWS Fault Injection Service (FIS), Stop Conditions, Target Filters & AWS Health Event Handling  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: AWS Fault Injection Service (FIS) Stop Conditions

A DevOps team is setting up a chaos engineering experiment using AWS Fault Injection Service (FIS) to simulate high CPU utilization and network latency on Amazon EC2 instances in production. The enterprise has strict safety requirements: if customer-facing synthetic canaries detect error rates exceeding 1%, the chaos experiment must abort immediately and roll back.

**How does AWS FIS ensure chaos experiments do not cause runaway production outages?**
- A. By requiring an engineer to hold down a physical button during the experiment.
- B. By configuring **FIS Stop Conditions** linked to Amazon CloudWatch Alarms. If any specified CloudWatch Alarm enters the `ALARM` state, FIS immediately stops the experiment and rolls back fault actions to prevent catastrophic blast radius expansion.
- C. By restricting FIS to run only between 1 AM and 2 AM.
- D. By shutting down all internet gateways during experiments.

---

### Question 2: FIS Target Selection and Percentage Limits

A team wants to test the resiliency of an Amazon EKS microservice running 20 pods. The experiment should terminate pods randomly, but the team must guarantee that no more than 20% of the pods are terminated at any time to prevent service outages.

**How is target selection constrained in an AWS FIS experiment template?**
- A. By manually typing the pod names into the experiment template every morning.
- B. By configuring the **Target Selection Mode** to `Percent(20)` or `Count(4)` using target filters (such as resource tags and Kubernetes namespaces), ensuring FIS strictly limits the impacted target count regardless of fleet size.
- C. FIS can only target 100% of resources in a target group.
- D. By configuring an S3 lifecycle rule.

---

### Question 3: Simulating Multi-AZ Availability Zone Outages with FIS

An architect wants to validate whether their multi-AZ application will seamlessly survive the complete loss of an Availability Zone (`us-east-1a`).

**Which native AWS FIS action accurately simulates an Availability Zone outage?**
- A. Deleting the subnets in `us-east-1a`.
- B. Executing the **`aws:network:disrupt-connectivity`** action targeting the subnets in `us-east-1a`, which blocks all inbound and outbound network traffic across the AZ boundary at the hypervisor level.
- C. Powering down the entire AWS physical data center.
- D. Unsubscribing the AWS account from the us-east-1 Region.

---

### Question 4: Automated Response to AWS Health Events

AWS occasionally schedules hardware maintenance or detects underlying server degradation on EC2 instances hosting production workloads, issuing an `AWS_EC2_PERSISTENCE_INSTANCE_RETIREMENT_SCHEDULED` event via AWS Health.

**How should a DevOps team automate the proactive replacement of affected EC2 instances before the scheduled retirement deadline?**
- A. Wait for AWS to forcefully terminate the instance on the retirement date.
- B. Configure an **Amazon EventBridge Rule** matching events from `aws.health` with event type code `AWS_EC2_PERSISTENCE_INSTANCE_RETIREMENT_SCHEDULED`. The rule triggers an **AWS Systems Manager Automation runbook** or Lambda function that gracefully drains connections from the instance, stops/starts the instance (migrating it to healthy hardware), and verifies health checks.
- C. Delete the AWS account and recreate it.
- D. Ignore the notification because EC2 instances heal automatically without reboot.

---

### Question 5: AWS Health Organizational View for Multi-Account Monitoring

A large enterprise manages 300 AWS accounts. The operations team needs a centralized dashboard showing all AWS service disruptions, planned maintenance windows, and resource health notifications across all 300 accounts.

**Which AWS Health feature provides centralized multi-account health visibility?**
- A. Personal Health Dashboard installed on an on-premises server.
- B. **AWS Health Organizational View**, enabled in the AWS Organizations management account, which aggregates health events from all member accounts into a single view and allows centralized EventBridge event streaming.
- C. Route 53 Health Checks.
- D. AWS Trusted Advisor Performance Checks.

---

### Question 6: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap

A centralized CI/CD pipeline in Account 111111111111 deploys an ECS service in Account 222222222222. The deployment stage fails with: 'The ciphertext refers to a customer master key that does not exist, does not allow access, or is invalid.' The S3 bucket is encrypted with the default AWS-managed key 'aws/s3'. How should the engineer resolve this?

- A. Update the S3 bucket policy to grant read access to Account 222222222222.
- B. Reconfigure the S3 artifact bucket to use a Customer Managed Key (CMK) and grant `kms:Decrypt` and `kms:DescribeKey` permissions to Account 222222222222's execution role in the KMS Key Policy.
- C. Create a VPC Endpoint for S3 in Account 222222222222.
- D. Enable S3 Object Ownership with Bucket Owner Preferred.

---

### Question 7: CodeDeploy In-Place ApplicationStop Lifecycle Failure

During an in-place deployment to an EC2 Auto Scaling Group, the deployment fails at the ApplicationStop lifecycle event hook because a previous buggy script had been deployed in the older revision. How can the engineer force CodeDeploy to deploy the new fixed revision without terminating the instances?

- A. Delete the CodeDeploy agent from all EC2 instances.
- B. Trigger a new deployment via the AWS CLI with the `--ignore-application-stop-failures` flag enabled.
- C. Manually stop the EC2 instances in the AWS Management Console.
- D. Reboot the instances and rerun the same deployment ID.

---

### Question 8: CloudFormation Custom Resource 60-Minute Stack Freeze

A CloudFormation stack creation hangs for exactly 1 hour in `CREATE_IN_PROGRESS` on a Lambda-backed custom resource before failing and rolling back. What was the root cause?

- A. The Lambda function exceeded its maximum execution timeout of 15 minutes.
- B. An unhandled exception crashed the Lambda function before it could send an HTTP PUT response with status SUCCESS or FAILED to the pre-signed Amazon S3 ResponseURL.
- C. CloudFormation lost IAM permissions to invoke Lambda.
- D. The S3 bucket holding the CloudFormation template was deleted.

---

## Answer Key & Detailed Explanations

### Question 1: AWS Fault Injection Service (FIS) Stop Conditions

**Answer:** B  
**Explanation:** 
A fundamental principle of chaos engineering is blast radius control. AWS Fault Injection Service enforces this through **Stop Conditions**. A Stop Condition is an Amazon CloudWatch Alarm that monitors critical business health metrics (e.g., ALB 5xx error rate, synthetic canary success rate). If the alarm triggers while the experiment is running, FIS immediately terminates the experiment and cleans up injected faults.

---

### Question 2: FIS Target Selection and Percentage Limits

**Answer:** B  
**Explanation:** 
AWS FIS target definitions allow specifying the selection mode:
- `ALL`: Injects faults into all matching resources.
- `Count(N)`: Injects faults into exactly $N$ randomly selected targets.
- `Percent(N)`: Injects faults into $N$ percent of matching targets.
Combined with resource tags and filters, this guarantees that chaos experiments remain strictly bounded within safe operational thresholds.

---

### Question 3: Simulating Multi-AZ Availability Zone Outages with FIS

**Answer:** B  
**Explanation:** 
AWS FIS provides the `aws:network:disrupt-connectivity` action. When applied to subnets in a specific Availability Zone, AWS FIS disrupts network traffic between the specified subnets and other AZs/regions/internet at the AWS network hypervisor layer, perfectly simulating an entire AZ network isolation or power failure without destructively altering subnet route tables or deleting resources.

---

### Question 4: Automated Response to AWS Health Events

**Answer:** B  
**Explanation:** 
AWS Health events are published to Amazon EventBridge in real time. By creating an EventBridge rule for `source: ["aws.health"]`, organizations can catch scheduled maintenance and degradation events programmatically. Triggering an SSM Automation runbook allows automated, graceful migration (e.g., stop/start on EBS-backed instances moves them to fresh physical host hardware) during non-peak hours before AWS performs forceful retirement.

---

### Question 5: AWS Health Organizational View for Multi-Account Monitoring

**Answer:** B  
**Explanation:** 
**AWS Health Organizational View** integrates AWS Health with AWS Organizations. When enabled, it provides centralized visibility into service health events, resource outages, and scheduled maintenance across all accounts in the organization. Furthermore, EventBridge rules in the delegated administrator account can receive aggregated health events from all accounts, enabling organization-wide automated event handling.

---


### Question 6: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap
**Answer:** B
**Explanation:** Default AWS-managed keys like `aws/s3` cannot have their key policies modified and CANNOT be accessed across AWS accounts. Cross-account CodePipeline deployments strictly require an S3 bucket encrypted with a Customer Managed Key (CMK) whose key policy explicitly grants cross-account decrypt rights.

---

### Question 7: CodeDeploy In-Place ApplicationStop Lifecycle Failure
**Answer:** B
**Explanation:** CodeDeploy executes the `ApplicationStop` hook from the PREVIOUS revision's AppSpec file. If that script fails, all future deployments fail at that step unless you specify `--ignore-application-stop-failures`.

---

### Question 8: CloudFormation Custom Resource 60-Minute Stack Freeze
**Answer:** B
**Explanation:** CloudFormation Custom Resources expect a signed JSON payload uploaded to the pre-signed S3 `ResponseURL`. If the Lambda function crashes or exits without sending this signal, CloudFormation waits 1 hour before timing out.

---

