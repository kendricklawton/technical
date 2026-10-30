# Domain 6 (Security and Compliance) — Quiz 3: AWS KMS Cryptography & Key Management

**Topic:** Envelope Encryption, Key Policies, Customer Managed Keys (CMKs), Grants & Multi-Region Keys  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: Envelope Encryption Mechanics and Memory Safety

An application engineer is developing a microservice that encrypts multi-gigabyte financial dataset archives before uploading them to Amazon S3. The security architect insists that the service must use **Envelope Encryption** via AWS Key Management Service (KMS).

**What is the correct sequence of API calls and cryptographic operations executed by the application client?**
- A. The application calls `kms:GenerateDataKey` specifying the KMS key ARN; KMS returns a plaintext data key and an encrypted copy of the data key; the application encrypts the file locally using the plaintext key via AES-GCM, securely zeroes the plaintext key from memory, and stores the encrypted data key alongside the encrypted ciphertext.
- B. The application uploads the raw multi-gigabyte file directly to KMS using `kms:Encrypt`, and KMS returns the ciphertext file to be stored in Amazon S3.
- C. The application calls `kms:CreateKey` for every file, encrypts the file locally using the key ARN, and stores the private key in AWS Secrets Manager.
- D. The application calls `kms:Decrypt` passing the plaintext file, which returns a hashed token stored in Amazon DynamoDB.

---

### Question 2: Resolving ASG Launch Failures with Encrypted EBS Volumes via KMS Grants

A DevOps engineer configures an Auto Scaling Group (ASG) Launch Template specifying Amazon EBS volumes encrypted with a Customer Managed Key (CMK). When the ASG triggers a scale-out event, instances fail to launch with the error message: `Client.InternalError: Client error on launch`. Further investigation reveals that the EC2 service cannot access the KMS key to create and attach the encrypted EBS volumes.

**What is the required KMS Key Policy configuration to enable the Auto Scaling service to manage encrypted EBS volumes?**
- A. The KMS Key Policy must grant the Auto Scaling Service-Linked Role (`AWSServiceRoleForAutoScaling`) permission to invoke **`kms:CreateGrant`** with condition `kms:GrantIsForAWSResource: "true"`, allowing the ASG to delegate temporary cryptographic grants to Amazon EC2 to attach the encrypted volume.
- B. The KMS Key Policy must allow `kms:*` to `*` without conditions to allow any EC2 instance to read the master key.
- C. The EBS volumes must be switched from a Customer Managed Key to the AWS-managed key `aws/ebs`, which cannot fail launch events.
- D. An IAM Instance Profile attached to the EC2 instances must be granted `kms:Decrypt` and `kms:Encrypt` on the CMK.

---

### Question 3: Multi-Region Active-Active Encryption via KMS Multi-Region Keys

A global payments platform deploys an active-active microservice architecture across `us-east-1` and `eu-west-1`. An application client in `us-east-1` encrypts customer payment tokens locally using the AWS Encryption SDK and stores the ciphertext in an Amazon DynamoDB Global Table. A secondary microservice in `eu-west-1` reads the ciphertext from the local DynamoDB replica table and must decrypt it with sub-millisecond cryptographic latency without making cross-region network API calls back to `us-east-1`.

**Which AWS KMS key architecture satisfies this multi-region requirement?**
- A. Deploy an **AWS KMS Multi-Region Primary Key** in `us-east-1` and create a **Multi-Region Replica Key** in `eu-west-1`; both keys share the identical key ID and cryptographic key material, enabling the European service to decrypt the ciphertext locally using `eu-west-1` KMS endpoints without cross-region network latency.
- B. Create two independent single-region CMKs in `us-east-1` and `eu-west-1`, and execute an AWS Lambda function that re-encrypts DynamoDB items when streams replicate.
- C. Export the plaintext master key material from `us-east-1` using the AWS CLI and manually import it into an external key store in `eu-west-1`.
- D. Configure an AWS Transit Gateway peering connection between the two regions and route all KMS API traffic back to `us-east-1`.

---

### Question 4: Cryptographic Integrity Verification via KMS Encryption Context

A healthcare provider uses AWS KMS to encrypt electronic health records stored in Amazon S3. The security team mandates that even if a rogue user obtains unauthorized `kms:Decrypt` permissions on the master key, they must be prevented from decrypting patient records belonging to other clinics or decrypting data that has been copied between different S3 buckets.

**How does the KMS Encryption Context provide this cryptographic defense?**
- A. Pass an **Encryption Context** (a set of key-value pairs, e.g., `clinicId: "hospital-42"`, `bucket: "patient-records-prod"`) during `kms:GenerateDataKey` or `kms:Encrypt`; KMS cryptographically binds the context to the ciphertext, requiring the exact same key-value pairs to be supplied during `kms:Decrypt`, and enforce this requirement using the `kms:EncryptionContext` condition in the KMS key policy.
- B. Store the clinic ID inside the S3 object metadata and write an S3 Bucket Policy denying access to other clinic IDs.
- C. Embed a signed JWT token inside the KMS key description field and validate the token via API Gateway Lambda authorizers.
- D. Create a separate AWS KMS Customer Managed Key for every single individual patient record.

---

### Question 5: Eliminating KMS API Throttling via Data Key Caching

A high-throughput telemetry application processes 40,000 requests per second. Each request encrypts a small sensor payload using AWS KMS. During peak operational surges, the application encounters severe `ThrottlingException` errors from KMS, and AWS KMS API request costs escalate to tens of thousands of dollars per month.

**Which architectural adjustment eliminates KMS throttling and dramatically reduces encryption costs without lowering security?**
- A. Implement **Data Key Caching** using the **AWS Encryption SDK**: Configure a Cryptographic Materials Cache (CMC) with a local cache limit and maximum age (e.g., 5 minutes or 10,000 encryptions), allowing the application to reuse a single plaintext data key across multiple requests before requesting a new one from KMS.
- B. Request an immediate increase in the regional KMS `GenerateDataKey` request quota to 100,000 TPS via AWS Service Quotas.
- C. Switch from KMS Customer Managed Keys to default AWS-managed keys (`aws/s3`), which have unlimited free API request capacity.
- D. Write an AWS Step Functions state machine with exponential backoff and jitter retry loops surrounding every KMS API call.

---

### Question 6: Key Policy vs IAM Policy Evaluation: Cross-Account KMS Access

An enterprise operates a centralized Cryptographic Management account (`111122223333`) containing a Customer Managed Key (CMK). A workload running in a Spoke account (`444455556666`) needs to use this CMK to encrypt and decrypt Amazon S3 objects. An IAM role in the Spoke account has been granted full `kms:*` permissions on the CMK ARN in its identity-based policy, but API calls fail with `AccessDeniedException`.

**Why does this failure occur, and what is required to enable cross-account KMS key access?**
- A. For cross-account KMS access, permissions must be granted in **both** the KMS Key Policy and the IAM Policy; the Key Policy in Account `111122223333` must explicitly delegate authority to Account `444455556666` (or its root principal), and the IAM policy in Account `444455556666` must grant the specific KMS actions to the role.
- B. KMS CMKs cannot be used cross-account; the S3 bucket must be migrated into the central cryptographic account.
- C. The IAM role in the spoke account must be granted `AdministratorAccess` at the AWS Organizations root.
- D. The KMS key policy must be deleted and replaced with an S3 Bucket Policy granting cross-account upload rights.

---

### Question 7: Ephemeral Cryptographic Delegation via KMS Grants vs Key Policies

An automated batch processing cluster running on Amazon EC2 dynamically provisions dozens of temporary worker instances per hour. Each worker requires temporary, scoped permissions to decrypt a specific encrypted S3 dataset for the 30-minute duration of its job. Modifying the centralized KMS Key Policy for every dynamic worker instance is unfeasible due to policy size limits (32 KB) and high update latency.

**Which KMS feature provides programmatic, short-lived, and scalable cryptographic permission delegation?**
- A. KMS Grants: Programmatically create a grant using `aws kms create-grant` specifying the worker's IAM role ARN, granted operations (`Decrypt`), and encryption context constraints; the worker uses the grant token immediately and calls `aws kms retire-grant` upon task completion.
- B. Programmatically append the worker's IAM role ARN to the KMS Key Policy JSON using `PutKeyPolicy` and remove it upon job completion.
- C. Store the master key material in an S3 bucket and generate an S3 pre-signed URL valid for 30 minutes.
- D. Issue temporary AWS STS credentials with an inline session policy granting `kms:Decrypt` without updating the key.

---

### Question 8: Automatic Key Rotation for Customer Managed Keys (CMKs)

Corporate compliance mandates that all cryptographic keys used to protect data at rest must undergo cryptographic key rotation every 365 days. The security team wants to enable rotation on their symmetric Customer Managed Keys without requiring any application code changes, re-encrypting existing ciphertext, or breaking access to historically encrypted data.

**How does AWS KMS native Automatic Key Rotation operate?**
- A. Enabling **Automatic Key Rotation** on a symmetric CMK instructs KMS to generate new backing cryptographic key material every year; KMS retains all previous key material versions indefinitely, allowing KMS to decrypt historical ciphertext automatically while using the newest key material to encrypt new data.
- B. Enabling key rotation creates a brand new CMK with a new Key ARN, requiring all applications to update their configuration files.
- C. Enabling key rotation automatically launches an AWS Batch job that re-encrypts all historical S3 objects using the new key.
- D. KMS automatically deletes the old key material after 30 days, requiring all data to be decrypted and re-encrypted before rotation occurs.

---

### Question 9: Dedicated Hardware Security Modules via CloudHSM Key Stores

A high-assurance government defense application requires FIPS 140-2 Level 3 cryptographic hardware security module (HSM) compliance. Corporate governance strictly dictates that cryptographic keys must be generated and stored inside single-tenant, dedicated physical hardware where the customer maintains exclusive control of key material, while still integrating seamlessly with AWS managed services (EBS, S3, RDS).

**Which AWS KMS deployment model fulfills these compliance standards?**
- A. Configure an **AWS KMS Custom Key Store** backed by an **AWS CloudHSM cluster**: KMS integrates directly with the customer's dedicated FIPS 140-2 Level 3 CloudHSM cluster to generate and store key material, allowing standard KMS API calls and AWS service integrations to utilize keys stored in the dedicated HSMs.
- B. Deploy an AWS KMS External Key Store (XKS) connected over the public internet to a third-party software virtual appliance.
- C. Utilize default multi-tenant AWS KMS keys configured with multi-region replication.
- D. Deploy an EC2 instance running open-source SoftHSM with an encrypted EBS volume.

---

## Answer Key & Explanations

### Question 1: Envelope Encryption Mechanics and Memory Safety
**Answer:** A  
**Explanation:** 
AWS KMS cannot encrypt payloads larger than **4 KB** directly via the `kms:Encrypt` API. To encrypt large datasets (files, databases, disks), AWS uses **Envelope Encryption**:
1. The application calls `kms:GenerateDataKey`, passing the KMS CMK ARN and key spec (e.g., `AES_256`).
2. KMS returns two components: a **Plaintext Data Key** and a **Ciphertext (Encrypted) Data Key** (encrypted under the CMK).
3. The application encrypts the large file locally in memory using the plaintext data key and an algorithm like AES-GCM.
4. The application immediately wipes (zeroes out) the plaintext key from memory.
5. The application stores the encrypted data key alongside the encrypted data (e.g., in S3 object metadata or as a file header).
To decrypt, the application passes the encrypted data key back to `kms:Decrypt` to recover the plaintext data key.

---

### Question 2: Resolving ASG Launch Failures with Encrypted EBS Volumes via KMS Grants
**Answer:** A  
**Explanation:** 
When an Auto Scaling Group provisions an EC2 instance with an encrypted EBS volume:
- The Auto Scaling service orchestrates the instance launch using its Service-Linked Role (`AWSServiceRoleForAutoScaling`).
- To allow the EC2 service to attach the encrypted volume, the Auto Scaling service-linked role must create a **KMS Grant** delegated to the EC2 service principal.
- If the KMS Key Policy does not grant `kms:CreateGrant` to `AWSServiceRoleForAutoScaling` (with condition `kms:GrantIsForAWSResource: "true"`), the grant cannot be issued, causing the EC2 instance launch to fail with `Client.InternalError`.
- Granting `CreateGrant` in the CMK's key policy resolves the issue immediately.

---

### Question 3: Multi-Region Active-Active Encryption via KMS Multi-Region Keys
**Answer:** A  
**Explanation:** 
Standard AWS KMS keys are strictly regional resources and cannot decrypt ciphertext created in another region.
- **AWS KMS Multi-Region Keys** solve this challenge for active-active architectures:
- You create a Primary Multi-Region Key in `us-east-1` and replicate it to `eu-west-1`.
- Both keys share the **exact same key ID, key material, and key spec**, but have distinct regional ARNs.
- Ciphertext encrypted with the key in `us-east-1` can be decrypted directly by the replica key in `eu-west-1` locally without cross-region network calls, enabling sub-millisecond local decryption and high regional autonomy.

---

### Question 4: Cryptographic Integrity Verification via KMS Encryption Context
**Answer:** A  
**Explanation:** 
The **KMS Encryption Context** is an optional set of key-value pairs containing non-secret context data (e.g., `{"clinicId": "hospital-42", "bucket": "patient-records-prod"}`):
- When supplied during encryption, KMS cryptographically binds the context to the ciphertext as Additional Authenticated Data (AAD).
- To decrypt the data, the caller **must provide the exact same key-value pairs**. If the context mismatches or is omitted, decryption fails cryptographically.
- Furthermore, administrators can enforce context checks in IAM or Key Policies using the condition:
  `"StringEquals": { "kms:EncryptionContext:clinicId": "hospital-42" }`
- This ensures that a compromised role cannot decrypt data outside its assigned tenant or context.

---

### Question 5: Eliminating KMS API Throttling via Data Key Caching
**Answer:** A  
**Explanation:** 
Making 40,000 KMS API calls per second rapidly exceeds KMS regional request quotas (typically 10,000 to 50,000 TPS) and generates substantial API fees ($0.03 per 10,000 requests adds up to thousands of dollars).
- The **AWS Encryption SDK** provides built-in **Data Key Caching (Cryptographic Materials Caching)**:
- Instead of calling `GenerateDataKey` for every individual transaction, the client caches the data key locally in memory.
- You configure a cache threshold (e.g., cache for 5 minutes or up to 10,000 messages).
- The application reuses the cached data key locally, dropping KMS network API calls by over 99.9%, eliminating throttling, and drastically reducing encryption latency and operational cost.

---

### Question 6: Key Policy vs IAM Policy Evaluation: Cross-Account KMS Access
**Answer:** A  
**Explanation:** 
Access evaluation for AWS KMS keys differs between single-account and cross-account access:
- **Same-Account Access:** If the Key Policy delegates access to the account root (`Principal: { AWS: "arn:aws:iam::<acc>:root" }`), identity-based IAM policies alone are sufficient to grant permissions.
- **Cross-Account Access:** Permissions must be granted in **both places**:
  1. The KMS Key Policy in the owning account must explicitly grant permission to the external account or external IAM role.
  2. The IAM identity policy in the external account must grant permission to the external role to use the CMK ARN.
Without both policies explicitly allowing the action, cross-account access is denied.

---

### Question 7: Ephemeral Cryptographic Delegation via KMS Grants vs Key Policies
**Answer:** A  
**Explanation:** 
**AWS KMS Grants** provide programmatic, fine-grained, and temporary permission delegation:
- Ideal for dynamic worker pools, ephemeral container tasks, and AWS services.
- A grant is created via the API (`create-grant`), specifying the grantee principal, permitted operations (`Decrypt`), and encryption context constraints.
- It bypasses KMS Key Policy size limits (32 KB maximum JSON size) and avoids continuous policy updates that introduce race conditions.
- When the job concludes, the grant is retired or revoked immediately.

---

### Question 8: Automatic Key Rotation for Customer Managed Keys (CMKs)
**Answer:** A  
**Explanation:** 
When **Automatic Key Rotation** is enabled on a symmetric Customer Managed Key:
- AWS KMS automatically creates new backing cryptographic material every 365 days.
- The Key ID, Key ARN, key policy, and alias remain **completely unchanged**.
- KMS retains all historical backing key material indefinitely.
- Decryption is completely seamless: when historical ciphertext arrives, KMS identifies which key material version was used to encrypt it and decrypts it automatically.
- No application code updates, S3 re-encryption batch jobs, or configuration changes are required.

---

### Question 9: Dedicated Hardware Security Modules via CloudHSM Key Stores
**Answer:** A  
**Explanation:** 
An **AWS KMS Custom Key Store** backed by an **AWS CloudHSM cluster**:
- Combines the convenience and AWS service integration of AWS KMS with the dedicated, single-tenant hardware security of AWS CloudHSM.
- Cryptographic keys are generated and stored exclusively within customer-dedicated FIPS 140-2 Level 3 CloudHSM instances.
- All cryptographic operations for keys in the custom key store execute inside the CloudHSM cluster rather than standard multi-tenant KMS hardware.
- Native AWS services (like Amazon EBS, S3, and RDS) can interact with keys in the custom key store via standard KMS APIs.
