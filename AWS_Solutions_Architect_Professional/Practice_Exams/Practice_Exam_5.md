# AWS Certified Solutions Architect - Professional (SAP-C02) Practice Exam 5

**Duration:** 60 Minutes (15 Full-Length Multi-Paragraph Scenario Questions with Distractor Analysis)  
**Passing Score Target:** $\ge 12 / 15$ (80%)  
**Exam Blueprint Focus:** Domain 1 (Organizational Complexity: 26%), Domain 2 (Design for New Solutions: 29%), Domain 3 (Continuous Improvement: 25%), Domain 4 (Migration & Modernization: 20%)

---

### Question 1 (Domain 1: Organizational Complexity)
An enterprise with multiple business units uses AWS Organizations with hundreds of accounts. Each business unit has an independent AWS Control Tower landing zone managed under a separate OU. The security team mandates that any S3 bucket created across ANY account must reject plaintext HTTP traffic and enforce server-side encryption with AWS KMS CMKs. The policy must be enforced preventatively across all existing and future accounts.

**Which strategy enforces this compliance guardrail globally with the LEAST administrative overhead?**
- A. Write an SCP attached to the Organization Root that denies `s3:PutObject` if the request header does not include `aws:SecureTransport: "true"` and does not specify `s3:x-amz-server-side-encryption: "aws:kms"`.
- B. Deploy an AWS Config custom rule using Lambda to scan all S3 buckets every 60 minutes and delete non-compliant buckets.
- C. Require developers to run a pre-commit script that checks CloudFormation templates.
- D. Deploy an AWS CloudTrail trail to send notifications to an SNS topic whenever an S3 bucket is created.

**Answer:** A  
**Explanation:** Attaching a preventative Service Control Policy (SCP) to the AWS Organization Root automatically cascades to all member accounts and OUs, preventing any IAM principal (including root in member accounts) from uploading objects that violate HTTPS transport or KMS encryption requirements.  
**Distractor Analysis:**
- *B is incorrect:* Deleting buckets destroys business data and is reactive.
- *C is incorrect:* Pre-commit hooks can be bypassed or skipped by developers using the AWS Management Console or CLI.
- *D is incorrect:* Notifications are detective, not preventative.

---

### Question 2 (Domain 2: Design for New Solutions)
A SaaS company is designing a serverless multi-tenant backend on AWS. The application must support thousands of tenant organizations. Each tenant requires strict data isolation, but compute resources (AWS Lambda functions and Amazon API Gateway) must be shared to optimize cost. The database is Amazon DynamoDB.

**Which architecture provides verifiable tenant isolation at the data tier with MINIMAL management overhead?**
- A. Deploy a separate DynamoDB table for every single tenant.
- B. Use a single partitioned DynamoDB table with a composite primary key where the partition key (`PK`) is prefixed with `TENANT#<TenantID>`. When Lambda executes on behalf of a tenant, assume an IAM session policy that restricts DynamoDB access using the `dynamodb:LeadingKeys` condition matching the tenant's ID.
- C. Store tenant data in plaintext files inside an Amazon EFS file system with separate subdirectories for each tenant.
- D. Deploy an Amazon RDS PostgreSQL instance with separate database clusters for each tenant.

**Answer:** B  
**Explanation:** The pooled multi-tenant pattern with **DynamoDB LeadingKeys** IAM condition provides strong cryptographic tenant isolation within a single shared table. By injecting `dynamodb:LeadingKeys: ["TENANT#<TenantID>"]` into the ephemeral STS session token assumed by the Lambda function, DynamoDB blocks any query or write targeting data outside that tenant's partition key range.  
**Distractor Analysis:**
- *A is incorrect:* Managing thousands of DynamoDB tables hits account limits and incurs significant operational overhead.
- *C is incorrect:* EFS POSIX permissions do not provide fine-grained API-level isolation for serverless Lambda execution.
- *D is incorrect:* Provisioning dedicated RDS instances for thousands of tenants is cost-prohibitive.

---

### Question 3 (Domain 4: Migration & Modernization)
A logistics firm is migrating 100 on-premises VMware vSphere virtual machines to AWS. Many of the virtual machines run legacy Linux kernels and specialized proprietary device drivers that cannot be easily containerized. The business requires the fastest possible migration path with zero application code changes, minimal network reconfiguration, and the ability to roll back if necessary.

**Which migration service should the solutions architect use?**
- A. AWS Application Migration Service (AWS MGN).
- B. AWS Database Migration Service (AWS DMS).
- C. AWS Schema Conversion Tool (AWS SCT).
- D. AWS App2Container (A2C).

**Answer:** A  
**Explanation:** AWS MGN provides automated, block-level, continuous asynchronous replication of source servers into AWS. It supports physical, virtual (VMware), and cloud servers, preserves operating system configurations and drivers, and allows non-disruptive testing prior to cutover with instant rollback capability.  
**Distractor Analysis:**
- *B is incorrect:* DMS migrates relational databases, not operating system VMs.
- *C is incorrect:* SCT converts database schemas.
- *D is incorrect:* App2Container containerizes Java/NET apps, which violates the requirement for zero application changes.

---

### Question 4 (Domain 3: Continuous Improvement)
An enterprise analytics workload processes 10 TB of clickstream logs daily using Amazon EMR. The EMR cluster runs on 40 Amazon EC2 On-Demand instances. EMR jobs run for 4 hours every night. Compute costs are unsustainable. The architecture team needs to cut compute expenses by at least 60% while ensuring the nightly job reliably completes within the 4-hour processing SLA.

**Which compute architecture achieves these savings reliably?**
- A. Run the EMR primary (master) node and core nodes on On-Demand or 3-year Compute Savings Plans instances, and run the task nodes entirely on EC2 Spot Instances with an EMR Instance Fleet diversifying across multiple instance types.
- B. Run all master, core, and task nodes on EC2 Spot Instances.
- C. Migrate the EMR cluster to Amazon Lightsail.
- D. Run the EMR cluster 24/7 on Reserved Instances.

**Answer:** A  
**Explanation:** In Amazon EMR, the master node coordinates the cluster and core nodes host the HDFS distributed file system. Terminating core nodes causes HDFS data loss and job failure. However, **task nodes** perform pure compute without hosting HDFS data; if a Spot task node is reclaimed, EMR automatically reschedules the tasks on remaining nodes. Using Spot instances for task nodes yields 70–90% cost savings, while keeping master and core nodes on On-Demand/Savings Plans guarantees cluster stability and SLA adherence.  
**Distractor Analysis:**
- *B is incorrect:* Running master and core nodes on Spot risks total cluster termination if Spot capacity is revoked.
- *C is incorrect:* Lightsail is for simple single-instance VPS, not 10 TB big data EMR analytics.
- *D is incorrect:* Running a 4-hour batch job 24/7 on RIs wastes 20 hours of compute daily.

---

### Question 5 (Domain 2: Design for New Solutions)
An international e-commerce site operates an active-active multi-region web architecture across `us-east-1` and `eu-central-1`. The application uses Amazon Aurora Global Database. Customers in Europe complain that when updating their user profile (a write operation), page load latency is over 350 ms. Profiling shows that the write request is originating in Frankfurt (`eu-central-1`), being forwarded across the transatlantic link to the primary writer in Northern Virginia (`us-east-1`), and waiting for the round-trip acknowledgement.

**How can the solutions architect reduce write latency for European users?**
- A. Enable Aurora write forwarding in `eu-central-1` and configure the web application to use connection pooling with TCP keep-alive over an AWS Direct Connect link.
- B. Redesign the user profile data store to use Amazon DynamoDB Global Tables with local active-active writes in both regions, using last-writer-wins conflict resolution.
- C. Deploy an Amazon ElastiCache cluster in Frankfurt and write user profile updates only to cache without persisting to database.
- D. Create an Aurora Multi-Master cluster spanning both regions.

**Answer:** B  
**Explanation:** Aurora Global Database uses a single primary writer model where all writes must physically execute in the home region (`us-east-1`). Physical speed-of-light propagation across the Atlantic guarantees $\ge 70\text{ ms}$ round-trip latency per query. **Amazon DynamoDB Global Tables** is a true multi-region active-active database that allows local writes to execute in Frankfurt with single-digit millisecond latency, automatically replicating changes asynchronously across regions.  
**Distractor Analysis:**
- *A is incorrect:* Write forwarding still routes the write across the ocean to `us-east-1`, so it cannot eliminate physical latency.
- *C is incorrect:* Storing profile data only in cache without durable persistence causes complete data loss upon cache eviction or restart.
- *D is incorrect:* Aurora does not support cross-region multi-master clusters.

---

### Question 6 (Domain 1: Organizational Complexity)
An enterprise connects 100 AWS accounts to an on-premises network using an AWS Transit Gateway. The security team mandates that production VPCs must NOT be able to communicate with development VPCs under any circumstances, but both production and development VPCs must be able to communicate with a Shared Services VPC and the on-premises data center.

**How should the Transit Gateway routing be configured?**
- A. Create a single Transit Gateway route table and disable route propagation.
- B. Create multiple Transit Gateway route tables: a Production Route Table, a Development Route Table, and a Shared Services Route Table. Associate Production VPC attachments with the Production Route Table and Development VPC attachments with the Development Route Table. Do NOT propagate routes between Production and Development tables.
- C. Use Security Groups to block traffic between VPCs.
- D. Deploy an AWS Network Firewall inside every VPC.

**Answer:** B  
**Explanation:** Transit Gateway route domain segmentation is achieved by creating distinct Transit Gateway route tables. By isolating Production and Development VPC attachments into separate route tables without cross-propagating routes, network isolation is enforced at the Layer 3 routing plane, making inter-environment routing physically impossible.  
**Distractor Analysis:**
- *A is incorrect:* A single route table allows flat routing between all attached VPCs.
- *C is incorrect:* Security Groups cannot reference security group IDs across Transit Gateway attachments.
- *D is incorrect:* Adding firewalls in 100 VPCs is extraordinarily expensive and complex compared to native TGW route table segmentation.

---

### Question 7 (Domain 3: Continuous Improvement)
A company runs a high-throughput microservices application on AWS. An API service writes 50,000 JSON payloads per second to an Amazon S3 bucket. The storage costs for S3 Standard are growing exponentially. Inspection shows that 95% of these payloads are accessed only within the first 48 hours of creation for audit purposes, after which access is extremely rare ($< 1$ query per month).

**Which storage lifecycle strategy optimizes costs while maintaining high availability?**
- A. Transition objects to S3 Standard-Infrequent Access (S3 Standard-IA) immediately upon creation.
- B. Configure an S3 Lifecycle rule to transition objects to **S3 One Zone-IA** after 30 days and to **S3 Glacier Flexible Retrieval** after 90 days.
- C. Use **S3 Intelligent-Tiering** with archive access tiers enabled, or configure an S3 Lifecycle rule transitioning objects to S3 Standard-IA after 30 days (due to the 30-day minimum storage duration) and S3 Glacier Flexible Retrieval after 90 days.
- D. Delete objects after 48 hours.

**Answer:** C  
**Explanation:** S3 Intelligent-Tiering automatically moves objects between frequent, infrequent, and archive access tiers based on changing access patterns without operational overhead or retrieval fees. Alternatively, S3 Lifecycle rules transitioning to S3 Standard-IA and Glacier optimize cold storage costs after the mandatory 30-day minimum duration.  
**Distractor Analysis:**
- *A is incorrect:* S3 Standard-IA has a mandatory minimum billable storage duration of 30 days and incurs retrieval fees.
- *B is incorrect:* S3 One Zone-IA does not provide multi-AZ resilience and is not suitable for critical audit logs.
- *D is incorrect:* Deleting objects after 48 hours violates audit retention requirements.

---

### Question 8 (Domain 4: Migration & Modernization)
A bank is migrating an on-premises mainframe application to AWS. The mainframe produces binary EBCDIC fixed-width transaction files that must be ingested into an Amazon Redshift data warehouse. The transformation process must parse EBCDIC, validate checksums, convert data to UTF-8 Parquet format, and partition files by transaction date. The pipeline must scale elastically to handle bursts of 50 GB files.

**Which AWS service provides the most efficient serverless ETL engine for this workload?**
- A. AWS Glue ETL job running a distributed PySpark script with custom EBCDIC parser libraries.
- B. Amazon EC2 instance running a shell script.
- C. Amazon Athena querying raw EBCDIC files directly in S3.
- D. AWS Lambda function with maximum memory.

**Answer:** A  
**Explanation:** AWS Glue is a fully managed, serverless Spark ETL service designed for large-scale data transformation. Glue automatically provisions worker nodes (DPUs), executes distributed transformations converting EBCDIC to Parquet, and writes partitioned outputs to S3 for Redshift Spectrum ingestion.  
**Distractor Analysis:**
- *B is incorrect:* Single EC2 instances do not scale elastically and create a single point of failure.
- *C is incorrect:* Athena cannot natively query raw binary EBCDIC files without prior conversion.
- *D is incorrect:* Lambda has a 15-minute execution limit and 10 GB ephemeral storage limit, which cannot reliably process 50 GB mainframe dumps.

---

### Question 9 (Domain 2: Design for New Solutions)
A company requires private, secure API communication between a backend microservice in VPC-A and an internal microservice hosted by a third-party vendor in VPC-B. VPC-A and VPC-B have overlapping IP address ranges (`10.0.0.0/16`), and neither company is willing to re-IP their VPC. Communication must be unidirectional from VPC-A to VPC-B over TCP port 443.

**Which networking architecture resolves IP overlap while securing communication?**
- A. Create an AWS Transit Gateway and configure NAT Gateway on both sides.
- B. Deploy an internal Network Load Balancer (NLB) in VPC-B fronting the vendor service, create an **AWS PrivateLink** (VPC Endpoint Service), and create an Interface VPC Endpoint in VPC-A.
- C. Establish an AWS Site-to-Site VPN between the two VPCs.
- D. Establish a VPC Peering connection with policy routing.

**Answer:** B  
**Explanation:** **AWS PrivateLink** uses an Interface VPC Endpoint that allocates an ENI with a private IP address within VPC-A's CIDR block. Traffic travels over the AWS private network to the Network Load Balancer in VPC-B via endpoint service abstraction. Because PrivateLink operates at Layer 4 without network-layer IP routing, overlapping CIDRs are completely supported.  
**Distractor Analysis:**
- *A is incorrect:* Transit Gateway cannot connect VPCs with identical overlapping CIDR blocks without complex multi-NAT routing tables.
- *C is incorrect:* Site-to-Site VPN with overlapping CIDRs requires complex source/destination NAT routers.
- *D is incorrect:* VPC Peering strictly prohibits overlapping IPv4 CIDR blocks.

---

### Question 10 (Domain 1: Organizational Complexity)
An enterprise needs to grant cross-account access allowing an Amazon QuickSight analytics account to read Parquet data stored in an Amazon S3 bucket located in a Central Data Lake account. Access must be managed centrally with column-level and row-level access permissions based on user departments.

**Which AWS service should be used to enforce centralized fine-grained data access control?**
- A. AWS Lake Formation.
- B. Amazon S3 Bucket Policies with IAM condition keys.
- C. AWS Organizations Service Control Policies.
- D. AWS Secrets Manager.

**Answer:** A  
**Explanation:** **AWS Lake Formation** provides centralized, fine-grained access control for data lake tables stored in S3. It allows administrators to define column-level, row-level, and cell-level filtering permissions across multiple accounts, integrating natively with QuickSight and Athena.  
**Distractor Analysis:**
- *B is incorrect:* S3 Bucket Policies can control access at the bucket or prefix level, but cannot filter specific columns or rows inside Parquet files.
- *C is incorrect:* SCPs control maximum account permissions, not row/column level data permissions.
- *D is incorrect:* Secrets Manager manages credentials, not data lake access control.

---

### Question 11 (Domain 2: Design for New Solutions)
A video rendering service executes batch rendering jobs that take between 2 and 6 hours to complete. The rendering tasks can run on containerized worker nodes. If an individual worker instance is interrupted, the job must resume from the last saved frame checkpoint stored in Amazon S3. The platform team wants to minimize compute costs by 70% using EC2 Spot Instances.

**Which orchestration architecture provides resilient Spot batch rendering?**
- A. AWS Batch configured with a compute environment using `SPOT_CAPACITY_OPTIMIZED` instance allocation strategy, diversified across multiple instance families, and integrated with Spot instance interruption warnings.
- B. A single large EC2 On-Demand instance running Docker.
- C. Amazon Elastic Container Service (ECS) with AWS Fargate.
- D. AWS Step Functions invoking Amazon EC2 On-Demand instances via Systems Manager.

**Answer:** A  
**Explanation:** AWS Batch natively supports Spot Instances using the `SPOT_CAPACITY_OPTIMIZED` allocation strategy, which automatically selects Spot pools with the highest capacity and lowest risk of interruption. When coupled with frame checkpointing in S3, interrupted jobs are seamlessly re-queued and completed with up to 90% cost savings.  
**Distractor Analysis:**
- *B is incorrect:* Single On-Demand instances provide zero Spot cost savings and zero resilience.
- *C is incorrect:* Fargate Spot is suitable for short tasks, but does not support custom GPU rendering configurations in the same cost-effective manner as diversified EC2 Spot fleets.
- *D is incorrect:* Step Functions without Batch requires complex custom instance provisioning logic.

---

### Question 12 (Domain 3: Continuous Improvement)
An application tier deployed on Amazon EC2 instances in an Auto Scaling Group experiences traffic spikes during flash sales. Metric analysis reveals that the CPU utilization metric lags behind rapid traffic surges, causing the Auto Scaling Group to scale out 5 minutes too late, leading to dropped customer connections.

**Which scaling policy configuration solves the scaling latency issue?**
- A. Configure a Target Tracking Scaling Policy based on **Application Load Balancer Request Count Per Target (`ALBRequestCountPerTarget`)**, and enable **Predictive Scaling**.
- B. Decrease the ASG cooldown period to 0 seconds.
- C. Switch from EC2 to EC2 Dedicated Hosts.
- D. Increase the instance size to `c6i.32xlarge` permanently.

**Answer:** A  
**Explanation:** ALB Request Count Per Target responds immediately to incoming HTTP traffic volume before CPU utilization spikes on instances. Additionally, **Predictive Scaling** uses machine learning to forecast upcoming traffic patterns based on historical data and pre-provisions capacity minutes ahead of anticipated flash sales.  
**Distractor Analysis:**
- *B is incorrect:* Setting cooldown to 0 can cause runaway scaling and thrashing without solving metric lag.
- *C is incorrect:* Dedicated Hosts do not improve scaling velocity.
- *D is incorrect:* Over-provisioning massive instances permanently inflates operational costs.

---

### Question 13 (Domain 4: Migration & Modernization)
A enterprise must migrate 2 PB of uncompressed archival tape backups to AWS. The tape data is stored in an on-premises physical tape library. The compliance department requires tapes to be archived securely in AWS for 10 years, accessible via standard virtual tape library (VTL) software without rewriting backup software scripts.

**Which AWS solution fulfills this requirement?**
- A. Deploy an **AWS Storage Gateway Tape Gateway (VTL)** on premises. Connect existing backup software (e.g. Commvault, Veeam) via iSCSI to the Tape Gateway, and configure lifecycle rules to automatically archive virtual tapes into **S3 Glacier Flexible Retrieval** and **S3 Glacier Deep Archive**.
- B. Ship physical tapes to an AWS partner facility via courier.
- C. Use AWS Snowball Edge Storage Optimized to copy tapes as raw `.tar` archives.
- D. Deploy AWS DataSync to copy tape drives to Amazon EFS.

**Answer:** A  
**Explanation:** AWS Storage Gateway Tape Gateway presents an industry-standard iSCSI-based Virtual Tape Library (VTL) interface that integrates seamlessly with existing backup software without changing operational workflows. Virtual tapes are automatically uploaded to S3 and archived into S3 Glacier Deep Archive for pennies per terabyte per month.  
**Distractor Analysis:**
- *B is incorrect:* AWS does not offer physical tape intake ingestion.
- *C is incorrect:* Snowball Edge does not emulate an iSCSI VTL tape interface.
- *D is incorrect:* EFS is an expensive POSIX file system, not a cost-effective archival storage target.

---

### Question 14 (Domain 1: Organizational Complexity)
An enterprise has multiple software development teams deploying infrastructure in their own AWS accounts. The CISO mandates that developers must only deploy approved, pre-vetted, hardened AWS CloudFormation templates (e.g., standard VPCs, secure S3 buckets, compliant RDS databases). Developers must not have permissions to author arbitrary infrastructure from scratch.

**Which AWS service provides a self-service catalog of approved CloudFormation templates?**
- A. **AWS Service Catalog**.
- B. AWS Systems Manager Parameter Store.
- C. AWS CodeArtifact.
- D. AWS Marketplace.

**Answer:** A  
**Explanation:** AWS Service Catalog allows organizations to create and manage catalogs of approved IT services and infrastructure templates (products). End users can self-serve and launch compliant resources via Launch Constraints without needing direct IAM administrative access to underlying cloud services.  
**Distractor Analysis:**
- *B is incorrect:* Parameter Store stores configuration strings and secrets, not self-service infrastructure catalogs.
- *C is incorrect:* CodeArtifact manages software packages (npm, pip, maven), not infrastructure templates.
- *D is incorrect:* Marketplace is for third-party commercial software subscriptions.

---

### Question 15 (Domain 2: Design for New Solutions)
A global social networking mobile app allows users to upload profile photos. Uploaded images must be resized into 3 different thumbnail dimensions asynchronously. Image processing must scale from 0 to 50,000 images per minute during viral events. The architecture must be completely serverless and decoupled.

**Which architecture meets these requirements with the LOWEST operational complexity?**
- A. Users upload images directly to an Amazon S3 bucket using S3 presigned URLs. The S3 `ObjectCreated` event sends a notification to an Amazon SQS queue. An AWS Lambda function processes messages from the SQS queue, resizes the images, and stores the thumbnails in a secondary S3 bucket.
- B. Users upload images to an Amazon EC2 instance running Nginx, which saves files to EBS and spawns local Python scripts.
- C. Users upload images to Amazon DynamoDB as binary attributes, triggering a DynamoDB Streams Lambda function.
- D. Images are sent over WebSocket to an AWS AppSync API.

**Answer:** A  
**Explanation:** S3 presigned URLs offload image uploads directly to S3 without traversing compute layers. S3 Event Notifications to SQS buffer upload bursts, and Lambda scales automatically to process thumbnails in parallel, providing a serverless, decoupled, cost-optimized pipeline.  
**Distractor Analysis:**
- *B is incorrect:* EC2 and EBS create compute and storage bottlenecks that cannot scale elastically from 0 to 50,000 images/minute.
- *C is incorrect:* DynamoDB items have a 400 KB limit and are not designed for large image blob storage.
- *D is incorrect:* WebSockets are inefficient for binary multi-megabyte image transfers.
