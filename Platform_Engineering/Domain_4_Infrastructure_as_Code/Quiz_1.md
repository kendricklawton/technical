# PE Domain 4: Infrastructure as Code — Scenario Quiz 1

Test your knowledge on Terraform core architecture, DAG evaluation, and state locking.

---

### Question 1
During a `terraform plan` execution, Terraform raises an error: `Error: Cycle: aws_security_group.sg_a, aws_security_group_rule.rule_b, aws_security_group.sg_a`. What is the root cause of this error within Terraform's core engine?

A) The AWS IAM credentials do not have permission to create security groups.  
B) The Directed Acyclic Graph (DAG) construction algorithm detected a circular dependency between resources, making it impossible to determine a valid topological execution order.  
C) The local operating system ran out of file descriptors.  
D) The state file S3 bucket is out of disk space.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Terraform constructs a Directed Acyclic Graph (DAG) to determine the order of operations. If Resource A references B, and B references A, a cycle is created. Because graphs must be acyclic (without cycles), Terraform halts immediately. Circular security group rules are resolved by separating the rules into independent `aws_security_group_rule` resources.
</details>

---

### Question 2
Two CI/CD pipelines run `terraform apply` concurrently on the same repository branch. The second pipeline fails immediately with: `Error: Error acquiring the state lock: ConditionalCheckFailedException`. Why did this happen?

A) Distributed state locking (via DynamoDB or S3 lock) prevented concurrent mutations, protecting the state file from catastrophic write corruption and race conditions.  
B) AWS was experiencing a regional outage.  
C) The second pipeline had an invalid Terraform syntax error.  
D) The Git branch was locked.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Distributed state locks ensure that only one Terraform process can mutate a state file at any given moment. The first process acquired the lock; the second process attempted to acquire the mutex and failed, gracefully aborting to prevent interleaving writes and state file corruption.
</details>

---

### Question 3
A CI job running `terraform apply` crashes abruptly due to an out-of-memory error on the runner. Subsequent pipeline runs fail because the state lock is still marked as held. After verifying that no other process is running, what is the correct remediation command?

A) `rm -rf .terraform`  
B) Delete the entire S3 bucket containing the state file.  
C) Reboot the AWS EC2 management console.  
D) `terraform force-unlock <LOCK-ID>`  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** When a process crashes violently without executing its release hook, the lock remains in the DynamoDB table. After confirming no active process is running, an administrator can manually release the lock using `terraform force-unlock <LOCK-ID>`.
</details>

---

### Question 4
What happens during the `terraform refresh` phase (executed by default during `terraform plan`)?

A) It formats all HCL files using `terraform fmt`.  
B) It queries real cloud provider APIs for all resources recorded in the state file and updates the in-memory state representation to reflect actual real-world attributes, detecting out-of-band drift.  
C) It downloads the latest version of the Terraform binary from the internet.  
D) It deletes all resources that were created more than 30 days ago.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Refresh reconciles state with reality: Terraform makes `Describe`/`Get` API calls to the cloud provider for every resource in the state file, updating its local view before computing the plan diff.
</details>

---

### Question 5
Why does storing all corporate cloud infrastructure (networking, DNS, databases, Kubernetes clusters) in a single monolithic Terraform state file represent an architectural anti-pattern?

A) OpenTofu does not support monolithic state files.  
B) Terraform only allows a maximum of 5 resources per state file.  
C) Monolithic state files cannot be stored in AWS S3.  
D) It creates a massive blast radius where a bug or syntax error in a minor resource blocks deployments across the entire company, and drastically slows down `plan` times due to thousands of sequential API refresh calls.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Monolithic states suffer from high blast radius (an accidental change can destroy foundational infrastructure) and severe performance degradation (a `plan` can take 30+ minutes querying thousands of resources). Best practice decouples state into distinct layers (e.g., networking, compute, data, applications).
</details>

---

### Question 6
What is the effect of passing the flag `-parallelism=20` to `terraform apply`?

A) It increases the number of concurrent graph node operations that Terraform executes in parallel from the default of 10 to 20, potentially speeding up applies on large configurations.  
B) It creates 20 duplicate copies of every resource.  
C) It uses 20 CPU cores on the local machine.  
D) It runs Terraform across 20 different cloud providers simultaneously.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** The `-parallelism` flag controls the maximum number of concurrent operations Terraform executes as it traverses independent paths in the dependency DAG.
</details>

---

### Question 7
In HCL, what is the primary benefit of using dynamic blocks (`dynamic "tag" { ... }`)?

A) They execute JavaScript inside the Terraform plan.  
B) They automatically decrypt KMS keys.  
C) They allow repeatable nested blocks (such as ingress rules, tags, or settings) to be generated programmatically based on a list or map variable, avoiding repetitive copy-pasting of boilerplate code.  
D) They bypass Terraform state locking.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Dynamic blocks construct repeated nested configuration blocks dynamically from collections, making modules highly reusable and DRY (Don't Repeat Yourself).
</details>

---

### Question 8
What is the function of the `terraform.lock.hcl` dependency lock file?

A) It pins the exact versions and cryptographic checksums (hashes) of all provider plugins used by the configuration, guaranteeing deterministic provider binaries across developer laptops and CI runners.  
B) It encrypts AWS credentials on disk.  
C) It prevents junior engineers from modifying code without approval.  
D) It locks the state file in DynamoDB.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** `terraform.lock.hcl` records the exact provider versions and multi-platform cryptographic hashes (hashes for Linux, macOS, Windows). Committing it to Git ensures every developer and CI runner executes identical provider binaries, preventing unexpected breaking changes from new provider releases.
</details>

---

### Question 9
What does the `sensitive = true` attribute on a Terraform output or variable accomplish?

A) It deletes the variable immediately after execution.  
B) It prevents Terraform from printing the plaintext value in standard CLI console outputs and logs during `plan` and `apply` operations.  
C) It encrypts the value in the `.tfstate` file using RSA-4096.  
D) It prevents the value from being passed to child modules.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Marking a variable or output as `sensitive` redacts it from stdout in terminal logs and CI displays. However, **the plaintext value is still stored unencrypted in the `.tfstate` file**, which is why securing remote state storage with KMS encryption and strict IAM access is critical.
</details>

---

### Question 10
When using `terraform state rm <resource_address>`, what happens to the physical cloud resource?

A) The resource is converted into a Kubernetes pod.  
B) The physical cloud resource is immediately terminated and deleted by the cloud provider.  
C) The resource is removed from Terraform's state file tracking, meaning Terraform will no longer manage it, but the physical cloud resource continues running completely untouched in the cloud.  
D) The cloud provider reboots the instance.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** `state rm` only severs the tracking link in the state file. It does not invoke cloud provider deletion APIs. The physical asset remains active in the cloud, unmanaged by Terraform.
</details>
