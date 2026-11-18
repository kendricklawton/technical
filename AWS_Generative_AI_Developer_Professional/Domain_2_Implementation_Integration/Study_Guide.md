# Domain 2: Implementation and Integration (26%) — Deep-Dive Study Guide

> **Exam Weighting**: 26% of the AWS Certified Generative AI Developer - Professional (AIP-C01) Exam  
> **Target Audience**: Principal Cloud Architects, Staff AI/ML Engineers, and Senior Distributed Systems Developers building production-grade Agentic AI, multi-model inference fleets, and event-driven Generative AI architectures on AWS.

---

## Table of Contents

1. [Task 2.1: Implement Agentic AI Solutions & Tool Integrations](#task-21-implement-agentic-ai-solutions--tool-integrations)
   - [1.1 Amazon Bedrock Agents Architecture & ReAct Loop](#11-amazon-bedrock-agents-architecture--react-loop)
   - [1.2 Prompt Lifecycle Phases & Custom Orchestration](#12-prompt-lifecycle-phases--custom-orchestration)
   - [1.3 Multi-Agent Collaboration on Amazon Bedrock](#13-multi-agent-collaboration-on-amazon-bedrock)
   - [1.4 Action Groups: OpenAPI 3.0 vs. Function Details](#14-action-groups-openapi-30-vs-function-details)
   - [1.5 Lambda Executor Contract & Resource-Based Policies](#15-lambda-executor-contract--resource-based-policies)
   - [1.6 Return of Control (ROC) & User Confirmation](#16-return-of-control-roc--user-confirmation)
   - [1.7 Session State, Prompt Attributes & Cross-Session Memory](#17-session-state-prompt-attributes--cross-session-memory)
   - [1.8 Built-In Action Groups: Code Interpreter & User Input](#18-built-in-action-groups-code-interpreter--user-input)
   - [1.9 Model Context Protocol (MCP) Integration on AWS & Strands Agents SDK](#19-model-context-protocol-mcp-integration-on-aws--strands-agents-sdk)
2. [Task 2.2: Implement Model Deployment Strategies](#task-22-implement-model-deployment-strategies)
   - [2.1 Amazon Bedrock Consumption & Deployment Modes](#21-amazon-bedrock-consumption--deployment-modes)
   - [2.2 Amazon Bedrock Custom Model Import (CMI)](#22-amazon-bedrock-custom-model-import-cmi)
   - [2.3 Amazon SageMaker AI Deployment Strategies](#23-amazon-sagemaker-ai-deployment-strategies)
   - [2.4 SageMaker Large Model Inference (LMI) Containers](#24-sagemaker-large-model-inference-lmi-containers)
   - [2.5 Multi-Model Endpoints (MME) vs. Inference Components & Multi-Adapter LoRA](#25-multi-model-endpoints-mme-vs-inference-components--multi-adapter-lora)
   - [2.6 Safe Model Rollouts: Blue/Green, Canary & Linear Deployments](#26-safe-model-rollouts-bluegreen-canary--linear-deployments)
3. [Task 2.3: Design & Implement Enterprise Integration Architectures](#task-23-design--implement-enterprise-integration-architectures)
   - [3.1 Event-Driven GenAI Architectures with EventBridge & SQS](#31-event-driven-genai-architectures-with-eventbridge--sqs)
   - [3.2 AWS Step Functions Orchestration & Distributed Map](#32-aws-step-functions-orchestration--distributed-map)
   - [3.3 Real-Time Token Streaming to Web & Mobile Clients](#33-real-time-token-streaming-to-web--mobile-clients)
   - [3.4 Private Multi-Account & Hybrid Connectivity via AWS PrivateLink](#34-private-multi-account--hybrid-connectivity-via-aws-privatelink)
4. [Task 2.4: Implement FM API Integrations](#task-24-implement-fm-api-integrations)
   - [4.1 Bedrock Converse & ConverseStream vs. InvokeModel](#41-bedrock-converse--conversestream-vs-invokemodel)
   - [4.2 Native Tool Use (Function Calling) with Converse API](#42-native-tool-use-function-calling-with-converse-api)
   - [4.3 Resiliency Engineering: Rate Limiting, Adaptive Retries & Circuit Breakers](#43-resiliency-engineering-rate-limiting-adaptive-retries--circuit-breakers)
   - [4.4 Multi-Region Failover Chains & Intelligent Prompt Routing](#44-multi-region-failover-chains--intelligent-prompt-routing)
5. [Task 2.5: Application Integration Patterns & Development Tools](#task-25-application-integration-patterns--development-tools)
   - [5.1 Infrastructure as Code (IaC) for GenAI Stacks](#51-infrastructure-as-code-iac-for-genai-stacks)
   - [5.2 Zero-Downtime Agent Versioning, Aliases & CI/CD Pipelines](#52-zero-downtime-agent-versioning-aliases--cicd-pipelines)
   - [5.3 Amazon Q Developer Enterprise Customizations & Workspace Indexing](#53-amazon-q-developer-enterprise-customizations--workspace-indexing)

---

## Task 2.1: Implement Agentic AI Solutions & Tool Integrations

### 1.1 Amazon Bedrock Agents Architecture & ReAct Loop

Amazon Bedrock Agents enable Foundation Models (FMs) to execute multi-step business workflows by combining **Reasoning and Acting (ReAct)** loops, **Knowledge Bases (RAG)**, **Action Groups (external tools/APIs)**, **Guardrails**, and **Cross-Session Memory**.

At runtime, an `InvokeAgent` request triggers a stateful orchestration pipeline managed by the `bedrock-agent-runtime` service:

```mermaid
flowchart TD
    Client["Client Application"] -->|"InvokeAgent (sessionId, memoryId, inputText)"| PreGuard["Guardrail (Input Evaluation)"]
    PreGuard --> PreProc["1. Pre-Processing Phase (Optional Classifier)"]
    PreProc -->|"Valid Input"| Router["2. Routing Classifier (Optional Fast Path)"]
    Router -->|"Complex / Multi-Step"| Orch["3. Orchestration Loop (ReAct: Thought -> Action -> Observation)"]
    Router -->|"Simple Direct Route"| ActionExec["Action Group / KB Query"]
    Orch -->|"Tool Call (OpenAPI / Function)"| Lambda["AWS Lambda / Return of Control (ROC)"]
    Orch -->|"Semantic Retrieval"| KB["Amazon Bedrock Knowledge Base"]
    Orch -->|"Sandbox Python Execution"| CodeInt["AMAZON.CodeInterpreter"]
    Lambda -->|"Observation Payload"| Orch
    KB -->|"Retrieved Chunks + Citations"| Orch
    CodeInt -->|"Stdout / Plots / Files"| Orch
    Orch -->|"Final Answer Ready"| PostProc["4. Post-Processing Phase (Optional Formatting)"]
    PostProc --> PostGuard["Guardrail (Output Evaluation)"]
    PostGuard -->|"ResponseStream (Chunk + Trace + Attribution)"| Client
```

#### Core Build-Time vs. Runtime API Separation
A high-frequency exam distinction is the separation between the control-plane SDK client and data-plane SDK client:
- **`bedrock-agent` (Control Plane)**:
  - `create_agent` / `update_agent`
  - `create_agent_action_group` / `update_agent_action_group`
  - `associate_agent_knowledge_base`
  - `associate_agent_collaborator`
  - `prepare_agent` (compiles the `DRAFT` version with all associated resources)
  - `create_agent_alias` / `update_agent_alias`
- **`bedrock-agent-runtime` (Data Plane)**:
  - `invoke_agent` (stateful agent invocation with `sessionId` and `memoryId`)
  - `invoke_inline_agent` (stateless/dynamic definition of tools and instructions at runtime)
  - `retrieve` / `retrieve_and_generate`
  - `get_agent_memory` / `delete_agent_memory`

---

### 1.2 Prompt Lifecycle Phases & Custom Orchestration

Amazon Bedrock Agents process each conversational turn through up to five distinct prompt templates. Each phase can be enabled/disabled, customized via **Prompt Overrides**, or assigned a specific inference configuration (`temperature`, `topP`, `topK`, `maximumLength`, `stopSequences`) and custom Lambda parser (`parserMode='OVERRIDDEN'`):

| Phase | Default State | Primary Responsibility | When to Customize / Enable |
| :--- | :--- | :--- | :--- |
| **1. Pre-processing** | Disabled | Evaluates user intent, detects prompt injection/out-of-bounds requests, contextualizes input against conversation history, and classifies whether the input can be handled. | Disable for lower latency and token cost when Bedrock Guardrails already handle input safety filtering. |
| **2. Routing Classifier** | Disabled | Uses a lightweight/fast classification prompt to route simple single-step requests directly to a target Action Group or Knowledge Base, bypassing full ReAct reasoning overhead. | Enable when 60%+ of requests map deterministically to a single API lookup or FAQ query to reduce Time-To-First-Token (TTFT) by 40–60%. |
| **3. Orchestration** | **Enabled (Mandatory)** | Drives the iterative ReAct loop (`<thought>`, `<tool_call>`, `<observation>`, `<final_response>`). Coordinates Action Groups, Knowledge Bases, and Collaborator Agents. | Override prompt template to enforce strict domain reasoning rules, XML output schemas, or **Custom Orchestration** via AWS Lambda. |
| **4. Knowledge Base Response Generation** | Enabled (per KB) | Synthesizes retrieved vector chunks into grounded answers when querying an associated Knowledge Base directly. | Customize to enforce citation formats or strict refusal when retrieval scores are low. |
| **5. Post-processing** | Disabled | Formats, redacts, or restructures the final response produced by the Orchestration loop (e.g., transforming prose into strict JSON or executive bullet points) before returning to the caller. | Enable only when strict schema formatting is required at the final output stage; adds an extra LLM invocation latency penalty. |

#### Custom Orchestration with AWS Lambda (`orchestrationType='CUSTOM_ORCHESTRATION'`)
For regulated enterprises requiring deterministic state-machine control interleaved with LLM reasoning (e.g., enforcing a mandatory identity verification or SOX compliance check before any tool invocation regardless of what the LLM decides), Bedrock Agents support **Custom Orchestration**. Instead of Bedrock running the default ReAct loop, Bedrock invokes a customer-managed AWS Lambda function after every lifecycle event (`START`, `MODEL_INVOCATION_OUTPUT`, `TOOL_INVOCATION_OUTPUT`, `REPROMPT`), allowing the Lambda function to inspect the state and explicitly command the next action:

```python
import json
from typing import Dict, Any

def custom_orchestrator_handler(event: Dict[str, Any], context: Any) -> Dict[str, Any]:
    """
    Bedrock Agent Custom Orchestration Lambda Handler.
    Controls transitions between INVOKE_MODEL, INVOKE_TOOL, and FINISH.
    """
    trigger_Event = event.get("event", "START")
    state = event.get("state", {})
    messages = state.get("messages", [])

    if trigger_Event == "START":
        # Force an immediate deterministic tool call before invoking the FM
        return {
            "version": "1.0",
            "actionEvent": "INVOKE_TOOL",
            "actionInput": {
                "toolUse": {
                    "toolUseId": "init-auth-check-001",
                    "name": " ComplianceActionGroup::verifyUserClearance",
                    "input": {
                        "userId": event.get("sessionAttributes", {}).get("user_id", "anonymous")
                    }
                }
            },
            "state": state
        }

    elif trigger_Event == "TOOL_INVOCATION_OUTPUT":
        # Pass the tool result into the Foundation Model to synthesize the response
        return {
            "version": "1.0",
            "actionEvent": "INVOKE_MODEL",
            "actionInput": {
                "messages": messages
            },
            "state": state
        }

    elif trigger_Event == "MODEL_INVOCATION_OUTPUT":
        # Complete the turn and return the FM output to the caller
        final_text = messages[-1]["content"][0].get("text", "Completed.")
        return {
            "version": "1.0",
            "actionEvent": "FINISH",
            "actionInput": {
                "text": final_text
            },
            "state": state
        }
```

---

### 1.3 Multi-Agent Collaboration on Amazon Bedrock

Complex enterprise domains often exceed the context window, tool limit, or reasoning accuracy of a single monolithic agent. When a single agent has more than 10–15 Action Groups, tool-selection accuracy degrades due to overlapping tool descriptions. **Amazon Bedrock Multi-Agent Collaboration** solves this through a hierarchical **Supervisor / Sub-Agent (Collaborator)** architecture.

```mermaid
sequenceDiagram
    participant User as Client App
    participant Sup as Supervisor Agent (Claude 3.5 Sonnet)
    participant Fin as Sub-Agent: Financial Analyst
    participant SQL as Sub-Agent: Redshift SQL Agent
    participant Sec as Sub-Agent: Compliance Auditor

    User->>Sup: "Analyze Q3 EMEA margin drop and verify SOX controls"
    Note over Sup: Mode: SUPERVISOR or SUPERVISOR_ROUTER
    Sup->>SQL: Invoke Collaborator (Query Q3 EMEA revenue/COGS)
    SQL-->>Sup: Observation: Margin fell 4.2% due to logistics
    Sup->>Fin: Invoke Collaborator (Run variance decomposition)
    Fin-->>Sup: Observation: Freight spot rates drove 78% of variance
    Sup->>Sec: Invoke Collaborator (Verify SOX sign-off on Q3 adjustments)
    Sec-->>Sup: Observation: Control SOX-404-B passed
    Sup-->>User: Synthesized Executive Report + Trace Citations
```

#### Supervisor Collaboration Modes
When configuring a Supervisor Agent (`agentCollaboration`), you must choose between two collaboration modes:

| Feature / Dimension | `SUPERVISOR` Mode (Full Orchestration) | `SUPERVISOR_ROUTER` Mode (Routing + Escalation) |
| :--- | :--- | :--- |
| **Execution Pattern** | The Supervisor decomposes complex multi-domain queries into sub-tasks, invokes multiple collaborator agents sequentially or in parallel, synthesizes intermediate outputs, and generates a unified answer. | A lightweight routing classifier first attempts to route the user request directly to a single matching collaborator sub-agent without Supervisor synthesis overhead. |
| **Fallback Behavior** | Always orchestrates through the Supervisor's ReAct loop. | If user intent is ambiguous or spans multiple sub-agents, automatically falls back to full `SUPERVISOR` orchestration mode. |
| **Latency & Token Cost** | Higher latency and token consumption due to multi-turn Supervisor synthesis. | Lowest latency and cost for single-intent utterances; relays sub-agent response directly to the caller. |
| **Conversation History Sharing** | Configurable per collaborator (`relayConversationHistory='TO_COLLABORATOR'` or `'DISABLED'`). | Passes full conversation context when `TO_COLLABORATOR` is enabled so sub-agents handle follow-up turns seamlessly. |

#### Configuring Multi-Agent Collaboration via `boto3`

```python
import boto3

bedrock_agent = boto3.client("bedrock-agent", region_name="us-east-1")

# 1. Enable Supervisor Mode on the Parent Agent
bedrock_agent.update_agent(
    agentId="SUPERVISOR12",
    agentName="enterprise-wealth-supervisor",
    foundationModel="us.anthropic.claude-3-5-sonnet-20241022-v2:0",
    agentResourceRoleArn="arn:aws:iam::111122223333:role/BedrockSupervisorRole",
    agentCollaboration="SUPERVISOR_ROUTER",
    instruction=(
        "You are the Executive Wealth Management Supervisor. Delegate quantitative "
        "portfolio risk calculations to the RiskCollaborator and tax harvesting queries "
        "to the TaxCollaborator. Synthesize their outputs into a unified client brief."
    )
)

# 2. Associate Sub-Agent Collaborators (Requires an Agent Alias ARN for each Sub-Agent)
bedrock_agent.associate_agent_collaborator(
    agentId="SUPERVISOR12",
    agentVersion="DRAFT",
    agentDescriptor={
        "aliasArn": "arn:aws:bedrock:us-east-1:111122223333:agent-alias/RISKAGENT1/PRODALIAS1"
    },
    collaboratorName="RiskCollaborator",
    collaborationInstruction=(
        "Invoke this collaborator whenever the user asks for Value-at-Risk (VaR), "
        "Sharpe ratio, Monte Carlo simulations, or fixed-income duration exposure."
    ),
    relayConversationHistory="TO_COLLABORATOR"
)

# 3. Prepare the Supervisor Agent to compile the collaborator routing table
bedrock_agent.prepare_agent(agentId="SUPERVISOR12")
```

---

### 1.4 Action Groups: OpenAPI 3.0 vs. Function Details

Action Groups define the external tools an agent can invoke. Bedrock supports two schema definition mechanisms:

1. **OpenAPI 3.0 Schema (`apiSchema`)**:
   - Defined in JSON or YAML (inline payload or stored in Amazon S3).
   - Must strictly adhere to OpenAPI 3.0.0+.
   - **Critical Requirement**: Every path/method operation **must** include an `operationId` and rich `description` fields for the path, parameters, and `requestBody` properties. The FM relies entirely on these descriptions to extract parameters from natural language.
2. **Function Details (`functionSchema`)**:
   - A simplified declarative list of functions (`name`, `description`, `parameters` map with types `string`, `number`, `integer`, `boolean`, `array`, and `required` flags).
   - Ideal when you do not need REST path/method semantics (`GET /orders/{id}` vs `POST /orders`) and want minimal schema maintenance.

#### Complete OpenAPI 3.0 Schema with `x-requireConfirmation` Extension

```yaml
openapi: 3.0.1
info:
  title: Enterprise Treasury Wire Transfer API
  version: 1.0.0
  description: API for querying institutional treasury accounts and initiating wire transfers.
paths:
  /accounts/{accountId}/balance:
    get:
      operationId: getAccountBalance
      summary: Retrieve real-time treasury account balance
      description: Returns available and ledger balances for an institutional account ID.
      parameters:
        - name: accountId
          in: path
          required: true
          description: The 10-digit institutional treasury account identifier (e.g., TR-90817263).
          schema:
            type: string
      responses:
        "200":
          description: Account balance retrieved successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  accountId:
                    type: string
                  availableBalanceUsd:
                    type: number
  /wires/initiate:
    post:
      operationId: initiateWireTransfer
      summary: Initiate an outbound Fedwire transfer
      description: Executes an irrevocable outbound wire transfer. Requires explicit user confirmation.
      x-requireConfirmation: ENABLED
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - sourceAccountId
                - destinationRoutingNumber
                - amountUsd
              properties:
                sourceAccountId:
                  type: string
                  description: The 10-digit source treasury account ID.
                destinationRoutingNumber:
                  type: string
                  description: The 9-digit ABA routing number of the receiving institution.
                amountUsd:
                  type: number
                  description: Transfer amount in USD.
      responses:
        "200":
          description: Wire transfer queued
          content:
            application/json:
              schema:
                type: object
                properties:
                  wireReferenceId:
                    type: string
                  status:
                    type: string
```

---

### 1.5 Lambda Executor Contract & Resource-Based Policies

When a Bedrock Agent invokes an AWS Lambda Action Group (`actionGroupExecutor={'lambda': lambda_arn}`), two strict technical requirements apply:

#### 1. Resource-Based Permission on the Lambda Function
The Agent's IAM Service Role (`AmazonBedrockExecutionRoleForAgents_*`) is **not** sufficient on its own to invoke the Lambda function. You **must** attach a resource-based policy on the AWS Lambda function granting `bedrock.amazonaws.com` permission to call `lambda:InvokeFunction`, scoped with `aws:SourceAccount` and `aws:SourceArn`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowBedrockAgentInvoke",
      "Effect": "Allow",
      "Principal": {
        "Service": "bedrock.amazonaws.com"
      },
      "Action": "lambda:InvokeFunction",
      "Resource": "arn:aws:lambda:us-east-1:111122223333:function:erp-order-lookup",
      "Condition": {
        "StringEquals": {
          "aws:SourceAccount": "111122223333"
        },
        "ArnLike": {
          "aws:SourceArn": "arn:aws:bedrock:us-east-1:111122223333:agent/ABCDEFGHIJ"
        }
      }
    }
  ]
}
```

#### 2. Exact Lambda Input Event and Response Envelope
If your Lambda function returns a standard API Gateway response (`{"statusCode": 200, "body": "..."}`), **the Bedrock Agent will fail with a `BadGatewayException` / `DependencyFailedException`**. The Lambda response must strictly match the Bedrock Agent Action Group contract (`messageVersion: "1.0"`):

```python
import json
from typing import Any, Dict

def lambda_handler(event: Dict[str, Any], context: Any) -> Dict[str, Any]:
    """
    Production Lambda Executor supporting both OpenAPI and Function Detail schemas.
    """
    action_group = event.get("actionGroup", "")
    message_version = event.get("messageVersion", "1.0")
    session_attributes = event.get("sessionAttributes", {})
    prompt_session_attributes = event.get("promptSessionAttributes", {})

    # Extract parameters from list of {"name": ..., "type": ..., "value": ...}
    raw_params = event.get("parameters", [])
    params = {p["name"]: p["value"] for p in raw_params}

    # If OpenAPI POST requestBody is used, extract from requestBody.content
    request_body = event.get("requestBody", {}).get("content", {})
    if "application/json" in request_body:
        props = request_body["application/json"].get("properties", [])
        for prop in props:
            params[prop["name"]] = prop["value"]

    # Determine whether invoked via OpenAPI (apiPath + httpMethod) or Function Schema (function)
    if "apiPath" in event:
        api_path = event["apiPath"]
        http_method = event["httpMethod"]
        result_payload = {
            "orderId": params.get("orderId", "ORD-999"),
            "status": "SHIPPED",
            "tenantId": session_attributes.get("tenant_id", "default")
        }
        action_response = {
            "actionGroup": action_group,
            "apiPath": api_path,
            "httpMethod": http_method,
            "httpStatusCode": 200,
            "responseBody": {
                "application/json": {
                    "body": json.dumps(result_payload)
                }
            }
        }
    else:
        function_name = event["function"]
        result_payload = {
            "orderId": params.get("orderId", "ORD-999"),
            "status": "SHIPPED"
        }
        action_response = {
            "actionGroup": action_group,
            "function": function_name,
            "functionResponse": {
                "responseState": "SUCCESS",  # Optional: "FAILURE" or "REPROMPT"
                "responseBody": {
                    "TEXT": {
                        "body": json.dumps(result_payload)
                    }
                }
            }
        }

    return {
        "messageVersion": message_version,
        "response": action_response,
        "sessionAttributes": session_attributes,
        "promptSessionAttributes": prompt_session_attributes
    }
```

> [!TIP]
> **Exam Trap (`responseState` in Function Response)**: Inside `functionResponse`, you can set `"responseState": "FAILURE"` (instructs the agent to abort the action and report the failure) or `"responseState": "REPROMPT"` (instructs the agent to retry or ask the user to clarify invalid parameters).

---

### 1.6 Return of Control (ROC) & User Confirmation

Not all tools can or should be executed directly by an AWS Lambda function inside the Bedrock VPC environment.
- **Scenario A (Client-Side Execution / On-Premises / Browser Context)**: The tool must run on the user's mobile device, inside a legacy on-premises firewall without inbound connectivity, or using the caller's ephemeral frontend OAuth token.
- **Scenario B (Human-in-the-Loop Approval)**: A destructive or financial action (e.g., `initiateWireTransfer`, `terminateCluster`) requires explicit human confirmation before execution.

#### Complete Client-Side Return of Control (ROC) Handler in `boto3`

```python
import boto3
import json

runtime = boto3.client("bedrock-agent-runtime", region_name="us-east-1")

def invoke_agent_with_roc(agent_id: str, alias_id: str, session_id: str, user_input: str) -> str:
    response = runtime.invoke_agent(
        agentId=agent_id,
        agentAliasId=alias_id,
        sessionId=session_id,
        inputText=user_input
    )

    completion_text = ""
    return_control_payload = None

    for event in response["completion"]:
        if "chunk" in event:
            completion_text += event["chunk"]["bytes"].decode("utf-8")
        elif "returnControl" in event:
            return_control_payload = event["returnControl"]

    if not return_control_payload:
        return completion_text

    # Agent paused for Return of Control (ROC)
    invocation_id = return_control_payload["invocationId"]
    invocation_inputs = return_control_payload["invocationInputs"]
    results = []

    for item in invocation_inputs:
        if "functionInvocationInput" in item:
            fn_input = item["functionInvocationInput"]
            action_group = fn_input["actionGroup"]
            fn_name = fn_input["function"]
            params = {p["name"]: p["value"] for p in fn_input.get("parameters", [])}

            # Execute locally or check human confirmation
            local_output = {"wireReferenceId": "FW-20261116-8821", "status": "EXECUTED", "params": params}

            results.append({
                "functionResult": {
                    "actionGroup": action_group,
                    "function": fn_name,
                    "confirmationState": "CONFIRM",  # Or "DENY" if user rejected
                    "responseState": "SUCCESS",
                    "responseBody": {
                        "TEXT": {
                            "body": json.dumps(local_output)
                        }
                    }
                }
            })

    # Resume the suspended Agent Session with the invocationId and results
    resume_response = runtime.invoke_agent(
        agentId=agent_id,
        agentAliasId=alias_id,
        sessionId=session_id,
        sessionState={
            "invocationId": invocation_id,
            "returnControlInvocationResults": results
        }
    )

    final_answer = ""
    for event in resume_response["completion"]:
        if "chunk" in event:
            final_answer += event["chunk"]["bytes"].decode("utf-8")
    return final_answer
```

---

### 1.7 Session State, Prompt Attributes & Cross-Session Memory

Understanding the three state scopes in Bedrock Agents is critical for multi-tenant security and personalization:

| State Mechanism | Lifetime | Injected into FM Prompt? | Sent to Action Group Lambda? | Primary Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **`sessionAttributes`** | Single Session (`sessionId`) | **No** (Hidden from LLM prompt!) | **Yes** (`event['sessionAttributes']`) | Passing sensitive context (JWT tokens, `tenant_id`, internal IAM role names, database connection strings) safely to the Lambda tool without exposing them to prompt injection or LLM token leakage. |
| **`promptSessionAttributes`** | Single Session (`sessionId`) | **Yes** (Injected into `$prompt_session_attributes$` placeholder) | **Yes** (`event['promptSessionAttributes']`) | Temporal/contextual prompt variables (e.g., `"current_date": "2026-11-16"`, `"user_timezone": "America/Chicago"`, `"customer_tier": "Enterprise"`). |
| **Agent Memory (`memoryId`)** | Cross-Session (`memoryStorageDays`, up to 365 days) | **Yes** (Retrieved via `$memory_search_results$` or pre-loaded into orchestration context) | No | Summarizing past sessions (`SESSION_SUMMARY`) so returning users (`memoryId="user-4829"`) maintain preferences, historical issues, and context across separate `sessionId` conversations. |

---

### 1.8 Built-In Action Groups: Code Interpreter & User Input

Bedrock provides two managed parent action group signatures (`parentActionGroupSignature`):
1. **`AMAZON.UserInput`**:
   - Must be explicitly enabled if you want the agent to ask clarifying questions when required tool parameters are missing from the user's prompt. If disabled, the agent may hallucinate missing parameter values or fail the tool call.
2. **`AMAZON.CodeInterpreter`**:
   - Spins up an isolated, ephemeral sandbox container capable of writing and executing Python code, performing mathematical calculations, parsing uploaded CSV/Excel/JSON files (`sessionState.files`), and generating Matplotlib charts returned as `files` byte streams in the `InvokeAgent` response.

---

### 1.9 Model Context Protocol (MCP) Integration on AWS & Strands Agents SDK

The **Model Context Protocol (MCP)** standardizes how AI applications and agents discover and invoke external tools, read data resources, and load reusable prompt templates.

```mermaid
flowchart LR
    subgraph AgentHost["Agent Host (ECS / Lambda / Strands SDK)"]
        Agent["Orchestrator / MCP Client"]
        Bedrock["Amazon Bedrock ConverseStream API"]
        Agent <-->|"toolConfig + toolResult"| Bedrock
    end

    subgraph MCPServers["MCP Server Fleet"]
        LocalMCP["Local Sidecar MCP Server (stdio Transport)"]
        RemoteMCP["Serverless MCP Server on Lambda / Fargate (Streamable HTTP / SSE)"]
    end

    Agent <-->|"JSON-RPC 2.0 over stdin/stdout"| LocalMCP
    Agent <-->|"JSON-RPC 2.0 over HTTPS (SigV4 / OAuth)"| RemoteMCP
    RemoteMCP --> RDS[("Amazon Aurora PostgreSQL")]
    RemoteMCP --> S3[("Amazon S3")]
```

#### MCP Primitives & Transports on AWS
- **Three Core MCP Primitives**:
  1. **Tools** (`tools/list`, `tools/call`): Model-controlled executable functions (mapped directly to Bedrock `toolConfig.tools[].toolSpec`).
  2. **Resources** (`resources/list`, `resources/read`): Application-controlled contextual data (URI-addressed files, database schemas, system logs).
  3. **Prompts** (`prompts/list`, `prompts/get`): User-controlled templated workflows and slash-commands.
- **Transport Mechanisms**:
  - **`stdio` (Standard Input/Output)**: Used when the MCP Server runs as a local child process or sidecar container inside the same AWS Fargate task or EC2 instance. Zero network hop overhead.
  - **Streamable HTTP / Server-Sent Events (SSE)**: Used when hosting shared enterprise MCP servers centrally behind Amazon API Gateway, Application Load Balancer (ALB), AWS Lambda Function URLs (with `RESPONSE_STREAM`), or Amazon ECS Service Connect. Secured via AWS IAM SigV4 request signing or Amazon Cognito OAuth 2.0 bearer tokens.

#### Bridging MCP Tools to Amazon Bedrock `Converse` API & Strands Agents SDK
When an MCP client connects to an MCP Server and calls `session.list_tools()`, each MCP tool exposes `name`, `description`, and `inputSchema` (JSON Schema). With the **AWS Strands Agents SDK (`strands-agents`)**, connecting a Bedrock model to both local `stdio` and remote Streamable HTTP MCP servers takes only a few lines of production code:

```python
from strands import Agent
from strands.models import BedrockModel
from strands.tools.mcp import MCPClient
from mcp import stdio_client, StdioServerParameters
from mcp.client.streamable_http import streamablehttp_client

# 1. Configure Amazon Bedrock Model with Cross-Region Inference Profile
bedrock_model = BedrockModel(
    model_id="us.anthropic.claude-3-5-sonnet-20241022-v2:0",
    region_name="us-east-1",
    temperature=0.0,
    max_tokens=2048
)

# 2. Connect to an AWS Cloud Control / Documentation MCP Server via stdio
aws_mcp_client = MCPClient(
    lambda: stdio_client(
        StdioServerParameters(
            command="uvx",
            args=["awslabs.aws-documentation-mcp-server@latest"]
        )
    )
)

# 3. Connect to an Enterprise Internal MCP Server on AWS Fargate via Streamable HTTP
enterprise_mcp_client = MCPClient(
    lambda: streamablehttp_client(
        url="https://mcp.internal.corp.example.com/mcp",
        headers={"Authorization": "Bearer eyJhbGciOiJSUzI1NiIs..."}
    )
)

# 4. Discover tools from both MCP servers and run the Strands Agent loop
with aws_mcp_client, enterprise_mcp_client:
    combined_tools = aws_mcp_client.list_tools_sync() + enterprise_mcp_client.list_tools_sync()
    agent = Agent(
        model=bedrock_model,
        tools=combined_tools,
        system_prompt="You are a Principal Cloud Architect Agent. Verify all AWS limits and internal VPC CIDRs using MCP tools."
    )
    response = agent("Verify our prod-us-east-1 VPC CIDR allocation and check PrivateLink quotas for Bedrock.")
```

---

## Task 2.2: Implement Model Deployment Strategies

### 2.1 Amazon Bedrock Consumption & Deployment Modes

Choosing the optimal deployment and pricing model on Amazon Bedrock is a foundational architectural decision tested heavily on the AIP-C01 exam:

| Deployment Mode | Billing Model | Throughput / Quota Scope | Supported Models & Custom Weights | Best Fit Workload |
| :--- | :--- | :--- | :--- | :--- |
| **Standard On-Demand** | Pay-per-input/output token | Single AWS Region account quotas (RPM / TPM) | Base Foundation Models only | Variable traffic workloads requiring strict single-region data residency (e.g., `eu-central-1` only). |
| **Cross-Region Inference (CRIS)** | Pay-per-token (same price as source region On-Demand; no routing surcharge) | Dynamically pools capacity across 2–4 regions in a geography (`us.*`, `eu.*`, `apac.*`, `global.*`) — up to 2x+ burst quotas | Base Foundation Models (Claude, Nova, Llama, Mistral) | Production spiky/bursty interactive workloads that can tolerate inference compute running across approved regional boundaries within a geography. |
| **Batch Inference** | **50% discount** vs. On-Demand token pricing | Asynchronous job queues reading/writing JSONL files in Amazon S3 (no impact on real-time On-Demand quotas!) | Supported Base FMs (Claude, Nova, Llama, Titan) | Offline bulk document extraction, nightly summarization, classification of millions of records, and dataset evaluation. |
| **Provisioned Throughput (PT)** | Hourly commitment per **Model Unit (MU)** (No Commit, 1-Month, or 6-Month commitment) | Dedicated guaranteed throughput (input/output tokens per minute per MU) | Base FMs **AND** Fine-Tuned / Continued Pre-Trained Custom Bedrock Models | Mission-critical low-latency SLA workloads with sustained high utilization (>60–70% 24x7) OR serving custom fine-tuned Bedrock models. |
| **Custom Model Import (CMI)** | Billed per **Custom Model Unit (CMU)** active in 5-minute windows (scales to zero after inactivity!) | Dedicated auto-scaling serverless capacity measured in CMUs (no upfront time commitment required) | Customer-owned weights (Safetensors) for supported architectures (Llama, Mistral, Qwen, Flan) | Serving custom fine-tuned open-weight models trained in SageMaker/on-prem via unified Bedrock APIs (`InvokeModel`, `Converse`, Agents, KBs, Guardrails) without managing GPUs. |

> [!WARNING]
> **Critical Exam Distinction: Fine-Tuned Bedrock Models vs. Custom Model Import (CMI)**
> - If you fine-tune a base model **natively inside Amazon Bedrock** (e.g., fine-tuning Amazon Nova or Titan via a Bedrock Model Customization job), you **must** purchase **Provisioned Throughput** to host and invoke it.
> - If you import custom weights from S3 using **Bedrock Custom Model Import (CMI)** (e.g., a Llama 3.1 or Mistral model fine-tuned in SageMaker JumpStart or on-premises in HuggingFace `.safetensors` format), you invoke it **on-demand without Provisioned Throughput commitments**! You are billed only for **Custom Model Units (CMUs)** while the model is active (with a minimum 5-minute billing window after each invocation, scaling down to zero when idle).

---

### 2.2 Amazon Bedrock Custom Model Import (CMI)

When an organization trains custom open-weight models (such as Meta Llama 3, Mistral, Mixtral, or Qwen) in Amazon SageMaker AI or on-premises GPU clusters, managing dedicated SageMaker Real-Time Endpoints 24x7 can be cost-prohibitive if traffic is intermittent. **Amazon Bedrock Custom Model Import** bridges this gap:

1. **Artifact Packaging**: Store the model weights in **HuggingFace `.safetensors`** format along with `config.json`, `tokenizer.json`, and `tokenizer_config.json` in an Amazon S3 bucket.
2. **Import Job (`create_model_import_job`)**: Bedrock validates the architecture, determines the required **Custom Model Units (CMUs)** based on parameter count and context length, and registers an `imported-model` ARN.
3. **Cold Start vs. Warm Caching**: If an imported model receives no traffic for roughly 5 minutes, Bedrock scales the underlying compute down to zero (0 CMUs billed). The first request after an idle period incurs a cold-start loading delay (typically 30–90 seconds) and returns `ModelNotReadyException` while warming up. Clients should configure retry logic for `ModelNotReadyException` or send periodic synthetic keep-alive pings during business hours.

---

### 2.3 Amazon SageMaker AI Deployment Strategies

When workloads require custom model architectures unsupported by Bedrock CMI, custom pre/post-processing C++ extensions, sub-millisecond VPC colocation, or fine-grained GPU tensor parallelism control, **Amazon SageMaker AI** provides four specialized inference modes:

```mermaid
flowchart TD
    Start["Select SageMaker AI Deployment Strategy"] --> Q1{"Payload Size > 6 MB or Inference Time > 60s?"}
    Q1 -->|"Yes (Up to 1 GB payload, 1 hr timeout)"| Async["Asynchronous Inference (S3 In/Out + SNS + Auto-Scale to Zero)"]
    Q1 -->|"No (<= 6 MB payload, <= 60s)"| Q2{"Traffic Pattern & Model Size?"}
    Q2 -->|"Intermittent / Spiky, Small Model (<10 GB CPU/RAM, No GPU)"| Serverless["Serverless Inference (Scales to Zero, Cold Starts)"]
    Q2 -->|"Large Bulk Offline Dataset in S3 (Hours/Days)"| Batch["Batch Transform (Ephemeral Cluster per Job)"]
    Q2 -->|"Low-Latency Sustained or GPU-Accelerated LLMs"| Q3{"Hosting Multiple Models / LoRA Adapters?"}
    Q3 -->|"Single Large LLM"| RealTime["Real-Time Endpoint with LMI Container (vLLM / TensorRT-LLM)"]
    Q3 -->|"Dozens of Base Models or Hundreds of LoRA Adapters"| IC["Inference Components / Multi-Adapter LoRA on Shared GPU Fleet"]
```

#### SageMaker Inference Mode Limits & Quotas Comparison

| Dimension | Real-Time Inference | Asynchronous Inference | Serverless Inference | Batch Transform |
| :--- | :--- | :--- | :--- | :--- |
| **Max Payload Size** | **6 MB** (25 MB for streaming responses) | **1 GB** (via Amazon S3 URI pointers) | **4 MB** | **100 MB** per mini-batch record (S3 dataset) |
| **Max Processing Timeout** | **60 seconds** (up to 600s for streaming) | **1 hour (3,600 seconds)** | **60 seconds** | **Days** (Job-level timeout) |
| **GPU Instance Support** | **Yes** (`ml.g5`, `ml.g6e`, `ml.p4d`, `ml.p5`, `ml.inf2`, `ml.trn1`) | **Yes** (Full GPU & Inferentia2 support) | **No** (CPU-only: 1–6 GB RAM) | **Yes** (Full GPU & Inferentia2 support) |
| **Scale to Zero (`MinCapacity=0`)** | **Yes**, when using **Inference Components** (Standard non-IC endpoints require $\ge 1$ instance) | **Yes** (Scales down to 0 instances when internal queue is empty) | **Yes** (Built-in scale-to-zero; optional Provisioned Concurrency) | **Yes** (Cluster terminates automatically upon job completion) |
| **Streaming Support** | **Yes** (`InvokeEndpointWithResponseStream`) | **No** (Outputs written to S3; completion event sent to SNS) | **No** | **No** |

---

### 2.4 SageMaker Large Model Inference (LMI) Containers

Deploying 7B to 405B+ parameter models on SageMaker Real-Time Endpoints uses AWS Deep Learning Containers for **Large Model Inference (LMI)** powered by **DJL Serving (Deep Java Library)** with **vLLM**, **TensorRT-LLM**, or **Transformers NeuronX** (for AWS Inferentia2 `ml.inf2` / Trainium `ml.trn1`).

Key LMI configuration parameters (`serving.properties` or environment variables):
- **`OPTION_ROLLING_BATCH` (`vllm` or `trtllm`)**: Enables **Continuous Batching (Iteration-Level Scheduling)** and **PagedAttention**. Instead of waiting for an entire static batch of sequences to finish generating before accepting new requests, the engine evicts completed sequences at every token decoding step and inserts waiting requests immediately—increasing GPU throughput by 3x–8x.
- **`OPTION_TENSOR_PARALLEL_DEGREE`**: Splits individual model layers across multiple GPUs within a single instance (e.g., setting `OPTION_TENSOR_PARALLEL_DEGREE=4` on an `ml.g5.12xlarge` with 4x NVIDIA A10G GPUs, or `8` on an `ml.p4d.24xlarge`).
- **`OPTION_QUANTIZATION` (`awq`, `gptq`, `fp8`, `smoothquant`)**: Reduces VRAM footprint to fit larger models onto fewer GPUs.
- **`OPTION_SPECULATIVE_DRAFT_MODEL`**: Configures speculative decoding using a smaller draft model to propose multiple tokens verified in a single forward pass of the target LLM.

---

### 2.5 Multi-Model Endpoints (MME) vs. Inference Components & Multi-Adapter LoRA

When serving multiple models or tenant-specific fine-tuned models on SageMaker, architects must choose between three distinct multi-tenancy patterns:

| Pattern | How It Works | Strengths & Limitations |
| :--- | :--- | :--- |
| **1. Multi-Model Endpoints (MME)** | Loads/unloads complete model artifacts dynamically from S3 into shared container memory based on `TargetModel` header. | Great for hundreds of small tabular/XGBoost/BERT models. **Poor fit for large LLMs (7B+)** because dynamically swapping 15 GB+ model weights from S3 into GPU VRAM causes severe cold-start latency spikes. |
| **2. SageMaker Inference Components (IC)** | Decouples the endpoint's underlying EC2/GPU instance fleet from logical models (`InferenceComponent`). Each IC specifies exact resource requirements (`CopyCount`, `NumberOfAcceleratorDevicesRequired`, CPU/RAM) and has its **own independent Application Auto Scaling policy** (including scale-to-zero!). | Ideal for bin-packing multiple distinct LLMs or embedding models (e.g., an 8B reranker + a 7B classifier + a 13B generator) onto a shared fleet of `ml.g5.48xlarge` or `ml.p4d.24xlarge` instances with guaranteed GPU allocation and zero noisy-neighbor VRAM eviction. |
| **3. Multi-Adapter LoRA on LMI / Inference Components** | Loads a single shared **Base Foundation Model** into GPU VRAM once, and registers dozens or hundreds of lightweight **LoRA (Low-Rank Adaptation)** adapter weights (typically 50–200 MB each) as separate Inference Components or dynamic adapters. | **The gold standard for multi-tenant SaaS LLM customization**. Serves 100+ tenant-specific fine-tuned LoRA adapters on a single shared GPU cluster with millisecond adapter switching overhead. |

---

### 2.6 Safe Model Rollouts: Blue/Green, Canary & Linear Deployments

When updating a SageMaker Real-Time Endpoint (`UpdateEndpoint`) with a new model version, quantization setting, or LMI container image, SageMaker supports managed **Deployment Guardrails**:

1. **Blue/Green Traffic Shifting Modes**:
   - **`AllAtOnce`**: Shifts 100% of traffic to the new fleet immediately after health checks pass, then waits for the bake period before terminating the blue fleet. Fastest cutover, highest blast radius.
   - **`Canary`**: Shifts a small fixed percentage (e.g., 10%) of traffic to the green fleet for a specified canary interval (e.g., 15 minutes). If CloudWatch alarms (`ModelLatency`, `Invocation5XXErrors`) remain in `OK` state, shifts the remaining 90%.
   - **`Linear`**: Shifts traffic in equal increments (e.g., 20% every 10 minutes over 50 minutes), continuously monitoring CloudWatch alarms at each step.
2. **Production Variants (`TargetVariant`) for Dark Launches / A/B Testing**:
   - You can deploy multiple `ProductionVariants` on a single SageMaker Endpoint with explicit `InitialVariantWeight` distributions (e.g., `Variant-Llama3-FP16` at `0.90` and `Variant-Llama3-AWQ` at `0.10`), or set shadow/test clients to explicitly pass the `TargetVariant` header in `invoke_endpoint` to test a candidate model without affecting production traffic.

---

## Task 2.3: Design & Implement Enterprise Integration Architectures

### 3.1 Event-Driven GenAI Architectures with EventBridge & SQS

Foundation Model invocations are inherently slower (1–30+ seconds) and more strictly rate-limited (Tokens Per Minute / Requests Per Minute) than traditional microservices. Synchronous request/response chains across multiple LLM steps frequently trigger upstream HTTP 504 Gateway Timeouts (e.g., API Gateway's default **29-second timeout** limit) or exhaust downstream token quotas during traffic spikes.

```mermaid
flowchart LR
    S3[("Amazon S3 (Document Landing)")] -->|"S3 Event Notification"| EB["Amazon EventBridge"]
    EB -->|"Rule Filter (PDF/DOCX)"| SQS["Amazon SQS Main Queue"]
    SQS <-->|"MaxReceiveCount=3"| DLQ["Amazon SQS DLQ"]
    SQS -->|"ESM (MaxConcurrency=15, ReportBatchItemFailures)"| Lambda["AWS Lambda Consumer"]
    Lambda -->|"Converse API"| Bedrock["Amazon Bedrock"]
    Lambda -->|"Persist Structured JSON"| DDB[("Amazon DynamoDB")]
```

#### Critical SQS + Lambda Tuning Parameters for GenAI Workloads
1. **SQS Visibility Timeout (`VisibilityTimeout`)**:
   - AWS best practice mandates setting the SQS queue's `VisibilityTimeout` to **at least 6x the Lambda function's timeout** plus the maximum batching window.
   - *Example*: If a Lambda function executing a multi-turn Bedrock Agent or long-context synthesis has a timeout of `180 seconds` (3 minutes), set the SQS `VisibilityTimeout` to **at least `1,080 seconds` (18 minutes)**. If `VisibilityTimeout` is left at the default `30 seconds`, SQS will make the message visible again while the first Lambda invocation is still waiting on Bedrock—causing duplicate LLM invocations, wasted token spend, and downstream race conditions!
2. **Lambda Event Source Mapping `MaximumConcurrency`**:
   - Without a concurrency cap, a sudden burst of 5,000 messages in SQS will cause the Lambda poller to rapidly scale up to hundreds of concurrent executions, immediately triggering `ThrottlingException (HTTP 429)` on Amazon Bedrock.
   - Configure `ScalingConfig={'MaximumConcurrency': N}` on the SQS Event Source Mapping (where $N$ aligns with your Bedrock RPM/TPM quota) so excess messages remain safely buffered in SQS without burning Lambda retries.
3. **Partial Batch Responses (`ReportBatchItemFailures`)**:
   - When processing a batch of SQS messages (`BatchSize > 1`), enable `FunctionResponseTypes=['ReportBatchItemFailures']` and return `{"batchItemFailures": [{"itemIdentifier": msg_id}]}` for only the throttled/failed messages so successful LLM outputs in the same batch are not re-processed.

---

### 3.2 AWS Step Functions Orchestration & Distributed Map

**AWS Step Functions** provides serverless workflow orchestration with native **Optimized Integrations** for Amazon Bedrock (`arn:aws:states:::bedrock:invokeModel`, `createModelCustomizationJob`) and **AWS SDK Integrations** (`arn:aws:states:::aws-sdk:bedrockruntime:converse`, `arn:aws:states:::aws-sdk:bedrockagentruntime:invokeAgent`).

#### Standard vs. Express Workflows for GenAI
- **Standard Workflows**: Support execution durations up to **1 year**, built-in `Retry` with jitter, **Callback Patterns (`.waitForTaskToken`)** for human-in-the-loop review, and **Distributed Map**. Billed per state transition.
- **Express Workflows**: Support execution durations up to **5 minutes**, higher event rates (100,000+ per second), billed by execution count and duration/memory. Do **not** support `.waitForTaskToken` or Distributed Map.

#### High-Scale Document Processing with Step Functions Distributed Map
When an enterprise needs to process 100,000 documents stored in Amazon S3 through an extraction/summarization pipeline:
- **Distributed Map State (`ProcessorConfig.Mode = "DISTRIBUTED"`)** iterates directly over an S3 bucket inventory or manifest file.
- Use **`MaxConcurrency`** (e.g., `MaxConcurrency: 25`) on the Distributed Map state to strictly throttle parallel child workflow executions to match your Bedrock Provisioned Throughput or On-Demand RPM quota.
- Wait for human approval on low-confidence extractions using `arn:aws:states:::sqs:sendMessage.waitForTaskToken`, which pauses the workflow at zero compute cost until a human reviewer calls `SendTaskSuccess` or `SendTaskFailure`.

---

### 3.3 Real-Time Token Streaming to Web & Mobile Clients

Delivering low **Time-To-First-Token (TTFT)** to end users requires end-to-end streaming from `ConverseStream` / `InvokeAgent` all the way to the browser or mobile client.

```mermaid
flowchart TD
    subgraph PatternA["Pattern A: AWS Lambda Response Streaming"]
        ClientA["Web / Mobile Client"] -->|"HTTPS GET/POST"| EdgeA["CloudFront + Lambda Function URL / API GW REST Streaming"]
        EdgeA -->|"InvokeWithResponseStream"| StreamLambda["Lambda (RESPONSE_STREAM, up to 15 min / 200 MB)"]
        StreamLambda -->|"ConverseStream"| BedrockA["Amazon Bedrock"]
    end

    subgraph PatternB["Pattern B: Asynchronous Pub/Sub via AWS AppSync"]
        ClientB["Web / Mobile Client"] <-->|"WebSocket (GraphQL Subscription / Event API)"| AppSync["AWS AppSync"]
        ClientB -->|"1. Mutation: startGeneration"| AppSync
        AppSync -->|"2. Async Event"| WorkerLambda["Async Lambda Worker"]
        WorkerLambda -->|"3. ConverseStream"| BedrockB["Amazon Bedrock"]
        WorkerLambda -->|"4. Publish Token Chunks"| AppSync
    end
```

#### Comparing Streaming Delivery Architectures on AWS

| Architecture Pattern | Max Timeout | Max Payload | Protocol & Auth | Architectural Trade-offs |
| :--- | :--- | :--- | :--- | :--- |
| **1. Lambda Function URL with `RESPONSE_STREAM`** (fronted by CloudFront with Origin Access Control) | **15 minutes** (900s) | **200 MB** (first 6 MB free, then billed per MB) | HTTPS chunked transfer / SSE; IAM SigV4 or custom CloudFront@Edge JWT verifier | Simplest architecture for long-running streaming generations (>30s). Requires Node.js (`awslambda.streamifyResponse`), Python (via FastAPI/Lambda Web Adapter), or custom runtime. |
| **2. Amazon API Gateway REST API Response Streaming** | **Up to 15 minutes** (with response streaming enabled on `AWS_PROXY` integration) | **10 MB** | HTTPS chunked transfer; Native Cognito Authorizers, Lambda Authorizers, Usage Plans, WAF | Provides full API Gateway governance (throttling, API keys, custom authorizers) while bypassing the legacy 29s non-streaming limit via streaming proxy mode (`ResponseTransferMode: STREAM`). *Note: API Gateway HTTP APIs (`v2`) do NOT support response streaming and have a hard 30s timeout.* |
| **3. AWS AppSync (GraphQL Subscriptions or AppSync Event APIs)** | **24 hours** WebSocket connection lifetime; async worker up to 15m (Lambda) or unlimited (ECS) | **240 KB** per individual published WebSocket event frame | Secure WebSockets (`wss://`); Cognito, OIDC, IAM, API Key, Lambda Authorizer | Best for multi-client collaborative chat rooms, resilient mobile connections (automatic reconnection), and bi-directional pub/sub where multiple UI components subscribe to agent trace updates and token streams simultaneously. |
| **4. Application Load Balancer (ALB) + Amazon ECS Fargate** | **Up to 4,000 seconds** (`idle_timeout.timeout_seconds`) | Unlimited HTTP chunked stream | HTTPS / SSE / WebSockets / gRPC; OIDC Auth at ALB | Best for high-concurrency stateful MCP servers or Python FastAPI/Strands agent hosts where persistent in-memory connections and zero cold starts are required. |

---

### 3.4 Private Multi-Account & Hybrid Connectivity via AWS PrivateLink

Regulated enterprises mandate that prompts, RAG context, and model outputs never traverse the public internet. To enforce strict network isolation across multi-account AWS Organizations and hybrid data centers:

1. **Required Interface VPC Endpoints (`AWS PrivateLink`)**:
   - `com.amazonaws.<region>.bedrock-runtime`: Data-plane inference (`InvokeModel`, `Converse`, `ConverseStream`, `ApplyGuardrail`).
   - `com.amazonaws.<region>.bedrock`: Control-plane API (`CreateModelCustomizationJob`, `ListFoundationModels`, `CreateGuardrail`).
   - `com.amazonaws.<region>.bedrock-agent-runtime`: Agent & KB data plane (`InvokeAgent`, `Retrieve`, `RetrieveAndGenerate`).
   - `com.amazonaws.<region>.bedrock-agent`: Agent & KB control plane (`CreateAgent`, `StartIngestionJob`).
2. **Centralized Shared Services VPC Pattern (Route 53 Private Hosted Zones + Transit Gateway)**:
   - Instead of provisioning duplicate Interface VPC Endpoints in 50+ spoke workload VPCs (incurring hourly ENI charges per AZ per VPC), create the Bedrock Interface VPC Endpoints **once** in a centralized **Network / Shared Services VPC** with **Private DNS disabled** on the endpoint, then create a **Route 53 Private Hosted Zone** (`bedrock-runtime.<region>.amazonaws.com`) with an Alias `A` record pointing to the central VPC Endpoint regional DNS name, and associate that Private Hosted Zone with all spoke VPCs connected via **AWS Transit Gateway** (and on-premises via AWS Direct Connect + Route 53 Inbound Resolver Endpoints).
3. **VPC Endpoint Policies**:
   - Attach a resource-based VPC Endpoint Policy to restrict which principals (`aws:PrincipalOrgID`), which specific models (`arn:aws:bedrock:*::foundation-model/anthropic.claude-3-5-sonnet*`), or which Guardrails can be invoked through the network boundary:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "RestrictBedrockRuntimeToOrgAndApprovedModels",
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream"
      ],
      "Resource": [
        "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-5-sonnet-20241022-v2:0",
        "arn:aws:bedrock:*:111122223333:inference-profile/us.anthropic.claude-3-5-sonnet-20241022-v2:0",
        "arn:aws:bedrock:us-east-1:111122223333:guardrail/*"
      ],
      "Condition": {
        "StringEquals": {
          "aws:PrincipalOrgID": "o-enterprise123"
        }
      }
    }
  ]
}
```

---

## Task 2.4: Implement FM API Integrations

### 4.1 Bedrock `Converse` & `ConverseStream` vs. `InvokeModel`

Amazon Bedrock offers two families of runtime inference APIs on `bedrock-runtime`:

| Feature | `Converse` / `ConverseStream` (Recommended) | `InvokeModel` / `InvokeModelWithResponseStream` |
| :--- | :--- | :--- |
| **Request/Response Schema** | **Unified, model-agnostic schema** (`messages`, `system`, `inferenceConfig`, `toolConfig`, `guardrailConfig`). Identical code works across Anthropic Claude, Amazon Nova, Meta Llama, Mistral, and Cohere Command. | **Model-specific native JSON payload** (`anthropic_version` vs `messages` vs `inputText`). Switching models requires rewriting request construction and response parsing logic. |
| **Native Tool Use** | Standardized `toolConfig` and `stopReason == "tool_use"` across all tool-capable models. | Model-specific tool schemas and stop sequences. |
| **Prompt Caching & Guardrails** | Native `cachePoint: {"type": "default"}` blocks inside `system`, `messages`, and `toolConfig`; built-in `guardrailConfig` with selective `guarded_content` tags. | Requires model-specific cache headers or HTTP header configurations for Guardrails (`X-Amzn-Bedrock-GuardrailIdentifier`). |
| **Model-Specific Parameters** | Passed cleanly via `additionalModelRequestFields` (e.g., Claude 3.7 extended thinking `{"thinking": {"type": "enabled", "budget_tokens": 2000}}`) and read via `additionalModelResponseFieldPaths`. | Passed directly in the raw JSON body. |
| **When to Use `InvokeModel` Instead** | Use `Converse` for **all text, multimodal document/image, and tool-use conversational workloads**. | **Must use `InvokeModel`** for **Embedding models** (Titan Text Embeddings V2, Cohere Embed) and **Image/Video Generation models** (Amazon Titan Image Generator, Nova Canvas, Nova Reel, Stability AI), which do not follow a conversational turn schema. |

---

### 4.2 Native Tool Use (Function Calling) with Converse API

When orchestrating tool use directly in application code via `Converse` or `ConverseStream` (without a managed Bedrock Agent), you control the **`toolChoice`** behavior and the **`toolResult`** loop:

#### Controlling Model Tool Selection (`toolChoice`)
Inside `toolConfig`, the `toolChoice` field dictates how the model selects tools:
- **`{"auto": {}}` (Default)**: The model decides autonomously whether to answer directly in natural language OR call one or more tools.
- **`{"any": {}}`**: Forces the model to call **at least one** of the provided tools in `toolConfig.tools`. The model cannot respond with plain text alone.
- **`{"tool": {"name": "extract_invoice_schema"}}`**: Forces the model to invoke that **exact named tool**.
  > [!TIP]
  > **Structured Output Guarantee Pattern**: On the exam, if a scenario requires 100% deterministic JSON schema adherence from a `Converse` call without markdown wrapper hallucinations, define the target JSON Schema as a tool's `inputSchema` and set `toolChoice={"tool": {"name": "your_schema_tool"}}`!

#### Complete `Converse` Tool Use Loop in Python (`boto3`)

```python
import boto3
import json

bedrock = boto3.client("bedrock-runtime", region_name="us-east-1")
MODEL_ID = "us.anthropic.claude-3-5-sonnet-20241022-v2:0"

tools = [
    {
        "toolSpec": {
            "name": "get_fx_rate",
            "description": "Retrieves live institutional foreign exchange rate between two ISO 4217 currencies.",
            "inputSchema": {
                "json": {
                    "type": "object",
                    "properties": {
                        "base_currency": {"type": "string", "description": "3-letter ISO currency code, e.g., USD"},
                        "quote_currency": {"type": "string", "description": "3-letter ISO currency code, e.g., EUR"}
                    },
                    "required": ["base_currency", "quote_currency"]
                }
            }
        }
    }
]

def execute_local_tool(name: str, tool_input: dict) -> dict:
    if name == "get_fx_rate":
        return {"rate": 0.9245, "timestamp": "2026-11-16T21:00:00Z", **tool_input}
    raise ValueError(f"Unknown tool: {name}")

def run_conversation(user_prompt: str) -> str:
    messages = [{"role": "user", "content": [{"text": user_prompt}]}]

    while True:
        response = bedrock.converse(
            modelId=MODEL_ID,
            messages=messages,
            toolConfig={"tools": tools, "toolChoice": {"auto": {}}},
            inferenceConfig={"maxTokens": 1024, "temperature": 0.0}
        )

        output_message = response["output"]["message"]
        messages.append(output_message)
        stop_reason = response["stopReason"]

        if stop_reason == "tool_use":
            tool_results_content = []
            # A single assistant message may request MULTIPLE parallel tool calls!
            for content_block in output_message["content"]:
                if "toolUse" in content_block:
                    tool_use = content_block["toolUse"]
                    tool_id = tool_use["toolUseId"]
                    tool_name = tool_use["name"]
                    tool_args = tool_use["input"]

                    try:
                        result_data = execute_local_tool(tool_name, tool_args)
                        tool_results_content.append({
                            "toolResult": {
                                "toolUseId": tool_id,
                                "content": [{"json": result_data}],
                                "status": "success"
                            }
                        })
                    except Exception as exc:
                        tool_results_content.append({
                            "toolResult": {
                                "toolUseId": tool_id,
                                "content": [{"text": f"Tool execution error: {str(exc)}"}],
                                "status": "error"
                            }
                        })

            # Append ALL toolResult blocks in a SINGLE user message turn
            messages.append({"role": "user", "content": tool_results_content})
        else:
            # Final natural language response (stop_reason == "end_turn")
            return output_message["content"][0]["text"]
```

> [!IMPORTANT]
> **Parallel Tool Use Rule**: When a model emits multiple `toolUse` blocks in a single `assistant` message turn, your application **must** return the corresponding `toolResult` blocks (matching every `toolUseId`) grouped inside a **single subsequent `user` message**! Sending separate `user` messages for each parallel `toolUseId` causes a `ValidationException` because every `toolUse` in the preceding assistant turn must be resolved immediately.

---

### 4.3 Resiliency Engineering: Rate Limiting, Adaptive Retries & Circuit Breakers

By default, the `boto3` SDK uses `legacy` retry mode with a maximum of 5 attempts and does **not** implement client-side token-bucket rate limiting. For production GenAI workloads:

1. **Configure `botocore.config.Config` with `adaptive` Retry Mode**:
   - **`standard` mode**: Implements exponential backoff with full jitter on transient errors (`ThrottlingException`, `ServiceUnavailableException`, `ModelTimeoutException`).
   - **`adaptive` mode**: Builds on `standard` mode by adding a **client-side token-bucket rate limiter** that dynamically throttles outbound request rates locally as soon as `ThrottlingException (HTTP 429)` responses are observed, preventing retry storms.
2. **Read Throughput vs. Connect Timeouts**:
   - Large context inputs (100k+ tokens) or reasoning models can take >60 seconds (the `botocore` default `read_timeout`). Always increase `read_timeout` (e.g., `read_timeout=300`) when invoking large models or agents.

```python
from botocore.config import Config
import boto3

resilient_config = Config(
    region_name="us-east-1",
    connect_timeout=5,
    read_timeout=300,
    retries={
        "max_attempts": 8,
        "mode": "adaptive"
    }
)
bedrock_client = boto3.client("bedrock-runtime", config=resilient_config)
```

---

### 4.4 Multi-Region Failover Chains & Intelligent Prompt Routing

To achieve 99.99% availability and optimal price-performance across Foundation Models:

1. **Cross-Region Inference Profiles (System-Defined vs. Application Inference Profiles)**:
   - **System-Defined Inference Profiles** (e.g., `us.anthropic.claude-3-5-sonnet-20241022-v2:0`) automatically route requests across `us-east-1`, `us-east-2`, and `us-west-2` based on real-time capacity and throttling state.
   - **Application Inference Profiles (`create_inference_profile`)**: Wrap either a single-region foundation model or a Cross-Region system profile with custom **Cost Allocation Tags** so FinOps teams can track token spend per business unit, tenant, or microservice while preserving Cross-Region burst resiliency!
2. **Amazon Bedrock Intelligent Prompt Routing**:
   - Evaluates the complexity of each incoming prompt in real time using a lightweight router and routes simple prompts to a faster, cheaper model in the same family (e.g., Claude 3.5 Haiku or Amazon Nova Micro/Lite) while routing complex reasoning prompts to the flagship model (e.g., Claude 3.5 Sonnet or Amazon Nova Pro).
   - Reduces cost by up to 30–50% and improves latency without sacrificing quality on complex queries.

---

## Task 2.5: Application Integration Patterns & Development Tools

### 5.1 Infrastructure as Code (IaC) for GenAI Stacks

Enterprise GenAI stacks require reproducible deployments across `dev`, `staging`, and `prod` accounts using **AWS CloudFormation**, **AWS CDK**, or **HashiCorp Terraform**.

#### Complete Terraform Configuration for a Bedrock Agent + Action Group + Alias

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.80"
    }
  }
}

data "aws_caller_identity" "current" {}
data "aws_region" "current" {}

resource "aws_iam_role" "bedrock_agent_role" {
  name = "prod-financial-agent-execution-role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect    = "Allow"
      Principal = { Service = "bedrock.amazonaws.com" }
      Action    = "sts:AssumeRole"
      Condition = {
        StringEquals = { "aws:SourceAccount" = data.aws_caller_identity.current.account_id }
        ArnLike      = { "aws:SourceArn" = "arn:aws:bedrock:${data.aws_region.current.name}:${data.aws_caller_identity.current.account_id}:agent/*" }
      }
    }]
  })
}

resource "aws_bedrockagent_agent" "financial_agent" {
  agent_name                  = "prod-financial-analyst-agent"
  agent_resource_role_arn     = aws_iam_role.bedrock_agent_role.arn
  foundation_model            = "us.anthropic.claude-3-5-sonnet-20241022-v2:0"
  idle_session_ttl_in_seconds = 900
  prepare_agent               = true
  instruction                 = "You are a senior institutional financial analyst agent. Always verify account balances using the ledger action group before answering."

  memory_configuration {
    enabled_memory_types = ["SESSION_SUMMARY"]
    storage_days         = 30
  }
}

resource "aws_bedrockagent_agent_action_group" "ledger_tools" {
  action_group_name          = "ledger-query-tools"
  agent_id                   = aws_bedrockagent_agent.financial_agent.id
  agent_version              = "DRAFT"
  prepare_agent              = true
  action_group_state         = "ENABLED"

  action_group_executor {
    lambda = aws_lambda_function.ledger_executor.arn
  }

  function_schema {
    member_functions {
      functions {
        name        = "get_ledger_balance"
        description = "Retrieves the verified general ledger balance for a specific cost center."
        parameters {
          map_block_key = "cost_center_id"
          type          = "string"
          description   = "The 6-character alphanumeric cost center code"
          required      = true
        }
      }
    }
  }
}

resource "aws_bedrockagent_agent_alias" "prod_alias" {
  agent_alias_name = "production-live"
  agent_id         = aws_bedrockagent_agent.financial_agent.id
  description      = "Production alias pinned to immutable agent version"
}
```

---

### 5.2 Zero-Downtime Agent Versioning, Aliases & CI/CD Pipelines

```mermaid
flowchart LR
    CodeCommit["Git Repo (Prompt YAML + OpenAPI + CDK)"] -->|"PR Merge"| Pipeline["AWS CodePipeline"]
    Pipeline -->|"1. Update DRAFT + PrepareAgent"| Draft["Agent Working DRAFT"]
    Pipeline -->|"2. CreateAgentAlias / Snapshot"| V2["Immutable Agent Version 2"]
    Pipeline -->|"3. Run Bedrock Evaluation / Ragas Gate"| EvalGate{"Quality & Latency Pass?"}
    EvalGate -->|"Pass"| Alias["UpdateAgentAlias ('prod-alias' -> Version 2)"]
    EvalGate -->|"Fail"| Rollback["Keep 'prod-alias' -> Version 1"]
    Client["Production Traffic"] -->|"InvokeAgent (agentAliasId)"| Alias
```

#### How Bedrock Agent Versioning Works
1. **`DRAFT` Working Version**: All mutations (`update_agent`, `update_agent_action_group`, `associate_agent_knowledge_base`) can **only** be applied to the `DRAFT` version of an agent, followed by calling `prepare_agent`.
2. **Immutable Numbered Versions (`1`, `2`, `3`...)**: You cannot directly call a standalone `CreateAgentVersion` API on its own. Instead, an immutable numbered version is automatically created whenever you call **`create_agent_alias`** (without specifying an existing `routingConfiguration` version) or **`update_agent_alias`** to snapshot the current prepared `DRAFT`.
3. **Zero-Downtime Cutover & Instant Rollback**: Client applications always hardcode the stable `agentId` and `agentAliasId` (never `TSTALIASID` in production!). Updating the alias's `routingConfiguration` to point from `agentVersion='1'` to `agentVersion='2'` cuts traffic over atomically with zero downtime; reverting to `agentVersion='1'` provides instant rollback.

---

### 5.3 Amazon Q Developer Enterprise Customizations & Workspace Indexing

**Amazon Q Developer** accelerates enterprise software engineering and GenAI integration through two distinct context mechanisms:

| Capability | Scope & Execution Location | Requirements & Governance |
| :--- | :--- | :--- |
| **1. Local Workspace Indexing (`@workspace`)** | Indexes files locally within the developer's IDE (VS Code / JetBrains) using a local vector index on the developer's workstation. | Immediate zero-setup context for active repository files. Does not index external shared libraries or uncloned microservices across the enterprise. |
| **2. Amazon Q Developer Customizations (Enterprise Private Codebase Fine-Tuning / RAG)** | Managed centrally by AWS administrators in the Amazon Q console by connecting to enterprise GitHub, GitLab, Bitbucket, or Amazon S3 repositories. | Requires **Amazon Q Developer Pro** tier, a codebase containing at least **10–20 MB of source files** (recommended hundreds of files of idiomatic internal SDKs/APIs), and IAM Identity Center group assignments. Generates an evaluation score (`Customization Evaluation`) measuring improvement over the base Q model before publishing to authorized developers. Customer code is isolated via KMS encryption and **never** used to train shared global AWS models. |

---

## Summary Exam Checklist for Domain 2

- **Bedrock Agents Lambda Contract**: Must include a resource-based policy on the Lambda function for `bedrock.amazonaws.com` (`aws:SourceArn` + `aws:SourceAccount`) AND return the exact `messageVersion: "1.0"` envelope (`response.actionGroup`, `responseBody`).
- **`sessionAttributes` vs. `promptSessionAttributes`**: Use `sessionAttributes` for secrets/tenant IDs that the Lambda needs but the LLM must never see; use `promptSessionAttributes` for temporal variables injected into the prompt.
- **Custom Model Import (CMI) vs. Fine-Tuned Bedrock Models**: CMI uses **Custom Model Units (CMUs)** with scale-to-zero after 5 minutes of inactivity (no Provisioned Throughput commitment!); natively fine-tuned Bedrock models **require Provisioned Throughput**.
- **SageMaker Inference Components + Multi-Adapter LoRA**: Use Inference Components with LMI containers (`vLLM` / `TensorRT-LLM`) to host hundreds of LoRA adapters or bin-pack multiple LLMs on shared GPUs with independent auto-scaling to zero.
- **SQS + Lambda for GenAI**: Set `VisibilityTimeout >= 6x Lambda Timeout` and configure `ScalingConfig.MaximumConcurrency` on the SQS Event Source Mapping to prevent Bedrock `ThrottlingException` storms.
- **Parallel Tool Calls in `Converse`**: When `stopReason == "tool_use"` returns multiple `toolUse` blocks in a single assistant turn, return **all** matching `toolResult` blocks inside a **single** subsequent `user` message.
