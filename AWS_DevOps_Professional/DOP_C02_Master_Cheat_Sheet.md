# AWS Certified DevOps Engineer - Professional (DOP-C02) — Master Cheat Sheet & Exam Battle Guide

**Exam Code:** `DOP-C02`  
**Duration:** 180 Minutes (75 Questions — Multiple Choice & Multiple Response)  
**Passing Score:** 750 / 1000  

> [!IMPORTANT]
> **How to Read DOP-C02 Scenario Questions:**
> Every DOP-C02 question tests an architectural constraint: **"LEAST operational overhead"**, **"MOST secure"**, **"MOST cost-effective"**, **"Zero RPO / Lowest RTO"**, or **"Preventative vs. Detective vs. Responsive governance"**. Immediately eliminate options that rely on custom cron scripts on EC2, polling Lambda functions when native EventBridge events exist, or static IAM user credentials (`AWS_ACCESS_KEY_ID`) when IAM Roles / OIDC / `git-remote-codecommit` are available.

---

## 1. Domain Weight Breakdown & Core Service Map

- **Domain 1: SDLC Automation**
  - **Weight:** **22%**
  - **Core AWS Services:** CodeCommit, CodeBuild, CodeDeploy, CodePipeline (V1/V2), CodeArtifact, EC2 Image Builder, AWS SAM
  - **High-Frequency Exam Scenarios:** Cross-account pipelines with CMK + S3 Bucket Policy; CodeBuild in private VPC via NAT Gateway; CodeDeploy `appspec.yml` hook order & rollbacks; ECS Blue/Green Canary with Test Listener; Monorepo path filtering.
- **Domain 2: Configuration Management & IaC**
  - **Weight:** **17%**
  - **Core AWS Services:** CloudFormation, StackSets, `cfn-init`/`cfn-signal`/`cfn-hup`, Custom Resources, `cfn-guard`, CDK, SSM (Parameter Store, Patch Manager, State Manager, Session Manager, AppConfig), Elastic Beanstalk
  - **High-Frequency Exam Scenarios:** ASG `UpdatePolicy` (`AutoScalingRollingUpdate`) + `CreationPolicy`; StackSets `SERVICE_MANAGED` vs `SELF_MANAGED`; Custom Resource `ResponseURL` hang; SSM Patch Baselines + Patch Groups + Maintenance Windows; Beanstalk Immutable vs Rolling with Additional Batch.
- **Domain 3: Resilient Cloud Solutions**
  - **Weight:** **15%**
  - **Core AWS Services:** EC2 Auto Scaling (Lifecycle Hooks, Warm Pools, Mixed Instances), ALB/NLB/GWLB, Route 53 & ARC, RDS Multi-AZ Instance vs Cluster, RDS Proxy, Aurora Global DB, DynamoDB Global Tables, S3 CRR/RTC/MRAP, AWS DRS
  - **High-Frequency Exam Scenarios:** ASG `Terminating:Wait` hook -> EventBridge -> SSM/Lambda -> `CompleteLifecycleAction`; Route 53 ARC Routing Control (data plane 5-cluster quorum) vs Readiness Check; Aurora Global Managed Failover (`RPO=0`) vs Unplanned Failover (`RPO<1s`).
- **Domain 4: Monitoring & Logging**
  - **Weight:** **15%**
  - **Core AWS Services:** CloudWatch (Metrics, Unified Agent, EMF, Alarms, Composite Alarms, Metric Streams, Logs, Metric/Subscription Filters, Logs Insights, Data Protection, Contributor Insights, Synthetics), CloudTrail, X-Ray, ADOT, VPC Flow Logs
  - **High-Frequency Exam Scenarios:** CloudWatch Unified Agent `procstat` + `append_dimensions` + `aggregation_dimensions`; Cross-account Logs Destination (`put-destination`) to Kinesis; X-Ray indexed **Annotations** vs unindexed **Metadata**; VPC Flow Logs v5 `pkt-srcaddr`/`pkt-dstaddr` across TGW/NAT.
- **Domain 5: Incident & Event Response**
  - **Weight:** **14%**
  - **Core AWS Services:** EventBridge (Buses, Rules, Input Transformer, Archive/Replay, Global Endpoints, Pipes), SSM Incident Manager, AWS Chatbot, SSM Automation, AWS Config Remediations, Step Functions Forensics, AWS Health, AWS FIS
  - **High-Frequency Exam Scenarios:** GuardDuty finding -> EventBridge -> Step Functions EC2 isolation (`DetachInstances` without decrementing capacity + Quarantine SG + EBS Snapshot); Config + SSM Automation; Spot 2-min interruption + Rebalance Recommendation; FIS with CloudWatch Alarm Stop Condition.
- **Domain 6: Security & Compliance**
  - **Weight:** **17%**
  - **Core AWS Services:** IAM (Permission Boundaries, Evaluation Logic, Roles Anywhere), Organizations & SCPs, KMS (Key Policies, Grants, Multi-Region Keys), Secrets Manager, AWS Config & Conformance Packs, Control Tower, Service Catalog, GuardDuty, Inspector v2, Macie, Security Hub, WAF, Firewall Manager
  - **High-Frequency Exam Scenarios:** SCP vs Permission Boundary vs Identity/Resource Policy evaluation; Cross-account encrypted AMI launch with Auto Scaling Service-Linked Role KMS Grant; Secrets Manager Alternating Users rotation; Service Catalog Launch Constraints.

---

## 2. Domain 1: SDLC Automation — Deep Reference Matrices

### 2.1 CodeDeploy Lifecycle Hook Execution Order (Memorize Cold!)

```text
========================================================================================
           EC2 / On-Premises CodeDeploy Lifecycle Hooks (In-Place & Blue/Green)
========================================================================================

  [ BeforeBlockTraffic ] ──► [ BlockTraffic (Agent) ] ──► [ AfterBlockTraffic ]
                                                                 │
  ┌──────────────────────────────────────────────────────────────┘
  ▼
  [ ApplicationStop (Runs scripts from PREVIOUS revision!) ]
        │
        ▼
  [ DownloadBundle (Agent downloads from S3/GitHub) ]
        │
        ▼
  [ BeforeInstall ] ──► [ Install (Agent copies files) ] ──► [ AfterInstall ]
                                                                    │
  ┌─────────────────────────────────────────────────────────────────┘
  ▼
  [ ApplicationStart ] ──► [ ValidateService ]
                                 │
  ┌──────────────────────────────┘
  ▼
  [ BeforeAllowTraffic ] ──► [ AllowTraffic (Agent registers with ALB) ] ──► [ AfterAllowTraffic ]
```

- **EC2 / On-Premises**
  - **File Name:** `appspec.yml`
  - **Exact Hook Sequence:** `BeforeBlockTraffic` > `BlockTraffic`* > `AfterBlockTraffic` > `ApplicationStop` > `DownloadBundle`* > `BeforeInstall` > `Install`* > `AfterInstall` > `ApplicationStart` > `ValidateService` > `BeforeAllowTraffic` > `AllowTraffic`* > `AfterAllowTraffic`
  - **Critical Exam Nuances:** Hooks marked with `*` (`BlockTraffic`, `DownloadBundle`, `Install`, `AllowTraffic`) are **reserved for the CodeDeploy Agent** — you CANNOT attach custom scripts to them! `ApplicationStop` runs the script from the **previous** revision's `appspec.yml`. If `ApplicationStop` fails because the old script had a bug, deploy with `--ignore-application-stop-failures`.
- **Amazon ECS (Blue/Green)**
  - **File Name:** `appspec.yaml`
  - **Exact Hook Sequence:** `BeforeInstall` > `Install`* > `AfterInstall` > `AllowTestTraffic`* > `AfterAllowTestTraffic` > `BeforeAllowTraffic` > `AllowTraffic`* > `AfterAllowTraffic`
  - **Critical Exam Nuances:** Only supports **Blue/Green** (unless ECS native rolling update or External controller is used). Validation tests against the replacement task set must run in **`AfterAllowTestTraffic`** (which hits the ALB Test Listener port) and call `codedeploy:PutLifecycleEventHookExecutionStatus`.
- **AWS Lambda**
  - **File Name:** `appspec.yaml`
  - **Exact Hook Sequence:** `BeforeAllowTraffic` > `AllowTraffic`* > `AfterAllowTraffic`
  - **Critical Exam Nuances:** Used by AWS SAM (`AutoPublishAlias` + `DeploymentPreference`). Pre-traffic hook runs in `BeforeAllowTraffic`; post-traffic hook runs in `AfterAllowTraffic`.

### 2.2 Cross-Account CodePipeline Checklist (Tools Account `A` > Prod Account `B`)

Whenever a CodePipeline in **Account A** deploys to **Account B** and fails with `Access Denied` or `The ciphertext refers to a customer master key that does not exist, does not allow access, or is invalid`, verify all **4 pillars**:

1. **Customer Managed KMS Key (CMK) in Account A**:
   - You **MUST** use a Customer Managed Key (`aws/s3` AWS-managed key CANNOT have its key policy modified for cross-account access).
   - **KMS Key Policy in Account A** must grant `kms:Decrypt`, `kms:DescribeKey`, `kms:GenerateDataKey*`, `kms:ReEncrypt*` to:
     - The CodePipeline Service Role in Account A.
     - The Cross-Account Role in Account B (`arn:aws:iam::ACCOUNT_B:role/CrossAccountPipelineRole`).
     - The Execution Role in Account B (e.g., CloudFormation Role, CodeDeploy Role, or EC2 Instance Profile in Account B that downloads the artifact from S3).
2. **S3 Artifact Bucket Policy in Account A**:
   - Must grant `s3:GetObject*`, `s3:PutObject*`, and `s3:ListBucket` to `arn:aws:iam::ACCOUNT_B:role/CrossAccountPipelineRole` and the target service role/instance profile in Account B.
3. **Cross-Account IAM Role in Account B**:
   - **Trust Policy**: Allows `arn:aws:iam::ACCOUNT_A:root` (or the specific CodePipeline Service Role ARN in Account A) to call `sts:AssumeRole`.
   - **Permissions Policy**: Grants `s3:GetObject*` on Account A's artifact bucket, `kms:Decrypt` on Account A's CMK, and `codedeploy:*` / `cloudformation:*` / `ecs:*` to perform the deployment.
4. **CodePipeline Service Role in Account A**:
   - Must allow `sts:AssumeRole` on `arn:aws:iam::ACCOUNT_B:role/CrossAccountPipelineRole`.

### 2.3 CodeBuild Environment & Networking Gotchas

- **Env Var Precedence:** **1 (Highest):** `start-build` API / CLI override / CodePipeline action env vars > **2:** CodeBuild Project definition > **3 (Lowest):** `buildspec.yml` `env.variables`.
- **VPC Internet Access:** CodeBuild ENIs inside a VPC **NEVER** receive a public IPv4 address. Placing CodeBuild in a Public Subnet with an Internet Gateway causes **100% connection timeout** to the internet! **Fix:** Place CodeBuild in a **Private Subnet** whose route table sends `0.0.0.0/0` to a **NAT Gateway** located in a Public Subnet (or use VPC Interface/Gateway Endpoints).
- **Phase Failure Cleanup:** If a command in `build` fails, remaining commands in `build` are skipped, BUT the `finally:` block of `build` AND the `post_build` phase **still execute**! Always put resource cleanup in `finally:` blocks.
- **Docker-in-Docker:** Building Docker images or running `docker buildx` requires `privileged-mode: true` on the CodeBuild project environment.
- **Secrets in Buildspec:** Never put plaintext secrets in `variables:`. Use `env.parameter-store` (for SSM `SecureString`) or `env.secrets-manager: KEY: "secret-id:json-key:version-stage:version-id"`.

---

## 3. Domain 2: Configuration Management & IaC — Deep Reference Matrices

### 3.1 CloudFormation Policies & Helper Scripts

- **`CreationPolicy`**
  - **Purpose & Mechanics:** Attached to `AWS::EC2::Instance` or `AWS::AutoScaling::AutoScalingGroup`. Waits for `Count` signals within `Timeout` (e.g., `PT15M`) via `cfn-signal`.
  - **Exam Trap / Gotcha:** If `cfn-signal` cannot reach the CloudFormation API (missing NAT Gateway or missing `com.amazonaws.<region>.cloudformation` VPC Interface Endpoint in private subnet), the stack hangs until `Timeout` and rolls back!
- **`UpdatePolicy`**
  - **Purpose & Mechanics:** `AutoScalingRollingUpdate` (`MinInstancesInService`, `MaxBatchSize`, `PauseTime`, `WaitOnResourceSignals: true`, `SuspendProcesses: [HealthCheck, ReplaceUnhealthy, AZRebalance, AlarmNotification, ScheduledActions]`).
  - **Exam Trap / Gotcha:** Always include **`SuspendProcesses`** in `AutoScalingRollingUpdate` so ASG scheduled actions or AZ rebalancing don't interfere mid-deployment! If `WaitOnResourceSignals: true`, UserData MUST run `cfn-signal` on every update.
- **`DeletionPolicy`**
  - **Purpose & Mechanics:** Controls resource fate when removed from template or stack deleted: `Delete` (default for most; `Snapshot` default for RDS DBInstance), `Retain`, `RetainExceptOnCreate`, `Snapshot`.
  - **Exam Trap / Gotcha:** When importing existing resources into CloudFormation (`ResourceImport` Change Set), every imported resource **MUST** specify `DeletionPolicy: Retain` (or `Snapshot` where applicable).
- **Custom Resource**
  - **Purpose & Mechanics:** `AWS::CloudFormation::CustomResource` or `Custom::MyResource`. Sends `{RequestType: "Create"|"Update"|"Delete", ResponseURL: "https://s3...", PhysicalResourceId: "..."}` to Lambda/SNS.
  - **Exam Trap / Gotcha:** 
    - 1) If Lambda crashes before uploading JSON to the pre-signed S3 `ResponseURL`, CloudFormation waits **1 hour** before timing out! Always wrap Lambda handler in `try/except/finally` using `cfnresponse.send()`.
    - 2) Returning a **different** `PhysicalResourceId` during an `Update` causes CloudFormation to treat it as a **Replacement** and immediately send a `Delete` request for the old `PhysicalResourceId`!
- **StackSets**
  - **Purpose & Mechanics:** `SELF_MANAGED` requires creating `AWSCloudFormationStackSetAdministrationRole` in admin account and `AWSCloudFormationStackSetExecutionRole` in target accounts. `SERVICE_MANAGED` integrates with AWS Organizations, creates IAM roles automatically, and supports **Auto-Deployment** when accounts join/leave an OU.
  - **Exam Trap / Gotcha:** `FailureToleranceCount` / `FailureTolerancePercentage`: If failures **exceed** this number, the StackSet operation halts immediately. For `MaxConcurrentCount`, setting it higher speeds up rollouts across accounts/regions (`RegionConcurrencyType: PARALLEL`), but increases blast radius if a template bug exists.

### 3.2 Elastic Beanstalk Deployment Policies Comparison

- **All at once**
  - **Capacity During Deploy:** Drops to 0% briefly
  - **Downtime?** **Yes**
  - **Speed:** Fastest
  - **Rollback Mechanism:** Redeploy previous version manually
  - **Best Use Case:** Dev/Test environments
- **Rolling**
  - **Capacity During Deploy:** Reduced by batch size
  - **Downtime?** No
  - **Speed:** Medium
  - **Rollback Mechanism:** Another rolling update required
  - **Best Use Case:** Cost-sensitive non-critical workloads
- **Rolling with additional batch**
  - **Capacity During Deploy:** 100% maintained (launches extra batch first)
  - **Downtime?** No
  - **Speed:** Medium-Slow
  - **Rollback Mechanism:** Another rolling update required
  - **Best Use Case:** Production where capacity cannot drop below 100%
- **Immutable**
  - **Capacity During Deploy:** 100% maintained (creates temporary ASG matching full fleet size)
  - **Downtime?** No
  - **Speed:** Slowest
  - **Rollback Mechanism:** **Instant** — simply terminates the temporary ASG if health checks fail
  - **Best Use Case:** Mission-critical production needing zero-impact rollback
- **Traffic splitting**
  - **Capacity During Deploy:** 100% maintained (creates temporary ASG + shifts ALB target group weights)
  - **Downtime?** No
  - **Speed:** Configurable evaluation window
  - **Rollback Mechanism:** **Instant** — shifts 100% traffic back to original ASG
  - **Best Use Case:** Production Canary testing with CloudWatch Alarm rollback

> [!TIP]
> **`.ebextensions` vs `.platform/hooks`**:
> - In `.ebextensions/*.config`, **`commands:`** run **before** the web server/application is set up, whereas **`container_commands:`** run **after** the application source is unpacked (staged) but before it starts. Use **`leader_only: true`** inside `container_commands` to execute database schema migrations (`flyway migrate` / `alembic upgrade head`) on only **ONE** instance per deployment!
> - On Amazon Linux 2 / AL2023 platforms, prefer executable shell scripts in **`.platform/hooks/prebuild/`**, **`.platform/hooks/predeploy/`**, and **`.platform/hooks/postdeploy/`** for application-level lifecycle scripts.

### 3.3 AWS AppConfig Feature Flags & Deployment Strategies

- **Hierarchy:** Application -> Environment (e.g. `Dev`, `Prod`) -> Configuration Profile (Feature Flags vs. Freeform JSON/YAML) -> Deployment Strategy.
- **Deployment Strategies:**
  - **Growth Type:** `Linear` (fixed percentage every interval) vs. `Exponential` (doubling percentage every interval).
  - **Growth Percentage & Step Duration:** Percentage of targets updated per step duration (e.g. 10% every 2 minutes).
  - **Bake Time:** Duration AppConfig monitors CloudWatch Alarms **after** reaching 100% distribution before marking the deployment complete.
- **Validators:**
  - **JSON Schema:** Validates syntax and schema before rollout.
  - **AWS Lambda Validator:** Runs custom logic (e.g., verifying database compatibility or external dependency health) before deployment starts.
- **Automatic Rollback:** If configured CloudWatch Alarms trigger during deployment or bake time, AppConfig automatically rolls back configuration to the previous known good version.

---

## 4. Domain 3: Resilient Cloud Solutions — Deep Reference Matrices

### 4.1 Disaster Recovery (DR) Patterns Matrix

- **Backup & Restore**
  - **RTO:** Hours (2–24h)
  - **RPO:** Hours (time since last snapshot/backup)
  - **Target Region Footprint (Idle State):** S3 CRR + Cross-Region RDS/EBS Snapshots via AWS Backup. Zero compute running.
  - **Failover Action Required:** Provision infrastructure via CloudFormation/Terraform and restore snapshots.
  - **Relative Cost:** `$` (Lowest)
- **Pilot Light**
  - **RTO:** 10–30 Minutes
  - **RPO:** Seconds to Minutes
  - **Target Region Footprint (Idle State):** Database replicas actively replicating (Aurora Global / RDS Cross-Region Replica) or DRS staging. App servers **OFF** (ASG `Min=0, Desired=0`) or AMIs pre-baked.
  - **Failover Action Required:** Promote DB replica + scale ASG/ECS from `0` to production capacity + flip Route 53.
  - **Relative Cost:** `$$`
- **Warm Standby**
  - **RTO:** Minutes (1–5m)
  - **RPO:** Seconds
  - **Target Region Footprint (Idle State):** Full stack running at **minimum capacity** (e.g., 1–2 small EC2/ECS tasks, active DB replica) continuously handling synthetic health checks.
  - **Failover Action Required:** Flip Route 53 / ARC Routing Control + scale out ASG/ECS to 100% production size.
  - **Relative Cost:** `$$$`
- **Multi-Site Active-Active**
  - **RTO:** **Near Zero (ms–seconds)**
  - **RPO:** **Zero / Near Zero**
  - **Target Region Footprint (Idle State):** Full production capacity in 2+ regions simultaneously. DynamoDB Global Tables or Aurora Global DB (with Write Forwarding / Managed Switchover).
  - **Failover Action Required:** Route 53 Latency/Weighted + ARC Routing Control automatically drains unhealthy region.
  - **Relative Cost:** `$$$$` (Highest)

### 4.2 Route 53 Health Checks vs Application Recovery Controller (ARC)

- **Standard Route 53 Health Checks + DNS Failover**:
  - Relies on the Route 53 **Control Plane** (hosted in `us-east-1`) when updating DNS records or evaluating CloudWatch Alarms in `us-east-1`.
- **Route 53 Application Recovery Controller (ARC)**:
  - **Readiness Check (Control Plane)**: Audits quotas, capacity, ASG sizes, and configuration parity across regions *before* a disaster occurs.
  - **Routing Control (Data Plane)**: Provides 5 globally distributed, quorum-based cluster endpoints. Toggling a Routing Control (`On`/`Off`) updates Route 53 health check states entirely via the **Data Plane** — meaning it works even if the `us-east-1` control plane is completely unreachable!
  - **Safety Rules**: Prevent operator errors during chaos (e.g., an `AssertionRule` ensuring at least 1 region's routing control remains `On` so you never blackhole both regions simultaneously).

### 4.3 Auto Scaling Lifecycle Hooks State Machine

```text
========================================================================================
                        Auto Scaling Lifecycle State Machine
========================================================================================

   [ Start: Scale-Out Event ]
               │
               ▼
          ( Pending )
               │
               ├── [ Launch Lifecycle Hook Attached? ] ──► ( Pending:Wait )
               │                                                  │
               │        ┌─ CompleteLifecycleAction(CONTINUE) ─────┤
               │        │                                         ├─ Complete(ABANDON) / Timeout
               │        ▼                                         │             │
          ( Pending:Proceed )                                     │             ▼
               │                                                  │      ( Terminating )
               ▼                                                  │             │
          [ InService ] ◄─────────────────────────────────────────┘             ▼
          (Enters ALB Target Group & Serves Live Traffic)                  [ Terminated ]
               │
               ▼ ( Scale-In / Unhealthy / Spot Interruption )
       ( Terminating:Wait )
               │
               ├── CompleteLifecycleAction(CONTINUE) or Timeout
               │
               ▼
     ( Terminating:Proceed ) ──► [ Terminated ]
```

- **Key API Calls**:
  - Extend timeout while long script runs: `aws autoscaling record-lifecycle-action-heartbeat --lifecycle-hook-name ... --auto-scaling-group-name ... --lifecycle-action-token ...`
  - Finish hook and proceed: `aws autoscaling complete-lifecycle-action --lifecycle-action-result CONTINUE ...`
  - Default timeout is **3600 seconds (1 hour)**; max with heartbeats is **48 hours** (or max instance lifetime).

---

## 5. Domain 4: Monitoring & Logging — Deep Reference Matrices

### 5.1 CloudWatch Alarm Missing Data Treatments

- **`missing`** (Default)
  - **Behavior When Datapoints Are Missing:** Missing datapoints are ignored, but if all datapoints in evaluation range are missing, alarm goes to `INSUFFICIENT_DATA`.
  - **Ideal DOP-C02 Scenario:** Standard infrastructure metrics (EC2 CPU, RDS connections).
- **`notBreaching`**
  - **Behavior When Datapoints Are Missing:** Missing datapoints are treated as **within the safe threshold** (`OK`).
  - **Ideal DOP-C02 Scenario:** **Error-count Metric Filters** (e.g., counting `"ERROR"` in logs where 0 errors means no metric datapoints are emitted!). Without `notBreaching`, 0 errors causes `INSUFFICIENT_DATA`!
- **`breaching`**
  - **Behavior When Datapoints Are Missing:** Missing datapoints are treated as **violating the threshold** (`ALARM`).
  - **Ideal DOP-C02 Scenario:** **Heartbeat / Liveness Dead-Man's Switch metrics** (if an agent stops sending heartbeats, immediately trigger `ALARM`!).
- **`ignore`**
  - **Behavior When Datapoints Are Missing:** The current alarm state is **maintained** unchanged.
  - **Ideal DOP-C02 Scenario:** Sparse intermittent batch metrics where state shouldn't flip during idle windows.

### 5.2 AWS X-Ray: Annotations vs Metadata

- **Annotations**
  - **Indexed by X-Ray?** **YES**
  - **Searchable in Filter Expressions?** **YES** (`annotation.orderTier = "VIP"`)
  - **Max per Trace:** 50 key-value pairs
  - **Best Use Case:** Filtering traces in ServiceLens/X-Ray by `userId`, `tenantId`, `deploymentVersion`, `region`, `featureFlag`.
- **Metadata**
  - **Indexed by X-Ray?** **NO**
  - **Searchable in Filter Expressions?** **NO**
  - **Max per Trace:** 64 KB total
  - **Best Use Case:** Attaching large diagnostic objects, SQL query strings, request/response headers, or debug arrays for manual inspection.

### 5.3 VPC Flow Logs v5 Custom Fields for Network Debugging

- **Default v2 format** ONLY includes: `version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes start end action log-status`.
- **Why v2 fails across Transit Gateway / NAT Gateway**: `srcaddr` and `dstaddr` show the intermediate NAT Gateway or Transit Gateway ENI IP, hiding the true origin/destination instance IP!
- **v5 Custom Format Fields to Enable**:
  - `pkt-srcaddr` & `pkt-dstaddr`: Reveals the **original packet source IP and final destination IP** across NAT/TGW hops.
  - `flow-direction`: `ingress` vs `egress`.
  - `traffic-path`: Identifies whether egress traffic went via Intra-VPC (`1`), Internet Gateway (`2`), VPC Peering (`3`), Gateway VPC Endpoint (`6`), NAT Gateway (`7`), or Transit Gateway (`8`).
  - `tcp-flags`: Identifies `SYN` (`2`), `SYN-ACK` (`18`), `RST` (`4`), `FIN` (`1`).

---

## 6. Domain 5: Incident & Event Response — Deep Reference Matrices

### 6.1 EventBridge Pattern Matching Cheat Sheet

```json
{
  "source": ["aws.guardduty"],
  "detail-type": ["GuardDuty Finding"],
  "detail": {
    "severity": [{ "numeric": [">=", 7.0, "<=", 10.0] }],
    "type": [{ "prefix": "UnauthorizedAccess:EC2/" }],
    "resource": {
      "instanceDetails": {
        "instanceState": [{ "anything-but": ["terminated", "shutting-down"] }],
        "tags": {
          "Environment": [{ "equals-ignore-case": "production" }]
        }
      }
    }
  }
}
```

### 6.2 Compromised EC2 Instance Forensic Isolation Runbook (Order Matters!)

When an exam question asks how to respond to a compromised EC2 instance in an Auto Scaling Group **while preserving volatile memory and disk evidence**:
1. **Do NOT terminate or reboot** the instance (rebooting destroys volatile RAM!).
2. **Enable Termination Protection** (`ec2:ModifyInstanceAttribute --disable-api-termination`).
3. **Detach the instance from the Auto Scaling Group** (`autoscaling:DetachInstances` with `ShouldDecrementDesiredCapacity: false`) — this keeps the compromised instance alive outside the ASG AND forces the ASG to immediately launch a clean replacement instance! (Alternatively, place the instance in `Standby` if keeping it associated, but `DetachInstances` completely isolates it from ASG lifecycle actions).
4. **Isolate Network**: Replace the instance's Security Groups with a **Forensic Quarantine Security Group** (0 inbound rules, 0 outbound rules — or outbound only to forensic S3 VPC endpoint) and/or apply a deny-all NACL to clear active stateful connections! (Remember: Security Groups are **stateful**, so removing an inbound rule does NOT immediately drop an already established TCP connection in conntrack until timeout, whereas **NACLs are stateless** and immediately block packets in both directions!).
5. **Capture Memory & Disk**: Trigger an SSM Run Command to dump RAM (`LiME`) to a WORM-protected S3 bucket (S3 Object Lock in Compliance mode) and call `ec2:CreateSnapshots` on all attached EBS volumes.
6. **Revoke IAM Credentials**: Attach the `AWSRevokeOlderSessions` inline policy (`aws:TokenIssueTime < Now`) to the instance profile's IAM Role to invalidate stolen IMDS temporary credentials immediately.

---

## 7. Domain 6: Security & Compliance — Deep Reference Matrices

### 7.1 IAM Policy Evaluation Logic Flowchart

```text
========================================================================================
                            AWS IAM Policy Evaluation Logic
========================================================================================

                         [ Incoming AWS API Request ]
                                      │
                                      ▼
               { 1. Is there an Explicit DENY in ANY policy? }
               (SCP, Resource Policy, Identity Policy, Boundary, Session)
                                 │          │
                           (YES) │          │ (NO)
                                 ▼          ▼
            [ EXPLICIT DENY ] ◄──┘   { 2. In AWS Organizations? }
                                     { Do ALL SCPs in path ALLOW? }
                                                │          │
                                          (NO)  │          │ (YES)
                                                ▼          ▼
       [ IMPLICIT DENY (SCP Blocked) ] ◄────────┘   { 3. Does Resource Policy explicitly }
                                                    { ALLOW same-account principal?      }
                                                               │          │
                                                         (YES) │          │ (NO / Cross-Account)
                                                               │          ▼
                                                               │   { 4. Does Identity Policy }
                                                               │   { have an explicit ALLOW? }
                                                               │              │          │
                                                               │        (NO)  │          │ (YES)
                                                               │              ▼          ▼
                                 [ IMPLICIT DENY (No Allow) ] ◄┴──────────────┘   { 5. Permissions Boundary }
                                                                                  { present & ALLOWS?       }
                                                                                             │          │
                                                                                       (NO)  │          │ (YES)
                                                                                             ▼          ▼
                                                   [ IMPLICIT DENY (Boundary Blocked) ] ◄────┘   { 6. Session Policy }
                                                                                                 { present & ALLOWS? }
                                                                                                            │          │
                                                                                                      (NO)  │          │ (YES)
                                                                                                            ▼          ▼
                                                                [ IMPLICIT DENY (Session Blocked) ] ◄───────┘   [ ALLOW ]
```

### 7.2 High-Yield SCP Templates

#### Enforce IMDSv2 & Hop Limit = 1 Across All EC2 Launches
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "RequireIMDSv2OnLaunch",
      "Effect": "Deny",
      "Action": "ec2:RunInstances",
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "StringNotEquals": {
          "ec2:MetadataHttpTokens": "required"
        }
      }
    },
    {
      "Sid": "PreventDisablingIMDSv2OnExistingInstances",
      "Effect": "Deny",
      "Action": "ec2:ModifyInstanceMetadataOptions",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "ec2:MetadataHttpTokens": "required"
        }
      }
    }
  ]
}
```

#### Region Restriction SCP (Allow Only `us-east-1` & `eu-west-1` + Global Services Exemption)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyAllOutsideApprovedRegions",
      "Effect": "Deny",
      "NotAction": [
        "a4b:*", "acm:*", "aws-marketplace-management:*", "aws-marketplace:*",
        "aws-portal:*", "budgets:*", "ce:*", "chime:*", "cloudfront:*",
        "config:*", "cur:*", "directconnect:*", "ec2:DescribeRegions",
        "ec2:DescribeTransitGateways", "ec2:DescribeVpnGateways",
        "fms:*", "globalaccelerator:*", "health:*", "iam:*",
        "importexport:*", "kms:*", "mobileanalytics:*", "networkmanager:*",
        "organizations:*", "pricing:*", "route53:*", "route53domains:*",
        "route53-recovery-cluster:*", "route53-recovery-control-config:*",
        "route53-recovery-readiness:*", "s3:GetAccountPublic*",
        "s3:ListAllMyBuckets", "s3:PutAccountPublic*", "shield:*",
        "sso:*", "sts:*", "support:*", "trustedadvisor:*", "waf-regional:*",
        "waf:*", "wafv2:*", "wellarchitected:*"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": ["us-east-1", "eu-west-1"]
        },
        "ArnNotLike": {
          "aws:PrincipalARN": ["arn:aws:iam::*:role/OrganizationAdminRole"]
        }
      }
    }
  ]
}
```

---

## 8. Top 50 "Pass-or-Fail" DOP-C02 Exam Reflexes

1. **"Cross-account CodePipeline fails with KMS / AccessDenied"** > Replace default `aws/s3` key with a **Customer Managed Key (CMK)** in the Tools account, grant `kms:Decrypt` to the Target Account Role in the CMK Policy, and grant `s3:GetObject*` in the S3 Artifact Bucket Policy.
2. **"Cross-account Auto Scaling Group fails to launch EC2 instances from a shared encrypted AMI"** > Grant `kms:Decrypt`, `kms:DescribeKey`, and **`kms:CreateGrant`** (with condition `kms:GrantIsForAWSResource: true`) on the source CMK directly to the target account's **Auto Scaling Service-Linked Role** (`arn:aws:iam::<TARGET_ACCOUNT_ID>:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling`).
3. **"CodeBuild inside a VPC times out running `npm install` / `pip install` even though it is in a public subnet"** > Move CodeBuild to a **Private Subnet** routing `0.0.0.0/0` to a **NAT Gateway** in a Public Subnet (CodeBuild ENIs never get public IPs).
4. **"Monorepo in CodeCommit / GitHub triggers all microservice pipelines on every commit"** > Use **CodePipeline V2 trigger `filePaths` filter** (for GitHub/CodeConnections) OR an **EventBridge rule + Lambda** inspecting `codecommit:GetDifferences` to trigger only the modified microservice's pipeline.
5. **"Run database migrations on only ONE instance during an Elastic Beanstalk deployment"** > Use `container_commands` with **`leader_only: true`** in `.ebextensions/*.config`.
6. **"CloudFormation Custom Resource hangs for 60 minutes during `Delete` or `Update`"** > The backing Lambda function threw an unhandled exception and never uploaded a response JSON to the S3 pre-signed **`ResponseURL`**. Wrap the Lambda handler in `try/catch/finally` using `cfn-response`.
7. **"Cannot delete or update CloudFormation Stack A because Stack B uses `Fn::ImportValue` on its Output"** > Replace hard-coupled CloudFormation Exports with **SSM Parameter Store** parameters and dynamic references (`{{resolve:ssm:...}}`) or first update Stack B to remove the `Fn::ImportValue` reference.
8. **"Automatically deploy baseline IAM/Config/GuardDuty stacks whenever a new AWS Account is created or moved into an OU"** > Use **CloudFormation StackSets** with **`SERVICE_MANAGED`** permissions and `AutoDeployment: Enabled`.
9. **"Allow developers to create IAM Roles for Lambda/ECS without allowing privilege escalation to Admin"** > Attach an **IAM Permissions Boundary** and require `iam:PermissionsBoundary` condition on every `iam:CreateRole` / `iam:PutRolePermissionsBoundary` action.
10. **"Prevent even `AdministratorAccess` users in member accounts from turning off CloudTrail, GuardDuty, or launching in unapproved regions"** > Attach a **Service Control Policy (SCP)** at the Root or OU level in AWS Organizations (remember SCPs do not affect the Management Account).
11. **"Ensure zero-downtime database password rotation for an application that caches connection pools"** > Use **AWS Secrets Manager** with the **Alternating Users (`2-user`) rotation strategy** and/or **Amazon RDS Proxy**.
12. **"Capture memory dumps or application logs from EC2 instances in an Auto Scaling Group before they terminate on scale-in"** > Configure an ASG **`Terminating:Wait` Lifecycle Hook** > EventBridge rule (`EC2 Instance-terminate Lifecycle Action`) > SSM Run Command / Automation > `CompleteLifecycleAction`.
13. **"Speed up Auto Scaling out from 8 minutes to 30 seconds for an application with heavy boot-time initialization"** > Configure an **EC2 Auto Scaling Warm Pool** (with instances pre-initialized in `Stopped`, `Running`, or `Hibernated` state) rather than just baking an AMI if runtime state initialization is still slow.
14. **"Fail over multi-region DNS traffic even if the `us-east-1` Route 53 control plane is experiencing a service disruption"** > Use **Amazon Route 53 Application Recovery Controller (ARC) Routing Controls** hosted on the 5 regional data-plane cluster endpoints.
15. **"Perform a planned quarterly DR fail-over test of an Amazon Aurora Global Database with zero data loss (`RPO = 0`)"** > Use **Aurora Global Database Managed Planned Failover (`FailoverGlobalCluster`)**, which waits for secondary storage to catch up completely and preserves the replication topology.
16. **"Identify which specific DynamoDB partition key is causing `ProvisionedThroughputExceededException` throttling"** > Enable **CloudWatch Contributor Insights for DynamoDB**.
17. **"Mask credit card numbers and SSNs written to CloudWatch Logs before developers can view them"** > Apply a **CloudWatch Logs Data Protection Policy** (Account-level or Log Group-level); only IAM principals granted `logs:Unmask` can view unmasked data.
18. **"Suppress dozens of downstream microservice CloudWatch alarms during a scheduled maintenance window or when the primary database is down"** > Create a **CloudWatch Composite Alarm** (`ALARM(ServiceAlarm) AND NOT ALARM(DatabaseDownOrMaintenanceAlarm)`) and send notifications only from the Composite Alarm.
19. **"Metric Filter counting `ERROR` log lines goes into `INSUFFICIENT_DATA` whenever the application is healthy and emits zero errors"** > Set the CloudWatch Alarm Missing Data treatment to **`treatMissingData: notBreaching`**.
20. **"Filter AWS X-Ray traces by `CustomerTier` or `OrderID` in the ServiceLens console"** > Record the key-value pair as an X-Ray **Annotation** (indexed), NOT Metadata (unindexed).
21. **"Identify original client and destination pod/instance IPs in VPC Flow Logs when traffic crosses a Transit Gateway or NAT Gateway"** > Create a **Custom Format (v5) VPC Flow Log** including **`pkt-srcaddr`**, **`pkt-dstaddr`**, and **`traffic-path`**.
22. **"Isolate a GuardDuty-flagged EC2 instance in an Auto Scaling Group for forensics without ASG terminating it or losing capacity"** > Enable Termination Protection + call `DetachInstances` (`ShouldDecrementDesiredCapacity: false`) + attach Quarantine Security Group & NACL + snapshot EBS + dump RAM.
23. **"Test multi-AZ resilience of EC2, ECS, EKS, and RDS in production with an automatic emergency kill-switch if error rates spike"** > Use **AWS Fault Injection Service (FIS)** with a **CloudWatch Alarm Stop Condition**.
24. **"Allow non-technical teams to provision approved multi-tier environments without granting them `ec2:*` or `rds:*` IAM permissions"** > Use **AWS Service Catalog** with a **Launch Constraint** (Service Catalog assumes the privileged Launch Role; end-users only need `servicecatalog:*` permissions).
25. **"Ensure on-premises servers can assume AWS IAM Roles without static long-lived IAM User access keys or SSM Agent"** > Use **IAM Roles Anywhere** with X.509 certificates issued by an **AWS Private CA** and `aws_signing_helper`.
26. **"Calculate error rate percentage `(5xx / Total) * 100` dynamically in CloudWatch Alarms"** > Use **CloudWatch Metric Math** expressions (`e1 = (m1 / m2) * 100`) and set the alarm threshold on the resulting math expression `e1`.
27. **"Schedule millions of recurring one-time or recurring tasks with flexible execution time windows and high throughput"** > Use **Amazon EventBridge Scheduler** with flexible time windows (`FlexibleTimeWindow`) and native dead-letter queues (DLQs), rather than legacy CloudWatch Events cron rules.
28. **"Authenticate Kubernetes pods to AWS APIs without managing OIDC provider URLs or hitting 2KB trust policy limits across clusters"** > Use **Amazon EKS Pod Identity**; configure the EKS Pod Identity Agent and associate IAM roles directly with Kubernetes ServiceAccounts.
29. **"Deploy CloudFormation templates directly on Git push without setting up CodePipeline or external CI/CD servers"** > Configure **CloudFormation Git Sync** via AWS CodeConnections and a repository `deployment-definition.yaml`.
30. **"Enforce configuration state drift remediation continuously on a fleet of EC2 instances"** > Use **AWS Systems Manager State Manager Associations** on a recurring schedule with compliance reporting.
31. **"Deploy application updates to Elastic Beanstalk with zero downtime and instant rollback without waiting for a reverse rolling update"** > Use **Elastic Beanstalk Immutable Deployment**; it provisions a temporary ASG and performs an instant rollback if health checks fail.
32. **"Validate feature flag changes before distribution and automatically roll back if user error rates increase post-rollout"** > Use **AWS AppConfig** with a **JSON Schema/Lambda Validator** and a deployment strategy with a configured **Bake Time** monitored by a CloudWatch Alarm.
33. **"Protect backups against ransomware deletion and ensure regulatory compliance with immutable retention across multiple AWS accounts"** > Use **AWS Backup Vault Lock** in **Compliance Mode** (immutable, cannot be deleted even by root) managed via **AWS Organizations Centralized Backup Policies**.
34. **"Integrate DynamoDB Streams directly to Amazon SQS or Step Functions with built-in payload filtering without writing glue Lambda code"** > Use **Amazon EventBridge Pipes** with native source filtering and enrichment steps.
35. **"Centralize CloudWatch metrics, log groups, and X-Ray traces across hundreds of AWS accounts and regions in a single monitoring account"** > Configure **CloudWatch Cross-Account Observability** using **AWS Observability Access Manager (OAM)** links (`aws:oam:CreateLink`).
36. **"Perform graph-based visual root cause investigation of unauthorized API calls and anomalous IP traffic identified by GuardDuty"** > Use **Amazon Detective**; it automatically ingests CloudTrail, VPC Flow Logs, and EKS audit logs to build interactive visual graph trees.
37. **"Proactively verify that customer checkout workflows and private internal APIs succeed 24/7 before real users are impacted"** > Deploy **CloudWatch Synthetics Canaries** (running inside private VPC subnets for internal endpoints) with automated screenshots and alarms.
38. **"Allow engineers to securely connect and tunnel TCP/SSH traffic to private EC2 instances without opening port 22 or deploying bastion hosts"** > Use **AWS Systems Manager Session Manager** with **Port Forwarding** (`AWS-StartPortForwardingSession`).
39. **"Prevent accidental loss of RDS databases or S3 buckets when a CloudFormation stack is deleted or resources are replaced during updates"** > Set both **`DeletionPolicy: Retain`** (or `Snapshot`) and **`UpdateReplacePolicy: Retain`** (or `Snapshot`) on critical stateful resources.
40. **"Host private internal npm and PyPI packages securely while transparently caching external packages from npmjs.com and PyPI"** > Use **AWS CodeArtifact** with an **External Connection** to the public repository and configure upstream repository relationships.
41. **"Prevent Auto Scaling thrashing when an application takes 10 minutes to stabilize after scaling out"** > Adjust the **Scale-Out Cooldown** (or step adjustment metric aggregation interval) or use **Target Tracking Scaling Policies** with appropriate metric pre-warming.
42. **"Process millions of S3 objects concurrently in serverless workflows without hitting Lambda concurrency limits or Step Functions execution history limits"** > Use **AWS Step Functions Distributed Map** in `Distributed` mode with high concurrency (up to 10,000 parallel executions).
43. **"Stream near-real-time CloudWatch metrics to an external observability tool (Datadog/Splunk) with sub-2-minute latency"** > Configure **CloudWatch Metric Streams** with **Amazon Kinesis Data Firehose**.
44. **"Receive proactive notifications when underlying AWS hardware degradation or regional service issues impact your specific EC2/RDS resources"** > Deploy the **AWS Health Aware (AHA)** solution ingesting AWS Health events via EventBridge into Slack, Teams, or PagerDuty.
45. **"Route global DNS users to the closest healthy AWS region while balancing capacity and shifting traffic dynamically via administrative bias"** > Use **Amazon Route 53 Geoproximity Routing** via **Traffic Flow** and adjust routing biases.
46. **"Detect failed ECS container task deployments automatically and revert to the previous task definition revision without manual operator action"** > Enable the **Amazon ECS Deployment Circuit Breaker** with **Rollback** (`enable: true, rollback: true`).
47. **"Drastically reduce Java Lambda function cold start latency from several seconds to under 200 milliseconds"** > Enable **AWS Lambda SnapStart for Java**, which initializes the execution environment ahead of time and caches an encrypted microVM memory snapshot.
48. **"Encrypt data in `us-east-1` and decrypt it in `eu-central-1` during disaster recovery without making cross-region KMS network API calls"** > Use **AWS KMS Multi-Region Keys (MRKs)**; they share the same key material, key ID, and ARN prefix across regions.
49. **"Enforce corporate tagging and encryption compliance before CloudFormation templates can even be deployed in CI/CD pipelines"** > Implement **AWS CloudFormation Guard (`cfn-guard`)** policy-as-code rules in pre-commit and CodeBuild test stages.
50. **"Provide a single global S3 endpoint URL that transparently routes client requests to the closest regional bucket replica with automatic failover"** > Create an **Amazon S3 Multi-Region Access Point (MRAP)** backed by AWS Global Accelerator.
