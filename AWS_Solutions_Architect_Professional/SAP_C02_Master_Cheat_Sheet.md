# AWS Certified Solutions Architect – Professional (SAP-C02) Master Cheat Sheet

This high-density reference consolidates the critical decision matrices, routing precedence rules, cryptographic boundaries, disaster recovery RTO/RPO pairings, bandwidth formulas, FinOps equations, service limits, and top exam traps tested on the **AWS Certified Solutions Architect – Professional (`SAP-C02`)** exam.

---

## 1. Hybrid Networking, Direct Connect & BGP Precedence Matrix

### 1.1 AWS VGW & Transit Gateway Route Evaluation Order (Highest to Lowest Priority)

> [!IMPORTANT]
> **Longest Prefix Match (LPM) ALWAYS wins first!** A `/25` route over a slow Site-to-Site VPN BGP connection will ALWAYS beat a `/24` route over a 100 Gbps Direct Connect connection. The priority list below only applies when comparing **identical prefix lengths** (e.g., two `/16` routes).

* **1 (Highest)**
  * **Route Source / Type:** **Most Specific Prefix (Longest Prefix Match)**
  * **Mechanism & Tie-Breaking Notes:** `/32` > `/28` > `/24` > `/16` > `0.0.0.0/0` regardless of connection type (DX vs VPN, Static vs BGP).
* **2**
  * **Route Source / Type:** **BGP Propagated from AWS Direct Connect**
  * **Mechanism & Tie-Breaking Notes:** Routes learned via BGP over Private VIF or Transit VIF. Evaluated further by Local Preference communities (`7224:7300` > `7224:7200` > `7224:7100`), then shortest `AS_PATH`.
* **3**
  * **Route Source / Type:** **Static Routes (VPN Static or TGW Static)**
  * **Mechanism & Tie-Breaking Notes:** Manually configured static routes on a VGW or TGW route table. *(Note: On VGW, DX BGP beats Static VPN; on TGW route tables, Static routes beat propagated BGP routes of the same prefix unless both are propagated).*
* **4**
  * **Route Source / Type:** **BGP Propagated from Site-to-Site VPN**
  * **Mechanism & Tie-Breaking Notes:** Dynamic BGP routes advertised over IPsec VPN tunnels. Evaluated by shortest `AS_PATH`, then `MED` (Multi-Exit Discriminator, lowest wins).

### 1.2 Bidirectional Traffic Engineering Summary (AWS $\leftrightarrow$ On-Premises)

```mermaid
flowchart LR
    subgraph OnPrem["On-Premises Data Centers"]
        R1["Primary Router (DC1)"]
        R2["Secondary Router (DC2)"]
    end

    subgraph AWS["AWS Cloud (DXGW + TGW)"]
        DXGW["Direct Connect Gateway"]
        TGW["AWS Transit Gateway"]
    end

    R1 -- "Outbound (On-Prem -> AWS): Local_Pref 200\nInbound (AWS -> On-Prem): Community 7224:7300" --> DXGW
    R2 -- "Outbound (On-Prem -> AWS): Local_Pref 100\nInbound (AWS -> On-Prem): Community 7224:7100 + AS_PATH Prepend" --> DXGW
    DXGW --- TGW
```

* **Controlling AWS > On-Premises Traffic (Return Path from AWS)**:
  1. **First Choice (Across any AWS Region via DXGW)**: Advertise prefixes from on-premises with **BGP Local Preference Communities** (`7224:7300` High on Primary VIF, `7224:7100` Low on Secondary VIF).
  2. **Second Choice (Same Region or VPN fallback)**: Use **`AS_PATH` Prepending** on the secondary router (e.g., prepend ASN `65001 65001 65001` so the path looks longer to AWS). *Caveat*: For Direct Connect VIFs terminating in different AWS Regions connected to a DXGW, AWS prefers the **locally connected Region** before evaluating `AS_PATH` length unless BGP community tags (`7224:7300`) are used!
  3. **Last Resort (Emergency Failover)**: Advertise a more specific subnet (`/25` instead of `/24`) over the primary link, and a summary `/24` over the backup link.
* **Controlling On-Premises > AWS Traffic (Forward Path from On-Prem)**:
  1. Configure **higher BGP Local Preference** (e.g., `LOCAL_PREF 200` vs `100`) on your internal on-premises routers for routes learned from the Primary DX VIF.
  2. Filter or match based on AWS Scope Communities (`7224:9100` Local Region vs `7224:9300` Global).

### 1.3 AWS Direct Connect BGP Community Tags Reference

* **`7224:7100`**
  * **Direction:** On-Prem > AWS
  * **Meaning / Architectural Effect:** **Low Local Preference**: Marks this DX VIF as backup/passive for traffic returning from AWS to on-premises.
* **`7224:7200`**
  * **Direction:** On-Prem > AWS
  * **Meaning / Architectural Effect:** **Medium Local Preference (Default)**: Standard priority; enables ECMP active/active load balancing if multiple VIFs in the same Region advertise equal `AS_PATH` and community.
* **`7224:7300`**
  * **Direction:** On-Prem > AWS
  * **Meaning / Architectural Effect:** **High Local Preference**: Marks this DX VIF as primary/active for traffic returning from AWS to on-premises. Beats `7224:7200` and `7224:7100` even if `AS_PATH` is longer!
* **`7224:9100`**
  * **Direction:** AWS > On-Prem
  * **Meaning / Architectural Effect:** **Local Zone / Home Region Scope**: Prefix originated in the local AWS Direct Connect point of presence.
* **`7224:9200`**
  * **Direction:** AWS > On-Prem
  * **Meaning / Architectural Effect:** **Continent / Regional Scope**: Prefix originated within the same continent (e.g., North America or Europe).
* **`7224:9300`**
  * **Direction:** AWS > On-Prem
  * **Meaning / Architectural Effect:** **Global Scope**: Prefix advertised globally across all AWS Regions via Direct Connect Gateway.
* **`NO_EXPORT`**
  * **Direction:** On-Prem > AWS
  * **Meaning / Architectural Effect:** Prevents AWS from re-advertising the prefix to other DX locations even if SiteLink is enabled.

### 1.4 Direct Connect Virtual Interfaces (VIFs) & Encryption Matrix

* **Private VIF**
  * **Target Connectivity:** **1 VGW** (1 VPC) OR **1 Direct Connect Gateway (DXGW)** (up to 20 VGWs across Regions/Accounts)
  * **Max Payload MTU:** **1500** or **9001** (Jumbo Frames)
  * **Encryption Options & Architectural Rules:** Does NOT support TGW. To encrypt: use **MACsec** (Layer 2 on 10G/100G Dedicated DX) or terminate IPsec on EC2 virtual appliances.
* **Transit VIF**
  * **Target Connectivity:** **1 Direct Connect Gateway (DXGW)** > up to **6 Transit Gateways (TGWs)** across Regions
  * **Max Payload MTU:** **1500** or **8500** (Jumbo Frames)
  * **Encryption Options & Architectural Rules:** Required for TGW connectivity. Requires **1 Gbps+** DX connection. Supports **IPsec Site-to-Site VPN over Transit VIF** (attaching VPN to TGW) or **MACsec**.
* **Public VIF**
  * **Target Connectivity:** All AWS Public IP Ranges (S3, DynamoDB, SQS, Public VPN Endpoints, EC2 Elastic IPs)
  * **Max Payload MTU:** **1500** only (No Jumbo Frames)
  * **Encryption Options & Architectural Rules:** Requires a public `/31` or `/30` IP block + BGP. **Exam Classic**: Run **AWS Site-to-Site VPN over a Public VIF** to get an encrypted IPsec tunnel to a VGW or TGW without needing a Transit VIF!

### 1.5 Direct Connect Resiliency Models & Quotas

* **Maximum Resiliency**
  * **SLA:** **99.99%**
  * **Required Topology:** **4 Dedicated Connections** across **2 separate Direct Connect Locations** (2 connections per location on distinct AWS devices) terminating on redundant customer routers.
* **High Resiliency**
  * **SLA:** **99.9%**
  * **Required Topology:** **2 Dedicated Connections** across **2 separate Direct Connect Locations** (1 connection per location).
* **Development & Test**
  * **SLA:** **No SLA**
  * **Required Topology:** 1 Connection in 1 Location (or 2 connections in the *same* location—still no 99.9% SLA against location failure!).

* **Key Direct Connect Quotas to Memorize**:
  * **Transit VIF Prefix Quota (AWS > On-Prem via DXGW)**: Maximum **200 prefixes** across all Transit Gateways attached to a DXGW (`Allowed Prefixes` list must summarize VPC CIDRs if you have $>200$ VPCs!).
  * **BGP Prefixes Advertised (On-Prem > AWS)**: Maximum **100 prefixes** per Private/Transit VIF (up to 1,000 on Public VIF); summarize on-premises routes if exceeding 100!
  * **Site-to-Site VPN Single Tunnel Bandwidth**: Maximum **1.25 Gbps** per tunnel (use **Transit Gateway + Accelerated VPN + ECMP** across multiple VPN attachments with equal BGP attributes to scale horizontally beyond 1.25 Gbps, up to 50 Gbps).

### 1.6 VPC Connectivity & Inspection Comparison Matrix

* **VPC Peering**
  * **OSI Layer:** Layer 3
  * **Overlapping IPv4 CIDRs?** **No** (Unless paired via IPv6 or NAT)
  * **Transitive?** **No**
  * **Bandwidth / Limits:** Full VPC fabric (100+ Gbps), no per-GB processing fee (only Cross-AZ/Region transfer)
  * **Primary SAP-C02 Use Case:** High-throughput, low-cost point-to-point mesh between a small number of VPCs (`<10`) with massive data volume.
* **AWS Transit Gateway (TGW)**
  * **OSI Layer:** Layer 3
  * **Overlapping IPv4 CIDRs?** **No** (Requires Private NAT GW in egress/inspection VPC)
  * **Transitive?** **Yes** (Hub-and-Spoke)
  * **Bandwidth / Limits:** 50 Gbps burst per AZ per VPC attachment
  * **Primary SAP-C02 Use Case:** Enterprise hub-and-spoke connecting 10s–1000s of VPCs, DXGW, and VPNs with segmented route tables and Centralized Inspection.
* **AWS Cloud WAN**
  * **OSI Layer:** Layer 3
  * **Overlapping IPv4 CIDRs?** **No**
  * **Transitive?** **Yes** (Global Mesh)
  * **Bandwidth / Limits:** Global AWS backbone
  * **Primary SAP-C02 Use Case:** Declarative, policy-driven global WAN connecting VPCs and SD-WAN branches across multiple Regions without manual TGW peering tables.
* **AWS PrivateLink**
  * **OSI Layer:** Layer 4 (TCP)
  * **Overlapping IPv4 CIDRs?** **Yes!**
  * **Transitive?** **No** (Unidirectional Consumer > Provider)
  * **Bandwidth / Limits:** Scales to 100 Gbps per AZ behind NLB
  * **Primary SAP-C02 Use Case:** Exposing a specific TCP service (via NLB) or third-party appliance (via GWLB) across accounts/overlapping CIDRs with zero route table conflicts.
* **Amazon VPC Lattice**
  * **OSI Layer:** Layer 7 (HTTP/HTTPS/gRPC)
  * **Overlapping IPv4 CIDRs?** **Yes!**
  * **Transitive?** **Service Network** abstraction
  * **Bandwidth / Limits:** Managed proxy mesh
  * **Primary SAP-C02 Use Case:** Modern microservice-to-microservice communication across VPCs/accounts (EKS, ECS, Lambda, EC2) with overlapping CIDRs, IAM Auth Policies, and path/weight routing.

> [!CAUTION]
> **Stateful Inspection VPC Trap (`ApplianceModeSupport`)**: Whenever traffic traverses a Transit Gateway through a Centralized Inspection VPC (hosting **AWS Network Firewall** or **Gateway Load Balancer**) between two spoke VPCs in different AZs, TGW defaults to keeping traffic in the originating AZ. Return traffic may hit a firewall node in a different AZ, which drops the packet because it never saw the TCP `SYN`! **You MUST enable Appliance Mode (`--options ApplianceModeSupport=enable`) on the Inspection VPC's TGW attachment** so TGW hashes the 5-tuple and pins both forward and return flows to the exact same AZ!

---

## 2. Multi-Account Governance, IAM & Security Matrix

### 2.1 IAM Policy Evaluation & Boundary Intersection Logic

An IAM principal gets access **ONLY** if the action lies within the **intersection** ($\cap$) of all applicable policy types and has **zero explicit `Deny` statements**:

$$\text{Effective Permissions} = (\text{SCP} \cap \text{RCP} \cap \text{Permissions Boundary} \cap \text{Session Policy} \cap (\text{Identity Policy} \cup \text{Same-Account Resource Policy})) - \text{Explicit Deny}$$

* **Service Control Policy (SCP)**
  * **Grants Permissions?** **No** (Guardrail only)
  * **Scope / Target:** AWS Organization Root, OUs, or Member Accounts
  * **Key SAP-C02 Nuances & Exceptions:** **Never affects the Management Account**. **Does NOT affect Service-Linked Roles**. Affects Member Account Root user. Restricts maximum available permissions for principals in member accounts.
* **Resource Control Policy (RCP)**
  * **Grants Permissions?** **No** (Guardrail only)
  * **Scope / Target:** Resources in Member Accounts (S3, KMS, SQS, Secrets Manager, STS)
  * **Key SAP-C02 Nuances & Exceptions:** Enforces organization-wide data perimeter controls on **resources** regardless of which external principal calls them. Never affects Management Account.
* **IAM Permissions Boundary**
  * **Grants Permissions?** **No** (Guardrail only)
  * **Scope / Target:** Specific IAM User or IAM Role
  * **Key SAP-C02 Nuances & Exceptions:** Used for **Delegated Administration** (allowing developers to create new IAM roles for Lambda/EC2 without privilege escalation by requiring an `iam:PermissionsBoundary` condition on `iam:CreateRole`).
* **Identity-Based Policy**
  * **Grants Permissions?** **Yes**
  * **Scope / Target:** IAM Users, Groups, Roles, Permission Sets
  * **Key SAP-C02 Nuances & Exceptions:** For **Cross-Account** access, BOTH the calling account's Identity Policy AND the target account's Resource-Based Policy (or Role Trust Policy) must explicitly allow the action.
* **Session Policy**
  * **Grants Permissions?** **No** (Filter only)
  * **Scope / Target:** Passed inline during `AssumeRole` / `GetFederationToken`
  * **Key SAP-C02 Nuances & Exceptions:** Further restricts the permissions of the assumed role for that specific temporary session only.

### 2.2 High-Value IAM & SCP Condition Keys Reference

* **`aws:PrincipalOrgID`:** Used in S3 Bucket Policies, KMS Key Policies, EventBridge Buses, and IAM Role Trust Policies to allow access **only to principals belonging to your AWS Organization** (`o-xxxxxxxxxx`).
* **`aws:ResourceOrgID`:** Used in SCPs/RCPs to prevent principals inside your Organization from writing data to S3 buckets or SQS queues owned by external AWS accounts (Data Exfiltration prevention).
* **`aws:PrincipalOrgPaths`:** Restricts access to principals belonging to specific **Organizational Units (OUs)** (e.g., `o-xxx/r-xxx/ou-xxx-prod/*`).
* **`aws:PrincipalArn`:** Used in `StringNotLike` inside an SCP `Deny` statement to **exempt a specific administrative/CI-CD IAM Role** or Control Tower execution role from an organization-wide restriction.
* **`aws:RequestedRegion`:** Restricts API calls to approved AWS Regions (e.g., `us-east-1`, `eu-west-1`). **Must pair with `NotAction`** for global services (`iam:*`, `sts:*`, `route53:*`, `cloudfront:*`, `support:*`).
* **`aws:ViaAWSService` / `kms:ViaService`:** Allows KMS key usage only when invoked indirectly by a specific AWS service (e.g., `s3.us-east-1.amazonaws.com`) rather than direct `kms:Decrypt` API calls.
* **`aws:SourceVpce` / `aws:SourceVpc`:** Enforces a **Network Data Perimeter** so S3 buckets or KMS keys can only be accessed via specific VPC Endpoints or VPCs.
* **`aws:CalledVia`:** Restricts direct access to a backend resource (like S3 or KMS) unless the request was made through an upstream service (like `athena.amazonaws.com` or `cloudformation.amazonaws.com`).

### 2.3 AWS Resource Access Manager (RAM) Shareable vs. Non-Shareable Resources

#### Shareable via AWS RAM (No Duplication!)
* **VPC Subnets** (Shared VPCs within the same AWS Organization only)
* **AWS Transit Gateways** & **Cloud WAN Core Networks**
* **Route 53 Resolver Forwarding Rules** & **DNS Firewall Rule Groups**
* **Amazon Aurora DB Clusters** (Cloning only), **AWS Private CA**, **License Manager Configs**
* **Customer-Managed Prefix Lists**, **Network Firewall Policies**, **EC2 Capacity Reservations**, **Outposts**

#### NOT Shareable via AWS RAM (Common Exam Distractors!)
* **Entire VPCs** or **Internet Gateways / NAT Gateways directly** (you share the *Subnets*, and participants use the host VPC's IGW/NAT/VGW/TGW)
* **IAM Roles or IAM Policies** (use Cross-Account Role Assumption or IAM Identity Center instead)
* **Route 53 Resolver Inbound / Outbound Endpoints directly** (share the *Rules* via RAM; endpoints stay in the Network Hub VPC)
* **Amazon S3 Buckets** or **KMS Keys** (shared via Bucket Policies / KMS Key Policies, NOT AWS RAM)
* **Application Load Balancers / Network Load Balancers** (participants in a shared VPC cannot view/modify another account's ALB unless targeting instances)

### 2.4 Eliminating Long-Lived Credentials Across Hybrid Environments

* **On-Premises Servers Needing General AWS API Access (S3, SQS, DynamoDB, KMS)**
  * **Recommended Zero-Key Architecture:** **IAM Roles Anywhere** + **AWS Private CA (or External X.509 PKI)**
  * **Why It Wins on SAP-C02:** Servers use `aws_signing_helper` with local X.509 client certificates to authenticate against a **Trust Anchor** and assume an IAM Role via **Profile** for temporary STS tokens. Zero static IAM access keys!
* **On-Premises Servers Needing Patching, Inventory, or Shell Access**
  * **Recommended Zero-Key Architecture:** **AWS Systems Manager (SSM) Hybrid Activations**
  * **Why It Wins on SAP-C02:** Registers non-EC2 VMs (`mi-xxxxxx`) with SSM using an Activation Code/ID, automatically rotating temporary credentials for SSM Agent operations.
* **Human Workforce Users (Developers, Admins, Analysts)**
  * **Recommended Zero-Key Architecture:** **AWS IAM Identity Center** + **External IdP (Entra ID / Okta via SAML 2.0 & SCIM)**
  * **Why It Wins on SAP-C02:** Centralized multi-account SSO using **Permission Sets** and **ABAC (`aws:PrincipalTag`)** to scale access dynamically across hundreds of accounts.
* **Mobile / Web End-Users Needing Direct S3 Upload or DynamoDB Access**
  * **Recommended Zero-Key Architecture:** **Amazon Cognito User Pools** + **Cognito Identity Pools**
  * **Why It Wins on SAP-C02:** User Pool authenticates user (JWT) > Identity Pool exchanges JWT for scoped temporary STS credentials with fine-grained IAM policy variables (`${cognito-identity.amazonaws.com:sub}`).
* **CI/CD Pipelines (GitHub Actions, GitLab CI, Kubernetes Pods)**
  * **Recommended Zero-Key Architecture:** **IAM OIDC Identity Provider** + `sts:AssumeRoleWithWebIdentity`
  * **Why It Wins on SAP-C02:** Exchanges short-lived OIDC JWT tokens directly with AWS STS for temporary IAM role credentials without storing AWS secrets in Git repositories.

---

## 3. Edge, High Availability & Disaster Recovery Matrix

### 3.1 CloudFront vs. AWS Global Accelerator

* **Core Mechanism**
  * **Amazon CloudFront:** Global CDN with **Edge Caching** (600+ PoPs & Regional Edge Caches)
  * **AWS Global Accelerator:** Global Network Layer 4 Proxy with **2 Static Anycast IPv4/IPv6 Addresses**
* **Supported Protocols**
  * **Amazon CloudFront:** **HTTP / HTTPS / WebSocket / gRPC** only
  * **AWS Global Accelerator:** **Any TCP or UDP protocol** (HTTP, Gaming UDP, VoIP, MQTT, custom TCP ports)
* **Caching Capability**
  * **Amazon CloudFront:** **Yes** — caches static & dynamic responses at the edge
  * **AWS Global Accelerator:** **No caching** — every packet traverses the AWS global fiber backbone to the origin
* **IP Address Behavior**
  * **Amazon CloudFront:** Dynamic IP pool (changes frequently; cannot whitelist single IP at corporate firewalls unless using Managed Prefix List on origin)
  * **AWS Global Accelerator:** **2 Fixed Static Anycast IPs** (ideal for legacy clients or strict B2B partner firewall allowlists that forbid DNS CNAMEs)
* **Failover Speed**
  * **Amazon CloudFront:** Origin Groups fail over on HTTP 4xx/5xx per request; DNS updates depend on client TTL
  * **AWS Global Accelerator:** **Deterministic <30-second health-check failover** across Regions without relying on client DNS cache behavior
* **Client IP Preservation**
  * **Amazon CloudFront:** Passes client IP via `X-Forwarded-For` HTTP header
  * **AWS Global Accelerator:** Native **Client IP Preservation** at network layer for ALB and EC2 origins

### 3.2 CloudFront Functions vs. Lambda@Edge

* **Execution Location**
  * **CloudFront Functions:** **218+ CloudFront Edge Locations (PoPs)** (closest to viewer)
  * **Lambda@Edge:** **13+ Regional Edge Caches**
* **Supported Triggers**
  * **CloudFront Functions:** **Viewer Request** and **Viewer Response** ONLY
  * **Lambda@Edge:** **Viewer Request**, **Origin Request**, **Origin Response**, **Viewer Response**
* **Runtime & Max Duration**
  * **CloudFront Functions:** Lightweight JavaScript (ECMAScript 5.1 / ES6+); **$< 1\text{ ms}$** execution limit
  * **Lambda@Edge:** Node.js or Python; **5 seconds** (Viewer triggers) / **30 seconds** (Origin triggers)
* **Memory & Package Size**
  * **CloudFront Functions:** 2 MB memory; **10 KB** max code size
  * **Lambda@Edge:** 128 MB to 10 GB memory; **1 MB** (Viewer) / **50 MB** (Origin) code size
* **Network / File / Body Access**
  * **CloudFront Functions:** **No network access**, no filesystem, **no HTTP request body access** (KeyValueStore supported for fast edge lookups)
  * **Lambda@Edge:** **Full network access** (can call DynamoDB, S3, external IdPs), filesystem `/tmp`, **full HTTP body access**
* **Cost & Scale**
  * **CloudFront Functions:** ~1/6th the cost of Lambda@Edge; scales to 10,000,000+ req/sec instantaneously
  * **Lambda@Edge:** Higher cost; suited for heavy compute or cache-miss origin processing
* **Primary Exam Use Cases**
  * **CloudFront Functions:** URL rewrites/redirects, HTTP header manipulation, simple JWT validation, A/B cookie normalization
  * **Lambda@Edge:** Dynamic origin routing based on DynamoDB lookup, image resizing on cache miss, SSR, complex external OAuth calls

### 3.3 Disaster Recovery (DR) Tiers & Database Pairing Matrix

* **Backup & Restore**
  * **Target RTO:** **Hours to Days** (2–24h)
  * **Target RPO:** **Hours** (Last backup snapshot)
  * **Infrastructure State in DR Region:** Nothing running; backups replicated cross-region; provisioned via CloudFormation/IaC on disaster
  * **Ideal Database & Storage Architecture:** **AWS Backup** cross-region copy, **RDS Automated Snapshots**, **S3 Cross-Region Replication (CRR)** to Glacier/Standard-IA.
* **Pilot Light**
  * **Target RTO:** **10s of Minutes** (10–30m)
  * **Target RPO:** **Seconds to Minutes**
  * **Infrastructure State in DR Region:** **Data layer is live and continuously replicating** (RDS/Aurora Replica or DRS staging); compute/app tier is **off (AMI/Launch Templates at 0 capacity)**
  * **Ideal Database & Storage Architecture:** **Aurora Global Database** (headless secondary cluster or smallest instance), **RDS Cross-Region Read Replica**, **AWS Elastic Disaster Recovery (DRS)**.
* **Warm Standby**
  * **Target RTO:** **Minutes** (1–5m)
  * **Target RPO:** **Seconds**
  * **Infrastructure State in DR Region:** **Full functional stack is running 24/7 at minimum scale** (e.g., 1 small ECS task/EC2 per AZ); scales out on failover
  * **Ideal Database & Storage Architecture:** **Aurora Global Database**, **DynamoDB Global Tables**, **ElastiCache Global Datastore**, ALB + ASG min=1/2.
* **Multi-Site Active/Active**
  * **Target RTO:** **Near-Zero / Seconds** ($<30\text{s}$)
  * **Target RPO:** **Near-Zero** ($\approx 0\text{–}1\text{s}$)
  * **Infrastructure State in DR Region:** **Full production scale in 2+ Regions** serving live traffic concurrently via Route 53 Latency/Geolocation or Global Accelerator
  * **Ideal Database & Storage Architecture:** **DynamoDB Global Tables** (Multi-Active), **Aurora Global Database with Write Forwarding** (or Managed Failover via **Route 53 ARC**), **S3 Multi-Region Access Points (MRAP)**.

### 3.4 Enterprise Database High Availability & Scaling Comparison

* **RDS Multi-AZ DB Instance**
  * **Architecture Mechanics:** 1 Primary + 1 synchronous Standby in another AZ (Standby is **NOT readable**).
  * **Failover RTO / RPO:** RTO: 60–120s | RPO: 0
  * **SAP-C02 Decision Trigger:** Standard HA for Oracle, SQL Server, PostgreSQL, MySQL when extra read capacity is not required.
* **RDS Multi-AZ DB Cluster**
  * **Architecture Mechanics:** 1 Writer + **2 readable Standbys** across 3 AZs using local NVMe SSD transaction logs + quorum commit.
  * **Failover RTO / RPO:** RTO: **~35s** | RPO: 0
  * **SAP-C02 Decision Trigger:** Requires **MySQL or PostgreSQL**, **readable standbys**, up to **2x faster write latency**, and predictable `<35s` failover without refactoring to Aurora.
* **Aurora High Availability & Replicas**
  * **Architecture Mechanics:** Shared storage volume automatically striped across **6 copies in 3 AZs** (4/6 write quorum, 3/6 read quorum) + up to **15 low-lag (`<10ms`) Aurora Replicas**.
  * **Failover RTO / RPO:** RTO: **~30s** | RPO: 0
  * **SAP-C02 Decision Trigger:** Use **Aurora Custom Endpoints** to isolate heavy BI analytics/reporting queries onto dedicated large instances away from OLTP reader instances.
* **Aurora Global Database**
  * **Architecture Mechanics:** Storage-level replication from Primary Region to up to **5 Secondary Regions** (`<1s` typical lag).
  * **Failover RTO / RPO:** Unplanned: `<1m` RTO, `~1s` RPO | Managed Planned: **0 RPO**
  * **SAP-C02 Decision Trigger:** Multi-region relational DR, local low-latency global reads, and **Write Forwarding** so secondary regions can forward occasional writes to the primary.
* **Amazon RDS Proxy**
  * **Architecture Mechanics:** Managed connection pooler sitting between compute and RDS/Aurora.
  * **Failover RTO / RPO:** Reduces failover time by **up to 66%**
  * **SAP-C02 Decision Trigger:** Mandatory when **thousands of AWS Lambda functions or Fargate tasks** cause database connection exhaustion (`too many connections`) or stale DNS caching during failover.

---

## 4. Storage, Data Transfer & Migration Decision Matrix

### 4.1 Shared File Systems Comparison (EFS vs. FSx Portfolio)

* **Amazon EFS**
  * **Protocol(s):** **NFSv4** (Linux only)
  * **Max Performance:** Tens of GB/s; `Elastic`, `Provisioned`, or `Bursting` throughput
  * **Key Capabilities & SAP-C02 Selection Trigger:** Serverless Linux shared storage for EC2, ECS, EKS, and **AWS Lambda**. Supports EFS Intelligent-Tiering (Standard, IA, Archive).
* **FSx for Windows File Server**
  * **Protocol(s):** **SMB**
  * **Max Performance:** Multi-GB/s (SSD or HDD)
  * **Key Capabilities & SAP-C02 Selection Trigger:** Native Windows storage: **Active Directory integration**, NTFS ACLs, **DFS Namespaces/Replication**, SQL Server Failover Cluster Instances (FCI), **FSx File Gateway** for branch caching.
* **FSx for Lustre**
  * **Protocol(s):** **POSIX** (Linux)
  * **Max Performance:** **Hundreds of GB/s**, millions of IOPS, sub-ms latency
  * **Key Capabilities & SAP-C02 Selection Trigger:** **HPC, Machine Learning training, financial modeling, genomics**. Native bi-directional **S3 Data Repository Association** (lazy-loads S3 objects as POSIX files and exports results back to S3). Offers `Scratch` (temporary, cheaper) vs `Persistent` deployments.
* **FSx for NetApp ONTAP**
  * **Protocol(s):** **NFS, SMB, iSCSI** (Multiprotocol)
  * **Max Performance:** Tens of GB/s + NVMe cache
  * **Key Capabilities & SAP-C02 Selection Trigger:** **Multiprotocol (Linux + Windows + Block)** access to the same dataset. Supports **NetApp SnapMirror** (hybrid migration from on-prem NetApp), **FlexClone** (instant zero-space clones for DB dev/test), inline deduplication/compression/compaction, and intelligent tiering to capacity pool.
* **FSx for OpenZFS**
  * **Protocol(s):** **NFS** (v3/v4)
  * **Max Performance:** Up to 1,000,000+ IOPS, `<0.5ms` latency
  * **Key Capabilities & SAP-C02 Selection Trigger:** Lift-and-shift for on-premises **ZFS** workloads requiring ZFS snapshots, clones, and ultra-low latency NFS.

### 4.2 S3 Storage Classes & Compliance Matrix

* **S3 Standard**
  * **Availability Zones:** $\ge 3$ AZs
  * **Min Storage Duration:** None
  * **Min Billable Object Size:** None
  * **Retrieval Fee / Time:** None / Milliseconds
  * **Ideal SAP-C02 Scenario:** Active, frequently accessed hot data, static websites, dynamic data lakes.
* **S3 Express One Zone**
  * **Availability Zones:** 1 AZ (Directory Bucket)
  * **Min Storage Duration:** 1 Hour
  * **Min Billable Object Size:** None (40 KB metadata overhead)
  * **Retrieval Fee / Time:** Lower per-request fee / **Single-digit milliseconds**
  * **Ideal SAP-C02 Scenario:** High-performance ML training, interactive analytics, financial modeling co-located in the same AZ as compute.
* **S3 Intelligent-Tiering**
  * **Availability Zones:** $\ge 3$ AZs
  * **Min Storage Duration:** None
  * **Min Billable Object Size:** None (**Objects $<128\text{ KB}$ stay in Frequent Access and are not monitored**)
  * **Retrieval Fee / Time:** **Zero retrieval fees!** (Small monthly per-object monitoring fee)
  * **Ideal SAP-C02 Scenario:** **Unpredictable or changing access patterns**; automatically moves objects between Frequent, Infrequent (30d), Archive Instant (90d), and optional async Archive (90–730d) / Deep Archive (180–730d).
* **S3 Standard-IA**
  * **Availability Zones:** $\ge 3$ AZs
  * **Min Storage Duration:** **30 Days**
  * **Min Billable Object Size:** **128 KB**
  * **Retrieval Fee / Time:** Per-GB fee / Milliseconds
  * **Ideal SAP-C02 Scenario:** Infrequently accessed data (`<1/month`) requiring high durability and immediate millisecond retrieval.
* **S3 One Zone-IA**
  * **Availability Zones:** **1 AZ**
  * **Min Storage Duration:** **30 Days**
  * **Min Billable Object Size:** **128 KB**
  * **Retrieval Fee / Time:** Per-GB fee / Milliseconds
  * **Ideal SAP-C02 Scenario:** Easily **re-creatable** infrequent data (e.g., secondary image thumbnails or on-prem backup copies where AZ loss is acceptable).
* **S3 Glacier Instant Retrieval**
  * **Availability Zones:** $\ge 3$ AZs
  * **Min Storage Duration:** **90 Days**
  * **Min Billable Object Size:** **128 KB**
  * **Retrieval Fee / Time:** Higher per-GB fee / **Milliseconds**
  * **Ideal SAP-C02 Scenario:** Medical images, news media archives accessed once per quarter but requiring **instant millisecond access** when requested.
* **S3 Glacier Flexible Retrieval**
  * **Availability Zones:** $\ge 3$ AZs
  * **Min Storage Duration:** **90 Days**
  * **Min Billable Object Size:** 40 KB metadata
  * **Retrieval Fee / Time:** **Expedited** (`1–5 min`), **Standard** (`3–5 hr`), **Bulk** (`5–12 hr`, **Free!**)
  * **Ideal SAP-C02 Scenario:** Annual compliance archives where a few minutes to several hours of retrieval delay is acceptable.
* **S3 Glacier Deep Archive**
  * **Availability Zones:** $\ge 3$ AZs
  * **Min Storage Duration:** **180 Days**
  * **Min Billable Object Size:** 40 KB metadata
  * **Retrieval Fee / Time:** **Standard** (`12 hr`), **Bulk** (`48 hr`)
  * **Ideal SAP-C02 Scenario:** Lowest-cost long-term retention (7–10 years financial/healthcare regulatory archives replacing magnetic tape).

### 4.3 Online vs. Offline Data Transfer Crossover Math

> [!TIP]
> **Memorize the SAP-C02 Bandwidth Rule of Thumb**:
> * **$100\text{ Mbps}$ link** (at 80% utilization) $\approx 10\text{ MB/s} \approx \mathbf{860\text{ GB/day}} \approx \mathbf{8.6\text{ TB in 10 days}}$
> * **$1\text{ Gbps}$ link** (at 80% utilization) $\approx 100\text{ MB/s} \approx \mathbf{8.6\text{ TB/day}} \approx \mathbf{86\text{ TB in 10 days}}$
> * **$10\text{ Gbps}$ link** (at 80% utilization) $\approx 1\text{ GB/s} \approx \mathbf{86\text{ TB/day}} \approx \mathbf{860\text{ TB in 10 days}}$

$$\text{Transfer Time (Days)} = \frac{\text{Total Data Size (Bits)}}{\text{Available Bandwidth (bps)} \times \text{Utilization Factor} \times 86,400\text{ seconds/day}}$$

* **50 TB over a 10 Gbps Direct Connect (30% free capacity = 3 Gbps) with a 7-day deadline**
  * **Recommended Migration Pattern:** **AWS DataSync** (Online over Direct Connect)
  * **Architectural Reasoning:** $3\text{ Gbps} \approx 25.8\text{ TB/day}$. $50\text{ TB}$ finishes in **~2 days** online — faster and simpler than shipping physical Snowball devices!
* **500 TB over a 1 Gbps WAN (20% free capacity = 200 Mbps) with a 30-day deadline and active daily file changes**
  * **Recommended Migration Pattern:** **Two-Phase Hybrid: Multiple AWS Snowball Edge Storage Optimized devices + AWS DataSync**
  * **Architectural Reasoning:** Online $200\text{ Mbps} \approx 1.7\text{ TB/day}$ would take **~294 days**! Ship **7 Snowball Edge Storage Optimized (80 TB usable each)** for the initial baseline seed, import to S3/FSx, then run **AWS DataSync** over the WAN to sync only the incremental delta changes before cutover.
* **60 TB Oracle DB to Aurora PostgreSQL over a 500 Mbps link with `<15 minute` cutover downtime**
  * **Recommended Migration Pattern:** **AWS SCT + Snowball Edge + Amazon S3 + AWS DMS (CDC)**
  * **Architectural Reasoning:** Use **SCT Data Extraction Agents** to dump the 60 TB schema/data onto a **Snowball Edge**, ship to S3, load into Aurora via DMS, and use **AWS DMS Change Data Capture (CDC)** over the network from the exact Oracle SCN to catch up and cut over with near-zero downtime.

### 4.4 Application Migration & Modernization Service Selection

* **Unknown server dependencies & network conversations across 1,000+ VMs and physical servers**
  * **Target AWS Solution:** **AWS Application Discovery Service (Agent-based)** + **S3 / Athena / QuickSight**
  * **Why It Wins on SAP-C02:** Only the **Agent-based** collector captures **process-level details and TCP/UDP network connection dependencies** across both physical and virtual servers (Agentless OVA only works on VMware and lacks process/network dependency mapping).
* **Lift-and-shift (Rehost) 500 physical/virtual Linux & Windows servers with minimal downtime**
  * **Target AWS Solution:** **AWS Application Migration Service (MGN)**
  * **Why It Wins on SAP-C02:** Continuous block-level replication into a low-cost **Staging Area Subnet**; non-disruptive test launches and automated post-launch modernization (SSM agent, license conversion).
* **Migrate 300 VMware VMs to AWS in 2 weeks WITHOUT changing IP addresses or re-architecting**
  * **Target AWS Solution:** **VMware Cloud on AWS** + **VMware HCX (Layer 2 Network Extension & vMotion)**
  * **Why It Wins on SAP-C02:** Relocate (Hypervisor lift): stretches on-premises Layer 2 VLANs into AWS and performs live **vMotion** with zero IP changes or OS modifications.
* **Containerize legacy Java (Tomcat/JBoss) on Linux or .NET (IIS) on Windows VMs without source code changes**
  * **Target AWS Solution:** **AWS App2Container (A2C)**
  * **Why It Wins on SAP-C02:** Inspects running VM processes, extracts application artifacts and dependencies into a Docker image, pushes to **Amazon ECR**, and generates **ECS / EKS / App Runner** deployments.
* **Decompose a monolithic application into microservices incrementally using the Strangler Fig pattern**
  * **Target AWS Solution:** **AWS Migration Hub Refactor Spaces**
  * **Why It Wins on SAP-C02:** Provisions a multi-account **Transit Gateway + API Gateway + VPC Link** fabric that routes `/*` to the monolith and incrementally shifts specific URL paths (`/orders`, `/cart`) to new Lambda/ECS microservices seamlessly.
* **Modernize an IBM z/OS Mainframe (COBOL / CICS / VSAM / DB2) into cloud-native Java microservices**
  * **Target AWS Solution:** **AWS Mainframe Modernization – Automated Refactor (AWS Blu Age)**
  * **Why It Wins on SAP-C02:** Automatically converts legacy COBOL/PL1/RPG into modern **Java Spring Boot** applications and migrates VSAM/DB2 to **Aurora PostgreSQL** (choose **Micro Focus Replatform** instead if the requirement says *"preserve existing COBOL source code"*).
* **Migrate SQL Server to Aurora PostgreSQL WITHOUT rewriting thousands of lines of C# T-SQL & Stored Procedures**
  * **Target AWS Solution:** **Babelfish for Aurora PostgreSQL**
  * **Why It Wins on SAP-C02:** Enables Aurora PostgreSQL to natively understand the **Microsoft SQL Server wire protocol (TDS)** and **T-SQL** dialect alongside native PostgreSQL connections.

---

## 5. Data Analytics, Streaming & Serverless Integration Matrix

### 5.1 Real-Time Streaming & Ingestion Comparison

* **Amazon Kinesis Data Streams (KDS)**
  * **Scaling & Retention:** Provisioned Shards (`1 MB/s` in, `2 MB/s` out) or **On-Demand**; retention **24 hours to 365 days**
  * **Transformation & Delivery:** Custom consumers (Lambda, KCL, Flink, Firehose); **Enhanced Fan-Out** (`2 MB/s` dedicated per consumer via HTTP/2)
  * **SAP-C02 Selection Trigger:** Required when **multiple independent consumers** must read the same stream concurrently, when **message replay** (up to 1 year) is needed, or when sub-100ms processing is required.
* **Amazon Data Firehose**
  * **Scaling & Retention:** Serverless auto-scaling; **No retention / replay** (buffers `1–128 MB` or `0–900 seconds`)
  * **Transformation & Delivery:** Built-in Lambda transformation, **Parquet/ORC format conversion**, **Dynamic Partitioning** (by JSON keys)
  * **SAP-C02 Selection Trigger:** **Zero-administration ingestion** directly into **S3, Redshift, OpenSearch, or Splunk** with automatic columnar Parquet conversion and S3 prefix partitioning (`s3://bucket/year=.../customer_id=...`).
* **Amazon Managed Service for Apache Flink**
  * **Scaling & Retention:** Serverless auto-scaling KPUs
  * **Transformation & Delivery:** Stateful stream processing (SQL, Java, Python, Scala), **sliding/tumbling windows**, stream-to-stream joins
  * **SAP-C02 Selection Trigger:** Real-time **stateful anomaly detection**, 5-minute rolling averages, or joining clickstream events with reference data in flight between two Kinesis/MSK streams.
* **Amazon EventBridge Pipes**
  * **Scaling & Retention:** Serverless point-to-point
  * **Transformation & Delivery:** Native **Filtering**, **Enrichment** (via Lambda/API Gateway/Step Functions), and **Target Delivery**
  * **SAP-C02 Selection Trigger:** Connecting a single stream/queue source (DynamoDB Streams, Kinesis, SQS, MSK) to a target while **preserving strict message ordering** and filtering out 80% of unneeded events before invoking compute to save costs.

### 5.2 Serverless Orchestration & Messaging Comparison

* **SQS Standard Queue**
  * **Ordering & Delivery Guarantees:** **Best-effort ordering** | **At-least-once** delivery
  * **Throughput & Duration Limits:** **Nearly unlimited TPS**; max 14-day retention; 256 KB max payload (up to 2 GB via Extended Client Library + S3)
  * **Primary SAP-C02 Pattern:** Decoupling high-scale asynchronous workers; configure a **Dead-Letter Queue (DLQ)** with `maxReceiveCount` to isolate poison-pill messages.
* **SQS FIFO Queue**
  * **Ordering & Delivery Guarantees:** **Strict FIFO ordering per `MessageGroupId`** | **Exactly-once** processing (5-min deduplication)
  * **Throughput & Duration Limits:** 300 TPS (3,000 with batching); **High-Throughput Mode** scales to **70,000+ TPS** across multiple `MessageGroupId` values
  * **Primary SAP-C02 Pattern:** Financial transactions, order state updates, or sequential customer events. **Exam Trap**: Using a single `MessageGroupId` for all messages creates a head-of-line bottleneck; always shard `MessageGroupId` by `CustomerID` or `OrderID`!
* **Step Functions Standard Workflow**
  * **Ordering & Delivery Guarantees:** **Exactly-once** workflow execution
  * **Throughput & Duration Limits:** Max duration: **1 Year**; 2,000 executions/sec; **billed per state transition**
  * **Primary SAP-C02 Pattern:** Long-running orchestration, sagas, human approval workflows using **Task Tokens (`.waitForTaskToken`)**, and **Distributed Map** over millions of S3 objects.
* **Step Functions Express Workflow**
  * **Ordering & Delivery Guarantees:** **At-least-once** (Async) or **At-most-once** (Sync)
  * **Throughput & Duration Limits:** Max duration: **5 Minutes**; **100,000+ executions/sec**; **billed by execution count & memory-duration**
  * **Primary SAP-C02 Pattern:** High-volume, short-duration IoT data ingestion, synchronous microservice API orchestration behind API Gateway, or streaming data enrichment.

---

## 6. FinOps & Cost Optimization Formulas

### 6.1 Compute Purchasing Models Comparison

* **Compute Savings Plans**
  * **Max Discount:** **Up to 66%**
  * **Commitment Type:** `$/hour` for 1 or 3 years
  * **Flexibility Across Attributes:** **Maximum Flexibility**: Automatically applies across **EC2 (any instance family, size, OS, tenancy), AWS Fargate, and AWS Lambda** across **any AWS Region**!
* **EC2 Instance Savings Plans**
  * **Max Discount:** **Up to 72%**
  * **Commitment Type:** `$/hour` for 1 or 3 years
  * **Flexibility Across Attributes:** Locked to a **specific Instance Family in a specific Region** (e.g., `m6i` in `us-east-1`), but flexible across **Instance Size, OS (Windows/Linux), AZ, and Tenancy**.
* **Standard Reserved Instances (RIs)**
  * **Max Discount:** **Up to 72%**
  * **Commitment Type:** Specific instance attributes for 1 or 3 years
  * **Flexibility Across Attributes:** Regional RI gives size flexibility within a Linux family (no OS changes); **Zonal RI** provides an explicit **Capacity Reservation** in that AZ (Savings Plans do NOT reserve capacity by default; pair Savings Plans with On-Demand Capacity Reservations if needed).
* **Convertible Reserved Instances**
  * **Max Discount:** **Up to 66%**
  * **Commitment Type:** 1 or 3 years
  * **Flexibility Across Attributes:** Can exchange for a different instance family, OS, or tenancy of equal or greater value, but does NOT cover Fargate or Lambda (Compute Savings Plans are almost always preferred on `SAP-C02`).
* **EC2 Spot Instances**
  * **Max Discount:** **Up to 90%**
  * **Commitment Type:** No commitment (2-min interruption notice)
  * **Flexibility Across Attributes:** Ideal for stateless, fault-tolerant batch, CI/CD, EMR task nodes, and containerized workloads using **Capacity-Optimized** allocation strategy and **EC2 Instance Rebalance Recommendations**.

### 6.2 High-Yield Networking Cost Formulas

1. **S3 & DynamoDB Gateway VPC Endpoint Rule**:
   * **NAT Gateway Data Processing + Hourly Charge**: $\$0.045/\text{GB}$ processed + $\$0.045/\text{hour}$ per AZ.
   * **Gateway VPC Endpoint (S3 & DynamoDB only)**: **$\$0.00/\text{GB}$** and **$\$0.00/\text{hour}$**!
   * *Exam Rule*: If private EC2/ECS/Lambda workloads in a VPC transfer terabytes of data to/from **S3 or DynamoDB in the same Region** and incur huge NAT Gateway bills, **ALWAYS deploy a Gateway VPC Endpoint** and update the private subnet route tables!
2. **Interface VPC Endpoint (PrivateLink) vs. NAT Gateway Break-Even**:
   * **Interface VPC Endpoint**: $\$0.01/\text{hour}$ per AZ ($\approx \$7.30/\text{month/AZ}$) + $\$0.01/\text{GB}$ data processed.
   * For high-volume private traffic (`>500 GB/month`) to AWS services like ECR, Kinesis, KMS, SSM, Secrets Manager, or CloudWatch Logs, **Interface VPC Endpoints save ~$0.035/GB** (77% cheaper data processing) compared to hair-pinning through a NAT Gateway!
   * In a Multi-Account environment with 50+ VPCs, **centralize Interface VPC Endpoints in a Shared Network Hub VPC** connected via Transit Gateway + Route 53 Private Hosted Zones to avoid paying $\$7.30/\text{month/AZ}$ multiplied by 50 VPCs!

---

## 7. Critical AWS Quotas & Hard Architectural Limits (`SAP-C02` Must-Knows)

* **IAM Role Chaining Session Duration**
  * **Hard Limit / Quota:** **1 Hour maximum** (when Role A assumes Role B via CLI/SDK)
  * **Architectural Workaround When Exceeded:** Do not chain roles for jobs $>1\text{ hr}$; assume the target role directly from the compute/IdP (supports up to **12 hours**).
* **S3 Single PUT Object Size**
  * **Hard Limit / Quota:** **5 GB maximum** per single `PUT` request (5 TB max total object size)
  * **Architectural Workaround When Exceeded:** Use **S3 Multipart Upload** (recommended for $>100\text{ MB}$, mandatory for $>5\text{ GB}$, up to 10,000 parts).
* **S3 Prefix Request Rate**
  * **Hard Limit / Quota:** **3,500 `PUT/COPY/POST/DELETE`** and **5,500 `GET/HEAD`** req/sec per prefix
  * **Architectural Workaround When Exceeded:** Partition across multiple S3 prefixes (e.g., hash or date/customer prefixes) or place **CloudFront** in front of S3 reads.
* **SQS Message Payload Size**
  * **Hard Limit / Quota:** **256 KB maximum**
  * **Architectural Workaround When Exceeded:** Use **Amazon SQS Extended Client Library for Java/Python** to store payloads up to **2 GB** in Amazon S3 and pass the S3 pointer through SQS.
* **SNS / SQS Message Retention**
  * **Hard Limit / Quota:** **14 Days maximum** (SQS: 60s to 14 days)
  * **Architectural Workaround When Exceeded:** If longer retention/replay is needed, use **EventBridge Archive & Replay** (indefinite) or **Kinesis Data Streams** (up to 365 days).
* **AWS Lambda Execution Timeout**
  * **Hard Limit / Quota:** **15 Minutes (900 seconds) maximum**
  * **Architectural Workaround When Exceeded:** Migrate longer jobs to **AWS Fargate (ECS Batch)**, **AWS Batch**, or split into iterations orchestrated by **AWS Step Functions**.
* **API Gateway Integration Timeout**
  * **Hard Limit / Quota:** **29 Seconds hard limit** (REST/HTTP APIs; regional REST supports quota increase in specific cases, but exam treats 29s as the architectural boundary)
  * **Architectural Workaround When Exceeded:** Return an immediate `HTTP 202 Accepted` with a job ID via **SQS / Step Functions**, and poll status or push results via **WebSocket API / AppSync**.
* **DynamoDB Single Partition Throughput**
  * **Hard Limit / Quota:** **1,000 WCU/sec** or **3,000 RCU/sec** per physical partition key value
  * **Architectural Workaround When Exceeded:** Avoid low-cardinality partition keys; apply **Write Sharding** (append random suffix `1..N`) or use **DAX** to absorb hot read spikes.
* **DynamoDB Item Size**
  * **Hard Limit / Quota:** **400 KB maximum** (including attribute names)
  * **Architectural Workaround When Exceeded:** Compress attributes with gzip or store large blobs in **Amazon S3** and store only the S3 object URI in DynamoDB.
* **VPC Peering Connections per VPC**
  * **Hard Limit / Quota:** **125 maximum** (active peering connections per VPC; degrades route table management well before 125)
  * **Architectural Workaround When Exceeded:** Migrate full-mesh VPC Peering (`N(N-1)/2` connections) to **AWS Transit Gateway** or **AWS Cloud WAN** when exceeding 10–15 VPCs.

---

## 8. Top 50 SAP-C02 "If You See X, Think Y" Exam Triggers & Classic Traps

1. **"Stateful firewall (AWS Network Firewall or Gateway Load Balancer) in a central Inspection VPC drops intermittent return traffic when source and destination workloads are in different Availability Zones"** > Enable **Appliance Mode (`ApplianceModeSupport: enable`)** on the Inspection VPC's Transit Gateway attachment.
2. **"Two VPCs or on-premises networks have overlapping IPv4 CIDR blocks (`10.0.0.0/16`) and must communicate without re-IPing"** > Use **AWS PrivateLink** (Interface VPC Endpoint + NLB for TCP) or **Amazon VPC Lattice** (Service Network for HTTP/HTTPS/gRPC across accounts/VPCs).
3. **"Ensure AWS Direct Connect connection in `us-east-1` is ALWAYS preferred over a backup Direct Connect connection in `us-west-2` for traffic returning from AWS to on-premises"** > Tag prefixes advertised from on-premises over the primary VIF with BGP community **`7224:7300` (High Local Preference)** and the backup VIF with **`7224:7100` (Low Local Preference)**.
4. **"Detect Direct Connect link failure and fail over BGP to a backup DX or VPN in under 1 second instead of waiting 90 seconds"** > Enable **Bidirectional Forwarding Detection (BFD)** on the Direct Connect VIF (300ms interval $\times$ 3 multiplier = ~900ms detection).
5. **"Route traffic directly between two on-premises data centers in London and Tokyo using the AWS global network without traversing an AWS Region/VPC"** > Enable **AWS Direct Connect SiteLink** on the Private or Transit VIFs connected to the same Direct Connect Gateway.
6. **"Encrypt traffic over an existing 1 Gbps AWS Direct Connect connection to a Transit Gateway"** > Create a **Transit VIF**, attach it to a **Direct Connect Gateway**, and create an **AWS Site-to-Site VPN attachment** on the Transit Gateway running over the Transit VIF (or use a Public VIF + Site-to-Site VPN). *(Note: MACsec requires 10 Gbps or 100 Gbps Dedicated connections).*
7. **"50 spoke VPCs across an AWS Organization need to resolve on-premises `corp.internal` DNS records with minimal cost and operational overhead"** > Create **1 Route 53 Resolver Outbound Endpoint** and Conditional Forwarding Rules in a central Network Hub VPC, and **share the Forwarding Rules across the Organization using AWS RAM** (do NOT deploy Outbound Endpoints in all 50 VPCs!).
8. **"Centralize Interface VPC Endpoints (e.g., `sqs.us-east-1.amazonaws.com`) in a Network Hub VPC so 50 Spoke VPCs over Transit Gateway use them without changing application SDK URLs"** > Disable automatic Private DNS on the central Interface Endpoint, create a manual **Route 53 Private Hosted Zone** named `sqs.us-east-1.amazonaws.com` with an `A` Alias record pointing to the central Endpoint, and **associate the PHZ with all Spoke VPCs** (or resolve via central Inbound Endpoint).
9. **"Prevent any user—including Account Administrators—from disabling GuardDuty, CloudTrail, or AWS Config across all member accounts, except for an approved CI/CD pipeline role"** > Attach a **Service Control Policy (SCP)** to the Organization Root/OUs with an explicit `Deny` on `guardduty:DeleteDetector`, `cloudtrail:StopLogging`, etc., combined with a `StringNotLike` condition on **`aws:PrincipalArn`** for the pipeline role.
10. **"Restrict all member accounts to operate only in `us-east-1` and `eu-west-1` via SCP, but IAM and Route 53 suddenly break"** > Global services (`iam:*`, `sts:*`, `route53:*`, `cloudfront:*`, `waf:*`, `support:*`, `budgets:*`) execute in `us-east-1` or globally; a Region-restriction SCP using `StringNotEquals` on `aws:RequestedRegion` **MUST include a `NotAction` exemption list** for global AWS services.
11. **"Allow developers to create IAM Roles for Lambda functions, but prevent them from creating admin roles (Privilege Escalation)"** > Create an **IAM Permissions Boundary** limiting maximum permissions, and attach an IAM policy to the developers that allows `iam:CreateRole` and `iam:AttachRolePolicy` **ONLY if the `iam:PermissionsBoundary` condition equals that boundary ARN**.
12. **"Eliminate long-lived IAM User access keys stored on 200 on-premises Linux servers that upload files to S3; servers already have certificates from a corporate PKI"** > Use **IAM Roles Anywhere**, register the corporate CA as a **Trust Anchor**, create a **Profile**, and use `aws_signing_helper` on the servers to obtain temporary STS credentials.
13. **"Cross-account S3 access fails with `AccessDenied` when Account A's role copies SSE-KMS encrypted objects from Account B's S3 bucket"** > AWS Managed Keys (`aws/s3`) **cannot** be shared cross-account! Account B must encrypt objects with a **Customer Managed Key (CMK)**, grant Account A's role `kms:Decrypt` in the **KMS Key Policy**, AND Account A's IAM Role must have an **Identity Policy** allowing `kms:Decrypt` on that CMK ARN.
14. **"Encrypt sensitive credit card fields (`CVV`, `PAN`) at the CDN edge so that even web application servers in the VPC cannot read them—only a specific microservice with a private key can decrypt them"** > Configure **Amazon CloudFront Field-Level Encryption** using an uploaded RSA public key.
15. **"Inspect and rewrite URL paths or normalize HTTP headers on 5 million requests per second at the lowest possible cost; no network calls or body inspection needed"** > Use **CloudFront Functions** (NOT Lambda@Edge).
16. **"Dynamically route cache-miss requests to different S3 buckets based on a user attribute stored in DynamoDB, or inspect the HTTP POST body"** > Use **Lambda@Edge** on the **Origin Request** trigger (CloudFront Functions cannot make network calls or read the request body).
17. **"Global gaming or financial UDP/TCP application requires 2 static IP addresses for partner firewall whitelisting, client IP preservation, and sub-30-second regional failover"** > Use **AWS Global Accelerator** pointing to Regional ALBs/NLBs/EC2 instances.
18. **"Ensure that an S3 bucket backing a CloudFront distribution cannot be accessed directly via S3 URLs, while supporting SSE-KMS encryption and PUT/POST requests"** > Use **CloudFront Origin Access Control (OAC)** (replaces legacy OAI, which did not support SSE-KMS or dynamic PUT/POST requests) and restrict the S3 Bucket Policy to `cloudfront.amazonaws.com` with a `aws:SourceArn` condition.
19. **"Relational database requires Multi-Region Disaster Recovery with RPO < 1 second and RTO < 1 minute, plus low-latency local reads in secondary regions"** > Use **Amazon Aurora Global Database** (supports **Write Forwarding** for occasional secondary-region writes and **Managed Planned Failover** for RPO = 0).
20. **"Active-Active multi-region application needs a highly reliable mechanism to shift DNS traffic away from an impaired region even if the Route 53 control plane itself is unreachable"** > Use **Amazon Route 53 Application Recovery Controller (ARC) Routing Controls** (uses a 5-node global quorum data plane cluster).
21. **"Strict regulatory compliance mandate requires that backups cannot be deleted by anyone—including the AWS Account Root User or an attacker with full admin access—for 7 years"** > Use **AWS Backup Vault Lock in Compliance Mode** (or **S3 Object Lock in Compliance Mode** with Versioning enabled) after the cooling-off period expires.
22. **"Serverless application with 3,000 concurrent AWS Lambda functions causes `MaxConnectionsError` on an Amazon RDS database during traffic spikes"** > Place **Amazon RDS Proxy** between the Lambda functions and the RDS instance/cluster inside the VPC.
23. **"Java or .NET AWS Lambda function experiences 6-second cold-start latency during unpredictable traffic bursts; the company wants to eliminate cold starts without paying for idle Provisioned Concurrency"** > Enable **AWS Lambda SnapStart** (caches an encrypted Firecracker microVM snapshot after initialization; for Node.js/Python or strict sub-10ms SLA without SnapStart, use **Provisioned Concurrency**).
24. **"Ingest 50,000 JSON events/sec from IoT devices, convert them to Apache Parquet, partition them in S3 by `device_type` and `date`, and query them via SQL with the LEAST operational overhead"** > Use **Amazon Data Firehose** with **Dynamic Partitioning** and built-in **Parquet format conversion** (via a Glue Data Catalog table schema), delivering to S3, queried by **Amazon Athena**.
25. **"Enforce fine-grained Column-Level, Row-Level, and Cell-Level security across a petabyte-scale S3 Data Lake queried by Athena, Redshift, and EMR across 20 AWS accounts without duplicating S3 buckets"** > Use **AWS Lake Formation** with **LF-Tags (Tag-Based Access Control)**, Data Cells Filters, and cross-account sharing via **AWS RAM**.
26. **"Share live analytics tables from a producer Amazon Redshift cluster to a consumer department's Redshift Serverless workgroup in another AWS account without copying or moving data"** > Use **Amazon Redshift Data Sharing** (authorized via AWS Lake Formation / AWS RAM).
27. **"Orchestrate a distributed microservice workflow that pauses and waits up to 14 days for a human manager to click an approval link in an email"** > Use **AWS Step Functions Standard Workflow** with a **Task Token callback pattern (`.waitForTaskToken`)**.
28. **"SQS FIFO queue processing order events is throttling at 300 messages/sec even with High-Throughput Mode enabled"** > All messages are being sent with the **same `MessageGroupId`**! Change the producer to use a high-cardinality `MessageGroupId` (such as `CustomerID` or `AccountID`) so SQS FIFO can parallelize across partitions while preserving per-customer order.
29. **"Automatically rotate database credentials every 30 days with zero application downtime during rotation"** > Use **AWS Secrets Manager** configured with the **Alternating Users (Dual-User) rotation strategy** (`AWSCURRENT` and `AWSPREVIOUS`).
30. **"Eliminate SSH port 22, RDP port 3389, and public Bastion Hosts across all EC2 instances while logging every command executed by administrators to S3 and CloudWatch Logs"** > Use **AWS Systems Manager Session Manager** (requires SSM Agent + IAM Instance Profile with `AmazonSSMManagedInstanceCore` + Interface VPC Endpoints for `ssm`, `ssmmessages`, `ec2messages` if in isolated subnets).
31. **"Automatically deploy a baseline IAM role and GuardDuty configuration to every new AWS account created in an Organizational Unit"** > Use **AWS CloudFormation StackSets** with **`SERVICE_MANAGED` permissions** and **Automatic Deployment** enabled for the target OUs.
32. **"Stream all Amazon CloudWatch metrics from 100 AWS accounts to a third-party observability platform (Datadog / Splunk / New Relic) in near-real time with minimal API polling cost"** > Use **Amazon CloudWatch Metric Streams** delivering to **Amazon Data Firehose** (far faster and cheaper than third-party API polling `GetMetricData`).
33. **"Heavy BI reporting queries running on an Aurora cluster slow down customer-facing OLTP read queries because both hit the default Reader Endpoint"** > Create an **Aurora Custom Endpoint** comprising 2 dedicated large Aurora Replicas for the BI team, and exclude those instances from the OLTP Custom Reader Endpoint.
34. **"DynamoDB table with plenty of provisioned WCU capacity throws `ProvisionedThroughputExceededException` whenever a flash sale occurs for a single popular product ID"** > **Hot Partition Key** (a single partition key exceeds the `1,000 WCU/sec` or `3,000 RCU/sec` physical partition limit). Fix by **Write Sharding** (appending a random number `1..10` or hash suffix to the Partition Key) or placing **DAX / ElastiCache** in front of reads.
35. **"HPC cluster running tightly coupled MPI simulations across 64 EC2 instances suffers from high network latency and jitter"** > Launch the instances inside a single-AZ **Cluster Placement Group** and attach an **Elastic Fabric Adapter (EFA)** to each instance.
36. **"Critical distributed NoSQL database (Cassandra/HBase/Kafka) across 6 instances needs a guarantee that no two instances ever share the same underlying physical hardware rack"** > Use a **Spread Placement Group** (for $\le 7$ instances per AZ) or a **Partition Placement Group** (for large clusters with $>7$ instances per AZ).
37. **"Newly restored Amazon EBS volume from a snapshot exhibits severe I/O latency for the first several hours of production traffic (Lazy Loading penalty)"** > Enable **EBS Fast Snapshot Restore (FSR)** on the snapshot before restoring the volume.
38. **"Clustered Linux application requires a single high-performance block storage volume simultaneously attached to 4 EC2 instances in the same AZ with sub-millisecond latency"** > Use **Amazon EBS `io2 Block Express` with Multi-Attach enabled** and a cluster-aware file system (e.g., GFS2 or OCFS2; standard ext4/XFS will corrupt data!).
39. **"Unpredictable access patterns on a 500 TB S3 bucket where objects are heavily accessed for a few weeks and then rarely touched, with a strict requirement for zero retrieval fees"** > Use **Amazon S3 Intelligent-Tiering** (automatically moves objects $>128\text{ KB}$ between Frequent, Infrequent, and Archive Instant tiers with **zero retrieval fees**).
40. **"Company uses EC2 in `us-east-1`, plans to migrate 50% of workloads to AWS Fargate and AWS Lambda in `eu-west-1` next quarter, and wants maximum commitment discount flexibility"** > Purchase a **Compute Savings Plan** (covers EC2, Fargate, and Lambda across all Regions, families, and OS types).
41. **"Discover 800 on-premises physical Linux servers and VMware VMs, including exact inbound/outbound TCP network dependencies to group servers into migration waves"** > Install the **AWS Application Discovery Agent** on all servers, export data to S3, and query dependency graphs with **Amazon Athena** (Agentless Discovery Connector cannot discover physical servers or process/network dependencies).
42. **"Migrate a 20 TB Oracle database to Amazon Aurora PostgreSQL with minimal downtime"** > Heterogeneous migration: use **AWS Schema Conversion Tool (SCT)** to convert the schema/PL-SQL first, then use **AWS DMS** with **Full Load + CDC** to replicate data and cut over.
43. **"Replace on-premises SFTP servers with a managed AWS service backed by S3 while keeping existing partner usernames/passwords stored in an internal Active Directory or custom SQL database"** > Use **AWS Transfer Family for SFTP** integrated with a **Custom Identity Provider** via **Amazon API Gateway + AWS Lambda** (or AWS Directory Service).
44. **"On-premises legacy application writes files over NFS to an S3 File Gateway, and an AWS Lambda function is triggered by `s3:ObjectCreated:*` to upload processed results back to the same S3 bucket, but the on-premises server cannot see the new files"** > Configure an EventBridge/Lambda rule to invoke the **`RefreshCache` API** on the **AWS Storage Gateway (S3 File Gateway)** whenever objects are added directly to S3 outside the gateway!
45. **"Incrementally strangle a legacy monolithic Java application in Account A by routing `/api/v2/payments` to a new serverless microservice in Account B without changing the public domain or client code"** > Use **AWS Migration Hub Refactor Spaces** (automatically sets up the cross-account Transit Gateway, API Gateway, and VPC Link path routing).
46. **"Protect an Application Load Balancer, API Gateway, and CloudFront distributions across 60 AWS accounts against DDoS attacks with 24/7 access to the AWS Shield Response Team (SRT) and automatic cost protection against DDoS scaling spikes"** > Subscribe to **AWS Shield Advanced** and use **AWS Firewall Manager** to automatically enforce Shield Advanced and WAF policies across all accounts in the Organization.
47. **"Ensure that objects replicated across AWS Regions via S3 Cross-Region Replication (CRR) are guaranteed to arrive within 15 minutes with a contractual SLA and CloudWatch replication metrics"** > Enable **S3 Replication Time Control (S3 RTC)** on the replication rule (provides a 99.9% SLA that 99.99% of objects replicate within 15 minutes).
48. **"Client-side or KMS-encrypted records in DynamoDB Global Tables or S3 CRR need to be decrypted in a secondary Region using the exact same Key ID and key material without re-encrypting ciphertext during cross-region replication"** > Use **AWS KMS Multi-Region Keys** (primary key in Region A replicated to a Multi-Region replica key in Region B sharing the same Key ID and cryptographic material).
49. **"Auto Scaling group instances are terminated during scale-in before finishing in-flight batch jobs or before uploading final application logs to S3"** > Configure an **EC2 Auto Scaling Termination Lifecycle Hook** that places terminating instances into the `Terminating:Wait` state and triggers an **EventBridge rule > SSM Run Command / draining script**, calling `CompleteLifecycleAction` once finished.
50. **"Migrate an on-premises Microsoft SQL Server Enterprise fleet using Bring Your Own License (BYOL) with per-physical-core licensing while preventing VMs from launching on unlicensed hardware"** > Allocate **Amazon EC2 Dedicated Hosts**, track core/socket rules in **AWS License Manager**, and share the host resource group across the AWS Organization via **AWS RAM**.
