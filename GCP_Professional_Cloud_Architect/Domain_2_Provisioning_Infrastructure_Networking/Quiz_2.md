# Domain 2: Managing and Provisioning Infrastructure & Networking — Quiz 2

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
An enterprise has configured a Shared VPC network. A DevOps engineer in a Service Project attempts to create a GKE cluster but receives a permission denied error when attaching worker nodes to the shared subnet. What IAM role is missing, and where must it be granted?
- A. Grant `roles/owner` to the developer on the Service Project.
- B. Grant `roles/compute.networkUser` to the GKE Service Account of the Service Project on the specific subnet in the Host Project.
- C. Grant `roles/resourcemanager.organizationAdmin` to the developer.
- D. Delete the Shared VPC.

---

### Question 2
A retail company is preparing for Black Friday. They expect a 10x surge in image and video asset downloads. The assets are stored in a Cloud Storage bucket behind an External HTTP(S) Load Balancer. What feature should be enabled on the backend bucket to cache static assets at edge PoPs globally and minimize origin bucket traffic?
- A. Cloud CDN
- B. Cloud NAT
- C. Cloud Trace
- D. VPC Service Controls

---

### Question 3
An architect needs to configure internal load balancing for an array of private microservices running on Compute Engine VMs across multiple zones within the `us-east1` region. The load balancer must have zero proxy overhead, preserve client IP addresses, and support sub-second health checks. Which load balancer should be deployed?
- A. Global External HTTP(S) Load Balancer
- B. Internal TCP/UDP Network Load Balancer
- C. Classic Load Balancer
- D. Cloud DNS

---

### Question 4
A corporation connects their corporate office to Google Cloud using Partner Interconnect. The corporate network team wants to route traffic destined for Google Cloud subnets using BGP. What Google Cloud networking component terminates the BGP session inside the VPC?
- A. Cloud NAT
- B. Cloud Router
- C. Cloud Armor
- D. Bastion Host

---

### Question 5
A company wants to migrate an existing monolithic application running on VMware in an on-premises datacenter directly to Google Cloud without converting VM formats, refactoring architectures, or modifying IP networking configurations. What solution provides native VMware stack support on Google Cloud?
- A. Google Cloud VMware Engine (GCVE)
- B. Cloud Run
- C. App Engine
- D. Bigtable

---

### Question 6
An architect is designing firewall rules for an enterprise VPC. The security policy mandates that database instances in the `db` subnet must only accept inbound MySQL connections on port 3306 from application servers in the `app` subnet. What is the recommended Google Cloud firewall construct to identify and group these VMs securely?
- A. Hardcoded public IP addresses
- B. Network Service Accounts (IAM-based) or Network Tags
- C. Linux hostname matching
- D. Subnet mask `/32` on every VM

---

### Question 7
When using Cloud Router for dynamic routing with on-premises networks, what is the default routing mode for Google Cloud VPC networks, and how does it differ from Global dynamic routing?
- A. Default is Global; Regional routing is unsupported.
- B. Regional dynamic routing advertises and learns routes only for subnets in the same region as the Cloud Router; Global dynamic routing advertises and propagates routes across all regions in the VPC.
- C. Regional dynamic routing requires fiber cables.
- D. Global routing disables BGP.

---

### Question 8
An organization requires that egress traffic from private Compute Engine instances in a VPC pass through a third-party firewall appliance (Next-Generation Firewall VM) before going to the public internet. What Google Cloud networking construct directs traffic to the firewall VM?
- A. Custom Route with next-hop set to the firewall VM instance IP
- B. Cloud NAT gateway
- C. Cloud DNS Peering
- D. Cloud Armor

---

### Question 9
A company wants to manage Google Cloud resources using Kubernetes manifests alongside their container deployments in GKE. Which Google Cloud open-source tool runs as a Kubernetes operator and synchronizes CRDs into GCP cloud resources?
- A. Config Connector
- B. Helm alone
- C. Cloud Shell
- D. Skaffold

---

### Question 10
An architect needs to ensure that outbound network traffic between two Compute Engine VMs in the same Google Cloud region is encrypted. Does Google Cloud natively encrypt this traffic?
- A. No, all intra-cloud traffic is sent in plaintext unless the user configures IPsec tunnels.
- B. Yes, Google Cloud automatically encrypts all VM-to-VM traffic within Google's data centers and over Google's backbone network at the physical network layer (using hardware-level encryption / PSP / MACsec).
- C. Only if the VMs are in the same zone.
- D. Encryption is only supported on Windows VMs.

---

## Answer Key & Rationales

1. **B** — Service projects require `roles/compute.networkUser` on the host subnet so their service accounts can attach resources (VMs/GKE nodes) to that subnet.
2. **A** — Cloud CDN integrates with External HTTP(S) Load Balancer backend buckets to cache media at edge locations worldwide, slashing latency and origin egress costs.
3. **B** — Internal TCP/UDP Load Balancers operate at Layer 4 via SDN routing, preserving client IP with zero proxy latency.
4. **B** — Cloud Router handles dynamic BGP peering sessions, route advertisements, and learned routes for Interconnect and HA VPN.
5. **A** — GCVE provides a fully managed VMware Cloud Foundation stack (ESXi, vCenter, vSAN, NSX-T) on dedicated Google Cloud bare-metal infrastructure.
6. **B** — Network Service Accounts provide cryptographically verifiable identity-based firewall rules that cannot be spoofed by VM tag tampering.
7. **B** — Regional dynamic routing restricts route propagation to the Cloud Router's local region; Global dynamic routing propagates learned on-premises routes network-wide.
8. **A** — Custom VPC routes can specify a Next-Hop Instance (firewall appliance VM) to intercept and inspect egress internet traffic.
9. **A** — Config Connector enables declarative GCP infrastructure management through Kubernetes custom resource definitions.
10. **B** — Google Cloud encrypts all intra-cloud traffic automatically using default hardware-accelerated encryption across physical links.
