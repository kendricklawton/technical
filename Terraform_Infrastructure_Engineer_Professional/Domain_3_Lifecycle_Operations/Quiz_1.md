# Domain 3: Infrastructure Lifecycle and Operations — Quiz 1

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 15 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

---

### Question 1
An engineer updates the AMI ID of an auto-scaling group launch template. The launch template has `create_before_destroy = true`. However, the security group referenced by the launch template does NOT have `create_before_destroy = true`. When the security group is also updated, what error occurs?

A. The plan passes without errors during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. AWS automatically duplicates the security group through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. A cycle error or dependency inversion error: Terraform cannot destroy the old security group until the old launch template is destroyed, but the new launch template requires the new security group first. To fix this, any dependency of a `create_before_destroy` resource must also have `create_before_destroy = true`.
D. Terraform ignores the security group update while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** The `create_before_destroy` attribute must be propagated to upstream dependencies; otherwise, Terraform cannot order the creation of new resources ahead of destroying existing dependencies.
- **Why A is Incorrect:** `The plan passes without errors during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `AWS automatically duplicates the security group through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Terraform ignores the security group update while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 2
What happens if you run `terraform destroy` against a configuration containing a resource with `lifecycle { prevent_destroy = true }`?

A. Terraform immediately halts execution during the planning phase with an error, preventing any destruction.
B. Terraform prompts for an administrative password by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.
C. Terraform ignores `prevent_destroy` during a full destroy during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
D. Terraform deletes all resources except the protected one while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `prevent_destroy` acts as a fail-safe that rejects any plan attempting to destroy the resource.
- **Why B is Incorrect:** `Terraform prompts for an administrative password by executing an in-place provider schema refresh before committing the updated resource attributes to remote state.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Terraform ignores `prevent_destroy` during a full destroy during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Terraform deletes all resources except the protected one while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 3
An engineer uses a declarative `import` block in Terraform 1.5+:
```hcl
import {
  to = aws_s3_bucket.logs
  id = "prod-corp-access-logs-2026"
}
```
What CLI flag can be passed to `terraform plan` to automatically generate the corresponding resource HCL code if the file does not yet exist?

A. `-auto-import` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. `-export-resource` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. `-write-hcl` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `-generate-config-out=bucket.tf`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** `-generate-config-out=PATH` inspects the imported resource via provider APIs and synthesizes the HCL configuration.
- **Why A is Incorrect:** ``-auto-import` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``-export-resource` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** ``-write-hcl` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 4
In Kubernetes environments, a Pod Autoscaler constantly alters the `desired_capacity` of an AWS Auto Scaling Group. Every time the CI pipeline runs `terraform plan`, it reports that `desired_capacity` has drifted. How do you prevent Terraform from constantly reverting the autoscaler's changes?

A. Set `desired_capacity = null` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Add `lifecycle { ignore_changes = [desired_capacity] }` to the Auto Scaling Group resource.
C. Delete the Auto Scaling Group from state through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Remove the autoscaler from the Kubernetes cluster while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `ignore_changes` instructs Terraform's plan engine to ignore differences between state and code for specified attributes.
- **Why A is Incorrect:** `Set `desired_capacity = null` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Delete the Auto Scaling Group from state through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Remove the autoscaler from the Kubernetes cluster while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 5
Why should saved plan files (`terraform plan -out=tfplan`) always be used when running Terraform in automated CI/CD pipelines?

A. It guarantees that the exact changes reviewed and approved during the PR plan are applied during the deployment step, eliminating race conditions from concurrent changes or external drift.
B. Saved plan files are encrypted with RSA 4096-bit keys during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. Saved plan files execute 50% faster through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Plan files automatically roll back on failure while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** Specifying a plan file ensures deterministic, atomic application of the exact recorded plan.
- **Why B is Incorrect:** `Saved plan files are encrypted with RSA 4096-bit keys during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Saved plan files execute 50% faster through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Plan files automatically roll back on failure while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 6
What is the purpose of the `.terraform.lock.hcl` file in source control?

A. It locks the state file in S3 during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. It prevents git merge conflicts through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. It pins exact provider versions and cryptographic package hashes (checksums) across developer workstations and CI runners.
D. It stores API access tokens while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `.terraform.lock.hcl` acts as a security supply-chain barrier by pinning exact provider releases and hashes.
- **Why A is Incorrect:** `It locks the state file in S3 during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** `It prevents git merge conflicts through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `It stores API access tokens while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 7
You have multiple target operating systems in your engineering team (macOS ARM64, Linux AMD64, Windows). When initializing a new provider, only the local architecture's checksum is recorded in `.terraform.lock.hcl`. What command updates the lock file with checksums for all supported OS architectures?

A. `terraform init -all-platforms` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. `terraform get -update` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.
C. `terraform lock refresh` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `terraform providers lock -platform=linux_amd64 -platform=darwin_arm64 -platform=windows_amd64`

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: D**

- **Why D is Correct:** `terraform providers lock -platform=...` fetches package hashes for multiple platform targets.
- **Why A is Incorrect:** ``terraform init -all-platforms` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``terraform get -update` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why C is Incorrect:** ``terraform lock refresh` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 8
What is the effect of setting `destroy = false` inside a `removed` block?

A. Terraform deletes the resource from cloud and recreates it during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. Terraform removes the resource from the `.tfstate` file, leaving the real infrastructure running untouched in the cloud.
C. Terraform permanently prevents the resource from ever being destroyed through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. Terraform sets the resource to read-only mode while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Declarative `removed { lifecycle { destroy = false } }` removes resources from state safely.
- **Why A is Incorrect:** `Terraform deletes the resource from cloud and recreates it during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** `Terraform permanently prevents the resource from ever being destroyed through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** `Terraform sets the resource to read-only mode while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 9
An infrastructure pipeline fails with `HTTP 429 Too Many Requests` when creating 200 DNS records concurrently. What Terraform CLI flag addresses this directly?

A. `-timeout=60s` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
B. `-retry=true` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
C. `-parallelism=5` (reducing concurrent graph walking operations from default 10)
D. `-slow-mode` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: C**

- **Why C is Correct:** `-parallelism=N` limits concurrent API operations to respect cloud rate limits.
- **Why A is Incorrect:** ``-timeout=60s` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why B is Incorrect:** ``-retry=true` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``-slow-mode` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 10
Which `lifecycle` meta-argument allows you to trigger the recreation of a virtual machine when its bootstrap script hash changes?

A. `replace_triggered_by`
B. `create_before_destroy` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.
C. `precondition` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.
D. `ignore_changes` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** `replace_triggered_by` forces a replacement plan when referenced objects change.
- **Why B is Incorrect:** ``create_before_destroy` during the initial graph evaluation phase while preserving existing state serial metadata across workspaces.` fails because it violates Terraform's DAG evaluation rules or relies on unsupported CLI/HCL behavior.
- **Why C is Incorrect:** ``precondition` through the local provider plugin RPC channel without requiring additional cloud API authentication calls.` introduces state drift, security exposure, or unnecessary resource replacement in production.
- **Why D is Incorrect:** ``ignore_changes` while bypassing standard dependency graph ordering and deferring validation until the post-apply phase.` is not a valid mechanism for solving the architectural constraint described in the scenario.

</details>

---

### Question 11
An engineer configures a destroy-time provisioner inside an `aws_instance.cache_node` resource to deregister the node from an internal service discovery API when the instance is terminated:
```hcl
provisioner "local-exec" {
  when    = destroy
  command = "curl -X DELETE https://${var.registry_host}/nodes/${self.private_ip}"
}
```
When running `terraform validate`, Terraform rejects this configuration with an error. Why is this invalid, and how must it be fixed?

A. Destroy-time provisioners only work with `remote-exec`, not `local-exec`; switch the provisioner type to `remote-exec`.
B. Destroy-time provisioners cannot reference `var.*`, `local.*`, `data.*`, or other resources directly because the resource may be destroyed after those blocks are removed; store `var.registry_host` in the resource's `tags` (or `triggers_replace`) and reference `${self.tags.RegistryHost}`.
C. Destroy-time provisioners cannot reference `self.private_ip`; replace `self.private_ip` with `aws_instance.cache_node.private_ip`.
D. `when = destroy` must be placed inside the `lifecycle` block rather than inside the `provisioner` block.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Terraform strictly forbids referencing `var.*`, `local.*`, `data.*`, or external resources inside a `when = destroy` provisioner (only `self.*`, `count.index`, and `each.key` are permitted). Storing the endpoint in the resource's own attributes/tags allows `self.tags.*` to access it from state at destroy time.
- **Why A, C, and D are Incorrect:** `local-exec` supports `when = destroy`, `self.*` is the required way to access resource attributes, and `when = destroy` belongs inside the `provisioner` block.

</details>

---

### Question 12
In a single Terraform root module, an engineer defines an `aws_eks_cluster.main` resource, a `helm_release.cert_manager` resource that installs the Cert-Manager CRDs, and a `kubernetes_manifest.cluster_issuer` resource that creates a `ClusterIssuer` custom resource. Even with `depends_on = [helm_release.cert_manager]`, running `terraform plan` against a brand-new environment fails immediately. Why?

A. `depends_on` cannot be used between resources belonging to two different providers (`helm` and `kubernetes`).
B. The `kubernetes_manifest` resource queries the live Kubernetes API server during the `plan` phase to validate the `ClusterIssuer` CRD OpenAPI schema, which fails because neither the EKS cluster nor the CRD exists yet during the initial plan.
C. `helm_release` executes asynchronously in the background and does not wait for Kubernetes Deployments to become ready.
D. The `kubernetes` provider requires `create_before_destroy = true` whenever CRDs are modified.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** `kubernetes_manifest` requires live access to the cluster's OpenAPI/CRD discovery endpoint during `terraform plan` to validate custom resource schemas. When the cluster or CRD is being created in the same run, the API does not exist at `plan` time, causing an immediate failure even with `depends_on`. Separate cluster/CRD creation into an upstream workspace or Terraform Stack component.
- **Why A, C, and D are Incorrect:** `depends_on` works across all providers; the failure happens during `plan` before `helm_release` even runs.

</details>

---

### Question 13
You have an `aws_security_group.alb` resource attached to an `aws_lb.frontend` resource. You add `lifecycle { create_before_destroy = true }` to `aws_security_group.alb` and keep `name = "prod-alb-sg"` hardcoded. What happens when you change the `vpc_id` or `description` of `aws_security_group.alb` (forcing replacement)?

A. Terraform creates the new Security Group first, but the AWS API rejects the creation with `InvalidGroup.Duplicate` because a Security Group named `"prod-alb-sg"` already exists in the VPC; furthermore, `create_before_destroy` automatically propagates downstream to `aws_lb.frontend`.
B. Terraform temporarily renames the existing Security Group in AWS to `"prod-alb-sg-old"`, creates the new one, and deletes the old one.
C. Terraform ignores `create_before_destroy = true` whenever a static `name` attribute is detected on the resource.
D. Terraform destroys `aws_lb.frontend` first, then destroys `aws_security_group.alb`, and recreates both.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: A**

- **Why A is Correct:** With `create_before_destroy = true`, the replacement resource is provisioned *before* the old one is deleted. If a resource uses a fixed unique `name` instead of `name_prefix = "prod-alb-sg-"`, the cloud API fails with a duplicate name collision! Additionally, `create_before_destroy` propagates through the dependency graph.
- **Why B, C, and D are Incorrect:** Terraform never auto-renames existing cloud resources to avoid collisions, nor does it ignore `create_before_destroy`.

</details>

---

### Question 14
You want to verify that after `terraform apply` completes, your newly deployed internal API Gateway returns HTTP `200 OK` and a valid JSON payload, and you want HCP Terraform Continuous Health Assessments to monitor this invariant daily—WITHOUT causing transient HTTP 503 startup delays to abort the `terraform apply` or block state commits. Which construct should you use?

A. A resource `lifecycle { postcondition { ... } }` block attached to the `aws_apigatewayv2_stage` resource.
B. A top-level `check` block containing a scoped `data "http"` block and `assert` blocks.
C. A `local-exec` provisioner running `curl --fail` with `on_failure = fail`.
D. A variable `validation` block querying `data.http.api_health.status_code == 200`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Top-level `check` blocks execute at the end of `plan`/`apply` and during scheduled HCP Terraform Health Assessments. If an `assert` inside a `check` block fails, Terraform emits a non-blocking **Warning** after committing state rather than halting the run.
- **Why A & C are Incorrect:** Both `postcondition` and `on_failure = fail` are hard blockers that fail the run and halt downstream execution.
- **Why D is Incorrect:** Variable `validation` blocks validate input arguments before infrastructure is provisioned, not post-apply live HTTP endpoints.

</details>

---

### Question 15
You published version `1.0.0` of an internal module containing `resource "aws_s3_bucket" "logs"`. In version `1.1.0`, you renamed the resource to `resource "aws_s3_bucket" "audit_logs"` and added a `moved { from = aws_s3_bucket.logs, to = aws_s3_bucket.audit_logs }` block. Now in version `1.2.0`, you are moving `aws_s3_bucket.audit_logs` into a nested child module `module.storage`. How should `moved` blocks be maintained in `1.2.0` so teams upgrading directly from `1.0.0` -> `1.2.0` AND teams upgrading from `1.1.0` -> `1.2.0` both migrate cleanly?

A. Delete the `1.1.0` `moved` block and replace it with a single `moved` block from `aws_s3_bucket.logs` to `module.storage.aws_s3_bucket.audit_logs`.
B. Retain the first `moved` block (`aws_s3_bucket.logs` -> `aws_s3_bucket.audit_logs`) AND add a second chained `moved` block (`aws_s3_bucket.audit_logs` -> `module.storage.aws_s3_bucket.audit_logs`).
C. Remove all `moved` blocks and require consumers to run `terraform init -migrate-state` before upgrading to `1.2.0`.
D. Use a `removed` block for `aws_s3_bucket.logs` and an `import` block inside `module.storage`.

<details>
<summary>Show Answer & Explanation</summary>

**Correct Answer: B**

- **Why B is Correct:** Terraform evaluates chained `moved` blocks transitively (`A -> B` followed by `B -> C`). Retaining the first `moved` block and adding the second allows consumers on `1.0.0` (`A -> B -> C`) and consumers on `1.1.0` (`B -> C`) to upgrade to `1.2.0` seamlessly without destroying the S3 bucket.
- **Why A is Incorrect:** Deleting the `1.1.0` `moved` block and only mapping `A -> C` would break consumers who already upgraded to `1.1.0` (whose state currently holds address `B`!).
- **Why C & D are Incorrect:** `-migrate-state` migrates backends, not resource addresses, and `import` requires runtime bucket IDs.

</details>

---
