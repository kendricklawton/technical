# Domain 3 (Resilient Cloud Solutions) — Quiz 6: Resilient Storage, Messaging & Networking

**Topic:** S3 CRR & MRAP, EFS Multi-AZ, SQS Dead-Letter Queues, SNS Fan-Out & ALB/NLB Resilient Networking  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: S3 Cross-Region Replication with KMS CMK Encryption and Ownership Overwrite

An enterprise healthcare platform needs to replicate patient health records from an Amazon S3 source bucket in Account A (`us-east-1`) to a destination compliance bucket in Account B (`eu-central-1`). The source objects are encrypted with a Customer Managed Key (CMK) in Account A. The compliance team mandates that:
1. All replicated objects in Account B must be encrypted using Account B's KMS CMK.
2. Ownership of all replicated objects must transfer automatically to Account B to prevent Account A from retaining access.
3. The replication role must not be granted broad administrative access.

**Which replication configuration satisfies all three constraints?**
- A. Enable S3 Replication with `SourceSelectionCriteria` for `SseKmsEncryptedObjects`, specify Account B's KMS CMK ARN in the replication rule encryption settings, enable Account B bucket owner object ownership override, and grant Account A's replication IAM role `kms:Decrypt` on Account A's key and `kms:Encrypt` / `kms:GenerateDataKey` on Account B's key.
- B. Enable S3 Replication using default S3-managed keys (`aws/s3`), configure an S3 bucket policy in Account B granting `s3:PutObject` to Account A, and execute an AWS Lambda function to re-encrypt objects post-upload.
- C. Use AWS DataSync running in Account A scheduled every 10 minutes to sync objects over VPC Peering, passing Account B's KMS key in the DataSync task definition.
- D. Enable S3 Same-Region Replication (SRR) in Account A, copy snapshots via AWS Backup to Account B, and apply an S3 Bucket Policy denying read access to Account A principals.

---

### Question 2: SQS Dead-Letter Queue Redrive Allow Policies & Message Poison Pill Handling

An asynchronous payment processing pipeline utilizes an Amazon SQS standard queue and an AWS Lambda consumer function. A corrupted message payload ("poison pill") causes the Lambda execution to crash repeatedly, triggering unhandled exceptions. The dead-letter queue (DLQ) is configured with `maxReceiveCount: 5`. However, security compliance mandates that the DLQ must only accept failed messages originating from specifically authorized production payment queues, preventing rogue or test queues from polluting the audit DLQ.

**Which configuration enforces this authorization boundary while managing failed message retries?**
- A. Configure a **Redrive Allow Policy** on the Dead-Letter Queue set to `byQueue` specifying the ARN of the primary payment processing queue, and set `maxReceiveCount: 5` on the primary queue's redrive policy.
- B. Attach an IAM Resource Policy to the primary queue granting `sqs:SendMessage` to the DLQ ARN and set Lambda concurrency to 1.
- C. Configure an SQS Access Policy on the primary queue that blocks messages containing JSON syntax validation errors using AWS WAF.
- D. Configure an S3 event notification on the DLQ that moves poison pill messages to an S3 Glacier vault after 3 failed attempts.

---

### Question 3: Long-Running Worker Processing and SQS Visibility Timeout Extension

A distributed video rendering worker fleet deployed on Amazon EC2 polls an Amazon SQS standard queue. Each message contains instructions to transcode a video file, which takes between 15 and 45 minutes depending on the file resolution. The queue's default `VisibilityTimeout` is set to 15 minutes. Workers report that large video files are frequently picked up and transcoded concurrently by multiple workers, resulting in duplicated processing and wasted compute costs.

**How should the worker processing loop be architected to eliminate duplicate processing while preserving resiliency if a worker crashes?**
- A. While processing, the worker must run a background heartbeat thread that periodically calls `aws sqs change-message-visibility` to extend the message visibility timeout before the current visibility window elapses, deleting the message upon final task completion.
- B. Increase the SQS queue's global `VisibilityTimeout` to 12 hours so that no other worker can ever receive the message while transcoding is underway.
- C. Convert the SQS queue to an Amazon Kinesis Data Stream with 50 shards, assigning dedicated partition keys to each individual video file.
- D. Configure the workers to immediately call `aws sqs delete-message` upon pulling the message from the queue before beginning the transcoding process.

---

### Question 4: SQS FIFO Message Deduplication & Ordered Sharded Processing

A financial order-matching engine requires strict FIFO (First-In, First-Out) message processing and exactly-once delivery semantics. The system processes 12,000 transactions per second across 5,000 distinct customer trading accounts. Messages belonging to the same trading account must be processed in strict chronological order, but trades across different customer accounts must be processed in parallel to meet high throughput requirements.

**Which Amazon SQS FIFO architecture fulfills these requirements?**
- A. Enable high-throughput mode for SQS FIFO, populate the `MessageGroupId` with the unique customer account ID for each order, and enable content-based deduplication or supply a unique `MessageDeduplicationId` for every transaction.
- B. Create 5,000 separate standard SQS queues, assigning one queue to each customer account, and poll them using an Auto Scaling worker fleet.
- C. Use an SQS FIFO queue with a single static `MessageGroupId` across all messages and increase the worker Auto Scaling Group desired capacity to 500 instances.
- D. Deploy an Amazon SNS FIFO topic subscribed to an Amazon Kinesis Data Firehose delivery stream that batches orders into Amazon S3.

---

### Question 5: Multi-Protocol Fan-Out Architecture with Amazon SNS Message Filtering

An enterprise order management platform publishes order events (`OrderCreated`, `OrderCancelled`, `PaymentFailed`) to an Amazon SNS topic. Three microservices subscribe to this topic:
- The **Fulfillment Service** needs only `OrderCreated` events where `order_total > 50`.
- The **Fraud Detection Service** needs all events regardless of type or amount.
- The **Customer Notification Service** needs `OrderCreated` and `OrderCancelled` events.

The development team wants to avoid deploying intermediate Lambda functions or writing custom filtering code in downstream consumers.

**How should this fan-out architecture be implemented with MINIMAL compute overhead?**
- A. Create three dedicated Amazon SQS queues subscribed to the SNS topic, and configure **SNS Subscription Filter Policies** on the Fulfillment and Customer Notification subscriptions to filter payloads based on message attributes.
- B. Deploy an AWS Lambda function subscribed to the SNS topic that inspects message bodies and routes messages to individual SQS queues using the AWS SDK.
- C. Create three separate Amazon SNS topics (one for each event type) and instruct the publisher application to execute multi-topic publishing logic.
- D. Deploy Amazon EventBridge Pipes connecting the SNS topic to DynamoDB Streams with pattern matching rules.

---

### Question 6: Amazon EFS Multi-AZ Cross-Subnet Mount Target Failover

A legacy content management system (CMS) runs on an EC2 Auto Scaling Group across three Availability Zones (`us-east-1a`, `us-east-1b`, `us-east-1c`). The application instances mount a shared Amazon Elastic File System (EFS) volume containing PHP application code and uploaded media assets. Corporate compliance mandates that if an Availability Zone fails completely, surviving EC2 instances must retain immediate, uninterrupted read/write access to the file system without unmounting or restarting.

**Which Amazon EFS storage class and networking configuration ensures this multi-AZ resilience?**
- A. Deploy Amazon EFS using the **Standard Multi-AZ storage class**, and create dedicated **EFS Mount Targets** in each of the three VPC subnets corresponding to `us-east-1a`, `us-east-1b`, and `us-east-1c`, configuring EC2 instances to mount using the EFS DNS name.
- B. Deploy Amazon EFS using the **One Zone storage class** in `us-east-1a`, and establish AWS Transit Gateway VPC peering routes to allow instances in `us-east-1b` and `us-east-1c` to access the mount target.
- C. Deploy an Amazon FSx for OpenZFS Multi-AZ file system, mounting the volume over an Application Load Balancer configured with TCP listeners.
- D. Create three independent EFS One Zone file systems (one per AZ) and configure AWS DataSync to continuously synchronize file system contents every minute.

---

### Question 7: Network Load Balancer (NLB) Zonal DNS Failover and Cross-Zone Load Balancing

A real-time telemetry ingestion pipeline receives high-throughput UDP/TCP data over an internet-facing Network Load Balancer (NLB) spanning three Availability Zones. The backend targets are deployed in an Auto Scaling Group. During a sudden application crash, all backend instances in `us-east-1a` become unhealthy, while instances in `us-east-1b` and `us-east-1c` remain healthy. Clients connecting to the NLB's IP address in `us-east-1a` experience connection timeouts because the NLB continues routing traffic to the impaired zone.

**Which two Network Load Balancer configurations prevent clients from experiencing connection failures?**
- A. Enable **Cross-Zone Load Balancing** on the NLB to distribute traffic across all healthy targets in all AZs, and configure Route 53 health checks with **DNS failover** to automatically remove the impaired AZ's IP address from DNS responses.
- B. Convert the Network Load Balancer into an Application Load Balancer and enable AWS WAF IP rate limiting.
- C. Attach an Elastic IP to the NLB in `us-east-1a` and trigger an AWS Lambda function to detach the Elastic IP during outages.
- D. Enable target group sticky sessions and decrease the NLB target deregistration delay to 0 seconds.

---

### Question 8: Global Ingress Acceleration via AWS Global Accelerator Anycast IP Failover

A mission-critical multiplayer gaming backend operates across two AWS Regions: `us-east-1` and `ap-northeast-1`. Gamers connect over UDP. The architecture requires instant, deterministic traffic shifting if a regional endpoint experiences packet loss, while routing users to the geographically closest healthy region with the lowest latency over the AWS private global backbone network.

**Which networking service provides static anycast IP entry points and sub-minute automated regional health failover for UDP traffic?**
- A. Deploy **AWS Global Accelerator**: Allocates two static anycast IPv4 addresses acting as single entry points globally, onboarding traffic onto the AWS global network at the edge and rerouting traffic between regional Application/Network Load Balancers in under 30 seconds upon health check failure.
- B. Deploy Amazon CloudFront with an Origin Group configured for primary/secondary failover and RTMP streaming enabled.
- C. Configure Amazon Route 53 Latency-Based routing records with calculated health checks pointing to regional Elastic IP addresses.
- D. Deploy an AWS Transit Gateway peering connection between the regions and configure BGP route priority using autonomous system prepending.

---

### Question 9: Amazon S3 Multi-Region Access Points (MRAP) Dynamic Routing & Metrics

A global SaaS platform serves static assets to millions of mobile applications worldwide from S3 buckets located in `us-east-1`, `eu-west-1`, and `ap-southeast-1`. The platform needs a single global S3 endpoint hostname that automatically routes client requests to the lowest-latency S3 bucket, provides automatic active-active failover if an S3 region becomes unavailable, and provides centralized routing control dials to shift traffic away from specific regions during maintenance.

**Which Amazon S3 architectural capability satisfies these global multi-region requirements?**
- A. Create an **Amazon S3 Multi-Region Access Point (MRAP)** spanning the three regional buckets, and utilize **S3 MRAP Failover Controls** to inspect routing status and shift traffic dynamically between active and passive regions.
- B. Create an Amazon Route 53 Geolocation routing policy pointing to individual S3 static website hosting bucket URLs in each region.
- C. Deploy an AWS CloudFront distribution pointing to an Amazon API Gateway multi-region endpoint that executes Lambda functions to fetch S3 objects.
- D. Configure S3 Cross-Region Replication (CRR) between all three buckets in a full-mesh topology and instruct client apps to cycle through bucket URLs sequentially.

---

## Answer Key & Explanations

### Question 1: S3 Cross-Region Replication with KMS CMK Encryption and Ownership Overwrite
**Answer:** A  
**Explanation:** 
When replicating S3 objects encrypted with AWS KMS CMKs across accounts:
1. **CMK Replication Support:** By default, S3 does not replicate objects encrypted with KMS CMKs. You must explicitly configure `SourceSelectionCriteria` with `SseKmsEncryptedObjects: { Status: "Enabled" }` and specify the destination KMS key ARN in Account B.
2. **KMS Permissions:** Account A's S3 replication IAM role requires `kms:Decrypt` permissions on Account A's KMS key (to read and decrypt source objects) and `kms:Encrypt` / `kms:GenerateDataKey` permissions on Account B's KMS key (granted via Account B's KMS key policy).
3. **Ownership Transfer:** In cross-account replication, the source object owner retains ownership of the replica by default unless configured otherwise. Adding `<AccessControlTranslation><Owner>Destination</Owner></AccessControlTranslation>` (or using S3 Object Ownership set to `BucketOwnerEnforced`) automatically transfers object ownership to Account B.

---

### Question 2: SQS Dead-Letter Queue Redrive Allow Policies & Message Poison Pill Handling
**Answer:** A  
**Explanation:** 
A **Redrive Allow Policy** is an Amazon SQS security and governance feature configured directly on a Dead-Letter Queue (DLQ):
- It controls which source queues are permitted to specify this specific SQS queue as their DLQ in their redrive policies.
- Setting the Redrive Allow Policy to `byQueue` with specific source queue ARNs (or `byQueue` matching specific accounts/prefixes) establishes a strict authorization boundary. This prevents unauthorized, developer, or test queues from redriving failed messages into a production compliance DLQ.
- The source queue defines the `maxReceiveCount` (e.g., 5), moving the poison pill message to the DLQ after 5 consecutive failed processing attempts.

---

### Question 3: Long-Running Worker Processing and SQS Visibility Timeout Extension
**Answer:** A  
**Explanation:** 
When a worker receives a message from an SQS queue, the message remains hidden from other workers for the duration of the queue's `VisibilityTimeout`:
- If processing takes longer than the visibility timeout (e.g., a 45-minute transcoding job on a 15-minute timeout), the timeout expires and SQS makes the message visible again to other workers, resulting in duplicate processing.
- Setting the global timeout to 12 hours (Option B) is an anti-pattern: if a worker crashes 2 minutes into processing, the message remains trapped in limbo for 12 hours before any other worker can retry it.
- Deleting the message before processing (Option D) causes permanent data loss if the worker crashes mid-task.
- The standard AWS distributed architecture pattern is for the worker process to run a lightweight heartbeat thread that periodically calls `aws sqs change-message-visibility` (e.g., extending visibility by 10 minutes every 5 minutes). If the worker crashes, the heartbeat ceases, and the message becomes available for retry promptly.

---

### Question 4: SQS FIFO Message Deduplication & Ordered Sharded Processing
**Answer:** A  
**Explanation:** 
Amazon SQS FIFO queues guarantee First-In, First-Out delivery and exactly-once processing:
- **MessageGroupId:** SQS FIFO orders messages strictly *within* each message group. When multiple message groups exist (e.g., using customer account IDs as the `MessageGroupId`), SQS allows workers to process different message groups **in parallel**, enabling high overall throughput (thousands of TPS) while maintaining strict chronological sequencing per individual customer.
- **Deduplication:** Setting content-based deduplication (SHA-256 hash of message body) or providing a unique `MessageDeduplicationId` ensures that retried network requests within the 5-minute deduplication interval are ignored, preventing duplicate processing.

---

### Question 5: Multi-Protocol Fan-Out Architecture with Amazon SNS Message Filtering
**Answer:** A  
**Explanation:** 
**Amazon SNS Subscription Filter Policies** offload filtering logic entirely to the managed SNS service:
- Publishers attach attributes to messages (e.g., `event_type: OrderCreated`, `order_total: 75`).
- Each subscriber (SQS queue) defines a Subscription Filter Policy in JSON:
  - Fulfillment: `{"event_type": ["OrderCreated"], "order_total": [{"numeric": [">", 50]}]}`
  - Fraud Detection: No filter policy (receives 100% of messages).
  - Customer Notification: `{"event_type": ["OrderCreated", "OrderCancelled"]}`
- SNS evaluates filter policies *before* delivering messages to subscriptions. Unmatched messages are discarded without invoking compute resources (no Lambda required), eliminating operational overhead and infrastructure cost.

---

### Question 6: Amazon EFS Multi-AZ Cross-Subnet Mount Target Failover
**Answer:** A  
**Explanation:** 
Amazon Elastic File System (EFS) **Standard Multi-AZ storage class** stores data redundantly across multiple Availability Zones:
- To access an EFS file system within a VPC, you create **Mount Targets** in each Availability Zone's subnet.
- When EC2 instances mount the file system using the EFS DNS name (`file-system-id.efs.aws-region.amazonaws.com`), Amazon Route 53 automatically resolves the DNS name to the IP address of the EFS mount target located in the **same Availability Zone** as the querying instance.
- If an Availability Zone experiences an outage, instances running in surviving zones continue accessing their local mount targets without interruption, maintaining seamless POSIX file system read/write operations.

---

### Question 7: Network Load Balancer (NLB) Zonal DNS Failover and Cross-Zone Load Balancing
**Answer:** A  
**Explanation:** 
By default, Network Load Balancers route traffic only to targets in the same Availability Zone as the receiving NLB node, and DNS queries resolve to all configured zonal IPs:
- If all targets in `us-east-1a` become unhealthy, clients querying DNS and connecting to the `us-east-1a` NLB node will fail if cross-zone load balancing is disabled.
- **Cross-Zone Load Balancing:** When enabled on an NLB, the NLB node in `us-east-1a` routes incoming traffic across the AWS backbone to healthy targets in other Availability Zones (`us-east-1b` and `us-east-1c`).
- **Route 53 Zonal DNS Failover:** Pairing the NLB with Route 53 health checks allows Route 53 to detect that an entire Availability Zone has no healthy targets and automatically remove that AZ's IP address from DNS responses, steering new client connections away from the impaired zone.

---

### Question 8: Global Ingress Acceleration via AWS Global Accelerator Anycast IP Failover
**Answer:** A  
**Explanation:** 
**AWS Global Accelerator** optimizes global ingress and provides instant failover:
- It provides two static **Anycast IPv4 addresses** that route traffic to the nearest AWS edge location via BGP anycast.
- Traffic enters the AWS global private fiber network at the edge closest to the user, bypassing public internet congestion.
- Global Accelerator supports both **TCP and UDP** protocols.
- It continuously monitors endpoint health across regions (ALBs, NLBs, or EC2 instances). If a regional endpoint group fails health checks, Global Accelerator automatically shifts traffic to healthy endpoints in the alternate region in **under 30 seconds**, completely independent of client DNS caching.

---

### Question 9: Amazon S3 Multi-Region Access Points (MRAP) Dynamic Routing & Metrics
**Answer:** A  
**Explanation:** 
**Amazon S3 Multi-Region Access Points (MRAP)** provide a single global DNS endpoint (`<mrap-id>.accesspoint.s3-global.amazonaws.com`) that fronts multiple S3 buckets in different AWS Regions:
- Built on AWS Global Accelerator, MRAP automatically routes client requests across the private AWS network to the S3 bucket with the lowest network latency.
- If a regional bucket becomes unavailable, MRAP automatically routes requests to the nearest healthy bucket.
- **S3 MRAP Failover Controls:** Provides operational traffic dials that allow administrators to manually switch regional bucket routing between Active and Passive status within minutes during planned maintenance or disaster recovery drills.
