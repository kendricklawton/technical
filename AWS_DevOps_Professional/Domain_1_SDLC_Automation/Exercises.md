# Domain 1: SDLC Automation — Production-Grade Hands-On Exercises (DOP-C02)

---

## Exercise 1: Multi-Account Cross-Region CodePipeline with KMS CMK & S3 Bucket Policies

### Scenario
Your enterprise operates a centralized **DevOps Tools Account (`111111111111`)** in `us-east-1` and `eu-west-1` that builds and deploys applications into a **Production Account (`222222222222`)**.
You must architect a Cross-Account, Cross-Region AWS CodePipeline in `111111111111` that:
1. Pulls source code from CodeCommit and builds artifacts in `us-east-1`.
2. Deploys a CloudFormation stack in **Production Account (`222222222222`)** in both `us-east-1` (Primary) and `eu-west-1` (Secondary DR).
3. Encrypts all artifacts using regional Customer Managed KMS Keys (CMKs).

### Tasks
1. Write the **Tools Account KMS CMK Key Policy** (`us-east-1`) granting least-privilege access to the Tools Account CodePipeline Role and the Production Account Cross-Account Role.
2. Write the **Tools Account S3 Artifact Bucket Policy** (`us-east-1`).
3. Write the **Production Account Cross-Account Action Role** (`ProdCrossAccountPipelineRole`) Trust Policy, Permission Policy (including `iam:PassRole` for the CloudFormation Execution Role), and the `ArtifactStores` section of the CodePipeline definition.

---

### Complete Working Solution

#### 1. Tools Account KMS CMK Key Policy (`111111111111` in `us-east-1`)
> **Why CMK is mandatory:** AWS-managed keys (`alias/aws/s3`) do not allow modifying their key policy to trust `arn:aws:iam::222222222222:role/ProdCrossAccountPipelineRole`, which causes `AccessDeniedException` on `kms:Decrypt` during cross-account artifact retrieval.

```json
{
  "Version": "2012-10-17",
  "Id": "CrossAccountPipelineKMSPolicy",
  "Statement": [
    {
      "Sid": "EnableToolsAccountRootAdmin",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111111111111:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "AllowToolsCodePipelineAndCodeBuildUsage",
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::111111111111:role/ToolsCodePipelineServiceRole",
          "arn:aws:iam::111111111111:role/ToolsCodeBuildServiceRole"
        ]
      },
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
      ],
      "Resource": "*"
    },
    {
      "Sid": "AllowProdCrossAccountRolesCryptographicOperations",
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::222222222222:role/ProdCrossAccountPipelineRole",
          "arn:aws:iam::222222222222:role/ProdCloudFormationExecutionRole"
        ]
      },
      "Action": [
        "kms:Decrypt",
        "kms:DescribeKey",
        "kms:GenerateDataKey*",
        "kms:ReEncrypt*"
      ],
      "Resource": "*"
    }
  ]
}
```

#### 2. Tools Account S3 Artifact Bucket Policy (`codepipeline-artifacts-us-east-1-111111111111`)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyUnencryptedObjectUploads",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::codepipeline-artifacts-us-east-1-111111111111/*",
      "Condition": {
        "StringNotEquals": {
          "s3:x-amz-server-side-encryption": "aws:kms"
        }
      }
    },
    {
      "Sid": "DenyInsecureTransport",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::codepipeline-artifacts-us-east-1-111111111111",
        "arn:aws:s3:::codepipeline-artifacts-us-east-1-111111111111/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    },
    {
      "Sid": "AllowProdAccountGetAndPutArtifacts",
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::222222222222:role/ProdCrossAccountPipelineRole",
          "arn:aws:iam::222222222222:role/ProdCloudFormationExecutionRole"
        ]
      },
      "Action": [
        "s3:GetObject",
        "s3:GetObjectVersion",
        "s3:PutObject",
        "s3:PutObjectAcl"
      ],
      "Resource": "arn:aws:s3:::codepipeline-artifacts-us-east-1-111111111111/*"
    },
    {
      "Sid": "AllowProdAccountListBucket",
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::222222222222:role/ProdCrossAccountPipelineRole",
          "arn:aws:iam::222222222222:role/ProdCloudFormationExecutionRole"
        ]
      },
      "Action": [
        "s3:ListBucket",
        "s3:GetBucketLocation",
        "s3:GetBucketVersioning"
      ],
      "Resource": "arn:aws:s3:::codepipeline-artifacts-us-east-1-111111111111"
    }
  ]
}
```

#### 3. Production Account Cross-Account Role & Multi-Region `ArtifactStores` Configuration
```yaml
# Deployed in Production Account (222222222222)
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  ProdCrossAccountPipelineRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: ProdCrossAccountPipelineRole
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              AWS: arn:aws:iam::111111111111:role/ToolsCodePipelineServiceRole
            Action: sts:AssumeRole
      Policies:
        - PolicyName: CrossAccountDeployPermissions
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Sid: ArtifactBucketAccessBothRegions
                Effect: Allow
                Action:
                  - s3:GetObject*
                  - s3:GetBucket*
                  - s3:List*
                  - s3:PutObject*
                Resource:
                  - arn:aws:s3:::codepipeline-artifacts-us-east-1-111111111111
                  - arn:aws:s3:::codepipeline-artifacts-us-east-1-111111111111/*
                  - arn:aws:s3:::codepipeline-artifacts-eu-west-1-111111111111
                  - arn:aws:s3:::codepipeline-artifacts-eu-west-1-111111111111/*
              - Sid: KMSDecryptBothRegions
                Effect: Allow
                Action:
                  - kms:Decrypt
                  - kms:DescribeKey
                  - kms:GenerateDataKey*
                  - kms:ReEncrypt*
                Resource:
                  - arn:aws:kms:us-east-1:111111111111:key/mrk-1111aaaa2222bbbb
                  - arn:aws:kms:eu-west-1:111111111111:key/mrk-1111aaaa2222bbbb
              - Sid: CloudFormationAndPassRole
                Effect: Allow
                Action:
                  - cloudformation:CreateStack
                  - cloudformation:UpdateStack
                  - cloudformation:DeleteStack
                  - cloudformation:DescribeStacks
                  - cloudformation:CreateChangeSet
                  - cloudformation:ExecuteChangeSet
                  - cloudformation:DescribeChangeSet
                Resource: arn:aws:cloudformation:*:222222222222:stack/ProdAppStack*
              - Sid: PassCloudFormationRole
                Effect: Allow
                Action: iam:PassRole
                Resource: arn:aws:iam::222222222222:role/ProdCloudFormationExecutionRole
                Condition:
                  StringEquals:
                    iam:PassedToService: cloudformation.amazonaws.com
```

```yaml
# Multi-Region CodePipeline V2 Snippet in Tools Account (111111111111)
  MultiRegionProdPipeline:
    Type: AWS::CodePipeline::Pipeline
    Properties:
      Name: Enterprise-CrossAccount-MultiRegion-Pipeline
      PipelineType: V2
      ExecutionMode: QUEUED
      RoleArn: arn:aws:iam::111111111111:role/ToolsCodePipelineServiceRole
      ArtifactStores:
        - Region: us-east-1
          ArtifactStore:
            Type: S3
            Location: codepipeline-artifacts-us-east-1-111111111111
            EncryptionKey:
              Id: arn:aws:kms:us-east-1:111111111111:key/mrk-1111aaaa2222bbbb
              Type: KMS
        - Region: eu-west-1
          ArtifactStore:
            Type: S3
            Location: codepipeline-artifacts-eu-west-1-111111111111
            EncryptionKey:
              Id: arn:aws:kms:eu-west-1:111111111111:key/mrk-1111aaaa2222bbbb
              Type: KMS
      Stages:
        # Source & Build stages omitted for brevity
        - Name: DeployProdMultiRegion
          Actions:
            - Name: Deploy-US-East-1
              Region: us-east-1
              RoleArn: arn:aws:iam::222222222222:role/ProdCrossAccountPipelineRole
              ActionTypeId:
                Category: Deploy
                Owner: AWS
                Provider: CloudFormation
                Version: '1'
              Configuration:
                ActionMode: CREATE_UPDATE
                StackName: ProdAppStack
                TemplatePath: BuildOutput::template.yaml
                Capabilities: CAPABILITY_NAMED_IAM
                RoleArn: arn:aws:iam::222222222222:role/ProdCloudFormationExecutionRole
              InputArtifacts:
                - Name: BuildOutput
              RunOrder: 1
            - Name: Deploy-EU-West-1
              Region: eu-west-1
              RoleArn: arn:aws:iam::222222222222:role/ProdCrossAccountPipelineRole
              ActionTypeId:
                Category: Deploy
                Owner: AWS
                Provider: CloudFormation
                Version: '1'
              Configuration:
                ActionMode: CREATE_UPDATE
                StackName: ProdAppStack
                TemplatePath: BuildOutput::template.yaml
                Capabilities: CAPABILITY_NAMED_IAM
                RoleArn: arn:aws:iam::222222222222:role/ProdCloudFormationExecutionRole
              InputArtifacts:
                - Name: BuildOutput
              RunOrder: 2
```

---

## Exercise 2: CodeDeploy EC2 Lifecycle Hook Failure Diagnosis & CloudWatch Alarm Auto-Rollback

### Scenario
An EC2 deployment (`d-A1B2C3D4E`) fails during `ApplicationStop` because the previous deployment (`d-0000PREV0`) shipped a defective `stop_service.sh` script that exited with status code `1` when `systemd` service was not running. Furthermore, you must configure the `appspec.yml` and Deployment Group so that:
1. Local HTTP readiness (`ValidateService`) and graceful connection draining (`BeforeBlockTraffic`) are verified.
2. Any spike in ALB `HTTPCode_Target_5XX_Count` automatically rolls back the deployment.

---

### Complete Working Solution

#### 1. Diagnosing and Bypassing the Broken `ApplicationStop` Hook
```bash
# 1. Inspect exact script stderr on the target EC2 instance
sudo tail -n 100 /opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log
sudo cat /opt/codedeploy-agent/deployment-root/<deployment-group-id>/d-A1B2C3D4E/logs/scripts.log

# 2. Because ApplicationStop runs from the PREVIOUS revision's bundle before DownloadBundle,
#    trigger the new deployment with --ignore-application-stop-failures:
aws deploy create-deployment \
  --application-name OrderServiceApp \
  --deployment-group-name OrderServiceProdGroup \
  --s3-location bucket=prod-artifacts,key=orders/v2.1.zip,bundleType=zip \
  --ignore-application-stop-failures
```

#### 2. Production `appspec.yml` and Defensive Lifecycle Scripts
```yaml
version: 0.0
os: linux
files:
  - source: build/orders-service
    destination: /opt/orders/bin
  - source: systemd/orders.service
    destination: /etc/systemd/system
file_exists_behavior: OVERWRITE
permissions:
  - object: /opt/orders/bin
    pattern: "orders-service"
    owner: orders
    group: orders
    mode: 755
    type:
      - file
hooks:
  BeforeBlockTraffic:
    - location: scripts/before_block_traffic.sh
      timeout: 60
      runas: root
  ApplicationStop:
    - location: scripts/application_stop.sh
      timeout: 60
      runas: root
  BeforeInstall:
    - location: scripts/before_install.sh
      timeout: 120
      runas: root
  AfterInstall:
    - location: scripts/after_install.sh
      timeout: 120
      runas: root
  ApplicationStart:
    - location: scripts/application_start.sh
      timeout: 120
      runas: root
  ValidateService:
    - location: scripts/validate_service.sh
      timeout: 90
      runas: orders
```

```bash
#!/usr/bin/env bash
# scripts/application_stop.sh — Defensive idempotent stop script
set -euo pipefail
if systemctl is-active --quiet orders.service; then
  echo "Stopping orders.service..."
  systemctl stop orders.service
else
  echo "orders.service is not active; skipping stop cleanly."
fi
exit 0
```

```bash
#!/usr/bin/env bash
# scripts/validate_service.sh — Local readiness verification before ALB AllowTraffic
set -euo pipefail
for i in {1..15}; do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:8080/health/ready || true)
  if [ "$STATUS" = "200" ]; then
    echo "Service validated healthy on attempt $i"
    exit 0
  fi
  echo "Attempt $i: got HTTP $STATUS, retrying in 3s..."
  sleep 3
done
echo "ERROR: Service failed local readiness check!" >&2
exit 1
```

#### 3. CloudFormation CodeDeploy DeploymentGroup with Auto-Rollback on Alarm
```yaml
  OrderServiceDeploymentGroup:
    Type: AWS::CodeDeploy::DeploymentGroup
    Properties:
      ApplicationName: !Ref OrderServiceApp
      DeploymentGroupName: OrderServiceProdGroup
      ServiceRoleArn: !GetAtt CodeDeployServiceRole.Arn
      DeploymentConfigName: CodeDeployDefault.HalfAtATime
      AutoScalingGroups:
        - !Ref OrderServiceASG
      DeploymentStyle:
        DeploymentType: IN_PLACE
        DeploymentOption: WITH_TRAFFIC_CONTROL
      LoadBalancerInfo:
        TargetGroupInfoList:
          - Name: !GetAtt OrderServiceTargetGroup.TargetGroupName
      AlarmConfiguration:
        Enabled: true
        IgnorePollAlarmFailure: false
        Alarms:
          - Name: !Ref Target5xxSpikeAlarm
      AutoRollbackConfiguration:
        Enabled: true
        Events:
          - DEPLOYMENT_FAILURE
          - DEPLOYMENT_STOP_ON_ALARM
          - DEPLOYMENT_STOP_ON_REQUEST
```

---

## Exercise 3: CodeBuild in a Private VPC with Aurora Migrations, Caching, Multi-Arch Buildx, and JUnit Reporting

### Scenario
Your CodeBuild project must run inside a **Private VPC Subnet** to execute database schema migrations (`flyway migrate`) against a private **Amazon Aurora PostgreSQL** cluster before building a Docker container image, running `pytest` with JUnit XML and Cobertura coverage reporting, and pushing the image to Amazon ECR.

---

### Complete Working Solution

#### 1. Complete `buildspec.yml`
```yaml
version: 0.2

env:
  shell: bash
  git-credential-helper: yes
  variables:
    FLYWAY_CONNECT_RETRIES: "5"
    DOCKER_BUILDKIT: "1"
  parameter-store:
    AURORA_HOST: "/prod/Aurora/EndpointAddress"
    AURORA_DB: "/prod/Aurora/DatabaseName"
    CODEARTIFACT_DOMAIN: "/org/codeartifact/domain"
  secrets-manager:
    DB_USER: "prod/aurora/master-creds:username:AWSCURRENT"
    DB_PASS: "prod/aurora/master-creds:password:AWSCURRENT"
  exported-variables:
    - IMAGE_URI
    - GIT_COMMIT_SHORT

phases:
  install:
    runtime-versions:
      python: 3.12
    commands:
      - echo "Authenticating pip with private AWS CodeArtifact via VPC Endpoint..."
      - aws codeartifact login --tool pip --domain $CODEARTIFACT_DOMAIN --repository internal-pypi
      - pip install --upgrade pip pytest pytest-cov

  pre_build:
    commands:
      - export GIT_COMMIT_SHORT=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
      - export IMAGE_URI="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/payment-api:${GIT_COMMIT_SHORT}"
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
      - echo "Testing TCP connectivity to private Aurora PostgreSQL on 5432..."
      - timeout 5 bash -c "</dev/tcp/${AURORA_HOST}/5432"

  build:
    commands:
      - echo "Running unit and integration tests against private Aurora..."
      - pytest --junitxml=reports/junit.xml --cov=src --cov-report=xml:reports/coverage.xml
      - echo "Executing Flyway database schema migrations..."
      - flyway -url="jdbc:postgresql://${AURORA_HOST}:5432/${AURORA_DB}" -user="$DB_USER" -password="$DB_PASS" -locations="filesystem:./sql" migrate
      - echo "Building container image with local Docker layer cache..."
      - docker build --tag "$IMAGE_URI" --tag "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/payment-api:latest" .
    finally:
      - echo "Cleaning up temporary database credentials from memory..."
      - unset DB_PASS

  post_build:
    commands:
      - docker push "$IMAGE_URI"
      - docker push "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/payment-api:latest"
      - printf '{"ImageURI":"%s"}' "$IMAGE_URI" > imageDetail.json

reports:
  PytestJUnitReport:
    files:
      - "junit.xml"
    base-directory: "reports"
    file-format: JUNITXML
  PytestCoverageReport:
    files:
      - "coverage.xml"
    base-directory: "reports"
    file-format: COBERTURAXML

artifacts:
  files:
    - imageDetail.json
    - appspec.yaml
    - taskdef.json
  discard-paths: yes

cache:
  paths:
    - "/root/.cache/pip/**/*"
```

#### 2. CloudFormation CodeBuild Project with Private VPC, Security Group Chaining, & Local Cache
```yaml
  CodeBuildSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security group for CodeBuild ENIs in private subnets
      VpcId: !Ref AppVPC
      SecurityGroupEgress:
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0
          Description: HTTPS outbound via NAT Gateway or VPC Endpoints
        - IpProtocol: tcp
          FromPort: 5432
          ToPort: 5432
          DestinationSecurityGroupId: !Ref AuroraSecurityGroup
          Description: PostgreSQL access to private Aurora cluster

  AuroraIngressFromCodeBuild:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !Ref AuroraSecurityGroup
      IpProtocol: tcp
      FromPort: 5432
      ToPort: 5432
      SourceSecurityGroupId: !Ref CodeBuildSecurityGroup

  PaymentServiceBuildProject:
    Type: AWS::CodeBuild::Project
    Properties:
      Name: PaymentService-PrivateVPC-Build
      ServiceRole: !GetAtt CodeBuildServiceRole.Arn
      Artifacts:
        Type: CODEPIPELINE
      Environment:
        Type: LINUX_CONTAINER
        ComputeType: BUILD_GENERAL1_MEDIUM
        Image: aws/codebuild/amazonlinux2-x86_64-standard:5.0
        PrivilegedMode: true # Required for Docker builds and LOCAL_DOCKER_LAYER_CACHE
      Cache:
        Type: LOCAL
        Modes:
          - LOCAL_SOURCE_CACHE
          - LOCAL_DOCKER_LAYER_CACHE
          - LOCAL_CUSTOM_CACHE
      VpcConfig:
        VpcId: !Ref AppVPC
        Subnets:
          - !Ref PrivateSubnet1A # Must route 0.0.0.0/0 -> NAT Gateway!
          - !Ref PrivateSubnet1B
        SecurityGroupIds:
          - !Ref CodeBuildSecurityGroup
```

---

## Exercise 4: ECS Fargate Blue/Green Canary Deployment with Test Listener Validation Lambda Hook

### Scenario
You are deploying a containerized microservice (`payment-api`) on **Amazon ECS Fargate** behind an **Application Load Balancer**.
* Production traffic enters on **HTTPS Port 443** (`ProdListener`).
* Synthetic test traffic enters on **HTTPS Port 8443** (`TestListener`).
* You want a **Canary deployment** (`CodeDeployDefault.ECSCanary10Percent5Minutes`) that first exposes the new Green task set on Port `8443`, executes a Lambda hook during **`AfterAllowTestTraffic`** to validate the `/v1/health` endpoint on Port `8443`, and automatically aborts/rolls back if validation fails.

---

### Complete Working Solution

#### 1. `taskdef.json` (with `<IMAGE1_NAME>` Dynamic Placeholder)
```json
{
  "family": "payment-api-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::222222222222:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::222222222222:role/paymentApiTaskRole",
  "containerDefinitions": [
    {
      "name": "payment-api",
      "image": "<IMAGE1_NAME>",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 8080,
          "protocol": "tcp"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/payment-api",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

#### 2. `appspec.yaml` for ECS Blue/Green
```yaml
version: 0.0
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: "<TASK_DEFINITION>"
        LoadBalancerInfo:
          ContainerName: "payment-api"
          ContainerPort: 8080
        PlatformVersion: "LATEST"
Hooks:
  - AfterAllowTestTraffic: "arn:aws:lambda:us-east-1:222222222222:function:CodeDeployHook_ValidateTestListener8443"
```

#### 3. Python Lambda Hook (`CodeDeployHook_ValidateTestListener8443`)
```python
import os
import json
import urllib.request
import urllib.error
import boto3

codedeploy = boto3.client("codedeploy")
TEST_ALB_URL = os.environ["TEST_ALB_URL"]  # e.g., "https://internal-alb.acme.com:8443/v1/health"


def lambda_handler(event, context):
    print("Received CodeDeploy Hook Event:", json.dumps(event))
    deployment_id = event["DeploymentId"]
    hook_execution_id = event["LifecycleEventHookExecutionId"]
    status = "Failed"

    try:
        req = urllib.request.Request(
            TEST_ALB_URL,
            headers={"User-Agent": "CodeDeploy-AfterAllowTestTraffic-Validator"},
            method="GET",
        )
        with urllib.request.urlopen(req, timeout=10) as response:
            http_code = response.getcode()
            body = json.loads(response.read().decode("utf-8"))
            print(f"Test Listener Response: HTTP {http_code} - {body}")

            if http_code == 200 and body.get("status") == "UP":
                status = "Succeeded"
            else:
                print(f"Validation failed: unexpected status payload {body}")
    except Exception as exc:
        print(f"Exception hitting Test Listener {TEST_ALB_URL}: {exc}")
        status = "Failed"

    # Mandatory callback to CodeDeploy
    codedeploy.put_lifecycle_event_hook_execution_status(
        deploymentId=deployment_id,
        lifecycleEventHookExecutionId=hook_execution_id,
        status=status,
    )
    print(f"Reported status={status} to CodeDeploy for deployment {deployment_id}")
    return {"status": status}
```

---

## Exercise 5: Monorepo Selective CI/CD Triggering with EventBridge & Lambda Diff Parser

### Scenario
Your team stores three microservices (`services/auth/`, `services/billing/`, `services/inventory/`) and a shared library (`libs/common/`) inside a single **AWS CodeCommit** repository named `PlatformMonorepo`.
You must ensure that a push to `refs/heads/main`:
* Triggers **only** the CodePipeline(s) corresponding to the modified `services/<name>/` directory.
* Triggers **all three** pipelines if any file under `libs/common/` is modified.

---

### Complete Working Solution

#### 1. Amazon EventBridge Rule Matching `main` Branch Updates
```json
{
  "source": ["aws.codecommit"],
  "detail-type": ["CodeCommit Repository State Change"],
  "resources": ["arn:aws:codecommit:us-east-1:111111111111:PlatformMonorepo"],
  "detail": {
    "event": ["referenceUpdated"],
    "referenceType": ["branch"],
    "referenceName": ["main"]
  }
}
```

#### 2. Python Lambda Diff Inspector (`codecommit:GetDifferences` -> `codepipeline:StartPipelineExecution`)
```python
import json
import boto3

codecommit = boto3.client("codecommit")
codepipeline = boto3.client("codepipeline")

SERVICE_PIPELINE_MAP = {
    "services/auth/": "AuthService-Prod-Pipeline",
    "services/billing/": "BillingService-Prod-Pipeline",
    "services/inventory/": "InventoryService-Prod-Pipeline",
}
SHARED_LIB_PREFIX = "libs/common/"


def get_changed_paths(repo_name: str, before_commit: str, after_commit: str) -> set[str]:
    changed_files = set()
    next_token = None

    while True:
        kwargs = {
            "repositoryName": repo_name,
            "beforeCommitSpecifier": before_commit,
            "afterCommitSpecifier": after_commit,
        }
        if next_token:
            kwargs["NextToken"] = next_token

        resp = codecommit.get_differences(**kwargs)
        for diff in resp.get("differences", []):
            if "afterBlob" in diff and "path" in diff["afterBlob"]:
                changed_files.add(diff["afterBlob"]["path"])
            if "beforeBlob" in diff and "path" in diff["beforeBlob"]:
                changed_files.add(diff["beforeBlob"]["path"])

        next_token = resp.get("NextToken")
        if not next_token:
            break

    return changed_files


def lambda_handler(event, context):
    detail = event["detail"]
    repo_name = detail["repositoryName"]
    before_commit = detail.get("oldCommitId")
    after_commit = detail["commitId"]

    changed_files = get_changed_paths(repo_name, before_commit, after_commit)
    print(f"Changed files between {before_commit} and {after_commit}: {changed_files}")

    pipelines_to_trigger = set()

    for file_path in changed_files:
        if file_path.startswith(SHARED_LIB_PREFIX):
            # Shared library changed: trigger ALL microservice pipelines
            pipelines_to_trigger.update(SERVICE_PIPELINE_MAP.values())
            break
        for prefix, pipeline_name in SERVICE_PIPELINE_MAP.items():
            if file_path.startswith(prefix):
                pipelines_to_trigger.add(pipeline_name)

    triggered = []
    for pipeline in sorted(pipelines_to_trigger):
        exec_resp = codepipeline.start_pipeline_execution(name=pipeline)
        triggered.append(
            {"pipeline": pipeline, "executionId": exec_resp["pipelineExecutionId"]}
        )
        print(f"Started {pipeline}: {exec_resp['pipelineExecutionId']}")

    return {"triggeredPipelines": triggered}
```

---

## Exercise 6: Golden AMI Factory with EC2 Image Builder, STIG Hardening, and Multi-Account Organization Distribution

### Scenario
Your Security Operations team requires a weekly automated **Golden AMI Factory** that:
1. Starts from the latest Amazon Linux 2023 base AMI.
2. Installs the **CodeDeploy Agent** and **CloudWatch Agent**, applies **STIG hardening**, and validates that both agents are enabled and active.
3. Distributes the resulting encrypted AMI across `us-east-1` and `us-west-2`, shares it with the entire **Production Organizational Unit (`ou-prod-12345678`)**, updates SSM Parameter `/golden-ami/al2023/latest`, and creates a new version of the Production EC2 Launch Template.

---

### Complete Working Solution (CloudFormation Template)
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Enterprise Golden AMI Factory with EC2 Image Builder and Org Distribution

Resources:
  InstallAndVerifyAgentsComponent:
    Type: AWS::ImageBuilder::Component
    Properties:
      Name: Install-CodeDeploy-And-CWAgent
      Platform: Linux
      Version: 1.0.0
      Data: |
        name: InstallAndValidateOpsAgents
        schemaVersion: 1.0
        phases:
          - name: build
            steps:
              - name: InstallAgents
                action: ExecuteBash
                inputs:
                  commands:
                    - dnf update -y
                    - dnf install -y ruby wget amazon-cloudwatch-agent
                    - cd /tmp
                    - wget https://aws-codedeploy-us-east-1.s3.us-east-1.amazonaws.com/latest/install
                    - chmod +x ./install
                    - ./install auto
                    - systemctl enable codedeploy-agent
                    - systemctl enable amazon-cloudwatch-agent
          - name: validate
            steps:
              - name: VerifyServicesActive
                action: ExecuteBash
                inputs:
                  commands:
                    - systemctl is-enabled codedeploy-agent
                    - systemctl is-active codedeploy-agent
                    - rpm -q amazon-cloudwatch-agent

  GoldenAmiRecipe:
    Type: AWS::ImageBuilder::ImageRecipe
    Properties:
      Name: Enterprise-AL2023-Hardened-Recipe
      Version: 1.0.0
      ParentImage: !Sub "arn:aws:imagebuilder:${AWS::Region}:aws:image/amazon-linux-2023-x86/x.x.x"
      BlockDeviceMappings:
        - DeviceName: /dev/xvda
          Ebs:
            VolumeSize: 30
            VolumeType: gp3
            Encrypted: true
            KmsKeyId: !Sub "arn:aws:kms:${AWS::Region}:${AWS::AccountId}:alias/golden-ami-cmk"
      Components:
        - ComponentArn: !Ref InstallAndVerifyAgentsComponent
        - ComponentArn: !Sub "arn:aws:imagebuilder:${AWS::Region}:aws:component/stig-build-linux-medium/x.x.x"
        - ComponentArn: !Sub "arn:aws:imagebuilder:${AWS::Region}:aws:component/reboot-test-linux/x.x.x"

  GoldenAmiInfraConfig:
    Type: AWS::ImageBuilder::InfrastructureConfiguration
    Properties:
      Name: GoldenAmi-PrivateVPC-Infra
      InstanceProfileName: EC2InstanceProfileForImageBuilder
      InstanceTypes:
        - m6i.large
      SubnetId: !ImportValue SharedNetwork-PrivateSubnet1A
      SecurityGroupIds:
        - !ImportValue SharedNetwork-ImageBuilderSG
      TerminateInstanceOnFailure: true
      SnsTopicArn: !ImportValue SecOps-AlertsTopicArn

  GoldenAmiDistributionConfig:
    Type: AWS::ImageBuilder::DistributionConfiguration
    Properties:
      Name: GoldenAmi-OrgWide-MultiRegion-Dist
      Distributions:
        - Region: us-east-1
          AmiDistributionConfiguration:
            Name: "Golden-AL2023-{{ imagebuilder:buildDate }}"
            KmsKeyId: !Sub "arn:aws:kms:us-east-1:${AWS::AccountId}:alias/golden-ami-cmk"
            LaunchPermissionConfiguration:
              OrganizationalUnitArns:
                - arn:aws:organizations::111111111111:ou/o-exampleorgid/ou-prod-12345678
          LaunchTemplateConfigurations:
            - LaunchTemplateId: lt-0123456789abcdef0
              AccountId: "222222222222"
              SetDefaultVersion: true
          SsmParameterConfigurations:
            - ParameterName: /golden-ami/al2023/latest
              AmiAccountId: "222222222222"
              DataType: aws:ec2:image
        - Region: us-west-2
          AmiDistributionConfiguration:
            Name: "Golden-AL2023-{{ imagebuilder:buildDate }}"
            KmsKeyId: !Sub "arn:aws:kms:us-west-2:${AWS::AccountId}:alias/golden-ami-cmk"
            LaunchPermissionConfiguration:
              OrganizationalUnitArns:
                - arn:aws:organizations::111111111111:ou/o-exampleorgid/ou-prod-12345678

  GoldenAmiPipeline:
    Type: AWS::ImageBuilder::ImagePipeline
    Properties:
      Name: Weekly-Golden-AL2023-Pipeline
      ImageRecipeArn: !Ref GoldenAmiRecipe
      InfrastructureConfigurationArn: !Ref GoldenAmiInfraConfig
      DistributionConfigurationArn: !Ref GoldenAmiDistributionConfig
      ImageTestsConfiguration:
        ImageTestsEnabled: true
        TimeoutMinutes: 60
      Schedule:
        ScheduleExpression: "cron(0 2 ? * SUN *)"
        PipelineExecutionStartCondition: EXPRESSION_MATCH_AND_DEPENDENCY_UPDATES_AVAILABLE
      Status: ENABLED
```

### Verification Commands
```bash
# Trigger an on-demand build of the Image Pipeline
aws imagebuilder start-image-pipeline-execution \
  --image-pipeline-arn arn:aws:imagebuilder:us-east-1:111111111111:image-pipeline/weekly-golden-al2023-pipeline

# Verify the latest distributed AMI ID stored in SSM Parameter Store
aws ssm get-parameter --name "/golden-ami/al2023/latest" --query "Parameter.Value" --output text
```
