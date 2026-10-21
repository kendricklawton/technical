# Domain 3 (Resilient Cloud Solutions) — Quiz 3: Route 53 Advanced Routing

**Topic:** Route 53 Routing Policies, Health Checks, DNS Failover & Application Recovery Controller  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: Route 53 Health Checks on Internal / Private Resources

A company operates backend services on Amazon EC2 instances inside private VPC subnets with no public IP addresses or Internet Gateways. The company wants Amazon Route 53 to perform health checks and fail over internal DNS queries in a **Private Hosted Zone** if an instance fails.

**Can public Route 53 Health Checkers monitor private IP addresses directly, and how should this be configured?**
- A. Yes, public Route 53 health checkers can query private IPs directly over the internet.
- B. No. Public Route 53 health checkers can only monitor public IPs. For private resources, you must create an **Amazon CloudWatch Metric Alarm** that monitors internal application health (e.g. ALB metrics or custom metrics pushed by instances), and create a Route 53 Health Check based on the **CloudWatch Metric Alarm**.
- C. Open port 80 to `0.0.0.0/0` on the private instance.
- D. Private Hosted Zones do not support health checks or failover.

---

### Question 2: Geolocation vs. Geoproximity Routing

An international platform needs to route end-users to the closest AWS region based on the geographic location of the user and the physical location of the AWS data centers. The company also wants the ability to dynamically expand or contract the geographic boundary (bias) served by a specific AWS region during high-demand events.

**Which Route 53 routing policy provides this geographic bias shifting?**
- A. Geolocation Routing
- B. **Geoproximity Routing** using **Route 53 Traffic Flow** with custom **bias** values.
- C. Multi-Value Answer Routing
- D. Simple Routing

---

### Question 3: Multi-Value Answer Routing vs. Simple Round-Robin

A company needs to distribute web traffic across 8 web servers. The company wants DNS queries to return up to 8 healthy IP addresses in response to client DNS lookups, with Route 53 automatically removing any IP address that fails an HTTP health check.

**Which Route 53 routing policy fulfills this requirement?**
- A. Simple Routing with multiple values.
- B. **Multi-Value Answer Routing** with a Route 53 Health Check associated with each record.
- C. Weighted Routing with weight 1 on all records.
- D. Latency-Based Routing.

---

### Question 4: Route 53 Application Recovery Controller (ARC) Zonal Shift

An application is deployed across three Availability Zones (`us-east-1a`, `us-east-1b`, `us-east-1c`) behind an Application Load Balancer. Due to a localized hardware impairment in `us-east-1a`, application response times degrade. The team needs to immediately divert 100% of traffic away from `us-east-1a` to the remaining two healthy zones without modifying DNS records or restarting instances.

**Which feature enables fast evacuation of a single degraded Availability Zone?**
- A. Auto Scaling scale-in.
- B. **Amazon Route 53 Application Recovery Controller (ARC) Zonal Shift**.
- C. Deleting the subnet in `us-east-1a`.
- D. WAF rate-limiting rules.

---

### Question 5: Route 53 Invert Health Check Status

An engineer is configuring a disaster recovery failover drill. They need to temporarily force Route 53 to consider a primary healthy endpoint as "unhealthy" to trigger an automated failover to the standby region without actually shutting down the primary servers.

**How can this drill be triggered quickly in Route 53?**
- A. Delete the primary hosted zone.
- B. In the Route 53 Health Check configuration, enable **Invert health check status**.
- C. Add an invalid character to the DNS domain name.
- D. Reboot the primary load balancer.

---

### Question 6: Route 53 Resolver for Hybrid Cloud DNS Resolution

A company operates an on-premises data center connected to an AWS VPC via AWS Direct Connect. On-premises servers need to resolve domain names in an AWS Route 53 **Private Hosted Zone** (`corp.internal`), while EC2 instances in the VPC need to resolve on-premises hostnames (`dc.corp.local`).

**Which Route 53 hybrid architecture enables bi-directional DNS resolution?**
- A. Install public DNS forwarders on the internet.
- B. Deploy **Route 53 Resolver Endpoints**:
  - An **Inbound Resolver Endpoint** in the VPC so on-premises DNS servers can query AWS Private Hosted Zones.
  - An **Outbound Resolver Endpoint** with **Route 53 Resolver Rules** forwarding queries for `dc.corp.local` to on-premises DNS server IPs.
- C. Use a public Route 53 hosted zone for corporate internal domains.
- D. S3 static website hosting for DNS records.

---

### Question 7: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap

A centralized CI/CD pipeline in Account 111111111111 deploys an ECS service in Account 222222222222. The deployment stage fails with: 'The ciphertext refers to a customer master key that does not exist, does not allow access, or is invalid.' The S3 bucket is encrypted with the default AWS-managed key 'aws/s3'. How should the engineer resolve this?

- A. Update the S3 bucket policy to grant read access to Account 222222222222.
- B. Reconfigure the S3 artifact bucket to use a Customer Managed Key (CMK) and grant `kms:Decrypt` and `kms:DescribeKey` permissions to Account 222222222222's execution role in the KMS Key Policy.
- C. Create a VPC Endpoint for S3 in Account 222222222222.
- D. Enable S3 Object Ownership with Bucket Owner Preferred.

---

### Question 8: CodeDeploy In-Place ApplicationStop Lifecycle Failure

During an in-place deployment to an EC2 Auto Scaling Group, the deployment fails at the ApplicationStop lifecycle event hook because a previous buggy script had been deployed in the older revision. How can the engineer force CodeDeploy to deploy the new fixed revision without terminating the instances?

- A. Delete the CodeDeploy agent from all EC2 instances.
- B. Trigger a new deployment via the AWS CLI with the `--ignore-application-stop-failures` flag enabled.
- C. Manually stop the EC2 instances in the AWS Management Console.
- D. Reboot the instances and rerun the same deployment ID.

---

### Question 9: CloudFormation Custom Resource 60-Minute Stack Freeze

A CloudFormation stack creation hangs for exactly 1 hour in `CREATE_IN_PROGRESS` on a Lambda-backed custom resource before failing and rolling back. What was the root cause?

- A. The Lambda function exceeded its maximum execution timeout of 15 minutes.
- B. An unhandled exception crashed the Lambda function before it could send an HTTP PUT response with status SUCCESS or FAILED to the pre-signed Amazon S3 ResponseURL.
- C. CloudFormation lost IAM permissions to invoke Lambda.
- D. The S3 bucket holding the CloudFormation template was deleted.

---

## Answer Key & Detailed Explanations

### Question 1: Route 53 Health Checks on Internal / Private Resources

**Answer:** B  
**Explanation:** 
Route 53 health checker nodes run on the public internet and cannot reach private RFC 1918 IP addresses in a VPC. To enable health-checked failover in a **Private Hosted Zone**, the standard pattern is **Calculated / CloudWatch-Based Health Checks**: instances or internal load balancers publish health metrics to CloudWatch, a CloudWatch alarm triggers on failure, and a Route 53 health check monitors the CloudWatch alarm state to execute internal DNS failover.

---

### Question 2: Geolocation vs. Geoproximity Routing

**Answer:** B  
**Explanation:** 
- **Geolocation Routing:** Routes traffic based strictly on the user's geographic location (e.g. country, continent, or US state). It does not factor in resource proximity and does not support bias adjustments.
- **Geoproximity Routing:** Routes traffic based on the physical distance between the user and AWS resources. By applying a **bias** (from -99 to +99), operators can visually expand or shrink the geographic region served by a specific datacenter.

---

### Question 3: Multi-Value Answer Routing vs. Simple Round-Robin

**Answer:** B  
**Explanation:** 
- **Simple Routing** with multiple IPs returns all IP addresses to the client, but it does **not** support Route 53 Health Checks. If a server dies, Simple Routing still returns its dead IP.
- **Multi-Value Answer Routing** returns up to 8 healthy IP addresses selected randomly from the pool and actively checks the health of each record, automatically dropping unhealthy IPs from DNS responses.

---

### Question 4: Route 53 Application Recovery Controller (ARC) Zonal Shift

**Answer:** B  
**Explanation:** 
**Route 53 ARC Zonal Shift** allows operators to quickly shift traffic away from an impaired Availability Zone for supported resources (Application Load Balancers and Network Load Balancers with cross-zone load balancing disabled). Activating a zonal shift redirects traffic to healthy AZs in under a minute, restoring service while AWS investigates underlying zonal impairments.

---

### Question 5: Route 53 Invert Health Check Status

**Answer:** B  
**Explanation:** 
Route 53 Health Checks include an **Invert health check status** toggle. When enabled, a passing health check is treated as failed, and a failing check is treated as healthy. This allows operators to test automated failover logic cleanly without generating actual outages or terminating production resources.

---

### Question 6: Route 53 Resolver for Hybrid Cloud DNS Resolution

**Answer:** B  
**Explanation:** 
**Route 53 Resolver** bridges on-premises DNS and AWS VPC DNS:
- **Inbound Endpoints:** Provide VPC IP addresses that on-premises DNS resolvers forward queries to, allowing on-premises hosts to resolve AWS Private Hosted Zones.
- **Outbound Endpoints & Rules:** Allow VPC resolvers to forward queries for on-premises domains to on-premises DNS servers over Direct Connect or VPN.

---


### Question 7: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap
**Answer:** B
**Explanation:** Default AWS-managed keys like `aws/s3` cannot have their key policies modified and CANNOT be accessed across AWS accounts. Cross-account CodePipeline deployments strictly require an S3 bucket encrypted with a Customer Managed Key (CMK) whose key policy explicitly grants cross-account decrypt rights.

---

### Question 8: CodeDeploy In-Place ApplicationStop Lifecycle Failure
**Answer:** B
**Explanation:** CodeDeploy executes the `ApplicationStop` hook from the PREVIOUS revision's AppSpec file. If that script fails, all future deployments fail at that step unless you specify `--ignore-application-stop-failures`.

---

### Question 9: CloudFormation Custom Resource 60-Minute Stack Freeze
**Answer:** B
**Explanation:** CloudFormation Custom Resources expect a signed JSON payload uploaded to the pre-signed S3 `ResponseURL`. If the Lambda function crashes or exits without sending this signal, CloudFormation waits 1 hour before timing out.

---

