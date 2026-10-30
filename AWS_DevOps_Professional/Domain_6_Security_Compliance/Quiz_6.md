# Domain 6 (Security and Compliance) — Quiz 6: Workload, Perimeter & Container Security

**Topic:** Amazon Inspector v2, Macie, AWS WAF Bot Control, AWS Shield Advanced & EKS Pod Identity  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: Continuous Automated Vulnerability Management via Amazon Inspector v2

A enterprise security operations team needs a continuous vulnerability management solution across 5,000 Amazon EC2 instances, 200 Amazon ECR repositories, and 1,500 AWS Lambda functions. The solution must automatically discover newly published Common Vulnerabilities and Exposures (CVEs) and evaluate workloads in real time whenever a CVE is released, without scheduling manual scan windows or overloading production instance CPUs.

**Which AWS service and architecture satisfies these requirements natively?**
- A. Enable **Amazon Inspector v2** organization-wide from the delegated administrator account: Continuously and automatically assesses EC2 instances (using SSM Agent and hybrid agentless EBS volume scanning), ECR container images (continuous scanning on push and when new CVEs are published), and AWS Lambda functions (scanning application code and layer dependencies).
- B. Deploy an open-source vulnerability scanner on EC2 instances running daily cron jobs that execute network port sweeps.
- C. Configure an AWS Systems Manager Patch Manager maintenance window that runs every 4 hours.
- D. Deploy Amazon GuardDuty with S3 Protection and EKS Audit Log Protection enabled.

---

### Question 2: Zero-Trust On-Premises Authentication via IAM Roles Anywhere

An enterprise operates a hybrid cloud architecture with legacy financial applications hosted on physical servers in an on-premises data center. The on-premises applications need to upload sensitive transaction batches to an Amazon S3 bucket. Corporate security strictly prohibits provisioning long-lived IAM user access keys (`AKIA...`) due to the risk of credential leakage.

**Which AWS IAM feature enables on-premises workloads to obtain short-lived, temporary AWS STS credentials using existing corporate PKI?**
- A. Deploy **AWS IAM Roles Anywhere**: Establish a trust anchor between AWS and the corporate Certificate Authority (CA) using an X.509 certificate, define an IAM role with an IAM trust policy permitting `rolesanywhere.amazonaws.com`, and have the on-premises application exchange its local X.509 certificate for temporary STS credentials via the IAM Roles Anywhere credential helper.
- B. Establish an AWS Site-to-Site VPN and hardcode IAM user credentials in a local configuration file behind a firewall.
- C. Create an IAM user with programmatic access and rotate access keys every 24 hours using an automated Python script.
- D. Deploy an AWS Storage Gateway File Gateway on-premises and grant the storage gateway full S3 access.

---

### Question 3: Modern Kubernetes IAM Governance: EKS Pod Identity vs IRSA

A DevOps engineering team manages a large-scale Amazon EKS cluster with 300 microservices across 50 namespaces. The team previously used IAM Roles for Service Accounts (IRSA), but found that managing individual OpenID Connect (OIDC) identity provider endpoints, complex JSON trust policies, and IAM role annotations across multiple staging and production clusters introduced severe operational complexity.

**Which modern AWS EKS feature simplifies IAM credential vending to Kubernetes pods without managing cluster OIDC providers?**
- A. EKS Pod Identity: Deploy the EKS Pod Identity Agent DaemonSet on the cluster, configure direct associations between Kubernetes ServiceAccounts and IAM Roles via the EKS console or API, and allow pods to receive temporary STS credentials automatically with built-in ABAC session tags without OIDC identity provider management.
- B. Grant the EKS Node Instance IAM Role full administrative access to all AWS resources needed by all pods.
- C. Store AWS credentials in Kubernetes Secrets and mount them as environment variables inside pod containers.
- D. Configure an intermediate AWS Lambda proxy that authenticates Kubernetes tokens and returns STS credentials over HTTP.

---

### Question 4: Discovering and Classifying Sensitive Data at Scale via Amazon Macie

A healthcare company operates over 500 Amazon S3 buckets containing petabytes of diagnostic data and documents. Compliance regulations (HIPAA/HITECH) mandate that the company must automatically discover, classify, and inventory all buckets containing unencrypted Protected Health Information (PHI) or Personally Identifiable Information (PII), generating automated alerts if sensitive data is exposed publicly.

**Which AWS architecture fulfills this data discovery and compliance audit requirement?**
- A. Enable **Amazon Macie** organization-wide from the delegated administrator account: Macie continuously evaluates S3 bucket posture (public accessibility, encryption status), runs sensitive data discovery jobs using managed and custom data identifiers (e.g., medical IDs, SSNs), and emits findings to Security Hub and Amazon EventBridge.
- B. Write an AWS Glue crawler that scans S3 objects and writes schema definitions to the Glue Data Catalog.
- C. Deploy an Amazon Athena query script that parses S3 bucket access logs using regular expressions.
- D. Configure S3 Event Notifications to trigger an AWS Lambda function on `s3:ObjectCreated:*` to inspect object byte contents.

---

### Question 5: Sophisticated Automated Attack Mitigation via AWS WAF Bot Control & ATP

An e-commerce ticketing platform experiences severe credential stuffing and automated scraping attacks against its `/login` and `/checkout` endpoints during high-demand concert ticket releases. The attackers use distributed proxy networks that rotate residential IP addresses, easily bypassing simple IP rate-limiting rules.

**Which AWS WAF feature combination mitigates these distributed, automated attacks?**
- A. Associate **AWS WAF Bot Control** and **AWS WAF Account Takeover Prevention (ATP)** managed rule groups with the WebACL: Enforces client browser verification using silent JavaScript challenges, blocks automated headless browsers, evaluates compromised credential databases, and enforces CAPTCHA challenges on anomalous client behaviors.
- B. Configure a standard AWS WAF Rate-Based Rule with a limit of 100 requests per 5 minutes and an action of `Block`.
- C. Deploy an Amazon CloudFront distribution with a Geo-Restriction policy blocking traffic from non-US countries.
- D. Increase the Application Load Balancer target group idle timeout to 300 seconds.

---

### Question 6: Enterprise DDoS Protection and Cost Spike Protection via AWS Shield Advanced

An enterprise financial exchange operates critical trading APIs on AWS. The platform requires comprehensive Layer 3, Layer 4, and Layer 7 distributed denial-of-service (DDoS) protection, direct 24/7 engagement with specialized AWS security engineers during active attacks, and financial indemnity against sudden Auto Scaling bill spikes caused by malicious traffic floods.

**Which AWS service offering provides these comprehensive capabilities?**
- A. Subscribe to **AWS Shield Advanced**: Protects ALBs, CloudFront, Route 53, and Elastic IPs; grants 24/7 access to the **AWS Shield Response Team (SRT)** for manual rule customization; integrates with Route 53 health checks; and provides **DDoS Cost Protection** to refund operational scaling costs incurred during attacks.
- B. Rely on default **AWS Shield Standard**, which provides automatic protection against all Layer 7 application attacks free of charge.
- C. Deploy open-source HAProxy instances on Amazon EC2 in front of the Application Load Balancers.
- D. Configure AWS Network Firewall with stateful rules configured to drop all incoming UDP packets.

---

### Question 7: Enforcing Container Supply Chain Integrity via AWS Signer and ECR

A government defense contractor requires strict software supply chain security for all container images deployed to Amazon EKS. Corporate policy dictates that only container images that have been digitally signed by an authorized corporate CI/CD signing profile can be pulled and executed on Kubernetes worker nodes. Unsigned or tampered images must be rejected at admission time.

**Which container security architecture enforces this cryptographic supply chain verification?**
- A. Sign container images in the CI/CD pipeline using **AWS Signer** container signing profiles; store signatures in Amazon ECR alongside image manifests; deploy a Kubernetes **Admission Controller** (such as Kyverno or Gatekeeper) on EKS that verifies the AWS Signer cryptographic signature against the approved signing profile before permitting pod scheduling.
- B. Calculate the SHA-256 hash of the Dockerfile and store it in an AWS Systems Manager Parameter Store parameter.
- C. Configure Amazon ECR image tag immutability and apply an S3 Bucket Policy to the backing registry bucket.
- D. Configure an AWS Config rule that terminates EKS worker nodes if unauthorized container tags are detected.

---

### Question 8: Advanced Perimeter Inspection via AWS Network Firewall Suricata Rules

An enterprise mandates that all outbound internet traffic from private VPC workloads must undergo deep packet inspection (DPI) to prevent data exfiltration. The inspection firewall must inspect TLS Server Name Indication (SNI) headers to enforce an approved domain whitelist (e.g., `*.amazon.com`, `*.github.com`), drop unauthorized outbound TCP connections, and detect malware command-and-control signatures using open-source Suricata rule syntax.

**Which AWS managed networking service provides this stateful perimeter defense?**
- A. Deploy **AWS Network Firewall** in dedicated firewall subnets within the VPC transit architecture: Configure a Firewall Policy containing **Stateful Rule Groups** utilizing standard **Suricata-compatible IPS rules** and **Domain List inspection rules** to filter outbound HTTP/HTTPS traffic by domain name.
- B. Configure standard Amazon VPC Security Groups with outbound egress rules referencing domain names.
- C. Attach AWS WAF WebACLs directly to the VPC Internet Gateway.
- D. Deploy an Amazon Route 53 Resolver Outbound Endpoint with forwarding rules to a public DNS provider.

---

### Question 9: Automated Perimeter Defense via GuardDuty, EventBridge & WAF IP Sets

An enterprise security operations center needs an automated perimeter defense mechanism. When Amazon GuardDuty generates high-severity findings indicating that an external IP address is performing port scanning or brute-force attacks against internet-facing EC2 instances, the external IP address must be blocked automatically across all perimeter Application Load Balancers within 30 seconds.

**Which architecture satisfies this automated real-time perimeter defense?**
- A. Create an **Amazon EventBridge Rule** matching GuardDuty findings with severity $\ge 7.0$; target an **AWS Lambda function** that parses the offending external IP address from the finding payload and updates an **AWS WAF IP Set** associated with a global `BLOCK` rule on the perimeter WebACL.
- B. Configure an Amazon SNS topic subscribed to GuardDuty that emails network administrators instructions to update security groups.
- C. Deploy an AWS Systems Manager Automation runbook that adds the IP address to the EC2 instance's `/etc/hosts.deny` file.
- D. Configure Amazon Inspector to automatically delete the internet-facing Elastic IP address attached to the load balancer.

---

## Answer Key & Explanations

### Question 1: Continuous Automated Vulnerability Management via Amazon Inspector v2
**Answer:** A  
**Explanation:** 
**Amazon Inspector v2** is AWS's modern, continuous vulnerability management service:
- Unlike legacy periodic scanning tools, Inspector v2 operates **continuously and automatically**:
  - **Amazon EC2:** Uses the AWS Systems Manager (SSM) Agent or hybrid agentless EBS volume scanning.
  - **Amazon ECR:** Provides continuous scanning of container images on push and whenever new CVEs are published to the National Vulnerability Database (NVD).
  - **AWS Lambda:** Automatically scans Lambda function application code and runtime layer dependencies for code and package vulnerabilities.
- As soon as a new CVE is disclosed globally, Inspector v2 automatically correlates it against previously inventoried software packages across all accounts in the AWS Organization, generating contextualized findings with zero manual scan triggers.

---

### Question 2: Zero-Trust On-Premises Authentication via IAM Roles Anywhere
**Answer:** A  
**Explanation:** 
**AWS IAM Roles Anywhere** extends AWS Identity and Access Management (IAM) to workloads outside of AWS (such as on-premises servers, containers, or other cloud providers):
- It eliminates the high-risk anti-pattern of creating IAM users with long-lived access keys (`AKIA...`) for external servers.
- You establish a **Trust Anchor** between AWS IAM and your corporate Public Key Infrastructure (PKI) Certificate Authority (CA) using an X.509 certificate.
- The external workload presents its locally signed X.509 certificate to IAM Roles Anywhere via the credential helper tool.
- IAM Roles Anywhere validates the certificate and issues **temporary, short-lived AWS STS credentials** (valid from 15 minutes to 12 hours) corresponding to an authorized IAM role, bringing AWS native security and credential rotation to on-premises workloads.

---

### Question 3: Modern Kubernetes IAM Governance: EKS Pod Identity vs IRSA
**Answer:** A  
**Explanation:** 
**Amazon EKS Pod Identity** is the next-generation credential vending mechanism for Kubernetes pods:
- **Limitations of legacy IRSA:** Required configuring an IAM OIDC identity provider per cluster, managing complex trust policies with string conditions, and annotating Kubernetes ServiceAccounts.
- **How EKS Pod Identity works:** You install the EKS Pod Identity Agent DaemonSet on the cluster. You define direct associations between a Kubernetes ServiceAccount and an IAM Role via the EKS API.
- The agent intercepts STS credential requests from the AWS SDK inside pod containers and vends temporary credentials directly.
- It supports dynamic session tags (passing cluster name, namespace, and service account name to IAM policy conditions) and allows reusing the **exact same IAM Role across multiple clusters** without modifying trust policies.

---

### Question 4: Discovering and Classifying Sensitive Data at Scale via Amazon Macie
**Answer:** A  
**Explanation:** 
**Amazon Macie** is a fully managed data security and privacy service powered by machine learning and pattern matching:
- Continuously evaluates the security posture of all Amazon S3 buckets across an AWS Organization, identifying unencrypted buckets, publicly accessible buckets, and buckets shared outside the organization.
- Executes **Sensitive Data Discovery Jobs** that scan object payloads for sensitive data:
  - Managed Data Identifiers: PII (SSNs, passport numbers, driver's licenses), financial records (credit card numbers, bank accounts), and health data (medical record numbers).
  - Custom Data Identifiers: Define custom regex patterns for proprietary organizational data.
- Emits detailed findings to AWS Security Hub and Amazon EventBridge for automated compliance reporting and remediation.

---

### Question 5: Sophisticated Automated Attack Mitigation via AWS WAF Bot Control & ATP
**Answer:** A  
**Explanation:** 
Modern bot attacks rotate through thousands of clean residential IP addresses, making standard IP rate-limiting rules ineffective.
- **AWS WAF Bot Control:** Identifies and blocks automated bot traffic by inspecting client browser behavior, HTTP headers, and executing silent JavaScript challenges.
- **AWS WAF Account Takeover Prevention (ATP):** Specifically protects login endpoints against credential stuffing and brute force attacks. It inspects login attempts, cross-references credentials against global databases of stolen credentials, tracks login failure rates, and presents silent or interactive CAPTCHA challenges to suspicious clients.

---

### Question 6: Enterprise DDoS Protection and Cost Spike Protection via AWS Shield Advanced
**Answer:** A  
**Explanation:** 
While **AWS Shield Standard** provides automatic, free protection against common Layer 3 and Layer 4 infrastructure attacks:
**AWS Shield Advanced** provides enterprise-tier protection and business guarantees:
- Enhanced protection for CloudFront, Route 53, ALBs, and Elastic IPs against sophisticated L3, L4, and L7 DDoS attacks.
- Direct, 24/7 engagement with the **AWS Shield Response Team (SRT)** to write custom WAF rules during active attacks.
- **DDoS Cost Protection:** A financial safeguard that reimburses customers for unexpected compute, load balancer, or data transfer scaling charges caused by malicious traffic spikes during a verified DDoS attack.

---

### Question 7: Enforcing Container Supply Chain Integrity via AWS Signer and ECR
**Answer:** A  
**Explanation:** 
Securing the container supply chain requires cryptographic image signing and admission enforcement:
- **AWS Signer:** A fully managed code signing service that integrates with container registries (Amazon ECR) using the Notary v2 / Notation specification.
- In the CI/CD pipeline (e.g., CodeBuild), images are signed using an AWS Signer signing profile, and digital signatures are pushed alongside image layers to ECR.
- On Amazon EKS, a Kubernetes **Admission Controller** (such as Kyverno or OPA Gatekeeper) intercepts pod creation requests (`ValidatingWebhookConfiguration`).
- The admission controller verifies the cryptographic signature against the authorized AWS Signer certificate. If the image is unsigned or has been tampered with, the pod is rejected and blocked from running.

---

### Question 8: Advanced Perimeter Inspection via AWS Network Firewall Suricata Rules
**Answer:** A  
**Explanation:** 
**AWS Network Firewall** is a stateful, managed network firewall and intrusion detection/prevention system (IDS/IPS) for VPCs:
- Deployed across dedicated subnets in the VPC routing path (e.g., between workloads and the Internet Gateway / Transit Gateway).
- Supports stateful inspection using industry-standard **Suricata rule syntax**, enabling deep packet inspection (DPI) to detect exploit attempts and malicious protocol anomalies.
- **Domain List Filtering:** Evaluates TLS handshake Server Name Indication (SNI) headers and HTTP Host headers to enforce strict domain whitelists for outbound traffic, preventing malware command-and-control communication and data exfiltration.

---

### Question 9: Automated Perimeter Defense via GuardDuty, EventBridge & WAF IP Sets
**Answer:** A  
**Explanation:** 
Automated closed-loop perimeter defense combines threat intelligence with perimeter enforcement:
- **Amazon GuardDuty** continuously monitors VPC Flow Logs, DNS logs, and CloudTrail, detecting external threat actors (e.g., SSH brute force, port scanning, known malicious IPs).
- When a high-severity finding occurs, GuardDuty emits an event to **Amazon EventBridge**.
- An EventBridge rule triggers an **AWS Lambda function** (or SSM Automation runbook).
- The Lambda function extracts the external attacker IP address from the finding details and calls `wafv2:UpdateIPSet`, adding the IP to an AWS WAF IP Set configured with a `BLOCK` action on the perimeter WebACL.
- This immediately neutralizes the threat across all internet-facing Application Load Balancers in under 30 seconds.
