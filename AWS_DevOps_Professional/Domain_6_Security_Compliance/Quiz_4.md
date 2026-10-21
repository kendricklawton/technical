# Domain 6 (Security and Compliance) — Quiz 4: Secrets Manager Lifecycle & Kubernetes CSI Driver

**Topic:** Rotation Steps (`createSecret`, `setSecret`, `testSecret`, `finishSecret`), Dual-User Rotation & CSI Secrets Store  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: Secrets Manager 4-Step Rotation Lambda Lifecycle

When configuring automated rotation for a database secret in AWS Secrets Manager, Secrets Manager invokes a rotation Lambda function through four distinct lifecycle steps in sequence.

**What is the correct sequence of steps executed by the rotation Lambda function?**
- A. `validateSecret` -> `changeSecret` -> `saveSecret` -> `deleteOldSecret`
- B. **`createSecret`** -> **`setSecret`** -> **`testSecret`** -> **`finishSecret`**
- C. `testSecret` -> `createSecret` -> `finishSecret` -> `setSecret`
- D. `init` -> `rotate` -> `commit` -> `rollback`

---

### Question 2: Dual-User Rotation Strategy for Zero-Downtime Rotation

An enterprise database serves high-volume microservices with hundreds of active connection pools. If Secrets Manager changes the master database user's password while microservices are using connection pools, active connections may fail before applications can fetch the new secret.

**How does the Dual-User (Alternating Users) rotation strategy prevent application downtime?**
- A. By shutting down all microservices during the 15-minute rotation window.
- B. By using **two alternating database users** (e.g., `User_A` and `User_B`). While active microservices use `User_A`, Secrets Manager updates `User_B`'s password in the database, tests it, and switches the secret pointer to `User_B`. Applications gradually pick up `User_B` credentials while existing connections on `User_A` continue uninterrupted until the next rotation cycle, guaranteeing zero downtime.
- C. By keeping the database password static and rotating the database port.
- D. By using an unauthenticated public database proxy.

---

### Question 3: Secret Version Staging Labels (AWSCURRENT vs AWSPENDING)

During an automated rotation, the `testSecret` step fails because the database security group blocked the rotation Lambda function's IP address.

**What happens to the secret versions in Secrets Manager when rotation fails midway?**
- A. The entire secret is deleted and must be recreated manually.
- B. Secrets Manager keeps the original valid secret with the **`AWSCURRENT`** label. The untrusted candidate secret remains labeled **`AWSPENDING`**. Application clients requesting the secret continue to receive the working `AWSCURRENT` secret without outage.
- C. Secrets Manager forces a database restart.
- D. The previous secret is automatically invalidated.

---

### Question 4: Mounting Secrets in Amazon EKS via Secrets Store CSI Driver

An application running on Amazon EKS needs database credentials stored in AWS Secrets Manager. Security compliance forbids storing plaintext credentials in Kubernetes ConfigMaps or standard Kubernetes Secrets etcd storage.

**What is the recommended cloud-native approach to inject Secrets Manager secrets into EKS pods?**
- A. Hardcode secrets into container Dockerfiles.
- B. Deploy the **Kubernetes Secrets Store CSI Driver** with the **AWS Secrets Manager Provider**. Define a `SecretProviderClass` and mount the secret as an ephemeral volume directly into the pod's filesystem, using IAM Roles for Service Accounts (IRSA) for pod authentication.
- C. Run a cron job pod that writes credentials to a public NFS share.
- D. Allow the pods to run with host administrator privileges.

---

### Question 5: Secrets Manager Replication for Multi-Region Disaster Recovery

A global microservice operates in `us-east-1` (Primary) and `us-west-2` (Secondary). Database secrets are rotated automatically every 30 days in `us-east-1`.

**How does Secrets Manager ensure that the replica secret in `us-west-2` remains synchronized without running independent rotations in each region?**
- A. By configuring **Secrets Manager Multi-Region Secret Replication**. When the primary secret rotates in `us-east-1`, Secrets Manager automatically replicates the newly rotated secret value (and encryption metadata re-encrypted with the secondary region's KMS key) to the replica secret in `us-west-2`.
- B. By running a Lambda script in `us-west-2` that guesses the new password.
- C. Secrets cannot be replicated across regions; developers must manually copy them.
- D. By configuring an S3 cross-region sync on local disk.

---

### Question 6: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap

A centralized CI/CD pipeline in Account 111111111111 deploys an ECS service in Account 222222222222. The deployment stage fails with: 'The ciphertext refers to a customer master key that does not exist, does not allow access, or is invalid.' The S3 bucket is encrypted with the default AWS-managed key 'aws/s3'. How should the engineer resolve this?

- A. Update the S3 bucket policy to grant read access to Account 222222222222.
- B. Reconfigure the S3 artifact bucket to use a Customer Managed Key (CMK) and grant `kms:Decrypt` and `kms:DescribeKey` permissions to Account 222222222222's execution role in the KMS Key Policy.
- C. Create a VPC Endpoint for S3 in Account 222222222222.
- D. Enable S3 Object Ownership with Bucket Owner Preferred.

---

### Question 7: CodeDeploy In-Place ApplicationStop Lifecycle Failure

During an in-place deployment to an EC2 Auto Scaling Group, the deployment fails at the ApplicationStop lifecycle event hook because a previous buggy script had been deployed in the older revision. How can the engineer force CodeDeploy to deploy the new fixed revision without terminating the instances?

- A. Delete the CodeDeploy agent from all EC2 instances.
- B. Trigger a new deployment via the AWS CLI with the `--ignore-application-stop-failures` flag enabled.
- C. Manually stop the EC2 instances in the AWS Management Console.
- D. Reboot the instances and rerun the same deployment ID.

---

### Question 8: CloudFormation Custom Resource 60-Minute Stack Freeze

A CloudFormation stack creation hangs for exactly 1 hour in `CREATE_IN_PROGRESS` on a Lambda-backed custom resource before failing and rolling back. What was the root cause?

- A. The Lambda function exceeded its maximum execution timeout of 15 minutes.
- B. An unhandled exception crashed the Lambda function before it could send an HTTP PUT response with status SUCCESS or FAILED to the pre-signed Amazon S3 ResponseURL.
- C. CloudFormation lost IAM permissions to invoke Lambda.
- D. The S3 bucket holding the CloudFormation template was deleted.

---

## Answer Key & Detailed Explanations

### Question 1: Secrets Manager 4-Step Rotation Lambda Lifecycle

**Answer:** B  
**Explanation:** 
The AWS Secrets Manager rotation lifecycle consists of four specific sequential steps:
1. **`createSecret`**: Generates a new password and stores it in Secrets Manager under the staging label `AWSPENDING`.
2. **`setSecret`**: Modifies the credentials in the target database/system to match the new password in `AWSPENDING`.
3. **`testSecret`**: Validates the new credentials by attempting a connection to the database using `AWSPENDING`.
4. **`finishSecret`**: Moves the staging label `AWSCURRENT` to the new secret version, moving the previous version to `AWSPREVIOUS`.

---

### Question 2: Dual-User Rotation Strategy for Zero-Downtime Rotation

**Answer:** B  
**Explanation:** 
In the **Dual-User rotation pattern**, Secrets Manager manages two database users for the same application. When rotating, Secrets Manager resets the password of the inactive user (e.g., `User_B`), tests it, and then updates the current secret to point to `User_B`. The application transitions to `User_B` without breaking in-flight transactions running on `User_A`. On the next rotation cycle, `User_A` is updated and rotated.

---

### Question 3: Secret Version Staging Labels (AWSCURRENT vs AWSPENDING)

**Answer:** B  
**Explanation:** 
Secrets Manager protects against failed rotations by using staging labels (`AWSCURRENT`, `AWSPENDING`, `AWSPREVIOUS`). `AWSCURRENT` is only shifted to the new secret version during the final `finishSecret` step. If `testSecret` fails, rotation halts, and `AWSCURRENT` remains pointing to the known-good working secret, preventing service disruption.

---

### Question 4: Mounting Secrets in Amazon EKS via Secrets Store CSI Driver

**Answer:** B  
**Explanation:** 
The **Secrets Store CSI Driver** for Kubernetes, paired with the AWS Secrets Manager Provider, allows mounting secrets from AWS Secrets Manager directly into Pod volumes as files. The pod assumes an IAM role via IAM Roles for Service Accounts (IRSA) to retrieve the secret from AWS without storing sensitive plaintext in etcd. It also supports automated rotation synchronization (auto-rotation).

---

### Question 5: Secrets Manager Replication for Multi-Region Disaster Recovery

**Answer:** A  
**Explanation:** 
AWS Secrets Manager provides built-in multi-Region replication. When you replicate a secret to secondary Regions, Secrets Manager maintains read-only replicas in those Regions. When the primary secret is rotated, Secrets Manager transparently synchronizes the new secret version to all replica Regions and re-encrypts the secret using the designated KMS key in each target Region, ensuring DR readiness.

---


### Question 6: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap
**Answer:** B
**Explanation:** Default AWS-managed keys like `aws/s3` cannot have their key policies modified and CANNOT be accessed across AWS accounts. Cross-account CodePipeline deployments strictly require an S3 bucket encrypted with a Customer Managed Key (CMK) whose key policy explicitly grants cross-account decrypt rights.

---

### Question 7: CodeDeploy In-Place ApplicationStop Lifecycle Failure
**Answer:** B
**Explanation:** CodeDeploy executes the `ApplicationStop` hook from the PREVIOUS revision's AppSpec file. If that script fails, all future deployments fail at that step unless you specify `--ignore-application-stop-failures`.

---

### Question 8: CloudFormation Custom Resource 60-Minute Stack Freeze
**Answer:** B
**Explanation:** CloudFormation Custom Resources expect a signed JSON payload uploaded to the pre-signed S3 `ResponseURL`. If the Lambda function crashes or exits without sending this signal, CloudFormation waits 1 hour before timing out.

---

