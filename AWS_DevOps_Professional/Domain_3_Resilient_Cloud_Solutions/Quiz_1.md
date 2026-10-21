# Domain 3: Resilient Cloud Solutions — Knowledge Assessment & Mastery Quiz

**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Weight:** 15% of Exam  
**Format:** 15 Deep Scenario-Based Professional Practice Questions with Detailed Explanations

---

### Question 1: Auto Scaling Group Instance Refresh with Minimum Healthy Percentage

A company maintains a fleet of 50 Amazon EC2 instances inside an Auto Scaling Group behind an Application Load Balancer. The ASG has `DesiredCapacity: 50`, `MinSize: 30`, and `MaxSize: 80`. The DevOps Engineer needs to roll out an updated AMI across the entire fleet during business hours. To maintain application throughput, at least 80% of current capacity (40 instances) must remain healthy and in service at all times during the rolling replacement.

**Which AWS Auto Scaling CLI command initiates this update safely?**
- A. `aws autoscaling update-auto-scaling-group --auto-scaling-group-name prod-asg --launch-template Version=$Latest`
- B. `aws autoscaling start-instance-refresh --auto-scaling-group-name prod-asg --preferences '{"MinHealthyPercentage": 80, "InstanceWarmup": 300}'`
- C. Terminate all instances manually and let the ASG launch replacements.
- D. `aws autoscaling set-desired-capacity --auto-scaling-group-name prod-asg --desired-capacity 80`

---

### Question 2: S3 Cross-Region Replication with KMS Customer Managed Keys

An enterprise operates a backup architecture where critical document objects uploaded to an Amazon S3 bucket in `us-east-1` (Source) must be automatically replicated to a secondary S3 bucket in `eu-west-1` (Destination) for disaster recovery. Both buckets are encrypted using AWS KMS Customer Managed Keys (CMKs). 

After configuring the S3 Replication Rule, new objects are uploaded to the source bucket, but the replication status shows `FAILED`.

**What is the most likely cause of the replication failure? (Select TWO)**
- A. S3 Cross-Region Replication does not support objects encrypted with KMS Customer Managed Keys.
- B. The IAM replication role does not have `kms:Decrypt` permissions on the source CMK in `us-east-1`.
- C. The IAM replication role does not have `kms:Encrypt` permissions on the destination CMK in `eu-west-1`.
- D. Versioning is enabled on the source bucket but disabled on the destination bucket.
- E. The destination bucket must use the exact same KMS key ARN as the source bucket.

---

### Question 3: Route 53 Active-Passive Failover with CloudWatch Metric Alarms

A company deploys its web application in an Active-Passive disaster recovery architecture. The primary environment runs on Amazon EC2 instances in `us-east-1`, and the secondary standby environment runs in `us-west-2`. The company wants Amazon Route 53 to automatically route users to `us-west-2` if the primary application endpoint fails to respond or if the primary application's database connection pool is exhausted.

**How should the DevOps Engineer configure the DNS failover routing?**
- A. Configure a Simple Routing record with two IP addresses.
- B. Create an Amazon Route 53 **Failover Routing Policy**. Create a primary record pointing to `us-east-1` associated with a Route 53 Health Check that monitors a CloudWatch Composite Metric Alarm (tracking HTTP health and DB connection exhaustion). Create a secondary record pointing to `us-west-2`.
- C. Create a Latency-Based Routing policy with equal latency weights.
- D. Use Route 53 Geolocation routing to route European users to `us-west-2`.

---

### Question 4: Aurora Global Database Failover vs. Switchover

An e-commerce platform uses an Amazon Aurora Global Database spanning `us-east-1` (primary) and `us-west-2` (secondary). The platform team plans a disaster recovery drill to test regional failover with zero data loss. During normal operations, replication latency between regions is under 1 second.

**What is the difference between an Aurora Global Database Managed Switchover and an Unplanned Failover, and which should be used for the drill?**
- A. Managed Switchover should be used for planned maintenance; it synchronizes database storage before the transition, guarantees zero data loss (RPO = 0), and reverses the replication direction automatically.
- B. Unplanned Failover should be used because it is faster than switchover.
- C. Switchover tears down the secondary cluster and rebuilds it as a primary cluster.
- D. There is no difference; both operations promote the secondary cluster with potential data loss.

---

### Question 5: DynamoDB Global Tables Multi-Region Active-Active Conflict Resolution

A global ride-sharing application uses Amazon DynamoDB Global Tables across `us-east-1`, `eu-west-1`, and `ap-southeast-1` to provide low-latency writes to local users. In rare scenarios, concurrent updates are made to the same item in two different AWS regions at the same millisecond.

**How does DynamoDB Global Tables resolve concurrent write conflicts across regions?**
- A. DynamoDB uses two-phase commit (2PC) and locks the item globally across all regions before writing.
- B. DynamoDB uses a **Last-Writer-Wins** conflict resolution mechanism based on the timestamp of the write operation.
- C. DynamoDB places conflicting updates into an Amazon SQS Dead Letter Queue for manual resolution.
- D. DynamoDB rejects the second write with a `ConcurrentTransactionException`.

---

### Question 6: Elastic Load Balancer Cross-Zone Load Balancing and Connection Draining

A web service runs on Amazon EC2 instances distributed unevenly across three Availability Zones (`us-east-1a`: 2 instances, `us-east-1b`: 8 instances, `us-east-1c`: 10 instances). Traffic arrives through an Application Load Balancer. The operations team notices that the instances in `us-east-1a` are experiencing 95% CPU utilization while instances in `us-east-1c` are at 15% CPU.

**What is the root cause of this imbalance, and how should it be rectified?**
- A. The ALB DNS is caching requests to a single IP address. Use Route 53 Multi-Value routing.
- B. **Cross-Zone Load Balancing** is disabled on the Target Group. Enable Cross-Zone Load Balancing so the load balancer distributes traffic evenly across all registered targets regardless of which Availability Zone they reside in.
- C. The instances in `us-east-1a` are smaller instance types.
- D. Auto Scaling is failing to launch instances in `us-east-1c`.

---

### Question 7: Auto Scaling Target Tracking with ALB Request Count Per Target

An API backend running on an Auto Scaling Group behind an Application Load Balancer experiences sudden traffic changes. The team decides to implement an Auto Scaling policy that automatically scales the number of EC2 instances up and down to maintain an average of 1,000 HTTP requests per target instance.

**Which scaling policy should be implemented with the LEAST complexity?**
- A. Simple Scaling policy with two CloudWatch alarms for request rate.
- B. Step Scaling policy using metric math in CloudWatch.
- C. **Target Tracking Scaling Policy** using the predefined metric `ALBRequestCountPerTarget` with a target value of `1000`.
- D. Scheduled Scaling policy scaling up every morning at 08:00 UTC.

---

### Question 8: Disaster Recovery Strategy Comparison (RPO / RTO Trade-offs)

A healthcare compliance audit requires a company to document and test its disaster recovery strategy. The business requires a Recovery Point Objective (RPO) of under 15 minutes and a Recovery Time Objective (RTO) of under 30 minutes, while keeping idle disaster recovery infrastructure costs as low as possible.

**Which disaster recovery strategy meets these requirements at the LOWEST ongoing cost?**
- A. Backup and Restore (Daily S3 glacier backups)
- B. **Pilot Light** (Core data replicated in real-time to secondary region; minimal infrastructure running; application servers provisioned via CloudFormation/AMI on failover)
- C. **Warm Standby** (Scaled-down version of full production environment running 24/7 in secondary region)
- D. **Multi-Region Active-Active** (Full production infrastructure running in both regions with Route 53 latency routing)

---

### Question 9: High-Availability AWS Storage Gateway Architecture

An enterprise uses an on-premises AWS Storage Gateway Volume Gateway to present iSCSI block storage to local VMware hosts. The underlying storage is backed by Amazon S3. The company needs to ensure that the on-premises Storage Gateway VM can survive a local hypervisor host failure without manual reconfiguration.

**How can high availability be implemented for the on-premises Storage Gateway?**
- A. Deploy two independent Storage Gateway virtual appliances pointing to the same S3 bucket.
- B. Deploy the Storage Gateway appliance on **VMware vSphere High Availability (HA)** with the AWS Storage Gateway High Availability integration enabled.
- C. Use AWS DataSync to mirror the local cache to an EC2 instance.
- D. Storage Gateway automatically fails over over the public internet.

---

### Question 10: ASG Termination Policies and Scale-In Protection

An organization operates an Auto Scaling Group containing a mixture of On-Demand and Spot instances across multiple Availability Zones. During an automated scale-in event, the ASG must prioritize:
1. Retaining instances that are running critical, long-running batch jobs.
2. Balancing instances across Availability Zones.
3. Terminating instances running the oldest launch template version.

**Which combination of ASG configurations satisfies these requirements?**
- A. Enable **Instance Scale-In Protection** on the instances processing long-running jobs. Set the ASG **Termination Policy** to `AllocationStrategy` > `OldestLaunchTemplate` > `Default`.
- B. Set the ASG termination policy to `NewestInstance`.
- C. Terminate instances manually via the AWS CLI.
- D. Remove the instances from the ASG and place them in a standalone security group.

---

### Question 11: Route 53 Application Recovery Controller (ARC) Routing Controls

A global enterprise with an Active-Active deployment across `us-east-1` and `us-west-2` needs to manually shift 100% of traffic away from `us-east-1` within 5 seconds during a service degradation, without waiting for DNS TTL caches to expire across the internet.

**Which AWS service architecture satisfies this ultra-fast traffic shifting requirement?**
- A. Route 53 Weighted records with a 60-second TTL.
- B. **Amazon Route 53 Application Recovery Controller (ARC)** Routing Controls integrated with Route 53 DNS failover records.
- C. AWS Global Accelerator with custom traffic dial updates.
- D. Modifying the security group on the load balancer to block port 443.

---

### Question 12: Network Load Balancer (NLB) TCP Connection Resilience

An IoT platform ingests telemetry from 500,000 connected smart meters over persistent TCP connections. The backend instances run on Amazon EC2 inside an Auto Scaling Group behind a Network Load Balancer (NLB). The platform team needs to ensure that when backend instances are replaced, existing established TCP connections are preserved until the client cleanly closes them, up to a maximum of 2 hours.

**Which configuration parameter on the NLB Target Group controls this behavior?**
- A. Enable Cross-Zone Load Balancing on the NLB.
- B. Configure the Target Group **Deregistration Delay** to `7200` seconds (2 hours) and enable **TCP termination delay**.
- C. Set the ALB idle timeout to 7200 seconds.
- D. Configure Route 53 Latency routing with persistent client cookies.

---

### Question 13: ElastiCache Redis Cluster Multi-AZ Failover

A financial application uses Amazon ElastiCache for Redis as a distributed session and rate-limiting store. To survive an Availability Zone failure with minimal disruption, the cluster must support automatic failover to a read replica within 60 seconds with zero manual DNS changes.

**Which ElastiCache configuration satisfies these requirements?**
- A. Deploy a single-node Redis cluster and configure nightly automated snapshots.
- B. Deploy an ElastiCache Redis cluster in **Cluster Mode Disabled** (or Enabled) with **Multi-AZ enabled** and automatic failover configured across at least two Availability Zones. Configure the application to connect using the **Primary Endpoint** (or Configuration Endpoint).
- C. Deploy two independent Redis clusters and sync them using AWS DataSync.
- D. Deploy Memcached with multi-threading enabled.

---

### Question 14: Amazon EFS Mount Target High Availability

A containerized application running on Amazon ECS on EC2 requires shared persistent storage across multiple tasks. Tasks can be scheduled across any of three Availability Zones in the VPC. The shared storage must provide continuous availability even if one Availability Zone suffers a network partition.

**How should Amazon Elastic File System (Amazon EFS) be provisioned?**
- A. Provision an Amazon EFS file system with **EFS One Zone** storage class in `us-east-1a`.
- B. Provision an Amazon EFS file system with **Standard Storage Class** and create an **EFS Mount Target in each Availability Zone** within the VPC subnets where ECS container instances run.
- C. Provision an Amazon EBS Multi-Attach volume and attach it to all ECS hosts.
- D. Use Amazon S3 mounted via `s3fs-fuse`.

---

### Question 15: S3 Multi-Region Access Points (MRAP) with Failover Controls

A multinational enterprise serves static assets and media files to clients globally from Amazon S3 buckets located in `us-east-1` and `eu-central-1`. The company wants to route client uploads and downloads through the AWS Global Accelerator private network to the closest healthy bucket automatically. If one bucket is degraded, traffic must seamlessly shift to the other bucket.

**What S3 feature provides this capability?**
- A. Amazon CloudFront with single origin S3.
- B. **Amazon S3 Multi-Region Access Points (MRAP)** with Route 53 failover controls.
- C. S3 Transfer Acceleration enabled on both buckets independently.
- D. Amazon S3 Batch Operations.

---

## Answer Key & Detailed Explanations

### Question 1: Auto Scaling Group Instance Refresh with Minimum Healthy Percentage

**Answer:** B  
**Explanation:** 
AWS Auto Scaling **Instance Refresh** provides native, automated rolling replacement of EC2 instances in an ASG when a Launch Template or AMI is updated. Specifying `MinHealthyPercentage: 80` ensures that Auto Scaling never allows healthy instances to drop below 80% of desired capacity (maintaining at least 40 healthy instances throughout the process). `InstanceWarmup: 300` ensures that new instances are allowed 5 minutes to warm up and pass health checks before the next batch is replaced.

---

### Question 2: S3 Cross-Region Replication with KMS Customer Managed Keys

**Answer:** B, C  
**Explanation:** 
When using S3 Cross-Region Replication (CRR) with KMS CMKs:
1. KMS keys are regional; a key in `us-east-1` cannot be used in `eu-west-1`. Each region requires its own KMS CMK.
2. The IAM Replication Role used by S3 requires explicit permissions to decrypt objects from the source bucket (`kms:Decrypt` on source key) AND permissions to encrypt objects into the destination bucket (`kms:Encrypt` on destination key).
3. Both source and destination buckets must have S3 Versioning enabled (if versioning was disabled, S3 wouldn't allow configuring replication in the first place).

---

### Question 3: Route 53 Active-Passive Failover with CloudWatch Metric Alarms

**Answer:** B  
**Explanation:** 
Route 53 **Failover Routing** is specifically designed for Active-Passive disaster recovery. The primary record routes 100% of traffic to the active site as long as its associated Route 53 Health Check passes. Because database connection pool exhaustion cannot be detected by external HTTP ping checks alone, creating a CloudWatch Metric Alarm (or Composite Alarm) that monitors database health and linking it to a Route 53 Health Check allows Route 53 to trigger automated failover when deep application-tier metrics degrade.

---

### Question 4: Aurora Global Database Failover vs. Switchover

**Answer:** A  
**Explanation:** 
Amazon Aurora Global Database provides two distinct promotion mechanisms:
1. **Managed Planned Failover (Switchover):** Designed for planned maintenance and DR drills. It pauses writes on the primary, waits for all storage changes to replicate to the secondary (guaranteeing zero data loss, RPO = 0), promotes the secondary to the new primary, and seamlessly reverses replication so the former primary becomes a secondary.
2. **Unplanned Failover:** Designed for unplanned catastrophic regional outages. It immediately breaks replication and promotes the secondary cluster to standalone status, accepting the possibility that writes in-flight during the disaster might not have reached the secondary.

---

### Question 5: DynamoDB Global Tables Multi-Region Active-Active Conflict Resolution

**Answer:** B  
**Explanation:** 
Amazon DynamoDB Global Tables operates with an **eventually consistent, active-active** replication model. DynamoDB does not use distributed locking (which would introduce massive cross-region latency). Instead, it resolves write conflicts using a **Last-Writer-Wins** reconciliation strategy. DynamoDB compares the timestamps of the conflicting update operations; the write with the most recent timestamp prevails across all replica tables.

---

### Question 6: Elastic Load Balancer Cross-Zone Load Balancing and Connection Draining

**Answer:** B  
**Explanation:** 
When Cross-Zone Load Balancing is disabled on a load balancer target group, each load balancer node distributes traffic only among the targets in its own Availability Zone. If clients resolve ALB IPs evenly across AZs (33% traffic per AZ), the 2 instances in `us-east-1a` must absorb 33% of total traffic (16.5% per instance), while the 10 instances in `us-east-1c` share 33% (3.3% per instance). Enabling Cross-Zone Load Balancing distributes all requests evenly (5% per instance across the entire fleet).

---

### Question 7: Auto Scaling Target Tracking with ALB Request Count Per Target

**Answer:** C  
**Explanation:** 
AWS Auto Scaling Target Tracking policies operate like a home thermostat. Auto Scaling provides predefined metrics, including `ASGAverageCPUUtilization`, `ASGAverageNetworkIn`, `ASGAverageNetworkOut`, and `ALBRequestCountPerTarget`. By setting `ALBRequestCountPerTarget: 1000`, AWS automatically creates and manages the underlying CloudWatch metric alarms, calculates required capacity additions/removals, and adjusts desired capacity proportionally.

---

### Question 8: Disaster Recovery Strategy Comparison (RPO / RTO Trade-offs)

**Answer:** B  
**Explanation:** 
- *Backup and Restore:* Cheapest, but RTO is several hours (too slow for 30m RTO).
- *Pilot Light:* Maintains critical data stores running and continuously replicated (satisfying the 15m RPO). Compute instances are not running continuously, but AMIs and IaC templates are pre-staged so they can be launched within 15–20 minutes upon failover (satisfying the 30m RTO at minimal cost).
- *Warm Standby:* More expensive than Pilot Light because scaled-down fleets run 24/7.
- *Active-Active:* Most expensive ($2\times$ compute spend).

---

### Question 9: High-Availability AWS Storage Gateway Architecture

**Answer:** B  
**Explanation:** 
AWS Storage Gateway provides built-in integration with VMware vSphere High Availability (HA). By deploying the gateway on a VMware cluster with vSphere HA and enabling the Storage Gateway HA health monitor, vSphere automatically restarts the gateway VM on a healthy host if the underlying hardware fails, and the gateway automatically recovers its local cache without loss of cached data or iSCSI disconnections.

---

### Question 10: ASG Termination Policies and Scale-In Protection

**Answer:** A  
**Explanation:** 
- **Instance Scale-In Protection:** Prevents specific instances from being terminated by Auto Scaling during scale-in events or health check replacements. The application enables scale-in protection when a batch job starts and clears it when complete.
- **Custom Termination Policy:** Auto Scaling first balances instances across Availability Zones by default. Specifying `OldestLaunchTemplate` then ensures that among the candidate instances in the chosen AZ, instances running outdated templates are terminated first.

---

### Question 11: Route 53 Application Recovery Controller (ARC) Routing Controls

**Answer:** B  
**Explanation:** 
Amazon Route 53 Application Recovery Controller (ARC) **Routing Controls** act as simple on/off switches that control DNS routing for multi-region architectures. ARC uses highly available regional cluster endpoints (replicated across 5 regions) that execute state changes in under 1 second. When coupled with Route 53 routing control health checks, toggling an ARC switch immediately fails the health check and shifts traffic away from the degraded region with deterministic safety.

---

### Question 12: Network Load Balancer (NLB) TCP Connection Resilience

**Answer:** B  
**Explanation:** 
On Network Load Balancers, the **Deregistration Delay** attribute defines how long the NLB waits before completing target deregistration and terminating in-flight connections. For long-lived TCP connections (like IoT telemetry), setting the deregistration delay up to 3600 or 7200 seconds allows active TCP streams to continue transmitting data without interruption until the client disconnects or the timeout elapses.

---

### Question 13: ElastiCache Redis Cluster Multi-AZ Failover

**Answer:** B  
**Explanation:** 
Amazon ElastiCache for Redis supports **Multi-AZ with Automatic Failover**. When Multi-AZ is enabled, ElastiCache maintains a primary node and up to 5 read replicas across distinct Availability Zones. If the primary node fails, ElastiCache detects the failure, promotes a read replica to primary, and updates the DNS record of the **Primary Endpoint** to point to the promoted node in under 60 seconds.

---

### Question 14: Amazon EFS Mount Target High Availability

**Answer:** B  
**Explanation:** 
Amazon EFS Standard storage class stores file system data redundantly across multiple Availability Zones. To connect to an EFS file system from EC2 instances or ECS tasks, you must create an **EFS Mount Target in each Availability Zone** (subnet). Instances in an AZ mount the local mount target, ensuring low-latency access and uninterrupted availability even if an entire Availability Zone becomes unavailable.

---

### Question 15: S3 Multi-Region Access Points (MRAP) with Failover Controls

**Answer:** B  
**Explanation:** 
Amazon S3 Multi-Region Access Points (MRAP) provide a single global DNS endpoint (`s3-mrap.amazonaws.com`) that routes client requests over the AWS global network backbone directly to the lowest-latency healthy S3 bucket. MRAP includes native **Failover Controls** in the S3 console, allowing operators to actively or passively shift traffic between regional S3 buckets in minutes during an outage.

---

### Scoring Guide
- **14–15 Correct:** Mastery (Ready for DOP-C02 Resilient Cloud Solutions section)
- **12–13 Correct:** Solid Proficiency (Review specific Route 53 routing policies and ASG Instance Refresh)
- **$\le 11$ Correct:** Needs Reinforcement (Review `Domain_3_Resilient_Cloud_Solutions/Study_Guide.md` and `Exercises.md`)
