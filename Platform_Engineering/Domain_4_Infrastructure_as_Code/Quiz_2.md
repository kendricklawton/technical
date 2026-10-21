# PE Domain 4: Infrastructure as Code — Scenario Quiz 2

Test your mastery of state surgery (`moved`/`import`), preconditions, and native `terraform test`.

---

### Question 1
You want to refactor a root-level resource `aws_s3_bucket.logs` into a shared module `module.logging.aws_s3_bucket.bucket`. In Terraform 1.1+, what is the safest, declarative way to perform this refactoring without risking resource destruction or running manual CLI commands?

A) Add a `moved { from = aws_s3_bucket.logs; to = module.logging.aws_s3_bucket.bucket }` block directly in the HCL configuration and commit it to Git.  
B) Delete the resource from HCL and run `terraform apply`.  
C) Edit the JSON `.tfstate` file directly in a text editor.  
D) Execute manual `terraform state rm` and re-create the bucket.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Declarative `moved` blocks inform Terraform's graph engine of intentional refactorings. When `terraform plan` executes, it plans a state move (`1 to move, 0 to destroy`), safely updating the state during apply without touching physical infrastructure.
</details>

---

### Question 2
You have a legacy RDS database created manually through the AWS Web Console. You want to bring it under Terraform management. In Terraform 1.5+, what is the modern declarative method to achieve this?

A) Run `terraform apply -force`.  
```hcl
import {
  to = aws_db_instance.primary
  id = "prod-db-identifier"
}
```  
B) Delete the database in the console and let Terraform re-create it.  
C) Use an `import` block in HCL:  
D) Download the database data into a CSV file.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Terraform 1.5+ introduced declarative `import` blocks. It integrates imports into the standard plan/apply workflow and can even generate initial HCL code automatically via `terraform plan -generate-config-out=generated.tf`.
</details>

---

### Question 3
What is the primary difference between a **precondition** and a **postcondition** in a Terraform resource lifecycle block?

A) Preconditions are written in Python; postconditions are written in Rego.  
B) Preconditions evaluate conditions *before* the resource is evaluated or created (e.g., verifying input variables or dependencies); postconditions evaluate guarantees on the real attributes returned by the cloud provider *after* provisioning (e.g., verifying that an allocated volume is encrypted).  
C) Postconditions can only be used on data sources.  
D) Preconditions delete resources if they fail.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Preconditions act as guards before execution begins, halting plans if requirements are violated. Postconditions validate the state of the system immediately after creation, failing the run if the cloud provider returned an unexpected or non-compliant configuration.
</details>

---

### Question 4
In Terraform 1.6+ native testing (`terraform test`), what is the primary benefit of using `mock_provider` blocks in `.tftest.hcl` files?

A) It replaces Docker during integration tests.  
B) It converts HCL into Go code.  
C) It allows testing on real AWS accounts without paying bills.  
D) It mocks cloud provider APIs entirely in memory, allowing unit tests and assertions to run in milliseconds without requiring valid cloud credentials or making outbound network requests.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Mock providers simulate cloud responses based on schema definitions, enabling rapid, offline, and cost-free unit testing of Terraform module logic, assertions, and validation rules in local development and CI.
</details>

---

### Question 5
Why should platform engineers strictly avoid manual edits to the raw JSON state file using text editors (`vim` or `nano`)?

A) Text editors automatically delete all AWS resources.  
B) Terraform state files are encrypted with non-printable binary characters.  
C) JSON files cannot be opened on Linux.  
D) Manual edits bypass internal state serialization, easily corrupting JSON syntax, invalidating internal resource serials and lineage hashes, and risking catastrophic cluster desynchronization.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** The state file contains complex lineage counters, cryptographic checksums, and dependency indexes. Hand-editing raw JSON frequently corrupts the structure. State modifications should always be executed via declarative `moved`/`import` blocks or official CLI tools (`terraform state`).
</details>

---

### Question 6
An engineer writes a custom variable validation rule in HCL:
```hcl
variable "instance_count" {
  type = number
  validation {
    condition     = var.instance_count > 0 && var.instance_count <= 10
    message       = "Instance count must be between 1 and 10."
  }
}
```
When is this validation evaluated during a Terraform run?

A) Only after all resources have been deployed to AWS.  
B) Only when running `terraform destroy`.  
C) During the state lock release phase.  
D) Early during `terraform plan` and `terraform validate`, immediately halting execution before any cloud API calls are made if the input violates the condition.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Variable validation blocks execute early in the parse/plan phase. If a user inputs an invalid parameter, Terraform fails fast with the custom message before creating execution graphs or querying cloud APIs.
</details>

---

### Question 7
What is the function of the `terraform state replace-provider` command?

A) It updates the state file when migrating resources from one provider namespace to another (e.g., migrating from `hashicorp/aws` to an internal mirror or OpenTofu registry) without re-creating infrastructure.  
B) It forces all resources to reboot.  
C) It updates the operating system kernel.  
D) It deletes all cloud providers.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** When provider source addresses change, `state replace-provider` updates the provider metadata recorded in the state file, ensuring resources seamlessly bind to the new provider binary without recreation.
</details>

---

### Question 8
In native `terraform test`, what does the `command = plan` directive inside a `run` block specify?

A) The test will create real cloud resources and leave them running.  
B) The test will execute a bash script.  
C) The test will execute only through the `plan` phase, asserting conditions on planned state changes without actually provisioning physical infrastructure.  
D) The test will generate an architecture diagram.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Setting `command = plan` in a test run block performs a dry-run evaluation. It validates that variables and dynamic logic produce the expected planned attributes without calling mutating cloud APIs.
</details>

---

### Question 9
What happens if a Terraform resource has an `ignore_changes` lifecycle rule:
```hcl
lifecycle {
  ignore_changes = [tags["LastModified"]]
}
```

A) Terraform stops tracking the entire resource in state.  
B) Terraform ignores any external drift or changes made to the `LastModified` tag during refresh and plan, preventing unnecessary updates from overwriting external automated tag updates.  
C) The resource becomes read-only.  
D) Terraform deletes the tag from the physical resource.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** `ignore_changes` instructs Terraform's diff engine to disregard changes to specific attributes, preventing friction when external systems (like cost automation or autoscaling groups) modify attributes out-of-band.
</details>

---

### Question 10
What is the difference between `terraform validate` and `terraform plan`?

A) There is no difference.  
B) `validate` modifies state; `plan` does not.  
C) `validate` checks only internal syntactic validity, HCL structure, and variable consistency without querying remote cloud APIs or checking state; `plan` acquires the state lock, refreshes existing state against real cloud APIs, and computes the exact diff.  
D) `validate` requires AWS credentials; `plan` does not.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** `terraform validate` is a fast, offline syntax and type checker that runs without remote state or cloud credentials. `terraform plan` is a full operational preview that reads remote state and queries live cloud APIs.
</details>
