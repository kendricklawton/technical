# Domain 5 — Quiz 1: Amazon Bedrock Evaluations, RAG Metrics, Human Ground Truth & CI/CD Gates (Task 5.1)

---

### Question 1
A healthcare analytics company is migrating its clinical discharge summarization pipeline from `anthropic.claude-3-sonnet-20240229-v1:0` to `amazon.nova-pro-v1:0` to reduce inference latency and cost. During automated pre-production testing on a 400-sample golden dataset using Amazon Bedrock Evaluations, the team runs programmatic `Builtin.ROUGE` (`ROUGE-L`) and `Builtin.BERTScore` evaluations. Both models achieve a `ROUGE-L` score of `0.84` and a `BERTScore` of `0.91`. However, during spot checks, clinical reviewers discover that 7% of the candidate summaries invert medication instructions (for example, generating `"continue Metformin 500mg"` when the discharge note states `"discontinue Metformin 500mg due to renal impairment"`), which barely changes the n-gram overlap score.

Which evaluation architecture should the GenAI developer implement to catch these factual contradictions automatically in the CI/CD pipeline before promoting the model migration?

- **A.** Switch the automatic programmatic evaluation metric from `Builtin.ROUGE` to `Builtin.BLEU` and `Builtin.ExactMatch` while increasing the golden dataset size from 400 prompts to 1,000 prompts in Amazon S3.
- **B.** Configure an Amazon Bedrock Model Evaluation job using `Builtin.Robustness` and `Builtin.Toxicity` so the evaluation framework automatically perturbs medication names and flags unsafe clinical strings.
- **C.** Configure an Amazon Bedrock Model Evaluation job using LLM-as-a-Judge (`anthropic.claude-3-5-sonnet-20241022-v2:0`) with `Builtin.Faithfulness` and `Builtin.Correctness`, plus a Custom LLM-as-a-Judge metric (`customMetricConfig`) containing explicit instructions to penalize medication negation or dosage inversions with a score of `0.0`.
- **D.** Use `amazon.nova-pro-v1:0` as both the generator model and the LLM-as-a-Judge evaluator model with `Builtin.Completeness` to eliminate cross-model tokenization discrepancies during scoring.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**
Lexical overlap metrics (`ROUGE`, `BLEU`) and embedding similarity metrics (`BERTScore`) are notoriously insensitive to single-token negations (`"continue"` vs `"discontinue"`) because 95%+ of the surrounding tokens remain identical. Using **Amazon Bedrock Model Evaluation with an independent, high-reasoning LLM-as-a-Judge** (`Claude 3.5 Sonnet`) configured with `Builtin.Faithfulness` (entailment against source context), `Builtin.Correctness` (comparison against `referenceResponse`), and a **Custom LLM-as-a-Judge Metric** specifically instructing the judge to verify medication directives and dosage polarity deterministically catches clinical inversions.

**Distractor Analysis:**
- **Why A is incorrect:** `Builtin.BLEU` is a precision-oriented n-gram overlap metric that suffers from the exact same negation blindness as `ROUGE`, and `Builtin.ExactMatch` fails on any valid abstractive summary that paraphrases a sentence.
- **Why B is incorrect:** `Builtin.Robustness` measures model stability against input prompt perturbations (typos, case swaps), and `Builtin.Toxicity` detects hate speech or profanity—neither detects clinical negation hallucinations.
- **Why D is incorrect:** Using the candidate model (`amazon.nova-pro-v1:0`) to judge its own outputs introduces **self-preference (nepotism) bias** and risks repeating the exact same comprehension blind spot that caused the generation error.
</details>

---

### Question 2
A financial services enterprise has built a complex multi-agent orchestration workflow running on AWS Step Functions and Amazon ECS that combines proprietary quantitative risk models with Amazon Bedrock `Converse` calls. The engineering team wants to use Amazon Bedrock Evaluations (`CreateEvaluationJob`) with built-in LLM-as-a-Judge metrics (`Builtin.Correctness`, `Builtin.Completeness`, `Builtin.LogicalCoherence`) and a Custom Compliance Rubric to benchmark the end-to-end outputs of their custom multi-agent application without re-running inference inside Bedrock's managed evaluation environment.

How should the developer structure the evaluation dataset and `CreateEvaluationJob` request to meet this requirement?

- **A.** Package the custom ECS application container as a SageMaker Model Package, import it into Amazon Bedrock using Custom Model Import, and pass the imported model ARN in `inferenceConfig.models[].bedrockModel`.
- **B.** Format the S3 JSONL dataset using the Bring-Your-Own-Inference (BYOI) schema by populating the `modelResponses` array (`[{"response": "...", "modelIdentifier": "custom-risk-agent-v3"}]`) alongside `prompt` and `referenceResponse`, and specify `precomputedInferenceSource` (`inferenceSourceIdentifier: "custom-risk-agent-v3"`) in `inferenceConfig`.
- **C.** Configure `applicationType="RagEvaluation"` in `CreateEvaluationJob` and point `knowledgeBaseConfig` to an Amazon S3 bucket containing the JSON logs exported from the ECS tasks.
- **D.** Upload a CSV file containing `prompt`, `referenceResponse`, and `generatedOutput` columns to Amazon S3, and enable `Builtin.Accuracy` so Bedrock automatically detects the third column as pre-computed responses.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**
Amazon Bedrock Evaluations supports **Bring-Your-Own-Inference (BYOI)** (`precomputedInferenceSource`). By including the pre-generated outputs inside the `modelResponses` list of each JSONL line (`{"prompt": "...", "referenceResponse": "...", "modelResponses": [{"response": "...", "modelIdentifier": "custom-risk-agent-v3"}]}`) and referencing that `inferenceSourceIdentifier` in `inferenceConfig`, Bedrock skips running generator model inference and directly executes the Judge LLM across the imported responses.

**Distractor Analysis:**
- **Why A is incorrect:** Custom Model Import is only for importing supported open-weight model architectures (like Llama, Mistral, Qwen weights in SafeTensors format), not arbitrary ECS/Step Functions multi-agent container applications.
- **Why C is incorrect:** `knowledgeBaseConfig` requires a valid Amazon Bedrock Knowledge Base ID or precomputed RAG retrieval contexts schema, not raw ECS log buckets.
- **Why D is incorrect:** Amazon Bedrock Evaluations strictly requires UTF-8 `.jsonl` format (not CSV), and `Builtin.Accuracy` is a programmatic metric, not an ingestion mechanism for pre-computed responses.
</details>

---

### Question 3
An enterprise legal-tech team is configuring a Human-Based Model Evaluation job in Amazon Bedrock to have 15 internal corporate attorneys perform side-by-side preference ranking and Likert scoring comparing two fine-tuned models summarizing M&A regulatory filings. The team created an Amazon SageMaker Ground Truth private workteam backed by Amazon Cognito, uploaded the 250-prompt `.jsonl` dataset to a dedicated S3 bucket encrypted with an AWS KMS Customer Managed Key (CMK), and configured the Bedrock Evaluation IAM Service Role.

When the attorneys log into the SageMaker Ground Truth labeling portal to evaluate the model outputs, the UI fails to load the prompt contexts and displays a browser network error loading assets from Amazon S3. Furthermore, when the evaluation job attempts to write initial task manifests, it logs an `AccessDenied` error on the KMS key.

Which combination of actions will resolve BOTH issues? **(Choose TWO.)**

- **A.** Configure a Cross-Origin Resource Sharing (CORS) policy on the S3 bucket hosting the evaluation input and output data that allows `GET`, `HEAD`, `PUT`, `POST`, and `DELETE` methods from the Ground Truth worker portal origins.
- **B.** Enable Amazon S3 Transfer Acceleration and Public Read ACLs (`public-read`) on the S3 bucket so the attorneys' browsers can fetch the `.jsonl` prompts without Cognito session tokens.
- **C.** Convert the input dataset from `.jsonl` to Apache Parquet (`.parquet`) format so the SageMaker Ground Truth browser client can stream binary column chunks directly.
- **D.** Update the AWS KMS Customer Managed Key policy and the Bedrock Evaluation IAM Execution Role to grant `kms:Decrypt`, `kms:GenerateDataKey`, and `kms:DescribeKey` permissions to both the Bedrock Evaluation role and the SageMaker Ground Truth execution context.
- **E.** Replace the Amazon Cognito private workteam with the public Amazon Mechanical Turk workforce so AWS manages cross-account S3 bucket permissions automatically.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answers: A and D**

**Explanation:**
- **A is correct:** Human-based evaluation jobs in Amazon Bedrock use Amazon SageMaker Ground Truth worker portals, which load task content directly in the reviewer's browser via cross-origin requests to Amazon S3. AWS documentation explicitly mandates configuring an S3 CORS policy (`AllowedMethods: ["GET", "HEAD", "PUT", "POST", "DELETE"]`, `AllowedHeaders: ["*"]`) on the S3 bucket.
- **D is correct:** Because the S3 bucket uses a KMS Customer Managed Key (CMK), the IAM Service Role used by Bedrock/Ground Truth and the KMS Key Policy must allow `kms:Decrypt`, `kms:GenerateDataKey`, and `kms:DescribeKey` so task manifests can be decrypted and written.

**Distractor Analysis:**
- **Why B is incorrect:** Making proprietary M&A legal documents publicly readable via `public-read` ACLs is a severe security violation and is blocked by S3 Block Public Access; Ground Truth uses pre-signed URLs that require CORS, not public ACLs.
- **Why C is incorrect:** Bedrock Evaluations strictly requires `.jsonl` format; `.parquet` files are rejected with a `ValidationException`.
- **Why E is incorrect:** Using public Amazon Mechanical Turk workers for confidential M&A legal filings violates data confidentiality requirements and does not fix S3 CORS or KMS CMK permissions.
</details>

---

### Question 4
A retail company is evaluating its Amazon Bedrock Knowledge Base using a `RagEvaluation` job in `RETRIEVE_AND_GENERATE` mode across 300 customer support queries. The completed evaluation report shows the following mean metric scores (normalized 0.0 to 1.0):
- `Builtin.ContextRelevance`: **0.41**
- `Builtin.ContextCoverage`: **0.94**
- `Builtin.Faithfulness`: **0.96**
- `Builtin.Correctness`: **0.68**

What do these metrics indicate about the RAG pipeline's behavior, and what is the MOST effective architectural optimization?

- **A.** The retriever is failing to locate the documents containing the ground-truth facts because the fixed chunk size is too small; the team should increase the chunk size from 300 tokens to 1,500 tokens and re-ingest the S3 bucket.
- **B.** The generator foundation model is ignoring the retrieved chunks and hallucinating from its pre-trained weights; the team should lower `temperature` to `0.0` and attach a Bedrock Guardrail Contextual Grounding policy.
- **C.** The embedding model dimensions do not match the OpenSearch Serverless vector index; the team should switch from `amazon.titan-embed-text-v2:0` to `cohere.embed-english-v3`.
- **D.** The retriever is capturing almost all required ground-truth facts (`ContextCoverage = 0.94`), and the generator is not hallucinating (`Faithfulness = 0.96`), but the retrieved context window is heavily diluted with irrelevant chunks (`ContextRelevance = 0.41`), degrading final answer accuracy; the team should enable an Amazon Bedrock Reranker (`cohere.rerank-v3-5:0`) and metadata filtering to prune noisy chunks before generation.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: D**

**Explanation:**
Look closely at the metric signature:
1. **`ContextCoverage = 0.94` (High Recall)**: The retrieved set of chunks *does* contain 94% of the ground-truth facts needed to answer the questions.
2. **`ContextRelevance = 0.41` (Low Precision)**: Most of the retrieved chunks are irrelevant noise (for example, $k=25$ chunks were retrieved, and only 2 chunks are relevant while 23 are distraction).
3. **`Faithfulness = 0.96` (High Groundedness)**: The generator FM is faithfully sticking to the retrieved context without hallucinating outside facts.
4. **`Correctness = 0.68` (Degraded End-to-End Accuracy)**: Because the relevant facts are buried inside 23 noisy chunks ("Lost in the Middle" phenomenon), the generator gets confused by tangential policies in the noisy chunks.
Adding a cross-encoder **Reranker (`cohere.rerank-v3-5:0`)** and **Metadata Filtering** keeps `ContextCoverage` high while dramatically increasing `ContextRelevance` by passing only the top 3–5 relevant chunks to the generator.

**Distractor Analysis:**
- **Why A is incorrect:** If chunk size were causing missed facts, `ContextCoverage` would be low. Here `ContextCoverage` is `0.94`.
- **Why B is incorrect:** `Faithfulness` is already `0.96`, proving the generator is not hallucinating ungrounded claims.
- **Why C is incorrect:** A vector dimension mismatch prevents index creation or ingestion entirely with a `ValidationException`; it would not produce a `0.94` `ContextCoverage` score.
</details>

---

### Question 5
A DevOps team is building an automated CI/CD pipeline in AWS CodePipeline to validate updates to an Amazon Bedrock Knowledge Base's chunking strategy and embedding model before switching production traffic. Because the team iterates on 8 different chunk sizes and overlap combinations daily, running full `RETRIEVE_AND_GENERATE` evaluations with a large generator FM and Judge FM on every experiment takes 45 minutes and incurs high token costs. The team wants a fast, cost-optimized Stage 1 evaluation gate that isolates and benchmarks ONLY the vector retrieval quality before running full generation benchmarks on the top candidate.

How should the developer configure Stage 1 in Amazon Bedrock Evaluations?

- **A.** Configure `CreateEvaluationJob` with `applicationType="RagEvaluation"`, set the `ragConfigs` mode to `RETRIEVE` (Retrieval-Only) using `retrieveConfig`, and evaluate only `Builtin.ContextRelevance` and `Builtin.ContextCoverage`.
- **B.** Configure `CreateEvaluationJob` with `applicationType="ModelEvaluation"` and use programmatic metrics `Builtin.ROUGE` and `Builtin.BLEU` without specifying a Knowledge Base ID.
- **C.** Configure `CreateEvaluationJob` with `applicationType="RagEvaluation"` in `RETRIEVE_AND_GENERATE` mode, set `maxTokens=1` on the generator model, and evaluate `Builtin.Faithfulness`.
- **D.** Use Amazon Bedrock Guardrails `ApplyGuardrail` API in `INPUT` mode against the golden dataset prompts to score retrieval recall without invoking a Judge LLM.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: A**

**Explanation:**
Amazon Bedrock Knowledge Base Evaluations supports two distinct sub-tasks: **`RETRIEVE` (Retrieval-Only)** and **`RETRIEVE_AND_GENERATE`**. By configuring `retrieveConfig` (Retrieval-Only mode) and selecting retrieval-specific metrics (`Builtin.ContextRelevance` and `Builtin.ContextCoverage`), Bedrock queries the Knowledge Base `Retrieve` API directly and evaluates the retrieved passages against the prompt and `referenceResponses` without invoking a generator foundation model. This isolates retriever hyperparameter tuning (chunk size, overlap, hybrid search weight, reranker top-$n$) and cuts evaluation time and cost significantly.

**Distractor Analysis:**
- **Why B is incorrect:** `ModelEvaluation` evaluates raw foundation model generations, not Knowledge Base vector retrieval.
- **Why C is incorrect:** Setting `maxTokens=1` truncates the generated response to a single token, causing `Builtin.Faithfulness` and generation metrics to fail completely while still incurring generator input token charges.
- **Why D is incorrect:** `ApplyGuardrail` evaluates safety and contextual grounding of a generated output; it does not query a Knowledge Base or compute retrieval coverage against ground-truth answers.
</details>

---

### Question 6
A principal AI engineer is designing an offline benchmark to select a new primary foundation model for an insurance claims adjudication assistant. The three candidate generator models are `anthropic.claude-3-5-sonnet-20241022-v2:0`, `meta.llama3-1-70b-instruct-v1:0`, and `amazon.nova-pro-v1:0`. In an initial trial, the team configured `anthropic.claude-3-5-sonnet-20241022-v2:0` as the sole Judge LLM in Amazon Bedrock Evaluations. The results showed Claude 3.5 Sonnet scoring its own responses 14% higher on `Builtin.Helpfulness` and stylistic coherence than human claim adjusters scored the exact same responses.

Which evaluation methodology eliminates this evaluation bias while maintaining automated scalability for future regression runs?

- **A.** Replace LLM-as-a-Judge entirely with `Builtin.ExactMatch` and `Builtin.Accuracy` across the 2-page narrative claim justification letters.
- **B.** Increase the Judge model's `temperature` from `0.0` to `0.9` during the evaluation job so that its scoring distribution becomes randomized and unbiased across all three candidate models.
- **C.** Use a heterogeneous jury of Judge FMs from distinct model families (for example, pairing `mistral.mistral-large-2407-v1:0` and `anthropic.claude-3-5-sonnet-20241022-v2:0`), strip model-identifying metadata, calibrate the Custom Judge Rubric against a 100-sample human adjuster benchmark generated via SageMaker Ground Truth until inter-annotator agreement ($\kappa \ge 0.80$) is achieved, and anchor grading on objective factual criteria rather than open-ended style.
- **D.** Configure each candidate model to act as its own Judge LLM across three separate `CreateEvaluationJob` runs and compare the resulting average scores directly.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**
The scenario describes **self-preference (nepotism) bias** and **verbosity/style bias** in LLM-as-a-Judge pipelines. The industry and AWS best practice is to:
1. Calibrate the Judge prompt/rubric against a human SME reference set (collected via SageMaker Ground Truth) using statistical agreement metrics (Cohen's Kappa $\kappa$ or Krippendorff's Alpha $\alpha$).
2. Replace subjective stylistic criteria with atomic, objective rubric criteria (`customMetrics`).
3. Use an independent Judge model family (or an ensemble/jury of diverse model families) so no candidate model benefits from single-family stylistic bias.

**Distractor Analysis:**
- **Why A is incorrect:** `ExactMatch` and `Accuracy` are completely unusable for 2-page narrative insurance claim letters where valid phrasing varies widely.
- **Why B is incorrect:** Raising Judge `temperature` to `0.9` introduces non-deterministic noise and variance into the evaluation scores without fixing structural model-family bias.
- **Why D is incorrect:** Having each model grade itself compounds self-preference bias and makes scores completely incomparable across model families because each model applies different grading leniency.
</details>

---

### Question 7
An e-commerce company wants to validate a newly engineered system prompt in **Amazon Bedrock Prompt Management** (`version 4`) against their current production prompt (`version 3`) under live production traffic patterns. However, because the assistant handles high-value B2B procurement negotiations, product leadership mandates **zero customer-facing risk** during the initial 48-hour validation window—no live user may receive a response generated by `version 4` until its latency, tool-call syntax validity, and LLM-as-a-Judge faithfulness scores are proven on at least 5,000 real production queries.

Which deployment and validation architecture satisfies this requirement?

- **A.** Configure an Amazon Bedrock Agent Alias with a 95% / 5% canary traffic split so only 5% of live B2B users receive responses from `version 4`.
- **B.** Implement a **Shadow Deployment (Dark Launch)** pattern where the synchronous API handler serves 100% of users using `version 3`, while asynchronously publishing a copy of the incoming request payload and retrieved context to Amazon EventBridge / SQS, where a shadow worker invokes `version 4` and logs both responses to S3 for automated LLM-as-a-Judge comparative scoring.
- **C.** Configure an Amazon Route 53 latency-based routing policy that directs users with the lowest network latency to an API Gateway stage running `version 4`.
- **D.** Run an offline `CreateEvaluationJob` using 50 synthetic prompts before deployment and immediately cut 100% of production traffic over to `version 4`.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**
When a requirement explicitly specifies **zero customer-facing impact** while testing against **real live production queries**, neither canary routing nor A/B testing is acceptable because both expose a percentage of live users to the unproven candidate variant. A **Shadow Deployment (Dark Launch)** pattern asynchronously forks live production events (via EventBridge, SNS/SQS, or Kinesis Firehose) to invoke `version 4` out-of-band. Users only see `version 3` responses, while the shadow pipeline records `version 4` latency, tool errors, and side-by-side Judge LLM evaluations in the background.

**Distractor Analysis:**
- **Why A is incorrect:** A 5% canary still exposes 1 in 20 live B2B procurement sessions to `version 4`, violating the zero customer-facing risk mandate.
- **Why C is incorrect:** Route 53 latency routing exposes live users in specific geographic regions to `version 4`.
- **Why D is incorrect:** 50 synthetic prompts do not satisfy the requirement to validate against 5,000 real production queries prior to customer exposure.
</details>

---

### Question 8
A cybersecurity company is building a GenAI log-analysis assistant that accepts noisy, OCR-scanned terminal screenshots and hastily typed incident responder queries (containing typos, inconsistent casing, and character transpositions) and outputs structured MITRE ATT&CK technique IDs. During testing on clean, grammatically perfect prompts, the model achieves 94% accuracy. However, in real incident war-rooms, accuracy drops to 71%.

Which built-in Amazon Bedrock Evaluation metric is specifically designed to quantify how much a candidate model's accuracy degrades when inputs are subjected to orthographic perturbations, typos, and synonym substitutions?

- **A.** `Builtin.LogicalCoherence`
- **B.** `Builtin.Faithfulness`
- **C.** `Builtin.ContextCoverage`
- **D.** `Builtin.Robustness`

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: D**

**Explanation:**
`Builtin.Robustness` in Amazon Bedrock Evaluations automatically applies semantic-preserving perturbations to the input prompts in your evaluation dataset (such as keyboard typos, character swaps, random casing changes, whitespace noise, and synonym replacements) and calculates both the perturbed score and the **delta score** (how much accuracy/ROUGE dropped compared to the unperturbed baseline). This directly measures model resilience to noisy user inputs.

**Distractor Analysis:**
- **Why A is incorrect:** `Builtin.LogicalCoherence` evaluates whether the model's generated text is internally consistent and free of self-contradiction.
- **Why B is incorrect:** `Builtin.Faithfulness` evaluates whether the generated answer is grounded in provided context without hallucination.
- **Why C is incorrect:** `Builtin.ContextCoverage` is a RAG retrieval metric measuring whether retrieved chunks cover ground-truth facts.
</details>

---

### Question 9
An AI platform team is building an automated CI/CD regression gate in GitHub Actions and AWS CodeBuild for a customer-facing RAG application. The golden evaluation dataset contains 400 queries: 360 routine FAQ queries and 40 high-risk regulatory compliance queries. In the latest pull request, a prompt modification broke the handling of the 40 regulatory queries (causing their `Builtin.Faithfulness` score to drop from `0.95` to `0.25`), while the 360 routine FAQ queries scored `0.98`. Because the overall mean `Builtin.Faithfulness` across all 400 queries was $(360 \times 0.98 + 40 \times 0.25) / 400 = 0.907$, the CI/CD gate (which checked `Mean_Faithfulness >= 0.90`) passed the build and promoted the broken prompt to production.

Which two statistical and dataset engineering changes should the team implement in their evaluation gate to prevent this failure mode? **(Choose TWO.)**

- **A.** Remove the 360 routine FAQ queries permanently from all evaluation suites so only 40 prompts are ever tested.
- **B.** Tag each record in the JSONL dataset with a `"category"` field (e.g., `"category": "Regulatory_Compliance"`) and enforce per-category minimum score thresholds in addition to tail-percentile gates (such as 10th percentile $p_{10} \ge 0.85$).
- **C.** Switch the evaluation metric from `Builtin.Faithfulness` to `Builtin.BLEU` because `BLEU` applies a geometric brevity penalty that prevents high averages.
- **D.** Increase the generator model's `top_p` parameter to `1.0` during CI/CD evaluation runs to smooth out bimodal score distributions.
- **E.** Enforce a hard-fail assertion (`min_score >= threshold` or zero tolerance for `Faithfulness < 0.70`) specifically on the stratified `"Regulatory_Compliance"` golden subset so any critical safety regression blocks promotion regardless of the global mean.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answers: B and E**

**Explanation:**
- **B is correct:** Relying solely on a global arithmetic mean across an imbalanced dataset masks severe regressions in critical minority cohorts (Simpson's paradox / tail masking). Using the `"category"` attribute in the Bedrock Evaluation JSONL schema and computing both **per-category aggregations** and **tail percentiles ($p_{10}$ or $p_5$)** immediately exposes the `0.25` cohort failure.
- **E is correct:** For high-risk regulatory or safety test cases, LLMOps pipelines should enforce strict cohort-specific minimum thresholds (blocking promotion if any critical compliance slice falls below an acceptable floor).

**Distractor Analysis:**
- **Why A is incorrect:** Deleting routine regression tests leaves 90% of user traffic unvalidated against regressions.
- **Why C is incorrect:** `BLEU` measures n-gram precision for translation, not hallucination/groundedness, and a brevity penalty only penalizes short responses—not low-scoring subsets.
- **Why D is incorrect:** Adjusting `top_p` changes token sampling randomness; it does not fix the statistical aggregation flaw in the CI/CD gate.
</details>

---

### Question 10
A team is creating a custom LLM-as-a-Judge metric (`AutomatedEvaluationCustomMetricSource`) inside an Amazon Bedrock `CreateEvaluationJob` call to evaluate whether a SQL generation assistant follows enterprise data warehouse conventions. The team wants the Judge LLM to have access to the original natural-language question, the reference golden SQL query, and the candidate model's generated SQL query inside the custom grading prompt.

Which template placeholder variables must the developer include in the `instructions` string of the `customMetricDefinition`?

- **A.** `$INPUT_TEXT`, `$EXPECTED_OUTPUT`, and `$ACTUAL_OUTPUT`
- **B.** `{question}`, `{referenceResponse}`, and `{modelResponse}`
- **C.** `{{prompt}}`, `{{ground_truth}}`, and `{{prediction}}`
- **D.** `<% user_query %>`, `<% golden_sql %>`, and `<% generated_sql %>`

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**
In Amazon Bedrock Evaluations custom metrics (`customMetricDefinition.instructions`), the service uses Mustache-style double curly brace template variables:
- `{{prompt}}` for the input prompt from the dataset.
- `{{prediction}}` for the candidate model's generated response.
- `{{ground_truth}}` for the `referenceResponse` from the dataset (when comparing against a reference answer).

**Distractor Analysis:**
- **Why A, B, and D are incorrect:** None of these variable syntaxes are recognized by the Amazon Bedrock Evaluations custom metric template parser, which will either fail validation or leave the literal strings uninterpolated in the Judge prompt.
</details>

---

### Question 11
A SaaS company has deployed a multi-turn customer retention chatbot using the Amazon Bedrock `Converse` API. Product managers want to run an online A/B test comparing two different system prompt personas (`Persona_Empathetic` vs `Persona_Direct_Discount`) across 20% of live traffic to measure which persona achieves a higher 7-day subscription retention rate and higher explicit user thumbs-up ratings.

During the first 4 hours of the A/B test, the data science team notices that the experiment results are invalid because users are receiving `Persona_Empathetic` on Turn 1 of a conversation, `Persona_Direct_Discount` on Turn 2, and `Persona_Empathetic` on Turn 3, causing jarring tone shifts within the same conversation.

What caused this issue, and how should the developer fix the A/B traffic routing logic?

- **A.** The routing layer is generating a random float (`random.random() < 0.5`) on every individual `Converse` HTTP request; the developer should replace per-request random assignment with a deterministic cryptographic hash (e.g., `SHA-256(user_id + experiment_salt) % 100`) so every user and multi-turn session is consistently pinned to the same cohort across all turns.
- **B.** Amazon Bedrock `Converse` automatically randomizes system prompts when `temperature > 0.0`; the developer must set `temperature = 0.0` in `inferenceConfig`.
- **C.** The developer did not enable Amazon Bedrock Prompt Caching; enabling `cachePoint` on the system prompt forces sticky routing across all API Gateway instances.
- **D.** The `messages` array in `Converse` does not support A/B testing; the developer must provision two separate AWS accounts and use AWS Organizations Service Control Policies (SCPs) to route users.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: A**

**Explanation:**
In multi-turn conversational AI applications, stateless per-request random routing (`random()`) causes a user to flip between Variant A and Variant B on each turn of the same conversation, contaminating both the user experience and the downstream attribution of business metrics (7-day retention). By hashing the `user_id` (or `session_id`) with an experiment salt (`hash(user_id + salt) % 100`), cohort assignment becomes **stateless yet deterministic**, ensuring a user experiences a consistent persona across every turn and session during the experiment window.

**Distractor Analysis:**
- **Why B is incorrect:** `temperature` controls token probability distribution during decoding; it has nothing to do with which system prompt string the application passes to `Converse`.
- **Why C is incorrect:** Prompt Caching reduces input token latency and cost by caching KV prefix states on Bedrock servers; it does not control application-layer A/B variant selection.
- **Why D is incorrect:** AWS Organizations SCPs are IAM permission guardrails, not application traffic routers.
</details>

---

### Question 12
An enterprise team is running an Amazon Bedrock Knowledge Base evaluation job (`RagEvaluation`) to benchmark a new HR policy assistant. When calling `CreateEvaluationJob`, the API returns a `ValidationException` stating that the dataset metric configuration is invalid for the selected evaluation task. Upon inspecting the Python script, the developer finds the following configuration block:

```python
"datasetMetricConfigs": [
    {
        "taskType": "Summarization",
        "dataset": {
            "name": "HRPolicyQADataset",
            "datasetLocation": {"s3Uri": "s3://hr-eval-bucket/qa_dataset.jsonl"}
        },
        "metricNames": [
            "Builtin.ContextRelevance",
            "Builtin.ContextCoverage",
            "Builtin.Faithfulness"
        ]
    }
]
```

Why did `CreateEvaluationJob` reject this configuration, and how should it be corrected?

- **A.** `Builtin.ContextRelevance` and `Builtin.ContextCoverage` can only be evaluated when `applicationType` is set to `"ModelEvaluation"` with `taskType="Classification"`.
- **B.** Amazon Bedrock Evaluations only allows evaluating one single metric per `CreateEvaluationJob` call; the developer must launch three separate evaluation jobs.
- **C.** The `datasetLocation.s3Uri` must point to a folder prefix ending with a trailing slash (`/`) rather than a specific `.jsonl` object key.
- **D.** For `applicationType="RagEvaluation"`, the `taskType` parameter in `datasetMetricConfigs` must be set to `"QuestionAndAnswer"` (not `"Summarization"`), and the S3 JSONL dataset must follow the `conversationTurns` schema.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: D**

**Explanation:**
In Amazon Bedrock Evaluations, when `applicationType` is set to `"RagEvaluation"`, the only supported `taskType` in `datasetMetricConfigs` is `"QuestionAndAnswer"`, and the dataset must use the RAG `conversationTurns` JSONL format. Setting `taskType="Summarization"` is valid for `ModelEvaluation`, not `RagEvaluation`, triggering a `ValidationException`.

**Distractor Analysis:**
- **Why A is incorrect:** `Builtin.ContextRelevance` and `Builtin.ContextCoverage` are exclusively RAG evaluation metrics (`RagEvaluation`), not classification model evaluation metrics.
- **Why B is incorrect:** A single `CreateEvaluationJob` call can evaluate multiple metrics simultaneously in the `metricNames` list.
- **Why C is incorrect:** `datasetLocation.s3Uri` must point to the exact `.jsonl` object URI in S3 (`s3://bucket/path/file.jsonl`), whereas `outputDataConfig.s3Uri` points to an S3 folder prefix (`s3://bucket/output/`).
</details>

---

### Question 13
A media company is evaluating a fine-tuned headline generation model against a base foundation model using Amazon Bedrock Automatic Model Evaluation. The dataset contains 800 news articles paired with editor-written reference headlines. However, because many valid headlines use synonyms (for example, `"Federal Reserve Holds Interest Rates Steady"` vs `"Central Bank Leaves Borrowing Costs Unchanged"`), the team observes that `Builtin.ROUGE` and `Builtin.BLEU` give near-zero scores to creative, semantically identical headlines. The team wants a **fast, programmatic (non-LLM-as-a-Judge)** metric that captures semantic similarity via contextual token embeddings rather than exact lexical overlap.

Which metric should the developer select?

- **A.** `Builtin.ExactMatch`
- **B.** `Builtin.BERTScore`
- **C.** `Builtin.Accuracy`
- **D.** `Builtin.Toxicity`

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**
`Builtin.BERTScore` computes token-level cosine similarity using contextualized embeddings from a pre-trained bidirectional transformer encoder (BERT family) rather than matching exact surface strings or n-grams. Consequently, `"Federal Reserve Holds Interest Rates Steady"` and `"Central Bank Leaves Borrowing Costs Unchanged"` receive a high `BERTScore` despite having zero lexical word overlap, without requiring the latency and cost of a full generative LLM-as-a-Judge call.

**Distractor Analysis:**
- **Why A is incorrect:** `Builtin.ExactMatch` requires 100% string identity, scoring `0.0` whenever a single word differs.
- **Why C is incorrect:** `Builtin.Accuracy` is designed for discrete classification / exact label matching.
- **Why D is incorrect:** `Builtin.Toxicity` scores harmful or profane language, not semantic equivalence to a reference headline.
</details>

---

### Question 14
An enterprise security team audits a GenAI application's CI/CD evaluation pipeline and discovers that while the RAG assistant scores `0.95` on `Builtin.Correctness` for answerable questions, it frequently fabricates plausible-sounding policy numbers when users ask questions about topics that do **not** exist anywhere in the company's Knowledge Base (for example, asking about a non-existent `"Pet Sabbatical Leave Policy"`).

How should the developer update the golden evaluation dataset and evaluation rubric to systematically test and enforce proper abstention on unanswerable queries?

- **A.** Add a stratified subset of unanswerable / out-of-scope prompts to the golden JSONL dataset where `referenceResponse` explicitly states that the information is not present in the corporate policy corpus, and evaluate `Builtin.Faithfulness`, `Builtin.Correctness`, and a custom abstention metric verifying the model refuses to speculate.
- **B.** Delete all retrieval instructions from the Knowledge Base prompt template and increase `numberOfResults` from 5 to 50 so the model always finds at least one loosely related document.
- **C.** Enable `Builtin.BLEU` on the existing answerable dataset and set `temperature = 1.0` so the model expresses lower confidence.
- **D.** Switch the Knowledge Base search type from `HYBRID` to `SEMANTIC` so out-of-scope queries return zero vector results from OpenSearch Serverless.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: A**

**Explanation:**
If a golden evaluation dataset only contains answerable questions, a model that aggressively guesses will never be penalized for failing to abstain when context is missing. By curating an **Unanswerable / Negative Test Cohort** (10%–15% of the golden dataset) where the ground-truth `referenceResponse` is a polite abstention (`"Our corporate policies do not contain information regarding a Pet Sabbatical Leave Policy"`), `Builtin.Faithfulness` (which flags claims unsupported by retrieved chunks) and `Builtin.Correctness` (which compares the generated output to the abstention reference) will immediately penalize hallucinated policies.

**Distractor Analysis:**
- **Why B is incorrect:** Increasing `numberOfResults` to 50 injects 50 irrelevant chunks into the prompt, increasing token cost and confusing the model further.
- **Why C is incorrect:** Setting `temperature = 1.0` increases hallucination probability rather than preventing it.
- **Why D is incorrect:** Dense vector `SEMANTIC` search *always* returns the top-$k$ nearest neighbors by cosine similarity, even if the query is completely unrelated to the corpus.
</details>

---

### Question 15
A banking organization uses an Amazon Bedrock Agent connected to two Action Groups (`AccountLookup` and `DisputeFiling`) and one Knowledge Base (`CardholderAgreementsKB`). Before promoting a new Agent version to production, the team wants their automated Python test harness in AWS CodeBuild to verify not only the final text response, but also **trajectory correctness**—specifically proving that for a dispute request, the Agent's orchestration loop called `AccountLookup` *before* calling `DisputeFiling`, passed the correct `transaction_id` parameter, and retrieved chunks from `CardholderAgreementsKB`.

How can the automated test harness deterministically assert this execution trajectory?

- **A.** Call `bedrock.create_evaluation_job` with `applicationType="ModelEvaluation"` and enable `Builtin.LogicalCoherence`, which automatically inspects internal Agent Lambda invocations.
- **B.** Query AWS Cost Explorer via `ce.get_cost_and_usage` immediately after the test run to verify whether two Lambda invocations and one OpenSearch Serverless query were billed.
- **C.** Invoke the Agent via `bedrock-agent-runtime.invoke_agent` with `enableTrace=True`, iterate through the returned `completion` event stream to collect all `orchestrationTrace.invocationInput` events in chronological order, and assert that the ordered sequence of `actionGroupInvocationInput` and `knowledgeBaseLookupInput` steps matches the expected golden trajectory.
- **D.** Parse the final natural-language text returned in `chunk["bytes"]` using a regular expression to guess which internal APIs the Agent called.

<details>
<summary><strong>Answer & Detailed Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**
In agentic systems, evaluating only the final text response is insufficient because an agent might arrive at a plausible-looking text response while skipping mandatory prerequisite verification tools (`AccountLookup`) or invoking `DisputeFiling` with a hallucinated parameter. By invoking the agent with `enableTrace=True` and recording the ordered sequence of `orchestrationTrace.invocationInput` events (`actionGroupInvocationInput.actionGroupName`, `function`/`apiPath`, `parameters`, and `knowledgeBaseLookupInput`), the CI/CD test harness can perform deterministic **Trajectory / Tool-Call Assertions** alongside output quality checks.

**Distractor Analysis:**
- **Why A is incorrect:** `ModelEvaluation` evaluates single-turn prompt-response pairs on foundation models; it does not execute or inspect Bedrock Agent multi-step tool trajectories.
- **Why B is incorrect:** AWS Cost Explorer billing data takes 24+ hours to populate and cannot validate tool execution order or parameter values.
- **Why D is incorrect:** Final user-facing answers do not reliably expose internal API names or parameter payloads, making regex guessing brittle and inaccurate.
</details>
