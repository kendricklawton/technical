# Domain 3 (Resilient Cloud Solutions) — Quiz 4: Disaster Recovery Strategies & Routing Control

**Topic:** DR Patterns (Backup & Restore, Pilot Light, Warm Standby, Multi-Site Active-Active), RTO/RPO Tradeoffs & Route 53 ARC  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: Disaster Recovery Tier Selection for Strict RTO and RPO

A financial services enterprise requires a Disaster Recovery (DR) architecture between `us-east-1` (Primary) and `us-west-2` (Secondary). The regulatory mandate requires an **RPO < 1 minute** and an **RTO < 5 minutes**. Cost is a secondary consideration compared to business continuity, but operations must remain deterministic without relying on cold provisioning of EC2 instances or manual DNS updates during a regional outage.

**Which disaster recovery strategy meets these requirements?**
- A. Backup and Restore using AWS Backup with cross-Region copying scheduled once daily.
- B. Pilot Light with core data continuously replicated to `us-west-2`, and an Auto Scaling Group set to Desired=0 in `us-west-2` that scales up via CloudWatch alarms during an incident.
- C. Multi-Site Active-Active with Amazon Aurora Global Database, Amazon Route 53 latency-based routing with health checks, and fully provisioned application clusters processing live traffic simultaneously in both Regions.
- D. Warm Standby with Amazon DynamoDB local tables and nightly ETL data synchronization via AWS Glue jobs.

---

### Question 2: Amazon Route 53 Application Recovery Controller (ARC) Routing Controls

A mission-critical payment gateway operates across two AWS Regions using a Warm Standby architecture. The DevOps team implements Amazon Route 53 Application Recovery Controller (ARC) to govern regional failover. During a major primary region degradation, the team needs to divert 100% of incoming production traffic away from `us-east-1` to `us-west-2` without relying on Route 53 control plane APIs in the degraded region.

**How does Route 53 ARC ensure reliable failover execution during a regional disaster?**
- A. It requires SSH access to Route 53 nameservers in AWS local zones.
- B. Route 53 ARC Routing Controls utilize a cluster of 5 redundant regional endpoints (data plane) distributed globally; operators can toggle routing control states via any responsive regional endpoint even if the primary region's management console and control planes are completely unreachable.
- C. ARC executes CloudFormation rollbacks in all connected VPCs simultaneously.
- D. ARC dynamically modifies the underlying EC2 instance public IP addresses.

---

### Question 3: Safety Rules in Route 53 ARC to Prevent Split-Brain and Total Outage

An enterprise deploys routing controls across two Regions (`us-east-1` and `eu-west-1`). The operations team wants to ensure that an operator or automated script cannot accidentally turn off traffic to both Regions simultaneously, which would cause a total black hole of all global traffic.

**Which Route 53 ARC construct enforces this operational safeguard?**
- A. An IAM Permission Boundary on the administrator role.
- B. An ARC Assertion Rule (Safety Rule) configured with an evaluation rule specifying that the sum of active routing controls must always be greater than or equal to 1 (`ATLEAST 1`).
- C. A CloudWatch Synthetics canary that triggers an SNS email alert.
- D. A KMS key policy that blocks API calls during weekend hours.

---

### Question 4: Pilot Light vs Warm Standby Compute Sizing

A DevOps engineer is documenting the architectural differences between Pilot Light and Warm Standby for an internal compliance review. 

**What is the defining distinction regarding compute capacity between Pilot Light and Warm Standby?**
- A. In Pilot Light, compute resources in the secondary Region are either non-existent or stopped (only databases/data stores are kept live and synchronized), whereas in Warm Standby, a scaled-down but fully functional fleet of compute instances runs continuously in the secondary Region.
- B. Pilot Light has zero infrastructure deployed in AWS, relying on on-premises data centers.
- C. Warm Standby does not require data replication, whereas Pilot Light replicates continuously.
- D. Pilot Light requires an active Multi-Site configuration with Route 53 weighted records set to 50/50.

---

### Question 5: AWS Backup Cross-Region and Cross-Account Disaster Recovery

A healthcare organization must implement an automated disaster recovery strategy for Amazon EBS volumes, RDS instances, and DynamoDB tables. Corporate governance mandates:
1. Backups must be copied to a secondary AWS Region (`us-west-2`).
2. Backups must be copied to an isolated, dedicated Security & Compliance AWS Account.
3. Once written, backups cannot be deleted or modified by any IAM principal (including root) prior to their 90-day retention expiration.

**Which combination of AWS Backup features accomplishes this?**
- A. Standard EBS snapshots copied with a Lambda script using AWS STS AssumeRole.
- B. An AWS Backup Plan configured in AWS Organizations with a Copy Action to a secondary Region and a cross-account backup vault in the isolated account, protected by **AWS Backup Vault Lock** in **Compliance Mode**.
- C. S3 Glacier Flexible Retrieval with S3 Object Lock in Governance Mode.
- D. AWS Elastic Disaster Recovery (DRS) continuous replication without backup vaults.

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

### Question 1: Disaster Recovery Tier Selection for Strict RTO and RPO

**Answer:** C  
**Explanation:** 
Multi-Site Active-Active runs production traffic concurrently across two or more AWS Regions. Data stores like Amazon Aurora Global Database replicate storage changes across Regions with typical physical replication latencies under 1 second (satisfying RPO < 1 min). Because application stacks are fully running and actively serving traffic in both Regions, RTO is virtually instantaneous (satisfying RTO < 5 min).
- Option A has an RPO of up to 24 hours and RTO of hours.
- Option B (Pilot Light) requires provisioning compute instances at disaster time, which takes 10–15+ minutes, exceeding the 5-minute RTO.
- Option D's nightly ETL violates the 1-minute RPO.

---

### Question 2: Amazon Route 53 Application Recovery Controller (ARC) Routing Controls

**Answer:** B  
**Explanation:** 
Route 53 Application Recovery Controller (ARC) routing control data planes are hosted across five redundant AWS Regional endpoints. To change routing states (e.g., flipping a routing control from On to Off), client applications or administrators connect to any of the 5 Regional endpoints. This design ensures that even if one or two AWS Regions experience massive control plane or console outages, the ARC data plane remains fully available to execute safe, deterministic traffic redirection.

---

### Question 3: Safety Rules in Route 53 ARC to Prevent Split-Brain and Total Outage

**Answer:** B  
**Explanation:** 
Route 53 ARC includes **Safety Rules** (Assertion Rules and Gating Rules). An Assertion Rule can enforce operational invariants, such as requiring at least one target routing control to remain active (`ATLEAST 1`). If an automated runbook or human operator attempts an API call that sets all routing controls to `Off`, the ARC cluster evaluates the safety rule, detects a violation, and rejects the transaction with a 400 Client Error, preventing accidental global downtime.

---

### Question 4: Pilot Light vs Warm Standby Compute Sizing

**Answer:** A  
**Explanation:** 
In the **Pilot Light** strategy, core data infrastructure is running and synchronized, but application compute (EC2 instances, ECS tasks) is not running or is deployed with 0 replicas. Upon disaster declaration, the compute tier is launched/scaled up from AMIs or container images. In **Warm Standby**, a minimal ("pilot light" sized compute or scaled-down fleet) version of the application is always running in the secondary region, handling synthetic tests or minimal read traffic, ready to scale out immediately to handle full production load.

---

### Question 5: AWS Backup Cross-Region and Cross-Account Disaster Recovery

**Answer:** B  
**Explanation:** 
AWS Backup supports centralized management across AWS Organizations. A single Backup Plan can define scheduled backups with automated cross-Region and cross-account copies to destination backup vaults. Applying **AWS Backup Vault Lock** in **Compliance Mode** enforces WORM (Write Once, Read Many) compliance: once the cooling-off period passes, no user, administrator, or AWS account root user can delete or shorten the retention period of recovery points until the retention period expires.

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

