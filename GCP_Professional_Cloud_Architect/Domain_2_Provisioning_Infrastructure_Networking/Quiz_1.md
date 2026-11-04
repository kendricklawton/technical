# Domain 2: Managing and Provisioning Infrastructure & Networking — Quiz 1

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
An enterprise network team wants to centrally manage subnets, firewall rules, and hybrid Interconnect connections across 50 business departments. However, each department must retain full administrative control over their own Compute Engine instances and Kubernetes clusters without having permissions to modify subnets or firewall policies. Which architecture fulfills these requirements?
- A. Create a single monolithic project with 500 IAM users.
- B. Establish a Shared VPC architecture where a central Host Project manages the network infrastructure, and departmental Service Projects attach to specific subnets.
- C. Create 50 separate VPCs and connect all of them in a full-mesh VPC Network Peering topology.
- D. Give all developers the `Editor` role.

---

### Question 2
A financial enterprise requires a hybrid dedicated connection between their primary on-premises datacenter and Google Cloud. The connection must support 20 Gbps of mission-critical traffic, have a 99.99% availability SLA, and connect to two separate Google Cloud edge locations in different metropolitan areas. Which hybrid connectivity option should be selected?
- A. Public Internet with Cloud NAT
- B. Cloud HA VPN with 2 tunnels
- C. Dedicated Interconnect with 4 circuits configured across 2 metros and dual edge Cloud Routers
- D. VPC Network Peering over the public internet

---

### Question 3
An application deployed in a private subnet on Google Cloud needs to access Cloud Storage buckets and BigQuery datasets. Security policy strictly mandates that the VMs must NOT have external public IP addresses, and traffic must NOT traverse the public internet. Which feature should be enabled on the subnet?
- A. Cloud DNS Peering
- B. Assigning ephemeral public IPs
- C. Private Google Access
- D. Cloud NAT

---

### Question 4
A SaaS company wants to allow external customer VPC networks to consume their proprietary analytics microservice securely and privately, without allowing the customer VPCs to access any other internal services or requiring IP address coordination to prevent RFC 1918 overlaps. Which technology should the SaaS company implement?
- A. Private Service Connect (PSC) Published Service with Service Attachments
- B. VPC Network Peering
- C. Public Load Balancer with basic authentication
- D. Cloud VPN with static routing

---

### Question 5
An architect is designing an e-commerce website that serves users globally. The system must terminate TLS connections at the edge point of presence nearest to the customer, route traffic across Google's private backbone to healthy backends in Europe or the US, and automatically fail over within seconds if a regional outage occurs. Which load balancer meets these requirements?
- A. Regional External HTTP(S) Load Balancer
- B. Internal HTTP(S) Load Balancer
- C. Global External HTTP(S) Load Balancer (Anycast)
- D. External TCP/UDP Network Load Balancer (Passthrough)

---

### Question 6
An organization has three VPC networks: VPC-A, VPC-B, and VPC-C. VPC-A is peered with VPC-B, and VPC-B is peered with VPC-C. A developer in VPC-A attempts to ping an instance in VPC-C and the connection times out. Why is this occurring?
- A. The developer needs to reboot their laptop.
- B. Google Cloud VPCs do not support ICMP ping.
- C. Cloud Router is required for all VPCs.
- D. VPC Network Peering is non-transitive; traffic from VPC-A cannot transit through VPC-B to reach VPC-C.

---

### Question 7
Compute Engine instances in a private subnet need to download security OS patches from external software repositories on the public internet. The instances must not have public IP addresses assigned, and inbound connections from the internet must be blocked. Which managed service provides outbound internet connectivity?
- A. Private Service Connect
- B. Cloud Interconnect
- C. Cloud Armor
- D. Cloud NAT (Network Address Translation)

---

### Question 8
An architect wants to protect a web application from SQL injection, Cross-Site Scripting (XSS), and Layer 7 volumetric HTTP DDoS floods. Which Google Cloud service integrates natively with the Global External HTTP(S) Load Balancer to enforce these protections?
- A. VPC Firewall Rules alone
- B. Cloud Armor
- C. Cloud DNS
- D. Cloud NAT

---

### Question 9
A company wants to manage their Google Cloud infrastructure using Terraform. How should the team configure the Terraform backend to ensure safe team collaboration and state integrity?
- A. Store the Terraform state file in a Google Cloud Storage bucket with Object Versioning enabled and IAM permissions restricted to the CI/CD pipeline.
- B. Keep the state file on the lead architect's local laptop.
- C. Store the state file on an ephemeral compute instance disk.
- D. Store the `terraform.tfstate` file in a public GitHub repository.

---

### Question 10
When configuring Cloud HA VPN between Google Cloud and an on-premises data center, which routing mechanism must be used to achieve the 99.99% service availability SLA?
- A. Dynamic routing using Cloud Router and BGP (Border Gateway Protocol)
- B. Policy-based routing
- C. DNS-based round-robin routing
- D. Static routing with manual IP route creation

---

## Answer Key & Rationales

1. **B** — Shared VPC separates network management (central host project) from application resource provisioning (service projects), enforcing strict least privilege.
2. **C** — A 99.99% SLA for Dedicated Interconnect requires 4 circuits across dual metros and dual Cloud Routers to survive edge datacenter failures.
3. **C** — Private Google Access enables VMs without public IPs to communicate with Google APIs privately over Google's internal network.
4. **A** — Private Service Connect enables private, unidirectional service publication and consumption without network peering complexity or IP overlap conflicts.
5. **C** — Global External HTTP(S) Load Balancer uses Anycast IP addresses to terminate traffic at edge PoPs and route globally with sub-second failover.
6. **D** — VPC Network Peering is strictly non-transitive. VPC-A cannot route through VPC-B to reach VPC-C.
7. **D** — Cloud NAT provides outbound internet connectivity for private instances without exposing them to inbound internet connections.
8. **B** — Cloud Armor provides enterprise WAF rules (OWASP Core Rule Set) and DDoS mitigation integrated with Google Cloud external load balancers.
9. **A** — A Cloud Storage bucket with Object Versioning and state locking provides safe, auditable, and resilient remote state storage for Terraform.
10. **A** — HA VPN mandates dynamic BGP routing via Cloud Router across dual IPsec tunnels to qualify for the 99.99% SLA.
