# Domain 6: Terraform State Management — Study Guide (004)

> **Official 004 Blueprint Focus:** Objective 6 — Terraform State Management (`6a`, `6b`, `6c`, `6d`).  
> **Tested Version:** Terraform 1.12+ (Includes **Declarative `moved` & `removed` blocks, `-refresh-only`, and S3 Native Locking**!)

---

## 1. Describe the Local Backend & CLI Workspaces (`6a`)

By default, if no `backend` or `cloud` block is declared inside `terraform { ... }`, Terraform uses the **`local` backend**:
- Stores the current state in a plaintext JSON file named `terraform.tfstate` in the current working directory.
- Whenever `terraform apply` modifies state, it writes the previous state snapshot to `terraform.tfstate.backup`.
- **Why Local Backend Is Unsafe for Teams:** No remote access, no centralized audit trail, and high risk of developers committing `terraform.tfstate` (which contains plaintext secrets) to Git.

### 1.1 CLI Workspaces with the Local Backend
- Every directory begins with a single workspace named `default` (which uses `./terraform.tfstate`).
- Creating additional CLI workspaces stores separate state files under `./terraform.tfstate.d/<WORKSPACE_NAME>/terraform.tfstate`.
- Referenced in HCL via `${terraform.workspace}` (e.g., `count = terraform.workspace == "prod" ? 3 : 1`).

### 1.2 CLI Workspace Command Suite
```
Command                                     Operational Effect & 004 Exam Rules
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────
terraform workspace list                    Lists all existing CLI workspaces in the working directory (* marks active).
terraform workspace show                    Displays the name of the currently active workspace.
terraform workspace new <NAME>              Creates a new workspace and automatically switches to it.
terraform workspace select <NAME>           Switches context to an existing workspace.
terraform workspace delete <NAME>           Deletes a workspace.
                                            RULES:
                                            1. Cannot delete the 'default' workspace!
                                            2. Cannot delete the workspace you are currently in (switch out first)!
                                            3. Workspace state must be empty, or use -force to force deletion.
```

### 1.3 CLI Workspaces vs. Separate Directories vs. HCP Workspaces
- **CLI Workspaces:** Use the **exact same code and backend** for multiple states. Ideal for ephemeral feature branches or testing slight configuration variants. **Not recommended** for completely separate environments (`dev` vs. `prod`) that require different IAM access controls, variable definitions, and distinct backend credentials.
- **Separate Directories:** Use distinct directories with separate backend configurations and variable sets. Provides complete isolation of credentials and state.
- **HCP Terraform Workspaces:** Full isolated cloud environments with dedicated VCS branches, environment/Terraform variables, RBAC, speculative plans, and run histories.

---

## 2. Describe State Locking (`6b`)

**State Locking** prevents two engineers or CI pipelines from running state-mutating operations (`plan`, `apply`, `destroy`, `import`, `state mv/rm`) simultaneously against the same state file:
- **Automatic Acquisition:** Before executing an operation that could write state, Terraform automatically requests a lock from the backend. If the state is already locked, Terraform fails immediately with a `Lock Info` error showing the **Lock ID**, **Who** holds it, and **When** it was created.
- **`-lock=false` (Dangerous):** Disables state locking for a command; never recommended in collaborative environments.
- **`-lock-timeout=<DURATION>` (e.g., `-lock-timeout=5m`):** Instructs Terraform to wait and retry acquiring the lock for the specified duration instead of failing immediately.
- **Releasing an Orphaned Lock (`terraform force-unlock <LOCK_ID>`):**
  - If a CI runner crashes or network connectivity drops mid-apply, the state lock may remain orphaned in the backend.
  - After verifying that no other Terraform process is genuinely running, an administrator clears the lock using:
    ```bash
    terraform force-unlock 8f9d2a1b-3c4e-5f6a-7b8c-9d0e1f2a3b4c
    ```

---

## 3. Configure Remote State Using the `backend` Block (`6c`)

A **remote backend** stores `terraform.tfstate` in a shared network store (Amazon S3, Azure Blob Storage, Google Cloud Storage, HashiCorp Consul, or Postgres).

```hcl
terraform {
  backend "s3" {
    bucket       = "acme-prod-terraform-state"
    key          = "network/vpc/terraform.tfstate"
    region       = "us-east-1"
    encrypt      = true
    use_lockfile = true # Native S3 state locking (TF 1.10+ / 1.12+) via .tflock conditional writes!
    # Legacy alternative: dynamodb_table = "terraform-state-locks" (Partition Key: LockID)
  }
}
```

### 3.1 Crucial Rules for `backend` Blocks on the 004 Exam
1. **No Variables or Locals Inside `backend` Blocks:** A `backend` block **cannot** reference `var.*`, `local.*`, or `data.*` expressions because the backend is initialized during `terraform init` before variables or data sources are evaluated!
2. **Partial Backend Configuration:** To keep sensitive backend details or environment-specific keys out of version-controlled `.tf` files, omit those arguments from the `backend` block and pass them at initialization via:
   - `terraform init -backend-config="backend.hcl"` (file), or
   - `terraform init -backend-config="key=prod/terraform.tfstate"` (CLI flag), or
   - Provider-specific environment variables (e.g., `AWS_ACCESS_KEY_ID`, `AWS_REGION`).
3. **Switching or Migrating Backends:**
   - `terraform init -migrate-state`: Copies existing state from the old backend to the new backend interactively.
   - `terraform init -reconfigure`: Initializes the new backend configuration from scratch without copying existing state.

### 3.2 Sharing State Between Workspaces: `terraform_remote_state` Data Source
To access the root module outputs of another workspace (e.g., an Application workspace reading VPC and Subnet IDs from a separate Network workspace):

```hcl
data "terraform_remote_state" "network" {
  backend = "s3"
  config = {
    bucket = "acme-prod-terraform-state"
    key    = "network/vpc/terraform.tfstate"
    region = "us-east-1"
  }
}

resource "aws_instance" "app" {
  subnet_id = data.terraform_remote_state.network.outputs.public_subnet_ids[0]
}
```

> [!WARNING]
> **Security Implication of `terraform_remote_state`:**
> The `terraform_remote_state` data source downloads the **entire remote state file** of the target workspace into memory. If that target state contains sensitive values or database credentials, the calling configuration gains access to them. In HCP Terraform, prefer **`tfe_outputs`**, which exposes **only** explicitly declared root output values without granting full state file access.

---

## 4. Manage Resource Drift and Terraform State (`6d` — Expanded in 004!)

### 4.1 Managing Configuration Drift (`-refresh-only` vs. Deprecated `terraform refresh`)
- **Why `terraform refresh` is deprecated:** The standalone `terraform refresh` command immediately overwrote the state file without giving the operator a chance to review what changed—if cloud credentials were misconfigured (e.g., pointing to an empty region), `terraform refresh` could accidentally wipe every resource from the state file!
- **Modern Standard (`terraform plan -refresh-only` & `terraform apply -refresh-only`):**
  - Displays a safe interactive diff showing how real-world cloud changes (drift) would update the state file, and prompts for confirmation before writing to `terraform.tfstate`.
  - **Exam Note:** `-refresh-only` updates the **state file only**; it never modifies real-world cloud infrastructure. To revert real-world drift back to match your `.tf` code, run a normal `terraform plan` and `terraform apply`.

### 4.2 Declarative State Refactoring: `moved` and `removed` Blocks (004 Focus!)
In Terraform 1.12+, HashiCorp recommends declarative HCL blocks over imperative `terraform state mv` and `terraform state rm` CLI commands when refactoring shared modules, because HCL blocks travel through Git Pull Requests and apply automatically across all team workspaces:

```hcl
# 1. Declaratively rename a resource or move it into a child module without destroying it
moved {
  from = aws_security_group.app
  to   = aws_security_group.web_tier
}

# 2. Declaratively remove a resource from Terraform state without destroying the live cloud object
#    (You also delete or comment out the original 'resource "aws_s3_bucket" "legacy" {}' block)
removed {
  from = aws_s3_bucket.legacy

  lifecycle {
    destroy = false # false = untrack from state ONLY (do not delete bucket in AWS!)
                    # true  = destroy the bucket in AWS and remove from state
  }
}
```
