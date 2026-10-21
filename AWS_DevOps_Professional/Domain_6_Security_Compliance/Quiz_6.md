# Domain 6 (Security and Compliance) — Quiz 6: Workload, Perimeter & Container Security

**Topic:** Amazon Inspector v2, Macie, AWS WAF Bot Control & AWS Shield Advanced  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: Amazon Inspector v2 Automated Continuous Scanning

How does Amazon Inspector v2 differ from legacy Inspector v1 regarding agent deployment and scan scheduling for EC2 instances and container images?
- A. Inspector v2 requires installing a dedicated standalone Inspector agent on all EC2 instances and running manual scheduled scan jobs.
- B. Inspector v2 is **agentless or uses the AWS Systems Manager (SSM) agent** for EC2 scanning, continuously and automatically scans EC2 instances and container images in Amazon ECR upon push and when new CVEs are published, and supports scanning AWS Lambda function code and dependencies without manual schedules.
- C. Inspector v2 only scans static HTML files on S3.
- D. Inspector v2 requires instances to have public IP addresses.

---

### Question 2: AWS WAF Rate-Based Rules for Layer 7 DDoS Mitigation

An e-commerce API endpoint `/api/v1/login` experiences credential-stuffing attacks from distributed IP addresses attempting hundreds of login requests per minute.

**Which AWS WAF rule configuration protects this specific endpoint against brute-force / DDoS attacks?**
- A. A standard AWS WAF IP Set blocking all traffic from private IP ranges.
- B. A **Rate-Based Rule** evaluated on the URI path `/api/v1/login`, configured with an aggregation window (e.g., 100 requests per 5 minutes per IP address) and an action set to **Block** or **CAPTCHA**.
- C. Disabling CloudFront distribution caching.
- D. A security group rule blocking TCP port 80.

---

### Question 3: AWS Shield Advanced DDoS Response Team (DRT) Support

An enterprise runs mission-critical workloads behind AWS CloudFront and Application Load Balancers. The company subscribes to **AWS Shield Advanced**. During a massive multi-vector Layer 7 DDoS attack, the operations team needs AWS DDoS experts to write custom WAF rules on their behalf to mitigate the attack.

**Which feature of AWS Shield Advanced grants AWS experts permission to manage WAF rules during an attack?**
- A. Giving root user credentials to AWS Support.
- B. Enabling **Shield Advanced Proactive Engagement** and configuring **DDoS Response Team (DRT / SRT) Access**, granting the Shield Response Team IAM permissions to inspect WAF logs and create custom WAF mitigation rules directly on the customer's behalf during an active incident.
- C. Submitting a standard general inquiry support ticket.
- D. AWS Shield Advanced does not include human support.

---

### Question 4: Amazon Macie Sensitive Data Discovery & Automated Remediation

An organization stores millions of customer documents in Amazon S3 buckets. Corporate compliance mandates that no unencrypted buckets contain Personally Identifiable Information (PII) such as passport numbers, credit cards, or national IDs.

**How can the DevOps team automate the discovery and notification of sensitive data in S3?**
- A. Write a Python script to download all S3 objects locally and run grep.
- B. Enable **Amazon Macie**, configure automated sensitive data discovery jobs, and route Macie findings via **Amazon EventBridge** to trigger automated remediation (such as removing public access or applying KMS encryption) and alert security teams via SNS.
- C. Use Amazon Athena to query S3 bucket metadata.
- D. Use S3 Storage Lens.

---

### Question 5: AWS WAF Bot Control Managed Rule Group

A media streaming company experiences automated scrapers that bypass basic User-Agent header filtering to scrape proprietary content, consuming server bandwidth and corrupting web analytics.

**Which AWS WAF capability provides sophisticated protection against automated bots, search engine crawlers, and scrapers?**
- A. AWS Network Firewall stateless rules.
- B. The **AWS WAF Bot Control Managed Rule Group**, which uses machine learning, browser fingerprinting, and client-side challenge tokens (CAPTCHA / Silent Challenge) to identify and block malicious bots while allowing verified search engines.
- C. Blocking port 443 on the VPC route table.
- D. Setting the S3 bucket ACL to private.

---

### Question 6: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap

A centralized CI/CD pipeline in Account 111111111111 deploys an ECS service in Account 222222222222. The deployment stage fails with: 'The ciphertext refers to a customer master key that does not exist, does not allow access, or is invalid.' The S3 bucket is encrypted with the default AWS-managed key 'aws/s3'. How should the engineer resolve this?

- A. Update the S3 bucket policy to grant read access to Account 222222222222.
- B. Reconfigure the S3 artifact bucket to use a Customer Managed Key (CMK) and grant `kms:Decrypt` and `kms:DescribeKey` permissions to Account 222222222222's execution role in the KMS Key Policy.
- C. Create a VPC Endpoint for S3 in Account 222222222222.
- D. Enable S3 Object Ownership with Bucket Owner Preferred.

---

### Question 7: CodeDeploy In-Place ApplicationStop Lifecycle Failure

During an in-place deployment to an EC2 Auto Scaling Group, the deployment fails at the ApplicationStop lifecycle event hook because a previous buggy script had been deployed in the older revision. How can the engineer force CodeDeploy to deploy the new fixed revision without terminating the instances?

- A. Delete the CodeDeploy agent from all EC2 instances.
- B. Trigger a new deployment via the AWS CLI with the `--ignore-application-stop-failures` flag enabled.
- C. Manually stop the EC2 instances in the AWS Management Console.
- D. Reboot the instances and rerun the same deployment ID.

---

### Question 8: CloudFormation Custom Resource 60-Minute Stack Freeze

A CloudFormation stack creation hangs for exactly 1 hour in `CREATE_IN_PROGRESS` on a Lambda-backed custom resource before failing and rolling back. What was the root cause?

- A. The Lambda function exceeded its maximum execution timeout of 15 minutes.
- B. An unhandled exception crashed the Lambda function before it could send an HTTP PUT response with status SUCCESS or FAILED to the pre-signed Amazon S3 ResponseURL.
- C. CloudFormation lost IAM permissions to invoke Lambda.
- D. The S3 bucket holding the CloudFormation template was deleted.

---

## Answer Key & Detailed Explanations

### Question 1: Amazon Inspector v2 Automated Continuous Scanning

**Answer:** B  
**Explanation:** 
**Amazon Inspector v2** is a completely redesigned vulnerability management service:
- It eliminates the need for a standalone legacy Inspector agent, using the standard AWS Systems Manager (SSM) agent already present on Amazon Linux/Ubuntu/Windows AMIs.
- It scans continuously and automatically: whenever an image is pushed to Amazon ECR, whenever an EC2 instance launches, and automatically rescans when new Common Vulnerabilities and Exposures (CVEs) are added to the database.
- It also scans AWS Lambda functions and layer dependencies.

---

### Question 2: AWS WAF Rate-Based Rules for Layer 7 DDoS Mitigation

**Answer:** B  
**Explanation:** 
AWS WAF **Rate-Based Rules** track the number of requests originating from each client IP address over a sliding time window (1, 2, 5, or 10 minutes). When an individual IP exceeds the configured request threshold matching specific scope criteria (e.g., `URIPath startsWith /api/v1/login`), WAF automatically applies a mitigation action (Block, CAPTCHA, or Challenge) against that specific offending IP until its request rate drops below the threshold.

---

### Question 3: AWS Shield Advanced DDoS Response Team (DRT) Support

**Answer:** B  
**Explanation:** 
AWS Shield Advanced includes 24/7 access to the **AWS Shield Response Team (SRT)**. By configuring SRT Access with an IAM role and associating CloudWatch health checks with resources, customers can enable **Proactive Engagement**. When a DDoS attack degrades resource health checks, the SRT proactively contacts the customer and can write, test, and deploy customized AWS WAF rules directly to mitigate zero-day Layer 7 attacks.

---

### Question 4: Amazon Macie Sensitive Data Discovery & Automated Remediation

**Answer:** B  
**Explanation:** 
**Amazon Macie** is a fully managed data security and privacy service that uses machine learning and pattern matching to discover, monitor, and protect sensitive data in Amazon S3. Macie automatically scans objects for managed sensitive data identifiers (credit cards, PII, financial credentials, API keys). Findings are published to Amazon EventBridge, enabling automated event-driven remediation workflows.

---

### Question 5: AWS WAF Bot Control Managed Rule Group

**Answer:** B  
**Explanation:** 
The **AWS WAF Bot Control Managed Rule Group** provides comprehensive visibility and mitigation against common and evasive bot traffic. It inspects incoming HTTP request headers, browser fingerprints, and behavioral characteristics. It can detect scraping tools, automated frameworks (Selenium, Puppeteer), credential stuffers, while natively allowing verified bots (such as Googlebot or Bingbot) and enforcing CAPTCHA or silent JavaScript challenges against suspicious automated clients.

---


### Question 6: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap
**Answer:** B
**Explanation:** Default AWS-managed keys like `aws/s3` cannot have their key policies modified and CANNOT be accessed across AWS accounts. Cross-account CodePipeline deployments strictly require an S3 bucket encrypted with a Customer Managed Key (CMK) whose key policy explicitly grants cross-account decrypt rights.

---

### Question 7: CodeDeploy In-Place ApplicationStop Lifecycle Failure
**Answer:** B
**Explanation:** CodeDeploy executes the `ApplicationStop` hook from the PREVIOUS revision's AppSpec file. If that script fails, all future deployments fail at that step unless you specify `--ignore-application-stop-failures`.

---

### Question 8: CloudFormation Custom Resource 60-Minute Stack Freeze
**Answer:** B
**Explanation:** CloudFormation Custom Resources expect a signed JSON payload uploaded to the pre-signed S3 `ResponseURL`. If the Lambda function crashes or exits without sending this signal, CloudFormation waits 1 hour before timing out.

---

