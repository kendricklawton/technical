# AWS Certified Solutions Architect - Professional (SAP-C02) Practice Exam 2

**Duration:** 60 Minutes (25 Full-Length Multi-Paragraph Scenario Questions)  
**Passing Score Target:** $\ge 20 / 25$ (80%)  
**Exam Blueprint Focus:** Domain 1 (Organizational Complexity: 26%), Domain 2 (Design for New Solutions: 29%), Domain 3 (Continuous Improvement: 25%), Domain 4 (Migration & Modernization: 20%)

---

### Question 1 (Domain 1: Organizational Complexity)
An enterprise with an on-premises data center and multiple AWS accounts requires a high-availability Direct Connect deployment. The company provisions two 10 Gbps Direct Connect dedicated connections terminating at two distinct Direct Connect locations (Location A and Location B) in the same AWS Region. The network engineer establishes a Transit VIF on each connection to a centralized Direct Connect Gateway (DXGW) attached to an AWS Transit Gateway. The routing policy requires that Location A serve as the active link for all traffic, while Location B acts as a passive standby.

**Which BGP configuration ensures that return traffic from AWS to the on-premises data center strictly follows this active/passive design?**
- A. Configure the on-premises router at Location A to prepend the corporate AS number 3 times to BGP route advertisements sent to AWS.
- B. Configure the on-premises router at Location A to attach the BGP community `7224:7300` (high local preference) and Location B to attach the BGP community `7224:7100` (low local preference) to prefixes advertised to the DXGW.
- C. Configure the on-premises router at Location B to set a higher BGP MED attribute than Location A.
- D. Advertise a shorter prefix (e.g. `10.0.0.0/16`) from Location A and more specific prefixes (e.g. `10.0.1.0/24`) from Location B.

**Answer:** B  
**Explanation:** AWS Direct Connect supports specific BGP communities to control route preference on traffic exiting AWS towards on-premises:
- `7224:7100` = Low local preference (AWS assigns local preference 100)
- `7224:7200` = Medium local preference (AWS assigns local preference 200)
- `7224:7300` = High local preference (AWS assigns local preference 300)
By tagging the BGP routes advertised from Location A with `7224:7300` and Location B with `7224:7100`, AWS routers will strictly prefer Location A for all egress flows.  
**Distractor Analysis:**
- *A is incorrect:* AS-path prepending increases the AS path length, which makes that path *less* preferred. Prepending from Location A would make it the passive link.
- *C is incorrect:* Local preference takes precedence over MED in the BGP decision hierarchy.
- *D is incorrect:* Longest prefix matching rules dictate that traffic will always follow more specific routes (`/24`), which would make Location B active.

---

### Question 2 (Domain 2: Design for New Solutions)
A SaaS vendor operates an application on Amazon EKS with 200 worker nodes. Workloads consist of unpredictable, spiky batch analytics jobs mixed with baseline stateless APIs. During batch job bursts, the default Kubernetes Cluster Autoscaler takes over 10 minutes to provision new EC2 nodes due to Auto Scaling Group launch template constraints and instance type unavailability in specific Availability Zones.

**Which node provisioning solution reduces scale-up latency to under 60 seconds while optimizing compute costs?**
- A. Increase the maximum size of the existing Auto Scaling Groups and deploy Warm Pools of stopped EC2 instances.
- B. Deploy **Karpenter** on the Amazon EKS cluster, configuring a `NodePool` that allows flexible instance types, architectures (`amd64` and `arm64`), and capacity types (Spot and On-Demand) with consolidation enabled.
- C. Switch from Amazon EKS to Amazon ECS with EC2 launch types.
- D. Replace the EKS cluster with AWS Lambda functions running Docker containers.

**Answer:** B  
**Explanation:** **Karpenter** is an open-source, flexible, high-velocity Kubernetes node autoscaler built specifically for cloud environments. Unlike Cluster Autoscaler, Karpenter does not manage Auto Scaling Groups; it calls EC2 fleet APIs directly to evaluate unschedulable pod resource requirements and launches precisely right-sized compute nodes within 30–60 seconds. Karpenter natively optimizes costs through automated Spot instance selection and consolidation.  
**Distractor Analysis:**
- *A is incorrect:* EC2 Warm Pools do not support Spot instances and incur costs for stopped instance storage.
- *C is incorrect:* Migrating from EKS to ECS requires complete replatforming of manifests and tools without addressing underlying node provisioning speed.
- *D is incorrect:* Long-running, high-memory batch analytics workloads frequently exceed AWS Lambda 15-minute execution and 10 GB memory limits.

---

### Question 3 (Domain 3: Continuous Improvement)
A company runs a high-performance machine learning (ML) training model across 32 EC2 GPU instances. The training process requires frequent, high-bandwidth all-reduce collective communications between nodes. The engineering team observes that network throughput is bottlenecked at 25 Gbps with significant inter-node packet latency variations, causing GPU utilization to drop to 35%.

**Which combination of actions will maximize inter-node network throughput and eliminate latency variance?**
- A. Launch instances in multiple Availability Zones and connect them via Transit Gateway peering.
- B. Launch supported EC2 instances (e.g., `p4d.24xlarge`) with **Elastic Fabric Adapter (EFA)** enabled inside a single **Cluster Placement Group**, and configure the ML framework to utilize NVIDIA Collective Communications Library (NCCL) with EFA OS-bypass.
- C. Place instances behind an Application Load Balancer with cross-zone load balancing enabled.
- D. Attach multiple Elastic Network Interfaces (ENIs) to each instance and configure round-robin DNS.

**Answer:** B  
**Explanation:** An **Elastic Fabric Adapter (EFA)** is a custom network interface that provides Operating System (OS) bypass capabilities, allowing HPC and ML applications to communicate directly with the network interface hardware via the Scalable Reliable Datagram (SRD) protocol. Deploying instances within a single **Cluster Placement Group** ensures the physical nodes are located in close physical proximity on low-latency, non-blocking 100/400 Gbps network hardware fabrics.  
**Distractor Analysis:**
- *A is incorrect:* Spreading nodes across multiple AZs and traversing Transit Gateways significantly increases network latency and packet jitter.
- *C is incorrect:* An ALB is a Layer 7 HTTP reverse proxy, completely incapable of handling high-speed node-to-node GPU communication.
- *D is incorrect:* Multiple standard ENIs do not bypass the OS kernel network stack and cannot deliver microsecond inter-node MPI/NCCL performance.

---

### Question 4 (Domain 1: Organizational Complexity)
An enterprise with 50 AWS accounts requires centralized management of all database and application software licenses (e.g., Microsoft SQL Server, Oracle, Windows Server). The company needs to track license usage across physical vCPUs and cores, prevent non-compliant EC2 launches that exceed total purchased license counts, and manage this centrally without deploying agent software on every server.

**Which AWS service directly provides automated license tracking and hard enforcement?**
- A. AWS Systems Manager Inventory
- B. AWS License Manager integrated with AWS Organizations
- C. AWS Config with custom rules
- D. AWS Cost Explorer with tag allocation

**Answer:** B  
**Explanation:** **AWS License Manager** allows enterprises to centrally define and enforce software licensing rules based on vCPUs, physical cores, physical sockets, or number of instances across AWS accounts linked through AWS Organizations. When integrated with EC2 launch templates, License Manager can execute hard stop limits that automatically block non-compliant instance launches when license caps are reached.  
**Distractor Analysis:**
- *A is incorrect:* Systems Manager Inventory records software installed on managed instances, but does not enforce hard limits or manage complex core-based licensing rules.
- *C is incorrect:* AWS Config is detective and reactive; it does not natively prevent an EC2 instance from launching when a license limit is reached.
- *D is incorrect:* Cost Explorer is a financial reporting tool, not an operational license compliance enforcer.

---

### Question 5 (Domain 2: Design for New Solutions)
An enterprise operates a global banking portal with an Amazon CloudFront distribution in front of an Application Load Balancer. The security architecture mandates that URL redirects, HTTP security headers (`Strict-Transport-Security`, `X-Frame-Options`, `Content-Security-Policy`), and user geolocation-based language rewrites must execute at the edge with sub-millisecond execution times and minimal cost. No access to request bodies or external network calls is required.

**Which solution satisfies these constraints with the LOWEST execution latency and cost?**
- A. AWS Lambda@Edge running Node.js at `viewer-request` and `viewer-response` triggers.
- B. CloudFront Functions running lightweight JavaScript at `viewer-request` and `viewer-response` triggers.
- C. Custom Lua scripts running on NGINX EC2 proxy instances behind the ALB.
- D. AWS WAF custom response rules on the ALB.

**Answer:** B  
**Explanation:** **CloudFront Functions** is a native lightweight compute feature of CloudFront built specifically for high-volume, latency-sensitive operations (URL rewrites, header manipulation, cache key normalization). CloudFront Functions run in 600+ edge locations (CloudFront PoPs), execute in sub-milliseconds ($<1$ ms), and cost approximately one-sixth the price of Lambda@Edge.  
**Distractor Analysis:**
- *A is incorrect:* Lambda@Edge executes in regional edge caches (not all PoPs), has higher cold starts, longer execution durations, and costs significantly more than CloudFront Functions.
- *C is incorrect:* EC2 NGINX proxies require infrastructure management and run at origin, not at the global edge.
- *D is incorrect:* AWS WAF is designed for security inspection and blocking, not dynamic header injection or URL manipulation.

---

### Question 6 (Domain 4: Migration & Modernization)
A manufacturing company plans to migrate an on-premises VMware vSphere cluster containing 400 virtual machines to AWS. The company requires a zero-downtime, live migration (vMotion) capability to migrate virtual machines without changing their on-premises IP addresses or converting VM disk formats.

**Which solution fulfills these strict migration requirements?**
- A. AWS Application Migration Service (AWS MGN)
- B. VMware Cloud on AWS with VMware Hybrid Cloud Extension (HCX) and Layer 2 network extension
- C. AWS Server Migration Service (SMS)
- D. AWS Backup with cross-account restore

**Answer:** B  
**Explanation:** **VMware Cloud on AWS** provides a native VMware SDDC (Software-Defined Data Center) running on bare-metal AWS Nitro infrastructure. Utilizing VMware Hybrid Cloud Extension (HCX) with Layer 2 network extension allows live vMotion migrations where virtual machines can be moved while running directly into the AWS cloud without changing IP addresses or converting VMDK disks.  
**Distractor Analysis:**
- *A is incorrect:* AWS MGN converts virtual disks into EBS volumes and installs AWS drivers; it requires a reboot and does not support live vMotion.
- *C is incorrect:* AWS SMS is deprecated in favor of MGN and does not provide live vMotion.
- *D is incorrect:* AWS Backup is a backup and disaster recovery tool, not a live VMware migration platform.

---

### Question 7 (Domain 3: Continuous Improvement)
An analytics application processes clickstream logs using an Amazon S3 bucket. Files are ingested continually throughout the day. Queries are executed primarily during the first 30 days after data generation. After 90 days, queries occur rarely, but data must remain immediately retrievable within milliseconds if requested by data auditors. After 365 days, data is archived for 7 years and can tolerate retrieval times of 5–12 hours.

**Which Amazon S3 Lifecycle configuration meets these requirements at the LOWEST storage cost?**
- A. Transition objects to S3 Standard-IA after 30 days, to S3 Glacier Instant Retrieval after 90 days, and to S3 Glacier Deep Archive after 365 days.
- B. Transition objects to S3 One Zone-IA after 30 days, and delete them after 365 days.
- C. Transition objects to S3 Intelligent-Tiering immediately upon ingestion, and apply an archive policy to transition to Deep Archive after 365 days.
- D. Keep objects in S3 Standard for 365 days, then transition directly to S3 Glacier Flexible Retrieval.

**Answer:** A  
**Explanation:**
- 0–30 days: S3 Standard (frequent queries).
- 30–90 days: S3 Standard-IA (cheaper storage, immediate retrieval).
- 90–365 days: **S3 Glacier Instant Retrieval** provides the lowest storage cost for rarely accessed data while maintaining sub-second (millisecond) retrieval latency.
- 365+ days: **S3 Glacier Deep Archive** provides the absolute lowest cost storage class in the cloud (\$0.00099/GB/month) with standard retrieval times of 12 hours.  
**Distractor Analysis:**
- *B is incorrect:* S3 One Zone-IA lacks multi-AZ resilience and deleting objects after 365 days violates the 7-year audit retention mandate.
- *C is incorrect:* While Intelligent-Tiering is automated, explicit lifecycle transitions to Standard-IA and Glacier Instant Retrieval are more cost-effective for deterministic, predictable access patterns because they avoid Intelligent-Tiering monthly object monitoring fees.
- *D is incorrect:* Keeping data in S3 Standard for 365 days incurs substantial unnecessary storage costs.

---

### Question 8 (Domain 1: Organizational Complexity)
A financial enterprise with strict compliance requirements must prevent any AWS member account within its AWS Organization from creating non-encrypted Amazon EBS volumes, Amazon RDS database instances, or Amazon S3 buckets. The solution must enforce this preventatively across all developer and administrator roles.

**How should the architect implement this preventative guardrail?**
- A. Deploy an AWS CloudFormation StackSet with an AWS Lambda function that checks resources every 10 minutes and deletes unencrypted resources.
- B. Attach a Service Control Policy (SCP) to the Organization Root or Organizational Units (OUs) that denies `ec2:CreateVolume` and `ec2:RunInstances` when the condition `ec2:Encrypted` is false, denies `rds:CreateDBInstance` when `rds:StorageEncrypted` is false, and denies `s3:PutObject` without server-side encryption headers.
- C. Enable AWS Security Hub and configure automated notifications to Slack.
- D. Create IAM policies and manually attach them to every user and group in every member account.

**Answer:** B  
**Explanation:** Service Control Policies (SCPs) act as preventative guardrails that set the maximum permissions across accounts. An SCP containing explicit `Deny` statements with condition keys (e.g., `ec2:Encrypted: "false"`) stops the API call before the unencrypted resource is created. SCPs cannot be bypassed by any user or administrator in a member account.  
**Distractor Analysis:**
- *A is incorrect:* Reactive Lambda deletion allows unencrypted data to exist for up to 10 minutes, violating strict compliance.
- *C is incorrect:* Security Hub is a detective posture tool, not a preventative guardrail.
- *D is incorrect:* Manual IAM policy assignment is unmaintainable and does not restrict the member account root user.

---

### Question 9 (Domain 2: Design for New Solutions)
An application requires relational database storage for a multi-tenant SaaS CRM system. The application experiences unpredictable traffic surges where 80% of tenants are inactive for days, while sudden marketing campaigns generate thousands of queries per minute for individual tenants. The database must automatically scale compute capacity in fine-grained increments without connection drops or downtime.

**Which database solution is BEST suited for this workload?**
- A. Amazon RDS MySQL with Multi-AZ deployment and Storage Auto Scaling.
- B. Amazon Aurora Serverless v2 with Aurora MySQL.
- C. Amazon DynamoDB with On-Demand capacity mode.
- D. Self-managed MySQL on an EC2 Auto Scaling Group.

**Answer:** B  
**Explanation:** **Amazon Aurora Serverless v2** scales compute capacity instantaneously in fine-grained increments (measured in Aurora Capacity Units - ACUs) from 0.5 up to 128 ACUs within milliseconds. It scales dynamically without connection drops, handles transactional spikes seamlessly, and scales down during idle periods to reduce costs.  
**Distractor Analysis:**
- *A is incorrect:* Standard RDS Multi-AZ storage scales automatically, but compute scaling (instance type change) requires manual modification and brief failover downtime.
- *C is incorrect:* DynamoDB is a NoSQL database; the CRM requires a relational database model.
- *D is incorrect:* Self-managed MySQL on EC2 does not support native multi-tenant relational compute auto-scaling.

---

### Question 10 (Domain 4: Migration & Modernization)
A healthcare system is migrating a monolithic legacy application to AWS. Analysis reveals that the application consists of an Apache HTTP web server frontend, a Java application tier, and an IBM DB2 relational database. The company wants to containerize the web and application tiers without rewriting code, deploy them to Amazon Elastic Container Service (Amazon ECS), and generate automated CI/CD deployment pipelines.

**Which AWS tool automates this containerization and pipeline creation process?**
- A. AWS App2Container (A2C)
- B. AWS Systems Manager Distro
- C. AWS Schema Conversion Tool (SCT)
- D. AWS Serverless Application Model (SAM)

**Answer:** A  
**Explanation:** **AWS App2Container (A2C)** is a command-line tool that analyzes Java and .NET applications running on virtual machines, extracts runtime dependencies, containerizes the workloads into standard Docker images, registers them in Amazon ECR, and automatically generates ECS task definitions and CloudFormation/CI/CD deployment pipelines.  
**Distractor Analysis:**
- *B is incorrect:* Systems Manager Distro is not an application containerization tool.
- *C is incorrect:* SCT converts database schemas, not Java application runtimes.
- *D is incorrect:* AWS SAM is designed for serverless Lambda applications.

---

### Question 11 (Domain 1: Organizational Complexity)
An enterprise with 50 AWS accounts uses AWS IAM Identity Center (AWS SSO) integrated with an external Identity Provider (Okta). The security team needs to grant temporary administrative access to production accounts during emergency production incidents (break-glass access). The access must require multi-person approval, automatically expire after 4 hours, and record every console and CLI action in AWS CloudTrail.

**Which architecture meets these requirements with the LEAST custom development?**
- A. Create hardcoded IAM users in production accounts with MFA and issue passwords via encrypted email.
- B. Deploy an open-source or native Temporary Elevated Access Management (TEAM) solution integrated with AWS IAM Identity Center to manage just-in-time permission set assignment with multi-party approval and automated time-bound revocation.
- C. Store emergency IAM access keys in an on-premises safe and rotate them manually once a year.
- D. Write an AWS Lambda function that triggers every 5 minutes to delete all IAM administrators in the production accounts.

**Answer:** B  
**Explanation:** A Temporary Elevated Access Management (TEAM) integration with AWS IAM Identity Center enables just-in-time (JIT) access requests, multi-party approval workflows, and automated time-bounded expiration. Access is granted via temporary STS credentials that tie back directly to the federated corporate identity and are fully audited in AWS CloudTrail.  
**Distractor Analysis:**
- *A & C are incorrect:* Hardcoded credentials and shared passwords violate basic security compliance standards.
- *D is incorrect:* Reactive deletion script is unstable, error-prone, and does not provide structured approval workflows.

---

### Question 12 (Domain 2: Design for New Solutions)
An enterprise operates a global payments platform on AWS. The platform must comply with PCI-DSS Level 1. Credit card numbers must be tokenized immediately upon ingress, and primary account numbers (PAN) must never be stored in plaintext. The database tier must support global multi-master writes, transactional atomicity, and continuous multi-region replication.

**Which database service meets these multi-region multi-master transactional requirements natively?**
- A. Amazon Aurora PostgreSQL with Read Replicas
- B. Amazon DynamoDB Global Tables with ACID transactions enabled
- C. Amazon RDS for MySQL Multi-AZ
- D. Amazon Redshift Multi-Region Cluster

**Answer:** B  
**Explanation:** **Amazon DynamoDB Global Tables** provides a fully managed, active-active multi-region database solution. DynamoDB natively supports multi-item, multi-table ACID transactions (`TransactWriteItems`), automated multi-master replication across AWS regions with sub-second latency, and is fully certified for PCI-DSS Level 1 compliance.  
**Distractor Analysis:**
- *A is incorrect:* Aurora Global Database has a single primary writer cluster; it is not multi-master for write operations across all regions.
- *C is incorrect:* RDS Multi-AZ operates within a single AWS region; it is not multi-region active-active.
- *D is incorrect:* Redshift is an OLAP data warehousing service, not a transactional OLTP database.

---

### Question 13 (Domain 3: Continuous Improvement)
An enterprise runs 400 EC2 instances across 5 AWS accounts. The operations team struggles with disparate SSH keys, bastions, and open port 22 security risks. The security team mandates that all SSH keys and bastions must be decommissioned immediately, all shell sessions must be logged to a central immutable S3 bucket in the Security account, and access must be governed by corporate IAM credentials.

**Which solution fulfills this mandate with the LEAST operational overhead?**
- A. Install OpenSSH on all instances and manage authorized keys via an LDAP server running on EC2.
- B. Enforce the use of **AWS Systems Manager Session Manager** across all instances, configure session logging to a centralized, KMS-encrypted Amazon S3 bucket with Object Lock in the Security account, and restrict IAM policies to allow `ssm:StartSession`.
- C. Deploy Guacamole jump hosts in every VPC behind Application Load Balancers.
- D. Create an Amazon Route 53 private hosted zone and configure SSH port forwarding over IPsec VPN.

**Answer:** B  
**Explanation:** AWS Systems Manager Session Manager provides secure, one-click shell and port-forwarding access to EC2 instances without requiring public IP addresses, inbound open firewall ports (such as port 22), or SSH key pairs. Session Manager can stream complete keystroke audit logs directly to an encrypted S3 bucket or CloudWatch Logs group in a dedicated Security account.  
**Distractor Analysis:**
- *A is incorrect:* Managing LDAP and SSH keys across hundreds of instances requires high maintenance and leaves inbound ports open.
- *C is incorrect:* Self-managed jump hosts introduce operational burden, patching requirements, and attack surfaces.
- *D is incorrect:* Port forwarding over VPN does not eliminate SSH key management or provide centralized session command logging.

---

### Question 14 (Domain 2: Design for New Solutions)
A company needs to ingest, transform, and store clickstream analytics from mobile apps. The incoming volume is 50 MB/s continuously, with bursts up to 250 MB/s. The solution must support:
1. Buffering data for up to 7 days in case downstream processors fail.
2. Multiple consumer applications reading the same stream independently with dedicated throughput.
3. Automated delivery to Amazon S3 in Apache Parquet format.

**Which combination of services should be used?**
- A. Amazon SQS Standard Queue with SQS Dead Letter Queue and AWS Lambda.
- B. Amazon Kinesis Data Streams with retention set to 7 days, Enhanced Fan-Out (EFO) for real-time consumers, and Amazon Data Firehose for S3 Parquet delivery.
- C. Amazon SNS topic with email subscriptions and DynamoDB.
- D. Amazon Managed Streaming for Apache Kafka (Amazon MSK) with single broker.

**Answer:** B  
**Explanation:** Amazon Kinesis Data Streams supports data retention from 24 hours up to 365 days (meeting the 7-day requirement). Enhanced Fan-Out (EFO) provides each consumer application with dedicated 2 MB/s read throughput per shard over HTTP/2, completely eliminating consumer competition. Amazon Data Firehose connects directly to Kinesis Data Streams to convert JSON to Apache Parquet format and deliver directly to S3.  
**Distractor Analysis:**
- *A is incorrect:* SQS standard queues cannot be read by multiple independent consumer applications without messages being consumed and removed.
- *C is incorrect:* SNS does not buffer data for 7 days.
- *D is incorrect:* A single broker MSK cluster has a single point of failure and lacks native Parquet S3 conversion.

---

### Question 15 (Domain 1: Organizational Complexity)
An enterprise architecture team is designing an IP address allocation scheme for 150 upcoming VPCs across 10 business units. The company has a limited public RFC 1918 corporate address space (`10.0.0.0/8`). Spoke VPCs will host microservices that need outbound access to the Internet and need to access centralized APIs in a Shared Services VPC, but Spoke VPCs will rarely need to communicate directly with each other.

**Which IP design provides the HIGHEST scalability while avoiding IP exhaustion?**
- A. Allocate a unique `/16` CIDR block from `10.0.0.0/8` to every single Spoke VPC.
- B. Use the IPv4 Private Space (`100.64.0.0/10` Carrier-Grade NAT space) or repeat identical private RFC 1918 CIDRs (e.g. `10.100.0.0/16`) in Spoke VPCs, and connect Spoke VPCs to shared services using **AWS PrivateLink** or **AWS VPC Lattice**.
- C. Assign public IPv4 addresses directly to all EC2 instances and disable private networking.
- D. Request a `/8` public IPv4 allocation from ARIN and assign public IPs to all VPCs.

**Answer:** B  
**Explanation:** Using identical or non-routable CIDRs (such as CGNAT `100.64.0.0/10` or repeating `10.100.0.0/16`) prevents RFC 1918 exhaustion. By using AWS PrivateLink or AWS VPC Lattice to access shared services, services are published via Layer 7 proxies and unidirectional endpoints, completely eliminating the need for unique Layer 3 IP routing between Spoke VPCs.  
**Distractor Analysis:**
- *A is incorrect:* A `/8` network only contains 256 `/16` subnets; assigning a `/16` to every VPC will rapidly exhaust the corporate IP space.
- *C is incorrect:* Assigning public IPv4 addresses exposes workloads directly to the Internet and incurs immense IPv4 address costs.
- *D is incorrect:* Acquiring a public `/8` IPv4 block costs millions of dollars and is completely unnecessary.

---

### Question 16 (Domain 4: Migration & Modernization)
A bank is migrating an Oracle RAC database to Amazon Aurora PostgreSQL. During discovery, the database team identifies 300 stored procedures containing proprietary Oracle PL/SQL features, hierarchical queries (`CONNECT BY`), and packages that do not exist in PostgreSQL.

**Which tool should the solutions architect use to analyze the code compatibility and generate automated conversions and actionable migration recommendations?**
- A. AWS Application Migration Service (AWS MGN)
- B. AWS Schema Conversion Tool (AWS SCT)
- C. AWS Database Migration Service (AWS DMS)
- D. AWS DataSync

**Answer:** B  
**Explanation:** **AWS Schema Conversion Tool (AWS SCT)** is purpose-built to convert commercial database schemas, views, stored procedures, triggers, and functions to open-source or AWS-native database engines (such as Oracle to PostgreSQL). SCT generates an assessment report identifying the percentage of code automatically converted and provides prescriptive action items for code that must be manually refactored.  
**Distractor Analysis:**
- *A is incorrect:* AWS MGN performs block-level server lift-and-shift.
- *C is incorrect:* AWS DMS migrates live data; it does not analyze or refactor complex PL/SQL stored procedures or database schemas.
- *D is incorrect:* DataSync transfers filesystem files and objects.

---

### Question 17 (Domain 3: Continuous Improvement)
An application runs on an Amazon Aurora MySQL database cluster with 1 primary writer and 3 read replicas. During end-of-month reporting, large batch analytics read queries overwhelm the read replicas, driving replica lag to over 45 seconds. This causes normal web application read queries hitting the reader endpoint to serve stale data.

**How can the solutions architect isolate the batch reporting queries from normal web application reads?**
- A. Add 5 more read replicas to the existing cluster.
- B. Create an **Amazon Aurora Custom Endpoint** that includes a dedicated read replica instance for batch reporting, and configure the web application to use the default cluster reader endpoint.
- C. Convert the cluster to Amazon RDS MySQL Multi-AZ.
- D. Instruct the reporting tool to query the primary writer instance directly.

**Answer:** B  
**Explanation:** **Aurora Custom Endpoints** allow architects to create logical groups of DB instances within an Aurora cluster. By creating a custom endpoint that routes traffic exclusively to a dedicated, high-memory read replica provisioned for heavy reporting queries, reporting jobs are completely isolated from the standard reader endpoint utilized by the production web application.  
**Distractor Analysis:**
- *A is incorrect:* Adding replicas to the generic reader endpoint still leaves normal web traffic vulnerable to hitting overwhelmed replicas.
- *C is incorrect:* Moving to standard RDS MySQL loses Aurora's specialized storage architecture and custom endpoint capabilities.
- *D is incorrect:* Sending heavy analytics queries to the primary writer instance will degrade transactional write throughput for the entire application.

---

### Question 18 (Domain 2: Design for New Solutions)
A company is developing an image recognition SaaS application. Users upload high-resolution images that must be resized into 5 different thumbnail sizes, tagged using Amazon Rekognition, and cataloged in an Amazon Aurora database. Image processing takes approximately 10 seconds per image. The upload volume peaks at 1,000 images per minute.

**Which architecture provides the MOST scalable, decoupled processing?**
- A. Users upload images directly to the web servers; web servers process the images synchronously before returning an HTTP 200 response.
- B. Users upload images directly to an Amazon S3 bucket using pre-signed S3 URLs. S3 emits an `ObjectCreated` event to Amazon EventBridge, which triggers an AWS Step Functions workflow that coordinates AWS Lambda functions for resizing and Rekognition analysis in parallel.
- C. Web servers write images to an Amazon EFS mount, and a cron job runs a bash script every minute to process images.
- D. Users upload images via Amazon API Gateway directly to an Amazon DynamoDB table.

**Answer:** B  
**Explanation:** S3 pre-signed URLs offload upload traffic from web servers completely. S3 Event Notifications integrated with Amazon EventBridge and AWS Step Functions provide a fully decoupled, event-driven serverless architecture that scales automatically with incoming volume. Step Functions parallel states coordinate thumbnail generation and Rekognition tagging concurrently.  
**Distractor Analysis:**
- *A is incorrect:* Synchronous processing blocks web servers, consumes memory, and leads to HTTP request timeouts during peak upload bursts.
- *C is incorrect:* EFS cron jobs are batch-oriented, unscalable, and introduce a single point of failure.
- *D is incorrect:* DynamoDB item size limit is 400 KB; it cannot store high-resolution images directly.

---

### Question 19 (Domain 1: Organizational Complexity)
An enterprise requires all internet-bound outbound traffic from 100 spoke accounts to be inspected and logged for regulatory compliance. The architecture utilizes an AWS Transit Gateway connected to an Inspection VPC with AWS Network Firewall. The security team mandates that outbound web traffic must be filtered using domain name lists (e.g. allow `*.github.com`, block all other domains).

**Which component of AWS Network Firewall satisfies this requirement?**
- A. Stateless rule group inspecting Layer 4 IP headers.
- B. Stateful rule group with a **Domain list** rule group using TLS Server Name Indication (SNI) and HTTP Host header matching.
- C. Amazon GuardDuty DNS security findings.
- D. Route 53 Public Hosted Zone forwarding rules.

**Answer:** B  
**Explanation:** AWS Network Firewall stateful engine supports **Domain list** rule groups (powered by Suricata). For HTTPS traffic, the firewall inspects the TLS Server Name Indication (SNI) extension in the client TLS ClientHello packet. For HTTP traffic, it inspects the Host header. This allows enforcing strict domain-level whitelisting/blacklisting for all outbound internet traffic.  
**Distractor Analysis:**
- *A is incorrect:* Stateless rule groups evaluate Layer 3 and Layer 4 attributes (IP addresses, ports, protocols) without inspecting domain names or TLS SNI headers.
- *C is incorrect:* GuardDuty is a detective threat detection service, not an inline traffic filtering firewall.
- *D is incorrect:* Route 53 public zones do not inspect or filter inline egress network packets.

---

### Question 20 (Domain 4: Migration & Modernization)
A logistics company is migrating a legacy Java web application to Amazon ECS on AWS Fargate. The application currently writes user session state and temporary transaction logs to local disk storage (`/tmp/sessions`). If a container restarts, active user sessions must not be lost. The application cannot be immediately refactored to use Redis for session management due to code freeze deadlines.

**Which storage configuration provides persistent, shared file storage for the ECS Fargate tasks?**
- A. Amazon EBS General Purpose SSD (`gp3`) volume attached via ECS task definition.
- B. Amazon EFS file system mounted to the ECS Fargate task definition using EFS access points and IAM authorization.
- C. Amazon S3 bucket mounted via S3FS FUSE driver inside the container.
- D. Instance store volumes mapped in the Dockerfile.

**Answer:** B  
**Explanation:** AWS Fargate natively supports mounting **Amazon EFS** file systems directly inside task definitions. EFS provides shared, persistent POSIX-compliant network storage across multiple AZs. If a task terminates or restarts, the replacement task mounts the identical EFS access point, preserving existing user session files without requiring code changes.  
**Distractor Analysis:**
- *A is incorrect:* AWS Fargate does NOT support attaching Amazon EBS volumes directly as shared persistent storage across task restarts (EBS task attachments are ephemeral or single-AZ bounded).
- *C is incorrect:* S3FS is not POSIX-compliant, introduces severe file locking issues, and is not a native or supported Fargate feature.
- *D is incorrect:* AWS Fargate does not provide access to underlying EC2 instance store hardware.

---

### Question 21 (Domain 3: Continuous Improvement)
A company runs a mission-critical web application on Amazon EC2 instances behind an Application Load Balancer. The database is hosted on Amazon RDS MySQL. During a recent maintenance event, the database failed over to its standby replica in another AZ. Although the database completed failover within 40 seconds, the web application experienced connection errors and elevated error rates for over 10 minutes because application connection pools retained stale DNS records pointing to the old primary IP address.

**How can the solutions architect eliminate this connection lag during database failover?**
- A. Deploy an **Amazon RDS Proxy** between the application instances and the RDS MySQL cluster.
- B. Reduce the JVM DNS caching TTL to 0 seconds on the EC2 instances.
- C. Convert the database to self-managed MySQL on EC2 with Elastic IP reassignment scripts.
- D. Deploy an Application Load Balancer in front of the RDS database.

**Answer:** A  
**Explanation:** **Amazon RDS Proxy** maintains a pool of established connections to database instances. When an RDS Multi-AZ failover occurs, RDS Proxy automatically and transparently reroutes traffic to the newly promoted standby primary database instance without requiring the application tier to re-resolve DNS or drop its connection pool. This reduces failover connection recovery times by up to 66%.  
**Distractor Analysis:**
- *B is incorrect:* While tuning JVM DNS TTL helps slightly, application connection pools still hold broken TCP sockets that must time out and reconnect, causing errors.
- *C is incorrect:* Self-managed MySQL on EC2 increases operational overhead and introduces manual failover failure modes.
- *D is incorrect:* An Application Load Balancer (ALB) only routes HTTP/HTTPS traffic; it cannot load-balance MySQL database wire protocol (port 3306).

---

### Question 22 (Domain 2: Design for New Solutions)
An enterprise needs to establish an automated archival system for corporate communications. The system must ingest 5 million emails daily, scan them for sensitive keywords, store the raw emails in Amazon S3, and notify compliance officers if a high-risk violation is detected. The solution must scale automatically and require minimal ongoing maintenance.

**Which architecture satisfies these requirements with the HIGHEST scalability?**
- A. Ingest emails via Amazon Simple Email Service (SES). Configure an SES Receipt Rule to store emails in Amazon S3 and publish a notification to an Amazon SNS topic. The SNS topic triggers an AWS Lambda function that scans the message text using Amazon Comprehend for sentiment and PII. If a violation is found, Lambda sends an alert to a compliance SNS topic.
- B. Run an EC2 Postfix email server that writes files to local EBS storage and runs a cron job scanning files with grep.
- C. Ingest emails into Amazon Kinesis Video Streams.
- D. Ingest emails via an Application Load Balancer directly into Amazon RDS.

**Answer:** A  
**Explanation:** Amazon SES receipt rules can natively deliver incoming emails directly into Amazon S3 buckets while publishing event notifications to Amazon SNS. Integrating SNS with AWS Lambda and Amazon Comprehend provides a completely serverless, scalable, and automated content analysis pipeline requiring zero server maintenance.  
**Distractor Analysis:**
- *B is incorrect:* Managing Postfix servers on EC2 requires high operational overhead, scaling management, and patching.
- *C is incorrect:* Kinesis Video Streams is for video/audio streaming, not text-based email processing.
- *D is incorrect:* ALBs cannot receive SMTP email protocols.

---

### Question 23 (Domain 1: Organizational Complexity)
An enterprise with an active AWS Organization wants to enforce tag governance across all member accounts. The governance policy mandates that all Amazon EC2 instances, EBS volumes, and S3 buckets must have a `CostCenter` tag with an authorized 4-digit code (e.g. `1001`, `1002`, `1003`).

**Which AWS Organizations feature should be used to enforce tag consistency and prevent non-compliant tags?**
- A. Backup Policies
- B. Tag Policies
- C. Service Control Policies (SCPs) with wildcard string matching only
- D. CloudWatch Alarms

**Answer:** B  
**Explanation:** **Tag Policies** in AWS Organizations allow administrators to define tag keys and allowed tag values (such as approved 4-digit CostCenter codes) across the entire organization or specific OUs. Tag Policies can also be configured to enforce non-compliant resource creation prevention, ensuring that any API request attempting to create resources with invalid or missing tag values is rejected.  
**Distractor Analysis:**
- *A is incorrect:* Backup Policies define centralized backup schedules, not resource tagging rules.
- *C is incorrect:* While SCPs can check for the presence of tags, Tag Policies are specifically designed to validate tag capitalization, allowed value enums, and compliance reporting.
- *D is incorrect:* CloudWatch Alarms monitor metrics; they cannot validate or enforce resource tags.

---

### Question 24 (Domain 4: Migration & Modernization)
A bank is migrating 100 TB of historical loan documents stored on an on-premises Windows File Server (SMB shares) to Amazon FSx for Windows File Server. The solution must preserve all Windows Access Control Lists (ACLs), ownership metadata, and file attributes. The network link is a 500 Mbps Site-to-Site VPN with 70% daytime utilization.

**Which migration method guarantees metadata preservation and maximizes transfer speed?**
- A. Copy the files to an external USB drive and ship it to an AWS data center.
- B. Deploy an AWS DataSync agent on-premises. Configure a DataSync task with the SMB share as source and the Amazon FSx for Windows File Server file system as destination. Configure bandwidth throttling during business hours and enable verification.
- C. Use `xcopy` over the VPN link from an EC2 instance.
- D. Deploy an AWS Storage Gateway Tape Gateway.

**Answer:** B  
**Explanation:** AWS DataSync natively integrates with SMB shares and Amazon FSx for Windows File Server. DataSync automatically preserves Windows security descriptors, NTFS Access Control Lists (ACLs), file attributes, and timestamps. It includes built-in bandwidth throttling to protect the VPN link during business hours and performs automated data verification.  
**Distractor Analysis:**
- *A is incorrect:* AWS does not accept arbitrary consumer USB hard drives shipped to data centers.
- *C is incorrect:* `xcopy` over a high-latency WAN connection is extremely slow, lacks automated parallelization, and has no automated bandwidth throttling or integrity verification.
- *D is incorrect:* Tape Gateway is for virtual tape backup software, not for active Windows SMB file system migration.

---

### Question 25 (Domain 2: Design for New Solutions)
A company is designing a cross-region disaster recovery solution for an application running on Amazon EC2 and Amazon Aurora PostgreSQL in `us-east-1`. The business requires an RPO $< 1$ minute and an RTO $< 15$ minutes in the secondary region (`us-west-2`). Compute resources in the secondary region should only be provisioned and scaled up when an actual disaster occurs to minimize ongoing costs.

**Which disaster recovery strategy meets these requirements?**
- A. Multi-Region Active-Active
- B. Pilot Light using Amazon Aurora Global Database with dormant compute, where secondary region EC2 Auto Scaling Groups are maintained at desired capacity 0 and scaled up via automated scripts upon failover.
- C. Backup and Restore using daily AWS Backup snapshots.
- D. Warm Standby with identical full-capacity EC2 fleets running 24/7 in both regions.

**Answer:** B  
**Explanation:** The **Pilot Light** strategy maintains the core data tier continuously synchronized in the secondary region (achieving RPO $< 1$ second via Aurora Global Database physical replication) while keeping the compute layer dormant (ASG desired capacity = 0). Upon disaster declaration, automated orchestration (via CloudFormation, Terraform, or Route 53 ARC) scales up the EC2 Auto Scaling fleet in `us-west-2` to restore full capacity well within the 15-minute RTO, while saving significant compute costs during normal operations.  
**Distractor Analysis:**
- *A is incorrect:* Multi-Region Active-Active runs full compute fleets in both regions 24/7, incurring high unnecessary compute costs.
- *C is incorrect:* Daily Backup and Restore results in an RPO of up to 24 hours and an RTO of hours to restore petabyte snapshots, failing the RPO $< 1$ minute requirement.
- *D is incorrect:* Warm Standby maintains running compute instances in the secondary region, which increases ongoing operational costs.
