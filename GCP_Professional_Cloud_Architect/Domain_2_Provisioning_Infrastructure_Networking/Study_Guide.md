# Domain 2: Managing and Provisioning Infrastructure & Networking — Study Guide

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Weight:** ~15% of the certification curriculum

---

## 1. VPC Architecture & Multi-Project Topology
- **VPC Characteristics:** Google Cloud VPCs are global resources; subnets are regional. Instances in different zones or regions on the same VPC communicate privately over Google's global fiber backbone without internet transit or gateways.
- **Shared VPC:**
  - Allows an organization to connect resources from multiple projects to a common Virtual Private Cloud (VPC) network.
  - **Host Project:** Contains the shared VPC network and subnets, administered by the central Network Administrator.
  - **Service Projects:** Attached to the host project; project owners provision compute resources (GCE VMs, GKE clusters) within designated subnets without network admin privileges.
- **VPC Network Peering:**
  - Connects two VPC networks directly with RFC 1918 private IP connectivity.
  - Key constraint: **Non-transitive peering**. If VPC A peers with VPC B, and VPC B peers with VPC C, VPC A cannot communicate with VPC C through VPC B.
  - IP ranges across peered VPCs cannot overlap.

---

## 2. Hybrid Connectivity Solutions
- **Cloud VPN (HA VPN):**
  - High-availability IPsec VPN offering a $99.99\%$ service availability SLA.
  - Requires two VPN interfaces on the Google Cloud gateway connected via dynamic BGP routing (Cloud Router).
  - Bandwidth limit: 3 Gbps per tunnel (up to 4 tunnels supported).
- **Cloud Interconnect:**
  - **Dedicated Interconnect:** Direct physical fiber connection between customer network and Google colocation facility (10 Gbps or 100 Gbps circuits).
  - **Partner Interconnect:** Connects through a supported service provider when colocation is unavailable (50 Mbps to 50 Gbps).
  - Provides enterprise SLA ($99.9\%$ with 2 connections across 1 metro; $99.99\%$ with 4 connections across 2 metros and dual edge routers).
- **Private Access & Service Connect:**
  - **Private Google Access:** Enables VMs without external public IPs to reach Google APIs (Cloud Storage, BigQuery) over internal RFC 1918 routes.
  - **Private Service Connect (PSC):** Allows consumers to access managed services (GCP APIs, third-party SaaS, internal services) privately inside their own VPC using internal IP forwarding rules.

---

## 3. Load Balancing & Edge Networking
- **Global External HTTP(S) Load Balancer:**
  - Layer 7 proxy using Anycast global IP addresses. Traffic enters Google's edge Point of Presence (PoP) closest to the user and traverses Google's private backbone to the backend.
  - Supports multi-region backend services, cross-region automatic failover, SSL offloading, and HTTP/2 / HTTP/3.
  - Integrates natively with Cloud CDN for edge caching and Cloud Armor for DDoS / WAF protection.
- **Regional External HTTP(S) Load Balancer:**
  - Regional Layer 7 proxy for workloads requiring strict regional data compliance.
- **Network Load Balancers:**
  - **External TCP/UDP Network Load Balancer (Passthrough):** Layer 4 non-proxied load balancer preserving client source IP.
  - **Internal TCP/UDP Load Balancer:** Regional private load balancer for internal microservices with zero proxy latency.
- **Cloud NAT:**
  - Managed Network Address Translation allowing outbound internet egress for private VMs without assigning public external IPs.

---

## 4. Infrastructure as Code (IaC) & Automation
- **Terraform with Google Cloud Provider:** Declarative multi-resource provisioning. State management in Cloud Storage with Object Versioning and state locking.
- **Config Connector:** Kubernetes operator allowing GCP resources to be managed declaratively as native Kubernetes custom resources (CRDs).
- **Cloud Foundation Fabric / Blueprint:** Pre-architected enterprise landing zones following Google best practices.
