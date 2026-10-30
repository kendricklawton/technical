# Domain 5: Incident and Event Response — Production Hands-On Exercises (DOP-C02)

These 6 production-grade exercises cover every major incident and event response pattern tested on the AWS Certified DevOps Engineer – Professional (DOP-C02) exam, complete with working JSON, YAML, and Amazon States Language (ASL) solutions.

---

## Exercise 1: Multi-Account EventBridge Security Bus with Advanced Content Filtering & Input Transformer

### Scenario
You are the Principal DevOps Engineer for an AWS Organization (`o-prodorg999`) with 150 member accounts across `us-east-1` and `eu-west-1`. You must route all high-severity security and compliance events to a **Central Security Event Bus** (`arn:aws:events:us-east-1:999988887777:event-bus/central-secops-bus`) in the Security Account (`999988887777`).

On the Central Security Event Bus, you need a single EventBridge rule that matches:
1. Only live events (ignoring any EventBridge Archive replays).
2. Events from `aws.guardduty` OR `aws.ec2` (CloudTrail API calls), excluding any workload account whose Environment tag starts with `sandbox-` or `dev-`.
3. Either:
   - GuardDuty findings with `severity >= 7.0` and `<= 10.0`, OR
   - CloudTrail `AuthorizeSecurityGroupIngress` calls originating from an external IP outside `10.0.0.0/8` where an error code does **not** exist (`errorCode` is absent, meaning the API call succeeded!).
4. Format the matched event using an **EventBridge Input Transformer** (no Lambda!) to publish a clean, actionable alert to an encrypted SNS Topic (`arn:aws:sns:us-east-1:999988887777:secops-critical-alerts`) with a Dead-Letter Queue (`arn:aws:sqs:us-east-1:999988887777:secops-eventbridge-dlq`).

---

### Complete Working Solution

#### 1. Central Security Event Bus Resource-Based Policy (`central-secops-bus`)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowAllOrgAccountsPutEvents",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "events:PutEvents",
      "Resource": "arn:aws:events:us-east-1:999988887777:event-bus/central-secops-bus",
      "Condition": {
        "StringEquals": {
          "aws:PrincipalOrgID": "o-prodorg999"
        }
      }
    }
  ]
}
```

#### 2. Advanced EventBridge Rule Pattern on `central-secops-bus`
```json
{
  "source": ["aws.guardduty", "aws.ec2"],
  "replay-name": [
    { "exists": false }
  ],
  "detail": {
    "$or": [
      {
        "severity": [
          { "numeric": [">=", 7.0, "<=", 10.0] }
        ],
        "resource": {
          "instanceDetails": {
            "tags": {
              "value": [
                { "anything-but": { "prefix": "sandbox-" } },
                { "anything-but": { "prefix": "dev-" } }
              ]
            }
          }
        }
      },
      {
        "eventSource": ["ec2.amazonaws.com"],
        "eventName": ["AuthorizeSecurityGroupIngress"],
        "errorCode": [
          { "exists": false }
        ],
        "sourceIPAddress": [
          { "anything-but": { "cidr": "10.0.0.0/8" } }
        ]
      }
    ]
  }
}
```

#### 3. EventBridge Target Configuration with Input Transformer, Retry Policy, & DLQ
```json
[
  {
    "Id": "SecOpsCriticalSnsTarget",
    "Arn": "arn:aws:sns:us-east-1:999988887777:secops-critical-alerts",
    "InputTransformer": {
      "InputPathsMap": {
        "account": "$.account",
        "region": "$.region",
        "source": "$.source",
        "detailType": "$.detail-type",
        "time": "$.time",
        "gdType": "$.detail.type",
        "gdSeverity": "$.detail.severity",
        "apiEvent": "$.detail.eventName",
        "callerIp": "$.detail.sourceIPAddress"
      },
      "InputTemplate": "\"[SECOPS HIGH PRIORITY] Account: <account> | Region: <region> | Time: <time>\\nSource: <source> (<detailType>)\\nGuardDuty Finding: <gdType> (Severity: <gdSeverity>)\\nCloudTrail API Event: <apiEvent> from IP <callerIp>\\nAction Required: Review automated remediation workflow in Step Functions.\""
    },
    "RetryPolicy": {
      "MaximumRetryAttempts": 50,
      "MaximumEventAgeInSeconds": 14400
    },
    "DeadLetterConfig": {
      "Arn": "arn:aws:sqs:us-east-1:999988887777:secops-eventbridge-dlq"
    }
  }
]
```

---

## Exercise 2: Automated EC2 Forensic Isolation & Memory Capture State Machine Triggered by GuardDuty

### Scenario
When GuardDuty detects an active compromise on an EC2 instance (`CryptoCurrency:EC2/BitcoinTool.B!DNS` or `Backdoor:EC2/C&CActivity.B!DNS`, `severity >= 7.0`), your security architecture must automatically isolate the instance **without destroying volatile memory** and **without reducing production Auto Scaling capacity**.

Design the **AWS Step Functions Standard State Machine (ASL JSON)** that orchestrates:
1. Enabling EC2 API Termination Protection (`DisableApiTermination: true`).
2. Detaching the instance from its Auto Scaling Group with `ShouldDecrementDesiredCapacity: false` so a healthy replacement is immediately launched.
3. Swapping the instance's Security Groups to `sg-0quarantine999999` (which permits only VPC Endpoint traffic to SSM and S3).
4. Attaching an inline `AWSRevokeOlderSessions` policy to the instance's IAM Role to invalidate stolen IMDS credentials.
5. Executing an SSM Run Command document (`Forensic-Capture-LiME-RAM`) to upload volatile memory to an S3 Object Lock bucket, and creating EBS snapshots.

---

### Complete Working Solution: Step Functions ASL Definition

```json
{
  "Comment": "Automated EC2 Forensic Isolation, ASG Detachment, IAM Session Revocation, and RAM/EBS Capture",
  "StartAt": "EnableTerminationProtection",
  "States": {
    "EnableTerminationProtection": {
      "Type": "Task",
      "Resource": "arn:aws:states:::aws-sdk:ec2:modifyInstanceAttribute",
      "Parameters": {
        "InstanceId.$": "$.detail.resource.instanceDetails.instanceId",
        "DisableApiTermination": {
          "Value": true
        }
      },
      "ResultPath": "$.terminationProtectionResult",
      "Next": "CheckAutoScalingMembership"
    },
    "CheckAutoScalingMembership": {
      "Type": "Task",
      "Resource": "arn:aws:states:::aws-sdk:autoscaling:describeAutoScalingInstances",
      "Parameters": {
        "InstanceIds.$": "States.Array($.detail.resource.instanceDetails.instanceId)"
      },
      "ResultPath": "$.asgInfo",
      "Next": "IsInstanceInASG"
    },
    "IsInstanceInASG": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.asgInfo.AutoScalingInstances[0]",
          "IsPresent": true,
          "Next": "DetachFromAutoScalingGroupKeepCapacity"
        }
      ],
      "Default": "ApplyQuarantineSecurityGroup"
    },
    "DetachFromAutoScalingGroupKeepCapacity": {
      "Type": "Task",
      "Resource": "arn:aws:states:::aws-sdk:autoscaling:detachInstances",
      "Parameters": {
        "AutoScalingGroupName.$": "$.asgInfo.AutoScalingInstances[0].AutoScalingGroupName",
        "InstanceIds.$": "States.Array($.detail.resource.instanceDetails.instanceId)",
        "ShouldDecrementDesiredCapacity": false
      },
      "ResultPath": "$.detachResult",
      "Next": "ApplyQuarantineSecurityGroup"
    },
    "ApplyQuarantineSecurityGroup": {
      "Type": "Task",
      "Resource": "arn:aws:states:::aws-sdk:ec2:modifyInstanceAttribute",
      "Parameters": {
        "InstanceId.$": "$.detail.resource.instanceDetails.instanceId",
        "Groups": [
          "sg-0quarantine999999"
        ]
      },
      "ResultPath": "$.quarantineSgResult",
      "Next": "ParallelForensicsAndCredentialRevocation"
    },
    "ParallelForensicsAndCredentialRevocation": {
      "Type": "Parallel",
      "Next": "ApplyZeroEgressSecurityGroup",
      "Branches": [
        {
          "StartAt": "RevokeActiveIamRoleSessions",
          "States": {
            "RevokeActiveIamRoleSessions": {
              "Type": "Task",
              "Resource": "arn:aws:states:::aws-sdk:iam:putRolePolicy",
              "Parameters": {
                "RoleName.$": "$.detail.resource.instanceDetails.iamInstanceProfile.id",
                "PolicyName": "AWSRevokeOlderSessions",
                "PolicyDocument.$": "States.Format('{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Deny\",\"Action\":\"*\",\"Resource\":\"*\",\"Condition\":{\"DateLessThan\":{\"aws:TokenIssueTime\":\"{}\"}}}]}', $$.State.EnteredTime)"
              },
              "End": true
            }
          }
        },
        {
          "StartAt": "CaptureVolatileMemoryViaSSM",
          "States": {
            "CaptureVolatileMemoryViaSSM": {
              "Type": "Task",
              "Resource": "arn:aws:states:::aws-sdk:ssm:sendCommand",
              "Parameters": {
                "DocumentName": "AWS-RunShellScript",
                "InstanceIds.$": "States.Array($.detail.resource.instanceDetails.instanceId)",
                "Parameters": {
                  "commands": [
                    "#!/bin/bash",
                    "INSTANCE_ID=$(ec2-metadata -i | awk '{print $2}')",
                    "ss -tulpan > /tmp/netstat_${INSTANCE_ID}.txt",
                    "ps auxf > /tmp/processes_${INSTANCE_ID}.txt",
                    "/opt/forensics/avml /tmp/memory_${INSTANCE_ID}.lime",
                    "aws s3 cp /tmp/ s3://org-forensics-object-lock-vault/${INSTANCE_ID}/ --recursive --exclude '*' --include '*_${INSTANCE_ID}.*'"
                  ]
                }
              },
              "End": true
            }
          }
        },
        {
          "StartAt": "CreateForensicEbsSnapshots",
          "States": {
            "CreateForensicEbsSnapshots": {
              "Type": "Task",
              "Resource": "arn:aws:states:::aws-sdk:ec2:createSnapshots",
              "Parameters": {
                "InstanceSpecification": {
                  "InstanceId.$": "$.detail.resource.instanceDetails.instanceId",
                  "ExcludeBootVolume": false
                },
                "Description.$": "States.Format('Forensic snapshot for GuardDuty Finding {}', $.detail.id)",
                "TagSpecifications": [
                  {
                    "ResourceType": "snapshot",
                    "Tags": [
                      { "Key": "ForensicEvidence", "Value": "true" },
                      { "Key": "GuardDutyFindingId", "Value.$": "$.detail.id" }
                    ]
                  }
                ]
              },
              "End": true
            }
          }
        }
      ]
    },
    "ApplyZeroEgressSecurityGroup": {
      "Type": "Task",
      "Resource": "arn:aws:states:::aws-sdk:ec2:modifyInstanceAttribute",
      "Parameters": {
        "InstanceId.$": "$.detail.resource.instanceDetails.instanceId",
        "Groups": [
          "sg-0zeroegress000000"
        ]
      },
      "End": true
    }
  }
}
```

---

## Exercise 3: AWS Config Auto-Remediation with Custom SSM Automation Runbook & Manual Approval Gate (`aws:approve`)

### Scenario
AWS Config uses the managed rule `restricted-ssh` to detect Security Groups that allow unrestricted inbound SSH (`0.0.0.0/0` or `::/0` on port 22).
- If the Security Group has tag `Environment=dev`, your custom **SSM Automation Runbook** (`schemaVersion: '0.3'`) must **immediately** revoke the `0.0.0.0/0` port 22 rule.
- If the Security Group has tag `Environment=prod`, the runbook must **pause for manual approval (`aws:approve`)** from the `arn:aws:iam::111122223333:role/SecurityLeadApproverRole` (notifying `arn:aws:sns:us-east-1:111122223333:secops-approvals`) before revoking the rule.

---

### Complete Working Solution: SSM Automation Document (`Custom-RemediateRestrictedSSH.yaml`)

```yaml
schemaVersion: '0.3'
description: 'Remediates unrestricted SSH (0.0.0.0/0 on port 22) with automatic execution in dev and aws:approve gate in prod.'
assumeRole: '{{ AutomationAssumeRole }}'
parameters:
  GroupId:
    type: String
    description: 'The Security Group ID passed by AWS Config Remediation (RESOURCE_ID).'
  ApproverRoleArn:
    type: String
    default: 'arn:aws:iam::111122223333:role/SecurityLeadApproverRole'
  ApprovalTopicArn:
    type: String
    default: 'arn:aws:sns:us-east-1:111122223333:secops-approvals'
  AutomationAssumeRole:
    type: String
    description: 'The IAM Role ARN assumed by SSM Automation.'

mainSteps:
  - name: InspectSecurityGroupTags
    action: 'aws:executeScript'
    inputs:
      Runtime: python3.11
      Handler: get_env_tag
      InputPayload:
        GroupId: '{{ GroupId }}'
      Script: |-
        import boto3
        def get_env_tag(events, context):
            ec2 = boto3.client('ec2')
            resp = ec2.describe_security_groups(GroupIds=[events['GroupId']])
            tags = {t['Key']: t['Value'] for t in resp['SecurityGroups'][0].get('Tags', [])}
            return {'Environment': tags.get('Environment', 'prod').lower()}
    outputs:
      - Name: Environment
        Selector: '$.Payload.Environment'
        Type: String
    nextStep: BranchOnEnvironment

  - name: BranchOnEnvironment
    action: 'aws:branch'
    inputs:
      Choices:
        - NextStep: RevokeUnrestrictedSshIngress
          Variable: '{{ InspectSecurityGroupTags.Environment }}'
          StringEquals: 'dev'
      Default: RequestProductionApproval

  - name: RequestProductionApproval
    action: 'aws:approve'
    timeoutSeconds: 86400
    inputs:
      NotificationArn: '{{ ApprovalTopicArn }}'
      Message: 'AWS Config detected open SSH (0.0.0.0/0:22) on PRODUCTION Security Group {{ GroupId }}. Approve to revoke immediately.'
      MinRequiredApprovals: 1
      Approvers:
        - '{{ ApproverRoleArn }}'
    nextStep: RevokeUnrestrictedSshIngress

  - name: RevokeUnrestrictedSshIngress
    action: 'aws:executeAwsApi'
    inputs:
      Service: ec2
      Api: RevokeSecurityGroupIngress
      GroupId: '{{ GroupId }}'
      IpPermissions:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          IpRanges:
            - CidrIp: '0.0.0.0/0'
    nextStep: VerifySshClosed

  - name: VerifySshClosed
    action: 'aws:executeScript'
    inputs:
      Runtime: python3.11
      Handler: verify_handler
      InputPayload:
        GroupId: '{{ GroupId }}'
      Script: |-
        import boto3
        def verify_handler(events, context):
            ec2 = boto3.client('ec2')
            sg = ec2.describe_security_groups(GroupIds=[events['GroupId']])['SecurityGroups'][0]
            for perm in sg.get('IpPermissions', []):
                if perm.get('FromPort') == 22 and any(r.get('CidrIp') == '0.0.0.0/0' for r in perm.get('IpRanges', [])):
                    raise RuntimeError('Port 22 0.0.0.0/0 still present!')
            return {'ComplianceStatus': 'COMPLIANT'}
    isEnd: true
```

#### AWS Config Remediation Configuration JSON (`put-remediation-configurations`)
```json
[
  {
    "ConfigRuleName": "restricted-ssh",
    "TargetType": "SSM_DOCUMENT",
    "TargetId": "Custom-RemediateRestrictedSSH",
    "TargetVersion": "1",
    "Automatic": true,
    "MaximumAutomaticAttempts": 5,
    "RetryAttemptSeconds": 60,
    "Parameters": {
      "GroupId": {
        "ResourceValue": {
          "Value": "RESOURCE_ID"
        }
      },
      "AutomationAssumeRole": {
        "StaticValue": {
          "Values": [
            "arn:aws:iam::111122223333:role/SSMConfigRemediationRole"
          ]
        }
      }
    }
  }
]
```

---

## Exercise 4: Spot Instance Interruption & Rebalance Recommendation Graceful Draining

### Scenario
Your platform runs mixed On-Demand and Spot EC2 instances across two workloads:
1. An **Amazon ECS on EC2** cluster running stateless microservices behind an Application Load Balancer.
2. A fleet of **stateful video-transcoding EC2 Spot workers** in an Auto Scaling Group (`Transcode-Spot-ASG`) that must checkpoint their in-progress frame buffer to S3 (`s3://prod-transcode-checkpoints/`) when either an `EC2 Spot Instance Interruption Warning` or `EC2 Instance Rebalance Recommendation` fires.

Provide the complete configuration for:
- The ECS Agent UserData configuration for automatic Spot draining.
- The Auto Scaling Group Capacity Rebalancing & mixed instances policy configuration.
- The EventBridge Rule + SSM Run Command Target (using Input Transformer) that triggers the checkpoint script on the exact interrupted Spot Instance within seconds.

---

### Complete Working Solution

#### 1. ECS Container Instance Launch Template UserData (Automated Spot Draining)
```bash
#!/bin/bash
cat <<'EOF' >> /etc/ecs/ecs.config
ECS_CLUSTER=prod-microservices-cluster
ECS_ENABLE_SPOT_INSTANCE_DRAINING=true
ECS_CONTAINER_STOP_TIMEOUT=90s
EOF
```

#### 2. EventBridge Rule & SSM Run Command Target for Stateful Spot Workers
**EventBridge Event Pattern (`spot-interruption-checkpoint-rule.json`):**
```json
{
  "source": ["aws.ec2"],
  "detail-type": [
    "EC2 Spot Instance Interruption Warning",
    "EC2 Instance Rebalance Recommendation"
  ]
}
```

**EventBridge Target (`AWS-RunShellScript` via SSM Run Command with Input Transformer):**
```json
[
  {
    "Id": "CheckpointStatefulSpotWorker",
    "Arn": "arn:aws:ssm:us-east-1::document/AWS-RunShellScript",
    "RoleArn": "arn:aws:iam::111122223333:role/EventBridgeInvokeSsmRunCommandRole",
    "RunCommandParameters": {
      "RunCommandTargets": [
        {
          "Key": "tag:WorkloadRole",
          "Values": ["StatefulTranscoder"]
        }
      ]
    },
    "InputTransformer": {
      "InputPathsMap": {
        "instanceId": "$.detail.instance-id",
        "signalType": "$.detail-type"
      },
      "InputTemplate": "{\"commands\": [\"/opt/transcoder/graceful_checkpoint.sh <instanceId> '<signalType>'\"]}"
    }
  }
]
```
*(Note: Inside `/opt/transcoder/graceful_checkpoint.sh`, the script checks `if [ "$(ec2-metadata -i | awk '{print $2}')" == "$1" ]` or EventBridge targets a tiny SSM Automation document `AWS-RunShellScript` with `InstanceIds: ["<instanceId>"]` so only the affected Spot instance flushes its checkpoint to S3 and exits cleanly.)*

---

## Exercise 5: AWS Systems Manager Incident Manager Response Plan with Escalation Stages and AWS Chatbot Slack Remediation

### Scenario
Configure an automated Incident Response architecture in CloudFormation for critical database latency alarms:
1. Define an **SSM Incident Manager Escalation Plan** with two stages:
   - Stage 1 (`0` to `5` minutes): Engages Primary On-Call Contact (`arn:aws:ssm-contacts:us-east-1:111122223333:contact/primary-dba`) and stops progression if acknowledged (`StopIfAcknowledged: true`).
   - Stage 2 (after `5` minutes unacknowledged): Engages Secondary On-Call + Engineering Director (`arn:aws:ssm-contacts:us-east-1:111122223333:contact/director-eng`).
2. Define an **SSM Incident Manager Response Plan** that automatically launches an diagnostic SSM Automation Runbook (`Collect-RDS-Performance-Diagnostics`) and links to an **AWS Chatbot Slack Configuration**.
3. Configure the **AWS Chatbot Slack Channel** with a strict **Channel Guardrail Policy** (`ReadOnlyAccess` + `AWSSupportAccess` + custom `AllowOnlyApprovedSSMRunbooks`) so responders in Slack can run diagnostics and approved runbooks, but can never run raw destructive commands.

---

### Complete Working Solution: CloudFormation Template (`incident-manager-chatops.yaml`)

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'SSM Incident Manager Escalation Plan, Response Plan, and AWS Chatbot Slack Guardrails'

Resources:
  DbaEscalationPlan:
    Type: AWS::SSMContacts::Contact
    Properties:
      Alias: prod-dba-escalation-plan
      DisplayName: Production DBA Escalation Plan
      Type: ESCALATION
      Plan:
        - DurationInMinutes: 5
          Targets:
            - ContactTargetInfo:
                ContactId: arn:aws:ssm-contacts:us-east-1:111122223333:contact/primary-dba
                IsEssential: true
        - DurationInMinutes: 15
          Targets:
            - ContactTargetInfo:
                ContactId: arn:aws:ssm-contacts:us-east-1:111122223333:contact/secondary-dba
                IsEssential: true
            - ContactTargetInfo:
                ContactId: arn:aws:ssm-contacts:us-east-1:111122223333:contact/director-eng
                IsEssential: false

  ChatbotApprovedRunbookPolicy:
    Type: AWS::IAM::ManagedPolicy
    Properties:
      ManagedPolicyName: Chatbot-AllowApprovedRunbooksOnly
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Sid: AllowStartingApprovedRunbooksOnly
            Effect: Allow
            Action:
              - ssm:StartAutomationExecution
              - ssm:GetAutomationExecution
            Resource:
              - arn:aws:ssm:us-east-1:111122223333:automation-definition/Collect-RDS-Performance-Diagnostics:*
              - arn:aws:ssm:us-east-1:111122223333:automation-definition/Failover-Aurora-Cluster:*
          - Sid: DenyAllDirectResourceDeletions
            Effect: Deny
            Action:
              - rds:Delete*
              - ec2:Terminate*
              - dynamodb:Delete*
            Resource: '*'

  ProdOpsSlackChannel:
    Type: AWS::Chatbot::SlackChannelConfiguration
    Properties:
      ConfigurationName: prod-incident-war-room
      SlackWorkspaceId: T0123456789
      SlackChannelId: C0987654321
      IamRoleArn: arn:aws:iam::111122223333:role/AWSChatbotChannelExecutionRole
      GuardrailPolicies:
        - arn:aws:iam::aws:policy/ReadOnlyAccess
        - arn:aws:iam::aws:policy/AWSSupportAccess
        - !Ref ChatbotApprovedRunbookPolicy
      SnsTopicArns:
        - arn:aws:sns:us-east-1:111122223333:prod-incident-notifications
      LoggingLevel: INFO

  ProdDatabaseResponsePlan:
    Type: AWS::SSMIncidents::ResponsePlan
    Properties:
      Name: prod-rds-critical-latency-response
      DisplayName: Production RDS Critical Latency Response Plan
      IncidentTemplate:
        Title: 'CRITICAL: Production Aurora Database High Commit Latency'
        Impact: 1
        DedupeString: 'prod-aurora-cluster-01-latency'
        NotificationTargets:
          - SNSTopicArn: arn:aws:sns:us-east-1:111122223333:prod-incident-notifications
      Engagements:
        - !GetAtt DbaEscalationPlan.Arn
      ChatChannel:
        ChatbotSns:
          - arn:aws:sns:us-east-1:111122223333:prod-incident-notifications
      Actions:
        - SsmAutomation:
            DocumentName: Collect-RDS-Performance-Diagnostics
            RoleArn: arn:aws:iam::111122223333:role/SSMIncidentManagerAutomationRole
            TargetAccount: RESPONSE_PLAN_OWNER_ACCOUNT
            Parameters:
              - Key: DBClusterIdentifier
                Values:
                  - prod-aurora-cluster-01
```

---

## Exercise 6: Chaos Engineering Drill with AWS Fault Injection Service (FIS), SSM Latency Injection, and Composite Alarm Stop Conditions

### Scenario
Before a major shopping event, you must run an automated Chaos Engineering game day in your staging environment (`Env=Staging`) using **AWS Fault Injection Service (FIS)**:
1. **Action 1 (`InjectNetworkLatency`):** Use `aws:ssm:send-command` with the `AWSFIS-Run-Network-Latency` SSM document to inject `250ms` of latency (`jitter: 50ms`) on interface `eth0` across **50% of running EC2 instances** in `us-east-1a` tagged `Env=Staging,Tier=App` for 5 minutes (`PT5M`).
2. **Action 2 (`FailoverAuroraCluster`):** Immediately after Action 1 starts (`startAfter` or parallel), trigger `aws:rds:failover-db-cluster` on the staging Aurora cluster.
3. **Stop Condition:** Configure a **CloudWatch Composite Alarm** (`Staging-SLO-Breach-CompositeAlarm`) that monitors both Synthetic Canary failure and ALB HTTP 5xx Error Rate (`> 5%`). If the Composite Alarm enters `ALARM`, FIS must immediately abort the experiment and roll back the fault injection.

---

### Complete Working Solution: FIS Experiment Template JSON (`create-experiment-template`)

```json
{
  "description": "Chaos Game Day: 250ms Network Latency in us-east-1a (50% App Fleet) + Aurora Cluster Failover with Composite Alarm Stop Condition",
  "roleArn": "arn:aws:iam::111122223333:role/AWSFISExperimentExecutionRole",
  "stopConditions": [
    {
      "source": "aws:cloudwatch:alarm",
      "value": "arn:aws:cloudwatch:us-east-1:111122223333:alarm:Staging-SLO-Breach-CompositeAlarm"
    }
  ],
  "targets": {
    "AppInstancesAZ1a": {
      "resourceType": "aws:ec2:instance",
      "ResourceTags": {
        "Env": "Staging",
        "Tier": "App"
      },
      "filters": [
        {
          "path": "Placement.AvailabilityZone",
          "values": ["us-east-1a"]
        },
        {
          "path": "State.Name",
          "values": ["running"]
        }
      ],
      "selectionMode": "PERCENT(50)"
    },
    "StagingAuroraCluster": {
      "resourceType": "aws:rds:cluster",
      "resourceArns": [
        "arn:aws:rds:us-east-1:111122223333:cluster:staging-orders-aurora-cluster"
      ],
      "selectionMode": "ALL"
    }
  },
  "actions": {
    "Inject250msNetworkLatency": {
      "actionId": "aws:ssm:send-command",
      "description": "Inject 250ms network latency with 50ms jitter on eth0 for 5 minutes",
      "parameters": {
        "documentArn": "arn:aws:ssm:us-east-1::document/AWSFIS-Run-Network-Latency",
        "documentParameters": "{\"Interface\":\"eth0\",\"DelayMilliseconds\":\"250\",\"JitterMilliseconds\":\"50\",\"DurationSeconds\":\"300\",\"InstallDependencies\":\"True\"}",
        "duration": "PT5M"
      },
      "targets": {
        "Instances": "AppInstancesAZ1a"
      }
    },
    "ForceAuroraFailover": {
      "actionId": "aws:rds:failover-db-cluster",
      "description": "Force Aurora Reader promotion during high network latency",
      "startAfter": [
        "Inject250msNetworkLatency"
      ],
      "targets": {
        "Clusters": "StagingAuroraCluster"
      }
    }
  },
  "logConfiguration": {
    "cloudWatchLogsConfiguration": {
      "logGroupArn": "arn:aws:logs:us-east-1:111122223333:log-group:/aws/fis/experiments:*"
    },
    "logSchemaVersion": 2
  },
  "experimentOptions": {
    "accountTargeting": "single-account",
    "emptyTargetResolutionMode": "fail"
  }
}
```
