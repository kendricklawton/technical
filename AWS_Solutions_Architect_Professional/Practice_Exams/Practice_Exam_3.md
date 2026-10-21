# AWS Certified Solutions Architect - Professional (SAP-C02) Practice Exam 3

**Duration:** 60 Minutes (25 Full-Length Multi-Paragraph Scenario Questions)  
**Passing Score Target:** $\ge 20 / 25$ (80%)  
**Exam Blueprint Focus:** Domain 1 (Organizational Complexity: 26%), Domain 2 (Design for New Solutions: 29%), Domain 3 (Continuous Improvement: 25%), Domain 4 (Migration & Modernization: 20%)

---

### Question 1 (Domain 1: Organizational Complexity)
An enterprise with 80 AWS accounts is designing a multi-account transit network. Spoke VPCs in different accounts need to communicate with each other through a centralized AWS Transit Gateway. The security policy mandates that:
1. VPCs belonging to the Development OU must be able to communicate with other Development VPCs and the Shared Services VPC.
2. VPCs belonging to the Production OU must be able to communicate with other Production VPCs and the Shared Services VPC.
3. Development VPCs must be strictly isolated from Production VPCs with zero network route reachability.

**How should the transit network be configured with the LEAST operational overhead?**
- A. Deploy two separate Transit Gateways: one for Development and one for Production.
- B. Deploy a single Transit Gateway with three Transit Gateway Route Tables: a Development Route Table, a Production Route Table, and a Shared Services Route Table. Associate Development VPC attachments with the Dev Route Table, Production VPC attachments with the Prod Route Table, and propagate routes accordingly to enforce network segmentation.
- C. Deploy a single Transit Gateway Route Table and write security group rules on every EC2 instance.
- D. Connect all VPCs using full-mesh VPC Peering.

**Answer:** B  
**Explanation:** AWS Transit Gateway supports multiple isolated **Transit Gateway Route Tables**. By creating separate route tables for Development and Production, network traffic can be segmented at Layer 3:
- Development VPC attachments associate with the Dev Route Table (which has routes to Dev and Shared Services, but NOT Production).
- Production VPC attachments associate with the Prod Route Table (which has routes to Prod and Shared Services, but NOT Development).
This provides cryptographic and routing isolation on a single, shared Transit Gateway without managing duplicate infrastructure.  
**Distractor Analysis:**
- *A is incorrect:* Deploying multiple Transit Gateways doubles hourly gateway fees and complicates hybrid Direct Connect connectivity.
- *C is incorrect:* Relying on individual EC2 security groups lacks central network segmentation guardrails.
- *D is incorrect:* Full mesh VPC peering between 80 accounts requires thousands of peering connections, creating an unmaintainable topology.

---

### Question 2 (Domain 2: Design for New Solutions)
An enterprise operates an internal employee portal on Amazon EC2 behind an Application Load Balancer (ALB). The company uses Microsoft Entra ID (formerly Azure AD) as its corporate SAML 2.0 Identity Provider. The CISO mandates that:
1. Employees must authenticate against Entra ID before accessing the portal.
2. The web application code running on EC2 must NOT manage user login sessions, redirect flows, or validate SAML/OIDC tokens directly.

**Which solution fulfills this mandate with the LEAST application modifications?**
- A. Install an open-source SAML SP agent on every EC2 instance.
- B. Configure the Application Load Balancer to authenticate users natively through an **Authenticate OIDC** action using Microsoft Entra ID as the OpenID Connect identity provider before forwarding requests to the target group.
- C. Deploy an Amazon API Gateway in front of the ALB with a custom Lambda authorizer.
- D. Require employees to connect via AWS Client VPN before accessing the portal.

**Answer:** B  
**Explanation:** Application Load Balancers natively support user authentication via OpenID Connect (OIDC) compliant Identity Providers (including Microsoft Entra ID, Okta, Ping, and Amazon Cognito). The ALB intercepts unauthenticated requests, manages the redirect handshake with the IdP, validates the signed JSON Web Token (JWT), and passes user claims to backend targets via HTTP headers (`x-amzn-oidc-*`), completely removing authentication complexity from application code.  
**Distractor Analysis:**
- *A is incorrect:* Managing local SAML agents on web servers requires maintenance, certificate rotation, and application changes.
- *C is incorrect:* API Gateway in front of ALB adds unnecessary cost, latency, and proxy complexity when ALB natively supports OIDC.
- *D is incorrect:* Client VPN addresses network connectivity, not application user identity and authorization.

---

### Question 3 (Domain 3: Continuous Improvement)
An enterprise operates a fleet of 500 EC2 instances across 4 AWS regions. A security review reveals that multiple instances are running outdated AMIs with unpatched zero-day vulnerabilities. The cloud operations team needs an automated, centralized pipeline to build, test, and distribute hardened "Golden AMIs" compliant with CIS Benchmarks to all 4 regions monthly.

**Which AWS service is purpose-built to automate this Golden AMI lifecycle?**
- A. AWS Systems Manager Run Command
- B. EC2 Image Builder
- C. AWS CodeBuild
- D. AWS OpsWorks

**Answer:** B  
**Explanation:** **EC2 Image Builder** is a fully managed AWS service designed specifically to automate the creation, management, testing, and deployment of customized, secure, and up-to-date server images (AMIs and container images). Image Builder allows defining image recipes (OS + CIS hardening components), running automated validation tests, and distributing the resulting encrypted AMIs across multiple AWS accounts and regions on a scheduled monthly cadence.  
**Distractor Analysis:**
- *A is incorrect:* Run Command executes ad-hoc scripts; it does not provide an end-to-end AMI testing and multi-region distribution pipeline.
- *C is incorrect:* CodeBuild is a general-purpose build service, not a specialized operating system AMI builder.
- *D is incorrect:* AWS OpsWorks is deprecated and does not specialize in multi-region AMI lifecycle distribution.

---

### Question 4 (Domain 4: Migration & Modernization)
A hospital network needs to migrate an on-premises PACS (Picture Archiving and Communication System) archive consisting of 1.2 PB of DICOM medical images to AWS. The hospital has a single 1 Gbps Internet connection that is 80% utilized during business hours. Compliance mandates that all medical images must be encrypted in transit and at rest, and the migration must complete within 60 days.

**Which migration method satisfies the time constraint and security mandates?**
- A. Copy the files over the existing 1 Gbps Internet connection using AWS DataSync with bandwidth limits.
- B. Order multiple **AWS Snowball Edge Storage Optimized** devices. Copy the DICOM images onto the devices locally with AES-256 encryption, ship the devices back to AWS for ingestion into Amazon S3, and configure S3 server-side encryption with AWS KMS (SSE-KMS).
- C. Deploy an AWS Storage Gateway Volume Gateway on-premises.
- D. Use `rsync` over a public SSH connection.

**Answer:** B  
**Explanation:** Over an existing 1 Gbps link running at 80% utilization, only 200 Mbps is available for migration. Transferring 1.2 PB over 200 Mbps would take over 550 days, violating the 60-day deadline. Ordering AWS Snowball Edge Storage Optimized appliances (each holding up to 80 TB or 210 TB) enables local high-speed physical transfers over 10G/40G local network links. Snowball devices automatically enforce AES-256 hardware encryption, and AWS imports data directly into Amazon S3 with SSE-KMS within days.  
**Distractor Analysis:**
- *A is incorrect:* Bandwidth limitations make online WAN transfer mathematically impossible within the 60-day constraint.
- *C is incorrect:* Volume Gateway still relies on WAN bandwidth for underlying data transfer to S3.
- *D is incorrect:* Unencrypted public `rsync` violates healthcare compliance (HIPAA) and suffers from severe WAN latency.

---

### Question 5 (Domain 2: Design for New Solutions)
An e-commerce retailer experiences heavy bot traffic scraping product prices and executing credential stuffing attacks against its login endpoint (`/api/v1/auth/login`). The application runs on Amazon ECS behind an Application Load Balancer.

**Which AWS solution mitigates both automated scraping and credential stuffing with the LEAST custom development?**
- A. Write a custom AWS Lambda function that parses ALB access logs in CloudWatch Logs and modifies Network ACLs.
- B. Associate an AWS WAF Web ACL with the Application Load Balancer and enable the **AWSManagedRulesBotControlRuleSet** and **AWSManagedRulesAccountTakeoverPreventionRuleSet** (ATP).
- C. Place the application behind Amazon Route 53 with latency routing.
- D. Enable AWS Shield Standard on the ALB.

**Answer:** B  
**Explanation:** **AWS WAF Bot Control** and **Account Takeover Prevention (ATP)** are specialized AWS-managed rule sets. Bot Control detects and mitigates sophisticated, evasive bots and scrapers. ATP specifically monitors login pages to block credential stuffing attacks, brute force attempts, and the use of compromised credentials in real-time without writing custom code or parsing logs reactively.  
**Distractor Analysis:**
- *A is incorrect:* Parsing logs with Lambda is reactive, slow, and Network ACLs cannot handle dynamic application-layer bot signatures.
- *C is incorrect:* DNS routing does not inspect HTTP payloads or block malicious bot signatures.
- *D is incorrect:* AWS Shield Standard provides baseline Layer 3/4 DDoS protection; it does not protect against Layer 7 bots or credential stuffing.

---

### Question 6 (Domain 1: Organizational Complexity)
An enterprise with 40 AWS accounts uses AWS Control Tower. The compliance team mandates that:
1. Every AWS account must stream all AWS CloudTrail management events and data events into a centralized Amazon S3 bucket located in the dedicated **Log Archive** account.
2. The log files in the Log Archive bucket must be protected from deletion or tampering by any account administrator or compromised root credential for 5 years.

**Which combination of actions fulfills these mandates?**
- A. Configure CloudTrail in each member account to write to a local S3 bucket, and use S3 Cross-Region Replication to the Log Archive account.
- B. Deploy an **Organization Trail** in the AWS Organizations Management account that delivers logs directly to a central S3 bucket in the Log Archive account. Configure **S3 Object Lock in Compliance Mode** with a 5-year retention period and enable S3 Log File Validation.
- C. Store logs in Amazon DynamoDB with point-in-time recovery enabled.
- D. Stream CloudTrail events to Amazon CloudWatch Logs in each member account and export to Amazon OpenSearch.

**Answer:** B  
**Explanation:** An Organization Trail created in the management account automatically records events across all current and future accounts in the AWS Organization and writes directly to a centralized bucket in the Log Archive account. Enabling **S3 Object Lock in Compliance Mode** ensures that no user—including account root users or AWS Support—can delete or overwrite log objects for the specified 5-year duration. Log File Validation provides cryptographic proof of log integrity.  
**Distractor Analysis:**
- *A is incorrect:* Local trails in member accounts can be disabled or deleted by local administrators before replication occurs.
- *C is incorrect:* DynamoDB is not the standard or cost-effective target for high-volume CloudTrail event archiving.
- *D is incorrect:* CloudWatch Logs in member accounts does not provide centralized, tamper-proof WORM compliance.

---

### Question 7 (Domain 3: Continuous Improvement)
A company runs a web application on Amazon EC2 instances behind an Application Load Balancer. The database is hosted on Amazon Aurora PostgreSQL. Monitoring alerts indicate frequent connection timeouts during peak traffic. Investigation reveals that the web tier opens 5,000 concurrent database connections, exceeding the `max_connections` limit on the database instance and causing memory exhaustion.

**Which architectural enhancement solves this connection bottleneck with the HIGHEST scalability?**
- A. Increase the `max_connections` parameter in the Aurora DB Parameter Group to 50,000.
- B. Deploy **Amazon RDS Proxy** between the web application and the Aurora PostgreSQL cluster to enable connection pooling and connection multiplexing.
- C. Vertically scale the Aurora instance to `db.r6g.16xlarge`.
- D. Migrate the database to Amazon S3 and query with Amazon Athena.

**Answer:** B  
**Explanation:** **Amazon RDS Proxy** pools and shares established database connections. Instead of each web server process opening a dedicated physical database connection, RDS Proxy multiplexes thousands of frontend application requests across a small, healthy pool of physical database connections. This shields the database from connection exhaustion, preserves database memory, and improves failover resilience.  
**Distractor Analysis:**
- *A is incorrect:* Arbitrarily raising `max_connections` leads to database OS memory starvation and severe performance degradation.
- *C is incorrect:* Vertical scaling is extremely costly and only temporarily delays connection limits without solving connection sprawl.
- *D is incorrect:* S3 and Athena are analytical tools, not transactional OLTP relational databases.

---

### Question 8 (Domain 2: Design for New Solutions)
A video streaming provider needs to deliver video-on-demand (VOD) assets stored in Amazon S3 to authenticated subscribers worldwide. The company must prevent unauthorized link sharing (hotlinking) and ensure that only subscribers who have paid an active subscription can view the media files. Content links must expire after 4 hours.

**Which solution should the architect implement?**
- A. Make the Amazon S3 bucket publicly readable and use an AWS WAF rate limit.
- B. Serve the video assets through Amazon CloudFront configured with Origin Access Control (OAC), and generate **CloudFront Signed URLs** or **Signed Cookies** using a custom application signer key.
- C. Deploy an EC2 proxy fleet that downloads files from S3 and streams them over HTTP.
- D. Require users to establish an IPsec VPN to the S3 bucket.

**Answer:** B  
**Explanation:** Amazon CloudFront Signed URLs and Signed Cookies allow providers to control access to private content. CloudFront validates the cryptographic signature and expiration timestamp before serving the content from its edge cache. Origin Access Control (OAC) ensures that direct public access to the backing S3 origin bucket is completely blocked.  
**Distractor Analysis:**
- *A is incorrect:* Making the bucket public allows anyone with the URL to download the assets without authentication.
- *C is incorrect:* Self-managed EC2 streaming proxies add massive operational overhead, bandwidth bottlenecks, and high costs compared to CloudFront edge distribution.
- *D is incorrect:* Requiring end consumers to configure IPsec VPNs for video streaming is unworkable.

---

### Question 9 (Domain 4: Migration & Modernization)
An enterprise is planning to migrate 50 web and API applications from an on-premises data center to AWS. The architecture team needs to automatically discover running servers, map network communication dependencies between applications, and generate migration wave groups based on dependencies.

**Which AWS service provides automated application dependency mapping?**
- A. AWS Application Discovery Service with the Discovery Agent
- B. AWS CloudFormation
- C. AWS Systems Manager Session Manager
- D. AWS Trusted Advisor

**Answer:** A  
**Explanation:** **AWS Application Discovery Service** deployed with the **Discovery Agent** on on-premises physical and virtual servers collects detailed system configuration, performance metrics, and network connection dependencies (inbound/outbound TCP/UDP connections). These dependencies can be viewed in AWS Migration Hub to accurately map multi-tier applications and organize them into cohesive migration waves.  
**Distractor Analysis:**
- *B is incorrect:* CloudFormation provisions AWS infrastructure; it does not discover on-premises dependencies.
- *C is incorrect:* Session Manager provides remote shell access; it does not map enterprise network communication graphs.
- *D is incorrect:* Trusted Advisor inspects existing AWS accounts for cost and security best practices.

---

### Question 10 (Domain 1: Organizational Complexity)
A multinational financial institution with an AWS Organizations deployment must ensure that data stored in Amazon S3 buckets located in the European Union (`eu-west-1` and `eu-central-1`) is NEVER replicated or transferred to AWS regions outside the EU due to GDPR sovereignty regulations.

**Which preventative control strictly guarantees compliance across all member accounts?**
- A. Write an AWS Config rule that triggers an SNS email alert if an S3 bucket has cross-region replication configured.
- B. Apply a Service Control Policy (SCP) at the Organization Root level that denies `s3:PutReplicationConfiguration` if the destination region condition `s3:ResourceAccount` or `aws:RequestedRegion` is not in an approved EU region.
- C. Delete all S3 buckets located outside the EU.
- D. Restrict IAM permissions on developer laptops.

**Answer:** B  
**Explanation:** A Service Control Policy (SCP) acts as a preventative guardrail. By explicitly denying S3 replication configuration actions when the target bucket or region is outside the approved EU boundaries, no administrator or IAM user in any member account can configure cross-region replication to unapproved regions.  
**Distractor Analysis:**
- *A is incorrect:* Config rules are detective and reactive; replication could begin before the alert is processed.
- *C is incorrect:* Deleting buckets does not prevent future replication setup.
- *D is incorrect:* Restricting developer laptops does not prevent server-side IAM roles or automation pipelines from configuring replication.

---

### Question 11 (Domain 2: Design for New Solutions)
An application requires high-performance file storage for an engineering simulation workload running on Linux EC2 instances. The storage must deliver sub-millisecond latencies, support POSIX file system semantics, and provide sustained throughput of 50 GB/s with millions of IOPS. The data set is hydrated from Amazon S3 and output results must be synchronized back to S3.

**Which storage service fulfills these requirements?**
- A. Amazon Elastic Block Store (Amazon EBS) `gp3` volumes
- B. Amazon FSx for Lustre
- C. Amazon Elastic File System (Amazon EFS)
- D. Amazon S3 Glacier Instant Retrieval

**Answer:** B  
**Explanation:** **Amazon FSx for Lustre** is a high-performance parallel file system designed specifically for compute-intensive workloads such as machine learning, HPC, and simulations. FSx for Lustre scales up to hundreds of gigabytes per second of throughput and millions of IOPS with sub-millisecond latencies. It integrates natively with Amazon S3, allowing transparent bidirectional synchronization of datasets between the file system and S3 buckets.  
**Distractor Analysis:**
- *A is incorrect:* Individual EBS volumes cannot be shared across hundreds of EC2 instances as a parallel POSIX file system delivering 50 GB/s.
- *C is incorrect:* EFS is an NFS-based file system designed for general-purpose shared storage; it cannot deliver 50 GB/s parallel throughput for HPC.
- *D is incorrect:* S3 Glacier is an archival object storage tier, not an active POSIX file system.

---

### Question 12 (Domain 3: Continuous Improvement)
An enterprise operates 200 microservices across multiple Amazon EKS clusters and AWS Lambda functions. When an end-user request fails, developers struggle to identify which microservice in the call chain produced the error or introduced latency. The engineering team mandates distributed tracing across all microservices with minimal proprietary vendor lock-in.

**Which solution should the architect implement?**
- A. Write custom log parsing scripts using CloudWatch Logs Insights.
- B. Instrument application code using the **AWS Distro for OpenTelemetry (ADOT)**, configuring the OpenTelemetry Collector to export distributed traces to **AWS X-Ray** and Amazon CloudWatch.
- C. Deploy an EC2 instance running Jaeger and write custom UDP log forwarders.
- D. Enable VPC Flow Logs with custom fields across all subnets.

**Answer:** B  
**Explanation:** **AWS Distro for OpenTelemetry (ADOT)** provides an open-source, vendor-neutral distribution of the CNCF OpenTelemetry standard. It automatically instruments applications to capture metrics and distributed traces, propagating trace context across microservices (using W3C trace context headers) and exporting telemetry seamlessly to AWS X-Ray, CloudWatch, or third-party APM tools without proprietary agent lock-in.  
**Distractor Analysis:**
- *A is incorrect:* Log parsing lacks end-to-end distributed context propagation across asynchronous service boundaries.
- *C is incorrect:* Managing self-hosted Jaeger clusters on EC2 increases operational maintenance.
- *D is incorrect:* VPC Flow Logs captures Layer 4 IP packet flows; it provides zero application-layer distributed tracing or latency breakdown.

---

### Question 13 (Domain 4: Migration & Modernization)
A manufacturing company operates an on-premises SAP ERP system backed by a 20 TB Oracle database. The business requires migrating the SAP application to AWS. The target database is SAP HANA on Amazon EC2 memory-optimized instances. Network connectivity consists of a 10 Gbps AWS Direct Connect link. The maximum allowable business cutover window is 6 hours over a weekend.

**Which migration toolset provides continuous replication to satisfy the 6-hour cutover window?**
- A. SAP Software Update Manager (SUM) Database Migration Option (DMO) with System Move using AWS Direct Connect.
- B. Export the database to CSV files and upload via the AWS Management Console.
- C. AWS Snowcone
- D. AWS Serverless Application Repository

**Answer:** A  
**Explanation:** For SAP migrations to SAP HANA on AWS, the standard, vendor-supported methodology is the **SAP Software Update Manager (SUM) Database Migration Option (DMO) with System Move**. DMO combines database conversion (Oracle to HANA), Unicode conversion, and migration into a single workflow. Combined with a dedicated 10 Gbps Direct Connect connection, it allows pre-replicating data and executing final delta synchronization within the 6-hour cutover window.  
**Distractor Analysis:**
- *B is incorrect:* Manual CSV export and console upload is unviable for a 20 TB production SAP database.
- *C is incorrect:* Snowcone is an 8 TB edge device with insufficient capacity for a 20 TB database.
- *D is incorrect:* Serverless Application Repository is for serverless component sharing.

---

### Question 14 (Domain 1: Organizational Complexity)
An enterprise with multiple AWS accounts uses AWS IAM Identity Center (SSO). The security team requires that access to sensitive production accounts must be dynamically granted based on the employee's department and job code attributes passed from the corporate Active Directory identity provider, without creating individual IAM permission sets for every user.

**Which IAM architecture satisfies this requirement?**
- A. Role-Based Access Control (RBAC) with hardcoded IAM users
- B. **Attribute-Based Access Control (ABAC)** using SAML attributes as session tags (`aws:PrincipalTag`) in IAM Identity Center permission sets and resource policies.
- C. Sharing the AWS management account root password with department managers.
- D. Granting `AdministratorAccess` to all users and auditing with CloudTrail.

**Answer:** B  
**Explanation:** **Attribute-Based Access Control (ABAC)** leverages user attributes (such as department, cost center, or job code) passed from the external IdP as session tags (`aws:PrincipalTag`). Policies evaluate these tags against matching resource tags (`aws:ResourceTag/Department`), allowing access permissions to scale dynamically without authoring or managing individual roles and permission sets for every user.  
**Distractor Analysis:**
- *A is incorrect:* Hardcoded RBAC requires creating and modifying permissions every time an employee changes projects or departments.
- *C & D are incorrect:* Catastrophic security and compliance violations.

---

### Question 15 (Domain 2: Design for New Solutions)
A company is designing an asynchronous order processing pipeline. The pipeline receives order messages from an API. Orders must be processed strictly in the order they were placed on a per-customer basis. Duplicate order messages must be automatically detected and discarded within a 5-minute deduplication window.

**Which AWS messaging service satisfies these constraints?**
- A. Amazon SQS Standard Queue
- B. **Amazon SQS FIFO Queue** with `MessageGroupId` set to the customer ID and `MessageDeduplicationId` enabled.
- C. Amazon SNS Topic with standard subscribers.
- D. Amazon Kinesis Video Streams.

**Answer:** B  
**Explanation:** **Amazon SQS FIFO (First-In-First-Out)** queues guarantee that message order is strictly preserved. Setting the `MessageGroupId` to the customer ID ensures that messages belonging to the same customer are processed sequentially in exact order, while messages for different customers can be processed in parallel. Content-based deduplication or explicit `MessageDeduplicationId` guarantees that duplicate messages published within a 5-minute interval are automatically discarded.  
**Distractor Analysis:**
- *A is incorrect:* SQS Standard queues provide best-effort ordering and at-least-once delivery; messages can arrive out of order and duplicates can occur.
- *C is incorrect:* Standard SNS topics do not guarantee message ordering or deduplication.
- *D is incorrect:* Kinesis Video Streams is for streaming video/audio.

---

### Question 16 (Domain 3: Continuous Improvement)
An enterprise has 200 Amazon EC2 instances running web applications across 3 Availability Zones behind an Application Load Balancer. The instances are managed by an Auto Scaling Group. During an Availability Zone impairment event, instances in the impaired AZ become unresponsive, but ALB health checks take 3 minutes to declare instances unhealthy and reroute traffic. The company requires instant, automated traffic shifting away from impaired Availability Zones.

**Which AWS feature provides automated AZ evacuation without waiting for individual instance health check timeouts?**
- A. AWS Route 53 Application Recovery Controller (ARC) **Zonal Autoshift**
- B. Manually editing the Auto Scaling Group minimum capacity.
- C. Restarting the Application Load Balancer via the AWS CLI.
- D. Disabling cross-zone load balancing.

**Answer:** A  
**Explanation:** **Amazon Route 53 Application Recovery Controller (ARC) Zonal Autoshift** is an automated capability that detects when an Availability Zone experiences an impairment (such as power, networking, or hardware degradation) and automatically shifts application traffic away from the impaired AZ to healthy AZs in the region, restoring application availability within seconds without manual intervention.  
**Distractor Analysis:**
- *B is incorrect:* Manual capacity changes require human diagnosis and intervention during an active incident.
- *C is incorrect:* Restarting the ALB causes total application outage.
- *D is incorrect:* Disabling cross-zone load balancing does not shift traffic away from the impaired AZ.

---

### Question 17 (Domain 4: Migration & Modernization)
A logistics firm operates an on-premises mainframe application that produces 500,000 fixed-width flat files daily that must be ingested into an Amazon S3 data lake. The enterprise needs to automatically parse the fixed-width fields into structured formats, enrich records with geolocation data, and write the output in Apache Parquet format partitioned by date.

**Which serverless data integration service should the architect choose?**
- A. AWS Glue ETL job with a custom script and dynamic frames
- B. Amazon EC2 instances running bash scripts
- C. AWS Storage Gateway Tape Gateway
- D. Amazon CloudWatch Logs Insights

**Answer:** A  
**Explanation:** **AWS Glue** is a serverless data integration service that discovers, extracts, transforms, and loads (ETL) data. Glue ETL jobs natively process fixed-width, CSV, and JSON data using DynamicFrames, perform transformations and data enrichment, and write output in optimized Apache Parquet format with automated S3 hive-style partitioning (`year=YYYY/month=MM/day=DD`) without managing servers.  
**Distractor Analysis:**
- *B is incorrect:* Self-managed EC2 instances running bash scripts lack fault tolerance, autoscaling, and high operational overhead.
- *C is incorrect:* Tape Gateway is for backup archiving.
- *D is incorrect:* CloudWatch Logs Insights is a query engine for logs, not an ETL data pipeline.

---

### Question 18 (Domain 2: Design for New Solutions)
An enterprise is designing a multi-region disaster recovery architecture for an API running on Amazon Elastic Container Service (Amazon ECS). The RTO target is 1 minute, and the RPO target is 0 seconds (zero data loss). The data store is a document database.

**Which database tier satisfies these strict RPO and RTO requirements?**
- A. Amazon DocumentDB with cross-region snapshot replication.
- B. **Amazon DynamoDB Global Tables** deployed in active-active configuration across both regions.
- C. Self-managed MongoDB on EC2 with daily backups.
- D. Amazon RDS PostgreSQL Multi-AZ in `us-east-1` with read replica in `us-west-2`.

**Answer:** B  
**Explanation:** **Amazon DynamoDB Global Tables** provides fully managed active-active multi-region replication with near-real-time propagation (typically under 1 second). Because all regions are active writers and all data is synchronized continuously, an entire region failure allows traffic to be switched to the surviving region instantly ($RTO < 1$ minute) with zero data loss ($RPO = 0$).  
**Distractor Analysis:**
- *A is incorrect:* DocumentDB cross-region snapshots have an RPO of hours and an RTO of tens of minutes to restore.
- *C is incorrect:* Daily MongoDB backups result in an RPO of up to 24 hours.
- *D is incorrect:* RDS cross-region read replicas are asynchronous and do not support active-active writes or an RPO of 0.

---

### Question 19 (Domain 1: Organizational Complexity)
An enterprise needs to share a central Amazon OpenSearch Service domain hosted in the Logging account with 50 workload accounts in AWS Organizations. Workload accounts must send search queries to the OpenSearch cluster over private AWS networking without exposing the cluster to the public Internet or creating complex VPC peering meshes.

**Which networking service enables this secure private cross-account access?**
- A. AWS PrivateLink (VPC Interface Endpoints) backed by an internal Network Load Balancer in the Logging account.
- B. Public IP addresses on the OpenSearch domain with IP whitelists.
- C. Internet Gateways in every VPC.
- D. AWS Direct Connect.

**Answer:** A  
**Explanation:** An internal Network Load Balancer (NLB) can be placed in front of the Amazon OpenSearch domain in the Logging account and registered as an **AWS PrivateLink Endpoint Service**. Workload accounts create Interface VPC Endpoints in their local VPCs. All search traffic traverses the AWS private network securely without internet exposure, routing conflicts, or full mesh VPC peering.  
**Distractor Analysis:**
- *B & C are incorrect:* Exposing OpenSearch via public IP addresses violates the security mandate.
- *D is incorrect:* Direct Connect connects on-premises networks to AWS; it is not the service for inter-account VPC connectivity.

---

### Question 20 (Domain 3: Continuous Improvement)
An enterprise operates a multi-account AWS environment. The CFO notices that teams are provisioning expensive Amazon EC2 GPU instances (`p4d.24xlarge`) and high-memory instances (`r5b.24xlarge`) for development and testing environments, resulting in \$120,000 of monthly budget overruns.

**Which preventative governance mechanism should the architect implement immediately?**
- A. Write an AWS Lambda function that checks for running instances every morning and terminates them.
- B. Attach a **Service Control Policy (SCP)** to the Development Organizational Unit (OU) that denies `ec2:RunInstances` for non-approved instance families (e.g. denying any instance type not matching `t3.*`, `t4g.*`, `m6i.*`).
- C. Send an email to all developers asking them to stop launching large instances.
- D. Configure AWS Cost Anomaly Detection to notify the finance team after the billing cycle ends.

**Answer:** B  
**Explanation:** An SCP attached to the Development OU provides a hard, preventative guardrail that cannot be bypassed by developers or member account administrators. Using the `ec2:InstanceType` condition key with a `StringNotLike` or `StringNotEquals` condition, the SCP denies the `ec2:RunInstances` API call whenever an unapproved instance type (such as `p4d` or `r5b`) is requested.  
**Distractor Analysis:**
- *A is incorrect:* Terminating instances reactively still incurs partial hourly costs and disrupts developer workflows.
- *C is incorrect:* Manual emails do not prevent accidental or unauthorized provisioning.
- *D is incorrect:* Notifying after the billing cycle is detective and does not prevent budget overruns.

---

### Question 21 (Domain 2: Design for New Solutions)
An application requires storing billions of IoT sensor data points per month. Each record is 500 bytes and consists of a `DeviceID`, `Timestamp`, and sensor readings. Queries typically retrieve sensor readings for a specific `DeviceID` within a specific date range. Data older than 90 days is rarely accessed and should be automatically archived to low-cost storage.

**Which database architecture satisfies these performance and lifecycle requirements?**
- A. Amazon Relational Database Service (Amazon RDS) PostgreSQL with a single table.
- B. **Amazon DynamoDB** with `DeviceID` as the Partition Key and `Timestamp` as the Sort Key. Configure **DynamoDB Time to Live (TTL)** to expire items after 90 days, and use DynamoDB Streams with Amazon Data Firehose to archive expired items into Amazon S3.
- C. Amazon Elastic Block Store (EBS) volumes attached in RAID 0.
- D. Amazon Redshift with single-node cluster.

**Answer:** B  
**Explanation:** DynamoDB is ideally suited for key-value and time-series IoT data. Using `DeviceID` as the partition key and `Timestamp` as the sort key enables efficient range queries for a device over time. Enabling **DynamoDB Time to Live (TTL)** automatically purges items older than 90 days at zero cost. DynamoDB Streams captures the deleted items and streams them via Firehose directly into Amazon S3 for long-term low-cost archiving.  
**Distractor Analysis:**
- *A is incorrect:* A single PostgreSQL table will experience massive index bloat and vacuuming overhead at billions of rows per month.
- *C is incorrect:* EBS RAID 0 is block storage without database querying or automated lifecycle management.
- *D is incorrect:* A single-node Redshift cluster is not designed for continuous high-throughput IoT writes.

---

### Question 22 (Domain 4: Migration & Modernization)
A bank is migrating an on-premises IBM WebSphere application running on AIX hardware to AWS. The application code is Java-based and connects to an on-premises Oracle database. The bank wants to run the Java application in containerized microservices on Amazon EKS while keeping the Oracle database on-premises during the initial migration phase.

**Which hybrid connectivity and container migration strategy should be chosen?**
- A. Rehost the AIX operating system directly onto Amazon EC2 using AWS MGN.
- B. Replatform the Java application into container images using Docker/BuildKit, deploy to Amazon EKS in private subnets, and establish an **AWS Direct Connect** connection to communicate securely with the on-premises Oracle database with low latency.
- C. Migrate the AIX hardware using AWS Snowball.
- D. Expose the on-premises Oracle database directly to the public Internet.

**Answer:** B  
**Explanation:** AIX is a proprietary IBM UNIX operating system running on PowerPC processors; it cannot be rehosted natively onto standard x86/ARM Amazon EC2 instances via MGN. However, Java applications are portable across platforms. Replatforming the Java application into standard Linux container images deployed on Amazon EKS, combined with dedicated low-latency **AWS Direct Connect** connectivity to the on-premises Oracle database, fulfills the migration requirements.  
**Distractor Analysis:**
- *A is incorrect:* AWS MGN does not support migrating non-x86 AIX operating systems to EC2.
- *C is incorrect:* Snowball cannot run or emulate AIX hardware.
- *D is incorrect:* Exposing a production banking Oracle database to the public Internet violates security and compliance standards.

---

### Question 23 (Domain 1: Organizational Complexity)
An enterprise with 60 AWS accounts needs to share a central Amazon Route 53 Resolver DNS forwarding rule for an on-premises active directory domain (`internal.corp`) with all accounts in the organization. The architecture must automatically apply the forwarding rule to any newly created VPC in any current or future account.

**How should this be implemented with the LEAST administrative effort?**
- A. Create the Route 53 Resolver Rule in every account manually.
- B. Create the Route 53 Outbound Resolver Rule in the central Network account, share the rule with the entire AWS Organization using **AWS Resource Access Manager (RAM)**, and configure an AWS Control Tower lifecycle event or EventBridge rule to automatically associate the shared rule with new VPCs.
- C. Require developers to submit a support ticket to configure DNS.
- D. Modify `/etc/resolv.conf` on every EC2 instance using a cron job.

**Answer:** B  
**Explanation:** AWS Resource Access Manager (RAM) allows sharing Route 53 Resolver rules seamlessly across an entire AWS Organization. Combining RAM sharing with automated event-driven association (via EventBridge capturing `CreateVpc` API calls or Control Tower customization) ensures zero-touch DNS integration for all existing and future VPCs.  
**Distractor Analysis:**
- *A is incorrect:* Manually creating rules in 60+ accounts introduces massive administrative overhead and configuration drift.
- *C is incorrect:* Manual support ticketing slows down deployment velocity.
- *D is incorrect:* Modifying local files does not scale and breaks cloud-native DNS resolution.

---

### Question 24 (Domain 3: Continuous Improvement)
An enterprise runs an e-commerce platform on AWS. The platform experiences significant seasonal traffic spikes during the holidays (November–December), requiring 500 EC2 instances, while baseline traffic throughout the rest of the year requires only 100 EC2 instances.

**Which EC2 pricing strategy provides the HIGHEST cost savings while maintaining capacity flexibility?**
- A. Purchase 3-Year All-Upfront Standard Reserved Instances for 500 instances.
- B. Purchase a 3-Year All-Upfront **Compute Savings Plan** sized for the baseline load of 100 instances, and use a combination of **Spot Instances** and On-Demand instances managed by an EC2 Auto Scaling Group with a mixed instances policy for the seasonal holiday surge.
- C. Run all 500 instances On-Demand year-round.
- D. Purchase 1-Year Convertible Reserved Instances for 500 instances every November.

**Answer:** B  
**Explanation:** A 3-Year Compute Savings Plan covers the consistent baseline workload (100 instances) at the maximum discount rate (up to 66%). For unpredictable and seasonal traffic spikes, using Auto Scaling with a **Mixed Instances Policy** combining Spot instances (for stateless web/worker tiers offering up to 90% discount) and On-Demand instances provides massive cost optimization without committing to unneeded long-term capacity.  
**Distractor Analysis:**
- *A is incorrect:* Purchasing 500 3-year RIs means paying for 400 idle instances for 10 months every year.
- *C is incorrect:* Paying On-Demand rates for baseline 24/7 workloads is the most expensive option.
- *D is incorrect:* Purchasing 1-year RIs for seasonal spikes still forces paying for instances for 10 months after the holiday season ends.

---

### Question 25 (Domain 2: Design for New Solutions)
An application running on Amazon EC2 instances in an Auto Scaling Group in private subnets needs to securely retrieve database credentials from AWS Secrets Manager. The application processes millions of requests daily. The security team mandates that credentials must NEVER traverse the public Internet, and secrets access must be audited in CloudTrail.

**Which architectural components satisfy these requirements?**
- A. Deploy an Internet Gateway and assign public IP addresses to all EC2 instances.
- B. Create an **AWS Secrets Manager VPC Interface Endpoint (AWS PrivateLink)** in the private subnets. Attach an IAM role to the EC2 instances with permissions restricted to `secretsmanager:GetSecretValue` for the specific secret ARN.
- C. Store the database credentials in a public GitHub repository.
- D. Hardcode the credentials in the application source code.

**Answer:** B  
**Explanation:** Creating an Interface VPC Endpoint (powered by AWS PrivateLink) for AWS Secrets Manager provides private, internal network connectivity from private subnets directly to the Secrets Manager service without requiring an Internet Gateway, NAT Gateway, or public IP addresses. Attaching an IAM instance profile allows temporary STS credentials to invoke `GetSecretValue` with least privilege, and all API calls are logged in AWS CloudTrail.  
**Distractor Analysis:**
- *A is incorrect:* Violates the strict mandate that traffic must never traverse the public Internet.
- *C & D are incorrect:* Catastrophic security vulnerabilities that expose database credentials.
