# Domain 2: Managing and Provisioning Infrastructure & Networking — Quiz 2

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
An enterprise has configured a Shared VPC network. A DevOps engineer in a Service Project attempts to create a GKE cluster but receives a permission denied error when attaching worker nodes to the shared subnet. What IAM role is missing, and where must it be granted?
- A. Grant `roles/compute.networkUser` to the GKE Service Account of the Service Project on the specific subnet in the Host Project.
- B. Grant `roles/resourcemanager.organizationAdmin` to the developer.
- C. Delete the Shared VPC.
- D. Grant `roles/owner` to the developer on the Service Project.

---

### Question 2
A retail company is preparing for Black Friday. They expect a 10x surge in image and video asset downloads. The assets are stored in a Cloud Storage bucket behind an External HTTP(S) Load Balancer. What feature should be enabled on the backend bucket to cache static assets at edge PoPs globally and minimize origin bucket traffic?
- A. VPC Service Controls
- B. Cloud NAT
- C. Cloud Trace
- D. Cloud CDN

---

### Question 3
An architect needs to configure internal load balancing for an array of private microservices running on Compute Engine VMs across multiple zones within the `us-east1` region. The load balancer must have zero proxy overhead, preserve client IP addresses, and support sub-second health checks. Which load balancer should be deployed?
- A. Cloud DNS
- B. Classic Load Balancer
- C. Internal TCP/UDP Network Load Balancer
- D. Global External HTTP(S) Load Balancer

---

### Question 4
A corporation connects their corporate office to Google Cloud using Partner Interconnect. The corporate network team wants to route traffic destined for Google Cloud subnets using BGP. What Google Cloud networking component terminates the BGP session inside the VPC?
- A. Cloud Router
- B. Cloud NAT
- C. Bastion Host
- D. Cloud Armor

---

### Question 5
A company wants to migrate an existing monolithic application running on VMware in an on-premises datacenter directly to Google Cloud without converting VM formats, refactoring architectures, or modifying IP networking configurations. What solution provides native VMware stack support on Google Cloud?
- A. App Engine
- B. Google Cloud VMware Engine (GCVE)
- C. Bigtable
- D. Cloud Run

---

### Question 6
An architect is designing firewall rules for an enterprise VPC. The security policy mandates that database instances in the `db` subnet must only accept inbound MySQL connections on port 3306 from application servers in the `app` subnet. What is the recommended Google Cloud firewall construct to identify and group these VMs securely?
- A. Linux hostname matching
- B. Hardcoded public IP addresses
- C. Network Service Accounts (IAM-based) or Network Tags
- D. Subnet mask `/32` on every VM

---

### Question 7
When using Cloud Router for dynamic routing with on-premises networks, what is the default routing mode for Google Cloud VPC networks, and how does it differ from Global dynamic routing?
- A. Global routing disables BGP.
- B. Regional dynamic routing advertises and learns routes only for subnets in the same region as the Cloud Router; Global dynamic routing advertises and propagates routes across all regions in the VPC.
- C. Default is Global; Regional routing is unsupported.
- D. Regional dynamic routing requires fiber cables.

---

### Question 8
An organization requires that egress traffic from private Compute Engine instances in a VPC pass through a third-party firewall appliance (Next-Generation Firewall VM) before going to the public internet. What Google Cloud networking construct directs traffic to the firewall VM?
- A. Cloud NAT gateway
- B. Cloud DNS Peering
- C. Custom Route with next-hop set to the firewall VM instance IP
- D. Cloud Armor

---

### Question 9
A company wants to manage Google Cloud resources using Kubernetes manifests alongside their container deployments in GKE. Which Google Cloud open-source tool runs as a Kubernetes operator and synchronizes CRDs into GCP cloud resources?
- A. Skaffold
- B. Cloud Shell
- C. Helm alone
- D. Config Connector

---

### Question 10
An architect needs to ensure that outbound network traffic between two Compute Engine VMs in the same Google Cloud region is encrypted. Does Google Cloud natively encrypt this traffic?
- A. Only if the VMs are in the same zone.
- B. No, all intra-cloud traffic is sent in plaintext unless the user configures IPsec tunnels.
- C. Encryption is only supported on Windows VMs.
- D. Yes, Google Cloud automatically encrypts all VM-to-VM traffic within Google's data centers and over Google's backbone network at the physical network layer (using hardware-level encryption / PSP / MACsec).

---

## Answer Key & Rationales

1. **A** — Service projects require `roles/compute.networkUser` on the host subnet so their service accounts can attach resources (VMs/GKE nodes) to that subnet.
2. **D** — Cloud CDN integrates with External HTTP(S) Load Balancer backend buckets to cache media at edge locations worldwide, slashing latency and origin egress costs.
3. **C** — Internal TCP/UDP Load Balancers operate at Layer 4 via SDN routing, preserving client IP with zero proxy latency.
4. **A** — Cloud Router handles dynamic BGP peering sessions, route advertisements, and learned routes for Interconnect and HA VPN.
5. **B** — GCVE provides a fully managed VMware Cloud Foundation stack (ESXi, vCenter, vSAN, NSX-T) on dedicated Google Cloud bare-metal infrastructure.
6. **C** — Network Service Accounts provide cryptographically verifiable identity-based firewall rules that cannot be spoofed by VM tag tampering.
7. **B** — Regional dynamic routing restricts route propagation to the Cloud Router's local region; Global dynamic routing propagates learned on-premises routes network-wide.
8. **C** — Custom VPC routes can specify a Next-Hop Instance (firewall appliance VM) to intercept and inspect egress internet traffic.
9. **D** — Config Connector enables declarative GCP infrastructure management through Kubernetes custom resource definitions.
10. **D** — Google Cloud encrypts all intra-cloud traffic automatically using default hardware-accelerated encryption across physical links.
