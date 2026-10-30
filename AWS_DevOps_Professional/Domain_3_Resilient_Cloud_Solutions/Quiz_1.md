# Domain 3: Resilient Cloud Solutions — Knowledge Assessment & Mastery Quiz

**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Weight:** 15% of Exam  
**Format:** 15 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: Auto Scaling Group Instance Refresh with Fleet Availability Guarantees

An enterprise production web application runs across 60 Amazon EC2 instances inside an Auto Scaling Group (ASG) behind an Application Load Balancer. The ASG has `DesiredCapacity: 60`, `MinSize: 40`, and `MaxSize: 90`. A DevOps engineer needs to deploy a newly hardened golden AMI across the running fleet during regular business hours. To maintain service level objectives, production serving capacity must **NEVER DROP BELOW 80% (48 instances)** at any moment during the replacement cycle, and newly launched instances must complete an internal 5-minute health check warm-up before older instances are terminated.

**Which AWS CLI command initiates this rolling update while meeting all constraints?**
- A. `aws autoscaling start-instance-refresh --auto-scaling-group-name prod-asg --preferences '{"MinHealthyPercentage": 80, "InstanceWarmup": 300}'`
- B. `aws autoscaling update-auto-scaling-group --auto-scaling-group-name prod-asg --launch-template LaunchTemplateId=lt-0123,Version='$Latest' --default-cooldown 300`
- C. `aws autoscaling set-desired-capacity --auto-scaling-group-name prod-asg --desired-capacity 90 --honor-cooldown` followed by sequential manual terminations.
- D. `aws autoscaling start-instance-refresh --auto-scaling-group-name prod-asg --preferences '{"MaxHealthyPercentage": 120, "SkipMatching": false}'`

---

### Question 2: S3 Cross-Region Replication with Multi-Region KMS Customer Managed Keys

An organization operates a compliant financial archive where objects uploaded to an Amazon S3 bucket in `us-east-1` (Source) must replicate automatically to an S3 bucket in `eu-central-1` (Destination) for disaster recovery. Both buckets are encrypted using AWS KMS Customer Managed Keys (CMKs). After enabling S3 Cross-Region Replication (CRR), newly uploaded objects appear in the source bucket, but the object replication status displays `FAILED`.

**Which combination of IAM and KMS configurations will resolve the replication failure? (Select TWO)**
- A. In the KMS key policy of the source CMK in `us-east-1`, grant `kms:Decrypt` and `kms:DescribeKey` permissions to the IAM replication service role.
- B. In the KMS key policy of the destination CMK in `eu-central-1`, grant `kms:Encrypt` and `kms:GenerateDataKey*` permissions to the IAM replication service role.
- C. In the S3 bucket policy of the source bucket, grant `s3:ReplicateObject` and `s3:ReplicateTags` to the root principal of the destination account.
- D. Change the encryption configuration on the destination S3 bucket to use the default `aws/s3` AWS-managed KMS key to eliminate cross-region key policy conflicts.
- E. Ensure that S3 Object Lock is enabled in Compliance mode on both the source and destination S3 buckets prior to replication initiation.

---

### Question 3: Route 53 Active-Passive Multi-Region Failover with Composite Health Checks

A global payments company deploys its web tier in an Active-Passive disaster recovery architecture across `us-east-1` (Primary) and `us-west-2` (Standby). The primary environment runs behind an Application Load Balancer. Route 53 must route 100% of public DNS traffic to the primary ALB during normal operations. If the primary application HTTP endpoint returns 5xx errors OR if an Amazon Aurora database connection leak causes the primary database CPU to exceed 90% for 3 minutes, Route 53 must automatically fail over to `us-west-2`.

**How should the DevOps Engineer configure the DNS failover architecture?**
- A. Create a Route 53 **Failover Routing Policy** with a primary record pointing to `us-east-1` and a secondary record pointing to `us-west-2`. Associate the primary record with a Route 53 Health Check that monitors a **CloudWatch Composite Alarm** (combining ALB HTTP 5xx and RDS CPU utilization alarms).
- B. Create a Route 53 Latency-Based Routing policy with equal weights across both regions, associating each regional record with an endpoint health check that queries the local ALB `/healthz` path.
- C. Create Route 53 Weighted records set to 100 in `us-east-1` and 0 in `us-west-2`, and write an AWS Lambda function invoked every 60 seconds by EventBridge to swap DNS weights when database alarms fire.
- D. Deploy an AWS Global Accelerator with endpoint weights set to 128 for `us-east-1` and 0 for `us-west-2`, and configure client-side retry logic to detect primary region connectivity failures.

---

### Question 4: Aurora Global Database Planned Switchover vs. Unplanned Failover

An enterprise operates an Amazon Aurora Global Database spanning `us-east-1` (primary cluster) and `us-west-2` (secondary cluster). The SRE team is scheduling a quarterly disaster recovery operational exercise during a scheduled change window. The drill mandates proving that the secondary region can become the new primary writer with **ZERO DATA LOSS (`RPO = 0`)** and without breaking the underlying storage replication topology.

**Which database operation and procedure satisfies this operational requirement?**
- A. Execute an **Aurora Global Database Managed Planned Failover (`Switchover`)**; it synchronizes secondary storage with primary write transactions, promotes the secondary cluster to primary writer, and automatically reverses replication without rebuilding the cluster.
- B. Execute an **Unplanned Regional Failover (`FailoverGlobalCluster`)** with write forwarding enabled, which promotes the secondary region immediately while keeping the old primary online.
- C. Detach the secondary Aurora cluster from the global database using the RDS console, promote it to a standalone cluster, and configure AWS Database Migration Service (DMS) to replicate data back to `us-east-1`.
- D. Restore the latest automated snapshot of the `us-east-1` DB cluster into `us-west-2` using a new DB cluster identifier, and reconfigure application database connection endpoints.

---

### Question 5: DynamoDB Global Tables Multi-Region Active-Active Conflict Resolution

A ride-sharing platform utilizes Amazon DynamoDB Global Tables across three regions (`us-east-1`, `eu-west-1`, and `ap-southeast-1`) in an Active-Active multi-region deployment. In rare concurrent scenarios, two distinct users in different continents submit simultaneous ride status updates modifying the exact same item within the same millisecond.

**How does Amazon DynamoDB Global Tables resolve this concurrent write conflict across regions?**
- A. DynamoDB uses a **Last-Writer-Wins (LWW)** conflict resolution algorithm: DynamoDB examines the timestamp of each update at the physical storage layer and commits the update with the latest timestamp.
- B. DynamoDB uses a distributed two-phase commit (2PC) protocol: it acquires a global row lock across all regional replicas before acknowledging the write to the client application.
- C. DynamoDB routes conflicting write operations to an Amazon SQS Dead Letter Queue (DLQ) in each region, requiring a reconciliation Lambda function to resolve the conflict.
- D. DynamoDB rejects the concurrent write with a `TransactionCanceledException`, forcing the application to retry the update with an exponential backoff sequence.

---

### Question 6: Application Load Balancer Cross-Zone Load Balancing & Connection Draining

A microservice runs across 20 Amazon EC2 instances distributed across three Availability Zones (`us-east-1a`: 2 instances, `us-east-1b`: 8 instances, `us-east-1c`: 10 instances) behind an Application Load Balancer. The operations team notices that the 2 instances in `us-east-1a` are running at 98% CPU utilization and dropping connections, while instances in `us-east-1c` are idling at 12% CPU. Furthermore, during software updates, in-flight HTTP connections are prematurely dropped when instances are marked for termination.

**Which combination of load balancer configurations resolves both operational issues? (Select TWO)**
- A. Enable **Cross-Zone Load Balancing** on the ALB Target Group to distribute incoming client requests evenly across all registered targets regardless of their Availability Zone.
- B. Configure the Target Group **Deregistration Delay** (connection draining) attribute to an appropriate timeout (e.g., 300 seconds) to allow in-flight HTTP requests to complete before instances terminate.
- C. Modify Route 53 DNS records to use Geoproximity routing with negative bias applied to `us-east-1a` to reduce inbound connection volume.
- D. Increase the ALB idle timeout attribute from 60 seconds to 3600 seconds to keep client keep-alive connections persistently pinned to `us-east-1c`.
- E. Deploy an AWS Network Load Balancer in front of the Application Load Balancer to strip TCP headers and balance packet distribution across subnets.

---

### Question 7: Auto Scaling Target Tracking Policies vs. Step Scaling Policies

A high-volume REST API running on an Auto Scaling Group behind an Application Load Balancer experiences unpredictable, rapid traffic spikes during flash sales. The engineering team needs the ASG to automatically scale out rapidly when request volume surges, and scale in smoothly when traffic subsides, while maintaining an average of approximately 800 requests per target instance.

**Which scaling policy configuration achieves this requirement with the LEAST operational configuration overhead?**
- A. Implement a **Target Tracking Scaling Policy** using the predefined metric `ALBRequestCountPerTarget` configured with a target value of `800` and an appropriate instance warmup duration.
- B. Configure a **Step Scaling Policy** with multiple CloudWatch alarms that increase capacity by 20% when request count exceeds 1,000, and by 50% when request count exceeds 2,500.
- C. Create a **Simple Scaling Policy** attached to a CloudWatch Alarm monitoring `RequestCount` with a 300-second cooldown period.
- D. Configure **Predictive Scaling** with a forecast horizon of 24 hours and a buffer time of 30 minutes, disabling dynamic target tracking.

---

### Question 8: Disaster Recovery Strategy Architectural Trade-offs (RPO vs. RTO vs. Cost)

An enterprise architectural board is evaluating disaster recovery architectures for an ERP system. The business mandates a Recovery Point Objective (RPO) of under 15 minutes, a Recovery Time Objective (RTO) of under 45 minutes, and strictly requires the **LOWEST ONGOING INFRASTRUCTURE COST** during normal operations.

**Which disaster recovery architectural pattern best satisfies all three business constraints?**
- A. Pilot Light: Critical databases are replicated continuously to the disaster recovery region using Aurora Global Database or continuous block replication; core application servers remain unprovisioned until failover, where they are rapidly orchestrated via pre-baked AMIs and CloudFormation StackSets.
- B. Warm Standby: A functional, scaled-down version of the complete production environment (including minimal compute instances, load balancers, and replicated databases) runs 24/7 in the secondary region, scaling up to full capacity on failover.
- C. Multi-Region Active-Active: A full-capacity production deployment runs concurrently across two AWS regions, serving user requests dynamically via Route 53 Latency-Based routing with real-time bidirectional data synchronization.
- D. Backup and Restore: Daily Amazon EBS snapshots and RDS automated backups are copied asynchronously to the secondary region; instances and databases are restored from backup media only after a disaster is declared.

---

### Question 9: High-Availability AWS Storage Gateway Volume Gateway Architecture

An enterprise data center utilizes an on-premises AWS Storage Gateway (Volume Gateway) deployed as a virtual machine on a local VMware vSphere cluster to present iSCSI LUNs backed by Amazon S3. The corporate continuity policy requires that the local Storage Gateway service must survive a physical hypervisor hardware crash with zero data corruption and resume iSCSI operations automatically without manual re-installation.

**How should this high-availability requirement be implemented?**
- A. Deploy the Storage Gateway appliance on a **VMware vSphere High Availability (HA)** cluster and enable the **AWS Storage Gateway High Availability** integration using the Storage Gateway VM heartbeat probe.
- B. Provision two separate Storage Gateway virtual appliances in an active-active cluster pointing to the exact same S3 bucket and configure multipath I/O (MPIO) on the iSCSI initiators.
- C. Configure AWS DataSync to mirror the local cache volume continuously to an Amazon EC2 instance running in the nearest AWS region.
- D. Schedule nightly VMware vCenter snapshots of the Storage Gateway VM and upload the VMDK virtual disk images to Amazon S3 via the AWS CLI.

---

### Question 10: Auto Scaling Termination Policies & Stateful Job Protection

An Auto Scaling Group processes asynchronous, compute-intensive rendering jobs from an Amazon SQS queue using a combination of On-Demand and Spot instances. Individual rendering jobs take between 30 and 90 minutes to complete. During automated scale-in events triggered by low queue depth, the ASG must ensure that instances currently processing active rendering jobs are not terminated prematurely. If non-busy instances are available, scale-in should prioritize terminating instances running older launch template revisions.

**Which combination of Auto Scaling configurations satisfies these requirements? (Select TWO)**
- A. Configure the worker daemon to enable **Instance Scale-In Protection** via the AWS SDK (`SetInstanceProtection`) when a job begins, and remove scale-in protection when the job finishes.
- B. Configure the ASG custom **Termination Policy** order to prioritize `OldestLaunchTemplate` before evaluating `AllocationStrategy` and `Default`.
- C. Set the ASG termination policy exclusively to `NewestInstance` to protect older running instances from being selected for termination.
- D. Configure an ASG `EC2_INSTANCE_LAUNCHING` lifecycle hook that delays instance registration until the SQS queue depth reaches zero.
- E. Set the ASG Default Cooldown period to 5,400 seconds (90 minutes) to suppress all scale-in actions while jobs execute.

---

### Question 11: Sub-Second Multi-Region Failover with Route 53 ARC Routing Controls

A mission-critical financial clearinghouse operates an Active-Active deployment across `us-east-1` and `us-west-2`. During an acute AWS regional infrastructure impairment in `us-east-1`, the incident commander must manually shift 100% of global DNS traffic away from `us-east-1` within 5 seconds, even if the Route 53 control plane API in `us-east-1` is completely unavailable.

**Which architecture satisfies this high-resilience traffic management requirement?**
- A. Configure **Amazon Route 53 Application Recovery Controller (ARC) Routing Controls** hosted across the 5 regional data-plane cluster endpoints, associated with Route 53 DNS failover health checks.
- B. Configure Route 53 Weighted records with a 5-second TTL pointing to the regional ALBs, and update record weights via the Route 53 `ChangeResourceRecordSets` API during failover.
- C. Configure AWS Global Accelerator with regional endpoint groups, and update the traffic dial for `us-east-1` from 100 to 0 via the AWS Management Console during the incident.
- D. Deploy an Amazon CloudFront distribution fronting both regional ALBs and write a CloudFront Function that modifies origin request headers when an incident is declared.

---

### Question 12: Network Load Balancer (NLB) Long-Lived TCP Connection Draining

An IoT telemetry platform ingests MQTT data over persistent TCP connections from 1,000,000 smart sensors. The compute tier runs on Amazon EC2 instances behind a Network Load Balancer (NLB). When an EC2 instance is deregistered for maintenance or rolling updates, existing established TCP connections must remain open for up to 60 minutes to complete active transmissions, while all new incoming connections are directed exclusively to healthy instances.

**Which configuration satisfies this requirement?**
- A. Set the Target Group **Deregistration Delay** attribute to 3600 seconds, and ensure **Connection Termination on Deregistration** is disabled on the target group.
- B. Enable Cross-Zone Load Balancing on the NLB and set the idle timeout attribute to 3600 seconds in the load balancer attributes.
- C. Configure an `AWS::AutoScaling::LifecycleHook` for `EC2_INSTANCE_TERMINATING` with a heartbeat timeout of 3600 seconds and an `ABANDON` default action.
- D. Enable Proxy Protocol v2 on the NLB Target Group and configure the client devices to close TCP sockets after 60 seconds of inactivity.

---

### Question 13: Route 53 ARC Readiness Checks vs. Routing Controls

An enterprise SRE team is designing an automated disaster recovery readiness validation framework for a multi-region active-active microservice platform. The team needs continuous automated auditing to ensure that replica compute capacity, database read replica IOPS, and service quotas in the recovery region are sufficient to absorb 100% of production traffic *before* an emergency traffic shift is executed.

**Which AWS service feature provides this continuous auditing capability?**
- A. Route 53 ARC Readiness Checks: Continuously audits recovery region infrastructure against active region capacity, VPC configurations, and AWS service quotas, publishing a red/green readiness status.
- B. Route 53 ARC Routing Controls: Executes data-plane traffic shifting across regional clusters using simple on/off switches that interact with DNS failover health checks.
- C. AWS Fault Injection Service (FIS): Executes automated fault injection experiments that continuously disrupt availability zones to verify failover thresholds.
- D. AWS Config Conformance Packs: Evaluates CloudFormation templates in source control against CIS Benchmarks and triggers automated SSM remediation runbooks.

---

### Question 14: Amazon Aurora Read Replica Auto Scaling & Replica Lag Governance

An analytics application experiences high-volume, bursty read traffic against an Amazon Aurora PostgreSQL DB cluster. The cluster consists of 1 primary writer and 2 read replicas. During peak morning reporting hours, read query volume quadruples, causing Aurora replica lag to spike from 15 milliseconds to over 18 seconds, serving stale data to users.

**How should the DevOps Engineer configure Auto Scaling to absorb read spikes with the LEAST latency?**
- A. Configure **Aurora Auto Scaling** on the DB cluster targeting the predefined metric `RDSReaderAverageCPUUtilization` (e.g. at 60%) or create a custom target tracking policy on `AuroraReplicaLag` with a target threshold of 2,000 milliseconds.
- B. Implement an AWS Lambda function invoked every minute by EventBridge that calls `rds:CreateDBInstanceReadReplica` when replica lag exceeds 10 seconds.
- C. Convert the Aurora cluster into an Aurora Multi-Master cluster across multiple Availability Zones to allow all nodes to accept read and write traffic.
- D. Deploy an Amazon ElastiCache Redis cluster in front of Aurora, configuring write-through caching in the application tier without Auto Scaling Aurora replicas.

---

### Question 15: Amazon S3 Replication Time Control (RTC) & Cross-Region SLAs

A regulated healthcare platform stores patient diagnostic imaging files in Amazon S3. Compliance mandates require that 99.9% of all newly ingested medical images must be replicated to a geographically distant secondary AWS region within exactly 15 minutes of upload, and the compliance officer must receive automated alerts if replication exceeds this threshold.

**Which S3 feature configuration satisfies these regulatory requirements?**
- A. Enable **S3 Replication Time Control (RTC)** on the S3 Cross-Region Replication rule, and create an Amazon CloudWatch Alarm on the `BytesPendingReplication` and `OperationsPendingReplication` metrics.
- B. Configure standard S3 Cross-Region Replication (CRR) and use AWS DataSync to mirror modified objects every 10 minutes over an encrypted transit gateway connection.
- C. Deploy an Amazon S3 Multi-Region Access Point (MRAP) with active-active routing enabled, and write an AWS Lambda function that copies objects using `s3:CopyObject`.
- D. Configure an S3 Event Notification on `s3:ObjectCreated:*` to publish to an Amazon SQS queue, and use AWS Step Functions to copy objects between buckets.

---

## Answer Key & Detailed Explanations

### Question 1: Auto Scaling Group Instance Refresh with Fleet Availability Guarantees
**Answer:** A  
**Explanation:**  
`aws autoscaling start-instance-refresh` provides native rolling fleet updates for Auto Scaling Groups. Setting `"MinHealthyPercentage": 80` guarantees that at least 80% of current capacity (48 instances out of 60) remains healthy, in-service, and serving traffic throughout the entire refresh operation. Specifying `"InstanceWarmup": 300` ensures the ASG allows 300 seconds (5 minutes) for each newly launched instance to warm up and pass health checks before terminating older instances.

---

### Question 2: S3 Cross-Region Replication with Multi-Region KMS Customer Managed Keys
**Answer:** A, B  
**Explanation:**  
When S3 Cross-Region Replication replicates objects encrypted with Customer Managed Keys (CMKs), the IAM replication service role requires permissions in both regions: 1) It must have `kms:Decrypt` and `kms:DescribeKey` on the source CMK in `us-east-1` to read the ciphertext (Action A); and 2) It must have `kms:Encrypt` and `kms:GenerateDataKey*` on the destination CMK in `eu-central-1` to re-encrypt the object upon arrival (Action B). Default `aws/s3` keys cannot be shared cross-account or cross-region in custom replication topologies.

---

### Question 3: Route 53 Active-Passive Multi-Region Failover with Composite Health Checks
**Answer:** A  
**Explanation:**  
Route 53 Failover Routing supports linking health checks to **CloudWatch Metric Alarms**. By creating a **CloudWatch Composite Alarm** that combines ALB HTTP 5xx error metrics AND Amazon Aurora database CPU utilization metrics into a single boolean state (`ALARM(ALB5xxAlarm) OR ALARM(DatabaseCPUAlarm)`), Route 53 initiates an automatic failover to the standby region whenever either the web tier or the database tier fails, without writing custom polling scripts.

---

### Question 4: Aurora Global Database Planned Switchover vs. Unplanned Failover
**Answer:** A  
**Explanation:**  
For planned drills or maintenance, **Aurora Global Database Managed Planned Failover (`Switchover`)** is the mandatory operational choice. It synchronizes secondary cluster storage with the primary cluster before promoting the secondary to primary writer, guaranteeing zero data loss (`RPO = 0`). Furthermore, it automatically reverses the replication direction, making the old primary cluster a replica of the new primary without breaking the cluster topology or requiring data resynchronization.

---

### Question 5: DynamoDB Global Tables Multi-Region Active-Active Conflict Resolution
**Answer:** A  
**Explanation:**  
Amazon DynamoDB Global Tables uses a **Last-Writer-Wins (LWW)** conflict resolution mechanism. When concurrent writes occur to the same attribute of the same item in different regions, DynamoDB compares the internal timestamp of the write operations at the storage engine level. The write with the latest timestamp is applied across all replicas. DynamoDB does not use distributed locks or two-phase commit protocols across regions.

---

### Question 6: Application Load Balancer Cross-Zone Load Balancing & Connection Draining
**Answer:** A, B  
**Explanation:**  
When EC2 targets are distributed unevenly across Availability Zones, enabling **Cross-Zone Load Balancing** on the Target Group forces the ALB to distribute incoming requests evenly across all targets (5% of traffic per instance across the 20 instances) rather than splitting 33% evenly across each AZ's nodes (Action A). Configuring **Deregistration Delay** (connection draining) ensures that in-flight HTTP requests have up to 300 seconds to finish before the instance is shut down during updates (Action B).

---

### Question 7: Auto Scaling Target Tracking Policies vs. Step Scaling Policies
**Answer:** A  
**Explanation:**  
**Target Tracking Scaling Policies** require the least configuration overhead because AWS automatically creates and manages the underlying CloudWatch alarms and scaling calculations. Using the predefined metric `ALBRequestCountPerTarget` with a target value of `800` allows Auto Scaling to dynamically calculate how many instances to add or remove proportionally to keep the average request count per target near 800, adjusting automatically to sudden load changes.

---

### Question 8: Disaster Recovery Strategy Architectural Trade-offs (RPO vs. RTO vs. Cost)
**Answer:** A  
**Explanation:**  
The **Pilot Light** strategy provides the lowest ongoing cost while satisfying an RPO < 15 mins and RTO < 45 mins. Core data is replicated continuously in real-time (yielding sub-minute RPO), but expensive application and web compute servers are not run continuously in the secondary region. Upon disaster declaration, automated CloudFormation StackSets or ASGs provision compute instances from pre-baked AMIs within 15–30 minutes (meeting the 45-minute RTO) with zero idle compute costs during normal operations.

---

### Question 9: High-Availability AWS Storage Gateway Volume Gateway Architecture
**Answer:** A  
**Explanation:**  
AWS Storage Gateway provides native integration with **VMware vSphere High Availability (HA)**. By deploying the gateway VM on a VMware HA cluster and enabling the Storage Gateway HA integration, the gateway service continuously reports heartbeat probes. If the underlying ESXi hypervisor crashes, VMware HA automatically restarts the Storage Gateway VM on a surviving host within 60 seconds, preserving iSCSI initiators and cached data without data loss.

---

### Question 10: Auto Scaling Termination Policies & Stateful Job Protection
**Answer:** A, B  
**Explanation:**  
To protect active rendering jobs, worker nodes must dynamically enable **Instance Scale-In Protection** via `SetInstanceProtection` when beginning a job, and remove it upon completion (Action A). When the ASG evaluates unprotected instances during scale-in, setting a custom **Termination Policy** that evaluates `OldestLaunchTemplate` first ensures that instances running outdated configurations are terminated before instances running current revisions (Action B).

---

### Question 11: Sub-Second Multi-Region Failover with Route 53 ARC Routing Controls
**Answer:** A  
**Explanation:**  
**Amazon Route 53 Application Recovery Controller (ARC) Routing Controls** provide simple, highly reliable on/off switches deployed across 5 regional data-plane cluster endpoints. Because they execute purely on the data plane with a 5-cluster quorum, they are 100% immune to regional AWS control plane outages (such as disruptions to the Route 53 management console or APIs). Flipping an ARC routing control instantly changes Route 53 DNS failover health checks, redirecting 100% of traffic within seconds.

---

### Question 12: Network Load Balancer (NLB) Long-Lived TCP Connection Draining
**Answer:** A  
**Explanation:**  
For Network Load Balancers, setting the Target Group **Deregistration Delay** to 3600 seconds instructs the NLB to stop sending new connections to deregistering targets while allowing existing connections to remain active for up to 1 hour. Ensuring that **Connection Termination on Deregistration** is disabled guarantees that the NLB will not send TCP RST packets or forcibly close established TCP sockets when the deregistration process begins.

---

### Question 13: Route 53 ARC Readiness Checks vs. Routing Controls
**Answer:** A  
**Explanation:**  
**Route 53 ARC Readiness Checks** continuously monitor AWS resource configurations, capacity allocations, and service quotas in secondary recovery regions, comparing them against the primary active region. Readiness checks ensure that the recovery environment is physically prepared to handle 100% of production traffic before an operator executes a traffic shift, alerting teams to capacity drift or missing quotas ahead of time.

---

### Question 14: Amazon Aurora Read Replica Auto Scaling & Replica Lag Governance
**Answer:** A  
**Explanation:**  
Amazon Aurora supports **Aurora Auto Scaling** for read replicas. By configuring an Aurora Auto Scaling policy based on target tracking (either targeting average CPU utilization or custom metric math monitoring `AuroraReplicaLag`), Aurora automatically provisions additional read replicas dynamically when lag or query load rises, distributing read traffic across up to 15 replicas and reducing lag back to acceptable thresholds.

---

### Question 15: Amazon S3 Replication Time Control (RTC) & Cross-Region SLAs
**Answer:** A  
**Explanation:**  
**Amazon S3 Replication Time Control (RTC)** is designed specifically to meet compliance and regulatory requirements for data replication. S3 RTC provides a formal service level agreement (SLA) guaranteeing that 99.9% of newly uploaded objects are replicated to the destination region within 15 minutes. It emits CloudWatch metrics (`BytesPendingReplication`, `OperationsPendingReplication`, and `ReplicationLatency`) that allow automated alerting if replication thresholds are approached.
