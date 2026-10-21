# Domain 6 (Security and Compliance) — Quiz 3: AWS KMS Cryptography & Key Management

**Topic:** Envelope Encryption, Key Policies, Customer Managed Keys (CMKs) & Multi-Region Keys  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: Mandatory Default Key Policy Delegation to Root

When creating a Customer Managed Key (CMK) in AWS Key Management Service (KMS), the default key policy includes the following statement:
```json
{
  "Sid": "Enable IAM User Permissions",
  "Effect": "Allow",
  "Principal": { "AWS": "arn:aws:iam::111122223333:root" },
  "Action": "kms:*",
  "Resource": "*"
}
```

**What is the critical architectural purpose of this statement in the KMS key policy?**
- A. It grants the root user credentials the ability to log in without MFA.
- B. It **delegates authorization control to IAM policies** within account `111122223333`. Without this statement, IAM policies granting `kms:*` permissions have zero effect, and if no other principal is specified in the key policy, the KMS key becomes permanently unmanageable (orphaned).
- C. It allows all AWS accounts in the world to decrypt data.
- D. It enables automatic annual key rotation.

---

### Question 2: Mechanics of Envelope Encryption

A microservice encrypts 500 MB video files before uploading them to an Amazon S3 bucket. Why does AWS KMS use **Envelope Encryption** rather than passing the 500 MB video file directly to the `kms:Encrypt` API?
- A. The `kms:Encrypt` API has a hard payload limit of **4 KB** of data. In envelope encryption, the application calls `kms:GenerateDataKey` to obtain a plaintext Data Encryption Key (DEK) and a KMS-encrypted DEK. The application encrypts the 500 MB file locally in memory using the plaintext DEK (via AES-256-GCM), erases the plaintext DEK from memory, and stores the encrypted DEK alongside the encrypted video file.
- B. KMS cannot perform AES encryption.
- C. Envelope encryption is required only for compliance audits.
- D. S3 does not support files larger than 4 KB.

---

### Question 3: Multi-Region Keys vs Independent Regional Keys with Re-Encryption

A global application running in `us-east-1` and `eu-west-1` requires encrypted DynamoDB records to be replicated and read seamlessly in both regions with the lowest possible latency and without cross-region network calls to KMS.

**Which AWS KMS feature meets this requirement?**
- A. S3 Glacier Vault Lock.
- B. **AWS KMS Multi-Region Keys**, creating a Primary Key in `us-east-1` and replicating it to `eu-west-1`. Multi-Region keys share the exact same key ID, key material, and automatic key rotation status, allowing ciphertext encrypted in `us-east-1` to be decrypted locally in `eu-west-1` without cross-region API latency.
- C. Asymmetric RSA public keys exported to S3.
- D. Generating a single data key and storing it in a plaintext text file in Git.

---

### Question 4: Automatic vs On-Demand KMS Key Rotation

An organization's security policy requires that all Customer Managed Keys (CMKs) rotate their underlying cryptographic key material every year.

**How does automatic key rotation in AWS KMS handle existing data encrypted under previous versions of the key?**
- A. When a KMS key rotates automatically, all existing ciphertext is automatically decrypted and re-encrypted across all S3 buckets and EBS volumes.
- B. KMS retains all historical key material backing the CMK. KMS transparently uses the **new key material for all new `Encrypt` and `GenerateDataKey` calls**, while retaining previous key material to seamlessly **decrypt historical data** without requiring manual re-encryption of existing ciphertext.
- C. Historical data is deleted unless re-encrypted within 30 days.
- D. Automatic key rotation is not supported by AWS KMS; keys must be replaced manually.

---

### Question 5: KMS Grants vs Modifying Key Policies

An Auto Scaling Group launches EC2 instances that attach encrypted EBS volumes. The volume encryption uses a Customer Managed Key (CMK) owned by the infrastructure team. When new instances are launched, the EC2 service needs temporary permissions to use the KMS key to create and attach encrypted EBS volumes.

**How does AWS securely grant transient, least-privilege cryptographic access to AWS services without constantly updating the KMS Key Policy?**
- A. By making the KMS key publicly accessible.
- B. By using **KMS Grants** (`kms:CreateGrant`). A grant is an advanced access control mechanism that allows programmatic delegation of specific permissions (such as `Encrypt`, `Decrypt`, `CreateGrant`) to a principal on a KMS key, often with retirement tokens and constraints, without modifying the static key policy document.
- C. By sharing the IAM root password with the EC2 hypervisor.
- D. By disabling KMS encryption during scale-out events.

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

### Question 1: Mandatory Default Key Policy Delegation to Root

**Answer:** B  
**Explanation:** 
Unlike most AWS services where IAM policies alone can grant access, AWS KMS requires the **Key Policy** to authorize access. The statement allowing the account `root` principal (`arn:aws:iam::<AccountID>:root`) does not just mean the root user; in IAM policy terminology, it refers to the **account container**. It enables the account administrator to use standard IAM policies to grant permissions to users and roles. If this statement is omitted and no specific IAM users are listed in the key policy, the key is permanently orphaned and can never be modified or deleted.

---

### Question 2: Mechanics of Envelope Encryption

**Answer:** A  
**Explanation:** 
The AWS KMS `Encrypt` API can only encrypt up to **4,096 bytes (4 KB)** directly. For larger data, AWS uses **Envelope Encryption**:
1. Call `GenerateDataKey` against the KMS CMK. KMS returns a plaintext Data Key and a ciphertext Data Key.
2. The client uses the plaintext Data Key with an algorithm like AES-GCM to encrypt the file locally at high speed.
3. The plaintext Data Key is wiped from memory.
4. The encrypted Data Key is stored as metadata alongside the encrypted ciphertext.
To decrypt, the client calls `kms:Decrypt` passing the encrypted Data Key to recover the plaintext Data Key, which decrypts the file.

---

### Question 3: Multi-Region Keys vs Independent Regional Keys with Re-Encryption

**Answer:** B  
**Explanation:** 
Standard KMS keys are strictly bound to a single AWS Region. **AWS KMS Multi-Region Keys** are a set of synchronized keys in different Regions that share the same Key ID, key material, and key specs. Ciphertext created in one Region can be decrypted directly in another Region using the local replica key without making cross-region network calls, eliminating cross-region latency and dependencies during regional failover.

---

### Question 4: Automatic vs On-Demand KMS Key Rotation

**Answer:** B  
**Explanation:** 
When automatic key rotation is enabled for a symmetric KMS CMK (rotating every 365 days, or on-demand), KMS creates a new cryptographic backing key. The Key ID and ARN remain unchanged. KMS marks the new key material as active for all future encryption operations, but keeps the older key material so it can decrypt any existing ciphertext generated under earlier rotations without requiring data re-encryption.

---

### Question 5: KMS Grants vs Modifying Key Policies

**Answer:** B  
**Explanation:** 
**KMS Grants** allow dynamic, programmatic delegation of permissions on a KMS key. AWS services like Amazon EC2, EBS, and Auto Scaling use Grants behind the scenes. When an EC2 instance launches with an encrypted EBS volume, Auto Scaling creates a Grant on the KMS key allowing the EC2 service in that account to decrypt and attach the volume. When the instance terminates, the grant is retired or revoked, maintaining clean least-privilege security without Key Policy bloat.

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

