# AWS Certified Solutions Architect - Professional (SAP-C02) Practice Exam 1

**Duration:** 60 Minutes (25 Full-Length Multi-Paragraph Scenario Questions)  
**Passing Score Target:** $\ge 20 / 25$ (80%)  
**Exam Blueprint Focus:** Domain 1 (Organizational Complexity: 26%), Domain 2 (Design for New Solutions: 29%), Domain 3 (Continuous Improvement: 25%), Domain 4 (Migration & Modernization: 20%)

---

### Question 1 (Domain 1: Organizational Complexity)
An enterprise with 200 AWS accounts organized in AWS Organizations must enforce centralized egress inspection for all outbound Internet traffic across all VPCs. The architecture team has deployed an AWS Network Firewall cluster in a dedicated Central Inspection VPC in `us-east-1` attached to an AWS Transit Gateway (TGW). During testing, application teams in Spoke VPCs report intermittent TCP connection timeouts when initiating HTTPS requests to external SaaS partners. CloudWatch metrics show that packets are reaching the firewall instances, but stateful TCP connections are prematurely terminating after the initial SYN packet.

**Which architectural adjustment must the solutions architect make with the LEAST operational overhead?**
- A. Replace the AWS Network Firewall with an Application Load Balancer configured with cross-zone load balancing enabled.
- B. Enable Appliance Mode on the Transit Gateway VPC attachment in the Central Inspection VPC.
- C. Deploy dedicated NAT Gateways inside every Spoke VPC and remove the Central Inspection VPC attachment.
- D. Create bidirectional VPC peering connections between every Spoke VPC and the Central Inspection VPC, bypassing the Transit Gateway.

**Answer:** B  
**Explanation:** When traffic traverses an AWS Transit Gateway between attachments, the default routing behavior does not guarantee symmetric Availability Zone (AZ) routing for return flows. If Spoke VPC traffic originates in AZ-a, reaches the TGW, and crosses to the Inspection VPC in AZ-a, the return traffic from the internet or target might hash to AZ-b on the return Transit Gateway traversal. Without **Transit Gateway Appliance Mode** enabled, stateful inspection firewalls in AZ-b drop the connection because they never processed the initial SYN handshake. Enabling Appliance Mode on the Central Inspection VPC attachment forces Transit Gateway to route bidirectional traffic symmetrically to the same AZ.  
**Distractor Analysis:**
- *A is incorrect:* An ALB is a reverse proxy for inbound traffic, not a forward egress firewall for arbitrary TCP/HTTPS outbound traffic.
- *C is incorrect:* Violates the governing requirement for centralized inspection and dramatically increases NAT Gateway hourly costs across 200 accounts.
- *D is incorrect:* Full mesh VPC peering creates an unmanageable $O(N^2)$ peering web and does not support transitive centralized inspection.

---

### Question 2 (Domain 2: Design for New Solutions)
A financial SaaS platform processes high-frequency stock trading transactions. The system requires an active-active multi-region database tier across `us-east-1` and `eu-west-1`. Write transactions must have an RPO $< 1$ second, and in the event of an entire AWS region failure, global users must be redirected within 30 seconds with an RTO $< 1$ minute. The database schema relies heavily on complex relational foreign key constraints, stored procedures, and ACID transaction isolation.

**Which architecture meets these requirements with the LOWEST operational overhead?**
- A. Deploy an Amazon Aurora Global Database with the primary writer in `us-east-1` and a cross-region read replica in `eu-west-1`. Enable managed write forwarding in `eu-west-1`. Use Route 53 Application Recovery Controller (ARC) routing controls behind an AWS Global Accelerator.
- B. Deploy Amazon DynamoDB Global Tables with multi-region active-active replication, and use a Lambda abstraction layer to validate relational foreign key constraints.
- C. Deploy self-managed PostgreSQL clusters on EC2 instances across both regions using Bucardo multi-master asynchronous replication.
- D. Deploy Amazon RDS PostgreSQL Multi-AZ in `us-east-1` and configure AWS Database Migration Service (DMS) continuous CDC replication to an RDS instance in `eu-west-1`.

**Answer:** A  
**Explanation:** Amazon Aurora Global Database uses dedicated infrastructure at the storage engine level to replicate data across regions with typical latency under 1 second. Enabling managed write forwarding allows applications in `eu-west-1` to write directly to their local endpoint while Aurora transparently handles the write execution on the primary cluster. Route 53 Application Recovery Controller (ARC) routing controls combined with AWS Global Accelerator provide sub-30-second deterministic failover across edge locations, satisfying the strict RTO constraint.  
**Distractor Analysis:**
- *B is incorrect:* DynamoDB is a NoSQL key-value store; using custom Lambda glue to emulate relational foreign keys and complex ACID transactions introduces massive latency and catastrophic operational overhead.
- *C is incorrect:* Self-managed EC2 databases with Bucardo require high operational maintenance and failover complexity.
- *D is incorrect:* RDS with DMS CDC introduces significant replication lag under high transaction volumes and requires manual DNS and application failover orchestration.

---

### Question 3 (Domain 1: Organizational Complexity)
An enterprise with an on-premises data center in Frankfurt connects to AWS via a 1 Gbps AWS Direct Connect connection terminated at an AWS Direct Connect Gateway attached to a Transit Gateway in `eu-central-1`. The enterprise acquires a new subsidiary that has an on-premises network in London using the identical `10.50.0.0/16` IP address space as the parent company's Frankfurt data center. Both networks must immediately access shared services running in a corporate AWS Shared Services VPC (`172.16.0.0/16`) without IP address collisions and WITHOUT requiring re-IPing of either on-premises network.

**Which solution satisfies these constraints with the LEAST administrative effort?**
- A. Re-IP the London subsidiary's network using automated DHCP migration scripts.
- B. Deploy Private NAT Gateways on the Transit Gateway attachments associated with the London connection to perform bi-directional Source and Destination NAT.
- C. Deploy AWS VPC Lattice in the Shared Services account, expose the shared services through a VPC Lattice Service Network, and configure IAM authentication policies.
- D. Create an Amazon Route 53 Private Hosted Zone with split-horizon DNS routing based on client IP subnets.

**Answer:** C  
**Explanation:** AWS VPC Lattice operates at the application layer (Layer 7) and completely abstracts the underlying network topology, VPC CIDRs, and IP routing. By associating shared services with a VPC Lattice Service Network, workloads from overlapping IP spaces can securely discover and invoke services via standard DNS names and HTTP/HTTPS/gRPC, completely bypassing Layer 3 IP routing conflicts without requiring Private NAT or re-IPing.  
**Distractor Analysis:**
- *A is incorrect:* Re-IPing an entire enterprise network is extremely time-consuming and explicitly violates the requirement.
- *B is incorrect:* While Private NAT Gateway can handle overlapping CIDRs, managing complex bi-directional NAT translation tables and DNS mappings across enterprise services incurs significant operational overhead compared to native VPC Lattice.
- *D is incorrect:* Route 53 split-horizon DNS does not resolve underlying IP packet routing collisions on overlapping subnets.

---

### Question 4 (Domain 3: Continuous Improvement)
A media streaming company operates an application on AWS where 50,000 global users download dynamic, personalized user profile images stored in an Amazon S3 bucket. The monthly AWS bill reveals \$18,000 in S3 `DataTransfer-Out` costs to the Internet. In addition, latency metrics indicate that users in Asia Pacific experience 350 ms download latency because the S3 bucket is hosted in `us-east-1`.

**Which architectural refactoring will reduce costs the MOST while improving download latency?**
- A. Deploy an Amazon CloudFront distribution in front of the S3 bucket, configure Origin Access Control (OAC), and cache image responses at CloudFront edge locations with appropriate `Cache-Control` headers.
- B. Replicate the S3 bucket across 10 AWS regions using S3 Cross-Region Replication (CRR) and use Route 53 Geolocation routing.
- C. Enable Amazon S3 Transfer Acceleration on the bucket and direct client downloads to the accelerate endpoint.
- D. Launch an Auto Scaling fleet of EC2 instances running NGINX caching proxies in `ap-southeast-1`.

**Answer:** A  
**Explanation:** Amazon CloudFront edge caching drastically reduces latency for global viewers by serving cached content from the nearest Point of Presence (PoP). Furthermore, CloudFront pricing for data transfer out to the Internet is substantially lower than S3 direct Internet data transfer, and data transfer from Amazon S3 to Amazon CloudFront is \$0.00 (completely free).  
**Distractor Analysis:**
- *B is incorrect:* S3 CRR incurs storage costs in 10 regions plus inter-region replication transfer fees, dramatically increasing the monthly bill.
- *C is incorrect:* S3 Transfer Acceleration incurs an additional fee per gigabyte on top of standard S3 data transfer out, increasing costs.
- *D is incorrect:* Running dedicated EC2 instances introduces high operational overhead, EC2 hourly compute costs, and maintenance burdens.

---

### Question 5 (Domain 4: Migration & Modernization)
A global enterprise must migrate a critical 45 TB Microsoft SQL Server database from an on-premises Windows Server cluster to AWS. The application SLA permits a MAXIMUM downtime window of 30 minutes during cutover. The on-premises network has a dedicated 1 Gbps Direct Connect connection with 60% average utilization. The target architecture on AWS is Amazon RDS for SQL Server Multi-AZ.

**Which migration strategy achieves the target SLA with the LEAST operational disruption?**
- A. Take a full SQL Server backup (`.bak`), transfer the files over the Direct Connect link to Amazon S3, and use native RDS SQL Server backup restoration (`rds_restore_database`).
- B. Use AWS Database Migration Service (AWS DMS) with Full Load plus Continuous Replication (CDC). Prior to cutover, stop application traffic, allow DMS to replicate the remaining transaction logs, and repoint the connection string to RDS.
- C. Order two AWS Snowball Edge Storage Optimized devices, export the database, ship them to AWS, and perform native restore.
- D. Deploy AWS Application Migration Service (AWS MGN) to migrate the physical SQL Server host directly into an EC2 instance.

**Answer:** B  
**Explanation:** AWS DMS with Change Data Capture (CDC) performs an initial bulk load while the source database remains 100% online, and then continuously captures transaction log deltas. During the maintenance window, replication lag is typically only a few seconds. The cutover simply requires stopping the source writes, letting DMS drain the final seconds of changes, and repointing the application connection string to the Amazon RDS endpoint—well within the 30-minute SLA.  
**Distractor Analysis:**
- *A is incorrect:* Transferring a 45 TB backup file over a shared 1 Gbps link would take approximately 4–5 days, during which the database would either be offline or out of sync.
- *C is incorrect:* Snowball Edge shipping and data loading takes several days and lacks native continuous delta capture for the cutover window.
- *D is incorrect:* MGN migrates the underlying VM/host to EC2, but the target requirement specifies Amazon RDS for SQL Server (managed database).

---

### Question 6 (Domain 2: Design for New Solutions)
An online gaming company is launching a global multiplayer game. Game clients communicate via UDP for real-time player position updates and HTTPS for matchmaking. Game servers run on Amazon EC2 in `us-east-1` and `eu-west-1`. Players complain about high jitter and packet loss during peak Internet congestion hours when connecting from South America and Asia.

**Which architecture provides the FASTEST, most consistent global network performance?**
- A. Deploy Amazon CloudFront distributions in front of the EC2 instances with WebSocket support enabled.
- B. Deploy AWS Global Accelerator with two static Anycast IP addresses, routing client traffic over the private AWS global fiber network directly to Network Load Balancers in each region.
- C. Configure Route 53 Latency-Based Routing pointing to Application Load Balancers in each region.
- D. Deploy an AWS Transit Gateway peering mesh between all regions and have game clients connect via AWS Client VPN.

**Answer:** B  
**Explanation:** AWS Global Accelerator provides two static Anycast IP addresses that act as single fixed entry points. Global Accelerator ingests UDP and TCP traffic at the nearest AWS edge location and routes it over the congestion-free, private AWS global fiber backbone directly to the nearest healthy application endpoint (NLB or EC2). This drastically reduces latency, packet loss, and jitter compared to traversing the public Internet.  
**Distractor Analysis:**
- *A is incorrect:* CloudFront only supports HTTP, HTTPS, and WebSocket traffic; it does NOT support arbitrary UDP protocols required for real-time game positioning.
- *C is incorrect:* Route 53 Latency-Based Routing resolves DNS to regional ALBs over the public Internet, subjecting packets to public Internet congestion and routing hops.
- *D is incorrect:* Client VPN requires client authentication, tunnels, and introduces severe latency overhead unsuited for gaming.

---

### Question 7 (Domain 1: Organizational Complexity)
A pharmaceutical enterprise operates a multi-account structure with 50 AWS accounts under AWS Organizations. The chief information security officer (CISO) mandates that:
1. No IAM user or role—including account administrators and the root user of any member account—can disable AWS CloudTrail or delete Amazon S3 log buckets.
2. Compliance auditing tools in the Security Audit account must be able to inspect resource configurations across all accounts without being blocked.

**Which mechanism enforces these controls with the LEAST administrative effort?**
- A. Apply a Service Control Policy (SCP) at the Organization Root level with an explicit `Deny` on `cloudtrail:StopLogging`, `cloudtrail:DeleteTrail`, and `s3:DeleteBucket*`, with a `Condition` excluding the `SecurityAudit` account ARN.
- B. Create an IAM Permissions Boundary in every member account and attach it to all IAM roles.
- C. Configure AWS Config Rules in every account that trigger an AWS Lambda remediation function to re-enable CloudTrail.
- D. Deploy an AWS CloudFormation StackSet with AWS Systems Manager State Manager associations to re-enable CloudTrail every hour.

**Answer:** A  
**Explanation:** Service Control Policies (SCPs) apply guardrails across member accounts that cannot be overridden by any IAM user, role, or even the member account root user. Applying an SCP at the Organization Root level with a `Deny` statement ensures uniform enforcement. The `Condition` block can exempt specific central security roles or accounts (`StringNotEquals: {"aws:PrincipalAccount": "111122223333"}`).  
**Distractor Analysis:**
- *B is incorrect:* Permissions Boundaries must be attached to every role manually or via custom automation, can be circumvented if not strictly enforced, and do not restrict the account root user.
- *C is incorrect:* Reactive remediation means CloudTrail could remain disabled for minutes before detection, leaving an audit blind spot.
- *D is incorrect:* StackSets and SSM associations are reactive and do not prevent administrative tampering between hourly runs.

---

### Question 8 (Domain 2: Design for New Solutions)
An IoT fleet management platform ingests sensor telemetry from 500,000 commercial delivery trucks. Each truck emits a 1 KB JSON payload every 5 seconds (100,000 requests/sec total). The platform requires:
1. Ingestion with zero data loss.
2. Real-time anomaly detection with $<2$ second latency.
3. Long-term raw storage in Amazon S3 in Apache Parquet format partitioned by year, month, and day for cost-effective Amazon Athena queries.

**Which architecture meets these requirements with the LEAST operational overhead?**
- A. Ingest data via an Application Load Balancer into an EC2 Auto Scaling fleet running Apache Kafka; write custom consumer scripts to upload Parquet files to S3.
- B. Ingest telemetry into Amazon Kinesis Data Streams. Attach an Amazon Managed Service for Apache Flink application for real-time anomaly detection. Attach an Amazon Data Firehose delivery stream configured with dynamic partitioning and inline Apache Parquet format conversion to deliver data to Amazon S3.
- C. Ingest telemetry into Amazon SQS Standard queues. Trigger AWS Lambda functions in batches to parse anomalies and save files to S3.
- D. Stream telemetry directly into Amazon DynamoDB, and use DynamoDB Streams to trigger Lambda functions that write Parquet files to S3.

**Answer:** B  
**Explanation:** Amazon Kinesis Data Streams provides high-throughput, ordered streaming ingestion capable of scaling to hundreds of thousands of records per second. Amazon Managed Service for Apache Flink performs sub-second stateful stream processing and anomaly detection natively. Amazon Data Firehose can ingest directly from Kinesis Data Streams, perform automated conversion from JSON to Apache Parquet using AWS Glue Data Catalog schemas, and apply dynamic partitioning before storing files in S3—all with zero infrastructure management.  
**Distractor Analysis:**
- *A is incorrect:* Managing self-hosted Kafka clusters and custom consumer scripts requires substantial operational overhead and patching.
- *B is incorrect:* SQS does not support real-time streaming analytics at this scale cost-effectively and does not provide native Parquet conversion.
- *D is incorrect:* Writing 100,000 WCU directly to DynamoDB would incur enormous database costs compared to Kinesis and Firehose ingestion.

---

### Question 9 (Domain 3: Continuous Improvement)
An enterprise runs an enterprise resource planning (ERP) system on Amazon EC2 instances backed by Provisioned IOPS SSD (`io2`) Amazon EBS volumes. Performance monitoring shows that while the volumes were provisioned for 20,000 IOPS, the workload consistently peaks at only 3,500 IOPS with a throughput of 200 MB/s. The storage bill accounts for 45% of the total monthly cloud spend.

**Which action reduces storage costs the MOST without impacting system performance?**
- A. Modify the EBS volume type from `io2` to General Purpose SSD (`gp3`) and configure 3,500 IOPS and 200 MB/s throughput using EBS Elastic Volumes.
- B. Take an EBS snapshot of all volumes, delete the `io2` volumes, and restore them as Cold HDD (`sc1`) volumes.
- C. Migrate the ERP database to Amazon EFS with One Zone storage.
- D. Attach multiple Throughput Optimized HDD (`st1`) volumes in a RAID 0 configuration.

**Answer:** A  
**Explanation:** `gp3` volumes provide baseline performance of 3,000 IOPS and 125 MB/s throughput for free, and allow provisioning additional IOPS up to 16,000 and throughput up to 1,000 MB/s independently of volume size at a fraction of the cost of `io2` volumes. Elastic Volumes allows modifying the volume type, IOPS, and throughput online without unmounting the filesystem or taking instances offline.  
**Distractor Analysis:**
- *B is incorrect:* `sc1` Cold HDD is designed for infrequent sequential access and cannot deliver the low-latency random IOPS required by an ERP database.
- *C is incorrect:* Amazon EFS is an NFS network file system unsuited for low-latency relational ERP block storage.
- *D is incorrect:* `st1` is magnetic HDD storage that cannot provide sub-millisecond transaction latency.

---

### Question 10 (Domain 4: Migration & Modernization)
A bank operates an on-premises mainframe running legacy COBOL applications and batch processing. The leadership team mandates an aggressive cloud migration strategy to decompose the mainframe into modular Java Spring Boot microservices deployed on AWS. The solution must provide automated code transformation, dependency analysis, and complete runtime environment management.

**Which AWS service directly satisfies this modernization requirement?**
- A. AWS Application Migration Service (AWS MGN)
- B. AWS Mainframe Modernization with the AWS Blu Age engine
- C. AWS Database Migration Service (AWS DMS)
- D. AWS Serverless Application Model (AWS SAM)

**Answer:** B  
**Explanation:** AWS Mainframe Modernization service with the AWS Blu Age toolset is purpose-built to accelerate mainframe application migration. It provides automated refactoring that transforms legacy COBOL, CICS, and JCL assets into modern Java/Spring Boot microservices, while validating functional equivalence and deploying into a managed AWS runtime.  
**Distractor Analysis:**
- *A is incorrect:* AWS MGN performs block-level server rehosting (lift-and-shift) of x86 Windows and Linux VMs; it cannot execute automated COBOL-to-Java code refactoring.
- *C is incorrect:* AWS DMS migrates relational databases, not application code or mainframe execution environments.
- *D is incorrect:* AWS SAM is a framework for serverless infrastructure-as-code, not a legacy code transformation engine.

---

### Question 11 (Domain 1: Organizational Complexity)
An organization has deployed a shared AWS Transit Gateway across multiple AWS accounts. Workload accounts in `eu-west-1` need to securely resolve DNS names of internal services hosted in the Corporate on-premises data center (`corp.example.com`), while on-premises client machines need to resolve private AWS hosted zones (`cloud.example.com`).

**Which architecture provides seamless bidirectional hybrid DNS resolution with the MINIMAL administrative overhead?**
- A. Deploy EC2 instances running BIND DNS forwarders in two Availability Zones in each Spoke VPC.
- B. In a Central Networking VPC, deploy Route 53 Resolver Inbound Endpoints and Outbound Endpoints. Configure an Outbound Resolver Rule forwarding `corp.example.com` to on-premises DNS servers, and share this rule organization-wide via AWS RAM. On the on-premises DNS servers, configure conditional forwarders for `cloud.example.com` pointing to the Route 53 Inbound Endpoint IPs.
- C. Use Amazon Route 53 Public Hosted Zones with DNSSEC enabled and IP whitelist firewalls.
- D. Create a peering connection between all Spoke VPCs and on-premises routers, configuring static `/etc/hosts` mappings via Systems Manager.

**Answer:** B  
**Explanation:** Route 53 Resolver Inbound and Outbound Endpoints provide managed, highly available hybrid DNS integration without managing EC2 instances. Inbound Endpoints receive DNS queries from on-premises and resolve private AWS zones. Outbound Endpoints forward queries for on-premises domains to corporate DNS servers. Sharing the Outbound Resolver Rule across all accounts via AWS Resource Access Manager (RAM) eliminates the need to configure DNS endpoints in each spoke VPC.  
**Distractor Analysis:**
- *A is incorrect:* Managing self-hosted BIND servers on EC2 across multiple VPCs requires patching, scaling, and high operational overhead.
- *C is incorrect:* Public hosted zones expose internal corporate architecture to the public internet.
- *D is incorrect:* Static `/etc/hosts` mappings are impossible to maintain across dynamic cloud fleets.

---

### Question 12 (Domain 2: Design for New Solutions)
A healthcare company processes sensitive electronic health records (EHR). The compliance policy mandates that:
1. Sensitive data must be encrypted with a customer-managed key where key access is strictly audited.
2. Even if an attacker gains unauthorized `AdministratorAccess` to the AWS management console, they must NOT be able to delete the encrypted backups for at least 365 days.

**Which architectural combination fulfills these requirements?**
- A. Store backups in Amazon S3 with SSE-S3 encryption and apply an S3 Bucket Policy denying `s3:DeleteObject`.
- B. Store backups in Amazon S3 with SSE-KMS using an AWS KMS Customer Managed Key (CMK). Configure AWS Backup with an **AWS Backup Vault Lock in Compliance Mode** specifying a minimum retention period of 365 days.
- C. Store backups on EBS volumes and take snapshots with default AWS EBS encryption.
- D. Configure S3 Glacier Flexible Retrieval with a Glacier Vault Access Policy.

**Answer:** B  
**Explanation:** AWS Backup Vault Lock in **Compliance Mode** enforces non-deletable backups that cannot be modified, deleted, or shortened by any user, including the root user of the AWS account or AWS Support, once the mandatory cooling-off period expires. Utilizing a KMS Customer Managed Key (CMK) ensures cryptographic control, access separation, and detailed audit logging via CloudTrail.  
**Distractor Analysis:**
- *A is incorrect:* A bucket policy can be edited or deleted by anyone with `AdministratorAccess` or the account root user.
- *C is incorrect:* Standard EBS snapshots can be deleted by an administrator role.
- *D is incorrect:* Glacier Vault Access Policies without a locked Vault Lock Policy can be modified by account administrators.

---

### Question 13 (Domain 3: Continuous Improvement)
An enterprise runs a critical microservices application on Amazon Elastic Container Service (Amazon ECS) on AWS Fargate. During peak traffic hours, tasks scale out rapidly, but users report increased HTTP 500 errors during new task deployments. Investigation shows that the backend tasks take 90 seconds to bootstrap, download machine learning models, and establish database connection pools. However, the Application Load Balancer begins routing production traffic to the new tasks within 15 seconds of launch.

**How should the solutions architect resolve this deployment failure?**
- A. Increase the task CPU allocation in the ECS Task Definition to 4 vCPUs.
- B. Configure a container HealthCheck in the ECS Task Definition and increase the `deregistration_delay.timeout_seconds` on the ALB target group.
- C. Configure a container HealthCheck in the ECS Task Definition, and configure the ALB Target Group health check path, increasing the `healthy_threshold_count` and setting an appropriate `health_check_grace_period_seconds` on the ECS Service.
- D. Switch from AWS Fargate to EC2 launch type with Spot instances.

**Answer:** C  
**Explanation:** When tasks take a long time to bootstrap, the ECS Service `health_check_grace_period_seconds` must be set to a duration greater than the initialization time (e.g., 120 seconds). This instructs ECS to ignore unhealthy ALB target group checks during the bootstrapping window, preventing ECS from prematurely declaring the task unhealthy and killing it while it is loading dependencies.  
**Distractor Analysis:**
- *A is incorrect:* Adding CPU does not resolve application bootstrap synchronization with the ALB health check.
- *B is incorrect:* `deregistration_delay` controls connection draining during task shutdown, not task startup.
- *D is incorrect:* Changing the compute provider does not resolve health check timing mismatches.

---

### Question 14 (Domain 2: Design for New Solutions)
A financial services firm is building a serverless workflow to process mortgage applications. The application requires coordinating multiple distributed steps: credit score check, document OCR scanning, fraud evaluation, and human underwriter review. The underwriter review can take up to 5 business days to complete. If any check fails, the workflow must execute compensation logic to notify the applicant and update the audit log.

**Which service should be the core orchestrator for this workflow?**
- A. Amazon SQS FIFO queue with message visibility timeout set to 5 days.
- B. AWS Step Functions Standard Workflow using task tokens (`.waitForTaskToken`) for the human review step.
- C. AWS Step Functions Express Workflow executing inside an AWS Lambda function loop.
- D. Amazon Simple Notification Service (SNS) with message filtering policies.

**Answer:** B  
**Explanation:** AWS Step Functions Standard Workflows can run for up to 1 year and provide built-in state management, branching, error handling, and saga-pattern compensation execution. Using task tokens (`.waitForTaskToken`), Step Functions can pause the workflow indefinitely while awaiting an external callback from the human underwriter portal before resuming.  
**Distractor Analysis:**
- *A is incorrect:* SQS message visibility timeout maximum is 12 hours (43,200 seconds); it cannot wait 5 days.
- *C is incorrect:* Express Workflows have a maximum execution duration of 5 minutes; they cannot handle multi-day approvals.
- *D is incorrect:* SNS is a pub/sub messaging service, not a stateful workflow orchestrator.

---

### Question 15 (Domain 1: Organizational Complexity)
An enterprise with 30 AWS accounts needs to allow developers in workload accounts to securely deploy EC2 instances into a centrally managed VPC owned by the Networking account. The developers must be able to launch instances and manage their own local security groups, but must NOT have permissions to modify VPC route tables, Internet Gateways, NAT Gateways, or subnets.

**Which solution achieves this model with the LEAST complexity?**
- A. Create VPC Peering connections between every developer account VPC and the central VPC.
- B. Use AWS Resource Access Manager (RAM) in the central Networking account to share the subnets with the Organization or specific workload accounts.
- C. Provision Transit Gateway attachments for each developer account and configure static routing.
- D. Give developers IAM permissions in the central Networking account via cross-account role assumption.

**Answer:** B  
**Explanation:** **VPC Sharing** using AWS Resource Access Manager (RAM) allows the owner account (Networking) to manage the VPC structure (CIDRs, subnets, route tables, gateways) while sharing specific subnets with participant accounts. Participant accounts can view the subnets and launch resources (EC2, ECS, RDS) inside them, and participant accounts create and control their own local security groups.  
**Distractor Analysis:**
- *A is incorrect:* VPC Peering does not allow multiple accounts to share the same subnet and requires complex IP space management.
- *C is incorrect:* Transit Gateway connects distinct VPCs; it does not share common subnets directly.
- *D is incorrect:* Cross-account role assumption into the central network account violates least privilege by granting developers access to the central network infrastructure.

---

### Question 16 (Domain 4: Migration & Modernization)
An insurance company must migrate 200 TB of raw customer policy data from an on-premises Dell EMC Isilon NAS (NFS shares) to AWS. The network connection is a shared 500 Mbps connection. The data contains over 40 million small PDF and TIFF files. Previous attempts using standard `rsync` over VPN took weeks to compute directory listings and failed due to timeouts.

**Which solution should the architect implement to accelerate the data transfer and guarantee data integrity?**
- A. Order two AWS Snowball Edge Storage Optimized devices, install the AWS DataSync agent on the Snowball devices, and copy the files locally.
- B. Deploy an AWS DataSync agent as a VM on-premises, configure a task pointing to the on-premises NFS server as source and Amazon S3 as destination, and enable verification of all transferred data.
- C. Write a multi-threaded Python script using the AWS CLI `s3 sync` command with 64 threads.
- D. Deploy an AWS Storage Gateway File Gateway on-premises and mount the NFS share.

**Answer:** B  
**Explanation:** AWS DataSync uses a purpose-built network protocol and architecture specifically designed to parallelize file metadata scanning and transfer. It can transfer millions of small files up to 10x faster than open-source tools like `rsync`. DataSync performs automated end-to-end data integrity verification and preserves POSIX file permissions and metadata.  
**Distractor Analysis:**
- *A is incorrect:* The DataSync agent is deployed on the host hypervisor to transfer over network; you do not install DataSync agents onto Snowball devices for this workflow.
- *C is incorrect:* Python CLI scripts still suffer from high metadata latency and OS filesystem overhead over high-latency WAN links.
- *D is incorrect:* Storage Gateway is a hybrid cloud cache, not an optimized bulk data migration tool for 40 million files.

---

### Question 17 (Domain 3: Continuous Improvement)
A company runs a high-volume REST API on Amazon ECS with AWS Fargate backed by an Amazon Aurora PostgreSQL database. During morning traffic surges, database CPU spikes to 100%, and applications throw connection timeout errors. Analysis indicates that 90% of the queries are identical, read-only queries fetching product catalog metadata that changes only once per day.

**Which combination of actions will resolve the database saturation with the LEAST latency?**
- A. Add 5 Aurora Read Replicas to the cluster and use the cluster reader endpoint.
- B. Deploy an Amazon ElastiCache for Redis cluster in front of the database using a cache-aside pattern with a TTL of 24 hours.
- C. Upgrade the Aurora DB instance class from `db.r6g.xlarge` to `db.r6g.16xlarge`.
- D. Enable Aurora Serverless v2 auto-scaling up to 128 ACUs.

**Answer:** B  
**Explanation:** Caching identical, static product catalog metadata in an in-memory cache like Amazon ElastiCache for Redis offloads the read traffic completely from the relational database. Redis provides microsecond response times, eliminating the CPU bottleneck at the database tier at a significantly lower cost than scaling the database cluster.  
**Distractor Analysis:**
- *A is incorrect:* While read replicas help with distributed read queries, sending millions of identical queries still incurs relational database parsing and network overhead compared to microsecond in-memory caching.
- *C is incorrect:* Upgrading instance size is vertically scaling an expensive database to solve a problem easily addressed by caching.
- *D is incorrect:* Aurora Serverless v2 will scale up to handle load, but will result in high ongoing database costs compared to in-memory caching.

---

### Question 18 (Domain 2: Design for New Solutions)
A company hosts a web application behind an Application Load Balancer (ALB). An external security audit requires that all HTTP traffic must be inspected for SQL injection and cross-site scripting (XSS). Additionally, the solution must block requests from known malicious IP addresses published daily by third-party threat intelligence feeds, and rate-limit individual client IP addresses to 2,000 requests per 5 minutes.

**Which solution satisfies all requirements with the LEAST operational overhead?**
- A. Deploy an EC2 fleet running HAProxy and Snort IDS between the ALB and the web tier.
- B. Associate an AWS WAF Web ACL with the Application Load Balancer. Add the AWS-managed rule groups `AWSManagedRulesCommonRuleSet` and `AWSManagedRulesSQLiRuleSet`. Create a rate-based rule with a limit of 2,000 requests per 5 minutes, and configure an IP set updated automatically via an AWS Lambda function consuming the threat feed.
- C. Deploy Amazon GuardDuty and configure automated NACL blocking via AWS Lambda.
- D. Enable AWS Shield Advanced on the ALB and configure custom CloudWatch alarms.

**Answer:** B  
**Explanation:** AWS WAF integrates directly with ALBs without additional infrastructure. The AWS Managed Rules handle common vulnerabilities (SQLi and XSS) out-of-the-box. WAF rate-based rules natively enforce request thresholds per client IP, and WAF IP Sets can be updated programmatically via Lambda to ingest third-party threat intelligence feeds.  
**Distractor Analysis:**
- *A is incorrect:* Self-managing EC2 proxies and Snort rules requires high operational overhead, scaling, and patching.
- *C is incorrect:* Network ACLs have strict rule limits (default 20 rules, max 40) and are not designed for large dynamic IP blocklists.
- *D is incorrect:* AWS Shield Advanced focuses on Layer 3/4 and Layer 7 DDoS mitigation; it does not replace WAF for application-layer SQLi/XSS pattern inspection.

---

### Question 19 (Domain 1: Organizational Complexity)
An enterprise has multiple AWS accounts in an organization. The security team discovers that developers are launching Amazon EC2 instances with public IP addresses attached in non-default VPCs, exposing SSH port 22 directly to the Internet. The CISO mandates that:
1. No EC2 instance may be launched with a public IPv4 address.
2. Existing instances with public IPs must be flagged immediately.
3. Developers must access private instances using AWS Systems Manager Session Manager without open inbound firewall ports.

**Which combination of actions fulfills this mandate?**
- A. Apply a Service Control Policy (SCP) denying `ec2:RunInstances` if `ec2:AssociatePublicIpAddress` is true. Deploy an AWS Config rule `ec2-instance-no-public-ip` across the organization. Ensure instances have the `AmazonSSMManagedInstanceCore` policy attached and access SSM via private VPC endpoints or NAT.
- B. Delete all Internet Gateways across all VPCs in all accounts.
- C. Create a custom IAM policy denying `ec2:DescribeInstances` to developers.
- D. Configure an Amazon EventBridge rule that terminates any newly launched EC2 instance.

**Answer:** A  
**Explanation:** An SCP with a condition checking `ec2:AssociatePublicIpAddress: "true"` prevents the creation of instances with public IPs across all member accounts. The AWS Config managed rule continuously audits and flags existing non-compliant instances. Systems Manager Session Manager allows secure command-line shell access over outbound HTTPS (port 443) via SSM endpoints, requiring zero open inbound ports or public IP addresses.  
**Distractor Analysis:**
- *B is incorrect:* Deleting all Internet Gateways breaks outbound egress traffic for applications (e.g., calling external partner APIs).
- *C is incorrect:* Denying `DescribeInstances` does not prevent instance creation.
- *D is incorrect:* Terminating instances reactively creates disruption and does not prevent the launch event.

---

### Question 20 (Domain 4: Migration & Modernization)
A company is refactoring an on-premises monolithic application into microservices on AWS. As individual microservices are built and deployed into separate AWS accounts, the enterprise needs to transparently route customer requests to either the legacy monolith on-premises or the new AWS microservices based on the URL request path (e.g., `/orders/*` goes to the new service, while everything else goes to the monolith). The solution must handle cross-account networking automatically.

**Which service should be used to implement this pattern?**
- A. AWS Direct Connect Gateway
- B. AWS Migration Hub Refactor Spaces
- C. AWS Application Migration Service
- D. AWS Global Accelerator

**Answer:** B  
**Explanation:** **AWS Migration Hub Refactor Spaces** is the native service designed to implement the Strangler Fig pattern for microservices refactoring. It automatically orchestrates cross-account networking (using AWS Transit Gateway and RAM) and provisions an Amazon API Gateway with path-based routing rules that transparently forward client traffic to either legacy on-premises workloads or new containerized/serverless microservices across different AWS accounts.  
**Distractor Analysis:**
- *A is incorrect:* Direct Connect Gateway provides Layer 3 network connectivity; it cannot perform Layer 7 path-based routing or service decomposition.
- *C is incorrect:* AWS MGN is a block-level lift-and-shift migration tool for whole servers.
- *D is incorrect:* Global Accelerator routes traffic at Layer 4; it does not orchestrate cross-account microservice refactoring architectures.

---

### Question 21 (Domain 2: Design for New Solutions)
A company runs a multi-tier web application on AWS. The frontend is hosted on Amazon S3 and distributed via Amazon CloudFront. The backend consists of REST APIs running on AWS Lambda behind an Amazon API Gateway. Security policy mandates that:
1. End-users must authenticate using corporate SAML 2.0 credentials (Okta).
2. The Lambda functions must only be callable through API Gateway.
3. The S3 origin bucket must be accessible ONLY through CloudFront.

**Which architectural combination fulfills these requirements?**
- A. Configure an Amazon Cognito User Pool federated with Okta SAML 2.0. Attach a Cognito Authorizer to the API Gateway. Configure an S3 Bucket Policy granting access only to the CloudFront Origin Access Control (OAC) principal.
- B. Configure Okta directly inside the Lambda code. Use an S3 bucket with public website hosting enabled and an IP whitelist.
- C. Deploy an AWS Client VPN endpoint and require users to establish a VPN before browsing.
- D. Use IAM user credentials stored in the browser local storage to call API Gateway with AWS SigV4.

**Answer:** A  
**Explanation:** Amazon Cognito User Pools natively federate with external SAML 2.0 identity providers like Okta, issuing JWT tokens that can be validated at the API Gateway layer using a native Cognito Authorizer. For Amazon S3, configuring **Origin Access Control (OAC)** and restricting the bucket policy to the CloudFront distribution service principal (`cloudfront.amazonaws.com`) ensures the bucket cannot be accessed directly or publicly.  
**Distractor Analysis:**
- *B is incorrect:* Handling authentication in Lambda code increases execution duration and cost, and public S3 website hosting violates security best practices.
- *C is incorrect:* Requiring client VPN for public or corporate web users introduces massive overhead and poor user experience.
- *D is incorrect:* Storing long-lived IAM credentials in browser local storage is a severe security vulnerability.

---

### Question 22 (Domain 3: Continuous Improvement)
An enterprise has multiple workload VPCs that need access to an Amazon S3 bucket containing 500 TB of shared analytical datasets. Currently, all EC2 instances in private subnets access S3 via NAT Gateways. The monthly AWS cost for NAT Gateway data processing for S3 access has reached \$35,000.

**What is the MOST cost-effective architectural change?**
- A. Deploy an S3 Interface VPC Endpoint (AWS PrivateLink) in each VPC.
- B. Deploy an S3 Gateway VPC Endpoint in each VPC route table.
- C. Configure instances to access S3 over public IP addresses.
- D. Migrate the data to Amazon EFS with Infrequent Access tiering.

**Answer:** B  
**Explanation:** **Gateway VPC Endpoints** for Amazon S3 (and DynamoDB) are provided by AWS at **zero cost** (\$0.00 hourly charge and \$0.00 data processing charge). Adding an S3 Gateway Endpoint to the private route tables directs all S3 traffic privately across the internal AWS network, completely bypassing the NAT Gateway and reducing the \$35,000/month cost to \$0.  
**Distractor Analysis:**
- *A is incorrect:* S3 Interface Endpoints (PrivateLink) incur hourly endpoint fees plus \$0.01 per GB data processing fees, making them significantly more expensive than Gateway Endpoints for high-throughput VPC internal traffic.
- *C is incorrect:* Attaching public IPs to private instances violates security architecture.
- *D is incorrect:* Migrating 500 TB to EFS would dramatically increase storage and data access costs.

---

### Question 23 (Domain 1: Organizational Complexity)
A multinational bank needs to deploy an internal API in AWS that will be consumed by 40 external commercial partners. Each partner operates their own AWS account. The network architecture must meet these criteria:
1. Partners must not have direct IP network access to the bank's VPC.
2. Partner CIDR blocks may overlap with each other and with the bank's CIDR block.
3. Communication must stay entirely on the AWS private network without traversing the public Internet.

**Which solution satisfies these requirements?**
- A. Create VPC Peering connections with all 40 partner VPCs.
- B. Deploy an internal Network Load Balancer (NLB) in the bank's VPC in front of the API, create an **AWS PrivateLink Endpoint Service**, and grant permissions to the partner AWS account IDs to create Interface VPC Endpoints in their VPCs.
- C. Establish an AWS Transit Gateway and attach all 40 partner VPCs via cross-account TGW attachments.
- D. Deploy an Internet-facing Application Load Balancer with mutual TLS (mTLS).

**Answer:** B  
**Explanation:** **AWS PrivateLink** allows service providers to securely expose services to consumers across accounts via an Endpoint Service backed by an NLB. The consumer creates an Interface VPC Endpoint in their own subnet. PrivateLink operates via unidirectional NAT/proxy mechanisms at the network interface layer, meaning overlapping IP CIDRs between provider and consumers are completely irrelevant, and no direct VPC routing or peering is established.  
**Distractor Analysis:**
- *A is incorrect:* VPC Peering cannot be established between overlapping IP CIDR blocks.
- *C is incorrect:* Transit Gateway routing fails when connecting VPCs with overlapping CIDRs without complex NAT.
- *D is incorrect:* An Internet-facing ALB routes traffic over the public Internet, violating the private network requirement.

---

### Question 24 (Domain 4: Migration & Modernization)
A logistics firm is planning a migration of 350 on-premises Windows and Linux virtual machines running in a VMware vSphere environment to Amazon EC2. The IT director requires continuous replication of block-level changes to minimize the cutover maintenance window to under 15 minutes, with non-disruptive pre-cutover testing.

**Which AWS service is designed specifically for this lift-and-shift workload migration?**
- A. AWS Database Migration Service (AWS DMS)
- B. AWS Application Migration Service (AWS MGN)
- C. AWS DataSync
- D. AWS Elastic Disaster Recovery (AWS DRS)

**Answer:** B  
**Explanation:** **AWS Application Migration Service (AWS MGN)** is the primary AWS service for lift-and-shift migrations. It continuously replicates block-level data from physical, virtual, or cloud servers into a low-cost staging area in AWS. It supports launching test instances in isolated subnets for non-disruptive validation, and allows final cutover with minimal downtime.  
**Distractor Analysis:**
- *A is incorrect:* AWS DMS migrates relational databases, not complete operating system servers.
- *C is incorrect:* DataSync migrates file systems and object storage, not block-level operating system disks for server migration.
- *D is incorrect:* AWS DRS is designed for ongoing business continuity and disaster recovery, whereas AWS MGN is optimized specifically for one-time workload migrations.

---

### Question 25 (Domain 2: Design for New Solutions)
An online retail platform experiences severe traffic surges during flash sales. The application tier runs on Amazon EC2 instances inside an Auto Scaling Group behind an Application Load Balancer. During a recent sale, the database experienced write locks, causing application response times to increase to 45 seconds and triggering cascading instance termination by Auto Scaling health checks.

**Which architectural pattern should the solutions architect implement to decouple the web tier from the database and prevent cascading failures?**
- A. Replace the Auto Scaling Group with an AWS Lambda function with maximum concurrency set to 10.
- B. Place an Amazon SQS queue between the web tier and backend worker instances, and configure the Auto Scaling Group for the workers to scale based on the `ApproximateNumberOfMessagesVisible` metric divided by the number of instances (backlog per instance).
- C. Increase the ALB health check timeout to 60 seconds and decrease the healthy threshold to 1.
- D. Convert the database to an Amazon S3 bucket using Athena for transactional inserts.

**Answer:** B  
**Explanation:** Decoupling synchronous write operations using an Amazon SQS queue absorbs traffic spikes by buffering incoming requests. Backend worker fleets can then process orders asynchronously at a sustainable rate. Scaling the worker Auto Scaling Group based on the backlog per instance metric (`ApproximateNumberOfMessagesVisible / NumberOfInstances`) ensures appropriate scaling based on work volume rather than CPU spikes.  
**Distractor Analysis:**
- *A is incorrect:* Restricting Lambda concurrency to 10 will reject 99% of customer requests with HTTP 429 throttling errors.
- *C is incorrect:* Masking health check timeouts does not solve underlying database saturation and lock contention.
- *D is incorrect:* S3 and Athena are analytical querying tools, not ACID-compliant transactional OLTP engines.
