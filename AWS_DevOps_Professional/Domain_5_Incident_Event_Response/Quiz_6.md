# Domain 5 (Incident and Event Response) — Quiz 6: AWS Fault Injection Service & Disaster Drills

**Topic:** AWS Fault Injection Service (FIS), Stop Conditions, Target Filters & AWS Health Event Handling  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: Circuit Breaker Safety in Chaos Experiments via FIS Stop Conditions

A financial platform SRE team is authoring an AWS Fault Injection Service (FIS) experiment template to simulate the abrupt loss of an Amazon Aurora database writer instance during peak business hours. To protect end customers from catastrophic service degradation, executive management mandates that if overall application API error rates exceed 3% for more than 60 seconds at any time during the experiment, the chaos test must halt immediately and rollback any active disruptions.

**Which architectural component enforces this automated experiment safety boundary?**
- A. Configure a **CloudWatch Alarm** monitoring application 5xx error rate ($> 3\%$) with an evaluation period of 60 seconds, and attach the alarm ARN as a **Stop Condition** in the AWS FIS Experiment Template.
- B. Deploy an AWS Lambda function that polls the FIS API every 5 seconds and calls `aws fis stop-experiment` if errors rise.
- C. Write an Amazon EventBridge rule that detects FIS experiment state changes and disables the Auto Scaling Group.
- D. Attach an IAM permissions boundary to the FIS service role that revokes permissions if CloudWatch alarms trigger.

---

### Question 2: Simulating Availability Zone Outages via FIS Network Disruption

An enterprise operates a mission-critical microservice fleet deployed across three Availability Zones (`us-east-1a`, `us-east-1b`, `us-east-1c`). The architecture team needs to validate that the application gracefully survives a complete network partition affecting `us-east-1a` (simulating 100% packet loss between `us-east-1a` and all other zones/VPCs) for 15 minutes without modifying security group rules or restarting instances.

**Which AWS FIS action enables this network-level fault injection natively?**
- A. Use the **`aws:network:disrupt-connectivity`** FIS action: Target the subnets in `us-east-1a`, configure the disruption type to block traffic between the target subnets and external subnets/AZs, and specify a duration of 15 minutes.
- B. Execute an SSM Automation runbook that deletes the VPC route table entries corresponding to `us-east-1a`.
- C. Deploy an AWS Network Firewall rule group that drops all TCP packets with source IP belonging to `us-east-1a`.
- D. Execute an `aws:ssm:send-command` action that runs `iptables -F` inside the Linux operating systems.

---

### Question 3: FIS Target Selection Filters and Percentage Scoping

An SRE is configuring an FIS experiment to test container resilience on an Amazon ECS on EC2 cluster consisting of 100 container host instances. The test requires draining container instances, but the blast radius must be strictly capped: exactly 20% of instances tagged `Role = Worker` in Availability Zone `us-west-2a` must be selected at random.

**How should the FIS experiment Target definition be configured?**
- A. Configure the Target with `ResourceType: aws:ec2:instance`, specify **Target Filters** for `tag:Role = Worker` and `placement-availability-zone = us-west-2a`, and set **Selection Mode** to **`PERCENT(20)`**.
- B. Configure the Target with `ResourceType: aws:ecs:container-instance`, set `SelectionMode: ALL`, and manually list the 20 instance IDs in parameters.
- C. Configure an Auto Scaling Group lifecycle hook with `HeartbeatTimeout: 300` and `DefaultResult: ABANDON`.
- D. Set the Target Selection Mode to `COUNT(100)` and configure an inline Python script to filter instances using modulo division.

---

### Question 4: Automated Resilience Verification in CI/CD Deployment Pipelines

A platform engineering team wants to implement continuous resilience testing in AWS CodePipeline. Before an application revision can be promoted from the Staging environment to Production, the pipeline must automatically trigger an AWS FIS chaos experiment (injecting 50% CPU stress across staging nodes) and verify that Auto Scaling successfully scales out the fleet to maintain p99 response times under 200ms.

**How should this automated resilience gate be implemented within AWS CodePipeline?**
- A. Add a CodePipeline stage with an **AWS CodeBuild** or **AWS Lambda** action that invokes `aws fis start-experiment`, monitors experiment execution via the SDK until completion, and asserts that CloudWatch metric alarms for latency remained healthy before passing the stage.
- B. Embed the raw FIS Experiment Template JSON directly inside the application's `appspec.yml` file under the `AfterAllowTestTraffic` hook.
- C. Configure the Auto Scaling Group `NotificationConfigurations` to trigger FIS experiments whenever scale-out occurs.
- D. Add an AWS CodeDeploy Blue/Green deployment with traffic rerouting set to `AllAtOnce` and attach FIS directly to CodeDeploy.

---

### Question 5: Simulating EC2 Hardware Degradation via AWS Health Events

An enterprise architecture standard mandates that all applications must gracefully handle planned and unplanned AWS hardware retirements without human intervention. To prepare for disaster drills, the DevOps team needs to simulate an incoming `AWS_EC2_PERSISTENT_INSTANCE_RETIREMENT_SCHEDULED` event across development instances to verify that an automated event-driven replacement pipeline operates correctly.

**Which architecture allows the team to simulate AWS Health events and validate automated remediation?**
- A. Publish a mock event matching the schema of `AWS Health Event` (`source: "aws.health"`, `eventTypeCode: "AWS_EC2_PERSISTENT_INSTANCE_RETIREMENT_SCHEDULED"`) directly to the account's **Amazon EventBridge** default bus, triggering the automated remediation pipeline.
- B. Open a high-severity AWS Support case requesting AWS engineers to power down physical host racks in the data center.
- C. Run the AWS CLI command `aws health generate-mock-event --event-type AWS_EC2_RETIREMENT`.
- D. Stop the EC2 instance using the AWS CLI and manually edit the instance lifecycle metadata.

---

### Question 6: AWS FIS Service Role and Least-Privilege IAM Boundaries

A security administrator is reviewing the IAM permissions required to execute an AWS FIS experiment that stops Amazon EC2 instances and reboots Amazon RDS database clusters. During experiment execution, the FIS experiment immediately transitions to `Failed` status with an error message: `Action failed due to insufficient permissions`.

**What are the required IAM trust and permission configurations for AWS Fault Injection Service?**
- A. The FIS IAM Role must have a **Trust Policy** granting `sts:AssumeRole` to the service principal **`fis.amazonaws.com`**, and an **Identity Policy** granting specific permissions (`ec2:StopInstances`, `rds:RebootDBInstance`) scoped strictly to the targeted resource ARNs and tags.
- B. The FIS experiment must use the IAM Instance Profile attached to the target Amazon EC2 instances.
- C. The FIS IAM Role must be granted `AdministratorAccess` at the AWS Organizations management account root.
- D. AWS FIS does not use IAM roles; it authenticates using the IAM credentials of the user who clicked "Start Experiment" in the console.

---

### Question 7: In-Guest Resource Starvation Experiments via SSM Actions in FIS

A cloud architect needs to test whether an e-commerce checkout application deployed on EC2 can survive severe memory leaks. The architect wants to run an FIS experiment that consumes 90% of available RAM on target Linux instances for 10 minutes, observing whether target tracking policies scale out new instances before existing instances crash with Out-Of-Memory (OOM) kernel panics.

**How does AWS FIS inject OS-level resource starvation inside EC2 instances without opening SSH ports?**
- A. Use the **`aws:ssm:send-command`** action in the FIS template: Specify the AWS-managed SSM document **`AWSFIS-Run-Memory-Stress`**, define parameter `DurationSeconds: 600` and `MemoryPercent: 90`, executing the stress injection securely via the pre-installed SSM Agent.
- B. Configure an Auto Scaling launch template user data script that downloads a custom C program to allocate malloc loops.
- C. Deploy an AWS Lambda function that initiates SSH connections using public IP addresses and runs `dd if=/dev/zero of=/dev/null`.
- D. Use the FIS action `aws:ec2:modify-instance-attribute` to reduce instance RAM capacity to 1 GB.

---

### Question 8: Chaos Testing on Amazon ECS and Amazon EKS Workloads

A microservice architecture runs containerized workloads across Amazon ECS on AWS Fargate and Amazon EKS clusters. The reliability team wants to inject faults that randomly terminate 10% of application pods in EKS and drain container tasks in ECS to verify that service meshes and Kubernetes ReplicaSets self-heal immediately.

**Which FIS action types support containerized orchestration platforms natively?**
- A. Use FIS actions **`aws:ecs:drain-container-instances`** or **`aws:ecs:task-kill`** for ECS workloads, and execute **`aws:eks:inject-kubernetes-custom-resource`** or Chaos Mesh actions targeting EKS pods.
- B. Write a Kubernetes DaemonSet that calls `kubectl delete pod` in an infinite while loop.
- C. Manually execute `docker stop` via the Amazon ECR console against container image tags.
- D. Disable the Amazon VPC CNI plugin across the EKS worker nodes using an AWS CloudFormation stack update.

---

### Question 9: Validating Recovery Objectives (RTO / RPO) via FIS Post-Experiment Evaluation

Following an FIS experiment that simulated an unplanned Multi-AZ database failover on Amazon Aurora, the DevOps team needs to scientifically quantify the exact Recovery Time Objective (RTO) and Recovery Point Objective (RPO) observed during the drill.

**Which observability mechanism provides the empirical evidence required to validate DR metrics?**
- A. Deploy **CloudWatch Synthetics Canaries** sending heartbeat transactions every 5 seconds to measure the exact downtime window (RTO), and compare database transaction log commit sequences between writer and replica to verify zero data loss (RPO = 0).
- B. Review the CloudTrail audit log to verify the timestamp when the FIS `StopExperiment` API call was executed.
- C. Inspect the Auto Scaling Group desired capacity metric graph in the AWS Management Console.
- D. Measure the duration that the FIS Experiment execution status remained in `Running`.

---

## Answer Key & Explanations

### Question 1: Circuit Breaker Safety in Chaos Experiments via FIS Stop Conditions
**Answer:** A  
**Explanation:** 
Safety is the fundamental pillar of Chaos Engineering:
- **AWS Fault Injection Service (FIS) Stop Conditions** act as automated emergency circuit breakers.
- An experiment template must configure one or more Stop Conditions linked to **Amazon CloudWatch Alarms**.
- If a monitored alarm breaches its threshold (e.g., API 5xx error rate $> 3\%$ for 60 seconds), FIS immediately halts the experiment, ceases all active fault actions, and restores the targeted infrastructure to its normal state.
- This guarantees that chaos experiments can run safely in production without risking widespread customer impact or SLA violations.

---

### Question 2: Simulating Availability Zone Outages via FIS Network Disruption
**Answer:** A  
**Explanation:** 
The AWS FIS **`aws:network:disrupt-connectivity`** action allows injecting network-level faults natively at the hypervisor boundary without installing agent software or editing network configurations:
- You target specific subnets within an Availability Zone (e.g., `us-east-1a`).
- You configure the scope of the disruption (e.g., blocking traffic to all other subnets in the VPC, specific AZs, or external endpoints).
- FIS intercepts network packets at the AWS virtualization layer, dropping packets to simulate a total AZ network severance.
- When the experiment duration elapses or a stop condition triggers, FIS immediately removes the disruption, restoring full network connectivity seamlessly.

---

### Question 3: FIS Target Selection Filters and Percentage Scoping
**Answer:** A  
**Explanation:** 
In an AWS FIS Experiment Template, the **Target** definition controls the blast radius:
- **Filters:** Narrow down candidate resources based on AWS tags (`tag:Role = Worker`), Availability Zone (`placement-availability-zone = us-west-2a`), or instance states.
- **Selection Mode:** Dictates how many matching resources are selected for fault injection:
  - `PERCENT(n)`: Randomly selects a percentage of matching resources (e.g., `PERCENT(20)` selects 20% of the filtered instances).
  - `COUNT(n)`: Selects a fixed absolute number of resources.
  - `ALL`: Selects every matching resource.
Using `PERCENT(20)` with tag and AZ filters guarantees that exactly 20% of matching worker instances are targeted.

---

### Question 4: Automated Resilience Verification in CI/CD Deployment Pipelines
**Answer:** A  
**Explanation:** 
Integrating chaos experiments into CI/CD deployment pipelines ensures that resilience regressions are caught before code reaches production:
- In AWS CodePipeline, an **AWS CodeBuild** or **AWS Lambda** action can invoke `aws fis start-experiment --experiment-template-id <id>`.
- The build script polls the experiment status until completion.
- The pipeline verifies that the application satisfied its resilience objectives (e.g., Auto Scaling triggered, error rate alarms did not breach).
- If the application failed the resilience test (e.g., latency breached thresholds or the service crashed), the CodePipeline stage fails, automatically halting deployment and rolling back changes.

---

### Question 5: Simulating EC2 Hardware Degradation via AWS Health Events
**Answer:** A  
**Explanation:** 
AWS Health events (such as hardware retirement notifications) cannot be manufactured out-of-band by user command in the AWS Health service itself. However, because automated remediation architectures are event-driven:
- The remediation pipeline listens on **Amazon EventBridge** for events with `source: "aws.health"`.
- To test the automation during a drill, engineers emit a synthetic event conforming to the official `AWS Health Event` schema (specifying `detail-type: "AWS Health Event"` and `eventTypeCode: "AWS_EC2_PERSISTENT_INSTANCE_RETIREMENT_SCHEDULED"`) directly onto the default EventBridge bus using `aws events put-events`.
- The EventBridge rule intercepts the mock event and triggers the remediation pipeline, verifying that the automated replacement workflow works as intended.

---

### Question 6: AWS FIS Service Role and Least-Privilege IAM Boundaries
**Answer:** A  
**Explanation:** 
AWS Fault Injection Service executes actions on your behalf using an IAM service role:
- **Trust Relationship:** Must allow the FIS service principal (`fis.amazonaws.com`) to assume the role via `sts:AssumeRole`.
- **Identity Permissions:** Must grant specific AWS API permissions required to perform the declared actions (e.g., `ec2:StopInstances` for EC2 stop actions, `rds:RebootDBInstance` for database reboots).
- Following the principle of least privilege, permissions should be scoped strictly to targeted resource ARNs and tag conditions rather than granting broad administrative rights.

---

### Question 7: In-Guest Resource Starvation Experiments via SSM Actions in FIS
**Answer:** A  
**Explanation:** 
To inject operating system-level faults (such as memory leaks, CPU spikes, disk fill, or process termination) without opening network ports:
- AWS FIS integrates natively with **AWS Systems Manager (SSM)** via the `aws:ssm:send-command` action.
- AWS provides pre-authored, managed SSM documents for chaos injection, such as **`AWSFIS-Run-Memory-Stress`** and `AWSFIS-Run-CPU-Stress`.
- The action invokes the SSM document via the in-guest SSM Agent, passing duration and intensity parameters.
- Because communication runs entirely over outbound TLS to Systems Manager endpoints, no inbound ports or SSH keys are required.

---

### Question 8: Chaos Testing on Amazon ECS and Amazon EKS Workloads
**Answer:** A  
**Explanation:** 
AWS FIS natively supports containerized platforms:
- **Amazon ECS:** Actions such as `aws:ecs:task-kill` (terminating specific tasks) and `aws:ecs:drain-container-instances` (simulating spot termination or maintenance on container hosts).
- **Amazon EKS:** Actions such as `aws:eks:inject-kubernetes-custom-resource`, which interfaces with CNCF tools like Chaos Mesh or LitmusChaos to inject pod faults, node network latency, and container crashes directly into Kubernetes clusters.
This enables thorough resilience testing across both managed serverless container tiers (Fargate) and managed Kubernetes clusters.

---

### Question 9: Validating Recovery Objectives (RTO / RPO) via FIS Post-Experiment Evaluation
**Answer:** A  
**Explanation:** 
Validating disaster recovery objectives requires empirical, external measurement:
- **Recovery Time Objective (RTO):** Using automated **CloudWatch Synthetics Canaries** that continuously execute API transactions every 5 seconds provides exact, timestamped measurement of when requests failed and when the system successfully resumed healthy 200 responses. The duration of downtime directly measures RTO.
- **Recovery Point Objective (RPO):** Comparing database transaction log sequences and primary/replica Log Sequence Numbers (LSN) validates whether any committed transactions were lost during failover, proving that RPO = 0 was achieved.
