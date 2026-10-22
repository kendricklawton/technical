# AWS Certified Solutions Architect - Professional (SAP-C02) Practice Exam 4

**Duration:** 60 Minutes (15 Full-Length Multi-Paragraph Scenario Questions with Distractor Analysis)  
**Passing Score Target:** $\ge 12 / 15$ (80%)  
**Exam Blueprint Focus:** Domain 1 (Organizational Complexity: 26%), Domain 2 (Design for New Solutions: 29%), Domain 3 (Continuous Improvement: 25%), Domain 4 (Migration & Modernization: 20%)

---

### Question 1 (Domain 1: Organizational Complexity)
An enterprise with 150 AWS accounts managed in AWS Organizations requires an automated solution to ensure that no IAM user or role across any member account can create unencrypted Amazon EBS volumes. The solution must prevent the creation of unencrypted volumes at the API level rather than detecting and terminating them after creation. Additionally, existing workloads with legacy unencrypted volumes must not be terminated or disrupted.

**Which strategy accomplishes these objectives with the LEAST administrative effort?**
- A. Deploy an AWS Config rule `ec2-volume-inuse-check` across all accounts using AWS Systems Manager Quick Setup, and attach an automated remediation document that terminates non-compliant instances.
- B. Attach a Service Control Policy (SCP) to the organization root that denies the `ec2:CreateVolume` and `ec2:RunInstances` actions if the `ec2:Encrypted` condition key evaluates to `false`.
- C. Use AWS CloudFormation StackSets to deploy an Amazon EventBridge rule in every account that triggers an AWS Lambda function to encrypt volumes upon creation.
- D. Enable default EBS encryption in every AWS Region across all accounts using an AWS Systems Manager Automation runbook executed from the management account.

**Answer:** B  
**Explanation:** Service Control Policies (SCPs) act as centralized permission guardrails that filter the maximum permissions available to IAM users and roles in member accounts. By applying a `Deny` SCP with `Condition: { "Bool": { "ec2:Encrypted": "false" } }` on `ec2:CreateVolume` and `ec2:RunInstances`, any API call attempting to create an unencrypted volume is blocked synchronously at the API level before creation. SCPs do not affect existing running resources, satisfying the requirement to avoid disrupting legacy workloads.  
**Distractor Analysis:**
- *A is incorrect:* AWS Config is reactive and detective; terminating instances violates the requirement not to disrupt existing workloads.
- *C is incorrect:* Lambda executes asynchronously after the volume is already created, which fails the requirement to prevent creation at the API level.
- *D is incorrect:* Enabling default EBS encryption helps for new volumes, but it does not prevent an administrator from explicitly overriding or disabling encryption if IAM permissions permit it.

---

### Question 2 (Domain 2: Design for New Solutions)
A media streaming company delivers on-demand 4K video content globally. The video files are stored in an Amazon S3 bucket in `us-east-1` and distributed via Amazon CloudFront. CloudFront cache hit ratios are currently low (35%) because video requests include dynamic query string parameters used for viewer analytics (e.g., `?viewer_id=xyz&session=123`). The platform architecture team needs to maximize CloudFront edge caching while continuing to collect viewer analytics for real-time reporting.

**Which architecture provides the HIGHEST cache hit ratio with MINIMAL latency?**
- A. Configure the CloudFront distribution cache policy to forward all query strings to the origin S3 bucket.
- B. Deploy a CloudFront Function at Viewer Request to strip the analytics query strings before cache lookup, and use the CloudFront Function to write the analytics parameters directly to an Amazon Kinesis Data Stream.
- C. Deploy a CloudFront Function at Viewer Request to parse the analytics query strings, emit them into CloudFront real-time access logs or a CloudFront key-value store, and rewrite the request URI to exclude the query strings prior to cache key evaluation.
- D. Deploy an AWS Lambda@Edge function at Origin Request to remove query strings before sending requests to Amazon S3.

**Answer:** C  
**Explanation:** By stripping dynamic query strings at the **Viewer Request** stage using **CloudFront Functions**, the cache key calculation ignores the unique `viewer_id` and `session` parameters, allowing all viewers requesting the same video chunk to hit the same cached object at the edge (driving cache hit ratios to 95%+). CloudFront Functions execute sub-millisecond at scale across hundreds of edge locations at a fraction of the cost of Lambda@Edge, and analytics can be captured via CloudFront real-time logs.  
**Distractor Analysis:**
- *A is incorrect:* Forwarding dynamic unique query strings causes every unique viewer session to be treated as a cache miss, resulting in near-zero caching.
- *B is incorrect:* CloudFront Functions run in a restricted JavaScript runtime that cannot make network calls or talk directly to external AWS SDK endpoints like Kinesis.
- *D is incorrect:* Origin Request triggers only on cache misses, meaning it cannot improve the initial edge cache hit ratio.

---

### Question 3 (Domain 4: Migration & Modernization)
A legacy monolithic application running on an on-premises Oracle RAC database must be migrated to AWS. The database is 18 TB, receives 2,500 transactions per second, and cannot tolerate more than 15 minutes of downtime during the cutover window. The target cloud architecture must eliminate commercial database licensing costs while maintaining ACID transactional compliance and scaling read throughput.

**Which migration and target architecture should the solutions architect choose?**
- A. Use the AWS Schema Conversion Tool (SCT) to convert the Oracle schema and PL/SQL stored procedures to Amazon Aurora PostgreSQL. Deploy an Aurora PostgreSQL Multi-AZ cluster with Aurora Read Replicas. Use AWS Database Migration Service (DMS) with Ongoing Replication (CDC) over an AWS Direct Connect connection to replicate changes, followed by a final cutover within the maintenance window.
- B. Use AWS Application Migration Service (MGN) to block-level replicate the on-premises Oracle RAC virtual machines to Amazon EC2 instances running Oracle Database.
- C. Export the Oracle database using Oracle Data Pump, upload the dump files to Amazon S3 via AWS Snowball Edge, and import them into Amazon DynamoDB.
- D. Deploy an Amazon Redshift cluster and replicate data using AWS DataSync over the public internet.

**Answer:** A  
**Explanation:** AWS SCT converts Oracle schemas, views, and PL/SQL code to open-source PostgreSQL syntax. AWS DMS performs full load and continuous Change Data Capture (CDC) over Direct Connect without taking the on-premises database offline. During the cutover, traffic is paused, final CDC changes are applied in seconds ($< 15$ min downtime), and the application switches to Amazon Aurora PostgreSQL, eliminating commercial Oracle license costs.  
**Distractor Analysis:**
- *B is incorrect:* Retaining Oracle on EC2 fails the requirement to eliminate commercial licensing costs.
- *C is incorrect:* Snowball and Data Pump export/import take days and cannot achieve a $< 15$-minute cutover window for active transactional databases.
- *D is incorrect:* Redshift is an OLAP data warehouse, not an OLTP ACID database for transaction processing.

---

### Question 4 (Domain 3: Continuous Improvement)
A containerized microservices application runs on Amazon EKS across 50 EC2 worker nodes. The platform team notices that during peak traffic, new pods remain in `Pending` state for over 4 minutes because the AWS Cluster Autoscaler takes several minutes to provision Auto Scaling Group (ASG) instances. Additionally, compute costs are high due to fragmented resource allocations.

**Which solution resolves pod startup delays and optimizes compute costs with the LEAST operational complexity?**
- A. Replace the Cluster Autoscaler with **Karpenter**. Configure a `NodePool` that provisions right-sized Spot and On-Demand instances directly via AWS EC2 Fleet APIs with `WhenEmptyOrUnderutilized` consolidation.
- B. Increase the minimum size of the node group ASG to 80 instances to absorb peak traffic.
- C. Deploy Kubernetes Horizontal Pod Autoscaler (HPA) without changing the node autoscaling mechanism.
- D. Migrate all EKS workloads to AWS Fargate with dedicated pod profiles.

**Answer:** A  
**Explanation:** Karpenter evaluates pending pod resource requests directly and launches right-sized EC2 instances in under 45 seconds, bypassing rigid Auto Scaling Groups. Karpenter's consolidation policies automatically pack underutilized workloads and terminate redundant nodes, maximizing cost efficiency.  
**Distractor Analysis:**
- *B is incorrect:* Over-provisioning instances permanently inflates costs.
- *C is incorrect:* HPA creates pods faster, but pods remain pending if worker nodes lack compute capacity.
- *D is incorrect:* Fargate pods have fixed resource tiers, do not support Spot node consolidation in the same manner, and introduce pod initialization overhead.

---

### Question 5 (Domain 2: Design for New Solutions)
An IoT fleet management company receives telemetry messages from 250,000 delivery vehicles every 10 seconds. The ingestion architecture must ingest, validate, and store messages in real time. Processing must be completely serverless, scale dynamically during traffic surges, and ensure that malformed messages are captured for offline debugging without interrupting the ingestion pipeline.

**Which architecture meets these requirements?**
- A. Vehicles send telemetry to Amazon API Gateway (HTTP API), which forwards messages directly to an Amazon Kinesis Data Stream. An AWS Lambda function processes Kinesis records in micro-batches and writes them to Amazon Timestream. Configure a Dead-Letter Queue (DLQ) pointing to an Amazon SQS queue for failed Lambda invocations.
- B. Vehicles publish messages to Amazon EC2 instances running an Apache Kafka broker cluster behind a Network Load Balancer.
- C. Vehicles send HTTP requests to an Amazon S3 bucket using S3 presigned URLs. S3 Event Notifications trigger AWS Batch jobs.
- D. Vehicles connect to an Amazon RDS MySQL Multi-AZ database directly over JDBC connections.

**Answer:** A  
**Explanation:** API Gateway HTTP API integrated directly with Kinesis Data Streams provides a highly available, serverless ingestion gateway capable of handling hundreds of thousands of requests per second. Lambda processes Kinesis stream records with high concurrency and writes time-series metrics to Amazon Timestream. Configuring a DLQ on the Lambda event source mapping guarantees malformed messages are captured in SQS without pipeline blockage.  
**Distractor Analysis:**
- *B is incorrect:* Self-managed Kafka on EC2 requires significant operational maintenance and is not serverless.
- *C is incorrect:* Uploading 250,000 files every 10 seconds directly to S3 causes extreme API overhead and high S3 PUT costs.
- *D is incorrect:* Direct JDBC connections from 250,000 clients will exhaust MySQL database connection limits and crash the database.

---

### Question 6 (Domain 1: Organizational Complexity)
A multinational corporation with 50 member accounts in an AWS Organization requires a centralized logging solution. All VPC Flow Logs, AWS CloudTrail logs, and AWS WAF logs must be aggregated into an S3 bucket in a dedicated Log Archive account. To satisfy compliance regulations, log files must be immutable: no user—including the AWS account root user—may delete or overwrite log files for 7 years.

**Which combination of actions fulfills this compliance requirement?**
- A. Enable S3 Versioning and configure an S3 Lifecycle rule to transition logs to S3 Glacier Flexible Retrieval after 7 years.
- B. Enable S3 Versioning on the target S3 bucket in the Log Archive account. Configure **S3 Object Lock** in **Compliance Mode** with a retention period of 7 years. Apply an S3 Bucket Policy enforcing TLS and denying `s3:DeleteObject` and `s3:DeleteObjectVersion`.
- C. Enable AWS CloudTrail log file validation and create an AWS KMS customer managed key (CMK) with key deletion disabled.
- D. Store logs in an Amazon Elastic File System (EFS) volume mounted across all accounts with POSIX read-only permissions.

**Answer:** B  
**Explanation:** Amazon S3 Object Lock using **Compliance Mode** guarantees that objects cannot be deleted or overwritten by ANY user, including the root user of the AWS account, for the duration of the retention period (WORM: Write Once, Read Many). This is the only AWS mechanism that provides mathematically verified immutability against root user actions.  
**Distractor Analysis:**
- *A is incorrect:* S3 Versioning without Object Lock permits root or authorized admin users to delete object versions.
- *C is incorrect:* Log file validation proves whether a log was modified, but it does not prevent the deletion of the log file itself.
- *D is incorrect:* EFS does not provide compliance-certified WORM storage and cannot enforce cross-account immutability against root users.

---

### Question 7 (Domain 3: Continuous Improvement)
An online retail platform experiences severe database read latency during promotional flash sales. The application tier runs on Amazon EC2 instances querying an Amazon RDS MySQL Multi-AZ database. Database CPU utilization reaches 98%, and 80% of database queries are identical read-only requests fetching product catalog descriptions. The engineering team needs to eliminate database bottlenecks with the LOWEST operational cost and minimal application refactoring.

**Which solution should the solutions architect recommend?**
- A. Migrate the database to Amazon Aurora Serverless v2 with 128 ACUs.
- B. Deploy an Amazon ElastiCache for Redis cluster in front of the database. Implement a cache-aside (lazy loading) pattern in the application tier to store and serve product catalog queries from memory.
- C. Scale up the RDS instance to `db.m6i.32xlarge`.
- D. Enable Multi-AZ with two readable standbys on Amazon RDS.

**Answer:** B  
**Explanation:** A cache-aside caching tier using Amazon ElastiCache for Redis absorbs the 80% repetitive catalog read queries, serving them in sub-millisecond memory time. This drops database CPU utilization from 98% to under 25% without requiring expensive database instance scaling.  
**Distractor Analysis:**
- *A is incorrect:* Aurora Serverless v2 at 128 ACUs is extremely expensive compared to an ElastiCache cluster.
- *C is incorrect:* Vertical scaling is costly and does not solve the root cause of serving identical repetitive queries from disk.
- *D is incorrect:* Readable standbys add cost and still incur relational query parsing overhead compared to in-memory caching.

---

### Question 8 (Domain 2: Design for New Solutions)
An enterprise requires a secure, automated disaster recovery (DR) solution for an internal web application. The application runs on EC2 instances behind an ALB in `us-east-1` with an Amazon Aurora PostgreSQL database. The business requires an RTO $< 15$ minutes and RPO $< 5$ minutes in the disaster recovery region `us-west-2`. The budget requires the DR region to incur minimal compute cost during normal operations.

**Which disaster recovery strategy meets the RTO/RPO targets while minimizing idle costs?**
- A. **Pilot Light:** Deploy an Aurora Global Database with a secondary read replica cluster in `us-west-2`. Pre-provision the VPC, subnets, and ALB in `us-west-2`, but configure the Auto Scaling Group in `us-west-2` with `min=0, desired=0`. Use Route 53 Application Recovery Controller (ARC) health checks to scale up the ASG and promote the Aurora replica during failover.
- B. **Active-Active:** Run full-capacity compute clusters and databases in both `us-east-1` and `us-west-2` behind Route 53 Weighted Routing.
- C. **Backup and Restore:** Use AWS Backup to take daily snapshots of EC2 instances and the Aurora database and copy them to `us-west-2`.
- D. **Warm Standby:** Run an Auto Scaling Group with 50% capacity in `us-west-2` continuously.

**Answer:** A  
**Explanation:** Pilot Light maintains the data tier synchronized in real time via Aurora Global Database (cross-region replication latency is typically $< 1$ second, well within the 5-minute RPO), while keeping compute at zero instances to eliminate ongoing compute costs. In a disaster, Route 53 ARC triggers an automated scale-up of the ASG and promotes the database cluster in $< 15$ minutes RTO.  
**Distractor Analysis:**
- *B is incorrect:* Active-Active incurs 100% duplicate infrastructure cost, violating the budget constraint.
- *C is incorrect:* Restoring 18 TB databases and EC2 AMIs from cold snapshots takes hours, violating the 15-minute RTO.
- *D is incorrect:* Warm Standby runs idle compute 24/7, which is more expensive than Pilot Light.

---

### Question 9 (Domain 4: Migration & Modernization)
A hospital network stores 500 TB of medical imaging DICOM files on on-premises SAN storage. The hospital needs to migrate these images to Amazon S3 to integrate with cloud-native AI diagnostics. The network connection between the hospital data center and AWS is a 1 Gbps AWS Direct Connect connection with 70% average background utilization. The migration must complete in under 3 weeks without impacting production clinical traffic.

**Which migration method meets the time and network constraints?**
- A. Order two AWS Snowball Edge Storage Optimized devices. Copy the medical images to the Snowball devices on premises, ship them to AWS, and import the data into Amazon S3. Configure AWS DataSync over the Direct Connect with bandwidth throttling to sync delta changes.
- B. Use AWS DataSync over the Direct Connect without bandwidth limits to transfer all 500 TB continuously.
- C. Use the AWS S3 CLI `aws s3 sync` over the public internet via a VPN connection.
- D. Deploy an AWS Storage Gateway File Gateway on premises and mount it as an NFS share.

**Answer:** A  
**Explanation:** With a 1 Gbps Direct Connect utilized at 70%, only 300 Mbps is available for migration. Transferring 500 TB over 300 Mbps would take $\approx 154$ days, failing the 3-week deadline. Ordering two AWS Snowball Edge devices (each providing 210 TB usable storage) allows local 10 GbE copying in days, physical shipment to AWS in 2–3 days, and immediate S3 ingestion. Ongoing incremental changes are then synced using AWS DataSync over Direct Connect with bandwidth throttling.  
**Distractor Analysis:**
- *B is incorrect:* Running DataSync without throttling consumes 100% of the Direct Connect, crashing hospital clinical systems.
- *C is incorrect:* Internet VPN is even slower than Direct Connect.
- *D is incorrect:* Storage Gateway File Gateway does not accelerate the initial 500 TB upload over constrained bandwidth.

---

### Question 10 (Domain 1: Organizational Complexity)
An organization uses AWS IAM Identity Center (successor to AWS Single Sign-On) integrated with Microsoft Entra ID (Azure AD) via SAML 2.0. Security policy requires multi-factor authentication (MFA) for all administrative access. However, external contractors who do not possess Entra ID corporate credentials must be granted temporary, federated access to specific developer accounts.

**Which solution fulfills this requirement while adhering to security best practices?**
- A. Create local IAM users with static access keys in each developer account for every contractor.
- B. Configure an Amazon Cognito user pool for contractors, and configure a SAML identity provider in IAM Identity Center or configure an IAM Role with Web Identity Federation allowing contractors to assume the role using their Cognito credentials with MFA enforced.
- C. Share the root account password with contractors.
- D. Deploy a self-managed OpenLDAP server on EC2 and sync it to IAM.

**Answer:** B  
**Explanation:** Using Amazon Cognito as an identity pool or secondary IdP integrated with IAM Identity Center or IAM Role Web Identity Federation allows external contractors to authenticate without cluttering the corporate Entra ID tenant. MFA can be enforced at the Cognito or IAM policy level, and temporary STS credentials are issued without static keys.  
**Distractor Analysis:**
- *A is incorrect:* Creating static IAM users in member accounts violates enterprise credential management and least-privilege standards.
- *C is incorrect:* Sharing root credentials is a severe security violation.
- *D is incorrect:* Self-managed LDAP introduces high operational overhead.

---

### Question 11 (Domain 2: Design for New Solutions)
A payment processing platform requires an event-driven microservices architecture where payment transactions must be processed strictly in the order they occurred. If an individual transaction fails downstream processing, subsequent transactions for THAT SPECIFIC customer must pause, while transactions for other customers must continue processing without delay. The architecture must handle 10,000 transactions per second.

**Which messaging architecture satisfies these ordering and throughput requirements?**
- A. Amazon SNS Standard Topic fan-out to Amazon SQS Standard Queues.
- B. Amazon SQS FIFO Queue configured with high-throughput mode enabled, using the `CustomerID` as the `MessageGroupId`.
- C. Amazon Kinesis Data Firehose delivering records to Amazon OpenSearch Service.
- D. AWS Step Functions standard workflow executing in a sequential loop.

**Answer:** B  
**Explanation:** SQS FIFO Queues guarantee strictly ordered processing (FIFO) and exactly-once processing per **Message Group ID**. By setting `MessageGroupId = CustomerID`, SQS guarantees that transactions for that specific customer are processed in order and blocked if a message fails, while transactions with different Message Group IDs are processed in parallel across multiple consumers. SQS FIFO high-throughput mode supports up to 30,000 messages per second with batching.  
**Distractor Analysis:**
- *A is incorrect:* SQS Standard queues do not guarantee strict message ordering or message deduplication.
- *C is incorrect:* Kinesis Data Firehose is for batch loading data to data lakes, not for microservice transaction orchestration.
- *D is incorrect:* Step Functions standard workflows cannot scale to 10,000 TPS cost-effectively for simple queue decoupling.

---

### Question 12 (Domain 3: Continuous Improvement)
A financial firm runs a mission-critical risk analytics platform on AWS. The compliance department mandates that all AWS API calls across the entire multi-region organization must be audited, and any unauthorized API call attempting to disable security services (such as AWS GuardDuty, AWS Security Hub, or AWS Config) must trigger an immediate automated incident response within 60 seconds.

**Which architecture meets this requirement with the FASTEST response time?**
- A. Configure AWS CloudTrail organization-wide to log events to Amazon S3. Run an Amazon Athena scheduled query every 5 minutes to detect unauthorized calls.
- B. Configure AWS CloudTrail organization trail sending events to Amazon CloudWatch Logs. Create an Amazon EventBridge rule that matches specific `errorCode: AccessDenied` or security termination API calls (e.g. `guardduty:DeleteDetector`), which immediately triggers an AWS Lambda remediation function and an Amazon SNS alert.
- C. Deploy an EC2 instance running open-source Osquery to parse CloudTrail S3 logs.
- D. Use AWS Trusted Advisor notifications sent via weekly email digest.

**Answer:** B  
**Explanation:** CloudTrail streams events to CloudWatch Logs and EventBridge in near real time ($< 15$ seconds). EventBridge pattern matching evaluates the event immediately and triggers an automated AWS Lambda function to remediate the action or isolate the compromised credential within seconds.  
**Distractor Analysis:**
- *A is incorrect:* Athena scheduled queries take 5+ minutes, violating the 60-second requirement.
- *C is incorrect:* Self-managed EC2 log parsers introduce latency and operational failure modes.
- *D is incorrect:* Weekly emails fail the real-time automated response requirement.

---

### Question 13 (Domain 4: Migration & Modernization)
A company is refactoring an on-premises 3-tier Java application to run on AWS. The application uses shared file storage to store user profile pictures and invoices accessed concurrently by hundreds of web servers. The shared storage requires POSIX compliance, high throughput for small files, and integration with AWS IAM for access control.

**Which storage service should the solutions architect select?**
- A. Amazon Elastic Block Store (EBS) Multi-Attach with `io2` volumes.
- B. Amazon Elastic File System (EFS) configured with General Purpose performance mode and IAM authorization for NFS clients.
- C. Amazon S3 mounted on EC2 instances using `s3fs-fuse`.
- D. Amazon FSx for Lustre.

**Answer:** B  
**Explanation:** Amazon EFS is a serverless, fully managed, POSIX-compliant file system designed for concurrent access by hundreds or thousands of compute instances. It supports IAM authorization for NFS clients (`aws:elasticfilesystem:AccessPoint`) to enforce granular access controls.  
**Distractor Analysis:**
- *A is incorrect:* EBS Multi-Attach requires a cluster-aware file system (such as GFS2 or OCFS2) to prevent file system corruption, which adds high complexity.
- *C is incorrect:* S3 is an object store, not a POSIX file system; `s3fs-fuse` does not provide true POSIX semantics and suffers from severe performance and locking limitations.
- *D is incorrect:* FSx for Lustre is optimized for high-performance computing (HPC) and batch processing, not generic web application shared file storage.

---

### Question 14 (Domain 1: Organizational Complexity)
An enterprise connects 50 AWS VPCs in `us-east-1` and 20 on-premises data center sites. The company has deployed an AWS Direct Connect dedicated connection with a Transit Virtual Interface (Transit VIF). The network engineering team must ensure that all traffic between the on-premises network and AWS VPCs is encrypted in transit at the network layer using MACsec (802.1AE).

**Which requirement must be met to enable MACsec encryption on AWS Direct Connect?**
- A. Direct Connect dedicated connection at 10 Gbps or 100 Gbps speeds at a MACsec-compatible Direct Connect location, with encryption keys stored in AWS Secrets Manager.
- B. Any 1 Gbps hosted connection configured with IPsec VPN.
- C. An AWS Site-to-Site VPN attached to a Virtual Private Gateway (VGW).
- D. AWS Cloud WAN with software-defined WAN (SD-WAN) edge routers.

**Answer:** A  
**Explanation:** MACsec (IEEE 802.1AE) provides line-rate hardware encryption at Layer 2. On AWS Direct Connect, MACsec is available exclusively on **dedicated 10 Gbps or 100 Gbps connections** provisioned at specific MACsec-enabled Direct Connect locations. The MACsec pre-shared CKN/CAK keys are managed in AWS Secrets Manager.  
**Distractor Analysis:**
- *B is incorrect:* MACsec is not supported on hosted connections or 1 Gbps connections.
- *C is incorrect:* Site-to-Site VPN provides Layer 3 IPsec encryption, not Layer 2 MACsec line-rate encryption.
- *D is incorrect:* Cloud WAN does not implement MACsec hardware encryption directly on physical Direct Connect ports.

---

### Question 15 (Domain 3: Continuous Improvement)
A high-traffic e-commerce application serves millions of global users. The application runs on EC2 instances in an Auto Scaling Group behind an Application Load Balancer in `us-west-2`. Users in Europe and Asia report high latency when loading dynamic web pages. Static assets are already cached on CloudFront. The dynamic API responses cannot be cached due to personalized user data.

**Which solution reduces network latency for dynamic global users with MINIMAL architectural change?**
- A. Deploy Amazon Global Accelerator in front of the Application Load Balancer.
- B. Deploy duplicate EC2 application clusters in `eu-west-1` and `ap-southeast-1` with database replication.
- C. Increase the size of the ALB in `us-west-2`.
- D. Switch from HTTP/2 to HTTP/1.1.

**Answer:** A  
**Explanation:** **AWS Global Accelerator** provides static anycast IP addresses that route user traffic onto the AWS global private fiber network at the nearest edge location. From the edge location, traffic travels over the uncongested AWS backbone directly to the ALB in `us-west-2`, significantly reducing TCP handshake latency, packet loss, and jitter for dynamic, non-cacheable traffic.  
**Distractor Analysis:**
- *B is incorrect:* Deploying full multi-region active-active compute and database tiers involves massive architectural refactoring and cost, violating the minimal change constraint.
- *C is incorrect:* ALB size does not reduce public internet transatlantic fiber latency.
- *D is incorrect:* HTTP/1.1 has higher latency than HTTP/2 due to head-of-line blocking.
