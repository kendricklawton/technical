# AWS DevOps Engineer - Professional (DOP-C02) Practice Exam 6: Organizational Governance & Security Guardrails

**Difficulty:** Advanced — calibrated to roughly twice the difficulty of the real DOP-C02 exam  
**Format:** 15 multi-paragraph scenario questions · single answer · four options (A–D)  
**Time Allowed:** 75 minutes (≈5 minutes per question; the real exam allows ≈2.4)  
**Passing Target:** 12 / 15 (80%)  
**Domains:** D1 ×3 · D2 ×2 · D3 ×3 · D4 ×2 · D5 ×2 · D6 ×3

**How these questions work:** each question has exactly one governing business constraint, written in CAPITALS in the question line. All four options are working architectures that meet the functional requirements; the three distractors fail *only* the constraint. Answer all 15 before opening the answer key at the end.

---

## Question 1 (Domain 6: Security and Compliance)

A company with 300 AWS accounts must enforce a data perimeter: S3 buckets, KMS keys, SQS queues, and Secrets Manager secrets may be accessed only by principals in the company's organization, except when an AWS service acts on the company's behalf. The control must apply to existing and future resources, and account administrators must not be able to override it.

Some teams have bucket policies that grant access to vendor accounts outside the organization, and those grants must be blocked. The company already uses SCPs from the management account, and all workload accounts are in OUs under the root.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Use StackSets to apply standard resource policies to existing buckets, keys, queues, and secrets, and deploy AWS Config custom rules with automatic remediation that rewrites the policies of new or changed resources.
- B. Create an EventBridge rule in every account on CloudTrail events that create or change these resources, invoking a Lambda function that adds a deny statement for principals outside the organization to each resource policy.
- C. Allow these resource types to be created only through AWS Service Catalog products with approved policies, add an SCP that denies direct creation outside the launch roles, and update existing resources' policies with a StackSet.
- D. Attach a resource control policy at the root that denies access to these services' resources when the principal's `aws:PrincipalOrgID` isn't the company's, unless `aws:PrincipalIsAWSService` is true.

---

## Question 2 (Domain 1: SDLC Automation)

A company's GitHub repository `acme/payments-api` builds container images with GitHub Actions and pushes them to the Amazon ECR repository `payments-api` in a production account. Today the workflow uses an IAM user's long-term access keys stored as a GitHub secret. The `acme` organization has 400 other repositories, and pushes to production must come only from this repository's `production` environment, which requires reviewer approval.

An IAM OIDC identity provider for `token.actions.githubusercontent.com` with the audience `sts.amazonaws.com` already exists in the production account.

**Which solution meets these requirements with the LEAST privilege?**

- A. Create a role whose trust policy requires the `sts.amazonaws.com` audience and a `:sub` claim equal to `repo:acme/payments-api:environment:production`, with a policy allowing image push actions only on the `payments-api` repository.
- B. Create a role whose trust policy requires the `sts.amazonaws.com` audience and a `:sub` claim that matches `repo:acme/*`, with a policy allowing image push actions only on the `payments-api` repository.
- C. Create a role whose trust policy requires the `sts.amazonaws.com` audience and a `:sub` claim that matches `repo:acme/payments-api:*`, with a policy allowing image push actions only on the `payments-api` repository.
- D. Create a role whose trust policy requires the `sts.amazonaws.com` audience and a `:sub` claim equal to `repo:acme/payments-api:environment:production`, with the `AmazonEC2ContainerRegistryPowerUser` managed policy attached.

---

## Question 3 (Domain 3: Resilient Cloud Solutions)

A company with 120 accounts must keep immutable copies of EC2, EBS, RDS, and DynamoDB backups that no one in the source accounts — including administrators — can delete or alter for 35 days, and must be able to restore them into an isolated recovery account during a ransomware event. Backup schedules must apply automatically to every account, including new ones.

AWS Backup cross-account management is enabled with a delegated administrator, and a recovery account already exists in a separate OU. The company doesn't want to manage KMS keys for the copies.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Use an organization backup policy that backs up the resources by tag and copies each recovery point to a logically air-gapped vault, which is shared with the recovery account through AWS RAM for restores.
- B. Use an organization backup policy that copies recovery points to a standard vault in a central backup account locked with Vault Lock in compliance mode, and share a customer managed KMS key from that account with every source account.
- C. Deploy backup plans and vaults locked with Vault Lock to every account through StackSets, and add a cross-account copy rule that sends recovery points to a locked vault in the recovery account.
- D. Copy snapshots to the recovery account with a Lambda function triggered by backup job completion events, and protect the copies there with an SCP that denies deletion except by a break-glass role.

---

## Question 4 (Domain 2: Configuration Management and IaC)

A company wants every account and Region in its organization — including accounts and Regions enabled later — to block public sharing of AMIs, block public access to EBS snapshots, and default new instances to IMDSv2. Account administrators must not be able to change these settings, and member accounts should see a clear message explaining why an action is blocked.

The company manages 210 accounts from a management account with SCPs and uses 9 Regions. Some legacy instances still use IMDSv1 and are scheduled for replacement next year.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Use a service-managed StackSet to set image block public access, snapshot block public access, and instance metadata defaults in every account and Region, and add an SCP that denies the API actions that change those settings.
- B. Create a Lambda function in the management account, triggered by new-account events and a weekly schedule, that assumes a role in every account and Region to set the three settings, and add an SCP that denies changes to them.
- C. Add SCPs that deny `ec2:RunInstances` unless `ec2:MetadataHttpTokens` is `required`, deny making AMIs or snapshots public through their attribute-modification actions, and deny changes to these protections.
- D. Attach a declarative policy for EC2 at the root that sets image block public access, snapshot block public access, and IMDSv2 instance metadata defaults, with a custom error message for member accounts.

---

## Question 5 (Domain 4: Monitoring and Logging)

A company with 180 accounts in 4 Regions wants organization-wide security posture visibility. Production OUs must run the CIS AWS Foundations Benchmark and AWS Foundational Security Best Practices standards; sandbox OUs must run only Foundational Security Best Practices, with several controls disabled. New accounts must be configured automatically, and findings from all Regions must be visible in one Region.

AWS Config is enabled everywhere, and the security account is already the delegated administrator for AWS Security Hub CSPM.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Use StackSets to enable Security Hub CSPM, the standards, and per-OU control settings in every account and Region, and configure finding aggregation to a home Region in the delegated administrator account.
- B. Deploy organization conformance packs per OU that reproduce the standards' checks with AWS Config rules, and build compliance dashboards from the Config aggregator with advanced queries and a scheduled export to S3.
- C. Use Security Hub CSPM central configuration from the delegated administrator, with configuration policies associated with the production and sandbox OUs and a home Region that aggregates findings from the others.
- D. Turn on automatic enablement of new accounts with the default standards, and run a Lambda function on account-creation events that adjusts standards and disables controls per OU in each Region.

---

## Question 6 (Domain 5: Incident and Event Response)

A security operations team receives 40,000 Security Hub CSPM findings a week across 150 accounts. About 60% are known-benign — for example, findings on resources tagged `sandbox=true` — and analysts want those automatically marked as suppressed with a note. Critical findings in production accounts must keep their severity and automatically start an SSM Automation runbook.

The delegated administrator account aggregates findings from all accounts and Regions in a home Region, and auditors review suppressed findings in the Security Hub CSPM console.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Create an EventBridge rule for imported findings that invokes a Lambda function, which calls `BatchUpdateFindings` to suppress findings on sandbox resources and starts the runbook for critical production findings.
- B. Create automation rules in the delegated administrator account that suppress findings on `sandbox=true` resources with an explanatory note, and an EventBridge rule that starts the runbook for critical production findings.
- C. Run a nightly script in each member account that suppresses findings on sandbox-tagged resources, and add an EventBridge rule that starts the runbook for critical findings from production accounts.
- D. Export all findings to a SIEM through EventBridge and Amazon Data Firehose, suppress sandbox findings with SIEM rules that write the status back with `BatchUpdateFindings`, and trigger the runbook from the SIEM.

---

## Question 7 (Domain 6: Security and Compliance)

A company has 200 member accounts, each created with a root user whose password and MFA device are kept in a physical safe. Audits flag these credentials as a risk. The company wants to remove root credentials from member accounts entirely while keeping the ability to perform root-only tasks, such as deleting a bucket policy that denies access to all principals.

The company uses AWS Organizations with all features enabled and has an IAM delegated administrator account. Root-only tasks are needed about twice a year.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Keep the root credentials, but replace the physical safe with hardware MFA tokens registered to each account's root user, stored in a dual-custody vault with quarterly audits of every token.
- B. Enable centralized root access management for member accounts, delete their root credentials, and perform root-only tasks from the delegated administrator with `sts:AssumeRoot` and the matching task policy.
- C. Delete the root passwords, and whenever a root-only task is needed, recover the root password through the account's email address, register a temporary MFA device, perform the task, and delete the credentials again.
- D. Store each account's root password and virtual MFA seed in a privileged access management vault with session recording and approval workflows, and rotate the password after every use.

---

## Question 8 (Domain 1: SDLC Automation)

A platform team maintains 25 AWS Service Catalog products whose CloudFormation templates live in a Git repository. Each merge to `main` must publish a new product version for every changed product so that teams can update their provisioned products. Today an engineer uploads templates through the console, and versions are sometimes missed.

An existing CodePipeline (V2) pipeline already validates templates with cfn-lint on each merge, and the products are shared with 60 accounts through a portfolio.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Add an AWS Service Catalog deploy action to the existing pipeline for each product, pointing at the product ID and its template so that each merge creates a new product version.
- B. Add a CodeBuild action to the pipeline that detects changed templates and calls `CreateProvisioningArtifact` for each corresponding product, with a version name derived from the commit ID.
- C. Upload changed templates to an S3 bucket from the pipeline, and have an S3 event invoke a Lambda function that maps each object key to a product and creates the new product version.
- D. Keep the console process, but add a manual approval action that reminds the engineer to upload each changed template, with a checklist of products in the approval message.

---

## Question 9 (Domain 3: Resilient Cloud Solutions)

A company has 80 VPCs in 30 accounts that must resolve on-premises domains through centralized Route 53 Resolver outbound endpoints in a networking account, resolve 12 private hosted zones, and apply the same DNS Firewall rule groups. Teams create about five new VPCs a month, and each must get the full DNS configuration with a single association step in its VPC template.

The Resolver endpoints already exist in the networking account across two Availability Zones, and a Transit Gateway connects all VPCs to the data center. The company uses AWS RAM with organization sharing enabled.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Share the Resolver rules and DNS Firewall rule groups through AWS RAM, and run a StackSet with a Lambda-backed custom resource in each account that associates the rules, rule groups, and 12 private hosted zones with every VPC.
- B. Create a Route 53 Profile in the networking account that contains the Resolver rules, private hosted zones, and DNS Firewall rule groups, share it through AWS RAM, and associate it with each VPC.
- C. Deploy Resolver outbound endpoints in two Availability Zones of every VPC, with forwarding rules, private hosted zone associations, and DNS Firewall associations created by a StackSet in each account.
- D. Run a pair of DNS forwarder instances in two Availability Zones in the networking account, set every VPC's DHCP options to use them, and configure them with the on-premises zones and blocklists.

---

## Question 10 (Domain 2: Configuration Management and IaC)

A company wants every EC2 instance, EBS volume, and RDS instance in its 150 accounts to carry a `CostCenter` tag whose value comes from an approved list of 40 values, with consistent capitalization. Resources of these types must not be created without the tag, tagging operations that use an unapproved value must fail, and finance needs an organization-wide report of noncompliant resources.

The company uses AWS Organizations with all features enabled and already uses SCPs. Most resources are created with CloudFormation, but some are created from the console.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Create a tag policy that defines `CostCenter` with the approved values and enforcement for these resource types, add an SCP that denies their creation when `aws:RequestTag/CostCenter` is missing, and use tag policy compliance reports.
- B. Add SCPs that deny creating these resource types unless `aws:RequestTag/CostCenter` matches one of the 40 approved values, deny tagging actions that use other values, and build the finance report with the AWS Config `required-tags` rule and an aggregator.
- C. Require all resource creation to go through CloudFormation with a Guard hook that validates `CostCenter`, deny console creation with an SCP, and build the finance report from AWS Config `required-tags` results.
- D. Allow these resource types to be created only through Service Catalog products whose TagOptions provide the approved values, deny direct creation with an SCP, and build the finance report from AWS Config.

---

## Question 11 (Domain 4: Monitoring and Logging)

A company must log every object upload, copy, and deletion under three prefixes — `payroll/`, `contracts/`, and `pii/` — in 12 S3 buckets for 7 years, with CloudTrail log file integrity validation. Uploads to those prefixes are mostly large files sent through multipart uploads, which generate thousands of `UploadPart` calls per file. Reads and other write operations, such as tagging, don't need to be logged.

An organization trail already logs management events to a central bucket. Data events are not logged today.

**Which solution meets these requirements in the MOST cost-effective way?**

- A. Add basic event selectors to the organization trail for S3 data events on the three prefixes of each bucket, with the read/write type set to write-only.
- B. Add advanced event selectors to the organization trail that log all S3 write data events for the 12 buckets, and filter the events to the three prefixes and required operations later with Athena.
- C. Add advanced event selectors to the organization trail for S3 object data events where `readOnly` is false, `resources.ARN` starts with each prefix, and `eventName` matches the upload, copy, and delete operations.
- D. Create a second organization trail that logs S3 data events for the three prefixes, including reads and writes, so that one trail holds the data events and the other keeps the management events.

---

## Question 12 (Domain 6: Security and Compliance)

Developers in a workload account must be able to create and update IAM roles for their Lambda functions and pass those roles to Lambda without involving a platform engineer. The security team must ensure that no role a developer creates can exceed the permissions in the `dev-boundary` managed policy, and developers must not be able to alter or remove that boundary.

Developer-created roles use the IAM path `/app/`, and the account belongs to an OU with existing SCPs.

**Which solution meets these requirements with the LEAST privilege?**

- A. Allow `iam:CreateRole`, `iam:PutRolePolicy`, and `iam:AttachRolePolicy` only when `iam:PermissionsBoundary` equals the `dev-boundary` ARN, deny changes to the boundary policy and its removal from roles, and allow `iam:PassRole` on all roles.
- B. Allow `iam:*` on roles under `/app/`, and add an SCP that denies attaching AWS managed policies with administrative access to any role, so developer-created roles can't become administrators.
- C. Allow role creation and policy changes only when `iam:PermissionsBoundary` equals the `dev-boundary` ARN, and allow `iam:PassRole` on `/app/` roles to Lambda, but leave the `dev-boundary` policy itself unprotected.
- D. Allow role creation and policy changes only when `iam:PermissionsBoundary` equals the `dev-boundary` ARN, deny edits to the boundary policy and its removal, and allow `iam:PassRole` only on `/app/` roles passed to Lambda.

---

## Question 13 (Domain 1: SDLC Automation)

A company publishes 300 internal npm packages under the `@acme` scope to an AWS CodeArtifact repository that also has an upstream repository with an external connection to npmjs.com. A security review warns that an attacker could publish a higher version of an `@acme` package to the public registry and builds might resolve it — a dependency confusion attack. New internal packages are created every week.

CodeBuild projects and developer laptops use the CodeArtifact repository for all packages, and public packages from other scopes must still resolve through the upstream.

**Which solution meets these requirements with the LEAST operational overhead?**

- A. Write a script that sets package origin controls on each existing `@acme` package to block upstream versions, and run it from an EventBridge rule whenever a new `@acme` package is first published.
- B. Move internal packages to a separate repository with no upstreams, and update the `.npmrc` in every repository and on every laptop to resolve the `@acme` scope from it and everything else from the existing repository.
- C. Run a self-managed proxy in front of CodeArtifact that rejects requests for `@acme` package versions missing from the internal publish log, and point all clients at the proxy.
- D. Create a CodeArtifact package group that matches the `@acme` namespace with external upstream access blocked, so that current and future `@acme` packages resolve only from internally published versions.

---

## Question 14 (Domain 3: Resilient Cloud Solutions)

A company has 40 spoke VPCs across three Availability Zones that must send internet-bound traffic through AWS Network Firewall for inspection. Traffic must stay symmetric so the stateful firewall sees both directions of each flow, egress must continue if any one Availability Zone fails, and spoke VPCs must not have internet gateways.

All spokes attach to a Transit Gateway, and egress averages 30 TB per month. The security team will accept either a combined inspection-and-egress VPC or separate VPCs.

**Which solution meets these requirements in the MOST cost-effective way?**

- A. Create a central inspection VPC with Network Firewall endpoints in three AZs and a separate central egress VPC with NAT gateways in three AZs, routing spoke traffic through the inspection VPC and back through the Transit Gateway to the egress VPC, with appliance mode on the inspection attachment.
- B. Create one central inspection-and-egress VPC with Network Firewall endpoints and NAT gateways in every Availability Zone in the Region, not just three, and enable appliance mode on its Transit Gateway attachment.
- C. Create one central inspection-and-egress VPC with Network Firewall endpoints and NAT gateways in each of the three AZs, enable appliance mode on its Transit Gateway attachment, and route all spoke internet traffic to it.
- D. Deploy Network Firewall endpoints in three AZs inside each spoke VPC, and send inspected traffic through the Transit Gateway to a central egress VPC with NAT gateways in each of the three AZs.

---

## Question 15 (Domain 5: Incident and Event Response)

A company's security team must be alerted — and an automatic re-attachment triggered — within seconds whenever someone detaches an SCP from an OU or the root, or modifies or deletes an SCP. All such changes are made through the AWS Organizations API from the management account, usually by administrators who sign in to the console from Europe.

The management account's home Region is eu-west-1, where its EventBridge rules and incident tooling live. An organization trail delivers logs to S3 and to a CloudWatch Logs log group in eu-west-1.

**Which solution meets these requirements with the FASTEST detection?**

- A. Create a CloudWatch Logs metric filter on the organization trail's log group in eu-west-1 for the SCP-related events, with an alarm that invokes the remediation Lambda function and notifies the team.
- B. Create an EventBridge rule in the management account in us-east-1 that matches the SCP-related Organizations API calls from CloudTrail, targeting the remediation function and forwarding events to eu-west-1.
- C. Run a Lambda function every 5 minutes in the management account that lists the policies attached to the root and each OU, compares them with a baseline, and re-attaches any missing SCPs.
- D. Run an Athena query every 15 minutes over the organization trail in S3 for the SCP-related events, and invoke the remediation Lambda function for any matches.

---

## Answer Key & Detailed Explanations

| Q | Answer | Domain | Governing constraint |
|---|---|---|---|
| 1 | D | Domain 6: Security and Compliance | LEAST operational overhead |
| 2 | A | Domain 1: SDLC Automation | LEAST privilege |
| 3 | A | Domain 3: Resilient Cloud Solutions | LEAST operational overhead |
| 4 | D | Domain 2: Configuration Management and IaC | LEAST operational overhead |
| 5 | C | Domain 4: Monitoring and Logging | LEAST operational overhead |
| 6 | B | Domain 5: Incident and Event Response | LEAST operational overhead |
| 7 | B | Domain 6: Security and Compliance | LEAST operational overhead |
| 8 | A | Domain 1: SDLC Automation | LEAST operational overhead |
| 9 | B | Domain 3: Resilient Cloud Solutions | LEAST operational overhead |
| 10 | A | Domain 2: Configuration Management and IaC | LEAST operational overhead |
| 11 | C | Domain 4: Monitoring and Logging | MOST cost-effective |
| 12 | D | Domain 6: Security and Compliance | LEAST privilege |
| 13 | D | Domain 1: SDLC Automation | LEAST operational overhead |
| 14 | C | Domain 3: Resilient Cloud Solutions | MOST cost-effective |
| 15 | B | Domain 5: Incident and Event Response | FASTEST detection |

---

### Question 1

**Answer:** D  
**Constraint:** LEAST operational overhead · **Services:** AWS Organizations (resource control policies), Amazon S3, AWS KMS

**Socratic walkthrough**
1. *Can an SCP stop a vendor account's principal from reading your bucket?* — No. SCPs only restrict principals in your own organization.
2. *What restricts access to your resources regardless of who calls?* — Resource policies, or organization-wide resource control policies (RCPs).
3. *Which option covers existing and future resources with no per-resource work?* — An RCP attached at the root, for services that support RCPs (S3, KMS, SQS, and Secrets Manager do).

**Why D is correct:** An RCP sets the maximum permissions on resources in every member account. Denying requests from principals outside the organization — with an exception for AWS services acting on your behalf — overrides permissive bucket policies, can't be changed by account administrators, and needs no resource-by-resource deployment.

**Why the others fail the constraint:**
- **A.** Works, but means policy rewriting across four services plus Config remediation, with drift windows.
- **B.** Works, but is custom code in 300 accounts that rewrites resource policies after the fact.
- **C.** Works, but forces all creation through Service Catalog and still requires a StackSet for existing resources.

**Exam reflex:** SCPs limit your principals; RCPs limit access to your resources — data perimeters need RCPs.

---

### Question 2

**Answer:** A  
**Constraint:** LEAST privilege · **Services:** GitHub Actions OIDC, AWS IAM (OIDC identity provider), Amazon ECR

**Socratic walkthrough**
1. *Which claim identifies the repository and environment?* — `sub`, for example `repo:acme/payments-api:environment:production`.
2. *What do wildcards in `sub` allow?* — Other repositories (`acme/*`) or any branch or pull request workflow in the repository (`:*`).
3. *What should the role be able to do?* — Push to one repository, not manage every ECR repository.

**Why A is correct:** An exact `sub` match limits role assumption to workflows running in the approved `production` environment of one repository, and the permissions policy limits actions to pushing images to `payments-api`. Both who can assume the role and what it can do are minimal.

**Why the others fail the constraint:**
- **B.** Works, but any of the 400 repositories in the organization can assume the role.
- **C.** Works, but any branch, tag, or pull request workflow in the repository can assume it, bypassing environment approvals.
- **D.** Works, but the power-user managed policy grants broad ECR actions on every repository.

**Exam reflex:** For GitHub OIDC least privilege, match the exact `sub` (repository plus environment) and scope the permissions to the resource.

---

### Question 3

**Answer:** A  
**Constraint:** LEAST operational overhead · **Services:** AWS Backup (organization backup policies), AWS Backup logically air-gapped vault, AWS Organizations / AWS RAM

**Socratic walkthrough**
1. *What makes copies immune to compromised source-account administrators?* — Immutable, isolated storage outside the source accounts.
2. *Which vault type provides that by design, with AWS-managed encryption?* — A logically air-gapped vault.
3. *How do recovery and new accounts work?* — Share the vault through AWS RAM for restores; organization backup policies apply automatically to new accounts.

**Why A is correct:** Organization backup policies schedule backups everywhere and copy recovery points into a logically air-gapped vault, which is locked by design. Sharing it with the recovery account through RAM enables isolated restores without managing cross-account KMS keys.

**Why the others fail the constraint:**
- **B.** Works, but you operate a central vault account and share a customer managed key with 120 accounts.
- **C.** Works, but means per-account vault deployment and locking plus cross-account copy configuration.
- **D.** Works, but custom copy code and SCP-based protection are more to build and audit.

**Exam reflex:** For ransomware-resilient backups without managing keys, use a logically air-gapped vault with organization backup policies.

---

### Question 4

**Answer:** D  
**Constraint:** LEAST operational overhead · **Services:** AWS Organizations (declarative policies), Amazon EC2, service control policies

**Socratic walkthrough**
1. *Are these settings permissions or service configuration?* — Configuration: account-level EC2 attributes, set per Region.
2. *What enforces service configuration across accounts and Regions, including future ones?* — Declarative policies for EC2.
3. *Can they show a custom message?* — Yes. Declarative policies support custom error messages.

**Why D is correct:** A declarative policy at the root sets image block public access, snapshot block public access, and instance metadata defaults for every account and Region, and AWS maintains the configuration as accounts and Regions are added. Account administrators can't change the settings, and blocked actions show your message.

**Why the others fail the constraint:**
- **A.** Works, but means StackSets across 9 Regions plus an SCP to maintain, and new Regions need updates.
- **B.** Works, but it's custom cross-account code with a weekly reconciliation loop.
- **C.** Works, but condition-key SCPs only block specific API calls and don't set account defaults or show custom messages.

**Exam reflex:** For durable EC2 account-wide settings across the organization, use declarative policies.

---

### Question 5

**Answer:** C  
**Constraint:** LEAST operational overhead · **Services:** AWS Security Hub CSPM (central configuration), AWS Organizations, AWS Config

**Socratic walkthrough**
1. *What varies by OU?* — Enabled standards and disabled controls.
2. *Which Security Hub CSPM feature applies configuration by OU from one place, including to new accounts?* — Central configuration with configuration policies.
3. *How do you see all Regions in one?* — Designate a home Region that aggregates findings from linked Regions.

**Why C is correct:** Central configuration lets the delegated administrator define production and sandbox configuration policies and associate them with OUs. Accounts that join those OUs inherit them automatically, and the home Region aggregates findings.

**Why the others fail the constraint:**
- **A.** Works, but means StackSets for every account, Region, and OU variation.
- **B.** Works, but re-implements standards as custom rule sets and builds custom dashboards.
- **D.** Works, but per-OU adjustments are custom code that runs Region by Region.

**Exam reflex:** For OU-specific Security Hub CSPM configuration, use central configuration policies.

---

### Question 6

**Answer:** B  
**Constraint:** LEAST operational overhead · **Services:** AWS Security Hub CSPM (automation rules), Amazon EventBridge, AWS Systems Manager Automation

**Socratic walkthrough**
1. *Can Security Hub CSPM change findings by itself as they arrive?* — Yes. Automation rules update fields such as workflow status and notes when criteria match.
2. *Where are the rules defined?* — Once, in the administrator account, applying to member findings.
3. *How do you trigger the runbook?* — An EventBridge rule on critical production findings.

**Why B is correct:** Automation rules suppress sandbox findings with a note at ingestion, so auditors see the status in the console. An EventBridge rule starts the Automation runbook for critical production findings, with no custom code.

**Why the others fail the constraint:**
- **A.** Works, but a Lambda function re-implements automation rules.
- **C.** Works, but nightly scripts in 150 accounts are delayed and duplicated.
- **D.** Works, but adds a SIEM pipeline and write-back integration for something configuration handles.

**Exam reflex:** To triage findings in Security Hub CSPM, use automation rules, and EventBridge for response actions.

---

### Question 7

**Answer:** B  
**Constraint:** LEAST operational overhead · **Services:** AWS Organizations (centralized root access), AWS IAM (`sts:AssumeRoot`), AWS CloudTrail

**Socratic walkthrough**
1. *Can member accounts exist without root credentials?* — Yes. Centralized root access management lets you delete them.
2. *How are root-only tasks done then?* — With short-lived privileged sessions from the management or delegated administrator account through `sts:AssumeRoot`, scoped by a task policy.
3. *Is a bucket-policy unlock one of those tasks?* — Yes. There's a task policy for unlocking S3 bucket policies.

**Why B is correct:** Centralized root access removes 200 sets of credentials to guard and audit. Twice-a-year tasks run as scoped, logged root sessions from one account, with nothing physical to manage.

**Why the others fail the constraint:**
- **A.** Works, but you still manage 200 tokens, custody processes, and audits.
- **C.** Works, but email recovery and MFA setup every time is slow and manual.
- **D.** Works, but you still store and rotate 200 root credentials in a vault.

**Exam reflex:** To remove member-account root credentials while keeping root tasks possible, use centralized root access management with `AssumeRoot`.

---

### Question 8

**Answer:** A  
**Constraint:** LEAST operational overhead · **Services:** AWS Service Catalog, AWS CodePipeline, AWS CloudFormation

**Socratic walkthrough**
1. *Does CodePipeline have a native Service Catalog integration?* — Yes. There's an AWS Service Catalog deploy action.
2. *What does it do?* — Creates a new product version from a template in the pipeline's artifact.
3. *What would otherwise be custom?* — Change detection, API calls, and mapping templates to products.

**Why A is correct:** One deploy action per product in the existing pipeline publishes a new product version on each merge, and the portfolio's existing sharing delivers it to the 60 accounts. It's pure pipeline configuration.

**Why the others fail the constraint:**
- **B.** Works, but scripts must detect changes and call the API.
- **C.** Works, but adds an S3 event, a mapping function, and its upkeep.
- **D.** Works, but keeps the manual step that caused missed versions.

**Exam reflex:** To publish Service Catalog products from CI/CD, use CodePipeline's Service Catalog deploy action.

---

### Question 9

**Answer:** B  
**Constraint:** LEAST operational overhead · **Services:** Route 53 Resolver, Route 53 Profiles, AWS RAM

**Socratic walkthrough**
1. *How many separate associations does each VPC need without Profiles?* — Rules, 12 private hosted zones, and firewall rule groups — 14 or more per VPC.
2. *What bundles DNS configuration so it can be applied with one association?* — A Route 53 Profile.
3. *How does it reach other accounts?* — Through AWS RAM sharing.

**Why B is correct:** A Route 53 Profile packages the Resolver rules, private hosted zones, and DNS Firewall rule groups. After sharing through RAM, each VPC needs one profile association, and updating the profile updates every associated VPC. The existing multi-AZ endpoints provide resilience.

**Why the others fail the constraint:**
- **A.** Works, but custom resources must perform and maintain many associations per VPC.
- **C.** Works, but 160 extra endpoints plus per-VPC associations are costly to operate.
- **D.** Works, but you patch, scale, and keep self-managed DNS servers highly available.

**Exam reflex:** To apply DNS configuration across many VPCs and accounts, use Route 53 Profiles shared through RAM.

---

### Question 10

**Answer:** A  
**Constraint:** LEAST operational overhead · **Services:** AWS Organizations (tag policies), service control policies, tag policy compliance reports

**Socratic walkthrough**
1. *What standardizes tag keys, values, and capitalization, and blocks tagging with bad values?* — A tag policy with enforcement for the resource types.
2. *What blocks creating resources without the tag at all?* — Tag policies don't. An SCP with `aws:RequestTag/CostCenter` does.
3. *Is there a built-in organization-wide report?* — Yes. Tag policy compliance reports.

**Why A is correct:** The tag policy holds the 40 approved values in one place and enforces them on tagging operations. A small SCP requires the tag at creation, and compliance reports give finance the organization view — there's no value list duplicated in IAM and no custom code.

**Why the others fail the constraint:**
- **B.** Works, but encoding 40 values in SCP conditions across services is unwieldy, and reporting needs another service.
- **C.** Works, but forces all creation through CloudFormation and needs hook rules plus Config reporting.
- **D.** Works, but routes every resource through Service Catalog — the heaviest process change.

**Exam reflex:** Tag policies standardize values; SCPs with `aws:RequestTag` require tags at creation. Use both.

---

### Question 11

**Answer:** C  
**Constraint:** MOST cost-effective · **Services:** AWS CloudTrail (advanced event selectors), Amazon S3, CloudTrail log file validation

**Socratic walkthrough**
1. *How are data events billed?* — Per event delivered.
2. *Which events dominate here?* — `UploadPart`, thousands per file, which the requirement doesn't need.
3. *Which selector type can filter by event name as well as prefix?* — Advanced event selectors.

**Why C is correct:** Advanced event selectors combine resource ARN prefixes, `readOnly`, and `eventName`, so the trail delivers only uploads (including `CompleteMultipartUpload`), copies, and deletions on the three prefixes. Every unneeded `UploadPart` event is excluded before it's billed.

**Why the others fail the constraint:**
- **A.** Works, but write-only basic selectors still log every `UploadPart` and tagging call.
- **B.** Works, but pays for all write events across entire buckets and filters them afterward.
- **D.** Works, but logs reads and multipart parts and adds a second trail.

**Exam reflex:** To cut CloudTrail data-event cost, filter at the selector with advanced fields such as `eventName`.

---

### Question 12

**Answer:** D  
**Constraint:** LEAST privilege · **Services:** AWS IAM (permissions boundaries), AWS Lambda, AWS Organizations (SCPs)

**Socratic walkthrough**
1. *What forces every developer-created role to carry the boundary?* — Conditions on `iam:PermissionsBoundary` for role creation and policy changes.
2. *What else could developers do to escape?* — Edit the boundary policy, remove the boundary, or pass roles they shouldn't.
3. *How narrow can `iam:PassRole` be?* — Roles under `/app/` only, and only to Lambda.

**Why D is correct:** D enforces the boundary, protects it from editing and removal, and limits `PassRole` by both role path and target service. Developers keep self-service with no escalation paths.

**Why the others fail the constraint:**
- **A.** Works, but `iam:PassRole` on all roles lets developers attach administrative roles to their functions.
- **B.** Works, but `iam:*` on `/app/` roles with an SCP denylist leaves many escalation paths open.
- **C.** Works, but developers could edit `dev-boundary` itself and widen every role.

**Exam reflex:** Permissions-boundary delegation needs the boundary condition, protection of the boundary policy, and scoped `PassRole`.

---

### Question 13

**Answer:** D  
**Constraint:** LEAST operational overhead · **Services:** AWS CodeArtifact (package groups and origin controls), AWS CodeBuild, npm

**Socratic walkthrough**
1. *Where must the block apply?* — To every current and future `@acme` package at the CodeArtifact level, for every client.
2. *Can origin controls be applied by pattern instead of package by package?* — Yes. Package groups define origin restrictions for all packages that match.
3. *Does anything change on the clients?* — No.

**Why D is correct:** A package group matching the `@acme` namespace blocks external upstream versions for all matching packages, including those created next week. Public packages from other scopes keep resolving normally.

**Why the others fail the constraint:**
- **A.** Works, but per-package automation must react to every new package.
- **B.** Works, but every repository and laptop must keep scoped registry configuration correct.
- **C.** Works, but a self-managed proxy and its publish log are heavy to run.

**Exam reflex:** To prevent dependency confusion in CodeArtifact, block external upstreams with package groups.

---

### Question 14

**Answer:** C  
**Constraint:** MOST cost-effective · **Services:** AWS Transit Gateway (appliance mode), AWS Network Firewall, NAT gateways

**Socratic walkthrough**
1. *How is Transit Gateway billed?* — Per attachment-hour and per GB processed.
2. *What does a separate egress VPC add?* — A second trip through the Transit Gateway for 30 TB a month.
3. *How many firewall endpoints and NAT gateways do you need for resilience across three AZs?* — One per AZ in a single central VPC, not one per spoke and not one in every AZ in the Region.

**Why C is correct:** A combined inspection-and-egress VPC with endpoints and NAT gateways in each of the three AZs survives an AZ failure. Appliance mode keeps flows symmetric through the stateful firewall, and traffic crosses the Transit Gateway only once on its way out.

**Why the others fail the constraint:**
- **A.** Works, but doubles Transit Gateway data processing and adds an attachment.
- **B.** Works, but pays for firewall endpoints and NAT gateways in AZs with no spoke traffic.
- **D.** Works, but 120 firewall endpoints across 40 spokes cost far more than three central ones.

**Exam reflex:** Centralize inspection and egress in one VPC per Region with one endpoint per AZ and appliance mode on.

---

### Question 15

**Answer:** B  
**Constraint:** FASTEST detection · **Services:** AWS Organizations (CloudTrail events), Amazon EventBridge, AWS Lambda

**Socratic walkthrough**
1. *In which Region are AWS Organizations API events recorded?* — In us-east-1, because Organizations is a global service.
2. *So where must the EventBridge rule live?* — In the management account in us-east-1, not in the eu-west-1 home Region.
3. *Which other options wait on log delivery or polling?* — A, C, and D.

**Why B is correct:** An EventBridge rule on CloudTrail events in us-east-1 receives the Organizations API calls in near real time. It invokes remediation immediately and forwards the event to the eu-west-1 tooling.

**Why the others fail the constraint:**
- **A.** Works, but CloudTrail delivery to CloudWatch Logs plus alarm evaluation takes minutes.
- **C.** Works, but detection waits up to 5 minutes.
- **D.** Works, but waits for log delivery and a 15-minute schedule.

**Exam reflex:** Global-service API events (Organizations, IAM) land in us-east-1 — put the EventBridge rule there.

---

## Scoring & Performance Breakdown

| Score | Band | Next step |
|---|---|---|
| **14–15 / 15** | Exam ready | You're ready for the real exam; revisit any domain you missed above. |
| **12–13 / 15** | Passing | For each miss, re-read the distractor breakdown and note which constraint signal you overlooked. |
| **≤ 11 / 15** | Needs remediation | Use the domain column in the key table to pick study guides; retake this exam in a week. |
