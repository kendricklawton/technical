# Domain 3: Lifecycle Operations & Declarative Refactoring — Complete Study Guide

**Exam Weight:** 16% of the Terraform Infrastructure Engineer Professional (002) Exam  
**Target Mastery:** Resource `lifecycle` meta-arguments (`create_before_destroy` DAG propagation, `prevent_destroy`, `ignore_changes`, `replace_triggered_by`), `terraform_data` replacement orchestration (replacing `null_resource`), `-replace` vs. deprecated `taint`, Declarative State Refactoring (`moved`, `removed`, and `import` blocks with `for_each` & `-generate-config-out`), and Continuous Validation (`check` blocks vs. `precondition` & `postcondition`).

---

## 1. Resource `lifecycle` Meta-Arguments & DAG Graph Behavior

By default, when a resource change cannot be performed in-place (a **ForceNew** attribute change in the provider schema, such as changing an AWS Subnet's `cidr_block` or an EC2 instance's `ami`), Terraform executes a **Destroy-then-Create** sequence (`-/+`). The `lifecycle` block alters how Terraform's Directed Acyclic Graph (DAG) plans and executes these transitions.

```text
+-----------------------------------------------------------------------------------+
|          DEFAULT DESTROY-THEN-CREATE (-/+) vs. CREATE_BEFORE_DESTROY (+/-)        |
+-----------------------------------------------------------------------------------+
  DEFAULT (create_before_destroy = false):
    [1. Destroy Old Resource] ---> (Downtime Window) ---> [2. Create New Resource]

  ZERO-DOWNTIME (create_before_destroy = true):
    [1. Create New Resource] ---> [2. Update Downstream Attachments] ---> [3. Destroy Old]
    * CRITICAL DAG RULE: create_before_destroy propagates UPSTREAM to dependencies!
    * NAMING TRAP: Fixed unique names (e.g. name = "prod-sg") will collide on Step 1!
                   Always use name_prefix = "prod-sg-" when create_before_destroy = true.
```

### 1.1 `create_before_destroy = true`
* **Behavior:** Reverses the replacement order (`+/-`): Terraform provisions the replacement resource **first**, updates dependent downstream resources (such as an ALB target group or Auto Scaling Group) to point to the new resource ID, and **only then destroys** the prior resource.
* **Upstream DAG Propagation Rule (Heavily Tested!):**
  * If Resource B depends on Resource A (`A -> B`), and you set `create_before_destroy = true` on Resource A, Terraform **automatically propagates `create_before_destroy = true` to Resource B** in the dependency graph so that A and B can be replaced in a consistent order without dependency cycles.
  * **Naming Collision Pitfall:** Because both the old and new instances exist simultaneously in your cloud account during Step 1, hardcoding a globally or regionally unique identifier (e.g., `name = "prod-web-sg"` on `aws_security_group` or a fixed S3 `bucket` name) causes Step 1 to fail with `ResourceAlreadyExists` / `InvalidGroup.Duplicate`! Always pair `create_before_destroy = true` with **`name_prefix`** (or random pet/UUID suffixes rotated via `terraform_data`).

### 1.2 `prevent_destroy = true`
* **Behavior:** Acts as a compile/plan-time safety brake against accidental resource destruction. If any change would cause Terraform to plan a `destroy` (or `replace`) action for that resource, `terraform plan` aborts immediately with `Error: Instance cannot be destroyed`.
* **Critical Exam Limitations of `prevent_destroy = true`:**
  1. **Deleting the HCL Block Bypasses `prevent_destroy`!** If an engineer deletes the entire `resource "aws_db_instance" "prod" { ... }` block from `.tf` files and runs `terraform apply`, Terraform no longer sees the `lifecycle { prevent_destroy = true }` configuration in HCL and **WILL DESTROY THE RESOURCE**!
  2. **How to Safely Remove a Resource from Terraform Without Destroying It:** Use a declarative **`removed` block** with `lifecycle { destroy = false }` (Terraform 1.7+) instead of deleting the resource block directly.

### 1.3 `ignore_changes`
* **Behavior:** Instructs Terraform to ignore out-of-band modifications to specific resource attributes after initial creation:
  ```hcl
  resource "aws_autoscaling_group" "workers" {
    min_size         = 2
    max_size         = 20
    desired_capacity = 4 # Initial bootstrap capacity only

    lifecycle {
      # Allow external HPA / Dynamic Scaling policies to mutate desired_capacity and tags["LastScaled"]
      ignore_changes = [
        desired_capacity,
        tags["LastScaledTimestamp"],
      ]
      # Note: `ignore_changes = all` ignores ALL future attribute changes to this resource.
    }
  }
  ```

### 1.4 `replace_triggered_by` & Built-In `terraform_data` (Terraform 1.4+)
* **`replace_triggered_by`:** Forces a resource to be replaced whenever any referenced resource or specific resource attribute changes.
  * **Strict Rule:** `replace_triggered_by` can **ONLY** reference **managed resources** or **specific attributes of managed resources** (e.g., `aws_ecs_task_definition.app.revision` or `terraform_data.replacement_trigger.output`). It **cannot** directly reference `var.*` or `local.*`!
* **How to Trigger Replacement from a Variable or Local (`terraform_data`):**
  * Built directly into Terraform 1.4+ (requiring zero external providers and replacing the legacy `hashicorp/null` `null_resource`), `terraform_data` stores arbitrary values in state and bridges variables/locals to `replace_triggered_by`:
  ```hcl
  resource "terraform_data" "ami_version_trigger" {
    input = var.golden_ami_build_version
  }

  resource "aws_instance" "immutable_worker" {
    ami           = data.aws_ami.latest.id
    instance_type = "c6i.large"

    lifecycle {
      create_before_destroy = true
      replace_triggered_by  = [
        terraform_data.ami_version_trigger
      ]
    }
  }
  ```

---

## 2. Targeted Replacement (`-replace`) vs. Deprecated `terraform taint`

* **Why `terraform taint` Was Deprecated (Terraform 0.15.2+):**
  * `terraform taint <addr>` immediately mutated the remote `.tfstate` file before any `plan` was reviewed. If another engineer or CI job triggered an unrelated `apply` moments later, the tainted resource was unexpectedly destroyed and recreated without their knowledge!
* **The Modern `-replace` Workflow:**
  ```bash
  terraform plan -replace='aws_instance.worker["us-east-1b"]' -out=replace.tfplan
  terraform apply replace.tfplan
  ```
  * **Advantage:** `-replace` does **not** mutate remote state during planning; the replacement intent is scoped strictly to the generated execution plan (`replace.tfplan`) and goes through normal PR/policy review.

---

## 3. Declarative State Refactoring (`moved`, `removed`, `import`)

Imperative `terraform state mv`, `terraform state rm`, and `terraform import` commands break GitOps automation because they require manual CLI intervention outside of Pull Requests and fail when decomposing modules published to external consumers. Modern Terraform provides three **declarative, GitOps-native refactoring blocks**:

```text
+-----------------------------------------------------------------------------------+
|              DECLARATIVE STATE REFACTORING BLOCKS (GITOPS WORKFLOW)               |
+-----------------------------------------------------------------------------------+
  [1. moved { from = old_addr, to = new_addr }] (TF 1.1+)
       --> Renames resource, moves across child modules, or converts count -> for_each
       --> Zero cloud recreation; safe for published module version upgrades!

  [2. removed { from = old_addr, lifecycle { destroy = false } }] (TF 1.7+)
       --> Removes resource block from HCL AND forgets it from .tfstate during apply
       --> Leaves physical cloud resource running untouched!

  [3. import { for_each = ..., to = addr, id = cloud_id }] (TF 1.5+ / 1.6+)
       --> Imports brownfield cloud resources into state during standard plan/apply
       --> Pair with `terraform plan -generate-config-out=generated.tf`!
```

### 3.1 `moved` Blocks (Terraform 1.1+)
* **Capabilities:**
  1. Rename a resource within the same module (`aws_s3_bucket.old` $\rightarrow$ `aws_s3_bucket.new`).
  2. Move a resource from the root module into a child module (`aws_vpc.main` $\rightarrow$ `module.network.aws_vpc.this`), or between two child modules.
  3. Convert a resource or module from `count` to `for_each` (`aws_subnet.private[0]` $\rightarrow$ `aws_subnet.private["us-east-1a"]`).
* **Chaining `moved` Blocks for Module Consumers:**
  * If you publish a shared module `v2.0.0` where an internal resource was renamed, including the `moved` block inside the module's `.tf` code allows all 100 downstream teams to upgrade from `v1.x` to `v2.0.0` with **zero manual `terraform state mv` commands**! If you rename it again in `v2.1.0`, keep the prior `moved` block and add a second `moved` block chaining the old target to the new target.
* **Limitation:** A `moved` block **cannot move resources across two separate state files/workspaces** (it operates strictly within a single state boundary).

### 3.2 `removed` Blocks (Terraform 1.7+)
When retiring a resource from Terraform management while keeping the underlying cloud infrastructure alive (replaces `terraform state rm`):
```hcl
# Delete the original `resource "aws_s3_bucket" "legacy_logs" { ... }` block from your .tf file
# and replace it with this declarative `removed` block:
removed {
  from = aws_s3_bucket.legacy_logs

  lifecycle {
    destroy = false # Instructs Terraform to remove from .tfstate WITHOUT deleting the S3 bucket!
  }
}
```

### 3.3 Config-Driven `import` Blocks (Terraform 1.5+ & `for_each` in 1.6+)
Replaces imperative `terraform import <addr> <id>`:
```hcl
variable "legacy_buckets" {
  type = map(string)
  default = {
    audit    = "corp-prod-audit-logs-2023"
    billing  = "corp-prod-billing-exports-2023"
    security = "corp-prod-security-findings-2023"
  }
}

import {
  for_each = var.legacy_buckets
  to       = aws_s3_bucket.adopted[each.key]
  id       = each.value
}
```
* **Automatic HCL Code Generation (`-generate-config-out`):**
  ```bash
  # Generate the matching `resource "aws_s3_bucket" "adopted"` HCL code automatically:
  terraform plan -generate-config-out=adopted_buckets.tf
  ```
  * **Exam Rule:** The target file passed to `-generate-config-out` **must NOT already exist** on disk (Terraform refuses to overwrite an existing file). After generation, review/prune read-only or conflicting default attributes in `adopted_buckets.tf`, then run `terraform plan` to confirm `0 to add, 0 to change, 0 to destroy, 3 to import`.
  * **Plan-Time Invariant:** The `id` expression inside an `import` block must evaluate to a string known at `plan` time!

---

## 4. Continuous Validation: `precondition`, `postcondition` & `check` Blocks

### 4.1 Custom `precondition` & `postcondition` Blocks (Hard Blockers)
Placed inside the `lifecycle` block of a `resource`, `data` source, or `output`:
```hcl
data "aws_ami" "eks_worker" {
  most_recent = true
  owners      = ["amazon"]
  filter {
    name   = "name"
    values = ["amazon-eks-node-1.31-*"]
  }

  lifecycle {
    # Evaluated AFTER reading the data source; uses `self` to inspect returned attributes:
    postcondition {
      condition     = self.architecture == "x86_64" && self.root_device_type == "ebs"
      error_message = "Selected AMI must use x86_64 architecture and an EBS root volume."
    }
  }
}

resource "aws_instance" "app" {
  ami           = data.aws_ami.eks_worker.id
  instance_type = var.instance_type
  subnet_id     = var.subnet_id

  lifecycle {
    # Evaluated BEFORE creating/updating the instance:
    precondition {
      condition     = !startswith(var.instance_type, "t2.")
      error_message = "Legacy t2 burstable instances are prohibited in production."
    }

    # Evaluated AFTER creating/refreshing the instance; uses `self` to verify live cloud state:
    postcondition {
      condition     = self.ebs_optimized == true
      error_message = "Provisioned EC2 instance must have EBS optimization active."
    }
  }
}
```
* **Behavior on Failure:** If a `precondition` or `postcondition` evaluates to `false`, Terraform **immediately fails the run with an Error** and blocks any downstream dependent resources in the DAG from executing!

### 4.2 Top-Level `check` Blocks (Non-Blocking Health Assertions — Terraform 1.5+)
Unlike `precondition`/`postcondition` (which halt execution on failure), a top-level `check` block executes **at the very end of a `plan` or `apply`** to validate real-world end-to-end health (such as HTTP status codes, TLS certificate validity, or DNS resolution) and emits a **Warning** on failure without aborting the run!

```hcl
check "api_gateway_live_health_check" {
  # Scoped data source evaluated ONLY for this check block after infrastructure is applied:
  data "http" "healthz" {
    url      = "https://${aws_lb.public.dns_name}/healthz"
    insecure = false

    retry {
      attempts     = 3
      min_delay_ms = 2000
    }
  }

  assert {
    condition     = data.http.healthz.status_code == 200
    error_message = "ALB health endpoint returned HTTP ${data.http.healthz.status_code} instead of 200 OK!"
  }

  assert {
    condition     = jsondecode(data.http.healthz.response_body).db_status == "connected"
    error_message = "Application health check reports database connectivity failure!"
  }
}
```

---

## 5. Domain 3 Production Scenario & Exam Trap Matrix

* **Scenario 1: Relying on `prevent_destroy = true` when deleting a resource block from Git**
  * **Trap:** Assuming that deleting a `resource` block containing `prevent_destroy = true` will fail the next `terraform plan`.
  * **Reality:** Once the HCL block is deleted from `.tf` files, `prevent_destroy` is no longer present in the configuration, so Terraform plans and executes a full destroy! Use a `removed` block with `lifecycle { destroy = false }` to safely orphan a resource from state.
* **Scenario 2: Passing `var.app_version` directly into `replace_triggered_by`**
  * **Trap:** Writing `replace_triggered_by = [var.app_version]`.
  * **Reality:** `replace_triggered_by` only accepts managed resource addresses or resource attributes. Wrap `var.app_version` inside `resource "terraform_data" "ver" { input = var.app_version }` and reference `terraform_data.ver`.
* **Scenario 3: Choosing between `postcondition` and `check` block for HTTP endpoint verification**
  * **Trap:** Using a `postcondition` on an ALB when a transient HTTP 502 during initial container warmup shouldn't brick the entire state apply and block all other unrelated outputs.
  * **Reality:** `postcondition` is a **hard blocker** that halts the DAG and marks the run as failed. A `check` block with a scoped `data "http"` block is **non-blocking**—it completes the apply, commits state, and surfaces a continuous health warning in CLI and HCP Terraform Health Assessments.

---

## 6. Platform Engineering Deep Dive: Two-Stage Cluster & CRD Bootstrapping Trap

A classic Platform Engineering failure mode tested on the Professional exam occurs when provisioning a Kubernetes cluster (`aws_eks_cluster` or `google_container_cluster`), installing Custom Resource Definitions (CRDs via `helm_release` such as Argo CD, Cert-Manager, or Karpenter), and creating custom resources (`kubernetes_manifest`) inside the **same single Terraform configuration**:

```text
+-----------------------------------------------------------------------------------+
|       THE KUBERNETES_MANIFEST PLAN-TIME CRD SCHEMA DEADLOCK & RESOLUTION          |
+-----------------------------------------------------------------------------------+
  ANTI-PATTERN (Single Workspace with `kubernetes_manifest` + `aws_eks_cluster`):
    [terraform plan] ---> `kubernetes_manifest` attempts to connect to EKS API
                          during PLAN to validate the CRD OpenAPI schema!
                     ---> FAILS immediately: EKS cluster & CRD do not exist yet!

  RESOLUTION PATTERNS:
    1. Split into Layered Workspaces / Stack Components:
       * Layer 1 (Cluster & Core Helm CRDs) -> Layer 2 (Custom Resources / GitOps)
    2. Or use `kubectl_manifest` / `helm_release` (which defer API schema validation
       until the apply phase rather than querying the cluster during plan).
```

* **Why `kubernetes_manifest` Fails During Initial `terraform plan`:**
  * The official `hashicorp/kubernetes` provider's `kubernetes_manifest` resource performs **Server-Side OpenAPI Schema Validation during the `plan` phase**. If the EKS cluster or the CRD (`Certificate`, `Application`, `NodePool`) is being created in the same run, the Kubernetes API server does not have the CRD registered at `plan` time, causing `terraform plan` to abort!
* **Architectural Solutions:**
  1. **Terraform Stacks (`*.tfstack.hcl`) or Layered Workspaces:** Separate Cluster Infrastructure (`component "eks"`) from Cluster Add-ons (`component "addons"`).
  2. **Bootstrap via GitOps Handoff:** Have Terraform provision the EKS cluster and install the **Argo CD / Flux** Helm release (`helm_release` does not query the K8s API during `plan`), and let Argo CD reconcile all CRDs and `ApplicationSet` manifests asynchronously.

---

## 7. Provisioners, `connection` Blocks & Destroy-Time Restrictions

While HashiCorp strongly recommends cloud-native mechanisms (`user_data` / `cloud-init`, SSM Run Command, Packer golden images, or `terraform_data`) over provisioners (`local-exec`, `remote-exec`, `file`), the exam tests specific edge cases and strict syntax rules for **Destroy-Time Provisioners**:

```hcl
resource "aws_instance" "clustered_node" {
  ami           = var.node_ami
  instance_type = "m6i.large"

  # Store deregistration endpoint in tags or resource attributes so `self` can access it!
  tags = {
    ConsulClusterAddr = var.consul_cluster_address
  }

  # 1. Creation-time provisioner (Default: when = create)
  provisioner "local-exec" {
    when       = create
    on_failure = fail # Default is `fail` (halts apply & marks resource tainted); alternative is `continue`
    command    = "echo 'Registered ${self.private_ip}' >> /tmp/nodes.log"
  }

  # 2. Destroy-time provisioner (when = destroy)
  provisioner "local-exec" {
    when    = destroy
    # CRITICAL EXAM RULE: Destroy-time provisioners can ONLY reference `self.*`, `count.index`, or `each.key`!
    # Referencing `var.consul_cluster_address` or `data.*` or another resource directly here is FORBIDDEN!
    command = "curl -X PUT http://${self.tags.ConsulClusterAddr}:8500/v1/agent/leave -d '{\"node\":\"${self.private_ip}\"}'"
  }
}
```

* **Three Mandatory Exam Rules for Destroy-Time Provisioners (`when = destroy`):**
  1. **`self` Reference Restriction:** Inside a `when = destroy` provisioner (and its `connection` block), expressions **cannot** reference `var.*`, `local.*`, `data.*`, or any other resource (`aws_vpc.main.id`). They may **only** reference **`self.<attribute>`**, **`count.index`**, or **`each.key`**! Why? Because a resource might be destroyed *after* its block was already deleted from `.tf` files (where only the resource's prior `.tfstate` attributes in `self` still exist).
  2. **Interaction with `create_before_destroy`:** If a resource has `lifecycle { create_before_destroy = true }`, any `when = destroy` provisioner still runs during the destruction of the old instance (*after* the replacement instance has been created).
  3. **Removing the Resource Block Skips Destroy Provisioners:** If you delete the entire `resource` block from your `.tf` files, the inline `provisioner { when = destroy }` block is also gone from the configuration and **will not run**! To ensure a destroy-time provisioner runs before removing a resource, first set `count = 0` (or remove its key from `for_each`), run `terraform apply`, and only then delete the HCL block.

