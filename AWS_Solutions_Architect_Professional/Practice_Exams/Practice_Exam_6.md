# AWS Certified Solutions Architect - Professional (SAP-C02) Practice Exam 6

**Duration:** 60 Minutes (15 Full-Length Multi-Paragraph Scenario Questions with Distractor Analysis)  
**Passing Score Target:** $\ge 12 / 15$ (80%)  
**Exam Blueprint Focus:** Domain 1 (Organizational Complexity: 26%), Domain 2 (Design for New Solutions: 29%), Domain 3 (Continuous Improvement: 25%), Domain 4 (Migration & Modernization: 20%)

---

### Question 1 (Domain 1: Organizational Complexity)
An enterprise operates a centralized security inspection architecture using AWS Transit Gateway attached to a centralized Inspection VPC in `us-east-1`. An AWS Network Firewall is deployed across three Availability Zones. During an incident drill, an entire AWS Availability Zone experiences an outage. Inbound and outbound traffic through the remaining two AZs fails for several minutes due to routing asymmetry.

**Which architectural feature ensures symmetric return traffic routing across AZs for stateful firewalls on Transit Gateway?**
- A. Transit Gateway Appliance Mode enabled on the Transit Gateway VPC attachment to the Inspection VPC.
- B. Cross-zone load balancing enabled on an internal Application Load Balancer.
- C. Multipath BGP routing with Equal-Cost Multi-Path (ECMP) enabled on AWS Direct Connect.
- D. AWS Global Accelerator with endpoint health checks.

**Answer:** A  
**Explanation:** Transit Gateway **Appliance Mode** ensures that bidirectional traffic between spoke VPCs and the inspection VPC is pinned to the same Availability Zone for the lifetime of a TCP/UDP session, preventing stateful firewall drops caused by cross-AZ routing asymmetry.  
**Distractor Analysis:**
- *B is incorrect:* Cross-zone load balancing distributes HTTP requests across targets in different AZs, but does not enforce symmetric L3/L4 return path routing for network firewalls.
- *C is incorrect:* ECMP load balances across multiple physical links, but does not solve Transit Gateway inter-AZ routing asymmetry.
- *D is incorrect:* Global Accelerator routes traffic across public edge locations, not internal VPC inspection paths.

---

### Question 2 (Domain 2: Design for New Solutions)
A gaming company requires a low-latency, scalable player leaderboard system. The leaderboard tracks real-time scores for 10 million concurrent active players. The architecture must support retrieving the top 100 players globally in under 5 milliseconds and retrieving an individual player's relative rank in real time.

**Which database architecture satisfies these performance and query requirements?**
- A. Amazon ElastiCache for Redis using Redis Sorted Sets (`ZADD`, `ZREVRANGE`, `ZREVRANK`).
- B. Amazon RDS MySQL with a B-tree index on the `score` column.
- C. Amazon S3 storing JSON player files queried via Amazon Athena.
- D. Amazon DynamoDB using an On-Demand table with a Global Secondary Index (GSI) on score.

**Answer:** A  
**Explanation:** Redis Sorted Sets (`ZSET`) are specifically designed for real-time leaderboards. Commands like `ZADD` insert scores in $O(\log(N))$ time, and `ZREVRANGE` retrieves the top 100 players in sub-millisecond in-memory time, easily handling 10 million players with single-digit millisecond latency.  
**Distractor Analysis:**
- *B is incorrect:* Relational databases running `ORDER BY score DESC LIMIT 100` against 10 million rows will experience severe disk I/O bottlenecks and high latency under heavy write concurrency.
- *C is incorrect:* Athena queries take seconds to minutes, which fails the 5 ms latency requirement.
- *D is incorrect:* DynamoDB GSIs with high-cardinality score sorting suffer from hot partition issues and cannot compute relative player rank natively without custom pagination.

---

### Question 3 (Domain 4: Migration & Modernization)
An on-premises data center hosts a legacy Windows file sharing cluster (SMB) totaling 60 TB used by 5,000 corporate domain-joined Windows laptops. The business is closing the data center and migrating all files to AWS. The cloud file system must integrate with on-premises Active Directory via Kerberos, support Windows ACLs and shadow copies, and allow laptops to access files over an AWS Direct Connect connection with low latency.

**Which AWS service should the solutions architect recommend?**
- A. **Amazon FSx for Windows File Server** deployed in a Multi-AZ configuration and joined to the self-managed Active Directory domain.
- B. Amazon EFS mounted over Samba on an EC2 instance.
- C. Amazon S3 with AWS Transfer Family.
- D. AWS Storage Gateway Volume Gateway.

**Answer:** A  
**Explanation:** Amazon FSx for Windows File Server provides native, fully managed Windows file storage built on Windows Server, with full support for SMB protocol, NTFS file permissions, Active Directory integration, DFS namespaces, and shadow copies.  
**Distractor Analysis:**
- *B is incorrect:* EFS is a Linux POSIX file system; running an EC2 Samba proxy introduces high operational overhead and loses native Windows ACL fidelity.
- *C is incorrect:* Transfer Family is for SFTP/FTP/FTPS transfers, not transparent SMB shared drive mounting for Windows laptops.
- *D is incorrect:* Volume Gateway exposes iSCSI block storage, not native multi-user Windows SMB file shares.

---

### Question 4 (Domain 3: Continuous Improvement)
An enterprise operates a high-traffic e-commerce application on AWS. An analysis of the monthly AWS bill reveals that data transfer out (DTO) from Amazon S3 and Amazon EC2 to the public internet accounts for 40% of the total cloud spend ($60,000/month). The website serves static images, JavaScript files, and CSS stylesheets to global web browsers.

**Which architectural adjustment reduces data transfer costs with the HIGHEST percentage savings?**
- A. Deploy an **Amazon CloudFront** distribution in front of the S3 and EC2 origins to cache static assets globally.
- B. Enable S3 Transfer Acceleration on the origin bucket.
- C. Compress all images using gzip on EC2 instances.
- D. Migrate S3 buckets to S3 Glacier Deep Archive.

**Answer:** A  
**Explanation:** Amazon CloudFront drastically reduces Data Transfer Out (DTO) costs. Data transfer from S3/EC2 origins to CloudFront edge locations is **$0.00/GB (free)**. CloudFront DTO rates to the internet are significantly lower than direct S3/EC2 internet egress rates, and high edge cache hit ratios eliminate origin egress traffic entirely.  
**Distractor Analysis:**
- *B is incorrect:* S3 Transfer Acceleration incurs additional data transfer fees; it accelerates uploads, not download egress costs.
- *C is incorrect:* Compression helps marginally, but does not provide the massive architectural cost reduction of CDN caching.
- *D is incorrect:* S3 Glacier Deep Archive is for cold archiving; assets cannot be served directly to web browsers in real time.

---

### Question 5 (Domain 2: Design for New Solutions)
A healthcare company processes sensitive patient medical records (HIPAA compliance). Data must be encrypted in transit using TLS 1.3 and encrypted at rest using a dedicated, single-tenant hardware security module (HSM) that is FIPS 140-2 Level 3 validated. The company must have exclusive cryptographic ownership of the root keys, with no AWS personnel having access under any circumstances.

**Which encryption service satisfies this FIPS 140-2 Level 3 requirement?**
- A. **AWS CloudHSM**.
- B. AWS Key Management Service (AWS KMS) multi-tenant key store.
- C. AWS Secrets Manager with customer managed keys.
- D. Amazon S3 server-side encryption with Amazon S3 managed keys (SSE-S3).

**Answer:** A  
**Explanation:** **AWS CloudHSM** provides dedicated, single-tenant physical HSM appliances inside the customer's VPC that are certified FIPS 140-2 Level 3. The customer maintains exclusive cryptographic ownership; AWS has zero access to the key material or HSM admin credentials.  
**Distractor Analysis:**
- *B is incorrect:* Standard AWS KMS hardware is validated to FIPS 140-2 Level 2 (with overall Level 3 cryptographic module, but shared multi-tenant infrastructure).
- *C is incorrect:* Secrets Manager relies on KMS.
- *D is incorrect:* SSE-S3 uses AWS-managed multi-tenant keys without FIPS 140-2 Level 3 single-tenant control.

---

### Question 6 (Domain 1: Organizational Complexity)
An enterprise with 80 AWS accounts needs to enforce consistent tag compliance across all EC2 instances, EBS volumes, and S3 buckets. All resources must contain the mandatory tags `CostCenter`, `Environment`, and `Owner`. The platform team needs an automated mechanism to prevent the creation of non-compliant resources and to report existing non-compliant resources across the organization.

**Which combination of AWS features provides preventative and detective tag governance?**
- A. Define an **AWS Organizations Tag Policy** to standardize tag keys and allowed values. Attach an **SCP** that denies creation of resources unless the mandatory tag keys are present. Use **AWS Resource Groups Tag Editor** and **AWS Config** to audit existing compliance.
- B. Run an EC2 cron job that executes Python scripts across accounts using STS AssumeRole.
- C. Send weekly email reminders to all AWS account administrators.
- D. Restrict AWS Management Console access to read-only.

**Answer:** A  
**Explanation:** Tag Policies standardize allowed tag keys and values across an entire AWS Organization. SCPs enforce preventative guardrails (blocking API calls without mandatory tags), and AWS Config provides continuous detective compliance auditing across all member accounts.  
**Distractor Analysis:**
- *B is incorrect:* Custom EC2 scripts introduce maintenance overhead and lack native organizational policy enforcement.
- *C is incorrect:* Manual emails are not automated governance.
- *D is incorrect:* Making console access read-only prevents developers from doing productive work.

---

### Question 7 (Domain 3: Continuous Improvement)
A company hosts a web application on Amazon EC2 instances fronted by an Application Load Balancer. The application experiences sporadic Layer 7 Distributed Denial of Service (DDoS) attacks consisting of HTTP GET floods targeting the search endpoint `/search?q=...`. The attacks saturate EC2 CPU and crash the backend database.

**Which solution mitigates this Layer 7 DDoS attack with the LEAST latency impact?**
- A. Deploy **AWS WAF** associated with the Application Load Balancer. Create a **Rate-based rule** that limits requests to the `/search` path to 2,000 requests per 5-minute window per client IP, and apply the AWS Managed Rules **Common Rule Set (CRS)** and **Amazon IP Reputation List**.
- B. Deploy AWS Shield Standard on Amazon Route 53.
- C. Write an iptables firewall script on each EC2 instance.
- D. Scale up the backend RDS database instances.

**Answer:** A  
**Explanation:** AWS WAF inspects L7 HTTP/HTTPS traffic at the load balancer. A rate-based rule automatically tracks request volume per client IP address and temporarily blocks IPs that exceed the threshold (e.g. 2,000 requests per 5 minutes on `/search`), mitigating HTTP floods before traffic ever reaches EC2 instances.  
**Distractor Analysis:**
- *B is incorrect:* Shield Standard protects against L3/L4 volumetric attacks (SYN floods, UDP reflection), not L7 application-layer HTTP floods.
- *C is incorrect:* Host-level iptables still consumes EC2 compute and network bandwidth to process the flood.
- *D is incorrect:* Scaling the database is an expensive anti-pattern that does not stop malicious traffic.

---

### Question 8 (Domain 4: Migration & Modernization)
A media archive company has 1.2 PB of raw video files stored in an on-premises NetApp NAS device. The files must be migrated to an Amazon S3 bucket. The company has a 10 Gbps AWS Direct Connect connection. The migration must preserve POSIX metadata (timestamps, ownership, permissions), verify data integrity using end-to-end checksums, and run continuously in the background without disrupting daily video editors.

**Which migration tool should the solutions architect deploy?**
- A. **AWS DataSync**.
- B. `aws s3 sync` via AWS CLI.
- C. AWS Snowball Edge Storage Optimized.
- D. AWS Storage Gateway File Gateway.

**Answer:** A  
**Explanation:** **AWS DataSync** is purpose-built for high-performance online data transfers. It natively supports NFS and SMB, preserves POSIX file metadata, performs automatic cryptographic checksum verification in flight and at rest, and can saturate 10 Gbps Direct Connect links with built-in bandwidth throttling.  
**Distractor Analysis:**
- *B is incorrect:* AWS S3 CLI does not natively preserve full POSIX metadata (UID, GID, permissions) and is significantly slower than DataSync.
- *C is incorrect:* Physical Snowball appliances are unnecessary when a dedicated 10 Gbps Direct Connect is already available.
- *D is incorrect:* File Gateway is an ongoing hybrid caching gateway, not an accelerated bulk migration agent.

---

### Question 9 (Domain 2: Design for New Solutions)
An IoT agricultural monitoring system collects soil moisture data from 50,000 field sensors. Sensors transmit data over MQTT. The backend architecture must ingest MQTT messages, authenticate each sensor using X.509 client certificates, and route readings to an analytics pipeline. The solution must scale elastically without managing server infrastructure.

**Which AWS service provides native MQTT ingestion and X.509 mutual authentication?**
- A. **AWS IoT Core**.
- B. Amazon API Gateway.
- C. Amazon Kinesis Data Streams.
- D. Network Load Balancer with TLS termination.

**Answer:** A  
**Explanation:** **AWS IoT Core** provides a fully managed serverless message broker supporting MQTT, WebSockets, and HTTP. It natively enforces mutual TLS (mTLS) authentication using X.509 client certificates and routes telemetry through the IoT Rules Engine to downstream AWS services (DynamoDB, SQS, Kinesis, Lambda).  
**Distractor Analysis:**
- *B is incorrect:* API Gateway does not support persistent stateful MQTT protocol connections.
- *C is incorrect:* Kinesis Data Streams does not speak MQTT or authenticate client X.509 certificates directly.
- *D is incorrect:* NLB terminates TCP/TLS, but does not provide an MQTT topic broker or rules engine.

---

### Question 10 (Domain 1: Organizational Complexity)
A financial enterprise requires a dedicated network connection between its on-premises headquarters and its AWS VPCs. The architecture requires 99.99% (four 9s) high availability SLA for the hybrid connection to support real-time securities trading.

**Which AWS Direct Connect topology achieves the 99.99% High Availability SLA recommended by AWS?**
- A. Deploy two Direct Connect connections terminating at a single Direct Connect location on a single customer router.
- B. Deploy **four Direct Connect connections across two separate Direct Connect locations** (two connections per location) terminating on **two separate customer premise routers**, using redundant Transit Virtual Interfaces (Transit VIFs) attached to an AWS Transit Gateway.
- C. Deploy a single Direct Connect connection with an IPsec VPN backup.
- D. Deploy two AWS Site-to-Site VPN connections over the public internet.

**Answer:** B  
**Explanation:** The AWS Direct Connect High Availability Architecture framework specifies that achieving a **99.99% SLA** requires dual Direct Connect locations (protecting against data center outages) with dual physical connections at each location (protecting against device/fiber failure) terminating on separate customer edge routers.  
**Distractor Analysis:**
- *A is incorrect:* A single Direct Connect location is vulnerable to location-wide power or fiber cuts, qualifying only for a 99.9% SLA.
- *C is incorrect:* Direct Connect with VPN backup qualifies only for a 99.9% SLA due to bandwidth disparity and public internet variability.
- *D is incorrect:* Internet VPNs do not provide a 99.99% SLA.

---

### Question 11 (Domain 3: Continuous Improvement)
An analytics application queries a 20 TB Amazon Aurora MySQL database. Analytical reports run for 45 minutes and execute complex table joins that degrade transactional (OLTP) performance for online shoppers. The analytics queries only need data that is updated within 15 minutes of real time.

**Which architecture isolates analytical query load from the transactional database with the LOWEST cost?**
- A. Create an **Aurora Read Replica** dedicated to analytics queries and configure the analytics reporting tool to connect to the read replica endpoint.
- B. Export the Aurora database daily to Amazon S3 using AWS Glue and query with Athena.
- C. Scale up the Aurora primary instance to `db.r6g.16xlarge`.
- D. Migrate the entire database to Amazon Redshift.

**Answer:** A  
**Explanation:** Aurora Read Replicas share the same distributed storage volume as the primary writer instance with near-zero replication lag (typically $< 20\text{ ms}$). Pointing analytical queries to a dedicated read replica completely isolates CPU and memory load from the primary writer, preventing analytical queries from degrading transactional e-commerce checkout performance.  
**Distractor Analysis:**
- *B is incorrect:* Daily exports fail the requirement for data freshness within 15 minutes.
- *C is incorrect:* Scaling the primary instance increases cost without isolating analytical query thread contention.
- *D is incorrect:* Migrating the entire transactional database to Redshift is inappropriate because Redshift is an OLAP data warehouse not suited for high-concurrency transactional OLTP.

---

### Question 12 (Domain 2: Design for New Solutions)
An enterprise architecture requires users in an Amazon VPC to connect privately to an Amazon S3 bucket without traffic traversing the public internet. The security team mandates that NO data transfer charges may be incurred for internal S3 access, and no changes to existing IP routing tables or NAT Gateways should be required.

**Which type of VPC endpoint satisfies these requirements?**
- A. **S3 Gateway Endpoint**.
- B. S3 Interface Endpoint (AWS PrivateLink).
- C. Transit Gateway peering.
- D. NAT Gateway with an S3 bucket policy.

**Answer:** A  
**Explanation:** **Amazon S3 Gateway Endpoints** are provisioned directly in VPC route tables at **$0.00 cost** (no hourly endpoint fee and no per-GB data processing fee). Traffic routes privately across the AWS internal network to S3 without traversing internet gateways or NAT gateways.  
**Distractor Analysis:**
- *B is incorrect:* S3 Interface Endpoints (PrivateLink) incur hourly ENI charges ($0.01/hour) plus per-GB data processing fees ($0.01/GB).
- *C is incorrect:* Transit Gateway incurs hourly attachment and data processing costs.
- *D is incorrect:* NAT Gateway incurs significant hourly and data processing fees and routes out to public S3 endpoints.

---

### Question 13 (Domain 4: Migration & Modernization)
A retail company is migrating a legacy ASP.NET Framework monolithic application running on Windows Server 2012 R2 to AWS. The company wants to modernize the application to run as Linux containers on Amazon ECS with AWS Fargate to eliminate Windows licensing costs and reduce patching overhead.

**Which AWS modernization tool accelerates containerizing and porting this .NET application?**
- A. **AWS App2Container (A2C)** combined with **AWS Porting Assistant for .NET**.
- B. AWS Application Migration Service (MGN).
- C. AWS Schema Conversion Tool (SCT).
- D. AWS DataSync.

**Answer:** A  
**Explanation:** **AWS Porting Assistant for .NET** scans legacy .NET Framework source code and identifies incompatible APIs, recommending .NET Core / .NET 8 Linux-compatible replacements. **AWS App2Container (A2C)** automatically analyzes running applications, generates container images, and creates ECS task definitions and deployment pipelines.  
**Distractor Analysis:**
- *B is incorrect:* MGN performs lift-and-shift of the Windows VM, which preserves Windows licensing and does not modernize to Linux containers.
- *C is incorrect:* SCT is for database schema conversion.
- *D is incorrect:* DataSync migrates file systems.

---

### Question 14 (Domain 1: Organizational Complexity)
A healthcare compliance audit requires an organization to demonstrate that all data stored in Amazon S3 buckets across all member accounts is encrypted using customer-managed AWS KMS keys (SSE-KMS), that AWS KMS key policies restrict access to specific authorized roles, and that key rotation is enabled on all KMS keys.

**Which combination of AWS services should be used to continuously monitor and report compliance across all accounts?**
- A. **AWS Config** with organizational conformance packs and **AWS Security Hub**.
- B. Amazon CloudWatch alarms on billing metrics.
- C. AWS IAM Access Analyzer for S3 only.
- D. AWS Trusted Advisor via email digest.

**Answer:** A  
**Explanation:** AWS Config organizational conformance packs package compliance rules (such as `s3-bucket-server-side-encryption-enabled` and `cmk-backing-key-rotation-enabled`) and deploy them across all accounts in an AWS Organization. AWS Security Hub aggregates findings into a centralized dashboard mapped to regulatory standards (CIS AWS Foundations, NIST, HIPAA).  
**Distractor Analysis:**
- *B is incorrect:* CloudWatch billing alarms track spend, not encryption compliance.
- *C is incorrect:* IAM Access Analyzer detects public or cross-account access, but does not verify KMS key rotation.
- *D is incorrect:* Trusted Advisor lacks customizable organizational compliance rule packs.

---

### Question 15 (Domain 2: Design for New Solutions)
A company needs to implement a global user authentication service for a mobile application. The service must support social login (Apple, Google, Facebook), enterprise SAML/OIDC federation, and built-in sign-up/sign-in flows with multi-factor authentication (MFA). The solution must handle millions of users and scale elastically without managing database infrastructure.

**Which AWS service should the solutions architect choose?**
- A. **Amazon Cognito User Pools**.
- B. AWS IAM Identity Center.
- C. Amazon DynamoDB with a custom authentication Lambda function.
- D. Amazon EC2 running Keycloak.

**Answer:** A  
**Explanation:** **Amazon Cognito User Pools** is a fully managed, serverless identity provider designed for web and mobile apps. It supports self-service sign-up, sign-in, MFA, password recovery, social identity federation (Google, Apple, Facebook), and enterprise SAML/OIDC identity providers, scaling seamlessly to millions of users.  
**Distractor Analysis:**
- *B is incorrect:* IAM Identity Center is for enterprise employee single sign-on to AWS accounts and business SaaS apps, not consumer-facing mobile application user directories.
- *C is incorrect:* Building custom authentication in Lambda/DynamoDB introduces massive security risks (token handling, crypto hashing, MFA) and high development overhead.
- *D is incorrect:* Self-managed Keycloak on EC2 requires ongoing server patching, scaling, and database maintenance.
