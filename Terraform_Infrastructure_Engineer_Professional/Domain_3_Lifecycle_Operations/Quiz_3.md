# Domain 3: Lifecycle Operations & Day-2 Management — Quiz 3

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
In an autoscaling group or database cluster, you need to ensure that a newly updated instance is fully provisioned and healthy BEFORE the old instance is terminated. Which lifecycle configuration must be used?
- A. `lifecycle { create_before_destroy = true }`
- B. `lifecycle { prevent_destroy = true }`
- C. `lifecycle { ignore_changes = [all] }`
- D. `lifecycle { replace_triggered_by = [] }`

---

### Question 2
You want to force a resource (e.g. `aws_instance.bastion`) to be recreated whenever a specific cryptographic hash changes on a bootstrap user-data script. In modern Terraform (v1.2+), which lifecycle argument replaces legacy `taint` commands?
- A. `replace_triggered_by = [terraform_data.bootstrap.output]`
- B. `recreate_on_change = true`
- C. `auto_taint = true`
- D. `taint_when = true`

---

### Question 3
A developer accidentally updates a production database configuration and runs `terraform apply`. Terraform attempts to delete the production database. Which lifecycle rule acts as a safety guardrail by failing the plan if a destruction is attempted?
- A. `lifecycle { prevent_destroy = true }`
- B. `lifecycle { create_before_destroy = true }`
- C. `lifecycle { immutable = true }`
- D. `lifecycle { lock = true }`

---

### Question 4
Why is `terraform apply -target=...` strongly discouraged for day-to-day routine operations in production environments?
- A. It runs 10x slower than full apply.
- B. Targeting isolates a sub-graph, ignoring dependency relationships and leaving other resources un-refreshed, which leads to hidden configuration drift and inconsistent state.
- C. It deletes all untargeted resources.
- D. It invalidates AWS IAM credentials.

---

### Question 5
You have an AWS EC2 instance whose tags are modified by external automated compliance bots. Terraform plan continuously proposes reverting these external tags. How can you configure Terraform to ignore changes to the `tags` attribute?
- A. `lifecycle { ignore_changes = [tags] }`
- B. `lifecycle { ignore_changes = all }`
- C. `lifecycle { static_tags = true }`
- D. Remove the `tags` argument from the HCL manifest.

---

### Question 6
What is the function of the built-in `terraform_data` resource introduced in Terraform v1.4 (replacing `null_resource`)?
- A. It provisions local Docker containers.
- B. It stores arbitrary values in state without interacting with an external provider, frequently used to trigger lifecycle `replace_triggered_by` events.
- C. It connects to MySQL databases.
- D. It generates TLS private keys.

---

### Question 7
What is the consequence of configuring `create_before_destroy = true` on a resource with a strictly unique name (such as an S3 bucket or an IAM role) without using a dynamic prefix?
- A. Terraform automatically appends a random GUID to the name.
- B. The apply will fail because the new resource cannot be created with a duplicate name while the old resource still exists.
- C. The old resource is deleted first.
- D. AWS allows duplicate S3 bucket names.

---

### Question 8
When refactoring configurations by changing module source paths, what happens if an engineer simply edits the `source` attribute and runs `terraform apply` without running `terraform init`?
- A. Terraform automatically downloads the new module in memory.
- B. Terraform aborts with an error stating that the module is not installed, requiring `terraform init`.
- C. Terraform deletes all resources managed by that module.
- D. The plan executes using the old cached module code.

---

### Question 9
How does Terraform handle circular dependencies between resources in its directed acyclic graph (DAG)?
- A. It breaks the cycle arbitrarily at runtime.
- B. It fails during graph evaluation with a `Cycle error` before executing any cloud API calls.
- C. It converts circular resources into outputs.
- D. It executes the first resource twice.

---

### Question 10
In Terraform v1.5+, what happens when an `import` block targets a resource that does not exist in the remote cloud provider?
- A. Terraform creates the resource from scratch.
- B. Terraform plan reports an error stating that the resource could not be found with the specified ID.
- C. Terraform hangs indefinitely.
- D. The state file is deleted.

---

## Answer Key & Explanations

1. **A** — `create_before_destroy = true` reverses Terraform's default destroy-then-create ordering, spinning up the replacement resource before terminating the original.
2. **A** — `replace_triggered_by` accepts references to other resources or attributes, triggering a recreation whenever those references change.
3. **A** — `prevent_destroy = true` causes Terraform to throw an immediate error during plan if the execution plan would destroy the resource.
4. **B** — Targeted applies bypass dependency graph updates and can introduce subtle state divergence and orphaned infrastructure.
5. **A** — `ignore_changes = [tags]` instructs Terraform's diff engine to disregard any drift in the specified attributes.
6. **B** — `terraform_data` is a core-managed primitive that holds data and triggers lifecycle replacements without requiring the third-party `hashicorp/null` provider.
7. **B** — Cloud resources with globally or regionally unique names will collide if created before the predecessor is destroyed; use `name_prefix` instead.
8. **B** — Any alteration to module sources, versions, or providers requires `terraform init` to download and link packages into `.terraform/modules`.
9. **B** — Terraform builds a Directed Acyclic Graph (DAG); cycles cannot be resolved topologically, triggering a `Cycle:` graph error.
10. **B** — If the cloud provider API returns a 404/not found for the imported ID, Terraform plan fails with an import resolution error.
