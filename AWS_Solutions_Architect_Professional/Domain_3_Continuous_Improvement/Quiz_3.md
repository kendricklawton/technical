# Domain 3: Continuous Improvement — Quiz 3 (Reliability & SPoF Remediation)

**Exam:** AWS Certified Solutions Architect - Professional (SAP-C02)
**Format:** Complex Multi-Tier Scenario-Based Professional Practice Questions

---

### Question 1: Transit Gateway Appliance Mode for Symmetric State Inspection

A financial enterprise deploys stateful firewall appliances (e.g., AWS Network Firewall or third-party NGFWs) behind a Gateway Load Balancer inside a centralized Inspection VPC attached to AWS Transit Gateway. During heavy cross-VPC traffic testing, client TCP connections intermittently stall or drop after handshake.

- A. The Network Firewall rules are improperly configured.
- B. Transit Gateway routes traffic asymmetrically across different Availability Zones for request and response packets. Solution: Enable Appliance Mode (`ApplianceModeSupport: enable`) on the Inspection VPC attachment.
- C. Transit Gateway MTU size must be lowered to 1420.
- D. Gateway Load Balancer cross-zone load balancing must be disabled.

---

### Question 2: Transit Gateway Inter-Region Peering MTU & Jumbo Frames

An enterprise connects two AWS Transit Gateways across us-east-1 and eu-west-1 using Transit Gateway Peering. Large database backup transfers between EC2 instances in the peered VPCs experience frequent packet fragmentation and reduced throughput. What is the maximum transmission unit (MTU) supported over Transit Gateway Peering attachments, and how should this be addressed?

- A. Transit Gateway peering supports 9001 MTU jumbo frames natively; verify that EC2 instances have jumbo frames enabled.
- B. Transit Gateway peering attachments support a maximum MTU of 8500 bytes. The EC2 instance network interfaces should be configured with an MTU of 8500 or path MTU discovery (PMTUD) must be allowed by ensuring ICMP Type 3 Code 4 ('Fragmentation Needed') packets are not blocked by Security Groups or NACLs.
- C. Transit Gateway peering is limited to 1500 MTU; instances must be clamped to 1500 bytes.
- D. Deploy an IPsec VPN tunnel over Transit Gateway peering to enable 9001 MTU.

---

### Question 3: Direct Connect Link Aggregation Group (LAG) Active/Active Redundancy

A financial enterprise requires 40 Gbps of dedicated bandwidth between their primary data center and AWS with zero single points of failure. They plan to use AWS Direct Connect Link Aggregation Groups (LAG). Which configuration provides the required resiliency?

- A. Provision a single 40 Gbps connection to one Direct Connect location.
- B. Provision four 10 Gbps connections terminating on a single Direct Connect router.
- C. Provision two separate Direct Connect LAGs across two different Direct Connect Locations (points of presence), each LAG composed of two 10 Gbps connections terminating on diverse AWS Direct Connect endpoint devices.
- D. Connect Direct Connect directly to an Internet Gateway.

---

### Question 4: AWS Network Firewall vs Gateway Load Balancer Architecture

An organization wants to inspect all east-west (VPC-to-VPC) and north-south (internet ingress/egress) traffic with deep packet inspection and intrusion prevention system (IPS). Which architectural solution minimizes infrastructure management while providing centralized inspection?

- A. Deploy proxy servers in every private subnet.
- B. Deploy an AWS Network Firewall endpoint inside a dedicated Inspection VPC attached to AWS Transit Gateway with Transit Gateway Appliance Mode enabled.
- C. Install Snort directly on every EC2 instance.
- D. Attach an Internet Gateway to every private subnet.

---

### Question 5: Amazon Route 53 Resolver DNS Firewall and Threat Prevention

A security audit mandates that EC2 instances across all VPCs in an AWS Organization must be prevented from resolving domain names associated with known command-and-control (C2) botnets, cryptocurrency mining pools, and malware distribution sites.

- A. Edit `/etc/hosts` on all EC2 instances via SSM State Manager.
- B. Associate an Amazon Route 53 Resolver DNS Firewall rule group containing AWS Managed Domain Lists (e.g., MalwareDomains, BotnetDomains) with all VPCs across the AWS Organization using AWS Firewall Manager.
- C. Deploy custom BIND DNS servers in every VPC.
- D. Block outbound port 53 in all Security Groups.

---

### Question 6: AWS Lake Formation Cross-Account Data Mesh Governance

A data engineering team in Account A maintains a central data lake on S3. Analysts in Account B, Account C, and Account D require access to specific tables in the AWS Glue Data Catalog. The security team mandates column-level masking (e.g. masking PII columns) and cell-level filtering without creating copies of the data.

- A. Create separate S3 buckets per consumer account and copy sanitized data via Lambda.
- B. Use AWS Lake Formation cross-account grants with LF-Tags, granting fine-grained column-level and row-level permissions directly to consumer accounts and IAM roles.
- C. Grant full S3 bucket access and rely on application-level filtering.
- D. Export data to CSV files and email them.

---

## Answer Key & Detailed Explanations

### Question 1: Transit Gateway Appliance Mode for Symmetric State Inspection
**Answer:** B
**Explanation:** By default, Transit Gateway does not guarantee AZ symmetry for VPC-to-VPC routing. Stateful firewalls track TCP state; if the forward packet enters AZ-a and the return packet traverses AZ-b, the firewall in AZ-b drops the packet as an invalid state. Enabling Transit Gateway Appliance Mode guarantees symmetric AZ routing.

---



### Question 2: Transit Gateway Inter-Region Peering MTU & Jumbo Frames
**Answer:** B
**Explanation:** Transit Gateway VPC attachments support 8500 MTU for intra-region traffic and Transit Gateway Peering supports 8500 MTU for inter-region traffic. If PMTUD is broken (e.g. ICMP blocked), packets larger than 8500 are dropped silently (black hole).

---

### Question 3: Direct Connect Link Aggregation Group (LAG) Active/Active Redundancy
**Answer:** C
**Explanation:** A single LAG only protects against port or cable failure on a single device at a single location. For high resiliency and data center diversity, multiple LAGs must terminate across distinct AWS Direct Connect Locations and distinct chassis.

---

### Question 4: AWS Network Firewall vs Gateway Load Balancer Architecture
**Answer:** B
**Explanation:** AWS Network Firewall is a managed stateful inspection service powered by Suricata rules. Deploying it in a centralized Inspection VPC with Transit Gateway Appliance Mode ensures all cross-VPC and internet traffic traverses the firewall symmetrically across AZs without managing EC2 firewall fleets.

---

### Question 5: Amazon Route 53 Resolver DNS Firewall and Threat Prevention
**Answer:** B
**Explanation:** Route 53 Resolver DNS Firewall allows filtering outbound DNS queries for domains matching allowlists or denylists. AWS Firewall Manager centrally manages and enforces these associations across the entire AWS Organization.

---

### Question 6: AWS Lake Formation Cross-Account Data Mesh Governance
**Answer:** B
**Explanation:** AWS Lake Formation supports fine-grained access control (table, column, row, and cell level) using LF-Tags. It seamlessly shares Glue catalog resources across AWS accounts without data duplication.

---

