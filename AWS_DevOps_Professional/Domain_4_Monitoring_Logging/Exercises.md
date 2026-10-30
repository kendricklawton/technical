# Domain 4: Monitoring and Logging — Production Architecture Exercises & Solutions

## Exercise 1: CloudWatch Unified Agent Deployment at Scale via SSM Parameter Store & State Manager with Process Monitoring (`procstat`)

### Scenario
An enterprise runs hundreds of Amazon EC2 instances across multiple Auto Scaling Groups (ASGs). The operations team requires:
1. Continuous collection of memory (`mem_used_percent`), swap, and disk metrics (`disk_used_percent`).
2. Per-process CPU and memory monitoring for a critical payment daemon (`pay-daemon.service`) using `procstat`.
3. Metric rollups at both the individual `InstanceId` level and aggregated across the entire `AutoScalingGroupName`.
4. Zero manual SSH/Session Manager configuration; agent deployment and configuration updates must be completely automated via AWS Systems Manager (SSM).

### Complete Reference Architecture & Solution

#### 1. CloudWatch Unified Agent Configuration Schema (`AmazonCloudWatch-linux`)
Store this JSON configuration inside **AWS Systems Manager Parameter Store** as a `String` parameter named `AmazonCloudWatch-linux`:

```json
{
  "agent": {
    "metrics_collection_interval": 60,
    "run_as_user": "root"
  },
  "metrics": {
    "namespace": "Custom/EC2Enterprise",
    "append_dimensions": {
      "AutoScalingGroupName": "${aws:AutoScalingGroupName}",
      "InstanceId": "${aws:InstanceId}",
      "InstanceType": "${aws:InstanceType}"
    },
    "aggregation_dimensions": [
      ["AutoScalingGroupName"],
      ["AutoScalingGroupName", "InstanceId"]
    ],
    "metrics_collected": {
      "mem": {
        "measurement": [
          "mem_used_percent",
          "mem_available",
          "mem_total"
        ],
        "metrics_collection_interval": 30
      },
      "swap": {
        "measurement": [
          "swap_used_percent"
        ]
      },
      "disk": {
        "measurement": [
          "used_percent",
          "free"
        ],
        "resources": [
          "/",
          "/var/log",
          "/data"
        ],
        "drop_device": true
      },
      "procstat": [
        {
          "pattern": "pay-daemon",
          "measurement": [
            "cpu_time",
            "cpu_usage",
            "memory_rss",
            "memory_vms"
          ],
          "metrics_collection_interval": 10
        }
      ]
    }
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/messages",
            "log_group_name": "/aws/ec2/system-messages",
            "log_stream_name": "{instance_id}",
            "retention_in_days": 30,
            "timezone": "UTC"
          },
          {
            "file_path": "/var/log/payment/daemon.log",
            "log_group_name": "/aws/ec2/pay-daemon",
            "log_stream_name": "{instance_id}",
            "retention_in_days": 90,
            "timezone": "UTC"
          }
        ]
      }
    }
  }
}
```

#### 2. SSM State Manager Automation Association
Deploy the CloudWatch Agent package and apply the configuration automatically across all instances tagged `Environment=production`:

```bash
# Step 1: Install AmazonCloudWatchAgent via SSM Distributor
aws ssm create-association \
    --name "AWS-ConfigureAWSPackage" \
    --targets "Key=tag:Environment,Values=production" \
    --parameters '{"action":["Install"],"name":["AmazonCloudWatchAgent"]}' \
    --schedule-expression "rate(14 days)" \
    --association-name "InstallCloudWatchAgentFleet"

# Step 2: Push Configuration and Start the Agent
aws ssm create-association \
    --name "AmazonCloudWatch-ManageAgent" \
    --targets "Key=tag:Environment,Values=production" \
    --parameters '{
      "action":["configure"],
      "mode":["ec2"],
      "optionalConfigurationSource":["ssm"],
      "optionalConfigurationLocation":["AmazonCloudWatch-linux"],
      "optionalRestart":["yes"]
    }' \
    --schedule-expression "rate(7 days)" \
    --association-name "ConfigureCloudWatchAgentFleet"
```

#### 3. Verification & Validation
```bash
# Verify agent status on target instance
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
    -m ec2 \
    -a status

# Query aggregated ASG memory utilization via AWS CLI
aws cloudwatch get-metric-data --cli-input-json '{
  "MetricDataQueries": [
    {
      "Id": "asg_mem",
      "MetricStat": {
        "Metric": {
          "Namespace": "Custom/EC2Enterprise",
          "MetricName": "mem_used_percent",
          "Dimensions": [
            { "Name": "AutoScalingGroupName", "Value": "PaymentProcessingASG" }
          ]
        },
        "Period": 60,
        "Stat": "Average"
      }
    }
  ],
  "StartTime": "'$(date -u -d '15 minutes ago' +%FT%TZ)'",
  "EndTime": "'$(date -u +%FT%TZ)'"
}'
```

---

## Exercise 2: Cross-Account Centralized Logging Architecture with CloudWatch Logs Destination, Kinesis Data Streams, and Firehose

### Scenario
An enterprise follows an AWS Multi-Account Organization strategy:
- **Member Accounts (`Account-A`: 111111111111, `Account-B`: 222222222222)** run containerized and serverless applications.
- **Central Log Archive Account (`Account-Logs`: 999999999999)** must ingest, compress, encrypt, and archive all application and security log streams into Amazon S3 with 7-year WORM compliance, while streaming critical security logs to an OpenSearch Service cluster.

```text
========================================================================================
                 Cross-Account CloudWatch Logs Streaming to Kinesis
========================================================================================

  [ Account-A: 111111111111 (Workload Member) ]
  ┌────────────────────────────────────────────────────────────────────────┐
  │  [ Log Group: /app/orders ] ──► [ Subscription Filter: CentralFilter ] │
  └──────────────────────────────────────────┬─────────────────────────────┘
                                             │ Cross-Account PutLogEvents
                                             ▼
  [ Account-Logs: 999999999999 (Central Logging) ]
  ┌────────────────────────────────────────────────────────────────────────┐
  │  [ Logs Destination: CentralLogDestination ]                           │
  │         │                                                              │
  │         │ (AssumeRole: CWLtoKinesisRole)                               │
  │         ▼                                                              │
  │  [ Amazon Kinesis Data Stream ]                                        │
  │         │                                                              │
  │         ▼                                                              │
  │  [ Kinesis Data Firehose ]                                             │
  │         │                                                              │
  │         ├──► [ S3 Log Bucket (SSE-KMS + Object Lock) ]                 │
  │         └──► [ Amazon OpenSearch Service Cluster ]                     │
  └────────────────────────────────────────────────────────────────────────┘
```

### Complete Reference Architecture & Solution

#### 1. Central Account (`999999999999`): IAM Role for CloudWatch Logs Destination
Create `CWLtoKinesisRole` allowing CloudWatch Logs service to put records into the central Kinesis Data Stream:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudWatchLogsAssume",
      "Effect": "Allow",
      "Principal": {
        "Service": "logs.us-east-1.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Attach permissions policy to `CWLtoKinesisRole`:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowPutRecordToKinesis",
      "Effect": "Allow",
      "Action": [
        "kinesis:PutRecord",
        "kinesis:PutRecords",
        "kinesis:DescribeStream"
      ],
      "Resource": "arn:aws:kinesis:us-east-1:999999999999:stream/CentralLogStream"
    }
  ]
}
```

#### 2. Central Account (`999999999999`): Create CloudWatch Logs Destination
```bash
# Step 1: Create Destination
aws logs put-destination \
    --destination-name "CentralLogDestination" \
    --target-arn "arn:aws:kinesis:us-east-1:999999999999:stream/CentralLogStream" \
    --role-arn "arn:aws:iam::999999999999:role/CWLtoKinesisRole"

# Step 2: Attach Destination Access Policy for the Entire AWS Organization
aws logs put-destination-policy \
    --destination-name "CentralLogDestination" \
    --access-policy '{
      "Version": "2012-10-17",
      "Statement": [
        {
          "Sid": "AllowOrgAccountsPutSubscription",
          "Effect": "Allow",
          "Principal": "*",
          "Action": "logs:PutSubscriptionFilter",
          "Resource": "arn:aws:logs:us-east-1:999999999999:destination:CentralLogDestination",
          "Condition": {
            "StringEquals": {
              "aws:PrincipalOrgID": "o-xyz1234567"
            }
          }
        }
      ]
    }'
```

#### 3. Member Account (`111111111111`): Create Subscription Filter
```bash
aws logs put-subscription-filter \
    --log-group-name "/app/orders" \
    --filter-name "CentralLogFilter" \
    --filter-pattern "" \
    --destination-arn "arn:aws:logs:us-east-1:999999999999:destination:CentralLogDestination"
```

---

## Exercise 3: CloudWatch Logs Data Protection Policy for PII Masking, JSON Metric Filters, and Composite Alarms

### Scenario
An online banking application logs transaction requests to `/banking/transactions`.
1. Customer Social Security Numbers (SSNs) and Credit Card Numbers must be masked automatically at the ingestion boundary using a CloudWatch Logs Data Protection Policy.
2. An audit finding report must stream unmasked alerts to an S3 compliance bucket.
3. A JSON Metric Filter must track HTTP 5xx responses that take over 1,500ms (`statusCode >= 500 && latencyMs > 1500`).
4. A **CloudWatch Composite Alarm** must trigger emergency on-call paging ONLY when high latency and error rates occur while the system is **NOT** undergoing a scheduled maintenance window.

### Complete Reference Architecture & Solution

#### 1. CloudWatch Logs Data Protection Policy (`data-protection-policy.json`)
```json
{
  "Name": "BankingPIIDataProtectionPolicy",
  "Description": "Mask SSNs and Credit Card numbers in banking transaction logs",
  "Version": "2021-06-01",
  "Statement": [
    {
      "Sid": "AuditAndMaskPII",
      "DataIdentifications": [
        "arn:aws:dataprotection::aws:data-identifier/CreditCardNumber",
        "arn:aws:dataprotection::aws:data-identifier/Ssn-US"
      ],
      "Operation": {
        "Audit": {
          "FindingsDestination": {
            "S3": {
              "Bucket": "banking-pii-audit-archive-us-east-1"
            }
          }
        },
        "Deidentify": {
          "MaskConfig": {}
        }
      }
    }
  ]
}
```

Apply via CLI:
```bash
aws logs put-data-protection-policy \
    --log-group-identifier "/banking/transactions" \
    --policy-document file://data-protection-policy.json
```

#### 2. JSON Metric Filter Configuration
```bash
aws logs put-metric-filter \
    --log-group-name "/banking/transactions" \
    --filter-name "SevereSlow5xxErrors" \
    --filter-pattern '{ ($.statusCode >= 500) && ($.latencyMs > 1500) }' \
    --metric-transformations \
        metricName=Slow5xxCount,metricNamespace=Banking/API,metricValue=1,defaultValue=0
```

#### 3. CloudWatch Metric Alarms & Composite Alarm Definition
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'CloudWatch Metric Alarms and Composite Alarm with Maintenance Window Suppression'

Resources:
  Slow5xxErrorAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: Banking-HighSlow5xxErrors
      AlarmDescription: 'Fires when slow 5xx errors exceed 10 in a 5-minute period'
      Namespace: Banking/API
      MetricName: Slow5xxCount
      Statistic: Sum
      Period: 300
      EvaluationPeriods: 2
      DatapointsToAlarm: 2
      Threshold: 10
      ComparisonOperator: GreaterThanOrEqualToThreshold
      TreatMissingData: notBreaching

  HighLatencyAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: Banking-HighP99Latency
      AlarmDescription: 'Fires when P99 latency exceeds 2000ms'
      Namespace: AWS/ApplicationELB
      MetricName: TargetResponseTime
      Dimensions:
        - Name: LoadBalancer
          Value: app/banking-alb/1234567890abcdef
      ExtendedStatistic: p99
      Period: 60
      EvaluationPeriods: 3
      DatapointsToAlarm: 3
      Threshold: 2.0
      ComparisonOperator: GreaterThanThreshold
      TreatMissingData: missing

  MaintenanceWindowActiveAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: Banking-MaintenanceWindowActive
      AlarmDescription: 'State flag alarm set to ALARM by maintenance script to suppress paging'
      Namespace: Banking/Operations
      MetricName: MaintenanceActive
      Statistic: Maximum
      Period: 300
      EvaluationPeriods: 1
      Threshold: 1
      ComparisonOperator: GreaterThanOrEqualToThreshold
      TreatMissingData: notBreaching

  ProductionIncidentCompositeAlarm:
    Type: AWS::CloudWatch::CompositeAlarm
    Properties:
      AlarmName: Banking-CRITICAL-ProductionIncident
      AlarmDescription: 'Pages On-Call team if severe errors occur outside maintenance windows'
      AlarmRule: '(ALARM("Banking-HighSlow5xxErrors") OR ALARM("Banking-HighP99Latency")) AND NOT ALARM("Banking-MaintenanceWindowActive")'
      AlarmActions:
        - arn:aws:sns:us-east-1:111111111111:PagerDutyCriticalAlerts
```

---

## Exercise 4: End-to-End Distributed Tracing with AWS X-Ray, Custom Subsegments, Indexed Annotations, and Dynamic Sampling

### Scenario
A distributed microservices checkout application runs on Amazon ECS Fargate and calls third-party payment gateways (Stripe/PayPal) and an Amazon DynamoDB table.
- DevOps engineers must trace the exact time spent executing DynamoDB calls versus external HTTP gateways.
- High-priority enterprise customers (`tier = "enterprise"`) must have their traces indexed and searchable via X-Ray annotations.
- Sampling overhead must be strictly controlled: sample 100% of errors and 5% of healthy traffic using **dynamic centralized sampling rules**.

### Complete Reference Architecture & Solution

#### 1. Centralized X-Ray Sampling Rule (`sampling-rule.json`)
```json
{
  "SamplingRule": {
    "RuleName": "CheckoutEnterprisePriority",
    "RuleARN": "",
    "ResourceARN": "*",
    "Priority": 10,
    "FixedRate": 0.05,
    "ReservoirSize": 50,
    "ServiceName": "checkout-service",
    "ServiceType": "*",
    "Host": "*",
    "HTTPMethod": "*",
    "URLPath": "/checkout/*",
    "Version": 1,
    "Attributes": {
      "customerTier": "enterprise"
    }
  }
}
```

Apply via CLI:
```bash
aws xray create-sampling-rule --cli-input-json file://sampling-rule.json
```

#### 2. Python Microservice Implementation with AWS X-Ray SDK (`app.py`)
```python
import os
import time
import requests
import boto3
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

# Automatically patch boto3, requests, etc.
patch_all()

dynamodb = boto3.resource('dynamodb', region_name='us-east-1')
orders_table = dynamodb.Table('Orders')

def process_checkout(order_id, customer_id, customer_tier, amount):
    # Root segment managed by framework or manually started
    with xray_recorder.in_segment('CheckoutTransaction') as segment:
        # 1. Add searchable ANNOTATIONS (indexed, max 50 per trace)
        segment.put_annotation('orderId', str(order_id))
        segment.put_annotation('customerTier', str(customer_tier))
        segment.put_annotation('environment', 'production')

        # 2. Add METADATA (unindexed, for detailed debugging payloads)
        segment.put_metadata('orderDetails', {
            'customerId': customer_id,
            'itemCount': 3,
            'currency': 'USD'
        }, 'checkout_namespace')

        # 3. Custom Subsegment for External Payment Gateway Call
        subsegment = xray_recorder.begin_subsegment('ExternalPaymentGateway')
        try:
            subsegment.put_annotation('gatewayProvider', 'Stripe')
            start_time = time.time()
            response = requests.post(
                'https://api.stripe.com/v1/charges',
                data={'amount': amount, 'currency': 'usd'},
                timeout=3.0
            )
            subsegment.put_metadata('httpStatus', response.status_code)
            if response.status_code >= 400:
                subsegment.add_error_flag()
        except Exception as exc:
            subsegment.add_exception(exc)
            raise
        finally:
            xray_recorder.end_subsegment()

        # 4. DynamoDB Write (boto3 is auto-instrumented)
        orders_table.put_item(
            Item={
                'OrderId': order_id,
                'CustomerId': customer_id,
                'Amount': amount,
                'Status': 'COMPLETED',
                'Timestamp': int(time.time())
            }
        )

        return {'status': 'SUCCESS', 'orderId': order_id}
```

#### 3. Filter Expression Queries for Troubleshooting
In the AWS X-Ray / CloudWatch ServiceLens console:
- Find slow requests for enterprise customers:
  `annotation.customerTier = "enterprise" AND responsetime > 2`
- Find failed payment gateway calls:
  `annotation.gatewayProvider = "Stripe" AND error = true`

---

## Exercise 5: Organization-Wide CloudTrail with KMS CMK Encryption, SHA-256 Digest Validation, and Athena Partition Projection DDL

### Scenario
An AWS Organization requires centralized auditing across 50+ accounts.
1. An Organizational Trail must be created in the Management or Delegated Administrator account and written to a dedicated S3 bucket in the `Security-Audit` account (`999999999999`).
2. Log files must be encrypted with a Customer Managed KMS Key (CMK) with cross-account permissions.
3. Log File Validation must be enabled to prevent undetected tampering.
4. An Amazon Athena table with **Partition Projection** must be established to query millions of API events in seconds without running expensive `MSCK REPAIR TABLE` scans.

### Complete Reference Architecture & Solution

#### 1. KMS Key Policy in Audit Account (`999999999999`)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EnableRootManagement",
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::999999999999:root" },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "AllowCloudTrailDescribeAndGenerateDataKey",
      "Effect": "Allow",
      "Principal": { "Service": "cloudtrail.amazonaws.com" },
      "Action": [
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
      ],
      "Resource": "*",
      "Condition": {
        "StringLike": {
          "kms:EncryptionContext:aws:cloudtrail:arn": "arn:aws:cloudtrail:*:*:trail/*",
          "aws:PrincipalOrgID": "o-xyz1234567"
        }
      }
    },
    {
      "Sid": "AllowAthenaAndSecurityAdminsToDecrypt",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::999999999999:role/SecurityAnalystRole"
      },
      "Action": [
        "kms:Decrypt",
        "kms:DescribeKey"
      ],
      "Resource": "*"
    }
  ]
}
```

#### 2. Create the Organizational Multi-Region Trail via AWS CLI
```bash
aws cloudtrail create-trail \
    --name "EnterpriseOrganizationAuditTrail" \
    --s3-bucket-name "enterprise-audit-logs-999999999999" \
    --s3-key-prefix "org-cloudtrail" \
    --include-global-service-events \
    --is-multi-region-trail \
    --enable-log-file-validation \
    --kms-key-id "arn:aws:kms:us-east-1:999999999999:key/abcd-1234-efgh-5678" \
    --is-organization-trail

aws cloudtrail start-logging --name "EnterpriseOrganizationAuditTrail"
```

#### 3. Log File Integrity Validation Command
```bash
aws cloudtrail validate-logs \
    --trail-arn "arn:aws:cloudtrail:us-east-1:999999999999:trail/EnterpriseOrganizationAuditTrail" \
    --start-time "2026-10-15T00:00:00Z" \
    --end-time "2026-10-16T12:00:00Z" \
    --verbose
```

#### 4. High-Performance Athena Table DDL with Partition Projection
```sql
CREATE EXTERNAL TABLE IF NOT EXISTS security_audit.cloudtrail_org_logs (
    eventversion STRING,
    useridentity STRUCT<
        type: STRING,
        principalid: STRING,
        arn: STRING,
        accountid: STRING,
        invokedby: STRING,
        accesskeyid: STRING,
        username: STRING,
        sessioncontext: STRUCT<
            attributes: STRUCT<
                mfaauthenticated: STRING,
                creationdate: STRING>,
            sessionissuer: STRUCT<
                type: STRING,
                principalid: STRING,
                arn: STRING,
                accountid: STRING,
                username: STRING>>>,
    eventtime STRING,
    eventsource STRING,
    eventname STRING,
    awsregion STRING,
    sourceipaddress STRING,
    useragent STRING,
    errorcode STRING,
    errormessage STRING,
    requestparameters STRING,
    responseelements STRING,
    requestid STRING,
    eventid STRING,
    resources ARRAY<STRUCT<
        arn: STRING,
        accountid: STRING,
        type: STRING>>,
    eventtype STRING,
    recipientaccountid STRING
)
PARTITIONED BY (
    account STRING,
    region STRING,
    date STRING
)
ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
STORED AS INPUTFORMAT 'com.amazon.emr.cloudtrail.CloudTrailInputFormat'
OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION 's3://enterprise-audit-logs-999999999999/org-cloudtrail/AWSLogs/o-xyz1234567/'
TBLPROPERTIES (
    'projection.enabled' = 'true',
    'projection.account.type' = 'enum',
    'projection.account.values' = '111111111111,222222222222,999999999999',
    'projection.region.type' = 'enum',
    'projection.region.values' = 'us-east-1,us-west-2,eu-west-1',
    'projection.date.type' = 'date',
    'projection.date.range' = '2025/01/01,NOW',
    'projection.date.format' = 'yyyy/MM/dd',
    'projection.date.interval' = '1',
    'projection.date.interval.unit' = 'DAYS',
    'storage.location.template' = 's3://enterprise-audit-logs-999999999999/org-cloudtrail/AWSLogs/o-xyz1234567/${account}/CloudTrail/${region}/${date}'
);
```

---

## Exercise 6: VPC Flow Logs Custom Format (v5) Analysis for Transit Gateway & NAT Gateway Troubleshooting

### Scenario
An EC2 instance in a private application subnet cannot establish HTTPS connections to an external vendor endpoint.
- Traffic routes from `App VPC` through an **AWS Transit Gateway** to a centralized **Egress VPC** containing a NAT Gateway and Internet Gateway.
- Default VPC Flow Logs (v2) only reveal the Transit Gateway ENI and NAT Gateway IP, concealing whether the packets were dropped by Security Groups, Network ACLs, or route tables.
- DevOps engineers must deploy a **v5 Custom Format VPC Flow Log** capturing packet source/destinations, TCP flags, and traffic path identifiers.

### Complete Reference Architecture & Solution

#### 1. Create CloudWatch Log Group & IAM Role for VPC Flow Logs
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "vpc-flow-logs.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Attach Flow Log permissions:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "logs:DescribeLogGroups",
        "logs:DescribeLogStreams"
      ],
      "Resource": "*"
    }
  ]
}
```

#### 2. Create Custom Format v5 Flow Log via AWS CLI
```bash
aws ec2 create-flow-logs \
    --resource-type VPC \
    --resource-ids vpc-0a1b2c3d4e5f67890 \
    --traffic-type ALL \
    --log-destination-type cloud-watch-logs \
    --log-group-name "/aws/vpc/flow-logs-v5" \
    --deliver-logs-permission-arn "arn:aws:iam::111111111111:role/VPCFlowLogsDeliveryRole" \
    --log-format '${version} ${account-id} ${interface-id} ${srcaddr} ${dstaddr} ${srcport} ${dstport} ${protocol} ${packets} ${bytes} ${start} ${end} ${action} ${log-status} ${vpc-id} ${subnet-id} ${instance-id} ${tcp-flags} ${type} ${pkt-srcaddr} ${pkt-dstaddr} ${region} ${az-id} ${sublocation-type} ${sublocation-id} ${pkt-src-aws-service} ${pkt-dst-aws-service} ${flow-direction} ${traffic-path}'
```

#### 3. CloudWatch Logs Insights Diagnostic Queries

##### Query 1: Find Dropped Packets with Original Client and Target IPs
```text
fields @timestamp, instanceId, pktSrcaddr, pktDstaddr, dstport, action, tcpFlags
| filter action = "REJECT"
| stats count(*) as dropCount by pktSrcaddr, pktDstaddr, dstport, action
| sort dropCount desc
| limit 20
```

##### Query 2: Trace Egress Route Path and Identify NAT Gateway Bottlenecks
```text
fields @timestamp, interfaceId, srcaddr, dstaddr, packets, bytes, trafficPath
| filter flowDirection = "egress" and trafficPath = "7"
| stats sum(bytes) / 1024 / 1024 as TotalMegabytes by bin(5m)
| sort TotalMegabytes desc
```
*(Note: `trafficPath = 7` represents traffic egressing via a NAT Gateway).*
