# Domain 3 (Resilient Cloud Solutions) — Quiz 2: Auto Scaling Lifecycle Deep Dive

**Topic:** Auto Scaling Lifecycle Hooks, Instance Refresh & Custom Termination Policies  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: Auto Scaling Lifecycle Hook Heartbeat Extension

An Amazon EC2 instance in an Auto Scaling Group receives a termination notification. An Auto Scaling Lifecycle Hook for `EC2_INSTANCE_TERMINATING` moves the instance to `Terminating:Wait` with a configured heartbeat timeout of 300 seconds (5 minutes). The instance runs a script that backs up a 50 GB local database to Amazon S3, which is taking longer than 5 minutes due to network bandwidth constraints.

**How can the instance prevent Auto Scaling from terminating it before the backup completes?**
- A. By rebooting the instance.
- B. By invoking the AWS CLI command `aws autoscaling record-lifecycle-action-heartbeat` periodically before each 300-second window elapses, extending the wait state (up to a maximum total limit of 48 hours).
- C. By changing the ASG desired capacity to 10.
- D. By detaching the network interface.

---

### Question 2: Custom Termination Policies and AZ Balancing

An Auto Scaling Group operates across three Availability Zones (`us-east-1a`, `us-east-1b`, `us-east-1c`) with 6 instances total (3 in `us-east-1a`, 2 in `us-east-1b`, 1 in `us-east-1c`). The ASG has a termination policy defined as `[OldestInstance, Default]`. The ASG triggers a scale-in event to terminate one instance.

**Which instance will Auto Scaling terminate first?**
- A. The oldest instance across the entire ASG, regardless of which Availability Zone it is in.
- B. Auto Scaling **first identifies the Availability Zone with the most instances** (`us-east-1a`, with 3 instances). It then selects the oldest instance located strictly within `us-east-1a` to preserve multi-AZ balance.
- C. The instance in `us-east-1c` because it is isolated.
- D. Auto Scaling terminates an instance at random.

---

### Question 3: Warm Pools for Rapid Auto Scaling

A Java application running on EC2 instances takes 12 minutes to initialize, load memory caches, and pass ALB health checks on boot. During flash sales, dynamic scaling takes too long to launch instances, resulting in server saturation.

**Which AWS Auto Scaling feature pre-initializes instances so they can enter service in under 30 seconds during scale-out events?**
- A. EC2 Reserved Instances
- B. **AWS Auto Scaling Warm Pools** with instances kept in the `Stopped` (or `Running`) state.
- C. Increasing the ALB target group health check interval.
- D. Amazon CloudFront origin request timeouts.

---

### Question 4: ASG Instance Refresh with Checkpoint Verifications

An enterprise deploys a new AMI across an ASG of 100 instances using Auto Scaling Instance Refresh. The operations team mandates that the refresh must pause at 20% and 50% completion to allow manual smoke testing and business sign-off before proceeding with remaining instances.

**Which Instance Refresh parameter configures these mandatory pauses?**
- A. Using a bash script that sleeps between batches.
- B. Configuring **Checkpoints** (e.g. `Checkpoints: [20, 50]`) in the `InstanceRefresh` preferences, along with a `CheckpointDelay` (in seconds).
- C. Setting minimum healthy hosts to 50%.
- D. Creating three separate ASGs.

---

### Question 5: Scale-In Protection for Stateful Batch Workers

A batch processing cluster runs on EC2 instances in an Auto Scaling Group. Instances pull jobs from an Amazon SQS queue. Processing a single job takes between 10 minutes and 2 hours. The ASG scales in during off-peak hours.

**How can instances protect themselves from being terminated while actively executing a job?**
- A. By disabling Auto Scaling entirely.
- B. When an instance starts processing a job, it executes `aws autoscaling set-instance-protection --instance-ids $(ec2-metadata -i) --protected-from-scale-in`. When the job completes, it clears the flag using `--no-protected-from-scale-in`.
- C. By changing the instance security group.
- D. By taking an EBS snapshot.

---

### Question 6: Predictive Scaling Metric Configurations

When configuring Predictive Scaling for an Auto Scaling Group, which metric is commonly used to forecast incoming workload demand?

**Which metric selection accurately reflects a standard predictive scaling configuration?**
- A. `CPUUtilization` only.
- B. **`ALBRequestCount`** (Application Load Balancer total request volume) as the load metric, paired with `ALBRequestCountPerTarget` as the scaling metric.
- C. Memory free in bytes.
- D. Disk space utilization.

---

### Question 7: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap

A centralized CI/CD pipeline in Account 111111111111 deploys an ECS service in Account 222222222222. The deployment stage fails with: 'The ciphertext refers to a customer master key that does not exist, does not allow access, or is invalid.' The S3 bucket is encrypted with the default AWS-managed key 'aws/s3'. How should the engineer resolve this?

- A. Update the S3 bucket policy to grant read access to Account 222222222222.
- B. Reconfigure the S3 artifact bucket to use a Customer Managed Key (CMK) and grant `kms:Decrypt` and `kms:DescribeKey` permissions to Account 222222222222's execution role in the KMS Key Policy.
- C. Create a VPC Endpoint for S3 in Account 222222222222.
- D. Enable S3 Object Ownership with Bucket Owner Preferred.

---

### Question 8: CodeDeploy In-Place ApplicationStop Lifecycle Failure

During an in-place deployment to an EC2 Auto Scaling Group, the deployment fails at the ApplicationStop lifecycle event hook because a previous buggy script had been deployed in the older revision. How can the engineer force CodeDeploy to deploy the new fixed revision without terminating the instances?

- A. Delete the CodeDeploy agent from all EC2 instances.
- B. Trigger a new deployment via the AWS CLI with the `--ignore-application-stop-failures` flag enabled.
- C. Manually stop the EC2 instances in the AWS Management Console.
- D. Reboot the instances and rerun the same deployment ID.

---

### Question 9: CloudFormation Custom Resource 60-Minute Stack Freeze

A CloudFormation stack creation hangs for exactly 1 hour in `CREATE_IN_PROGRESS` on a Lambda-backed custom resource before failing and rolling back. What was the root cause?

- A. The Lambda function exceeded its maximum execution timeout of 15 minutes.
- B. An unhandled exception crashed the Lambda function before it could send an HTTP PUT response with status SUCCESS or FAILED to the pre-signed Amazon S3 ResponseURL.
- C. CloudFormation lost IAM permissions to invoke Lambda.
- D. The S3 bucket holding the CloudFormation template was deleted.

---

## Answer Key & Detailed Explanations

### Question 1: Auto Scaling Lifecycle Hook Heartbeat Extension

**Answer:** B  
**Explanation:** 
When an instance is paused in a lifecycle hook wait state (`Pending:Wait` or `Terminating:Wait`), the application or script can call `record-lifecycle-action-heartbeat`. Each call resets the heartbeat timer, allowing the instance to continue running until its data transfer or graceful shutdown is complete, up to an overall limit of 48 hours.

---

### Question 2: Custom Termination Policies and AZ Balancing

**Answer:** B  
**Explanation:** 
AWS Auto Scaling always prioritizes **Availability Zone balance** before evaluating custom termination policies. When a scale-in event occurs, Auto Scaling first identifies the AZ with the most instances. It then applies the custom termination policy (`OldestInstance`) exclusively among the candidate instances in that specific over-represented AZ, ensuring the fleet remains balanced across zones.

---

### Question 3: Warm Pools for Rapid Auto Scaling

**Answer:** B  
**Explanation:** 
AWS Auto Scaling **Warm Pools** maintain a pool of pre-initialized EC2 instances that sit alongside the Auto Scaling Group. When instances scale out, they are pulled from the warm pool where packages are already installed and warmed, transitioning to `InService` in seconds rather than waiting minutes for cold boots. Keeping warm pool instances in the `Stopped` state incurs EBS storage costs but zero EC2 compute charges.

---

### Question 4: ASG Instance Refresh with Checkpoint Verifications

**Answer:** B  
**Explanation:** 
AWS Auto Scaling Instance Refresh supports **Checkpoints**. By specifying percentage thresholds (e.g. `Checkpoints: [20, 50]`), Auto Scaling automatically halts the instance refresh when those percentages of the fleet have been updated, waiting for the configured `CheckpointDelay` or until an operator resumes the operation after manual verification.

---

### Question 5: Scale-In Protection for Stateful Batch Workers

**Answer:** B  
**Explanation:** 
**Instance Scale-In Protection** can be toggled dynamically by the application code on the instance itself via the AWS CLI or SDK. When enabled on an instance, Auto Scaling will never select that instance for termination during scale-in events. Once the job finishes, the application clears protection, allowing the instance to scale down gracefully when idle.

---

### Question 6: Predictive Scaling Metric Configurations

**Answer:** B  
**Explanation:** 
Predictive Scaling pairs a **Load Metric** (which represents overall incoming customer demand, such as total ALB request count) with a **Scaling Metric** (which represents average utilization per instance, such as `ALBRequestCountPerTarget`). By forecasting total load, predictive scaling calculates how many target instances will be required to maintain the desired capacity per instance ahead of time.

---


### Question 7: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap
**Answer:** B
**Explanation:** Default AWS-managed keys like `aws/s3` cannot have their key policies modified and CANNOT be accessed across AWS accounts. Cross-account CodePipeline deployments strictly require an S3 bucket encrypted with a Customer Managed Key (CMK) whose key policy explicitly grants cross-account decrypt rights.

---

### Question 8: CodeDeploy In-Place ApplicationStop Lifecycle Failure
**Answer:** B
**Explanation:** CodeDeploy executes the `ApplicationStop` hook from the PREVIOUS revision's AppSpec file. If that script fails, all future deployments fail at that step unless you specify `--ignore-application-stop-failures`.

---

### Question 9: CloudFormation Custom Resource 60-Minute Stack Freeze
**Answer:** B
**Explanation:** CloudFormation Custom Resources expect a signed JSON payload uploaded to the pre-signed S3 `ResponseURL`. If the Lambda function crashes or exits without sending this signal, CloudFormation waits 1 hour before timing out.

---

