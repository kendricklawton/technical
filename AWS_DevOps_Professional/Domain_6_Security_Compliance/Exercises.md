# Domain 6: Security and Compliance — Production Hands-On Exercises (DOP-C02)

These 6 production-grade exercises cover every high-value security, cryptography, secret rotation, and compliance governance scenario tested on the AWS Certified DevOps Engineer – Professional (DOP-C02) exam, complete with working JSON, YAML, Python, and `cfn-guard` solutions.

---

## Exercise 1: Delegated IAM Administration with Permission Boundaries & Self-Escalation Prevention

### Scenario
Your platform engineering team wants to allow application developers (`arn:aws:iam::111122223333:role/DeveloperCicdRole`) to create IAM Roles (`arn:aws:iam::111122223333:role/svc-*`) and custom customer-managed IAM policies (`arn:aws:iam::111122223333:policy/svc-*`) for their microservices without opening tickets.

However, you must prevent privilege escalation:
1. Write the **Permissions Boundary Policy (`WorkloadPermissionsBoundary`)** that caps any `svc-*` role so it can ONLY use `s3:*`, `dynamodb:*`, `sqs:*`, `sns:*`, `logs:*`, and `xray:*` in `us-east-1`, and explicitly denies `iam:*`, `organizations:*`, and `kms:ScheduleKeyDeletion`.
2. Write the **Developer Identity Policy (`DelegatedIamAdminPolicy`)** that allows creating `svc-*` roles and attaching `svc-*` policies **ONLY IF** `WorkloadPermissionsBoundary` is attached, while blocking any attempt to delete/modify the boundary policy or remove the boundary from any role.

---

### Complete Working Solution

#### 1. Permissions Boundary Managed Policy (`WorkloadPermissionsBoundary.json`)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "MaximumAllowedApplicationServicesInApprovedRegion",
      "Effect": "Allow",
      "Action": [
        "s3:*",
        "dynamodb:*",
        "sqs:*",
        "sns:*",
        "logs:*",
        "xray:*",
        "kms:Decrypt",
        "kms:GenerateDataKey*"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "us-east-1"
        }
      }
    },
    {
      "Sid": "ExplicitlyDenyPrivilegeEscalationAndKeyDeletion",
      "Effect": "Deny",
      "Action": [
        "iam:*",
        "organizations:*",
        "account:*",
        "kms:ScheduleKeyDeletion",
        "kms:PutKeyPolicy",
        "s3:PutBucketPolicy",
        "s3:PutAccountPublicAccessBlock"
      ],
      "Resource": "*"
    }
  ]
}
```

#### 2. Developer Delegated IAM Creation Policy (`DelegatedIamAdminPolicy.json`)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "RequireBoundaryOnRoleCreationAndPolicyAttachment",
      "Effect": "Allow",
      "Action": [
        "iam:CreateRole",
        "iam:AttachRolePolicy",
        "iam:DetachRolePolicy",
        "iam:PutRolePolicy",
        "iam:DeleteRolePolicy",
        "iam:PutRolePermissionsBoundary"
      ],
      "Resource": "arn:aws:iam::111122223333:role/svc-*",
      "Condition": {
        "StringEquals": {
          "iam:PermissionsBoundary": "arn:aws:iam::111122223333:policy/WorkloadPermissionsBoundary"
        }
      }
    },
    {
      "Sid": "AllowCreatingScopedCustomerManagedPolicies",
      "Effect": "Allow",
      "Action": [
        "iam:CreatePolicy",
        "iam:CreatePolicyVersion",
        "iam:DeletePolicy",
        "iam:DeletePolicyVersion"
      ],
      "Resource": "arn:aws:iam::111122223333:policy/svc-*"
    },
    {
      "Sid": "ProtectBoundaryPolicyAndAttachmentFromTampering",
      "Effect": "Deny",
      "Action": [
        "iam:DeleteRolePermissionsBoundary",
        "iam:CreatePolicyVersion",
        "iam:DeletePolicy",
        "iam:DeletePolicyVersion",
        "iam:SetDefaultPolicyVersion"
      ],
      "Resource": [
        "arn:aws:iam::111122223333:policy/WorkloadPermissionsBoundary",
        "arn:aws:iam::111122223333:role/svc-*"
      ]
    },
    {
      "Sid": "AllowPassRoleOnlyToComputeServices",
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": "arn:aws:iam::111122223333:role/svc-*",
      "Condition": {
        "StringEquals": {
          "iam:PassedToService": [
            "lambda.amazonaws.com",
            "ecs-tasks.amazonaws.com"
          ]
        }
      }
    }
  ]
}
```

---

## Exercise 2: Enterprise AWS Organizations SCP Suite

### Scenario
Write a production **AWS Organizations Service Control Policy (SCP)** (`EnterpriseGuardrailsSCP.json`, staying well under the 5,120-byte limit) attached to the `Workloads-OU` that enforces:
1. **Region Lock (`us-east-1` and `us-west-2`)** while properly exempting global services (`iam`, `sts`, `route53`, `cloudfront`, `support`, `health`, `organizations`, `budgets`, `wafv2`) so multi-account operations never break.
2. **Mandatory IMDSv2** (`ec2:MetadataHttpTokens == "required"` and `ec2:MetadataHttpPutResponseHopLimit <= 2`) at launch time AND blocking any runtime API call made using IMDSv1 credentials (`ec2:RoleDelivery < 2.0`).
3. **Root User Lockdown** (`aws:PrincipalArn` matching `arn:aws:iam::*:root`).
4. **Security Tool Tamper Protection** for CloudTrail, GuardDuty, AWS Config, and Security Hub (exempting only `arn:aws:iam::*:role/CentralSecOpsAdminRole`).

---

### Complete Working Solution (`EnterpriseGuardrailsSCP.json`)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyAllOutsideApprovedRegionsExceptGlobal",
      "Effect": "Deny",
      "NotAction": [
        "a4b:*",
        "acm:*",
        "budgets:*",
        "ce:*",
        "cloudfront:*",
        "config:*",
        "cur:*",
        "directconnect:*",
        "ec2:DescribeRegions",
        "fms:*",
        "globalaccelerator:*",
        "health:*",
        "iam:*",
        "kms:*",
        "networkmanager:*",
        "organizations:*",
        "pricing:*",
        "route53:*",
        "route53domains:*",
        "s3:GetAccountPublicAccessBlock",
        "s3:ListAllMyBuckets",
        "s3:PutAccountPublicAccessBlock",
        "shield:*",
        "sts:*",
        "support:*",
        "trustedadvisor:*",
        "waf:*",
        "waf-regional:*",
        "wafv2:*"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": [
            "us-east-1",
            "us-west-2"
          ]
        }
      }
    },
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
      "Sid": "EnforceIMDSv2HopLimitCeiling",
      "Effect": "Deny",
      "Action": [
        "ec2:RunInstances",
        "ec2:ModifyInstanceMetadataOptions"
      ],
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "NumericGreaterThan": {
          "ec2:MetadataHttpPutResponseHopLimit": "2"
        }
      }
    },
    {
      "Sid": "DenyRuntimeCallsFromIMDSv1",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "NumericLessThan": {
          "ec2:RoleDelivery": "2.0"
        }
      }
    },
    {
      "Sid": "DenyAllRootUserActions",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringLike": {
          "aws:PrincipalArn": "arn:aws:iam::*:root"
        }
      }
    },
    {
      "Sid": "ProtectSecurityServicesFromTampering",
      "Effect": "Deny",
      "Action": [
        "cloudtrail:DeleteTrail",
        "cloudtrail:StopLogging",
        "cloudtrail:UpdateTrail",
        "config:DeleteConfigurationRecorder",
        "config:StopConfigurationRecorder",
        "guardduty:DeleteDetector",
        "guardduty:DisassociateFromAdministratorAccount",
        "guardduty:StopMonitoringMembers",
        "securityhub:DisableSecurityHub",
        "securityhub:DisassociateFromAdministratorAccount"
      ],
      "Resource": "*",
      "Condition": {
        "ArnNotLike": {
          "aws:PrincipalArn": "arn:aws:iam::*:role/CentralSecOpsAdminRole"
        }
      }
    }
  ]
}
```

---

## Exercise 3: Cross-Account Encrypted AMI & EBS Auto Scaling Deployment with KMS Customer Managed Key Grants

### Scenario
A Central Golden AMI Factory pipeline in `Account A (111111111111)` builds encrypted AMIs using a KMS Customer Managed Key (`arn:aws:kms:us-east-1:111111111111:key/cmk-golden-ami-factory`).
An EC2 Auto Scaling Group in `Account B (222222222222)` needs to launch instances directly from `Account A`'s encrypted AMI without copying the AMI first.

1. Write the exact **KMS Key Policy** in `Account A (111111111111)` that allows `Account B` and `Account B`'s **Auto Scaling Service-Linked Role** (`AWSServiceRoleForAutoScaling`) to decrypt the AMI and create KMS grants for EBS volumes.
2. Write the AWS CLI commands run in `Account B (222222222222)` to explicitly create the KMS Grant for `AWSServiceRoleForAutoScaling` if `Account A`'s KMS Key Policy only delegates to `arn:aws:iam::222222222222:root`.

---

### Complete Working Solution

#### 1. Account A (`111111111111`) Golden AMI KMS Key Policy (`cmk-golden-ami-policy.json`)
```json
{
  "Version": "2012-10-17",
  "Id": "cross-account-golden-ami-cmk-policy",
  "Statement": [
    {
      "Sid": "EnableIAMUserPermissionsInAccountA",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111111111111:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "AllowAccountBAndItsAutoScalingSLRToUseKey",
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::222222222222:root",
          "arn:aws:iam::222222222222:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling"
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
      "Sid": "AllowAccountBAutoScalingSLRToCreateGrantsForEBS",
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::222222222222:root",
          "arn:aws:iam::222222222222:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling"
        ]
      },
      "Action": "kms:CreateGrant",
      "Resource": "*",
      "Condition": {
        "Bool": {
          "kms:GrantIsForAWSResource": "true"
        }
      }
    }
  ]
}
```

#### 2. CLI Commands to Share the AMI (in Account A) and Create KMS Grant (in Account B)
```bash
# In Account A (111111111111): Share the CMK-encrypted AMI with Account B
aws ec2 modify-image-attribute \
  --image-id ami-0abc12345def67890 \
  --launch-permission "Add=[{UserId=222222222222}]"

# In Account B (222222222222): Create a persistent KMS Grant for AWSServiceRoleForAutoScaling
aws kms create-grant \
  --key-id arn:aws:kms:us-east-1:111111111111:key/cmk-golden-ami-factory \
  --grantee-principal arn:aws:iam::222222222222:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling \
  --operations "Encrypt" "Decrypt" "ReEncryptFrom" "ReEncryptTo" "GenerateDataKey" "GenerateDataKeyWithoutPlaintext" "DescribeKey" "CreateGrant"
```

---

## Exercise 4: Zero-Downtime Secrets Manager Alternating-Users Rotation for Amazon RDS

### Scenario
Your mission-critical payment service maintains long-lived PostgreSQL connection pools to an Amazon RDS PostgreSQL database. Rotating the database password using the Single-User strategy causes brief authentication failures while ECS tasks refresh their cached secrets.

Implement a custom **Python Rotation Lambda Handler** (`lambda_function.py`) that executes the **4-step Alternating-Users (`app_user_a` / `app_user_b`) Rotation Lifecycle** (`createSecret`, `setSecret`, `testSecret`, `finishSecret`) using a separate Master Secret (`masterarn`) so active connections on the current user experience **zero downtime**.

---

### Complete Working Solution: Python Alternating-Users Rotation Lambda (`lambda_function.py`)

```python
import json
import logging
import boto3
import psycopg2

logger = logging.getLogger()
logger.setLevel(logging.INFO)

sm_client = boto3.client("secretsmanager")


def lambda_handler(event, context):
    arn = event["SecretId"]
    token = event["ClientRequestToken"]
    step = event["Step"]

    metadata = sm_client.describe_secret(SecretId=arn)
    versions = metadata.get("VersionIdsToStages", {})
    if token not in versions:
        raise ValueError(f"Secret version {token} has no stage for rotation of {arn}.")
    if "AWSCURRENT" in versions[token]:
        logger.info(f"Secret version {token} is already AWSCURRENT for {arn}.")
        return
    if "AWSPENDING" not in versions[token]:
        raise ValueError(f"Secret version {token} not set as AWSPENDING for {arn}.")

    if step == "createSecret":
        create_secret(arn, token)
    elif step == "setSecret":
        set_secret(arn, token)
    elif step == "testSecret":
        test_secret(arn, token)
    elif step == "finishSecret":
        finish_secret(arn, token)
    else:
        raise ValueError(f"Invalid step parameter: {step}")


def get_secret_dict(arn, stage, token=None):
    params = {"SecretId": arn, "VersionStage": stage}
    if token:
        params["VersionId"] = token
    resp = sm_client.get_secret_value(**params)
    return json.loads(resp["SecretString"])


def create_secret(arn, token):
    """Step 1: Generate new password and alternate username (app_user_a <-> app_user_b)."""
    current_dict = get_secret_dict(arn, "AWSCURRENT")
    try:
        sm_client.get_secret_value(SecretId=arn, VersionId=token, VersionStage="AWSPENDING")
        logger.info("createSecret: AWSPENDING version already exists.")
        return
    except sm_client.exceptions.ResourceNotFoundException:
        pass

    # Alternate between app_user_a and app_user_b for zero downtime
    current_user = current_dict["username"]
    next_user = "app_user_b" if current_user == "app_user_a" else "app_user_a"

    passwd_resp = sm_client.get_random_password(
        PasswordLength=32, ExcludeCharacters="/@\"'\\"
    )
    pending_dict = dict(current_dict)
    pending_dict["username"] = next_user
    pending_dict["password"] = passwd_resp["RandomPassword"]

    sm_client.put_secret_value(
        SecretId=arn,
        ClientRequestToken=token,
        SecretString=json.dumps(pending_dict),
        VersionStages=["AWSPENDING"],
    )
    logger.info(f"createSecret: Created AWSPENDING secret for alternating user {next_user}.")


def set_secret(arn, token):
    """Step 2: Connect using Master Secret and update password of the AWSPENDING alternating user."""
    pending_dict = get_secret_dict(arn, "AWSPENDING", token)
    master_dict = get_secret_dict(pending_dict["masterarn"], "AWSCURRENT")

    conn = psycopg2.connect(
        host=master_dict["host"],
        port=master_dict.get("port", 5432),
        dbname=master_dict["dbname"],
        user=master_dict["username"],
        password=master_dict["password"],
        connect_timeout=5,
    )
    conn.autocommit = True
    try:
        with conn.cursor() as cur:
            cur.execute(
                psycopg2.sql.SQL("ALTER USER {} WITH PASSWORD %s").format(
                    psycopg2.sql.Identifier(pending_dict["username"])
                ),
                (pending_dict["password"],),
            )
    finally:
        conn.close()
    logger.info(f"setSecret: Updated DB password for {pending_dict['username']}.")


def test_secret(arn, token):
    """Step 3: Verify the AWSPENDING user and new password can authenticate and query the DB."""
    pending_dict = get_secret_dict(arn, "AWSPENDING", token)
    conn = psycopg2.connect(
        host=pending_dict["host"],
        port=pending_dict.get("port", 5432),
        dbname=pending_dict["dbname"],
        user=pending_dict["username"],
        password=pending_dict["password"],
        connect_timeout=5,
    )
    try:
        with conn.cursor() as cur:
            cur.execute("SELECT 1;")
    finally:
        conn.close()
    logger.info("testSecret: Successfully authenticated with AWSPENDING credentials.")


def finish_secret(arn, token):
    """Step 4: Promote AWSPENDING to AWSCURRENT and demote old AWSCURRENT to AWSPREVIOUS."""
    metadata = sm_client.describe_secret(SecretId=arn)
    current_version = None
    for version_id, stages in metadata["VersionIdsToStages"].items():
        if "AWSCURRENT" in stages:
            if version_id == token:
                return
            current_version = version_id
            break

    sm_client.update_secret_version_stage(
        SecretId=arn,
        VersionStage="AWSCURRENT",
        MoveToVersionId=token,
        RemoveFromVersionId=current_version,
    )
    logger.info(f"finishSecret: Promoted version {token} to AWSCURRENT.")
```

---

## Exercise 5: Organization-Wide AWS Config Conformance Pack & Proactive `cfn-guard` Custom Policy Rules

### Scenario
Your Compliance Engineering team wants a single **AWS Config Conformance Pack YAML** (`org-security-baseline-pack.yaml`) deployed across all member accounts via `PutOrganizationConformancePack` that includes:
1. A managed rule (`s3-bucket-level-public-access-prohibited`) with automatic SSM remediation (`AWSConfigRemediation-ConfigureS3BucketPublicAccessBlock`).
2. A **Custom Policy Rule (`GUARD` / `cfn-guard`)** running in **BOTH Detective (`CONFIGURATION_CHANGES`) and Proactive (`PROACTIVE`) evaluation modes** that enforces:
   - Every `AWS::RDS::DBInstance` must have `StorageEncrypted == true`, `MultiAZ == true`, `PubliclyAccessible == false`, and `BackupRetentionPeriod >= 7`.

---

### Complete Working Solution: Conformance Pack Template (`org-security-baseline-pack.yaml`)

```yaml
Parameters:
  RemediationRoleArn:
    Type: String
    Default: arn:aws:iam::111122223333:role/AWSConfigS3RemediationRole

Resources:
  S3BlockPublicAccessRule:
    Type: AWS::Config::ConfigRule
    Properties:
      ConfigRuleName: org-s3-bucket-level-public-access-prohibited
      Scope:
        ComplianceResourceTypes:
          - AWS::S3::Bucket
      Source:
        Owner: AWS
        SourceIdentifier: S3_BUCKET_LEVEL_PUBLIC_ACCESS_PROHIBITED

  S3BlockPublicAccessAutoRemediation:
    Type: AWS::Config::RemediationConfiguration
    DependsOn: S3BlockPublicAccessRule
    Properties:
      ConfigRuleName: !Ref S3BlockPublicAccessRule
      TargetType: SSM_DOCUMENT
      TargetId: AWSConfigRemediation-ConfigureS3BucketPublicAccessBlock
      TargetVersion: '1'
      Automatic: true
      MaximumAutomaticAttempts: 5
      RetryAttemptSeconds: 60
      Parameters:
        BucketName:
          ResourceValue:
            Value: RESOURCE_ID
        AutomationAssumeRole:
          StaticValue:
            Values:
              - !Ref RemediationRoleArn

  RdsHardeningCustomGuardRule:
    Type: AWS::Config::ConfigRule
    Properties:
      ConfigRuleName: org-rds-proactive-and-detective-guard
      Scope:
        ComplianceResourceTypes:
          - AWS::RDS::DBInstance
      EvaluationModes:
        - Mode: DETECTIVE
        - Mode: PROACTIVE
      Source:
        Owner: CUSTOM_POLICY
        SourceDetails:
          - EventSource: aws.config
            MessageType: ConfigurationItemChangeNotification
        CustomPolicyDetails:
          EnableDebugLogDelivery: true
          PolicyRuntime: guard-2.x.x
          PolicyText: |
            rule rds_instance_hardening when
                resourceType == "AWS::RDS::DBInstance" {
                configuration.storageEncrypted == true
                configuration.multiAZ == true
                configuration.publiclyAccessible == false
                configuration.backupRetentionPeriod >= 7
            }
```

---

## Exercise 6: Self-Service Governance with AWS Service Catalog Launch Constraints, TagOptions, and Firewall Manager

### Scenario
Build a CloudFormation governance template that accomplishes two goals:
1. Configures an **AWS Service Catalog Portfolio**, an approved **Encrypted S3 Data Lake Product**, **TagOptions** (`CostCenter=DataPlatform`, `DataClassification=Confidential`), and a **Service Catalog Launch Constraint** (`AWS::ServiceCatalog::LaunchRoleConstraint`) so Data Scientists can provision compliant S3 buckets without having `s3:CreateBucket` or `kms:*` permissions in their own IAM roles.
2. Configures an **AWS Firewall Manager WAFv2 Organization Policy** (`AWS::FMS::Policy`) that automatically attaches a baseline Web ACL (with `AWSManagedRulesCommonRuleSet` and `AWSManagedRulesKnownBadInputsRuleSet`) to every Application Load Balancer across all accounts in `ou-prod-12345678` with `RemediationEnabled: true`.

---

### Complete Working Solution: CloudFormation Template (`service-catalog-and-fms.yaml`)

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'AWS Service Catalog Launch Constraint + TagOptions and AWS Firewall Manager Org-Wide WAF Policy'

Resources:
  DataPlatformPortfolio:
    Type: AWS::ServiceCatalog::Portfolio
    Properties:
      DisplayName: Data-Platform-Self-Service-Portfolio
      ProviderName: PlatformEngineering
      Description: Approved self-service data products with Launch Constraints

  ApprovedS3Product:
    Type: AWS::ServiceCatalog::CloudFormationProduct
    Properties:
      Name: Compliant-KMS-DataLake-Bucket
      Owner: PlatformEngineering
      ProvisioningArtifactParameters:
        - Name: v1.0
          Description: Encrypted S3 Bucket with Object Lock and Block Public Access
          Info:
            LoadTemplateFromURL: https://s3.amazonaws.com/org-catalog-templates/s3-datalake-v1.yaml

  PortfolioProductAssociation:
    Type: AWS::ServiceCatalog::PortfolioProductAssociation
    Properties:
      PortfolioId: !Ref DataPlatformPortfolio
      ProductId: !Ref ApprovedS3Product

  ServiceCatalogLaunchRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: ServiceCatalog-S3DataLake-LaunchRole
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: servicecatalog.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AmazonS3FullAccess
        - arn:aws:iam::aws:policy/AWSKeyManagementServicePowerUser
      Policies:
        - PolicyName: AllowCloudFormationStackOps
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - cloudformation:CreateStack
                  - cloudformation:DeleteStack
                  - cloudformation:DescribeStacks
                  - cloudformation:DescribeStackEvents
                  - cloudformation:GetTemplateSummary
                  - cloudformation:SetStackPolicy
                Resource: '*'

  ProductLaunchRoleConstraint:
    Type: AWS::ServiceCatalog::LaunchRoleConstraint
    DependsOn: PortfolioProductAssociation
    Properties:
      PortfolioId: !Ref DataPlatformPortfolio
      ProductId: !Ref ApprovedS3Product
      RoleArn: !GetAtt ServiceCatalogLaunchRole.Arn
      Description: Assumes ServiceCatalog-S3DataLake-LaunchRole so end users need no s3:* permissions

  CostCenterTagOption:
    Type: AWS::ServiceCatalog::TagOption
    Properties:
      Key: CostCenter
      Value: DataPlatform
      Active: true

  PortfolioTagAssociation:
    Type: AWS::ServiceCatalog::TagOptionAssociation
    Properties:
      ResourceId: !Ref DataPlatformPortfolio
      TagOptionId: !Ref CostCenterTagOption

  OrgWafBaselinePolicy:
    Type: AWS::FMS::Policy
    Properties:
      PolicyName: Org-ALB-Baseline-WAFv2-Policy
      RemediationEnabled: true
      ResourceType: AWS::ElasticLoadBalancingV2::LoadBalancer
      ExcludeResourceTags: false
      IncludeMap:
        ORG_UNIT:
          - ou-prod-12345678
      SecurityServicePolicyData:
        Type: WAFV2
        ManagedServiceData: !Sub |
          {
            "type": "WAFV2",
            "defaultAction": { "type": "ALLOW" },
            "preProcessRuleGroups": [
              {
                "ruleGroupType": "ManagedRuleGroup",
                "overrideAction": { "type": "NONE" },
                "managedRuleGroupIdentifier": {
                  "vendorName": "AWS",
                  "managedRuleGroupName": "AWSManagedRulesCommonRuleSet"
                },
                "excludeRules": []
              },
              {
                "ruleGroupType": "ManagedRuleGroup",
                "overrideAction": { "type": "NONE" },
                "managedRuleGroupIdentifier": {
                  "vendorName": "AWS",
                  "managedRuleGroupName": "AWSManagedRulesKnownBadInputsRuleSet"
                },
                "excludeRules": []
              }
            ],
            "postProcessRuleGroups": [],
            "overrideCustomerWebACLAssociation": true,
            "loggingConfiguration": {
              "logDestinationConfigs": [
                "arn:aws:firehose:us-east-1:999988887777:deliverystream/aws-waf-logs-central-secops"
              ]
            }
          }
```
