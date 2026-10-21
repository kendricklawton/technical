# AWS Certified Solutions Architect - Professional (SAP-C02) Practice Exam 2

**Duration:** 180 Minutes (75 Complex Scenario Questions)  
**Passing Score:** 750 / 1000  

---

### Question 1 (Domain 1: Organizational Complexity)
An enterprise operates a multi-account organization with AWS Organizations. The security team mandates that no member account root user or IAM administrator can disable AWS CloudTrail, AWS Config, or Amazon GuardDuty in any region. Furthermore, developers must only deploy resources into `us-east-1` and `eu-west-1`.

**Which combination of steps achieves this requirement with the LEAST operational overhead?**
- A. Deploy IAM Permission Boundaries across all developer roles in member accounts.
- B. Apply a Service Control Policy (SCP) at the Root level denying `cloudtrail:StopLogging`, `config:DeleteConfigRule`, and denying all actions where `aws:RequestedRegion` is not `us-east-1` or `eu-west-1`, with appropriate `NotAction` exemptions for global services like IAM and Route 53.
- C. Write an AWS Config rule backed by Lambda in every account that evaluates compliance every 5 minutes.
- D. Enable AWS Systems Manager State Manager across all instances to verify logging state.

**Answer:** B  
**Explanation:** SCPs set the maximum permission boundary for an entire AWS Organization or OU. An explicit Deny in an SCP overrides all local IAM permissions, including `AdministratorAccess`. The `NotAction` block allows global services (IAM, Route 53, STS, CloudFront) to function while restricting regional API calls.

---

### Question 2 (Domain 2: Design for New Solutions)
A video rendering SaaS platform processes multi-terabyte uncompressed video uploads from global users. Currently, uploads time out when transiting public internet peering links to the centralized S3 bucket in `us-east-1`.

**Which architecture provides the lowest latency and highest upload throughput?**
- A. Direct users to upload through an Application Load Balancer in each region.
- B. Enable Amazon S3 Transfer Acceleration on the bucket and have the application generate pre-signed Amazon S3 URLs utilizing the S3 Transfer Acceleration endpoint (`<bucket-name>.s3-accelerate.amazonaws.com`).
- C. Deploy an EC2 proxy fleet in 10 global regions to ingest uploads.
- D. Ship AWS Snowcone devices to all global users.

**Answer:** B  
**Explanation:** Amazon S3 Transfer Acceleration utilizes Amazon CloudFront's globally distributed Edge Locations. Traffic enters the nearest edge location and traverses AWS's optimized, private backbone network directly to the S3 bucket in `us-east-1`.
