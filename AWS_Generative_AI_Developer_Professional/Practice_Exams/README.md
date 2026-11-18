# AWS Certified Generative AI Developer – Professional (`AIP-C01`) Practice Exams

This directory contains three full-length, scenario-based practice examinations (75 questions total — equivalent to a complete 180-minute official `AIP-C01` exam sitting) engineered to mirror the multi-paragraph narrative depth, distractor subtlety, and multi-service architectural integration of the official **AWS Certified Generative AI Developer – Professional (`AIP-C01`)** certification.

---

## 1. Practice Exam Directory & Links

| Exam | Focus & Scope | Questions | Recommended Time Limit | Passing Target | Link |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **Practice Exam 1** | Hybrid RAG + Reranking, Hierarchical Chunking, Multi-Agent Collaboration, Return of Control (ROC), MCP, Guardrails Contextual Grounding & Automated Reasoning, Prompt Caching (`cachePoint`), Agent Trace Debugging | **25** | **60 Minutes** | **$\ge 20 / 25$ (80%)** | [**Open `Practice_Exam_1.md`**](Practice_Exam_1.md) |
| **Practice Exam 2** | Multimodal RAG (Bedrock Data Automation, Nova/Marengo), S3 Vectors vs OpenSearch vs MemoryDB, HyDE, Agent Cross-Session Memory, SageMaker Async & Inference Components, Streaming Guardrails, Intelligent Prompt Routing, LLM-as-a-Judge | **25** | **60 Minutes** | **$\ge 20 / 25$ (80%)** | [**Open `Practice_Exam_2.md`**](Practice_Exam_2.md) |
| **Practice Exam 3** | Cross-Account Lake Formation RAG, Custom Lambda Chunking, `pgvector` HNSW Tuning, Custom Agent Orchestration Templates, SageMaker LMI vLLM Speculative Decoding, Standalone `ApplyGuardrail`, OpenTelemetry GenAI & EMF Chargeback, CI/CD Evaluation Gates | **25** | **60 Minutes** | **$\ge 20 / 25$ (80%)** | [**Open `Practice_Exam_3.md`**](Practice_Exam_3.md) |

---

## 2. Domain Distribution Across Practice Exams

All practice exams are balanced to reflect the official `AIP-C01` exam blueprint percentages across the 75-question suite:

| Domain | Official Weight | Questions Across Suite | Key Competencies Tested |
| :--- | :---: | :---: | :--- |
| **Domain 1: Foundation Model Integration, Data Management, and Compliance** | **31%** | 24 / 75 | FM selection & customization (Fine-Tuning, Continued Pre-Training, Model Distillation), Bedrock Data Automation, hierarchical/semantic chunking, vector stores (OpenSearch Serverless, Aurora `pgvector`, S3 Vectors, MemoryDB, Neptune Analytics GraphRAG), Hybrid Search + Reranking, Prompt Management & Prompt Flows. |
| **Domain 2: Implementation and Integration** | **26%** | 19 / 75 | Bedrock Agents, Multi-Agent Collaboration (Supervisor vs Routing), Action Groups, Return of Control (ROC), Model Context Protocol (MCP), Custom Model Import, SageMaker LMI / Inference Components, Step Functions Distributed Map, `ConverseStream`, AppSync WebSockets, API resiliency. |
| **Domain 3: AI Safety, Security, and Governance** | **20%** | 15 / 75 | Bedrock Guardrails (Content, Denied Topics, PII Anonymization, Prompt Attack, Contextual Grounding, Automated Reasoning, `ApplyGuardrail`), Multi-Tenant RAG isolation, VPC PrivateLink, KMS CMKs, Model Invocation Logging, SageMaker Clarify, Titan Watermarking, Amazon A2I HITL. |
| **Domain 4: Operational Efficiency and Optimization** | **12%** | 9 / 75 | Prompt Caching (`cachePoint`), Semantic Caching, Intelligent Prompt Routing, Batch Inference (50% savings), Provisioned Throughput Model Units (MU) math, Latency-Optimized Inference, Cross-Region Inference (`us.`/`eu.`/`apac.`), Application Inference Profiles & EMF FinOps chargeback. |
| **Domain 5: Testing, Validation, and Troubleshooting** | **11%** | 8 / 75 | Bedrock Model & RAG Evaluations (Automatic, LLM-as-a-Judge, Context Relevance, Groundedness, Completeness), Ground Truth human evaluation, CI/CD regression gates, Agent Trace (`enableTrace=True`) diagnostics, RAG chunking/ingestion debugging, `Converse` API exception troubleshooting. |

---

## 3. How to Use These Practice Exams Effectively

1. **Simulate Real Exam Pacing**: You have 180 minutes for 75 questions on the actual `AIP-C01` exam (**2.4 minutes / 144 seconds per question**). Complete each 25-question exam in a single uninterrupted 60-minute block, or take all three back-to-back for a full 3-hour simulation.
2. **Audit Why Every Distractor Fails**: On `AIP-C01`, multiple options often look plausible at first glance, but contain a fatal flaw (e.g., placing dynamic user messages *before* the `cachePoint` block, relying on post-filtering instead of pre-filtering for multi-tenant RAG security, or hitting API Gateway's 29-second synchronous timeout instead of using AppSync/Lambda Response Streaming). Read the **Distractor Analysis** for every single question—even those you answered correctly.
3. **Cross-Reference Weak Domains**:
   - Missed questions in **Domain 1** $\rightarrow$ Review [Domain 1 Study Guide](../Domain_1_FM_Integration_Data_Management_Compliance/Study_Guide.md) & [Exercises](../Domain_1_FM_Integration_Data_Management_Compliance/Exercises.md)
   - Missed questions in **Domain 2** $\rightarrow$ Review [Domain 2 Study Guide](../Domain_2_Implementation_Integration/Study_Guide.md) & [Exercises](../Domain_2_Implementation_Integration/Exercises.md)
   - Missed questions in **Domain 3** $\rightarrow$ Review [Domain 3 Study Guide](../Domain_3_AI_Safety_Security_Governance/Study_Guide.md) & [Exercises](../Domain_3_AI_Safety_Security_Governance/Exercises.md)
   - Missed questions in **Domain 4** $\rightarrow$ Review [Domain 4 Study Guide](../Domain_4_Operational_Efficiency_Optimization/Study_Guide.md) & [Exercises](../Domain_4_Operational_Efficiency_Optimization/Exercises.md)
   - Missed questions in **Domain 5** $\rightarrow$ Review [Domain 5 Study Guide](../Domain_5_Testing_Validation_Troubleshooting/Study_Guide.md) & [Exercises](../Domain_5_Testing_Validation_Troubleshooting/Exercises.md)
   - Review rapid decision matrices, token math, and top exam traps in the [AIP-C01 Master Cheat Sheet](../AIP_C01_Master_Cheat_Sheet.md).
