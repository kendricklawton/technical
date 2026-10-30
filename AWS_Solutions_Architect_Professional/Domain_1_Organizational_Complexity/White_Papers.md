# Domain 1: Design Solutions for Organizational Complexity — Essential White Papers & Documentation Guide

> **Blueprint Weight:** 26% of AWS Certified Solutions Architect - Professional (SAP-C02)  
> **Core Objective:** Architect secure, multi-account enterprise landing zones, complex hybrid network interconnects (Direct Connect, Transit Gateway, VPN), cross-account identity federation, and global network routing with centralized inspection.

---

## 1. Priority AWS White Papers & Architecture Guides

### 1.1 [Organizing Your AWS Environment Using Multiple Accounts](https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/welcome.html)
* **Canonical Scope:** Architectural blueprint for enterprise multi-account structures using AWS Organizations and AWS Control Tower.
* **Key Organizational Units (OUs):**
  * **Core / Foundation OUs:** Log Archive (immutable S3 logs), Security Tooling (GuardDuty master, Security Hub admin), Network (Transit Gateway, Direct Connect, Inspection VPC), Shared Services (CI/CD, AD, artifact repos).
  * **Workload OUs:** Segregated by environment (Dev, Staging, Prod) and compliance classification (PCI, HIPAA).
  * **Policy Framework:** Guardrails enforced via Service Control Policies (SCPs), Tag Policies, and Backup Policies.

### 1.2 [Building a Transit Gateway Network with Centralized Inspection (AWS Prescriptive Guidance)](https://docs.aws.amazon.com/prescriptive-guidance/latest/centralized-network-inspection/welcome.html)
* **Canonical Scope:** Enterprise hub-and-spoke network architecture combining AWS Transit Gateway, AWS Network Firewall, and Gateway Load Balancer (GWLB).
* **Key Architectural Rules:**
  * **Appliance Mode:** MUST be enabled on the Transit Gateway attachment in the Central Inspection VPC to guarantee symmetric bidirectional traffic routing through stateful firewalls across Availability Zones.
  * **East-West & North-South Inspection:** Routing all VPC-to-VPC and Internet-to-VPC traffic through dedicated inspection subnets before traversing TGW route tables.

### 1.3 [Hybrid Cloud DNS Options for Amazon VPC](https://docs.aws.amazon.com/whitepapers/latest/hybrid-cloud-dns-options-for-vpc/welcome.html)
* **Canonical Scope:** Resolving domain names bi-directionally between on-premises data centers and AWS VPCs.
* **Architecture Pattern:**
  * **Inbound Resolver Endpoints:** On-premises DNS forwarders query Route 53 Inbound Endpoints (static ENI IPs in VPC) to resolve private `.aws.internal` hosted zones.
  * **Outbound Resolver Endpoints & Forwarding Rules:** VPC workloads resolve on-premises corporate domains (`corp.internal`) via Route 53 Resolver forwarding rules routing queries across Direct Connect / VPN to on-premises DNS servers.

### 1.4 [AWS Direct Connect and Hybrid Connectivity Options](https://docs.aws.amazon.com/whitepapers/latest/aws-direct-connect-network-design-guide/welcome.html)
* **Canonical Scope:** High-resilience hybrid cloud interconnects.
* **VIF Architecture:**
  * **Private VIF:** Connects directly to a specific Virtual Private Gateway (VGW) in a single VPC.
  * **Transit VIF:** Connects directly to an AWS Direct Connect Gateway attached to a Transit Gateway (supports interconnecting multiple VPCs across accounts/regions).
  * **Public VIF:** Connects to public AWS endpoints (S3, DynamoDB, public APIs) over dedicated connection; can run IPSec VPN over Public VIF for end-to-end encryption.
  * **BGP Routing & Failover:** Active/Passive DX connections with BGP Local Preference (higher preferred) and AS Path Prepending for return traffic path selection.

---

## 2. Official AWS Documentation & User Guides to Master

### 2.1 [AWS Organizations & AWS RAM](https://docs.aws.amazon.com/organizations/latest/userguide/)
* **Deep Dive:**
  * **AWS Resource Access Manager (RAM):** Sharing Transit Gateways, Subnets (VPC sharing), Route 53 Resolver Rules, and License Manager licenses across accounts without duplicating resources.
  * **VPC Sharing:** Central Network Account owns the VPC and subnets; Workload Accounts deploy EC2/EKS resources into shared subnets. Security groups remain isolated per account.

### 2.2 [AWS IAM Identity Center (Successor to AWS SSO)](https://docs.aws.amazon.com/singlesignon/latest/userguide/)
* **Deep Dive:**
  * Multi-account centralized user authentication integrating with external Identity Providers (Okta, Azure AD / Microsoft Entra, Ping) via SAML 2.0 and SCIM directory synchronization.
  * Permission Sets assigning IAM policies and session duration dynamically across targeted accounts and OUs.

### 2.3 [AWS VPC Lattice](https://docs.aws.amazon.com/vpc-lattice/latest/ug/)
* **Deep Dive:**
  * Application-layer service-to-service networking across VPCs and accounts without complex peering, Transit Gateways, or IP CIDR management. Built-in auth policies via IAM SigV4.

---

## 3. High-Frequency Exam Traps & Architectural Reflexes

1. **Direct Connect Failover with BGP:**
   * Primary DX link fails over to secondary DX or Site-to-Site VPN.
   * *Trap:* For ingress traffic into on-premises, advertise more specific CIDRs or shorter AS Path over primary DX. For egress traffic from on-premises to AWS, configure BGP Local Preference (e.g., 200 for primary DX, 100 for backup DX/VPN).
2. **Transit Gateway Asymmetric Routing Failure:**
   * Stateful third-party firewalls behind Gateway Load Balancer drop TCP traffic because return packets hit a different firewall in another AZ.
   * *Resolution:* Enable **Transit Gateway Appliance Mode** on the inspection VPC attachment to pin traffic symmetrically to the same AZ.
3. **Overlapping CIDR Interconnects:**
   * Two acquired companies with identical `10.0.0.0/16` networks must interconnect.
   * *Resolution:* Use Private NAT Gateways on Transit Gateway OR AWS PrivateLink / VPC Lattice to expose specific service endpoints without IP address collisions.

---

## 4. Reading & Study Checklist

- [ ] Read *Organizing Your AWS Environment Using Multiple Accounts*
- [ ] Read *Building a Transit Gateway Network with Centralized Inspection*
- [ ] Read *Hybrid Cloud DNS Options for Amazon VPC*
- [ ] Master Direct Connect Private VIF vs Transit VIF vs Public VIF routing models
- [ ] Review AWS RAM shared VPC topology and security group boundaries
- [ ] Master Route 53 Inbound and Outbound Resolver Endpoint configurations
