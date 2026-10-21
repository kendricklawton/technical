# Domain 3 (Resilient Cloud Solutions) — Quiz 5: High Availability & Distributed Databases

**Topic:** Aurora Global Database, DynamoDB Global Tables, RDS Multi-AZ vs Read Replicas & ElastiCache HA  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: Unplanned Regional Failover in Amazon Aurora Global Database

A company runs its core transactional workload on an Amazon Aurora MySQL Global Database with the primary cluster in `us-east-1` and a secondary cluster in `us-west-2`. A catastrophic natural disaster takes `us-east-1` completely offline. The DevOps engineer must promote the secondary cluster in `us-west-2` to take over write operations as quickly as possible.

**Which AWS CLI command or management action must the engineer execute to restore write availability?**
- A. Run `aws rds switchover-global-cluster` targeting `us-west-2`.
- B. Run `aws rds failover-global-cluster --global-cluster-identifier my-global-db --target-db-cluster-identifier arn:aws:rds:us-west-2:123456789012:cluster:aurora-west-cluster`.
- C. Delete the primary cluster in `us-east-1` using `--skip-final-snapshot`.
- D. Reboot all reader instances in `us-west-2`.

---

### Question 2: Amazon DynamoDB Global Tables Conflict Resolution

An e-commerce mobile app uses Amazon DynamoDB Global Tables across three Regions: `us-east-1`, `eu-west-1`, and `ap-southeast-1`. Two concurrent updates are performed on the same item attribute (`customer_address`) at nearly identical times:
- Transaction A writes in `us-east-1` at timestamp `10:00:00.100Z`.
- Transaction B writes in `eu-west-1` at timestamp `10:00:00.150Z`.

**How does DynamoDB Global Tables resolve this concurrent write conflict across Regions?**
- A. It throws a `TransactionConflictException` and rolls back both writes.
- B. It uses a **Last-Writer-Wins** reconciliation strategy based on DynamoDB internal write timestamps; Transaction B (`10:00:00.150Z`) supersedes Transaction A across all regions.
- C. It locks the partition across all three Regions using two-phase commit (2PC).
- D. It merges the attributes into an array of conflicting values.

---

### Question 3: RDS Multi-AZ Instance vs Cross-Region Read Replica Failover

A DevOps architect needs to clarify the operational differences between an Amazon RDS Multi-AZ deployment and an Amazon RDS Cross-Region Read Replica for database failover planning.

**Which statement accurately describes the synchronization and failover characteristics of standard RDS Multi-AZ?**
- A. Multi-AZ uses asynchronous block-level replication; DNS endpoint changes require manual route table modification.
- B. Multi-AZ uses **synchronous physical replication** to a standby instance in a different Availability Zone within the same Region. During an automated failover, RDS automatically updates the database CNAME record to point to the standby instance without changing connection string hostnames.
- C. Multi-AZ standby instances can serve active read traffic concurrently with the primary writer instance in standard Multi-AZ.
- D. Multi-AZ failover is triggered only by manual CLI commands and takes 60 minutes.

---

### Question 4: Aurora Multi-AZ Storage Replication Architecture

How does Amazon Aurora protect data durability and ensure high availability against storage failure within a single AWS Region?
- A. Aurora mirrors data across two EBS gp3 volumes in two Availability Zones.
- B. Aurora replicates 6 copies of data across 3 Availability Zones (2 copies per AZ). Aurora storage maintains write quorum with 4 of 6 nodes and read quorum with 3 of 6 nodes, automatically tolerating the loss of an entire AZ plus an additional storage node without losing write or read availability.
- C. Aurora writes data to a single S3 bucket and relies on S3 lifecycle policies.
- D. Aurora uses local instance NVMe disks with nightly snapshots.

---

### Question 5: Redis Cluster Multi-AZ Failover in Amazon ElastiCache

An application uses Amazon ElastiCache for Redis with Cluster Mode disabled, running 1 primary node and 2 read replicas across three Availability Zones with Multi-AZ and Auto-Failover enabled.

**What happens if the primary cache node experiences a catastrophic hardware failure?**
- A. ElastiCache promotes one of the read replicas to become the new primary node automatically, updates the primary cluster DNS endpoint, and provisions a replacement replica in the failed AZ.
- B. The cache cluster becomes read-only permanently until recreated via Terraform.
- C. All cached data is permanently erased and must be reloaded from cold storage.
- D. The secondary nodes terminate immediately to prevent data divergence.

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

### Question 1: Unplanned Regional Failover in Amazon Aurora Global Database

**Answer:** B  
**Explanation:** 
Amazon Aurora Global Database provides two promotion workflows:
1. **Planned switchover (`switchover-global-cluster`)**: Synchronizes primary and secondary clusters before flipping roles, ensuring zero data loss (RPO = 0). This requires the primary cluster to be healthy.
2. **Unplanned failover / forced promotion (`failover-global-cluster`)**: Used when the primary region is unavailable or degraded. It breaks replication and promotes the target secondary cluster to become a standalone read/write cluster immediately, accepting whatever replication lag existed at the time of failure.

---

### Question 2: Amazon DynamoDB Global Tables Conflict Resolution

**Answer:** B  
**Explanation:** 
Amazon DynamoDB Global Tables is a multi-Region, multi-active database. It relies on asynchronous replication and uses a **Last-Writer-Wins** conflict resolution mechanism. DynamoDB makes a best effort to determine the last write based on internal update timestamps. The write with the latest timestamp prevails and overwrites concurrent earlier updates across all replica tables. If deterministic locking is required across regions, applications must implement external distributed locking or route writes for a specific partition key to a designated home region.

---

### Question 3: RDS Multi-AZ Instance vs Cross-Region Read Replica Failover

**Answer:** B  
**Explanation:** 
In standard RDS Multi-AZ:
- Replication between primary and standby is **synchronous** at the physical storage block level, ensuring zero data loss (RPO = 0).
- The standby instance is passive and cannot serve read traffic (unlike Multi-AZ DB Cluster deployments with two readable standbys).
- During failover (hardware failure, OS patching, AZ outage), RDS automatically flips the CNAME DNS record to the standby instance within 60–120 seconds.

---

### Question 4: Aurora Multi-AZ Storage Replication Architecture

**Answer:** B  
**Explanation:** 
Aurora's distributed storage architecture partitions database volumes into 10 GB protection groups, each replicated 6 ways across 3 Availability Zones within the Region. Write quorum requires confirmation from 4 out of 6 nodes; read quorum requires 3 out of 6 nodes. This allows Aurora to lose an entire Availability Zone (2 copies) and an additional storage node (1 copy) simultaneously without compromising data durability or read availability, and it can lose an entire AZ without losing write availability.

---

### Question 5: Redis Cluster Multi-AZ Failover in Amazon ElastiCache

**Answer:** A  
**Explanation:** 
When Multi-AZ with Auto-Failover is enabled in ElastiCache for Redis, ElastiCache monitors primary node health. Upon detecting primary failure, it elects and promotes the replica with the least replication lag to become the new primary writer. ElastiCache updates the primary DNS record so application clients reconnect without configuration updates, and it spins up a replacement node to restore the original replica count and AZ distribution.

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

