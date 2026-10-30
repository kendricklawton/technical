# Domain 3 (Resilient Cloud Solutions) — Quiz 4: Disaster Recovery Strategies & Routing Control

**Topic:** DR Patterns (Backup & Restore, Pilot Light, Warm Standby, Multi-Site Active-Active), RTO/RPO Tradeoffs & AWS DRS  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: Disaster Recovery Tiers: RTO / RPO Tradeoffs and Cost Optimization

An enterprise financial core system mandates a Recovery Point Objective (RPO) of $\le 5\text{ minutes}$ and a Recovery Time Objective (RTO) of $\le 15\text{ minutes}$ for regional disasters. To minimize ongoing operational infrastructure expenses, the executive leadership team mandates that production compute fleets (EC2/EKS) must NOT run continuously in the standby disaster recovery region during normal operations.

**Which disaster recovery strategy fulfills these exact RPO/RTO constraints at the MINIMAL ongoing operational cost?**
- A. Pilot Light: Continuously replicate database transactions to the secondary region (e.g., Aurora Global Database storage replication with sub-second lag), keep supporting network/IAM topology pre-provisioned, and use automated CloudFormation templates to rapidly scale compute fleets upon failover.
- B. Backup and Restore: Take hourly EBS snapshots and RDS automated backups, export them cross-region via AWS Backup, and restore complete environments when a disaster declaration occurs.
- C. Warm Standby: Maintain a scaled-down fleet of EC2 instances and minimal container pods running 24/7 in the secondary region behind an active load balancer, scaling out instance counts during failover.
- D. Multi-Site Active-Active: Run identical full-scale compute and storage infrastructure concurrently across both `us-east-1` and `us-west-2` using DynamoDB Global Tables and AWS Global Accelerator.

---

### Question 2: AWS Elastic Disaster Recovery (AWS DRS) Replication Mechanics

A regulated healthcare organization migrates critical on-premises Windows and Linux database workloads to AWS. The compliance framework requires continuous block-level data replication into an AWS recovery region with near-zero RPO and automated non-disruptive disaster recovery testing drills.

**How does AWS Elastic Disaster Recovery (AWS DRS) maintain continuous data synchronization with MINIMAL infrastructure overhead?**
- A. AWS DRS installs an agent on source servers that intercepts block-level storage I/O and replicates data continuously over TLS into low-cost EBS volumes managed by lightweight Replication Servers in a dedicated Staging Area subnet.
- B. AWS DRS creates hourly Amazon Machine Images (AMIs) of source servers, copies the AMIs cross-region via AWS Backup, and maintains idle `c5.large` instances ready to mount them.
- C. AWS DRS captures database transaction logs via AWS Database Migration Service (DMS) change data capture (CDC) and replays them onto an active Aurora replica.
- D. AWS DRS utilizes hypervisor-level snapshot exports scheduled every 15 minutes, pushing raw VMDK files directly into an Amazon S3 Glacier Flexible Retrieval bucket.

---

### Question 3: Amazon S3 Replication Time Control (RTC) Compliance SLA

A legal compliance archive mandates that 99.9% of all new contract documents uploaded to an Amazon S3 bucket in `us-east-1` must be replicated to a secondary bucket in `eu-west-1` within 15 minutes of ingestion. If replication lag exceeds 15 minutes, automated alerts must notify the compliance engineering team immediately.

**Which Amazon S3 architectural feature satisfies this SLA and monitoring requirement?**
- A. Enable S3 Cross-Region Replication (CRR) with **S3 Replication Time Control (RTC)**, and configure CloudWatch Alarms on the `ReplicationLatency` and `BytesPendingReplication` metrics.
- B. Configure S3 Event Notifications to trigger an AWS Lambda function on `s3:ObjectCreated:*` that copies objects to `eu-west-1` using the `aws s3 cp` CLI command.
- C. Create an AWS DataSync task scheduled to run every 15 minutes between the primary and secondary S3 buckets, monitoring CloudWatch DataSync task execution status.
- D. Enable S3 Multi-Region Access Points (MRAP) with active-active synchronization, and monitor the `MRAPRequestLatency` CloudWatch metric.

---

### Question 4: Aurora Global Database Managed Planned Failover vs Unplanned Detachment

An enterprise runs a mission-critical e-commerce application on Amazon Aurora Global Database with the primary cluster in `us-east-1` and a read-only secondary cluster in `us-west-2`. The DevOps team is executing a planned DR drill to test application operations in `us-west-2`. The business mandates ZERO data loss during the exercise and seamless reversal back to `us-east-1` after the test.

**What is the correct operational procedure to execute this planned failover?**
- A. Initiate a **Managed Planned Failover** via the Aurora console or AWS CLI; Aurora synchronizes the secondary cluster storage with the primary cluster, promotes the secondary to primary with zero data loss, and demotes the old primary to a read-only secondary replica.
- B. Detach the secondary cluster from the Aurora Global Database, modify its writer endpoint configuration, and manually initiate snapshot restoration in `us-east-1` post-drill.
- C. Reboot the primary DB cluster instance with failover enabled; Aurora automatically redirects all global DNS write endpoints to `us-west-2`.
- D. Execute an unplanned failover by promoting the `us-west-2` cluster; Aurora breaks global replication, requiring a complete redeployment of the global database from scratch to reverse the roles.

---

### Question 5: Eliminating EBS Initialization Latency with Fast Snapshot Restore (FSR)

A high-frequency trading platform utilizes an automated Disaster Recovery pipeline that restores multi-terabyte Amazon EBS volumes from snapshots in a secondary region during DR events. SREs observe that while volumes become available in seconds, application read latency spikes up to 400ms for several hours post-recovery because storage blocks must be pulled lazily from Amazon S3.

**Which EBS configuration eliminates this first-touch performance penalty upon volume restoration?**
- A. Enable **Amazon EBS Fast Snapshot Restore (FSR)** on the specific snapshot in the destination Availability Zone, ensuring that newly created volumes are immediately fully initialized with zero first-touch I/O latency.
- B. Run an automated bash script that executes `dd` across all restored block devices to read every block sequentially into memory immediately after volume attachment.
- C. Convert the EBS volumes from `gp3` to Provisioned IOPS SSD (`io2 Block Express`) with 64,000 IOPS before creating the snapshots.
- D. Attach an Amazon Elastic File System (EFS) cache layer between the EC2 instances and the restored EBS volumes using AWS DataSync.

---

### Question 6: AWS Backup Cross-Region & Cross-Account Vault Locking

A multinational enterprise must protect all production EBS snapshots, RDS automated backups, and DynamoDB tables against catastrophic accidental deletion or malicious ransomware attacks. Corporate governance mandates that backup copies must be stored in a separate, isolated AWS account in a secondary region with an immutable WORM (Write Once, Read Many) policy that cannot be deleted or altered by any IAM principal, including the root user.

**Which architecture fulfills these security and compliance requirements?**
- A. Configure AWS Backup with cross-account, cross-region backup copy rules pushing snapshots to an isolated Disaster Recovery AWS account; apply an **AWS Backup Vault Lock** in **Compliance Mode** to the destination vault with a mandatory retention window.
- B. Author an AWS Lambda function that copies snapshots cross-account, encrypts them using an AWS-managed KMS key, and denies `kms:Decrypt` via Service Control Policies.
- C. Apply S3 Object Lock in Governance Mode on the default AWS Backup S3 export bucket and configure IAM MFA Delete on the root account.
- D. Enable AWS Config rule `backup-recovery-point-minimum-retention-check` with an automated remediation runbook that re-creates deleted snapshots.

---

### Question 7: Route 53 Application Recovery Controller Zonal Autoshift

An enterprise hosts a high-availability microservice fleet deployed across three Availability Zones (`us-east-1a`, `us-east-1b`, `us-east-1c`) behind an Application Load Balancer. AWS publishes an internal infrastructure advisory regarding intermittent packet drops affecting one specific Availability Zone (`us-east-1a`). The application requires immediate, automated evacuation of traffic from the impaired zone without manual operator intervention.

**Which AWS Route 53 feature automatically steers traffic away from an impaired Availability Zone based on AWS telemetry?**
- A. Enable **Amazon Route 53 Application Recovery Controller (ARC) Zonal Autoshift** on the Application Load Balancer, allowing AWS to automatically redirect traffic away from the impaired Availability Zone upon detecting potential internal infrastructure issues.
- B. Configure Route 53 Latency-Based routing records pointing to individual private IP addresses of instances in each Availability Zone.
- C. Configure ALB Target Group health checks with `UnhealthyThresholdCount: 1` and set the check interval to 5 seconds.
- D. Write an Amazon EventBridge rule listening for AWS Health events that triggers an AWS Lambda function to update the ALB subnet configuration.

---

### Question 8: Orchestrating Multi-Tier Failover with AWS Step Functions and AWS Systems Manager

During an emergency disaster recovery failover to a secondary region, multiple dependent operational steps must execute in precise chronological sequence: promote Aurora database replica $\rightarrow$ update SSM parameter endpoints $\rightarrow$ deploy CloudFormation compute stacks $\rightarrow$ flip Route 53 routing controls. If any individual step fails or times out, the failover process must halt and notify the incident commander with full execution diagnostics.

**Which orchestration engine provides this reliable, stateful execution pipeline with native error handling?**
- A. AWS Step Functions: Define a state machine that orchestrates AWS SDK service integrations directly (or via SSM Automation runbooks), implementing native `Retry`, `Catch`, and timeout configurations with visual execution history.
- B. A shell script running on a bastion EC2 instance in the primary region using the AWS CLI in a loop with `wait` commands.
- C. An Amazon EventBridge rule that fires multiple asynchronous targets concurrently upon receiving a disaster declaration event.
- D. An AWS CodePipeline pipeline containing sequential CodeBuild build projects executing custom python deployment scripts.

---

### Question 9: Cross-Region Multi-Site Active-Active Data Synchronization via DynamoDB Global Tables

A global reservation platform requires a multi-region active-active deployment across `us-east-1` and `eu-west-1` with sub-second read and write access for local users in both geographies. In the rare event that concurrent writes occur on the same item in both regions simultaneously, the conflict must be resolved deterministically without application errors.

**How does Amazon DynamoDB Global Tables handle concurrent write conflicts across regions?**
- A. DynamoDB Global Tables uses a **Last-Writer-Wins** conflict resolution model, comparing the client-side or system-level timestamps of conflicting writes and applying the update with the latest timestamp across all replica tables.
- B. DynamoDB Global Tables acquires distributed locks across all replica regions via Paxos before committing write transactions to any single regional table.
- C. DynamoDB Global Tables generates an error response (`TransactionConflictException`) and forces both clients to retry the write operation.
- D. DynamoDB Global Tables drops both writes and streams the conflicting records to an Amazon Kinesis Data Stream for manual administrative reconciliation.

---

## Answer Key & Explanations

### Question 1: Disaster Recovery Tiers: RTO / RPO Tradeoffs and Cost Optimization
**Answer:** A  
**Explanation:** 
AWS Disaster Recovery strategies span four classic tiers:
1. **Backup and Restore:** Lowest cost, highest RTO/RPO (hours to days). Backups must be restored and environments re-provisioned from scratch.
2. **Pilot Light:** Core data is continuously replicated to the secondary region (e.g., Aurora Global Database storage replication maintaining near-zero RPO $\le 1\text{s}$), and foundational infrastructure (VPCs, IAM, security groups) is pre-provisioned. Compute fleets (EC2/EKS) are **not running** in the standby region, keeping idle costs minimal. Upon disaster declaration, automated IaC templates scale compute fleets up, meeting RTOs of 10–15 minutes.
3. **Warm Standby:** A scaled-down but live functional version of the entire environment runs continuously in the standby region (higher ongoing cost; RTO minutes).
4. **Multi-Site Active-Active:** Full infrastructure runs concurrently in multiple regions handling live production traffic (highest cost, near-zero RTO/RPO).

The prompt explicitly mandates RPO $\le 5\text{m}$, RTO $\le 15\text{m}$, and **zero running compute fleets in the standby region** to minimize operational expenses, making **Pilot Light** the exact matching pattern.

---

### Question 2: AWS Elastic Disaster Recovery (AWS DRS) Replication Mechanics
**Answer:** A  
**Explanation:** 
**AWS Elastic Disaster Recovery (AWS DRS)** minimizes disaster recovery operational costs while providing near-zero RTO and RPO:
- An AWS DRS Agent installed on the source server captures continuous, block-level storage writes and replicates them over TLS directly into the AWS target region.
- In AWS, lightweight **Replication Servers** receive the block data and write it directly to low-cost staging EBS volumes.
- Production-grade compute instances (EC2) are **not launched during replication**. They are launched only when executing a DR drill or initiating a failover.
- This architecture enables non-disruptive disaster recovery testing without interrupting ongoing data synchronization or impacting source server performance.

---

### Question 3: Amazon S3 Replication Time Control (RTC) Compliance SLA
**Answer:** A  
**Explanation:** 
**Amazon S3 Replication Time Control (S3 RTC)** is an enterprise compliance feature designed specifically for predictable replication performance:
- S3 RTC provides an SLA guaranteeing that **99.9% of new objects are replicated within 15 minutes** of upload.
- RTC automatically exposes high-resolution CloudWatch metrics: `ReplicationLatency` (tracking maximum replication lag in seconds), `BytesPendingReplication`, and `OperationsPendingReplication`.
- Attaching a CloudWatch Alarm to `ReplicationLatency` with a threshold $> 900\text{ seconds}$ notifies compliance engineers immediately if SLA violations occur.
- Standard CRR without RTC offers only best-effort replication without timing guarantees.

---

### Question 4: Aurora Global Database Managed Planned Failover vs Unplanned Detachment
**Answer:** A  
**Explanation:** 
Amazon Aurora Global Database provides two distinct failover mechanisms:
1. **Managed Planned Failover:** Designed for planned maintenance, testing, and DR drills. Aurora pauses write activity on the primary cluster, ensures all pending write transactions are synchronized to the target secondary cluster (guaranteeing **zero data loss** / RPO = 0), promotes the secondary cluster to primary, and converts the old primary cluster into a read-only secondary replica. The global database topology remains completely intact.
2. **Unplanned Failover (Detachment):** Used during catastrophic, unannounced regional outages where the primary region is unreachable. This operation breaks the global database replication topology, promotes the secondary cluster to an independent regional cluster, and requires manual reconfiguration to re-establish global replication once the failed region recovers.

---

### Question 5: Eliminating EBS Initialization Latency with Fast Snapshot Restore (FSR)
**Answer:** A  
**Explanation:** 
When an Amazon EBS volume is created from an EBS snapshot, the volume becomes immediately available, but data blocks are pulled lazily from Amazon S3 upon first access. This "first touch" causes significant I/O latency spikes (up to hundreds of milliseconds per read) until every block is read at least once.

**Amazon EBS Fast Snapshot Restore (FSR)** completely eliminates this first-touch penalty. When FSR is enabled on a snapshot in an Availability Zone, AWS pre-populates storage blocks onto the underlying storage media. Volumes created from an FSR-enabled snapshot deliver their full provisioned performance (IOPS and throughput) immediately upon creation with zero initialization delay.

---

### Question 6: AWS Backup Cross-Region & Cross-Account Vault Locking
**Answer:** A  
**Explanation:** 
To protect against ransomware and malicious administrative actions:
- **AWS Backup Cross-Account Copy:** Backups are copied across accounts into a dedicated, isolated Disaster Recovery account where operational IAM principals have no access.
- **AWS Backup Vault Lock:** Enforces WORM (Write Once, Read Many) compliance. In **Compliance Mode**, once the grace period expires, the lock is permanent. **No one**, including the AWS account root user or AWS Support, can delete or alter the vault lock configuration, decrease the retention period, or delete recovery points before their expiration dates.
- Option C (S3 Object Lock) applies only to raw S3 objects, not native EBS snapshots or RDS database automated recovery points.

---

### Question 7: Route 53 Application Recovery Controller Zonal Autoshift
**Answer:** A  
**Explanation:** 
**Route 53 ARC Zonal Autoshift** is an automated resilience capability:
- It monitors AWS internal infrastructure telemetry for potential impairments within specific Availability Zones (e.g., degraded network connectivity, power issues).
- When AWS detects a potential impairment affecting an AZ, Zonal Autoshift automatically redirects traffic away from the affected AZ for supported resources (such as Application Load Balancers and Network Load Balancers) without requiring manual customer intervention or custom synthetic canaries.
- Once the AZ recovers, traffic is gradually restored.

---

### Question 8: Orchestrating Multi-Tier Failover with AWS Step Functions and AWS Systems Manager
**Answer:** A  
**Explanation:** 
**AWS Step Functions** provides stateful, visual, and highly reliable workflow orchestration:
- Failover pipelines require strict sequential dependencies: database promotion must succeed before updating application configuration endpoints, which must succeed before scaling compute and flipping DNS routing controls.
- Step Functions features native AWS SDK service integrations (e.g., `rds:failoverGlobalCluster`, `ssm:putParameter`, `route53:updateRoutingControlStates`).
- It provides built-in `Retry` logic with exponential backoff, `Catch` handlers for operational alerting, execution timeouts, and an immutable visual audit trail of every state transition.

---

### Question 9: Cross-Region Multi-Site Active-Active Data Synchronization via DynamoDB Global Tables
**Answer:** A  
**Explanation:** 
Amazon DynamoDB Global Tables provides fully managed multi-region active-active data replication:
- In Global Tables, writes to any replica table are asynchronously propagated to all other replica tables across AWS Regions (typically within 1–2 seconds).
- In the event of concurrent conflicting writes occurring simultaneously on the same item in two different regions, DynamoDB resolves the conflict using a **Last-Writer-Wins** strategy.
- DynamoDB compares the timestamps of the conflicting writes and applies the write with the most recent timestamp to all replicas.
- DynamoDB does not use distributed multi-region locking (Option B), as that would introduce severe cross-region latency penalties on every write.
