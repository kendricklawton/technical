# Domain 3 — Quiz 3: AI Governance, Compliance, Audit Logging, & Responsible AI (Tasks 3.3 & 3.4)

**Instructions:** This 15-question Staff/Principal-level diagnostic quiz tests **Task 3.3 (Implement AI Governance and Compliance Mechanisms)** and **Task 3.4 (Implement Responsible AI Principles)** of the AWS Certified Generative AI Developer - Professional (AIP-C01) exam. Questions include single-select (`A`–`D`) and multiple-response (`Choose TWO` from `A`–`E`) items.

---

### Question 1
A healthcare insurer runs an executive assistant on Amazon Bedrock that processes confidential Mergers & Acquisitions (M&A) documents and patient actuarial summaries. The Chief Compliance Officer has two strict, seemingly conflicting audit requirements:
1. The central Security Operations Center (SOC) team must be able to audit **every single `InvokeModel`, `Converse`, `Retrieve`, and `InvokeAgent` API call**—capturing the exact caller IAM Principal ARN, source IP address, timestamp, foundation model ID, and HTTP response code—in their centralized SIEM.
2. However, because of M&A insider-trading rules and HIPAA privacy boundaries, SOC analysts **must never be able to view the actual prompt text or model completion text** inside those audit logs.

Which logging configuration satisfies both requirements?

- **A.** Enable **AWS CloudTrail Data Events** for `AWS::Bedrock::Model`, `AWS::Bedrock::KnowledgeBase`, and `AWS::Bedrock::AgentAlias` on the organization trail delivering to the SOC SIEM, while keeping **Amazon Bedrock Model Invocation Logging** disabled (or restricted to a separate locked-down vault inaccessible to the SOC).
- **B.** Enable **Amazon Bedrock Model Invocation Logging** (`PutModelInvocationLoggingConfiguration`) with `textDataDeliveryEnabled: true` delivering directly to the SOC SIEM's CloudWatch Log Group.
- **C.** Rely exclusively on default **AWS CloudTrail Management Events**, which automatically record all `InvokeModel` and `Converse` calls without logging prompt bodies.
- **D.** Enable VPC Flow Logs on the `bedrock-runtime` PrivateLink interface endpoint to extract the IAM Principal ARN and foundation model ID from TCP packet headers.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: A**

#### Explanation
- **AWS CloudTrail Data Events** for Bedrock (`AWS::Bedrock::Model`, `AWS::Bedrock::KnowledgeBase`, `AWS::Bedrock::AgentAlias`, `AWS::Bedrock::Guardrail`) capture runtime data-plane API invocations (`InvokeModel`, `Converse`, `Retrieve`, `InvokeAgent`, `ApplyGuardrail`) including the caller's `userIdentity` (IAM ARN, account ID, session name), `sourceIPAddress`, `eventTime`, and `requestParameters` (such as `modelId`), **without ever recording the prompt input body or model completion text**.
- Conversely, **Bedrock Model Invocation Logging** records the full prompt and completion payloads.

#### Distractor Analysis
- **Why B is incorrect:** Model Invocation Logging with `textDataDeliveryEnabled: true` writes the full confidential M&A and patient prompt/completion text into the logs, violating Requirement 2.
- **Why C is incorrect:** CloudTrail Management Events only record control-plane API calls (e.g., `CreateGuardrail`, `DeleteAgent`); they **never** record data-plane calls like `InvokeModel`, `Converse`, or `Retrieve`.
- **Why D is incorrect:** VPC Flow Logs only capture Layer 3/4 IP addresses, ports, and byte counts; TLS encrypts the HTTP payload so VPC Flow Logs cannot see the IAM Principal ARN or `modelId`.

</details>

---

### Question 2
An enterprise RAG platform uses `amazon.titan-embed-text-v2:0` to generate vector embeddings for 15 million document chunks per day and uses `anthropic.claude-3-5-sonnet-20241022-v2:0` to answer customer chat queries. The security team enables **Amazon Bedrock Model Invocation Logging** (`PutModelInvocationLoggingConfiguration`) to an S3 bucket and CloudWatch Logs so they can inspect customer chat prompts, multimodal image uploads, and Claude responses for prompt injection forensics. One week later, the FinOps team alerts the architect that CloudWatch Logs ingestion and S3 storage costs have spiked by \$18,000/month because 15 million 1,024-dimensional float arrays from the Titan Embedding model are being logged every day.

How can the architect immediately eliminate the embedding log storage bloat while retaining full text and image forensic logging for all generative chat invocations?

- **A.** Disable Model Invocation Logging for the entire AWS account and rely on CloudTrail Management Events to capture chat prompt text.
- **B.** Call `put_model_invocation_logging_configuration` and set `embeddingDataDeliveryEnabled: False` while keeping `textDataDeliveryEnabled: True` and `imageDataDeliveryEnabled: True`.
- **C.** Set `textDataDeliveryEnabled: False` and `embeddingDataDeliveryEnabled: True` in `put_model_invocation_logging_configuration`.
- **D.** Add a Denied Topic for `"Vector Embeddings"` in Amazon Bedrock Guardrails to stop Bedrock from logging float arrays.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: B**

#### Explanation
`PutModelInvocationLoggingConfiguration` provides three independent modality toggles:
- `textDataDeliveryEnabled` (`True` | `False`)
- `imageDataDeliveryEnabled` (`True` | `False`)
- `embeddingDataDeliveryEnabled` (`True` | `False`)

Setting `embeddingDataDeliveryEnabled: False` stops Bedrock from logging invocation payloads for embedding models (like `amazon.titan-embed-text-v2:0`), eliminating massive float-array log volume while preserving full text and image invocation logging for generative models.

#### Distractor Analysis
- **Why A is incorrect:** Disables text/image forensic logging completely, and CloudTrail Management Events do not capture prompts or completions.
- **Why C is incorrect:** Does the exact opposite of what is needed—disabling text chat logs while keeping expensive embedding float logs.
- **Why D is incorrect:** Bedrock Guardrails have no effect on `PutModelInvocationLoggingConfiguration` modality delivery settings.

</details>

---

### Question 3
A global media organization operates a news verification platform. Investigative journalists frequently receive dramatic photographs from anonymous tipsters and need to determine whether an image was synthetically generated by an AI model using **Amazon Titan Image Generator** or **Amazon Nova Canvas** (even if the anonymous sender cropped the image by 15%, saved it as a compressed JPEG, or applied a slight contrast filter).

Which Responsible AI capability in Amazon Bedrock enables the platform to verify the provenance of these images?

- **A.** Amazon Bedrock Guardrails Contextual Grounding Checks, which inspect image EXIF GPS headers against a database of known camera serial numbers.
- **B.** Amazon SageMaker Clarify Kernel SHAP feature attribution, which identifies synthetic pixels by training a linear regression model on JPEG headers.
- **C.** **Amazon Bedrock Watermark Detection** (`DetectGeneratedContent` / Bedrock console watermark verification), which detects the tamper-resistant, imperceptible cryptographic watermark automatically embedded into the pixels of all images generated by Amazon Titan Image Generator and Amazon Nova Canvas and outputs a confidence level alongside C2PA provenance metadata.
- **D.** Amazon Rekognition `DetectModerationLabels`, which automatically identifies the exact AWS Account ID that generated any AI image.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: C**

#### Explanation
All images generated by **Amazon Titan Image Generator** and **Amazon Nova Canvas** automatically contain an invisible, tamper-resistant watermark embedded directly into the pixel frequencies (which cannot be disabled by customers). This watermark is engineered to survive common image manipulations such as JPEG compression, resizing, cropping, and minor color adjustments. Amazon Bedrock's watermark detection capability analyzes an image and returns a confidence score indicating whether it was generated by an AWS image model, supporting C2PA (Coalition for Content Provenance and Authenticity) digital credentials.

#### Distractor Analysis
- **Why A is incorrect:** Contextual Grounding Checks evaluate text claims against reference passages, not image watermarks or EXIF headers (which are trivially stripped).
- **Why B is incorrect:** SageMaker Clarify SHAP explains feature importance in ML models; it does not verify cryptographic watermarks in images.
- **Why D is incorrect:** Rekognition `DetectModerationLabels` detects NSFW/violent content, not invisible provenance watermarks or source AWS Account IDs.

</details>

---

### Question 4
A large enterprise manages 180 AWS member accounts using **AWS Organizations**. Each product team has `PowerUserAccess` in its own member account to build generative AI features on Amazon Bedrock. Central AI Governance policy mandates two non-negotiable guardrails across all 180 member accounts:
1. No IAM user or role in any member account (except the dedicated `arn:aws:iam::*:role/Central-SecOps-AI-Governance-Role`) may disable or modify the account's **Amazon Bedrock Model Invocation Logging** configuration, nor delete or modify production **Amazon Bedrock Guardrails**.
2. Product teams may only invoke foundation models in `us-east-1` and `us-west-2`; invoking Bedrock models in any other AWS Region must be blocked organization-wide.

How should the Principal Cloud Governance Architect enforce these rules so even a member-account administrator cannot bypass them?

- **A.** Create an IAM Managed Policy in each of the 180 member accounts and ask product teams not to detach it.
- **B.** Configure an Amazon Bedrock Guardrail with a Denied Topic for `"Unapproved AWS Regions"`.
- **C.** Enable AWS Config Detective Rules that send a weekly email summary of unapproved region invocations to the CISO.
- **D.** Attach an **AWS Organizations Service Control Policy (SCP)** to the root or production Organizational Units (OUs) containing explicit `Deny` statements for `bedrock:DeleteModelInvocationLoggingConfiguration`, `bedrock:PutModelInvocationLoggingConfiguration`, `bedrock:DeleteGuardrail`, and `bedrock:UpdateGuardrail` (when `aws:PrincipalARN` is not `Central-SecOps-AI-Governance-Role`), plus an explicit `Deny` statement on `bedrock:*` when `aws:RequestedRegion` is `StringNotEquals` to `["us-east-1", "us-west-2"]`.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: D**

#### Explanation
**AWS Organizations Service Control Policies (SCPs)** establish the maximum available permissions boundary across member accounts—even for principals with `AdministratorAccess` or `PowerUserAccess` inside a member account. An explicit `Deny` in an SCP blocking `bedrock:DeleteModelInvocationLoggingConfiguration`, `bedrock:PutModelInvocationLoggingConfiguration`, `bedrock:DeleteGuardrail`, and `bedrock:UpdateGuardrail` (exempting only the central SecOps role via `ArnNotLike: {"aws:PrincipalARN": ...}`) and restricting `aws:RequestedRegion` to `["us-east-1", "us-west-2"]` provides preventative, tamper-proof governance across all 180 accounts.

#### Distractor Analysis
- **Why A is incorrect:** Principals with `PowerUserAccess` or `AdministratorAccess` inside a member account can modify or detach local IAM policies.
- **Why B is incorrect:** Bedrock Guardrails inspect prompt text; they do not enforce AWS IAM control-plane or regional API restrictions.
- **Why C is incorrect:** Weekly detective emails do not *prevent* unapproved regional invocations or log tampering in real time.

</details>

---

### Question 5
A commercial lender is preparing a tabular + text credit risk dataset in Amazon S3 to fine-tune an internal credit scoring and explanation model on Amazon SageMaker. Before spending compute budget on training, the Fair Lending Compliance team requires a **Pre-Training Bias Report** quantifying two specific statistical conditions in the historical training dataset across protected demographic facets (`age_group` and `marital_status`):
1. Whether certain demographic subgroups have disproportionately fewer historical loan records in the dataset compared to the majority group.
2. Whether the historical proportion of positive credit approvals (`label == 1`) differs significantly between privileged and unprivileged demographic facets.

Which **Amazon SageMaker Clarify** pre-training bias metrics directly measure these two conditions?

- **A.** **ROUGE-L** and **BLEU**, which measure n-gram overlap across demographic groups prior to training.
- **B.** **Class Imbalance (CI)** to measure differences in the number of training examples between demographic facets, and **Difference in Proportions of Labels (DPL)** (along with **Kullback-Leibler Divergence `KL`** / **Jensen-Shannon Divergence `JS`**) to measure disparities in positive outcome label distributions across facets.
- **C.** **Perplexity** and **Cosine Similarity**, which measure token probability distributions in the raw CSV file.
- **D.** **Disparate Impact (DI)** and **Counterfactual Fliptest (FT)**, because those two metrics require no model predictions and only count rows in S3.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: B**

#### Explanation
**Amazon SageMaker Clarify** separates bias metrics into **Pre-Training Bias Metrics** (which analyze only the dataset features and ground-truth labels $y$, requiring no trained model) and **Post-Training Bias Metrics** (which analyze the trained model's predicted labels $\hat{y}$).
- **Class Imbalance (`CI`)**: Pre-training metric measuring whether one demographic facet has significantly fewer representation rows than another: $CI = \frac{n_a - n_d}{n_a + n_d}$.
- **Difference in Proportions of Labels (`DPL`)**: Pre-training metric measuring the difference in positive ground-truth label rates ($q_a - q_d$) between demographic facets.

#### Distractor Analysis
- **Why A is incorrect:** ROUGE-L and BLEU are text summarization/translation quality metrics, not demographic fairness metrics.
- **Why C is incorrect:** Perplexity measures language model token uncertainty; Cosine Similarity measures vector angles.
- **Why D is incorrect:** **Disparate Impact (`DI`)** and **Counterfactual Fliptest (`FT`)** are **Post-Training Bias Metrics** that require invoking a trained model to compare predicted labels ($\hat{y}$).

</details>

---

### Question 6
After training and deploying an NLP loan-application triage model on Amazon SageMaker, loan officers ask *why* the model flagged a specific commercial real estate narrative as `"High Default Risk"`. Regulatory auditors under the Equal Credit Opportunity Act (ECOA) require local **feature attribution explanations** showing which specific words or phrases in the applicant's business plan contributed positively or negatively toward the `"High Default Risk"` prediction, as well as continuous monitoring to alert the ML team if the feature attribution rankings drift significantly in production traffic over time.

Which AWS capability should the ML Architect implement?

- **A.** Enable Amazon Bedrock Guardrails Automated Reasoning Checks on the SageMaker endpoint to calculate Shapley values.
- **B.** Use Amazon Macie to rank which words in the business plan are PII entities.
- **C.** Query AWS CloudTrail Data Events to see the attention weights of the transformer layers.
- **D.** Configure **Amazon SageMaker Clarify Explainability** using the **Kernel SHAP (SHapley Additive exPlanations)** algorithm with a text `TextConfig` (`granularity="sentence"` or `"token"`), and schedule **SageMaker Clarify Model Monitor** jobs on the production endpoint to detect **Feature Attribution Drift**.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: D**

#### Explanation
**Amazon SageMaker Clarify** uses **Kernel SHAP** to compute local (per-instance) and global (dataset-wide) feature attributions. For NLP text models, configuring `TextConfig` (`granularity` set to `"token"`, `"sentence"`, or `"paragraph"`, and `language`) instructs Clarify to mask/replace text segments with baseline tokens and measure the resulting change in model prediction probability, assigning a signed SHAP attribution score to each phrase. Furthermore, **SageMaker Model Monitor** integrates with Clarify to continuously monitor production endpoints for both **Bias Drift** and **Feature Attribution (SHAP) Drift**.

#### Distractor Analysis
- **Why A is incorrect:** Automated Reasoning Checks perform formal logic verification in Bedrock Guardrails; they do not compute Kernel SHAP feature attributions on SageMaker models.
- **Why B is incorrect:** Amazon Macie discovers sensitive PII in S3 buckets, not model prediction feature attributions.
- **Why C is incorrect:** CloudTrail Data Events log API metadata, never internal neural network weights or SHAP scores.

</details>

---

### Question 7
A hospital network is deploying a clinical prior-authorization AI assistant on Amazon Bedrock. When a physician submits a prior-authorization request alongside a patient's clinical chart, an AWS Step Functions workflow invokes `bedrock-runtime:converse` with an attached Amazon Bedrock Guardrail (`trace="enabled"`). Clinical safety governance mandates a **three-tiered confidence routing policy** based on the Guardrail's **Contextual Grounding (`GROUNDING`)** score:
- **Tier 1 (`score >= 0.88`)**: Automatically approve and return the AI-generated determination to the physician portal.
- **Tier 2 (`0.70 <= score < 0.88`)**: Do **not** return the AI output directly to the physician; instead, route the query, clinical chart reference, and candidate AI output to a private team of licensed clinical nurses so a nurse can review, edit, or approve the response before it is released.
- **Tier 3 (`score < 0.70`)**: Automatically block the response and log a compliance incident.

Which AWS service integrates natively with AWS Step Functions to manage the private clinical nurse workforce portal, task queuing, and human review UI for Tier 2 requests?

- **A.** **Amazon Augmented AI (Amazon A2I)** (`sagemaker-a2i-runtime:StartHumanLoop`) configured with a Human Review Workflow (`FlowDefinition`), a custom Liquid HTML worker task template, and an **Amazon SageMaker Ground Truth Private Workforce** backed by Amazon Cognito.
- **B.** Amazon Mechanical Turk public crowd workforce invoked via an unauthenticated S3 presigned URL.
- **C.** AWS Systems Manager Incident Manager runbooks triggered via CloudTrail Management Events.
- **D.** Amazon Rekognition Custom Labels.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: A**

#### Explanation
**Amazon Augmented AI (Amazon A2I)** provides built-in human-in-the-loop (HITL) workflows (`StartHumanLoop`) with customizable worker UI templates and native integration with **Amazon SageMaker Ground Truth Private Workforces** (authenticated via Amazon Cognito / OIDC). When Step Functions inspects the Guardrail trace and finds a grounding score in the gray zone (`0.70 <= score < 0.88`), invoking `StartHumanLoop` dispatches the candidate response and source chart to the licensed clinical nurse review portal and emits an EventBridge event upon human completion.

#### Distractor Analysis
- **Why B is incorrect:** Sending protected health information (PHI) to the public, unvetted Amazon Mechanical Turk workforce is a severe HIPAA violation.
- **Why C is incorrect:** Systems Manager Incident Manager is designed for IT infrastructure outage response, not clinical document annotation and editing workflows.
- **Why D is incorrect:** Rekognition Custom Labels is a computer vision training service, not a text human-in-the-loop review portal.

</details>

---

### Question 8
A European bank is subject to the **EU AI Act** and **SR 11-7 Model Risk Management (MRM)** audits. The bank's data science team fine-tunes multiple domain-specific models on Amazon SageMaker and registers them in the **Amazon SageMaker Model Registry**. Before the MRM committee approves a model version for production deployment, regulators require a standardized, versioned, exportable governance artifact that documents:
- The model's intended business purpose and out-of-scope uses
- Risk rating (`High`, `Medium`, `Low`)
- Training dataset S3 URIs and lineage
- Quantitative evaluation metrics and SageMaker Clarify bias/explainability results
- Ethical considerations, caveats, and formal sign-off status

Which native AWS feature provides this exact centralized documentation and governance artifact?

- **A.** AWS Artifact Compliance Reports.
- **B.** Amazon Bedrock Prompt Management templates.
- **C.** **Amazon SageMaker Model Cards** (with PDF export and integration with SageMaker Model Registry and SageMaker Clarify evaluation reports).
- **D.** AWS CloudFormation StackSets drift detection logs.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: C**

#### Explanation
**Amazon SageMaker Model Cards** provide a structured, centralized system of record for responsible AI governance and regulatory documentation. Model Cards automatically pull training details, hyperparameters, evaluation metrics, and SageMaker Clarify bias/SHAP reports from the **SageMaker Model Registry** and **SageMaker ML Lineage Tracking**, allow risk officers to record intended uses, risk ratings, and approval statuses (`Draft`, `PendingReview`, `Approved`, `Archived`), and support immutable PDF exports for external auditors.

#### Distractor Analysis
- **Why A is incorrect:** AWS Artifact provides *AWS's own* ISO/SOC2/PCI compliance certificates, not documentation for a customer's custom ML models.
- **Why B is incorrect:** Bedrock Prompt Management stores versioned prompt strings, not MRM governance documentation or bias metrics.
- **Why D is incorrect:** CloudFormation drift detection tracks infrastructure-as-code resource changes, not ML model risk governance.

</details>

---

### Question 9
A securities trading firm must retain all **Amazon Bedrock Model Invocation Logs** (including full prompt inputs, model completions, and Guardrail intervention traces) for **7 years** to comply with SEC Rule 17a-4. The regulation strictly mandates **Write-Once-Read-Many (WORM)** immutability: once a log file is written to storage, **no user—not even an IAM Administrator or the AWS Account Root user—may delete, overwrite, or shorten the retention period of the log object** until the 7-year retention window expires.

How should the Security Architect configure the S3 destination bucket used by `PutModelInvocationLoggingConfiguration`?

- **A.** Enable S3 Versioning and configure an S3 Lifecycle rule to transition objects to S3 Glacier Deep Archive after 30 days.
- **B.** Enable **S3 Object Lock** in **Governance Mode** with a 2,555-day retention period and grant `s3:BypassGovernanceRetention` to the DevOps admin role.
- **C.** Attach an S3 Bucket Policy denying `s3:DeleteObject` to all principals except the AWS Account Root user.
- **D.** Enable **S3 Object Lock** on the S3 bucket configured with a default retention rule in **Compliance Mode** for 2,555 days (7 years) and encrypt the bucket with a KMS Customer Managed Key.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: D**

#### Explanation
**Amazon S3 Object Lock in `COMPLIANCE` mode** enforces strict regulatory WORM immutability (assessed for SEC 17a-4, CFTC, and FINRA compliance). In **Compliance Mode**, a protected object version **cannot be overwritten or deleted by any user, including the AWS Account Root user**, and its retention period cannot be shortened for the duration of the retention window (2,555 days).

#### Distractor Analysis
- **Why A is incorrect:** S3 Versioning and Lifecycle transitions to Glacier do not prevent an administrator or root user from permanently deleting object versions (`DeleteObjectVersion`).
- **Why B is incorrect:** In **Governance Mode**, users with `s3:BypassGovernanceRetention` or the Root user can still delete objects or disable retention, failing SEC 17a-4 strict WORM requirements.
- **Why C is incorrect:** Root users can always modify or delete an S3 Bucket Policy, whereas S3 Object Lock in Compliance Mode cannot be bypassed even by Root.

</details>

---

### Question 10
A multi-division retail conglomerate runs 12 different generative AI applications (customer chat, catalog translation, warehouse routing, supply chain forecasting) inside a single shared AWS production account, all invoking `anthropic.claude-3-5-sonnet-20241022-v2:0` on Amazon Bedrock. At the end of the month, the CFO asks the platform engineering team to implement **granular department-level FinOps cost allocation and token chargeback** so AWS Cost Explorer and AWS Cost and Usage Reports (CUR) show the exact Amazon Bedrock on-demand token spend incurred by each of the 12 applications.

What is the most accurate, AWS-native mechanism to attribute on-demand Bedrock model invocation costs to each individual application within the same AWS account?

- **A.** Pass a custom HTTP header `X-AWS-Cost-Center: 402` in the `Converse` API call, which AWS Cost Explorer automatically parses into billing line items.
- **B.** Create 12 **Application Inference Profiles** (`CreateInferenceProfile`) in Amazon Bedrock—one per application—assign distinct **Cost Allocation Tags** (e.g., `AppId = warehouse-routing`, `CostCenter = 402`) to each Application Inference Profile, activate those tags in AWS Billing & Cost Management, and configure each application to pass its dedicated Application Inference Profile ARN as the `modelId` in `Converse` / `InvokeModel`.
- **C.** Create 12 separate Amazon Bedrock Guardrails with identical rules and rely on Guardrail billing to split the foundation model token charges.
- **D.** Tag the foundation model ARN `arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-5-sonnet-20241022-v2:0` with all 12 Cost Center tags simultaneously.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: B**

#### Explanation
Base Foundation Model ARNs (`arn:aws:bedrock:region::foundation-model/*`) are global AWS-owned resources and cannot be tagged by customers. To track usage metrics and allocate on-demand token costs across multiple applications, teams, or tenants sharing a single AWS account, Amazon Bedrock provides **Application Inference Profiles** (`CreateInferenceProfile`). Each Application Inference Profile wraps a base foundation model (or cross-region system profile), receives its own unique ARN in your account, supports **AWS Cost Allocation Tags**, and emits per-profile CloudWatch token/invocation metrics.

#### Distractor Analysis
- **Why A is incorrect:** AWS Cost Explorer does not split Bedrock charges based on arbitrary custom HTTP headers.
- **Why C is incorrect:** Tagging a Guardrail only allocates the *Guardrail evaluation fee*, not the underlying foundation model's input/output token charges.
- **Why D is incorrect:** Customers cannot tag AWS-owned base `foundation-model` ARNs, and even if a single resource has 12 tags, Cost Explorer cannot split usage across 12 values of the same tag key.

</details>

---

### Question 11
A compliance engineering team uses **AWS Config** to continuously audit the security posture of all generative AI resources deployed across 50 AWS accounts. Specifically, the team must detect within minutes if a developer creates or updates an **Amazon Bedrock Guardrail** (`AWS::Bedrock::Guardrail`) or **Amazon Bedrock Knowledge Base** (`AWS::Bedrock::KnowledgeBase`) that is **not encrypted with an AWS KMS Customer Managed Key (CMK)**, and automatically mark the resource `NON_COMPLIANT` in AWS Security Hub.

Which solution meets this continuous compliance requirement with the lowest operational overhead?

- **A.** Deploy an **AWS Config Custom Policy Rule** (using CloudFormation Guard DSL) or **AWS Config Managed Rule** scoped to resource types `AWS::Bedrock::Guardrail` and `AWS::Bedrock::KnowledgeBase` triggered on **Configuration Changes**, verifying that `kmsKeyArn` (or `serverSideEncryptionConfiguration.kmsKeyArn`) is present and matches the organization's KMS ARN pattern.
- **B.** Write a Python script that runs once a month on a developer's laptop calling `list_guardrails` and emails a spreadsheet.
- **C.** Enable Amazon GuardDuty Malware Protection for S3 to check whether Bedrock Guardrails have a KMS key attached.
- **D.** Use Amazon Comprehend `DetectSyntax` to parse IAM policies every 24 hours.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: A**

#### Explanation
**AWS Config** tracks configuration state changes for supported Amazon Bedrock resources (`AWS::Bedrock::Guardrail`, `AWS::Bedrock::KnowledgeBase`, `AWS::Bedrock::Agent`). Creating a configuration-change-triggered **AWS Config Rule** (written declaratively in CloudFormation Guard DSL without maintaining Lambda code) evaluates the resource's configuration item immediately upon creation or update, verifies that the KMS CMK property is populated, and aggregates compliance status into AWS Security Hub / AWS Config Conformance Packs.

#### Distractor Analysis
- **Why B is incorrect:** Manual monthly scripts on a laptop fail the requirement to detect non-compliant resources within minutes and report to Security Hub.
- **Why C is incorrect:** GuardDuty Malware Protection scans EBS/S3 files for trojans; it does not audit Bedrock resource encryption configurations.
- **Why D is incorrect:** Comprehend `DetectSyntax` performs NLP part-of-speech tagging, not cloud configuration compliance auditing.

</details>

---

### Question 12
An AI Red Team is evaluating a newly fine-tuned customer service sentiment model hosted on Amazon SageMaker before production sign-off. The Red Team wants to run an automated post-training fairness check that takes 10,000 real customer support messages and tests whether swapping only a demographic indicator token in the prompt (for example, changing `"My husband and I..."` to `"My wife and I..."`, or swapping culturally associated first names) causes the model's predicted classification label to flip from `"Priority Escalation"` to `"Standard Queue"`.

Which **Amazon SageMaker Clarify** post-training bias metric directly measures this behavior?

- **A.** Class Imbalance (`CI`)
- **B.** Difference in Proportions of Labels (`DPL`)
- **C.** **Counterfactual Fliptest (`FT`)**
- **D.** Mean Squared Error (`MSE`)

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answer: C**

#### Explanation
The **Counterfactual Fliptest (`FT`)** is a post-training bias metric in **Amazon SageMaker Clarify** that assesses whether similar examples from different demographic facets receive different model predictions. Specifically, it examines members of demographic group $d$ and measures the proportion of instances whose predicted label $\hat{y}$ flips when their protected facet attribute (or k-nearest counterfactual neighbor across facets) is swapped to group $a$.

#### Distractor Analysis
- **Why A is incorrect:** `CI` is a pre-training metric that only counts dataset row frequencies per facet.
- **Why B is incorrect:** `DPL` is a pre-training metric that compares ground-truth labels in the raw dataset without invoking the model.
- **Why D is incorrect:** `MSE` is a general regression accuracy metric, not a counterfactual fairness metric.

</details>

---

### Question 13 *(Multiple Response — Choose TWO)*
A global financial institution is building an automated incident response pipeline for its customer-facing Amazon Bedrock generative AI applications. The CISO requires that whenever an external user attempts a prompt injection attack or submits a prohibited topic that causes an Amazon Bedrock Guardrail to intervene (`stopReason: "guardrail_intervened"`):
1. A real-time CloudWatch Alarm must fire within 60 seconds and trigger an Amazon EventBridge rule to block the offending user's session token in DynamoDB.
2. Security analysts must be able to run ad-hoc SQL queries over 12 months of historical invocation payloads in Amazon S3 to analyze attack trends by Guardrail policy type (`topicPolicy`, `contentPolicy`, `wordPolicy`, `sensitiveInformationPolicy`).

Which **TWO** architectural components should the engineering team implement? *(Choose TWO.)*

- **A.** Enable **Amazon Bedrock Model Invocation Logging** to both **Amazon CloudWatch Logs** and **Amazon S3**, and ensure all application `Converse` / `InvokeModel` calls set `guardrailConfig.trace = "enabled"`.
- **B.** Create a **CloudWatch Logs Metric Filter** matching `$.output.outputBodyJson.stopReason = "guardrail_intervened"` (or use the native `AWS/Bedrock` `GuardrailInterventions` CloudWatch metric) to trigger the CloudWatch Alarm and EventBridge remediation rule, and create an **Amazon Athena** external table over the S3 Model Invocation Logs prefix for 12-month SQL analytics.
- **C.** Enable AWS CloudTrail Management Events only, as Management Events contain the full `trace.guardrail` JSON payload for every blocked prompt.
- **D.** Use Amazon Rekognition Video to scan the S3 bucket for `guardrail_intervened` strings.
- **E.** Store the invocation logs exclusively in an ephemeral EC2 instance store volume.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answers: A and B**

#### Explanation
- **A is correct:** Delivering **Bedrock Model Invocation Logs** to both **CloudWatch Logs** (for sub-minute real-time streaming inspection) and **Amazon S3** (for low-cost historical retention) with `trace: "enabled"` captures the full input/output payloads and the detailed Guardrail assessment trace showing exactly which filter intervened.
- **B is correct:** A **CloudWatch Logs Metric Filter** (or the `GuardrailInterventions` metric in `AWS/Bedrock`) triggers a CloudWatch Alarm + EventBridge rule within 60 seconds, while **Amazon Athena** queries the JSON/GZ invocation log files directly in S3 using standard SQL.

#### Distractor Analysis
- **Why C is incorrect:** CloudTrail Management Events do not record `InvokeModel` / `Converse` calls or Guardrail traces.
- **Why D is incorrect:** Rekognition Video analyzes video streams, not JSON log files.
- **Why E is incorrect:** Ephemeral EC2 instance stores lose all data when stopped and do not receive Bedrock Model Invocation Logs.

</details>

---

### Question 14 *(Multiple Response — Choose TWO)*
A pharmaceutical enterprise trains custom bio-sequence and clinical NLP models using **Amazon SageMaker Pipelines**. During an FDA inspection, regulators select a specific clinical prediction made in production 8 months ago and ask the engineering team to prove the exact **end-to-end provenance and lineage** of the model that generated it—including the exact S3 URI and ETag/version of the training dataset, the preprocessing container image SHA-256 digest, the evaluation accuracy report, and the human governance sign-off record.

Which **TWO** AWS capabilities should the architecture use together to maintain this end-to-end audit trail automatically? *(Choose TWO.)*

- **A.** Amazon ElastiCache for Memcached with a 1-hour TTL to store training dataset URIs.
- **B.** **Amazon SageMaker ML Lineage Tracking**, which automatically records `TrialComponent`, `Artifact` (S3 datasets, container images, model weights), `Action`, and `Context` entities across SageMaker Pipelines steps and supports `QueryLineage` graph traversal.
- **C.** Amazon Route 53 Health Checks to track container SHA-256 digests over 8 months.
- **D.** **Amazon SageMaker Model Registry** paired with **SageMaker Model Cards**, linking each versioned `ModelPackage` to its pipeline lineage graph, evaluation metrics, and formal approval status (`Approved`).
- **E.** Disabling S3 Bucket Versioning on the training dataset bucket so only the latest file exists.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answers: B and D**

#### Explanation
- **B is correct:** **Amazon SageMaker ML Lineage Tracking** automatically builds and persists a queryable lineage graph (`QueryLineage`) linking raw S3 dataset artifacts, preprocessing jobs, container image digests, training jobs, and deployed endpoints.
- **D is correct:** **SageMaker Model Registry** versions every candidate model (`ModelPackageGroup`) alongside its evaluation metrics and deployment approval status, while **SageMaker Model Cards** package the lineage, risk profile, and governance sign-off for regulatory audits.

#### Distractor Analysis
- **Why A is incorrect:** Memcached is an ephemeral in-memory cache, not a durable regulatory lineage store.
- **Why C is incorrect:** Route 53 Health Checks monitor endpoint DNS availability, not ML artifact lineage.
- **Why E is incorrect:** Disabling S3 Versioning destroys historical dataset reproducibility if a training file is overwritten.

</details>

---

### Question 15 *(Multiple Response — Choose TWO)*
A public-sector agency is deploying a citizen benefits eligibility assistant using **Amazon Bedrock** and **Amazon Bedrock Guardrails**. To comply with federal Responsible AI and Algorithmic Accountability mandates, the agency must:
1. Ensure that any eligibility rule change in the statutory policy manual is **mathematically verified** against the LLM's outputs at runtime using formal logic, and that responses where the user did not supply enough personal eligibility variables (`SATISFIABLE`) or where the Guardrail grounding score is borderline are routed to a **human caseworker** via Amazon A2I before a final denial letter is issued.
2. Prevent any agency developer in the production AWS account from deleting the Bedrock Guardrail or disabling Bedrock Model Invocation Logging.

Which **TWO** architectural controls should the agency implement? *(Choose TWO.)*

- **A.** Configure the LLM `temperature` to `1.0` so the model randomly approves borderline citizen benefit requests.
- **B.** Use Amazon Titan Image Generator watermarking to verify text eligibility letters.
- **C.** Configure **Automated Reasoning Checks** and **Contextual Grounding Checks** on the Amazon Bedrock Guardrail, inspect the `trace.guardrail` verdict in an AWS Step Functions workflow, and invoke **Amazon A2I (`StartHumanLoop`)** whenever a denial recommendation has a borderline grounding score or `SATISFIABLE` reasoning status.
- **D.** Grant `bedrock:*` with `Effect: "Allow"` to all developers and rely on a Slack channel reminder not to delete the Guardrail.
- **E.** Apply an **AWS Organizations Service Control Policy (SCP)** (or IAM Permission Boundary) explicitly denying `bedrock:DeleteGuardrail`, `bedrock:UpdateGuardrail`, and `bedrock:DeleteModelInvocationLoggingConfiguration` for all developer principals.

<details>
<summary><strong>Reveal Answer & Detailed Architectural Analysis</strong></summary>

**Correct Answers: C and E**

#### Explanation
- **C is correct:** Combining **Automated Reasoning Checks** (for formal logical rule verification) and **Contextual Grounding Checks** with `trace: "enabled"` allows an AWS Step Functions orchestrator to intercept `SATISFIABLE` or borderline-grounded benefit determinations and escalate them to a human caseworker via **Amazon A2I (`StartHumanLoop`)**.
- **E is correct:** An **AWS Organizations Service Control Policy (SCP)** with an explicit `Deny` on `bedrock:DeleteGuardrail`, `bedrock:UpdateGuardrail`, and `bedrock:DeleteModelInvocationLoggingConfiguration` prevents developers from tampering with safety guardrails or compliance logging.

#### Distractor Analysis
- **Why A is incorrect:** Setting `temperature: 1.0` introduces random non-deterministic hallucinations in statutory benefit decisions.
- **Why B is incorrect:** Titan Image Generator watermarking applies to generated images, not text eligibility letters.
- **Why D is incorrect:** Relying on informal reminders violates preventive security governance.

</details>
