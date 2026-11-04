# Domain 2: Managing and Provisioning Infrastructure & Networking — Quiz 3

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
A media company needs to serve real-time video streams using HTTP Live Streaming (HLS) to millions of concurrent viewers across South America, Europe, and Asia. What combination of Google Cloud networking components provides the lowest latency and highest cache hit ratio?
- A. Cloud SQL and Compute Engine in a single zone
- B. Compute Engine VMs with public IP addresses and local Apache web servers
- C. Internal Load Balancer with Cloud NAT
- D. Global External HTTP(S) Load Balancer paired with Cloud CDN and Google Cloud Storage multi-region backend buckets

---

### Question 2
An enterprise security team requires that all outbound DNS queries from instances in a VPC be logged and inspected for DNS data exfiltration attempts. What Google Cloud networking feature should be enabled?
- A. VPC Flow Logs
- B. Cloud NAT
- C. Cloud DNS Logging
- D. Cloud Armor

---

### Question 3
A company wants to analyze network traffic patterns, troubleshoot connectivity issues, and verify that firewall rules are operating as intended without installing packet capture software on individual VMs. Which VPC feature provides sampled records of network flows?
- A. Cloud Profiler
- B. Cloud Trace
- C. Cloud Audit Logs
- D. VPC Flow Logs

---

### Question 4
An architect needs to set up a private connection between an application in VPC-1 and a Cloud SQL instance in a Google-managed VPC. The architect uses Private Services Access. Under the hood, what networking mechanism connects VPC-1 to the Google-managed services network?
- A. Dedicated Interconnect
- B. VPC Network Peering with an allocated RFC 1918 internal IP range
- C. Public internet routing
- D. Cloud NAT

---

### Question 5
A company is configuring cross-project networking. They have 10 separate production microservices running in 10 different GCP projects that need to communicate with a central authentication service in an identity project. They do not want to use Shared VPC because each project has independent networking teams with non-overlapping subnets. What networking feature allows direct, private, low-latency connectivity between these VPCs?
- A. VPC Network Peering
- B. Cloud NAT
- C. Public IP addresses on all VMs
- D. Cloud Storage FUSE

---

### Question 6
An architect wants to implement a disaster recovery test where traffic directed to an External HTTP(S) Load Balancer is shifted from the primary region (`us-east1`) to the secondary region (`us-west1`). How can this traffic shift be executed cleanly at the load balancer without altering DNS records?
- A. Change the domain registrar IP address.
- B. Update the backend service capacity settings (e.g. set max utilization / max rate to 0 on the `us-east1` backend group), allowing Anycast to seamlessly steer traffic to `us-west1`.
- C. Restart the load balancer.
- D. Delete the backend service in `us-east1`.

---

### Question 7
What is the primary operational benefit of using Hierarchical Firewall Policies in Google Cloud?
- A. They provide free DDoS mitigation.
- B. They eliminate the need for subnets.
- C. They allow security administrators to define and enforce organizational firewall rules at the Organization and Folder levels that cannot be overridden by project-level firewall rules.
- D. They make VMs boot 10% faster.

---

### Question 8
A multinational retailer wants to block all HTTP traffic originating from specific sanctioned countries from accessing their public web store. Which Google Cloud feature enables geographic IP filtering at the network edge?
- A. Linux `iptables` on every VM
- B. VPC subnet route tables
- C. Cloud NAT
- D. Cloud Armor Geographic Access Control rules

---

### Question 9
An enterprise deploys a fleet of Compute Engine instances in a managed instance group across three zones in `europe-west4`. The team wants instances to automatically scale up when CPU utilization exceeds 75% or when the HTTP request rate per instance exceeds 500 requests per second. What component coordinates this scaling?
- A. Compute Engine Autoscaler configured with multiple metrics (CPU target and Load Balancing serving capacity)
- B. Cloud Router
- C. Cron script running in Cloud Shell
- D. Cloud Storage Lifecycle manager

---

### Question 10
When designing a Shared VPC architecture, what is the maximum number of Host Projects that a single Service Project can be attached to at any given time?
- A. Unlimited
- B. Ten
- C. Exactly one
- D. Fifty

---

## Answer Key & Rationales

1. **D** — Global External HTTPS LB with Cloud CDN caches media fragments at Anycast edge PoPs worldwide, minimizing origin latency.
2. **C** — Cloud DNS Logging records all domain resolution queries made by VPC instances, enabling detection of DNS tunneling and exfiltration.
3. **D** — VPC Flow Logs capture sampled Layer 4 network flow telemetry (source/dest IP, port, packet count, latency) for network diagnostics and security.
4. **B** — Private Services Access establishes an internal VPC Network Peering connection to Google-managed service producer VPC networks.
5. **A** — VPC Network Peering connects independent VPCs privately across projects with zero gateway bottlenecks.
6. **B** — Adjusting backend capacity to zero at the load balancer triggers immediate, graceful traffic migration to alternate backends without DNS cache delay.
7. **C** — Hierarchical Firewall Policies enforce consistent security rules organization-wide, overriding project-level changes.
8. **D** — Cloud Armor supports geolocation-based filtering (e.g. `origin.region_code == 'XX'`) to enforce regional compliance.
9. **A** — Compute Engine Autoscalers dynamically size MIGs based on composite metrics including CPU utilization and load balancer request rates.
10. **C** — A Service Project can attach to only one Host Project at a time, establishing a single authoritative network boundary.
