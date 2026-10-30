# Domain 3 (Resilient Cloud Solutions) — Quiz 3: Route 53 Advanced Routing & Traffic Management

**Topic:** Route 53 Routing Policies, Health Checks, DNS Failover & Application Recovery Controller (ARC)  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: Cross-Account Private Hosted Zone VPC Association

A multi-account enterprise operates a centralized Shared Services AWS account hosting an internal Amazon Route 53 Private Hosted Zone (`corp.internal`). Workloads deployed across 40 spoke AWS accounts need to resolve records in `corp.internal`. Company policy strictly prohibits using public hosted zones or granting spoke accounts access to manage records in the centralized hosted zone.

**Which sequence of AWS CLI and API operations establishes this cross-account DNS resolution with LEAST privilege?**
- A. In the Shared Services account, execute `aws route53 create-vpc-association-authorization` for each spoke VPC; in each spoke account, execute `aws route53 associate-vpc-with-hosted-zone` specifying the Shared Services hosted zone ID.
- B. In the Shared Services account, share the Private Hosted Zone via AWS Resource Access Manager (RAM); spoke accounts accept the RAM share and automatically bind their VPCs.
- C. In each spoke account, create a local Private Hosted Zone named `corp.internal` and establish Route 53 Resolver Rules forwarding queries to the default Amazon VPC DNS resolver (`169.254.169.253`).
- D. In the Shared Services account, attach a Resource-based Policy to the hosted zone allowing `route53:AssociateVPCWithHostedZone` for `*`, enabling spoke accounts to attach VPCs directly without prior authorization.

---

### Question 2: Instant Failover Control Planes: Route 53 ARC Routing Controls vs DNS TTL

A global payments gateway requires an Active-Passive multi-region failover mechanism between `us-east-1` and `eu-central-1`. The system architecture demands an RTO of under 10 seconds. The infrastructure team observes that during past DR drills using standard Route 53 DNS Failover with a 60-second TTL, client ISP DNS resolvers cached stale records for up to 15 minutes, violating the 10-second SLA.

**Which AWS architectural solution achieves deterministic, sub-second regional failover without being subject to public DNS resolver caching delays?**
- A. Deploy Amazon Route 53 Application Recovery Controller (ARC) Routing Controls linked to regional ALB health checks, and expose endpoints through AWS Global Accelerator, which reroutes anycast IP traffic within seconds when routing controls toggle.
- B. Set Route 53 DNS record TTL to 0 seconds on the primary and secondary Alias records, forcing all public resolvers to re-resolve records on every single transaction.
- C. Deploy an AWS CloudFront distribution in front of regional ALBs, configuring CloudFront Origin Request Policies to dynamically switch origins using an AWS Lambda@Edge function triggered on origin errors.
- D. Author an AWS Step Functions state machine that invokes the Route 53 `ChangeResourceRecordSets` API to replace the primary IP with the secondary IP whenever CloudWatch alarms trigger.

---

### Question 3: Dynamic Traffic Steering via Geoproximity Routing and Bias Adjustments

A streaming video platform deploys backend transcoding infrastructure in two AWS Regions: `us-west-2` (Oregon) and `us-east-1` (N. Virginia). Due to unexpected hardware upgrades, `us-west-2` compute capacity is constrained, while `us-east-1` has massive idle capacity. The operations team needs to dynamically shift 40% of western US customer traffic eastward to Virginia without modifying application DNS endpoints or disrupting existing user connections.

**Which Route 53 routing architecture facilitates this dynamic geographic load redistribution?**
- A. Configure a Route 53 Geoproximity Routing policy using Route 53 Traffic Flow, decrease the bias on the `us-west-2` endpoint (e.g., to -20), and increase the bias on the `us-east-1` endpoint (e.g., to +20) to expand the eastern region's geographic catchment zone.
- B. Configure a Route 53 Geolocation Routing policy that maps individual US state codes to `us-east-1` and deploy an EventBridge rule to rotate states based on CPU utilization.
- C. Configure a Route 53 Latency-Based Routing policy with CloudWatch Metric Math filters that artificially inject synthetic latency into Oregon's DNS metrics.
- D. Configure a Route 53 Weighted Routing policy with a 60:40 ratio and attach an inverted health check to Oregon's Application Load Balancer.

---

### Question 4: Multi-Tier Calculated Health Checks with Inverted Dependencies

An enterprise web application relies on an Application Load Balancer frontend and a backend payment microservice. An engineer configures a primary Route 53 DNS Failover record pointing to the ALB. However, during an outage, the ALB continued returning HTTP 200 responses with an error HTML page when the backend payment microservice was down, preventing Route 53 from triggering failover to the disaster recovery region. Additionally, an administrative override switch is required to manually force failover during maintenance.

**How should Route 53 Health Checks be structured to handle both deep application dependencies and administrative manual overrides?**
- A. Create child health checks for ALB HTTP status, backend microservice string matching, and an inverted CloudWatch metric alarm health check for the manual override switch, combining them into a Calculated Health Check with an `AND` operator linked to the primary Alias record.
- B. Attach three independent standard HTTP health checks directly to the primary Alias record in Route 53; Route 53 will automatically evaluate all three using an implicit `AND` condition.
- C. Configure an AWS Lambda function that queries the payment microservice every 10 seconds and executes `aws route53 update-health-check` to toggle the status string.
- D. Deploy an AWS WAF rule that returns HTTP 503 whenever backend payment services fail, allowing standard ALB health checks to mark the instance unhealthy.

---

### Question 5: Hybrid DNS Architecture via Route 53 Resolver Endpoints

An enterprise connects its on-premises corporate data center to AWS through AWS Direct Connect. On-premises client machines need to resolve private AWS domains ending in `.aws.internal` hosted in Route 53 Private Hosted Zones. Simultaneously, Amazon EC2 instances in AWS need to resolve on-premises legacy corporate hostnames ending in `.corp.local` hosted on Microsoft Active Directory DNS servers (`10.0.10.50`, `10.0.10.51`).

**Which Route 53 Resolver architecture fulfills this bidirectional DNS resolution requirement?**
- A. Provision Route 53 Resolver Inbound Endpoints in the VPC for on-premises clients to query `.aws.internal`, and provision Route 53 Resolver Outbound Endpoints with a Forwarding Rule for `.corp.local` pointing to the Active Directory DNS IPs.
- B. Deploy Microsoft Active Directory Domain Controllers on EC2 in the VPC and configure DNS zone transfers (AXFR) over Direct Connect between Route 53 and on-premises DNS.
- C. Provision Route 53 Resolver Outbound Endpoints for both directions, and attach an S3 bucket containing static BIND zone files to the Route 53 Private Hosted Zone.
- D. Configure on-premises DNS forwarders to query the default Amazon VPC DNS resolver IP address (`169.254.169.253`) over the Direct Connect virtual interface.

---

### Question 6: Route 53 Resolver DNS Firewall for Perimeter Threat Mitigation

A financial services firm requires strict outbound network controls for its EC2 workloads. Workloads must be prohibited from resolving domain names associated with known command-and-control (C2) botnets, cryptocurrency miners, and newly registered domains. Furthermore, any unauthorized attempt to resolve domains outside of an approved corporate whitelist must be blocked and audited in Amazon CloudWatch Logs.

**Which AWS native service should be implemented to enforce DNS-layer inspection and domain filtering within the VPC?**
- A. Associate Route 53 Resolver DNS Firewall rule groups with the VPC, configure managed threat intelligence domain lists with `BLOCK` actions, attach a custom whitelist domain list with `ALLOW`, and enable query logging to CloudWatch Logs.
- B. Deploy AWS Network Firewall with Suricata rules inspecting port 53 traffic, disabling the Amazon VPC DNS server at `169.254.169.253`.
- C. Attach AWS WAF WebACLs to all VPC subnets with managed IP reputation rules and DNS rate-limiting expressions.
- D. Deploy custom BIND9 DNS forwarder proxy EC2 instances in each subnet and enforce iptables packet drops on external DNS queries.

---

### Question 7: Route 53 Application Recovery Controller (ARC) Readiness Checks

A multi-region enterprise deploys an active-active architecture across `us-east-1` and `us-west-2`. To ensure that either region can handle 100% of global traffic during a regional failover event, the SRE team needs continuous, automated verification of resource capacity, Auto Scaling Group limits, and service quotas (e.g., EBS volume limits, ALB bandwidth) across both regions **before** an emergency failover is initiated.

**Which AWS service capability provides this pre-failover automated capacity and configuration audit?**
- A. Route 53 Application Recovery Controller (ARC) Readiness Checks: Continuous automated auditing of AWS resource quotas, ASG capacity configurations, and network topology across recovery cells to verify failover readiness.
- B. AWS Trusted Advisor automated checks executed via Amazon EventBridge and Step Functions.
- C. AWS Config Conformance Packs evaluating `desired-instance-type` rules across both regions every 24 hours.
- D. CloudWatch Synthetics Canaries executing simulated user load tests against regional endpoints every 5 minutes.

---

### Question 8: High-Availability Failover to S3 Static Maintenance Site via Route 53

An e-commerce application frontend is hosted on Amazon EC2 instances behind an Application Load Balancer. If all EC2 instances or the entire backend ALB become unhealthy, Route 53 must automatically fail over to an Amazon S3 bucket configured for static website hosting that displays a branded maintenance page.

**What is the required configuration for the primary and secondary Route 53 DNS records and health checks?**
- A. Configure a primary Alias record pointing to the ALB with `EvaluateTargetHealth: true` and routing policy `Failover (Primary)`; configure a secondary Alias record pointing to the S3 website endpoint with routing policy `Failover (Secondary)` and no health check.
- B. Configure a primary CNAME record pointing to the ALB DNS name with a CloudWatch Alarm health check; configure a secondary CNAME record pointing to the S3 bucket ARN with `EvaluateTargetHealth: true`.
- C. Configure a primary A record with Weighted routing set to 100 pointing to the ALB; configure a secondary A record with Weighted routing set to 0 pointing to the S3 bucket IP address.
- D. Configure an ALB custom error response rule returning 302 redirects to the S3 website URL whenever target instances fail health checks.

---

### Question 9: Multivalue Answer Routing vs Standard Simple Routing

A DevOps engineer needs to configure DNS for a microservice fleet consisting of 8 independent, internet-facing EC2 instances. The system requires client-side load balancing where Route 53 returns multiple healthy IP addresses in response to a single DNS query, while automatically filtering out any instances that fail HTTP health checks.

**Which Route 53 routing policy satisfies these requirements?**
- A. Multivalue Answer Routing: Allows configuring up to 8 healthy records in response to each query, associating Route 53 health checks with each record so that unhealthy instances are automatically removed from DNS responses.
- B. Simple Routing: Allows entering multiple IP addresses in a single record and associates Route 53 health checks to selectively drop failed IPs.
- C. Weighted Routing: Assigns equal weights of 12.5 to all 8 records, returning all 8 IPs simultaneously to every DNS client.
- D. Latency-Based Routing: Returns all 8 IPs ordered strictly by client round-trip ping time.

---

## Answer Key & Explanations

### Question 1: Cross-Account Private Hosted Zone VPC Association
**Answer:** A  
**Explanation:** 
When associating a VPC in Account B with a Route 53 Private Hosted Zone located in Account A:
1. Account A (owner of the hosted zone) must explicitly authorize the cross-account association by running `aws route53 create-vpc-association-authorization --hosted-zone-id <ID> --vpc VPCRegion=<region>,VPCId=<spoke-vpc-id>`.
2. Account B (owner of the spoke VPC) can then execute `aws route53 associate-vpc-with-hosted-zone --hosted-zone-id <ID> --vpc VPCRegion=<region>,VPCId=<spoke-vpc-id>`.
3. Optionally, Account A can delete the authorization once the association is established.

Private Hosted Zones **cannot** be shared via AWS RAM (Option B is invalid; only Route 53 Resolver rules and VPC subnets can be shared via RAM). The default Amazon VPC DNS resolver address `169.254.169.253` (Option C) cannot resolve cross-account private hosted zones without VPC association.

---

### Question 2: Instant Failover Control Planes: Route 53 ARC Routing Controls vs DNS TTL
**Answer:** A  
**Explanation:** 
Standard DNS-based failover relies on updating DNS record pointers and waiting for client resolvers to expire their cached TTLs. Even with low TTLs (e.g., 60 seconds), many recursive public DNS resolvers (ISPs, corporate firewalls) cache DNS entries for minutes or hours, making an RTO under 10 seconds impossible to guarantee.

**AWS Route 53 Application Recovery Controller (ARC) Routing Controls** provide simple, highly available on/off switches deployed across 5 regional control plane endpoints (running independently of the AWS management plane). When paired with **AWS Global Accelerator**, traffic shifting occurs at the anycast BGP routing layer in AWS edge locations in **under a few seconds**, completely bypassing DNS TTL propagation constraints and public recursive resolver caching issues.

---

### Question 3: Dynamic Traffic Steering via Geoproximity Routing and Bias Adjustments
**Answer:** A  
**Explanation:** 
Route 53 **Geoproximity Routing** lets Amazon Route 53 route traffic based on the physical geographic location of users and resources. 
- Using Route 53 Traffic Flow, you can define **biases** to expand or shrink the geographic region from which traffic is routed to a given resource.
- To shift traffic from an overloaded region (`us-west-2`) to an underutilized region (`us-east-1`), you decrease the bias on `us-west-2` (negative bias, down to -99) and/or increase the bias on `us-east-1` (positive bias, up to +99).
- This dynamically redraws the geographic boundary between the two endpoints, causing customers in intermediate geographic zones to be routed to `us-east-1` without altering DNS record types.

---

### Question 4: Multi-Tier Calculated Health Checks with Inverted Dependencies
**Answer:** A  
**Explanation:** 
A **Calculated Health Check** in Route 53 monitors the status of up to 256 child health checks and evaluates them using boolean logic (`AND`, `OR`, `NOT`, or `At least M of N`):
1. **ALB Status Check:** Verifies the load balancer endpoint is reachable.
2. **Payment Microservice Check:** Uses HTTP string matching to inspect the payload returned by a deep diagnostic endpoint (e.g., `{"database": "ok", "stripe": "ok"}`).
3. **Manual Administrative Override:** Configured via a CloudWatch metric alarm that watches a custom metric. By setting `Inverted: true`, when an operator sets the alarm metric to 1, the health check becomes unhealthy, allowing engineers to manually trip failover for drills or emergency maintenance.
4. Combining these child checks with an `AND` operator in a Calculated Health Check ensures failover occurs if any condition fails.

---

### Question 5: Hybrid DNS Architecture via Route 53 Resolver Endpoints
**Answer:** A  
**Explanation:** 
Route 53 Resolver enables hybrid cloud DNS:
- **Inbound Endpoints:** Provide dedicated ENIs with private IP addresses in your VPC. On-premises DNS servers forward queries for AWS private domains (`.aws.internal`) to these Inbound IP addresses over Direct Connect or VPN.
- **Outbound Endpoints:** Provide dedicated ENIs in your VPC that Route 53 Resolver uses to send DNS queries to on-premises DNS servers. You create a **Resolver Forwarding Rule** for the on-premises domain (`.corp.local`) pointing to the on-premises AD DNS IPs (`10.0.10.50`, `10.0.10.51`).

The AmazonProvidedDNS address `169.254.169.253` (Option D) is a link-local address accessible only from within an EC2 instance; it is completely unroutable over Direct Connect.

---

### Question 6: Route 53 Resolver DNS Firewall for Perimeter Threat Mitigation
**Answer:** A  
**Explanation:** 
**Route 53 Resolver DNS Firewall** provides managed DNS-layer security for Amazon VPCs:
- It allows creating rule groups containing domain lists that block, allow, or alert on DNS queries originating from within the VPC.
- AWS provides managed threat intelligence domain lists (e.g., malware domains, botnets, cryptomining).
- Custom whitelists and blacklists can be attached. Setting a default `BLOCK` rule on unknown domains while allowing approved corporate endpoints guarantees zero exfiltration via rogue DNS lookups.
- All DNS query actions are logged to CloudWatch Logs or S3 for security audit trails.

---

### Question 7: Route 53 Application Recovery Controller (ARC) Readiness Checks
**Answer:** A  
**Explanation:** 
**Route 53 ARC Readiness Checks** continuously monitor AWS resource configurations, capacity limits, and service quotas across multiple Availability Zones and AWS Regions (termed "recovery cells"):
- Before an actual outage occurs, Readiness Checks verify that the standby region has sufficient Auto Scaling limits, provisioned RDS capacity, EBS throughput, and network quotas to absorb 100% of the production workload.
- If an engineer resizes an Auto Scaling group in the primary region without matching the standby region, the Readiness Check trips to `NOT_READY`, alerting the team to capacity drift before an emergency cutover fails.

---

### Question 8: High-Availability Failover to S3 Static Maintenance Site via Route 53
**Answer:** A  
**Explanation:** 
To achieve active-passive failover to a static maintenance site:
- Create an **Alias record** pointing to the primary Application Load Balancer with routing policy `Failover`, type `Primary`, and enable `EvaluateTargetHealth: true`. This allows Route 53 to automatically monitor the ALB's backend health checks without creating a separate Route 53 health check.
- Create a secondary **Alias record** pointing to the Amazon S3 static website bucket endpoint with routing policy `Failover`, type `Secondary`. No health check is required on the secondary record because it serves as the ultimate fallback target.
- Note: S3 buckets used for website hosting with Route 53 Alias records must have the exact same name as the DNS record (e.g., `app.example.com`).

---

### Question 9: Multivalue Answer Routing vs Standard Simple Routing
**Answer:** A  
**Explanation:** 
**Multivalue Answer Routing** allows Route 53 to return multiple DNS records (up to 8 healthy records chosen randomly from all available healthy endpoints) in response to a single DNS query:
- Crucially, Multivalue Answer Routing supports associating a **Route 53 Health Check** with every individual record. If an instance fails its health check, Route 53 immediately stops including that IP in DNS query answers.
- In contrast, **Simple Routing** allows specifying multiple IP addresses in a single record, but it **does not support Route 53 health checks**. If one IP in a Simple record goes down, Route 53 continues returning the dead IP address to clients.
