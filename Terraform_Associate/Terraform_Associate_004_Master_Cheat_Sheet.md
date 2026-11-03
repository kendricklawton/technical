# HashiCorp Certified: Terraform Associate (004) — Master Cheat Sheet

> [!IMPORTANT]
> **Exam Profile (004):** Tested on **Terraform 1.12+**. Key 004 additions over 003 include: **Custom Conditions** (`precondition`, `postcondition`, `check` blocks), **Ephemeral Values & Write-Only Arguments** (`ephemeral` blocks, `ephemeral = true`, `_wo` attributes), **Declarative State Refactoring & Imports** (`moved`, `removed`, `import` blocks with `-generate-config-out`), **S3 Native State Locking** (`use_lockfile = true`), and **HCP Terraform Projects, Variable Sets, and Dynamic Provider Credentials**.

---

## 1. What's New in 004 — High-Yield Quick Reference

### 1.1 Custom Conditions Comparison (`4g`)

| Validation Construct | Where Defined | When Evaluated | Failure Behavior | Primary Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **`validation` block** | Inside `variable` block | During `plan` / `validate` | **Hard Error** (halts execution) | Validating caller input syntax/ranges (e.g., AMI prefix `ami-`, CIDR format) |
| **`precondition` block** | Inside `resource`, `data`, or `output` `lifecycle` block | **Before** evaluating/creating the target object | **Hard Error** (halts plan/apply) | Verifying assumptions before provisioning (e.g., fetched AMI architecture is `x86_64`) |
| **`postcondition` block** | Inside `resource` or `data` `lifecycle` block (uses `self.*`) | **After** reading/provisioning the target object | **Hard Error** (halts dependent resources) | Guaranteeing provisioned resource attributes (e.g., `self.encrypted == true`) |
| **`check` block** | Standalone top-level block (can contain scoped `data` source) | At the end of `plan` and `apply` | **Warning ONLY** (does **NOT** halt execution!) | Continuous health/infrastructure assertions (e.g., verifying HTTP `200` status code) |

### 1.2 Sensitive vs. Ephemeral vs. Write-Only Arguments (`4h`)

```
Mechanism                     Redacted in CLI Output?   Saved in Plan File?   Saved in State (.tfstate)?
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────
sensitive = true              YES                       YES (Plaintext)       YES (Plaintext JSON!)
ephemeral = true (Variable)   YES                       NO                    NO (Never persisted!)
ephemeral "..." block         YES                       NO                    NO (Opened & closed per cycle)
Write-Only Attr (*_wo)        YES                       NO                    NO (Only *_wo_version integer saved)
```

```hcl
# 1. Ephemeral Resource (TF 1.10+): Fetched dynamically during plan/apply, NEVER stored in state
ephemeral "aws_secretsmanager_secret_version" "db_pass" {
  secret_id = "prod/rds/master-password"
}

# 2. Write-Only Argument (TF 1.11+): Accepts ephemeral values; never written to plan or state
resource "aws_db_instance" "prod" {
  identifier          = "prod-postgres"
  engine              = "postgres"
  instance_class      = "db.t3.medium"
  password_wo         = ephemeral.aws_secretsmanager_secret_version.db_pass.secret_string
  password_wo_version = 1 # Increment this integer to trigger a password update in-place!
}
```

### 1.3 Declarative State Refactoring & Import Blocks (`6d`, `7a`)

```hcl
# 1. Rename or move a resource in state declaratively without destroying it (replaces 'terraform state mv')
moved {
  from = aws_instance.web
  to   = module.compute.aws_instance.web
}

# 2. Stop tracking a resource in state declaratively without destroying the real cloud asset (replaces 'terraform state rm')
removed {
  from = aws_s3_bucket.legacy_audit_logs

  lifecycle {
    destroy = false # false = remove from state ONLY; keep real-world bucket alive
  }
}

# 3. Declaratively import existing infrastructure into state (TF 1.5+)
# Run: terraform plan -generate-config-out=generated_resources.tf
import {
  to = aws_vpc.imported_main
  id = "vpc-0a1b2c3d4e5f6g7h8"
}
```

---

## 2. Variable Definition Precedence Hierarchy (`4c`)

When the same variable is set across multiple sources, Terraform resolves the value from **LOWEST** to **HIGHEST** priority:

```
Priority   Variable Source                                             Example / Note
─────────────────────────────────────────────────────────────────────────────────────────────────────────────
1 (Lowest) Environment Variables                                       export TF_VAR_instance_type="t3.micro"
2          The terraform.tfvars file                                   terraform.tfvars (if present)
3          The terraform.tfvars.json file                              terraform.tfvars.json (if present)
4          Any *.auto.tfvars or *.auto.tfvars.json                     Processed in lexical (alphabetical) order
5 (Highest) Command-line flags (-var and -var-file)                    terraform apply -var="instance_type=t3.large"
```

---

## 3. Core Terraform Workflow & CLI Reference (`3a` – `3g`, `7b`, `7c`)

| Command | Key Flags | Operational Effect on 004 Exam |
| :--- | :--- | :--- |
| `terraform init` | `-upgrade` | Upgrades all providers and modules to the newest version allowed by constraints, updating `.terraform.lock.hcl` |
| `terraform init` | `-migrate-state` | Copies existing state to a newly configured backend |
| `terraform init` | `-reconfigure` | Initializes backend configuration cleanly, disregarding any existing state |
| `terraform validate` | `-json` | Offline check of HCL syntax, attribute names, and types (requires `terraform init` first; does not query cloud APIs) |
| `terraform fmt` | `-check`, `-diff`, `-recursive` | Rewrites `.tf` files to canonical 2-space style; `-check` returns non-zero exit code if unformatted |
| `terraform plan` | `-out=tfplan` | Saves binary execution plan so `terraform apply tfplan` executes deterministically without confirmation prompt |
| `terraform plan` | `-refresh-only` | Inspects drift between real-world infrastructure and state without proposing changes to cloud resources |
| `terraform apply` | `-auto-approve` | Bypasses interactive `yes` confirmation prompt |
| `terraform apply` | `-replace=ADDR` | Forces destruction and recreation of a specific degraded resource during apply (replaces deprecated `terraform taint`) |
| `terraform destroy` | `-auto-approve` | Destroys all managed resources in reverse dependency order (alias for `terraform apply -destroy`) |
| `terraform state` | `list`, `show`, `mv`, `rm`, `pull` | Inspects or mutates state via CLI; `pull` prints raw state JSON to stdout |
| `terraform console` | *(Interactive)* | Opens interactive REPL shell to test expressions, built-in functions, and inspect state (read-only) |
| `terraform graph` | *(Pipe to dot)* | Generates visual execution graph in DOT format (`terraform graph \| dot -Tpng > graph.png`) |
| `terraform show` | `-json` [tfplan] | Displays human-readable state or binary plan; `-json` prints machine-readable structure |
| `terraform providers` | `lock -platform=...` | Prints provider tree; `lock` pre-fetches multi-platform checksums into `.terraform.lock.hcl` |
| `terraform workspace` | `list`, `new`, `select`, `show`, `delete` | Manages CLI workspaces (`delete` cannot delete `default` or current active workspace) |

### Verbose Logging (`7c`)
- `TF_LOG`: Sets verbosity (`TRACE` [most detailed], `DEBUG`, `INFO`, `WARN`, `ERROR`).
- `TF_LOG_CORE` / `TF_LOG_PROVIDER`: Isolates logs to Terraform Core or Provider plugins specifically.
- `TF_LOG_PATH`: Writes logs to a file on disk (required if persisting output across runs).

---

## 4. Provider Versioning & Lock Files (`2a` – `2c`)

- **`= 1.2.0`:** Exact version match only.
- **`~> 1.2.0` (Pessimistic Patch):** Allows `>= 1.2.0` and `< 1.3.0`.
- **`~> 1.2` (Pessimistic Minor):** Allows `>= 1.2.0` and `< 2.0.0`.
- **`.terraform.lock.hcl`:** Locks provider versions and cryptographic hashes (`h1:` and `zh:`). **Always commit `.terraform.lock.hcl` to Git!**
- **Multi-Platform Checksums:** Run `terraform providers lock -platform=linux_amd64 -platform=darwin_arm64 -platform=windows_amd64` to populate multi-platform hashes for CI/CD runners.

---

## 5. Backends & State Locking (`6a` – `6c`)

- **Local Backend:** Stores state in `terraform.tfstate` (and previous version in `terraform.tfstate.backup`).
- **Amazon S3 Backend:** Supports native S3 object locking via `use_lockfile = true` (creates `.tflock` file using S3 conditional writes in Terraform 1.10+) as well as legacy `dynamodb_table` locking.
- **AzureRM / GCS / HCP Terraform:** Support state locking natively out of the box.
- **`terraform force-unlock <LOCK_ID>`:** Manually unlocks state if a previous CI/CD runner or terminal session crashed mid-operation.
- **`terraform_remote_state`:** Reads root outputs from another workspace's remote state file (downloads full state file; prefer `tfe_outputs` in HCP Terraform).

---

## 6. High-Yield Built-in Functions & Expressions (`4e`)

```
Function / Operator           Operational Effect on Exam
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────
lookup(map, key, default)     Returns map[key], or fallback default if key does not exist (prevents crashes).
element(list, index)          Returns element at index; wraps around modulo length (never errors out-of-bounds!).
merge(map1, map2, ...)        Combines maps; later maps override keys from earlier maps.
concat(list1, list2, ...)     Combines multiple lists into a single flat list.
flatten(list_of_lists)        Takes a list of lists and returns a single flattened 1D list.
distinct(list)                Removes duplicates while preserving original element order.
slice(list, start, end)       Extracts sublist from start (inclusive) to end (exclusive).
zipmap(keys, values)          Constructs a map from a list of string keys and list of values.
toset(list)                   Converts list to set (eliminates duplicates; required for for_each on lists).
file(path)                    Reads raw file contents as literal string (no template interpolation).
templatefile(path, vars)      Reads file and renders ${...} variables using the passed map.
jsonencode() / jsondecode()   Encodes HCL to JSON string / decodes JSON string to HCL structure.
can(expression)               Returns true if expression evaluates without error, false otherwise.
try(expr1, expr2, fallback)   Returns first expression that evaluates without error (clean fallback logic).
cidrsubnet(pfx, newbits, num) Calculates a child subnet CIDR address.
User-Defined Functions        NOT supported in standard Terraform HCL! Must use built-in functions.
```

---

## 7. Provisioners and `terraform_data` (`4a`, `4f`)

- **Types:**
  - `local-exec`: Runs shell command on machine running Terraform.
  - `remote-exec`: Runs command on newly provisioned instance over SSH/WinRM (requires `connection` block).
  - `file`: Copies files/directories from Terraform host to remote machine.
- **Execution Timing:**
  - Creation-time (default): Runs after resource creation. Failure marks resource **tainted / degraded** in state.
  - Destroy-time (`when = destroy`): Runs *before* resource is destroyed. Failure prevents resource deletion.
- **`on_failure`:** `fail` (default, halts run) vs. `continue` (logs error and continues).
- **Measure of Last Resort:** Provisioners break declarative idempotence and drift detection. Prefer `user_data` / cloud-init or Packer images.
- **`terraform_data`:** Built-in resource (TF 1.4+) that replaces `null_resource`, stores arbitrary values, and triggers actions via `triggers_replace`.

---

## 8. HCP Terraform Hierarchy & Governance (`8a` – `8d`)

```
Organization (Top-Level Tenant: Teams, Private Registry, Policy Sets, Variable Sets)
 └── Project (Logical Grouping & RBAC Boundary: e.g., "Payments-Prod", "Core-Networking")
      └── Workspace (Isolated State, Execution Queue, VCS Repo Mapping, Run Triggers)
```

* **Variable Sets:** Reusable groups of Terraform or Environment variables defined at the Organization level and applied globally, to specific **Projects**, or to specific **Workspaces** (Workspace variables override Project/Variable Set values).
* **Dynamic Provider Credentials:** Uses OIDC Workload Identity Federation to generate short-lived AWS/GCP/Azure/Vault tokens per run, eliminating static long-lived secrets.
* **Run Triggers:** Automatically queue a run in a downstream workspace (e.g., `app-cluster`) whenever an upstream workspace (e.g., `network-vpc`) finishes a successful `apply`.
* **Sentinel & OPA Enforcement Levels:**
  1. **Advisory:** Logs a warning; run continues uninterrupted.
  2. **Soft Mandatory:** Halts the run; authorized administrators can manually override.
  3. **Hard Mandatory:** Strictly blocks the run; **cannot be overridden by anyone**.
* **HCP Terraform (SaaS) vs. Terraform Enterprise (Self-Hosted):** HCP Terraform is cloud-hosted by HashiCorp; Terraform Enterprise is deployed on customer VPCs/on-prem, supporting strict air-gapped environments.
