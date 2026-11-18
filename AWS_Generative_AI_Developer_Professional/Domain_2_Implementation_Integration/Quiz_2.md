# Domain 2 — Quiz 2: Model Deployment Strategies on Amazon Bedrock & Amazon SageMaker AI (Task 2.2)

> **Track**: AWS Certified Generative AI Developer - Professional (AIP-C01)  
> **Scope**: Task 2.2 — Bedrock On-Demand, Cross-Region Inference, Batch Inference, Provisioned Throughput, Custom Model Import (CMI), SageMaker Real-Time (LMI / vLLM / TensorRT-LLM), Asynchronous, Serverless, Multi-Model Endpoints (MME), Inference Components & Multi-Adapter LoRA, and Blue/Green Rollouts.  
> **Total Questions**: 15 (12 Single-Response `A`–`D`, 3 Multiple-Response `Choose TWO` `A`–`E`)

---

### Question 1

A biotechnology research firm has fine-tuned a 7-billion-parameter Mistral-7B model inside an Amazon SageMaker training job using proprietary clinical trial synthesis records. The resulting HuggingFace `.safetensors` weights and tokenizer artifacts are stored in Amazon S3. The clinical research team uses this model interactively through an internal web application for approximately 90 minutes each morning and 60 minutes each afternoon; the system receives zero requests during the remaining 21.5 hours of the day and on weekends.

The engineering team wants to invoke this custom model using the unified Amazon Bedrock `Converse` API and apply Bedrock Guardrails without paying for 24x7 idle GPU instances or purchasing 1-month/6-month hourly commitments.

Which deployment strategy meets these requirements at the lowest operational cost?

- **A.** Use **Amazon Bedrock Custom Model Import (CMI)** (`CreateModelImportJob`) to import the `.safetensors` artifacts from Amazon S3 into Amazon Bedrock, and invoke the imported model ARN on-demand (paying only for active Custom Model Units in 5-minute billing windows with automatic scale-to-zero when idle).
- **B.** Recreate the fine-tuning job natively inside Amazon Bedrock using a Model Customization job, and purchase a 6-month **Provisioned Throughput** commitment of 2 Model Units (MUs).
- **C.** Deploy the model onto an Amazon SageMaker **Serverless Inference** endpoint configured with 6 GB of memory and 1 NVIDIA A10G GPU.
- **D.** Deploy the model onto a standard Amazon SageMaker **Multi-Model Endpoint (MME)** backed by two `ml.c6i.xlarge` CPU instances running 24x7.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: A**

#### Architectural Explanation
**Amazon Bedrock Custom Model Import (CMI)** allows organizations to import custom weights for supported open-weight architectures (such as Mistral, Mixtral, Meta Llama, and Qwen) packaged in HuggingFace `.safetensors` format from Amazon S3 directly into Amazon Bedrock. Unlike native Bedrock fine-tuning (which mandates purchasing hourly **Provisioned Throughput**), Custom Model Import bills per **Custom Model Unit (CMU)** in 5-minute active windows and **automatically scales to zero** after ~5 minutes of inactivity. It also integrates natively with `Converse`, Bedrock Guardrails, Knowledge Bases, and Agents.

#### Distractor Analysis
- **B is incorrect**: Natively fine-tuned Bedrock models require purchasing **Provisioned Throughput** billed hourly 24x7, which wastes money during the 21.5 hours per day the application is idle.
- **C is incorrect**: Amazon SageMaker **Serverless Inference** is CPU-only (1–6 GB RAM) and **does not support GPU accelerators** (`ml.g5` / NVIDIA A10G).
- **D is incorrect**: Running a 7B parameter LLM on `ml.c6i.xlarge` CPU instances results in unusable token generation latency, and standard MME endpoints do not provide native Bedrock `Converse` API or Guardrails integration.

</details>

---

### Question 2

A multi-tenant legal-tech SaaS company serves 85 enterprise law firms. Every law firm has its own custom Low-Rank Adaptation (**LoRA**) adapter (each 120 MB in size) fine-tuned on top of the same base **Meta Llama 3.1 8B Instruct** model. During business hours, dozens of law firms send concurrent interactive drafting requests requiring sub-second Time-To-First-Token (TTFT).

Previously, the company tried deploying 85 separate `ml.g5.2xlarge` SageMaker Real-Time Endpoints (costing over \$74,000/month), and then tried a single standard SageMaker Multi-Model Endpoint (MME), which suffered 45-second cold-start latency spikes whenever full 16 GB models were evicted and reloaded from Amazon S3.

Which architecture will serve all 85 tenant models with sub-second adapter switching while minimizing GPU cluster cost?

- **A.** Import all 85 models as separate full-weight models using Amazon Bedrock Provisioned Throughput with 85 dedicated Model Units.
- **B.** Deploy an Amazon SageMaker Real-Time Endpoint with **Managed Instance Scaling** (`ml.g5.12xlarge`), create a **Base Inference Component** running the SageMaker **Large Model Inference (LMI) vLLM** container (`OPTION_ENABLE_LORA=true`) holding the base Llama 3.1 8B model in GPU VRAM, and register the 85 tenant LoRA adapters as lightweight **Inference Components** referencing `BaseInferenceComponentName`.
- **C.** Deploy 85 AWS Lambda functions with 10 GB of RAM and mount the base Llama 3.1 8B weights from Amazon EFS on every invocation.
- **D.** Convert all interactive drafting requests into Amazon SageMaker Batch Transform jobs that spin up an `ml.p4d.24xlarge` cluster once every hour.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: B**

#### Architectural Explanation
**SageMaker Inference Components** combined with the **Large Model Inference (LMI) vLLM** container (`OPTION_ENABLE_LORA=true`) support **Multi-Adapter LoRA serving**. The 16 GB base Llama 3.1 8B model is loaded into GPU VRAM **once** via a Base Inference Component, while each tenant's 120 MB LoRA adapter is registered as a child Inference Component (`BaseInferenceComponentName`). Incoming requests specify `InferenceComponentName="ic-tenant-firm-42"` in `invoke_endpoint`, enabling vLLM to dynamically apply the tenant's LoRA rank matrices in milliseconds on a small shared GPU fleet.

#### Distractor Analysis
- **A is incorrect**: Purchasing 85 dedicated Provisioned Throughput Model Units for 85 separate copies of an 8B model is prohibitively expensive compared to sharing base weights with LoRA.
- **C is incorrect**: AWS Lambda does not offer GPU acceleration, and running an 8B LLM on Lambda CPU over EFS violates sub-second TTFT requirements.
- **D is incorrect**: Batch Transform is an offline batch processing mechanism taking minutes to launch clusters, unsuitable for interactive sub-second drafting.

</details>

---

### Question 3

A media analytics company processes 350 MB multi-page high-resolution PDF broadcast transcripts and embedded audio spectrograms using a custom multimodal foundation model hosted on Amazon SageMaker AI. Each inference request takes between **4 and 11 minutes** to complete. Traffic arrives in unpredictable waves: 400 documents may arrive within 10 minutes, followed by 6 hours of zero traffic overnight. When a document finishes processing, a downstream microservice must be notified immediately to update the customer dashboard.

Which SageMaker deployment configuration meets all payload size, timeout, notification, and cost-optimization requirements?

- **A.** Deploy the model on a **SageMaker Real-Time Inference** endpoint behind Amazon API Gateway REST API and poll the HTTP connection for 11 minutes.
- **B.** Deploy the model on a **SageMaker Serverless Inference** endpoint with `MaxConcurrency=50` and a 6 MB payload limit.
- **C.** Deploy the model on a **SageMaker Asynchronous Inference** endpoint configured with an Amazon S3 input/output location, an **Amazon SNS** `SuccessTopic` and `ErrorTopic` notification configuration, and an Application Auto Scaling policy with **`MinCapacity=0`** so the GPU instance fleet scales to zero overnight.
- **D.** Package the custom multimodal C++ container into an Amazon Bedrock Knowledge Base chunking Lambda function.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: C**

#### Architectural Explanation
**Amazon SageMaker Asynchronous Inference** is purpose-built for large payloads (**up to 1 GB**, stored in Amazon S3) and long processing times (**up to 1 hour / 3,600 seconds**). It maintains an internal managed queue, supports GPU instances, natively publishes completion/failure events to **Amazon SNS** (`NotificationConfig`), and supports **Application Auto Scaling down to `MinCapacity=0` instances** when the internal request queue is empty—eliminating overnight idle GPU spend.

#### Distractor Analysis
- **A is incorrect**: SageMaker Real-Time Inference has a strict **6 MB** payload limit (far below 350 MB) and a **60-second** synchronous invocation timeout (600s max for streaming).
- **B is incorrect**: SageMaker Serverless Inference has a **4 MB** payload limit, a **60-second** processing timeout, and **no GPU support**.
- **D is incorrect**: AWS Lambda has a 6 MB synchronous payload limit (256 KB async event limit) and no GPU support.

</details>

---

### Question 4

An enterprise e-commerce retailer operates a customer-facing shopping assistant in `us-east-1` powered by `anthropic.claude-3-5-sonnet-20241022-v2:0` on Amazon Bedrock. During flash sales, traffic spikes 4x above normal baseline for 20-minute intervals, causing `ThrottlingException (HTTP 429)` errors when the single-region On-Demand quota is reached.

The Chief Information Security Officer (CISO) approves processing inference requests across any AWS Region within the United States (`us-east-1`, `us-east-2`, `us-west-2`), provided that:
1. There is no fixed hourly commitment or idle capacity cost between flash sales.
2. The FinOps team can attach AWS Cost Allocation Tags (`CostCenter=RetailChat`, `Environment=Prod`) to track exact token consumption for this specific workload in AWS Cost Explorer.

Which two steps must the cloud architect take to implement this solution? **(Choose TWO)**

- **A.** Use the `us.anthropic.claude-3-5-sonnet-20241022-v2:0` **System-Defined Cross-Region Inference Profile** as the underlying model source so Bedrock dynamically routes burst traffic across US regions at standard On-Demand token rates.
- **B.** Create an **Application Inference Profile** (`create_inference_profile`) copying from the US Cross-Region Inference Profile ARN, attach the `CostCenter=RetailChat` and `Environment=Prod` tags to the Application Inference Profile, and pass the Application Inference Profile ARN as the `modelId` in `Converse` calls.
- **C.** Purchase 10 Model Units of **Provisioned Throughput** in `us-east-1`, `us-east-2`, and `us-west-2` with a 6-month commitment.
- **D.** Directly attach AWS Cost Allocation Tags to the AWS-managed system inference profile `us.anthropic.claude-3-5-sonnet-20241022-v2:0` using the `tag_resource` API.
- **E.** Switch the workload to Amazon Bedrock Batch Inference (`CreateModelInvocationJob`) so flash-sale shoppers receive responses within 24 hours.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answers: A and B**

#### Architectural Explanation
1. **Cross-Region Burst Capacity without Hourly Commitments (Option A)**: **Cross-Region Inference Profiles (CRIS)** (`us.*`) pool capacity across multiple AWS Regions within a geography (`us-east-1`, `us-east-2`, `us-west-2`) at no extra per-token surcharge and with zero hourly commitment.
2. **FinOps Tagging via Application Inference Profiles (Option B)**: You **cannot** directly tag AWS-owned System-Defined Inference Profiles or base foundation model ARNs. Instead, you create a customer-owned **Application Inference Profile** (`create_inference_profile` with `modelSource={"copyFrom": "arn:aws:bedrock:us-east-1::inference-profile/us.anthropic.claude-3-5-sonnet-20241022-v2:0"}`), attach your **Cost Allocation Tags** to that Application Inference Profile, and invoke its ARN.

#### Distractor Analysis
- **C is incorrect**: Provisioned Throughput incurs fixed 24x7 hourly charges across 3 regions, violating Requirement 1.
- **D is incorrect**: System-defined inference profiles are read-only AWS resources; attempting to call `tag_resource` on a system profile fails with a `ValidationException` / `AccessDeniedException`.
- **E is incorrect**: Batch Inference is asynchronous and unsuitable for real-time interactive flash-sale chat.

</details>

---

### Question 5

An AI platform team is deploying a 70-billion-parameter Llama 3.1 model in FP16 precision (~140 GB of VRAM required for model weights plus KV cache) onto an Amazon SageMaker Real-Time Endpoint using an `ml.p4d.24xlarge` instance equipped with **8x NVIDIA A100 (40 GB VRAM each, 320 GB total GPU VRAM)** GPUs and the SageMaker **Large Model Inference (LMI)** container.

When the team deploys the container with default settings, the container fails to start with an `OutOfMemoryError (CUDA OOM)` on `GPU 0` because the runtime attempts to load the entire 140 GB model onto a single 40 GB A100 GPU. Additionally, under concurrent load testing, long 4,000-token generations block short 50-token requests sitting in the same static batch.

How should the engineer configure the LMI container environment variables (`serving.properties`) to resolve the CUDA OOM error and eliminate Head-of-Line (HoL) batch blocking?

- **A.** Set `OPTION_TENSOR_PARALLEL_DEGREE=1`, `OPTION_ROLLING_BATCH=disable`, and `SAGEMAKER_MODEL_SERVER_WORKERS=8` so that 8 separate copies of the 140 GB model are loaded across the 8 GPUs.
- **B.** Switch the instance type from `ml.p4d.24xlarge` to a CPU-only `ml.r6i.32xlarge` instance and disable GPU drivers.
- **C.** Keep `OPTION_TENSOR_PARALLEL_DEGREE=1` and enable Amazon S3 Transfer Acceleration on the model artifact bucket.
- **D.** Set `OPTION_TENSOR_PARALLEL_DEGREE=8` (or `4` with quantization) to shard model layers across all GPUs on the instance, and set `OPTION_ROLLING_BATCH=vllm` (or `trtllm`) to enable continuous batching and PagedAttention.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: D**

#### Architectural Explanation
1. **Resolving Single-GPU CUDA OOM (`OPTION_TENSOR_PARALLEL_DEGREE=8`)**: Because a 70B FP16 model (~140 GB) exceeds the 40 GB VRAM of a single A100 GPU, you must use **Tensor Parallelism** (`OPTION_TENSOR_PARALLEL_DEGREE=8`) to shard the weight matrices horizontally across all 8 GPUs on the `ml.p4d.24xlarge` (~17.5 GB of weights per GPU, leaving ample VRAM for KV cache).
2. **Eliminating Head-of-Line Blocking (`OPTION_ROLLING_BATCH=vllm` or `trtllm`)**: Enabling **Continuous Batching (Rolling Batch)** with **PagedAttention** allows the inference server to evict finished short sequences at the token iteration level and immediately insert waiting requests without waiting for the longest sequence in the batch to finish.

#### Distractor Analysis
- **A is incorrect**: Setting `SAGEMAKER_MODEL_SERVER_WORKERS=8` with `OPTION_TENSOR_PARALLEL_DEGREE=1` tries to load 8 independent copies of the 140 GB model onto 8 individual 40 GB GPUs, causing immediate CUDA OOM on all 8 GPUs.
- **B is incorrect**: Running a 70B model on CPU instances is orders of magnitude too slow for real-time interactive inference.
- **C is incorrect**: S3 Transfer Acceleration speeds up network download from S3, not GPU VRAM allocation.

</details>

---

### Question 6

A financial services institution processes 2.5 million archived customer call transcripts every Sunday night to classify sentiment, extract product mentions, and generate compliance summaries. The transcripts are stored in Amazon S3, and the completed results must be written back to Amazon S3 by Monday at 6:00 AM.

Currently, a fleet of EC2 worker nodes reads the transcripts and calls Amazon Bedrock `Converse` On-Demand API (`anthropic.claude-3-5-haiku-20241022-v1:0`). However, the Sunday night batch frequently triggers `ThrottlingException` errors that interfere with the bank's live 24x7 fraud-detection chatbot running in the same AWS account and region, and management wants to reduce the token bill for the Sunday batch by 50%.

Which architecture resolves the quota contention with live production traffic AND reduces token inference costs by 50%?

- **A.** Format the 2.5 million prompts as JSONL files in Amazon S3 and submit an **Amazon Bedrock Batch Inference** job (`CreateModelInvocationJob`), which processes records asynchronously using dedicated batch capacity outside real-time On-Demand quotas at a **50% token price discount**.
- **B.** Increase the EC2 worker thread pool from 50 threads to 5,000 threads and disable `botocore` retries so requests fail faster.
- **C.** Purchase 50 Model Units of **Provisioned Throughput** with a 6-month commitment exclusively for the 6-hour Sunday night window.
- **D.** Route the 2.5 million Sunday batch requests through an Amazon Bedrock Agent with Pre-processing, Post-processing, and `AMAZON.CodeInterpreter` enabled.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: A**

#### Architectural Explanation
**Amazon Bedrock Batch Inference** (`CreateModelInvocationJob`) is purpose-built for large-scale asynchronous offline processing of JSONL datasets stored in Amazon S3. It provides two decisive advantages:
1. **50% Cost Discount**: Batch Inference is billed at **50% off** standard On-Demand input and output token prices.
2. **Quota Isolation**: Batch Inference jobs execute on separate asynchronous capacity pools and **do not consume** your account's real-time On-Demand RPM/TPM quotas—protecting the live 24x7 fraud chatbot from throttling.

#### Distractor Analysis
- **B is incorrect**: Increasing concurrency 100x against the same On-Demand endpoint worsens throttling and causes complete outage for the live chatbot.
- **C is incorrect**: Paying 24x7 for 6 months of Provisioned Throughput when the workload only runs 6 hours per week (3.5% utilization) is a massive cost anti-pattern.
- **D is incorrect**: Wrapping simple single-turn classification in an Agent with Code Interpreter multiplies token usage and still consumes real-time On-Demand quotas.

</details>

---

### Question 7

An MLOps engineer is updating a mission-critical Amazon SageMaker Real-Time Endpoint (`prod-fraud-llm-endpoint`) from an FP16 model container (`Blue` fleet) to a newly quantized AWQ 4-bit model container (`Green` fleet). The engineer wants to shift **15% of live traffic** to the new `Green` fleet for **20 minutes** while monitoring two Amazon CloudWatch alarms (`HighInvocation5XXErrors` and `P99ModelLatencyBreach`). If neither alarm enters the `ALARM` state during the 20-minute window, SageMaker should shift the remaining 85% of traffic to the `Green` fleet; if either alarm triggers, SageMaker must automatically roll back 100% of traffic to the `Blue` fleet.

How should the engineer configure the `DeploymentConfig` on the SageMaker Endpoint update?

- **A.** Configure `BlueGreenUpdatePolicy` with `TrafficRoutingConfiguration={"Type": "ALL_AT_ONCE"}` and set `WaitIntervalInSeconds=0`.
- **B.** Configure `BlueGreenUpdatePolicy` with `TrafficRoutingConfiguration={"Type": "LINEAR", "LinearStepSize": {"Type": "CAPACITY_PERCENT", "Value": 15}, "WaitIntervalInSeconds": 1200}` and omit `AutoRollbackConfiguration`.
- **C.** Configure `BlueGreenUpdatePolicy` with `TrafficRoutingConfiguration={"Type": "CANARY", "CanarySize": {"Type": "CAPACITY_PERCENT", "Value": 15}, "WaitIntervalInSeconds": 1200}` and specify both CloudWatch alarms in `AutoRollbackConfiguration`.
- **D.** Delete the existing SageMaker Endpoint, wait 20 minutes, and create a new endpoint with the same name.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: C**

#### Architectural Explanation
In Amazon SageMaker **Blue/Green Deployments**, **`Type="CANARY"`** shifts a single fixed initial percentage (`CanarySize={"Type": "CAPACITY_PERCENT", "Value": 15}`) to the new green fleet, bakes for `WaitIntervalInSeconds=1200` (20 minutes), and then shifts the remaining 85% in one step if all CloudWatch alarms listed in **`AutoRollbackConfiguration`** remain healthy (`OK`). If any alarm fires during the canary window or termination wait period, SageMaker automatically rolls traffic back to the blue fleet.

#### Distractor Analysis
- **A is incorrect**: `ALL_AT_ONCE` shifts 100% of traffic immediately, exposing all users to potential regressions.
- **B is incorrect**: `LINEAR` shifts traffic in repeated equal steps (15% $\rightarrow$ 30% $\rightarrow$ 45% $\rightarrow$ ...) every 20 minutes rather than a single 15% canary followed by 85%, and omitting `AutoRollbackConfiguration` prevents automated rollback on alarm breach.
- **D is incorrect**: Deleting and recreating the endpoint causes 20+ minutes of complete production downtime.

</details>

---

### Question 8

A startup has deployed an internal HR policy classifier built on a fine-tuned 350-million-parameter RoBERTa model (~700 MB artifact size, CPU-compatible). The model is invoked only when a new employee onboarding ticket is filed—roughly 40 times per day during weekday business hours, with zero traffic at night. The startup wants a fully managed endpoint on Amazon SageMaker AI that automatically scales down to **\$0 compute cost** when idle and requires zero instance type selection or Auto Scaling policy configuration, and can tolerate a 2–4 second cold start on the first request of the morning.

Which SageMaker deployment option is the MOST cost-effective and operationally simple choice?

- **A.** **Amazon SageMaker Real-Time Inference** on two `ml.p4d.24xlarge` GPU instances across two Availability Zones.
- **B.** **Amazon SageMaker Batch Transform** triggered manually by an engineer every Friday afternoon.
- **C.** **Amazon Bedrock Provisioned Throughput** with a 6-month commitment.
- **D.** **Amazon SageMaker Serverless Inference** configured with `MemorySizeInMB=2048` and `MaxConcurrency=5` (with no Provisioned Concurrency).

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: D**

#### Architectural Explanation
For small CPU-compatible models (<10 GB container, <4 MB payload, <60s processing time) with intermittent traffic that can tolerate occasional cold starts, **Amazon SageMaker Serverless Inference** is the ideal choice. It requires no instance type management, automatically scales down to zero (`$0` idle cost), and bills purely per millisecond of compute and data processed.

#### Distractor Analysis
- **A is incorrect**: Two `ml.p4d.24xlarge` instances cost over \$45,000/month for a 700 MB model invoked 40 times a day.
- **B is incorrect**: Manual weekly Batch Transform does not provide real-time ticket classification when onboarding tickets are filed.
- **C is incorrect**: Bedrock Provisioned Throughput is for large Bedrock Foundation Models and charges 24x7 hourly rates.

</details>

---

### Question 9

An enterprise has imported a custom 8-billion-parameter Qwen model into **Amazon Bedrock Custom Model Import (CMI)**. During the workday, traffic is steady, and inference latency averages 650 ms. However, every morning at 8:00 AM when the first employee sends a prompt after 10 hours of overnight inactivity, the `Converse` API call returns a `ModelNotReadyException` error, and the model becomes responsive approximately 45 seconds later.

Why does `ModelNotReadyException` occur after overnight inactivity, and what are the TWO valid engineering strategies to handle or prevent it? **(Choose TWO)**

- **A.** Re-upload the `.safetensors` weights to Amazon S3 and call `CreateModelImportJob` every morning at 7:00 AM because CMI deletes imported model weights from S3 after 8 hours.
- **B.** Because Bedrock CMI scales Custom Model Units (CMUs) to zero after ~5 minutes of inactivity, implement client-side retry logic with backoff specifically catching `ModelNotReadyException` while Bedrock reloads the model weights into compute memory.
- **C.** Schedule an Amazon EventBridge rule (e.g., running every 5 minutes during core business hours, or once at 7:55 AM) to invoke a minimal 1-token warm-up request against the imported model ARN so the CMUs are pre-warmed before employees arrive.
- **D.** Purchase a 6-month Provisioned Throughput commitment on the imported model ARN, as CMI models cannot be invoked without Provisioned Throughput.
- **E.** Switch the API call from `bedrock-runtime` to `bedrock-agent` control-plane API.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answers: B and C**

#### Architectural Explanation
In **Amazon Bedrock Custom Model Import (CMI)**, Bedrock scales the active **Custom Model Units (CMUs)** down to zero after roughly 5 minutes of inactivity so you are not billed when the model is idle. When a request arrives after scale-to-zero, Bedrock begins loading the model back into memory and immediately returns `ModelNotReadyException`.
1. **Reactive Handling (Option B)**: Configure the client application to catch `ModelNotReadyException` and retry with exponential backoff over 30–90 seconds until the model finishes warming up.
2. **Proactive Pre-Warming (Option C)**: Trigger an EventBridge scheduled rule right before business hours (or periodically during business hours) to warm the model so interactive users never experience a cold start.

#### Distractor Analysis
- **A is incorrect**: CMI never deletes your S3 artifacts or imported model resource when scaling to zero.
- **D is incorrect**: CMI uses on-demand CMUs, not Provisioned Throughput Model Units.
- **E is incorrect**: `bedrock-agent` cannot invoke foundation models.

</details>

---

### Question 10

A healthcare diagnostics company runs three distinct deep learning models on Amazon SageMaker AI:
1. `biomedical-ner-7b` (requires 1 NVIDIA A10G GPU)
2. `clinical-coding-8b` (requires 1 NVIDIA A10G GPU)
3. `radiology-report-summarizer-13b` (requires 2 NVIDIA A10G GPUs)

Currently, each model is deployed on its own separate SageMaker Real-Time Endpoint, resulting in low average GPU utilization and high EC2 instance overhead. The engineering team wants to consolidate all three models onto a single shared fleet of `ml.g5.12xlarge` instances (each having 4x NVIDIA A10G GPUs) while guaranteeing exact GPU allocation per model (`1`, `1`, and `2` GPUs respectively) and allowing each model to scale its replica `CopyCount` independently based on its own `InvocationsPerCopy` metric (including scaling individual models down to `0` copies when unused).

Which SageMaker capability supports this architecture?

- **A.** Deploy a legacy CPU-only **SageMaker Multi-Model Endpoint (MME)** using `SAGEMAKER_MMS_MODEL_STORE` on an `ml.t3.medium` instance.
- **B.** Deploy a single SageMaker Real-Time Endpoint with `ManagedInstanceScaling` enabled, and create three **SageMaker Inference Components** specifying each model's `ComputeResourceRequirements` (`NumberOfAcceleratorDevicesRequired`) and independent Application Auto Scaling policies.
- **C.** Deploy a single **SageMaker Asynchronous Inference** endpoint with a single container image that concatenates all three models into a single 28B weight file.
- **D.** Use **Amazon SageMaker Serverless Inference** to allocate 2 NVIDIA A10G GPUs per model.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: B**

#### Architectural Explanation
**Amazon SageMaker Inference Components** allow you to bin-pack multiple distinct models onto a shared GPU endpoint fleet (`ml.g5.12xlarge` with `ManagedInstanceScaling="ENABLED"`). Each `InferenceComponent` explicitly declares its hardware slice (`NumberOfAcceleratorDevicesRequired: 1.0` or `2.0`, CPU cores, and RAM), has its own `CopyCount`, and supports independent **Application Auto Scaling** policies (including scaling individual Inference Components down to `MinCapacity=0` copies when idle).

#### Distractor Analysis
- **A is incorrect**: Traditional Multi-Model Endpoints (MME) share a single container framework, do not allow pinning specific GPU counts per model, and suffer severe cold-start latency when swapping multi-GB LLMs from S3.
- **C is incorrect**: Concatenating three separate model architectures into a single weight file is invalid and prevents independent scaling.
- **D is incorrect**: SageMaker Serverless Inference does not support GPUs.

</details>

---

### Question 11

A banking customer has fine-tuned `amazon.nova-lite-v1:0` natively inside Amazon Bedrock using a supervised **Bedrock Model Customization (Fine-Tuning)** job (`create_model_customization_job`). The training job has completed with status `COMPLETED` and produced a custom model ARN (`arn:aws:bedrock:us-east-1:111122223333:custom-model/amazon.nova-lite-v1:0:12k/cust9988`).

When the developer immediately attempts to pass this `custom-model` ARN as the `modelId` in a `bedrock_runtime.converse()` API call, the request fails with a `ValidationException` stating that on-demand invocation is not supported for this custom model.

What must the developer do before invoking a model that was fine-tuned natively inside Amazon Bedrock?

- **A.** Export the fine-tuned Amazon Nova weights to an external HuggingFace account and run them on a local laptop.
- **B.** Purchase **Provisioned Throughput** (`create_provisioned_model_throughput`) for the `custom-model` ARN, wait for the `provisioned-model` status to become `InService`, and pass the resulting `provisioned-model` ARN as the `modelId` in `converse()`.
- **C.** Call `prepare_agent` on the `custom-model` ARN.
- **D.** Prefix the `custom-model` ARN with `us.` to turn it into a system-defined Cross-Region Inference Profile.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: B**

#### Architectural Explanation
Models customized natively via **Amazon Bedrock Model Customization jobs** (Fine-Tuning or Continued Pre-Training on first-party/supported Bedrock models) **cannot** be invoked via standard On-Demand pay-per-token APIs. You **must** provision dedicated capacity by purchasing **Provisioned Throughput** (`create_provisioned_model_throughput` with No Commit [available for select custom models during testing] or 1-Month / 6-Month commitment) and pass the **`provisioned-model` ARN** in your `InvokeModel` or `Converse` calls.

#### Distractor Analysis
- **A is incorrect**: First-party Amazon Nova weights fine-tuned in Bedrock cannot be exported as raw weights outside AWS.
- **C is incorrect**: `prepare_agent` is for Bedrock Agents, not fine-tuned model hosting.
- **D is incorrect**: System-defined Cross-Region Inference Profiles (`us.*`) apply only to base Foundation Models, not customer fine-tuned models.

</details>

---

### Question 12

An AI engineering team wants to run a dark launch (shadow test) of a new quantized `Llama-3.1-70B-FP8` model alongside their existing production `Llama-3.1-70B-FP16` model on a single **Amazon SageMaker Real-Time Endpoint** without exposing any regular end users to the `FP8` model's outputs until internal QA engineers have validated 1,000 test prompts in production.

How can the team configure the SageMaker Endpoint `ProductionVariants` to support this requirement?

- **A.** Set `InitialVariantWeight=0.5` on both variants and have customer support manually apologize to the 50% of end users who receive unvalidated `FP8` responses.
- **B.** Deploy both variants on SageMaker Serverless Inference with 4x NVIDIA H100 GPUs.
- **C.** Configure two `ProductionVariants` (`Variant-FP16` and `Variant-FP8`) and route internal QA test traffic exclusively to the candidate variant by explicitly passing the **`TargetVariant="Variant-FP8"`** header in `invoke_endpoint` calls from the QA test harness.
- **D.** Pass `TargetModel="Variant-FP8"` to an Amazon Bedrock Knowledge Base `Retrieve` call.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: C**

#### Architectural Explanation
When an Amazon SageMaker Real-Time Endpoint is configured with multiple **`ProductionVariants`**, callers can bypass the weighted random traffic distribution by specifying the **`TargetVariant`** parameter (`X-Amzn-SageMaker-Target-Variant` HTTP header) in `invoke_endpoint`. By routing normal customer traffic to `Variant-FP16` and having the internal QA/shadow harness pass `TargetVariant="Variant-FP8"`, the team can validate `Variant-FP8` in the live production environment with zero impact on end users.

#### Distractor Analysis
- **A is incorrect**: Sending 50% of live user traffic to an unvalidated model violates the zero-user-exposure requirement.
- **B is incorrect**: SageMaker Serverless Inference does not support GPUs (`H100`).
- **D is incorrect**: Bedrock Knowledge Bases do not route SageMaker `ProductionVariants`.

</details>

---

### Question 13

A Principal Machine Learning Architect is evaluating whether to deploy a self-hosted 70B LLM on **AWS Inferentia2 (`ml.inf2.48xlarge`)** instances vs. **NVIDIA GPU (`ml.g5.48xlarge`)** instances on Amazon SageMaker Real-Time Endpoints using the Large Model Inference (LMI) container.

Which two technical requirements apply specifically when deploying a HuggingFace LLM onto AWS Inferentia2 (`ml.inf2`) instances using the LMI NeuronX container? **(Choose TWO)**

- **A.** The model must be compiled ahead of time (AOT) or on startup into a **Neuron Executable File Format (NEFF)** graph using the AWS Neuron SDK (`transformers-neuronx` / `neuronx-distributed`) for a fixed combination of batch size, sequence length, and `tensor_parallel_degree` (Neuron core count).
- **B.** Set the LMI container engine/rolling-batch backend to use the **NeuronX** runtime (`OPTION_ENTRYPOINT=djl_python.transformers_neuronx` with `OPTION_TENSOR_PARALLEL_DEGREE` matching the number of NeuronCores, such as `24` on `ml.inf2.48xlarge`).
- **C.** Enable CUDA 12.4 and NVIDIA TensorRT-LLM (`OPTION_ROLLING_BATCH=trtllm`), as AWS Inferentia2 chips natively execute NVIDIA CUDA PTX instructions.
- **D.** Store the model weights exclusively as Microsoft Word `.docx` files in Amazon S3.
- **E.** Disable all tensor parallelism because `ml.inf2.48xlarge` contains only a single 4 GB accelerator core.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answers: A and B**

#### Architectural Explanation
1. **Neuron Graph Compilation (NEFF) (Option A)**: AWS Inferentia2 (`ml.inf2`) accelerators use the **AWS Neuron SDK** rather than NVIDIA CUDA. Models are compiled into **NEFF (Neuron Executable File Format)** artifacts targeting specific `batch_size`, `n_positions` (context length), and `tp_degree` (NeuronCores). Pre-compiling and caching the NEFF artifacts in Amazon S3 drastically reduces endpoint startup time.
2. **LMI NeuronX Engine Configuration (Option B)**: In SageMaker LMI containers on `ml.inf2`, you configure the `transformers-neuronx` / `vllm` Neuron backend and set `OPTION_TENSOR_PARALLEL_DEGREE` to match the available NeuronCores (an `ml.inf2.48xlarge` provides 12 Inferentia2 chips = **24 NeuronCores** with 384 GB total HBM).

#### Distractor Analysis
- **C is incorrect**: NVIDIA CUDA and TensorRT-LLM only run on NVIDIA GPUs (`ml.g5`, `ml.g6e`, `ml.p4d`, `ml.p5`), not AWS Inferentia2 ASICs.
- **D is incorrect**: Model weights are stored in `.safetensors` / PyTorch / compiled `.neff` format, never `.docx`.
- **E is incorrect**: `ml.inf2.48xlarge` has 24 NeuronCores (384 GB HBM) and requires tensor parallelism across multiple NeuronCores to fit a 70B model.

</details>

---

### Question 14

An enterprise runs an internal code documentation generator that processes 5,000 GitHub pull requests per hour (24 hours a day, 7 days a week) at a flat, predictable rate of 4,200,000 input tokens and 800,000 output tokens per minute using `anthropic.claude-3-5-sonnet-20241022-v2:0` on Amazon Bedrock. Compliance regulations strictly prohibit inference requests from leaving the `eu-central-1` (Frankfurt) AWS Region under any circumstances. Furthermore, the engineering SLA requires guaranteed token throughput with zero throttling during peak European business hours.

Which Amazon Bedrock deployment model satisfies the strict single-region `eu-central-1` residency mandate AND provides guaranteed dedicated throughput for this steady 24x7 workload?

- **A.** Use the `eu.anthropic.claude-3-5-sonnet-20241022-v2:0` Cross-Region Inference Profile so traffic routes across Dublin (`eu-west-1`), Paris (`eu-west-3`), and Stockholm (`eu-north-1`).
- **B.** Use **Amazon Bedrock Batch Inference** with a 24-hour completion window.
- **C.** Use **Amazon Bedrock Custom Model Import (CMI)** to import Anthropic Claude 3.5 Sonnet proprietary weights from HuggingFace.
- **D.** Purchase **Amazon Bedrock Provisioned Throughput** (Model Units with a 1-month or 6-month commitment) directly in the `eu-central-1` region and invoke the `provisioned-model` ARN in `eu-central-1`.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: D**

#### Architectural Explanation
When a workload has:
1. **Strict single-region data residency** (`eu-central-1` only—ruling out `eu.*` Cross-Region Inference Profiles), AND
2. **Sustained 24x7 predictable high utilization requiring guaranteed capacity SLAs**,
purchasing **Amazon Bedrock Provisioned Throughput** (Model Units) in `eu-central-1` provides dedicated, reserved token throughput within that exact single region and offers discounted hourly rates via 1-month or 6-month commitments.

#### Distractor Analysis
- **A is incorrect**: The `eu.*` Cross-Region Inference Profile routes inference compute across multiple EU regions (`eu-west-1`, `eu-west-3`, `eu-north-1`, `eu-central-1`), violating the strict rule that data must never leave `eu-central-1`.
- **B is incorrect**: PR documentation generation requires timely feedback per hour, not offline 24-hour batch jobs.
- **C is incorrect**: Anthropic Claude 3.5 Sonnet is a closed-weight proprietary model; its weights cannot be downloaded or imported via Custom Model Import.

</details>

---

### Question 15

An ML engineer is optimizing latency for a 70-billion-parameter LLM deployed on a SageMaker Real-Time Endpoint (`ml.p4d.24xlarge`) using the Large Model Inference (LMI) container. Profiling reveals that autoregressive decoding is memory-bandwidth bound because each output token requires reading all 70B parameters from GPU HBM. The workload consists of structured code refactoring and JSON transformation where 70–80% of output tokens are highly predictable.

Which LMI feature can accelerate token generation throughput by 1.5x–2.5x without changing the mathematical output distribution of the 70B target model?

- **A.** Configure **Speculative Decoding** (`OPTION_SPECULATIVE_DRAFT_MODEL`) using a smaller, fast draft model from the same family (e.g., an 8B model) to propose $K$ candidate tokens in parallel, which the 70B target model verifies in a single forward pass.
- **B.** Set `OPTION_TENSOR_PARALLEL_DEGREE=1` and run the 70B model on CPU swap memory.
- **C.** Increase the HTTP client `connect_timeout` from 5 seconds to 600 seconds.
- **D.** Enable Amazon CloudWatch Container Insights on the SageMaker Endpoint.

<details>
<summary><strong>Answer & Deep-Dive Explanation</strong></summary>

**Correct Answer: A**

#### Architectural Explanation
**Speculative Decoding** (`OPTION_SPECULATIVE_DRAFT_MODEL` or Lookahead/Medusa decoding in LMI vLLM/TensorRT-LLM) overcomes the memory-bandwidth bottleneck of large autoregressive LLMs. A tiny draft model rapidly generates $K$ draft tokens, and the 70B target model evaluates all $K$ tokens simultaneously in **one single parallel forward pass**. Because verifying $K$ tokens in parallel takes nearly the same time as generating 1 token on a memory-bound GPU, speculative decoding achieves 1.5x–2.5x lower latency on predictable text/code/JSON while preserving the exact output distribution of the 70B model.

#### Distractor Analysis
- **B is incorrect**: Offloading to CPU swap memory slows inference by 100x+.
- **C is incorrect**: Client socket timeouts do not speed up GPU token decoding.
- **D is incorrect**: CloudWatch Container Insights is a monitoring tool, not an inference acceleration technique.

</details>
