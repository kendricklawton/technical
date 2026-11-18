# AWS Certified Generative AI Developer – Professional (AIP-C01) Study & Architecture Workspace

Welcome to the comprehensive study, architecture design, and exam preparation workspace for the **AWS Certified Generative AI Developer – Professional (`AIP-C01`)** certification. This track is structured directly around the official AWS `AIP-C01` Exam Guide and engineered for Staff/Principal AI Engineers, Cloud Architects, and GenAI Platform Developers building production-grade foundation model pipelines, multi-agent systems, enterprise RAG architectures, guardrails, and LLMOps evaluation frameworks on AWS.

---

## 1. Exam Overview & Format

| Attribute | Details |
| :--- | :--- |
| **Exam Code** | `AIP-C01` |
| **Number of Questions** | **75 questions** (65 scored questions + 10 unscored pretest questions) |
| **Question Formats** | **Multiple Choice** (1 correct out of 4) and **Multiple Response** (2 or 3 correct out of 5 or 6) |
| **Allotted Time** | **180 minutes** (3 hours) — average **2.4 minutes (144 seconds) per question** |
| **Passing Score** | **750 / 1000** (scaled score; no penalty for guessing) |
| **Exam Fee** | **\$300 USD** |
| **Target Audience** | 2+ years of hands-on experience designing, developing, securing, optimizing, and operating production Generative AI applications on AWS |

---

## 2. Official AIP-C01 Exam Blueprint & Domain Weights

The `AIP-C01` exam is divided into five core architectural and engineering domains:

| Domain | Title | Exam Weight | Approx. Scored Questions | Core Focus Areas |
| :--- | :--- | :---: | :---: | :--- |
| **Domain 1** | **Foundation Model Integration, Data Management, and Compliance** | **31%** | ~20 | Prompt Engineering vs RAG vs Fine-Tuning vs Continued Pre-Training vs Model Distillation; Bedrock vs SageMaker JumpStart; Bedrock Data Automation (BDA); Hierarchical, Semantic & Custom Lambda chunking; Vector Stores (OpenSearch Serverless, Aurora `pgvector` Optimized Reads, S3 Vectors, MemoryDB, Neptune Analytics GraphRAG); Hybrid Search (BM25 + k-NN), Reranking (`Cohere Rerank 3.5`), Metadata Pre-Filtering, HyDE; Bedrock Prompt Management & Prompt Flows. |
| **Domain 2** | **Implementation and Integration** | **26%** | ~17 | Amazon Bedrock Agents (ReAct, custom orchestration prompts, Action Groups, OpenAPI schemas, Return of Control [ROC], cross-session Memory, Code Interpreter); Multi-Agent Collaboration (Supervisor vs Routing mode); Model Context Protocol (MCP) servers on AWS; Custom Model Import; SageMaker LMI (vLLM/TensorRT-LLM), Inference Components (Multi-Adapter LoRA), Async & Serverless Inference; Step Functions Distributed Map; `Converse` / `ConverseStream` unified APIs, native Tool Use (`toolConfig`), AppSync WebSockets streaming, adaptive retry & cross-region failover. |
| **Domain 3** | **AI Safety, Security, and Governance** | **20%** | ~13 | Amazon Bedrock Guardrails (Content Filters, Denied Topics, Word Filters, Sensitive Info PII Anonymization/Regex, Prompt Attack Defense, Contextual Grounding Checks, Automated Reasoning formal verification, Streaming Guardrails, standalone `ApplyGuardrail` API); indirect prompt injection defense (`guardContent`); Multi-Tenant RAG isolation (JWT claims to mandatory metadata pre-filters); VPC PrivateLink & Endpoint Policies; KMS CMKs; Model Invocation Logging (S3 Object Lock / CloudWatch); CloudTrail Data Events; SageMaker Clarify bias/SHAP; Titan Watermarking; Amazon A2I HITL. |
| **Domain 4** | **Operational Efficiency and Optimization for GenAI Applications** | **12%** | ~8 | Bedrock Prompt Caching (`cachePoint` prefix ordering & TTL math); Semantic Caching (MemoryDB / ElastiCache vector search); Intelligent Prompt Routing; Batch Inference (50% cost reduction); Provisioned Throughput Model Units (MUs) break-even math; Latency-Optimized Inference (`performanceConfig`); Cross-Region Inference (`us.`, `eu.`, `apac.` profiles); TTFT vs TPS optimization; CloudWatch Bedrock metrics, OpenTelemetry GenAI tracing, Application Inference Profiles & Cost Allocation Tags for multi-tenant FinOps chargeback. |
| **Domain 5** | **Testing, Validation, and Troubleshooting** | **11%** | ~7 | Amazon Bedrock Evaluations (Automatic programmatic metrics, LLM-as-a-Judge, Custom metrics, RAG Knowledge Base Evaluations for Context Relevance, Context Coverage, Groundedness/Faithfulness, Correctness, Completeness); SageMaker Ground Truth human evaluation; LLMOps CI/CD regression gates; Bedrock Agent Trace debugging (`enableTrace=True`: `preProcessingTrace`, `orchestrationTrace`, `guardrailTrace`, `failureTrace`); troubleshooting RAG retrieval/chunking/ingestion failures, OpenSearch Data Access & KMS policies, and `Converse` API exceptions. |

---

## 3. Workspace Navigation Directory

### Global Reference
* [**AIP-C01 Master Cheat Sheet (`AIP_C01_Master_Cheat_Sheet.md`)**](AIP_C01_Master_Cheat_Sheet.md) — High-density FM customization decision matrices, Vector Store architectural comparison tables, Chunking & RAG optimization guide, Bedrock Agent & MCP payload schemas, Guardrails enforcement matrix, Token FinOps & Prompt Caching math, CloudWatch metrics & exception codes, and Top 45 Exam Triggers & Traps.

### Domain 1: Foundation Model Integration, Data Management, and Compliance (31%)
* [**Domain 1 Study Guide**](Domain_1_FM_Integration_Data_Management_Compliance/Study_Guide.md)
* [**Domain 1 White Papers & Architecture Guide**](Domain_1_FM_Integration_Data_Management_Compliance/White_Papers.md) — Deep dive into FM Customization vs RAG, Multimodal Ingestion with Bedrock Data Automation, Vector Database Engine Internals, Hybrid Search + Reranking, and GraphRAG.
* [**Domain 1 Architecture Exercises**](Domain_1_FM_Integration_Data_Management_Compliance/Exercises.md) — 6 hands-on production labs (Hierarchical Chunking + Custom Lambda Knowledge Base, Hybrid Search + `Rerank` + `Converse` with Metadata Pre-Filtering, Aurora `pgvector` HNSW Tuning, Neptune Analytics GraphRAG, Bedrock Model Distillation Pipeline, Prompt Management & Prompt Flows).
* [**Domain 1 Quiz 1: FM Selection, Customization (Fine-Tuning, CPT, Distillation) & Multimodal Data Pipelines**](Domain_1_FM_Integration_Data_Management_Compliance/Quiz_1.md) (15 Questions)
* [**Domain 1 Quiz 2: Vector Store Architectures (OpenSearch, Aurora `pgvector`, S3 Vectors, MemoryDB, Neptune) & Chunking**](Domain_1_FM_Integration_Data_Management_Compliance/Quiz_2.md) (15 Questions)
* [**Domain 1 Quiz 3: Advanced Retrieval (Hybrid Search, Reranking, Pre-Filtering, GraphRAG) & Prompt Governance**](Domain_1_FM_Integration_Data_Management_Compliance/Quiz_3.md) (15 Questions)

### Domain 2: Implementation and Integration (26%)
* [**Domain 2 Study Guide**](Domain_2_Implementation_Integration/Study_Guide.md)
* [**Domain 2 White Papers & Architecture Guide**](Domain_2_Implementation_Integration/White_Papers.md) — Deep dive into Multi-Agent Collaboration, Return of Control (ROC), Model Context Protocol (MCP) on AWS, SageMaker LMI/Inference Components, Event-Driven Step Functions GenAI Pipelines, and Real-Time Token Streaming.
* [**Domain 2 Architecture Exercises**](Domain_2_Implementation_Integration/Exercises.md) — 6 hands-on production labs (Bedrock Multi-Agent Supervisor Collaboration, Action Group OpenAPI + Lambda + ROC Confirmation, MCP Server Integrated with `ConverseStream`, Resilient Multi-Region `ConverseStream` Client, Step Functions Distributed Map + Batch Inference, SageMaker Multi-LoRA Inference Components).
* [**Domain 2 Quiz 1: Agentic AI, Multi-Agent Collaboration, Action Groups, ROC, Memory & MCP**](Domain_2_Implementation_Integration/Quiz_1.md) (15 Questions)
* [**Domain 2 Quiz 2: Model Deployment Strategies (Bedrock Custom Model Import, Provisioned Throughput, Batch vs SageMaker LMI)**](Domain_2_Implementation_Integration/Quiz_2.md) (15 Questions)
* [**Domain 2 Quiz 3: Enterprise Event-Driven GenAI, Streaming APIs (`ConverseStream`, AppSync), Tool Use & IaC**](Domain_2_Implementation_Integration/Quiz_3.md) (15 Questions)

### Domain 3: AI Safety, Security, and Governance (20%)
* [**Domain 3 Study Guide**](Domain_3_AI_Safety_Security_Governance/Study_Guide.md)
* [**Domain 3 White Papers & Architecture Guide**](Domain_3_AI_Safety_Security_Governance/White_Papers.md) — Deep dive into AWS Security Reference Architecture for GenAI, OWASP Top 10 for LLMs Mapping, Bedrock Guardrails Internals, Automated Reasoning Checks, and Multi-Tenant RAG Isolation.
* [**Domain 3 Architecture Exercises**](Domain_3_AI_Safety_Security_Governance/Exercises.md) — 6 hands-on production labs (Bedrock Guardrails with Contextual Grounding & PII Regex, Indirect Prompt Injection Defense with `guardContent` & `ApplyGuardrail`, Multi-Tenant RAG Isolation with Cognito JWT Claims, Zero-Trust PrivateLink + KMS CMK + SCP Perimeter, Immutable Model Invocation Logging with S3 Object Lock, Step Functions + Amazon A2I HITL Escalation).
* [**Domain 3 Quiz 1: Bedrock Guardrails, Prompt Injection Defense, Contextual Grounding, Automated Reasoning & `ApplyGuardrail`**](Domain_3_AI_Safety_Security_Governance/Quiz_1.md) (15 Questions)
* [**Domain 3 Quiz 2: Data Security, PII Redaction, KMS CMKs, VPC PrivateLink, IAM Least Privilege & Multi-Tenant RAG**](Domain_3_AI_Safety_Security_Governance/Quiz_2.md) (15 Questions)
* [**Domain 3 Quiz 3: AI Governance, Model Invocation Logging, CloudTrail, SCPs, SageMaker Clarify, Watermarking & A2I**](Domain_3_AI_Safety_Security_Governance/Quiz_3.md) (15 Questions)

### Domain 4: Operational Efficiency and Optimization for GenAI Applications (12%)
* [**Domain 4 Study Guide**](Domain_4_Operational_Efficiency_Optimization/Study_Guide.md)
* [**Domain 4 White Papers & Architecture Guide**](Domain_4_Operational_Efficiency_Optimization/White_Papers.md) — Deep dive into Well-Architected GenAI Lens Cost & Performance Pillars, Prefix Prompt Caching vs Semantic Caching, Cross-Region Inference, and Multi-Tenant FinOps Chargeback.
* [**Domain 4 Architecture Exercises**](Domain_4_Operational_Efficiency_Optimization/Exercises.md) — 6 hands-on production labs (Bedrock Prompt Caching `cachePoint` Ordering, Sub-15ms Semantic Cache with MemoryDB Vector Search, Intelligent Prompt Routing + Latency-Optimized Cross-Region Inference, Bedrock Batch Inference S3 Pipeline, Multi-Tenant FinOps Chargeback with Application Inference Profiles & CloudWatch EMF, CloudWatch Anomaly & Token Burn-Rate Alarms).
* [**Domain 4 Quiz 1: Cost Optimization — Prompt Caching (`cachePoint`), Semantic Caching, Intelligent Routing & Batch/PT Math**](Domain_4_Operational_Efficiency_Optimization/Quiz_1.md) (15 Questions)
* [**Domain 4 Quiz 2: Performance Optimization — TTFT vs TPS, Latency-Optimized Inference, Cross-Region Profiles & SageMaker LMI**](Domain_4_Operational_Efficiency_Optimization/Quiz_2.md) (15 Questions)
* [**Domain 4 Quiz 3: GenAI Observability, CloudWatch Metrics, OpenTelemetry Tracing & Multi-Tenant FinOps Chargeback**](Domain_4_Operational_Efficiency_Optimization/Quiz_3.md) (15 Questions)

### Domain 5: Testing, Validation, and Troubleshooting (11%)
* [**Domain 5 Study Guide**](Domain_5_Testing_Validation_Troubleshooting/Study_Guide.md)
* [**Domain 5 White Papers & Architecture Guide**](Domain_5_Testing_Validation_Troubleshooting/White_Papers.md) — Deep dive into LLM-as-a-Judge Evaluation Frameworks, RAG Triad Diagnostics (Context Relevance, Groundedness, Answer Relevance), Agent Trace Forensics, and Production Troubleshooting Runbooks.
* [**Domain 5 Architecture Exercises**](Domain_5_Testing_Validation_Troubleshooting/Exercises.md) — 6 hands-on production labs (Automated RAG Knowledge Base Evaluation Job, Custom LLM-as-a-Judge Metric Pipeline, CI/CD LLMOps Regression Gate in CodeBuild, Programmatic Bedrock Agent Trace Debugger, Fixing Multi-Table Financial PDF RAG Recall & Groundedness, Diagnosing & Fixing OpenSearch Serverless Ingestion Failures).
* [**Domain 5 Quiz 1: Bedrock Evaluations (Automatic, LLM-as-a-Judge, RAG Metrics, Ground Truth & CI/CD Regression Gates)**](Domain_5_Testing_Validation_Troubleshooting/Quiz_1.md) (15 Questions)
* [**Domain 5 Quiz 2: Troubleshooting RAG Pipelines, Vector Stores, Chunking Boundaries, Hallucinations & Ingestion Errors**](Domain_5_Testing_Validation_Troubleshooting/Quiz_2.md) (15 Questions)
* [**Domain 5 Quiz 3: Troubleshooting Bedrock Agents (`enableTrace`), Multi-Agent Routing, Action Group Errors & API Exceptions**](Domain_5_Testing_Validation_Troubleshooting/Quiz_3.md) (15 Questions)

### Full-Length Practice Exams (75 Questions Total — Complete Exam Sitting)
* [**Practice Exams Guide (`Practice_Exams/README.md`)**](Practice_Exams/README.md) — Exam simulation guidelines, pacing strategies, and domain scoring rubric.
* [**Practice Exam 1 (`Practice_Exams/Practice_Exam_1.md`)**](Practice_Exams/Practice_Exam_1.md) — 25 full-length `AIP-C01` scenario questions across all 5 domains.
* [**Practice Exam 2 (`Practice_Exams/Practice_Exam_2.md`)**](Practice_Exams/Practice_Exam_2.md) — 25 full-length `AIP-C01` scenario questions across all 5 domains.
* [**Practice Exam 3 (`Practice_Exams/Practice_Exam_3.md`)**](Practice_Exams/Practice_Exam_3.md) — 25 full-length `AIP-C01` scenario questions across all 5 domains.
* **Total Track Question Bank:** **300 scenario questions** (225 domain quiz questions across 15 quizzes + 75 full-length practice exam questions with complete distractor analyses).

---

## 4. AIP-C01 Exam Strategy & Architectural Heuristics

1. **Decode the Primary Optimization Constraint First**:
   * *"Need up-to-date facts from frequently changing documents with source citations and tenant access control"* $\rightarrow$ **RAG (Amazon Bedrock Knowledge Bases)**, NEVER Fine-Tuning or Continued Pre-Training.
   * *"Need the model to learn proprietary domain jargon/unlabeled corpus terminology"* $\rightarrow$ **Continued Pre-Training (CPT)**.
   * *"Need the model to adopt a specific tone, classification behavior, or output format, or transfer capability from a large teacher FM to a low-latency student FM"* $\rightarrow$ **Instruction Fine-Tuning** or **Bedrock Model Distillation**.
   * *"Need to process 500,000 non-urgent night-time summarization prompts at the lowest cost"* $\rightarrow$ **Amazon Bedrock Batch Inference** (50% discount over On-Demand and bypasses real-time RPM/TPM quotas).
2. **Watch for Multi-Tenant & Security Disqualifiers**:
   * Never filter retrieved RAG chunks in application code *after* vector search (`post-filtering`) for multi-tenant isolation—a malicious query or top-$k$ saturation can either leak metadata or starve legitimate results. Always inject authenticated tenant claims (from Cognito/IdP JWT inside a backend Lambda/Authorizer, never trusting client-supplied headers) into **Knowledge Base Metadata Pre-Filtering (`retrievalConfiguration.vectorSearchConfiguration.filter`)**.
   * When protecting against indirect prompt injection inside retrieved RAG documents, wrap untrusted document chunks inside **`guardContent` blocks** in the `Converse` API and enable **Prompt Attack** + **Contextual Grounding** in **Amazon Bedrock Guardrails**.
   * When enforcing Bedrock Guardrails on models hosted *outside* Bedrock (e.g., SageMaker JumpStart endpoints, EKS vLLM clusters, or third-party APIs), invoke the standalone **`ApplyGuardrail`** API directly on inputs and outputs.
3. **Master Prompt Caching (`cachePoint`) & Token Latency Mechanics**:
   * **Prompt Caching (`cachePoint`)** requires an **exact static prefix match**. Always order prompt elements as: `System Prompt` $\rightarrow$ `Tool Definitions` $\rightarrow$ `Static Context / Reference Docs` $\rightarrow$ `cachePoint` $\rightarrow$ `Dynamic User History & Query`. Placing a dynamic timestamp or user ID at the top of the system prompt invalidates the entire cache on every call!
   * To stream LLM tokens to browser clients when generations exceed **29 seconds** (API Gateway REST synchronous limit), use **AWS AppSync (WebSockets / Event APIs)**, **Lambda Function URLs with `RESPONSE_STREAM`**, or **API Gateway REST Response Streaming**.

---

## 5. 4-Week Study Plan & Progress Tracker

### Week 1: Foundation Model Integration, Data Pipelines, Vector Stores & RAG (Domain 1 - 31%)
- [ ] Read [Domain 1 Study Guide](Domain_1_FM_Integration_Data_Management_Compliance/Study_Guide.md)
- [ ] Review [Domain 1 White Papers & Documentation](Domain_1_FM_Integration_Data_Management_Compliance/White_Papers.md)
- [ ] Complete [Domain 1 Architecture Exercises (Exercises 1–6)](Domain_1_FM_Integration_Data_Management_Compliance/Exercises.md)
- [ ] Score $\ge 80\%$ on [Domain 1 Quiz 1 (FM Selection, Customization & Multimodal Pipelines)](Domain_1_FM_Integration_Data_Management_Compliance/Quiz_1.md)
- [ ] Score $\ge 80\%$ on [Domain 1 Quiz 2 (Vector Stores & Chunking Strategies)](Domain_1_FM_Integration_Data_Management_Compliance/Quiz_2.md)
- [ ] Score $\ge 80\%$ on [Domain 1 Quiz 3 (Hybrid Retrieval, Reranking & Prompt Governance)](Domain_1_FM_Integration_Data_Management_Compliance/Quiz_3.md)

### Week 2: Agentic AI, MCP, Model Deployment & Enterprise Integration (Domain 2 - 26%)
- [ ] Read [Domain 2 Study Guide](Domain_2_Implementation_Integration/Study_Guide.md)
- [ ] Review [Domain 2 White Papers & Documentation](Domain_2_Implementation_Integration/White_Papers.md)
- [ ] Complete [Domain 2 Architecture Exercises (Exercises 1–6)](Domain_2_Implementation_Integration/Exercises.md)
- [ ] Score $\ge 80\%$ on [Domain 2 Quiz 1 (Bedrock Agents, Multi-Agent Collaboration, ROC & MCP)](Domain_2_Implementation_Integration/Quiz_1.md)
- [ ] Score $\ge 80\%$ on [Domain 2 Quiz 2 (Bedrock & SageMaker Model Deployment Strategies)](Domain_2_Implementation_Integration/Quiz_2.md)
- [ ] Score $\ge 80\%$ on [Domain 2 Quiz 3 (Event-Driven GenAI, Streaming APIs & Resiliency)](Domain_2_Implementation_Integration/Quiz_3.md)

### Week 3: AI Safety, Guardrails, Security, Governance & FinOps/Latency Optimization (Domains 3 & 4 - 32%)
- [ ] Read [Domain 3 Study Guide](Domain_3_AI_Safety_Security_Governance/Study_Guide.md) & [White Papers](Domain_3_AI_Safety_Security_Governance/White_Papers.md)
- [ ] Complete [Domain 3 Architecture Exercises (Exercises 1–6)](Domain_3_AI_Safety_Security_Governance/Exercises.md)
- [ ] Score $\ge 80\%$ on [Domain 3 Quizzes 1, 2 & 3](Domain_3_AI_Safety_Security_Governance/Quiz_1.md)
- [ ] Read [Domain 4 Study Guide](Domain_4_Operational_Efficiency_Optimization/Study_Guide.md) & [White Papers](Domain_4_Operational_Efficiency_Optimization/White_Papers.md)
- [ ] Complete [Domain 4 Architecture Exercises (Exercises 1–6)](Domain_4_Operational_Efficiency_Optimization/Exercises.md)
- [ ] Score $\ge 80\%$ on [Domain 4 Quizzes 1, 2 & 3](Domain_4_Operational_Efficiency_Optimization/Quiz_1.md)

### Week 4: Evaluation Systems, Troubleshooting & Full-Length Practice Exams (Domain 5 - 11% + Exams)
- [ ] Read [Domain 5 Study Guide](Domain_5_Testing_Validation_Troubleshooting/Study_Guide.md) & [White Papers](Domain_5_Testing_Validation_Troubleshooting/White_Papers.md)
- [ ] Complete [Domain 5 Architecture Exercises (Exercises 1–6)](Domain_5_Testing_Validation_Troubleshooting/Exercises.md)
- [ ] Score $\ge 80\%$ on [Domain 5 Quizzes 1, 2 & 3](Domain_5_Testing_Validation_Troubleshooting/Quiz_1.md)
- [ ] Review [AIP-C01 Master Cheat Sheet](AIP_C01_Master_Cheat_Sheet.md)
- [ ] Complete [Practice Exam 1 (Timed: 60 mins)](Practice_Exams/Practice_Exam_1.md) — Target: $\ge 20 / 25$ ($80\%$)
- [ ] Complete [Practice Exam 2 (Timed: 60 mins)](Practice_Exams/Practice_Exam_2.md) — Target: $\ge 20 / 25$ ($80\%$)
- [ ] Complete [Practice Exam 3 (Timed: 60 mins)](Practice_Exams/Practice_Exam_3.md) — Target: $\ge 20 / 25$ ($80\%$)
