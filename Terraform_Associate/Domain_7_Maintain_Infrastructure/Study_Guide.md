# Domain 7: Maintain Infrastructure with Terraform — Study Guide (004)

> **Official 004 Blueprint Focus:** Objective 7 — Maintain Infrastructure with Terraform (`7a`, `7b`, `7c`).  
> **Tested Version:** Terraform 1.12+ (Includes **Config-Driven `import` blocks with `-generate-config-out`, `terraform state` CLI, and `TF_LOG`**!)

---

## 1. Import Existing Infrastructure Into Your Terraform Workspace (`7a`)

Organizations frequently have brownfield cloud resources created manually ("ClickOps") or by legacy scripts that need to be brought under Terraform management without downtime. Terraform 1.12+ supports **two ways** to import existing infrastructure:

### 1.1 Method 1: Config-Driven Declarative `import` Block (Modern 004 Standard — TF 1.5+)
The declarative `import` block is part of your `.tf` code, works seamlessly with Git Pull Requests and HCP Terraform speculative plans, supports `for_each` for bulk imports, and can **automatically generate the matching HCL `resource` configuration**:

```hcl
# Step 1: Write the top-level import block in main.tf or imports.tf
import {
  to = aws_s3_bucket.existing_logs
  id = "corp-audit-logs-prod-2026"
}
```
```bash
# Step 2: Auto-generate the HCL resource block from the live cloud object!
terraform plan -generate-config-out=generated_resources.tf

# Step 3: Review generated_resources.tf, run 'terraform plan' to verify 0 unintended changes,
#         and run 'terraform apply' to commit the imported resource into terraform.tfstate.
```
* **Bulk Import with `for_each`:**
  ```hcl
  import {
    for_each = {
      "web" = "sg-01111111111111111"
      "db"  = "sg-02222222222222222"
    }
    to = aws_security_group.service[each.key]
    id = each.value
  }
  ```

### 1.2 Method 2: Imperative `terraform import` CLI Command
```bash
# Prerequisite: You MUST manually write an empty/matching 'resource "aws_s3_bucket" "existing_logs" {}' block in .tf FIRST!
terraform import aws_s3_bucket.existing_logs corp-audit-logs-prod-2026
```

| Feature | Declarative `import` Block (`import { to = ... id = ... }`) | Imperative `terraform import <ADDR> <ID>` CLI |
| :--- | :--- | :--- |
| **When State Is Modified** | During `terraform apply` (can be safely previewed in `terraform plan`!) | **Immediately** mutates state without a plan preview |
| **Generates HCL Code?** | **Yes**, via `terraform plan -generate-config-out=FILE.tf` | **No**; you must write the `resource` block manually before running `terraform import` |
| **Bulk / `for_each` Support** | **Yes** (`for_each` inside `import` block) | **No** (imports only 1 resource per CLI invocation) |
| **VCS / PR Reviewable?** | **Yes** (works in HCP Terraform VCS-driven workflows) | **No** (requires direct CLI state access) |

---

## 2. Use the CLI to Inspect and Manage State (`7b`)

Never edit `terraform.tfstate` manually in a text editor! Use the `terraform state` subcommands:

```
Command                                     Operational Effect & 004 Exam Notes
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────
terraform state list                        Lists all resource and data source addresses tracked in state.
                                            (Can filter by module/resource: 'terraform state list module.vpc')
terraform state show <ADDRESS>              Displays all recorded attributes of a single resource in state
                                            (e.g., 'terraform state show "aws_instance.web[0]"').
terraform state mv <SOURCE> <DESTINATION>   Imperatively renames a resource or moves it into/out of a module in state
                                            without destroying the live cloud object (CLI equivalent of 'moved' block).
terraform state rm <ADDRESS>                Imperatively deletes a resource from state so Terraform stops managing it,
                                            WITHOUT destroying the real-world cloud resource (CLI equivalent of 'removed').
terraform state pull                        Downloads and prints the raw state JSON from the remote backend to stdout.
terraform state push <FILE>                 Uploads a local state file to the remote backend (dangerous; checks lineage/serial).
terraform state replace-provider <A> <B>    Updates the provider source address recorded in state (e.g., migrating namespaces).
```

> [!TIP]
> **Inspecting Outputs vs. State:** While `terraform state show` displays all attributes of a resource, `terraform output` (or `terraform output -json` / `terraform output -raw <NAME>`) reads only the top-level root module **output values** stored in the state file.

---

## 3. Describe When and How to Use Verbose Logging (`7c`)

When troubleshooting provider crashes, authentication failures, unexpected API responses, or Terraform Core graph evaluation issues, enable verbose diagnostic logging via environment variables:

1. **`TF_LOG` (Global Log Level):**
   Sets the verbosity level for **both** Terraform Core and Provider plugins. From **most verbose** to **least verbose**:
   $$\texttt{TRACE} \longrightarrow \texttt{DEBUG} \longrightarrow \texttt{INFO} \longrightarrow \texttt{WARN} \longrightarrow \texttt{ERROR}$$
   * **`TRACE`:** The **most detailed** level; logs every internal step, gRPC call, and sanitized HTTP request/response payload. Recommended by HashiCorp Support when filing bug reports.
   * **`JSON`:** Special setting (`TF_LOG=JSON`) that outputs machine-parseable JSON log lines at the `TRACE` level.
2. **`TF_LOG_CORE` and `TF_LOG_PROVIDER` (Scoped Log Levels):**
   - `TF_LOG_CORE=TRACE`: Enables trace logs **only for Terraform Core** (graph building, HCL evaluation) without noisy provider HTTP dumps.
   - `TF_LOG_PROVIDER=TRACE`: Enables trace logs **only for Provider plugins** (cloud API calls, auth handshakes).
3. **`TF_LOG_PATH` (Persisting Logs to Disk):**
   - By default, `TF_LOG` prints to `stderr` (terminal screen).
   - To append logs to a file on disk, you **must** set both `TF_LOG` (or `TF_LOG_CORE`/`TF_LOG_PROVIDER`) **and** `TF_LOG_PATH`:
     ```bash
     export TF_LOG="TRACE"
     export TF_LOG_PATH="./terraform-debug.log"
     terraform plan
     ```
   - *(Exam Trap: Setting `TF_LOG_PATH` alone without setting `TF_LOG` produces **no log output** unless `TF_LOG` is also enabled!)*
