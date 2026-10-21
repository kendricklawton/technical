# Domain 2: Configuration Management & IaC — Production-Grade Hands-On Exercises (DOP-C02)

---

## Exercise 1: Zero-Downtime Auto Scaling Rolling Update with `CreationPolicy`, `UpdatePolicy`, `cfn-init`, `cfn-signal`, and `cfn-hup`

### Scenario
You manage a production web tier running on an **Auto Scaling Group (`ProdWebASG`)** of 10 EC2 instances behind an Application Load Balancer.
You must write a complete CloudFormation template that:
1. Pulls the latest Golden AMI dynamically from SSM Parameter Store (`/golden-ami/al2023/latest`).
2. Bootstraps Nginx and an application configuration file using **`cfn-init` configSets** (`01_configure_cfn_hup`, `02_install_nginx`).
3. Configures the **`cfn-hup` daemon** to poll every 1 minute for changes to `AWS::CloudFormation::Init` metadata so configuration file changes apply in-place without instance replacement.
4. Enforces a **Zero-Downtime Rolling Update** (`AutoScalingRollingUpdate`) when the Launch Template version changes, keeping at least **8 instances in service** at all times, updating **2 instances per batch**, suspending conflicting ASG scaling processes, and waiting up to **10 minutes (`PT10M`)** per batch for `cfn-signal`.

---

### Complete Working Solution (CloudFormation YAML)
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Zero-Downtime ASG Rolling Update with cfn-init, cfn-signal, and cfn-hup

Parameters:
  GoldenAmiId:
    Type: 'AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>'
    Default: '/golden-ami/al2023/latest'
  AppConfigBanner:
    Type: String
    Default: 'Production Release v3.4.0'
  VpcId:
    Type: AWS::EC2::VPC::Id
  PrivateSubnets:
    Type: List<AWS::EC2::Subnet::Id>
  TargetGroupArn:
    Type: String

Resources:
  WebInstanceRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: ec2.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
      Policies:
        - PolicyName: CloudFormationSignalAndDescribeMetadata
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - cloudformation:SignalResource
                  - cloudformation:DescribeStackResource
                Resource: !Sub 'arn:aws:cloudformation:${AWS::Region}:${AWS::AccountId}:stack/${AWS::StackName}/*'

  WebInstanceProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      Roles:
        - !Ref WebInstanceRole

  ProdWebLaunchTemplate:
    Type: AWS::EC2::LaunchTemplate
    Metadata:
      AWS::CloudFormation::Init:
        configSets:
          bootstrap:
            - 01_configure_cfn_hup
            - 02_install_nginx
        01_configure_cfn_hup:
          files:
            /etc/cfn/cfn-hup.conf:
              content: !Sub |
                [main]
                stack=${AWS::StackId}
                region=${AWS::Region}
                interval=1
                verbose=true
              mode: '000400'
              owner: root
              group: root
            /etc/cfn/hooks.d/cfn-auto-reloader.conf:
              content: !Sub |
                [cfn-auto-reloader-hook]
                triggers=post.update
                path=Resources.ProdWebLaunchTemplate.Metadata.AWS::CloudFormation::Init
                action=/opt/aws/bin/cfn-init -v --stack ${AWS::StackName} --resource ProdWebLaunchTemplate --configsets bootstrap --region ${AWS::Region}
                runas=root
              mode: '000400'
              owner: root
              group: root
          services:
            sysvinit:
              cfn-hup:
                enabled: true
                ensureRunning: true
                files:
                  - /etc/cfn/cfn-hup.conf
                  - /etc/cfn/hooks.d/cfn-auto-reloader.conf
        02_install_nginx:
          packages:
            yum:
              nginx: []
          files:
            /usr/share/nginx/html/index.html:
              content: !Sub |
                <!DOCTYPE html>
                <html><body><h1>${AppConfigBanner}</h1></body></html>
              mode: '000644'
              owner: nginx
              group: nginx
          commands:
            01_test_nginx_config:
              command: "nginx -t"
          services:
            sysvinit:
              nginx:
                enabled: true
                ensureRunning: true
                files:
                  - /usr/share/nginx/html/index.html
    Properties:
      LaunchTemplateData:
        ImageId: !Ref GoldenAmiId
        InstanceType: m6i.large
        IamInstanceProfile:
          Arn: !GetAtt WebInstanceProfile.Arn
        UserData:
          Fn::Base64: !Sub |
            #!/bin/bash -x
            /opt/aws/bin/cfn-init -v \
              --stack ${AWS::StackName} \
              --resource ProdWebLaunchTemplate \
              --configsets bootstrap \
              --region ${AWS::Region}
            INIT_STATUS=$?
            /opt/aws/bin/cfn-signal -e $INIT_STATUS \
              --stack ${AWS::StackName} \
              --resource ProdWebASG \
              --region ${AWS::Region}

  ProdWebASG:
    Type: AWS::AutoScaling::AutoScalingGroup
    CreationPolicy:
      ResourceSignal:
        Count: 10
        Timeout: PT15M
    UpdatePolicy:
      AutoScalingScheduledAction:
        IgnoreUnmodifiedGroupSizeProperties: true
      AutoScalingRollingUpdate:
        MinInstancesInService: 8
        MaxBatchSize: 2
        PauseTime: PT10M
        WaitOnResourceSignals: true
        MinSuccessfulInstancesPercent: 100
        SuspendProcesses:
          - HealthCheck
          - ReplaceUnhealthy
          - AZRebalance
          - AlarmNotification
          - ScheduledActions
    Properties:
      MinSize: '10'
      MaxSize: '12' # Must allow +2 headroom if scaling out before terminating!
      DesiredCapacity: '10'
      VPCZoneIdentifier: !Ref PrivateSubnets
      TargetGroupARNs:
        - !Ref TargetGroupArn
      HealthCheckType: ELB
      HealthCheckGracePeriod: 180
      LaunchTemplate:
        LaunchTemplateId: !Ref ProdWebLaunchTemplate
        Version: !GetAtt ProdWebLaunchTemplate.LatestVersionNumber
```

---

## Exercise 2: Lambda-Backed CloudFormation Custom Resource with Timeout Protection & `PhysicalResourceId` Lifecycle Management

### Scenario
When deleting a CloudFormation stack that contains a versioned S3 Bucket (`AuditArchiveBucket`), stack deletion fails with `The bucket you tried to delete is not empty. You must delete all versions in the bucket.`
You must write a **Lambda-backed CloudFormation Custom Resource (`Custom::S3BucketEmptier`)** that:
1. Safely deletes all objects, object versions, and delete markers from the target S3 bucket when the stack receives a **`Delete`** request.
2. Preserves a deterministic **`PhysicalResourceId`** across `Create` and `Update` events so CloudFormation never falsely triggers a replacement `Delete` during stack updates.
3. Includes a **watchdog timeout timer (`threading.Timer`)** and top-level exception handler that guarantees an HTTP `PUT` (`Status: "SUCCESS" | "FAILED"`) is sent to the S3 pre-signed `ResponseURL`, preventing 1-hour stack hangs.

---

### Complete Working Solution (Python 3.12 Custom Resource Lambda + CloudFormation YAML)
```python
import json
import threading
import urllib.request
import boto3

s3 = boto3.resource("s3")


def send_cfn_response(
    event: dict,
    context,
    response_status: str,
    response_data: dict,
    physical_resource_id: str,
    reason: str | None = None,
) -> None:
    response_url = event["ResponseURL"]
    body = {
        "Status": response_status,
        "Reason": reason
        or f"See CloudWatch Log Stream: {context.log_stream_name}",
        "PhysicalResourceId": physical_resource_id,
        "StackId": event["StackId"],
        "RequestId": event["RequestId"],
        "LogicalResourceId": event["LogicalResourceId"],
        "NoEcho": False,
        "Data": response_data,
    }
    json_body = json.dumps(body).encode("utf-8")
    req = urllib.request.Request(
        url=response_url,
        data=json_body,
        headers={"content-type": "", "content-length": str(len(json_body))},
        method="PUT",
    )
    with urllib.request.urlopen(req, timeout=10) as resp:
        print(f"Sent CFN response {response_status}: HTTP {resp.status}")


def timeout_watchdog(event, context, physical_id):
    print("ERROR: Lambda execution approaching timeout! Sending FAILED to CloudFormation.")
    send_cfn_response(
        event,
        context,
        "FAILED",
        {},
        physical_id,
        reason="Custom Resource Lambda timed out before completing operation.",
    )


def empty_versioned_bucket(bucket_name: str) -> int:
    bucket = s3.Bucket(bucket_name)
    # Delete all object versions AND delete markers in batches
    deleted_count = 0
    try:
        for batch in bucket.object_versions.pages():
            objs = [{"Key": v.key, "VersionId": v.id} for v in batch]
            if objs:
                bucket.delete_objects(Delete={"Objects": objs, "Quiet": True})
                deleted_count += len(objs)
    except s3.meta.client.exceptions.NoSuchBucket:
        print(f"Bucket {bucket_name} already does not exist; skipping cleanly.")
    return deleted_count


def lambda_handler(event, context):
    print("Received Custom Resource Event:", json.dumps(event))
    request_type = event["RequestType"]
    bucket_name = event["ResourceProperties"]["BucketName"]

    # Deterministic PhysicalResourceId tied to the bucket name so Update never triggers replacement Delete!
    physical_id = event.get("PhysicalResourceId") or f"S3BucketEmptier-{bucket_name}"

    # Arm watchdog timer 1.5 seconds before Lambda timeout
    remaining_seconds = max((context.get_remaining_time_in_millis() / 1000.0) - 1.5, 1.0)
    timer = threading.Timer(remaining_seconds, timeout_watchdog, args=[event, context, physical_id])
    timer.start()

    try:
        if request_type == "Create":
            send_cfn_response(
                event,
                context,
                "SUCCESS",
                {"BucketName": bucket_name, "Action": "Registered"},
                physical_id,
            )
        elif request_type == "Update":
            old_bucket = event.get("OldResourceProperties", {}).get("BucketName")
            # Keep the existing physical_id unless the target bucket name itself changed
            updated_physical_id = f"S3BucketEmptier-{bucket_name}"
            send_cfn_response(
                event,
                context,
                "SUCCESS",
                {"BucketName": bucket_name, "PreviousBucket": old_bucket},
                updated_physical_id,
            )
        elif request_type == "Delete":
            deleted_count = empty_versioned_bucket(bucket_name)
            send_cfn_response(
                event,
                context,
                "SUCCESS",
                {"DeletedObjectVersions": deleted_count},
                physical_id,
            )
    except Exception as exc:
        print(f"Exception in Custom Resource handler: {exc}")
        send_cfn_response(
            event,
            context,
            "FAILED",
            {},
            physical_id,
            reason=f"Custom Resource Exception: {str(exc)}",
        )
    finally:
        timer.cancel()
```

```yaml
  AuditArchiveBucket:
    Type: AWS::S3::Bucket
    Properties:
      VersioningConfiguration:
        Status: Enabled

  EmptyAuditBucketOnDelete:
    Type: Custom::S3BucketEmptier
    Properties:
      ServiceToken: !GetAtt S3BucketEmptierLambda.Arn
      ServiceTimeout: 300
      BucketName: !Ref AuditArchiveBucket
```

---

## Exercise 3: Organization-Wide CloudFormation StackSets with Delegated Administrator, Fault Tolerance, and Drift Remediation

### Scenario
Your Security Architecture team operates out of a dedicated **Security Tooling Account (`333333333333`)** registered as a **Delegated Administrator** for CloudFormation StackSets in an AWS Organization (`o-exampleorgid`).
You must deploy an IAM Security Audit Role and GuardDuty Detector configuration across **all 80 accounts** in the `Production` (`ou-prod-11111111`) and `Staging` (`ou-stg-22222222`) Organizational Units across `us-east-1`, `us-west-2`, and `eu-west-1` simultaneously:
* New accounts joining either OU must automatically receive the stack.
* Accounts leaving an OU must have the stack automatically deleted (`RetainStacksOnAccountRemoval: false`).
* Deploy across all 3 regions in **parallel**, up to **25% of accounts at a time**, halting only if **more than 10% of accounts fail**.

---

### Complete Working Solution (CloudFormation StackSet Resource & CLI Commands)
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Service-Managed Organization-Wide Security Baseline StackSet

Resources:
  OrgSecurityBaselineStackSet:
    Type: AWS::CloudFormation::StackSet
    Properties:
      StackSetName: Enterprise-SecOps-AuditBaseline
      PermissionModel: SERVICE_MANAGED
      CallAs: DELEGATED_ADMIN
      Capabilities:
        - CAPABILITY_NAMED_IAM
      AutoDeployment:
        Enabled: true
        RetainStacksOnAccountRemoval: false
      ManagedExecution:
        Active: true # Allows StackSets to queue concurrent operations safely
      OperationPreferences:
        RegionConcurrencyType: PARALLEL
        MaxConcurrentPercentage: 25
        FailureTolerancePercentage: 10
      StackInstancesGroup:
        - DeploymentTargets:
            OrganizationalUnitIds:
              - ou-prod-11111111
              - ou-stg-22222222
          Regions:
            - us-east-1
            - us-west-2
            - eu-west-1
      TemplateBody: |
        AWSTemplateFormatVersion: '2010-09-09'
        Resources:
          SecOpsCrossAccountAuditRole:
            Type: AWS::IAM::Role
            Properties:
              RoleName: SecOpsCentralAuditRole
              AssumeRolePolicyDocument:
                Version: '2012-10-17'
                Statement:
                  - Effect: Allow
                    Principal:
                      AWS: arn:aws:iam::333333333333:root
                    Action: sts:AssumeRole
              ManagedPolicyArns:
                - arn:aws:iam::aws:policy/SecurityAudit
```

```bash
# 1. Register Security Tooling Account (333333333333) as Delegated Admin (run from Management Account)
aws organizations register-delegated-administrator \
  --account-id 333333333333 \
  --service-principal member.org.stacksets.cloudformation.amazonaws.com

# 2. Run StackSet-wide Drift Detection across all 80 accounts and 3 regions (run from Delegated Admin)
aws cloudformation detect-stack-set-drift \
  --stack-set-name Enterprise-SecOps-AuditBaseline \
  --call-as DELEGATED_ADMIN \
  --operation-preferences RegionConcurrencyType=PARALLEL,MaxConcurrentPercentage=50,FailureTolerancePercentage=100

# 3. List any Stack Instances that have drifted (MODIFIED or DELETED)
aws cloudformation list-stack-instances \
  --stack-set-name Enterprise-SecOps-AuditBaseline \
  --call-as DELEGATED_ADMIN \
  --filters Name=DRIFT_STATUS,Values=DRIFTED
```

---

## Exercise 4: Enterprise Patch Automation with SSM Patch Baselines, Patch Groups, Maintenance Windows, and Pre/Post Lifecycle Hooks

### Scenario
You must configure automated OS patching for Amazon Linux 2023 instances in Production:
1. Create a Custom **SSM Patch Baseline** (`Prod-AL2023-Baseline`) that auto-approves `Security` and `Bugfix` patches rated `Critical` or `Important` **7 days after release**, blocks `kernel-debug*`, and assigns `CRITICAL` compliance severity.
2. Register the Patch Baseline with the case-sensitive Patch Group **`Prod-AL2023`**.
3. Create an **SSM Maintenance Window** (`cron(0 3 ? * SUN *)`) targeting instances tagged `Patch Group = Prod-AL2023` and executing `AWS-RunPatchBaseline` (`Operation: Install`, `RebootOption: RebootIfNeeded`) with pre-install and post-install hook documents across `20%` concurrency with `5%` max errors.

---

### Complete Working Solution (CloudFormation Template)
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Enterprise SSM Patch Baseline, Patch Group, and Maintenance Window with Hooks

Resources:
  PrePatchDrainHookDocument:
    Type: AWS::SSM::Document
    Properties:
      Name: Prod-PrePatch-HealthCheckAndDrain
      DocumentType: Command
      DocumentFormat: YAML
      Content:
        schemaVersion: '2.2'
        description: Pre-patch hook to verify disk space before yum update
        mainSteps:
          - action: aws:runShellScript
            name: VerifyDiskFreeSpace
            inputs:
              runCommand:
                - df -h /
                - AVAIL_KB=$(df --output=avail / | tail -n 1)
                - if [ "$AVAIL_KB" -lt 2097152 ]; then echo "Insufficient disk space (<2GB)!" >&2; exit 1; fi

  PostPatchVerifyHookDocument:
    Type: AWS::SSM::Document
    Properties:
      Name: Prod-PostPatch-ValidateServices
      DocumentType: Command
      DocumentFormat: YAML
      Content:
        schemaVersion: '2.2'
        description: Post-patch hook to verify nginx and codedeploy-agent after reboot
        mainSteps:
          - action: aws:runShellScript
            name: VerifyDaemonsActive
            inputs:
              runCommand:
                - systemctl is-active nginx
                - systemctl is-active codedeploy-agent

  ProdAl2023PatchBaseline:
    Type: AWS::SSM::PatchBaseline
    Properties:
      Name: Prod-AL2023-Baseline
      OperatingSystem: AMAZON_LINUX_2023
      ApprovedPatchesComplianceLevel: CRITICAL
      RejectedPatches:
        - "kernel-debug*"
      RejectedPatchesAction: BLOCK
      PatchGroups:
        - "Prod-AL2023" # Matched against exact EC2 Tag Key: "Patch Group"
      ApprovalRules:
        PatchRules:
          - ApproveAfterDays: 7
            ComplianceLevel: CRITICAL
            EnableNonSecurity: false
            PatchFilterGroup:
              PatchFilters:
                - Key: PRODUCT
                  Values: ["AmazonLinux2023"]
                - Key: CLASSIFICATION
                  Values: ["Security", "Bugfix"]
                - Key: SEVERITY
                  Values: ["Critical", "Important"]

  ProdPatchMaintenanceWindow:
    Type: AWS::SSM::MaintenanceWindow
    Properties:
      Name: Prod-Sunday-PatchWindow
      Schedule: "cron(0 3 ? * SUN *)"
      Duration: 3
      Cutoff: 1
      AllowUnassociatedTargets: false

  ProdPatchWindowTarget:
    Type: AWS::SSM::MaintenanceWindowTarget
    Properties:
      WindowId: !Ref ProdPatchMaintenanceWindow
      ResourceType: INSTANCE
      Name: ProdAL2023Instances
      Targets:
        - Key: "tag:Patch Group" # Case-sensitive!
          Values:
            - "Prod-AL2023"

  ProdPatchWindowInstallTask:
    Type: AWS::SSM::MaintenanceWindowTask
    Properties:
      WindowId: !Ref ProdPatchMaintenanceWindow
      TaskArn: AWS-RunPatchBaseline
      TaskType: RUN_COMMAND
      Priority: 1
      MaxConcurrency: "20%"
      MaxErrors: "5%"
      ServiceRoleArn: !Sub "arn:aws:iam::${AWS::AccountId}:role/aws-service-role/ssm.amazonaws.com/AWSServiceRoleForAmazonSSM"
      Targets:
        - Key: WindowTargetIds
          Values:
            - !Ref ProdPatchWindowTarget
      TaskInvocationParameters:
        MaintenanceWindowRunCommandParameters:
          Parameters:
            Operation: ["Install"]
            RebootOption: ["RebootIfNeeded"]
            PreInstallHookDocName: [!Ref PrePatchDrainHookDocument]
            PostInstallHookDocName: [!Ref PostPatchVerifyHookDocument]
```

---

## Exercise 5: CloudFormation Guard (`cfn-guard`) & CDK Aspects for Preventative IaC Compliance in CI/CD

### Scenario
Your Security Engineering team wants a two-layer preventative control for Infrastructure as Code:
1. **Layer 1 (CDK Synthesis Time):** A TypeScript **CDK `IAspect`** that inspects every construct in the CDK application, automatically enforces `DeletionPolicy: Retain` and `UpdateReplacePolicy: Retain` on all `AWS::RDS::DBInstance` and `AWS::DynamoDB::Table` resources, and throws a synthesis error (`Annotations.of(node).addError(...)`) if any `AWS::RDS::DBInstance` has `storageEncrypted !== true` or `publiclyAccessible === true`.
2. **Layer 2 (CI/CD Pre-Deployment Gate):** A **`cfn-guard`** policy rule file validated against the synthesized CloudFormation template in CodeBuild.

---

### Complete Working Solution

#### 1. TypeScript CDK Aspect (`lib/compliance-aspect.ts`)
```typescript
import { IAspect, Annotations, CfnResource, CfnDeletionPolicy } from 'aws-cdk-lib';
import { IConstruct } from 'constructs';
import { CfnDBInstance } from 'aws-cdk-lib/aws-rds';
import { CfnTable } from 'aws-cdk-lib/aws-dynamodb';

export class DatabaseProtectionAndSecurityAspect implements IAspect {
  public visit(node: IConstruct): void {
    if (node instanceof CfnDBInstance) {
      // 1. Preventative security checks (fails cdk synth on violation)
      if (node.storageEncrypted !== true) {
        Annotations.of(node).addError(
          `RDS DBInstance ${node.logicalId} MUST have storageEncrypted: true.`
        );
      }
      if (node.publiclyAccessible === true) {
        Annotations.of(node).addError(
          `RDS DBInstance ${node.logicalId} MUST NOT set publiclyAccessible: true.`
        );
      }

      // 2. Automatically enforce Retain on Delete and Update Replacement
      node.cfnOptions.deletionPolicy = CfnDeletionPolicy.RETAIN;
      node.cfnOptions.updateReplacePolicy = CfnDeletionPolicy.RETAIN;
    }

    if (node instanceof CfnTable) {
      node.cfnOptions.deletionPolicy = CfnDeletionPolicy.RETAIN;
      node.cfnOptions.updateReplacePolicy = CfnDeletionPolicy.RETAIN;
    }
  }
}
```

#### 2. CloudFormation Guard Policy (`policy/rds_and_dynamodb_compliance.guard`)
```ruby
let rds_instances = Resources.*[ Type == 'AWS::RDS::DBInstance' ]
let ddb_tables = Resources.*[ Type == 'AWS::DynamoDB::Table' ]

rule rds_must_be_encrypted_and_private when %rds_instances !empty {
    %rds_instances.Properties.StorageEncrypted == true
        <<VIOLATION: AWS::RDS::DBInstance must have StorageEncrypted set to true.>>

    %rds_instances.Properties.PubliclyAccessible !exists or
    %rds_instances.Properties.PubliclyAccessible == false
        <<VIOLATION: AWS::RDS::DBInstance must not be publicly accessible.>>

    %rds_instances.DeletionPolicy == 'Retain' or
    %rds_instances.DeletionPolicy == 'Snapshot'
        <<VIOLATION: AWS::RDS::DBInstance must declare DeletionPolicy: Retain or Snapshot.>>

    %rds_instances.UpdateReplacePolicy == 'Retain' or
    %rds_instances.UpdateReplacePolicy == 'Snapshot'
        <<VIOLATION: AWS::RDS::DBInstance must declare UpdateReplacePolicy: Retain or Snapshot.>>
}

rule dynamodb_must_retain_and_enable_pitr when %ddb_tables !empty {
    %ddb_tables.Properties.PointInTimeRecoverySpecification.PointInTimeRecoveryEnabled == true
        <<VIOLATION: AWS::DynamoDB::Table must enable Point-In-Time Recovery (PITR).>>

    %ddb_tables.DeletionPolicy == 'Retain'
    %ddb_tables.UpdateReplacePolicy == 'Retain'
}
```

#### 3. CodeBuild Validation Commands
```bash
npx cdk synth > synthesized-template.yaml
cfn-guard validate \
  --data synthesized-template.yaml \
  --rules policy/rds_and_dynamodb_compliance.guard \
  --show-summary fail
```

---

## Exercise 6: Elastic Beanstalk Immutable Deployment with `.ebextensions` Leader-Only Database Migrations and `.platform` Hooks

### Scenario
You are deploying a Python FastAPI application to an **AWS Elastic Beanstalk** environment (`Amazon Linux 2023`) connected to an external Amazon Aurora PostgreSQL cluster.
You must configure the application source bundle so that:
1. Deployments use the **Immutable** deployment policy with Enhanced Health Reporting.
2. OS-level `postgresql15` client libraries are installed **before** the application runs.
3. Database schema migrations (`alembic upgrade head`) execute **after** the application bundle is unpacked to `/var/app/staging`, **before** traffic hits the application, and run on **only a single leader instance** (`leader_only: true`).
4. A post-deploy `.platform/hooks/postdeploy/` script verifies the local HTTP health endpoint and emits a custom CloudWatch metric.

---

### Complete Working Solution

#### 1. `.ebextensions/01_immutable_and_migrations.config`
```yaml
packages:
  dnf:
    postgresql15: []
    jq: []

option_settings:
  aws:elasticbeanstalk:command:
    DeploymentPolicy: Immutable
    Timeout: 900
    HealthCheckSuccessThreshold: Ok
    IgnoreHealthCheck: false
  aws:elasticbeanstalk:healthreporting:system:
    SystemType: enhanced
  aws:autoscaling:updatepolicy:rollingupdate:
    RollingUpdateEnabled: true
    RollingUpdateType: Immutable

commands:
  01_configure_os_limits:
    command: "sysctl -w net.ipv4.ip_local_port_range='1024 65535'"

container_commands:
  01_run_alembic_migrations_leader_only:
    command: |
      source /var/app/venv/*/bin/activate
      export DATABASE_URL=$(aws secretsmanager get-secret-value --secret-id prod/aurora/fastapi-url --query SecretString --output text)
      alembic upgrade head
    leader_only: true
```

#### 2. `.platform/hooks/postdeploy/01_verify_and_publish_metric.sh` (`chmod +x`)
```bash
#!/usr/bin/env bash
# Must be committed with executable permissions: git update-index --chmod=+x .platform/hooks/postdeploy/01_verify_and_publish_metric.sh
set -euo pipefail

INSTANCE_ID=$(ec2-metadata -i | awk '{print $2}')
REGION=$(ec2-metadata -z | awk '{print $2}' | sed 's/[a-z]$//')

HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:8000/healthz)
if [ "$HTTP_CODE" != "200" ]; then
  echo "Postdeploy health verification failed with HTTP $HTTP_CODE" >&2
  exit 1
fi

aws cloudwatch put-metric-data \
  --region "$REGION" \
  --namespace "Custom/ElasticBeanstalk" \
  --metric-name "PostDeployHealthSuccess" \
  --dimensions InstanceId="$INSTANCE_ID" \
  --value 1 \
  --unit Count

echo "Postdeploy verification succeeded on instance $INSTANCE_ID"
```
