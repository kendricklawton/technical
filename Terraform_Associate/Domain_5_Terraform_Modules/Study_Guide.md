# Domain 5: Terraform Modules — Study Guide (004)

> **Official 004 Blueprint Focus:** Objective 5 — Terraform Modules (`5a`, `5b`, `5c`, `5d`).  
> **Tested Version:** Terraform 1.12+

---

## 1. Explain How Terraform Sources Modules (`5a`)

A **Terraform Module** is a self-contained directory of `.tf` files packaged together for reuse.
- **Root Module:** The top-level working directory where you execute `terraform init`, `plan`, and `apply`. Every Terraform configuration has exactly one root module.
- **Child Module:** Any module invoked by another module using a `module "<NAME>" { ... }` block.

### 1.1 Supported Module `source` Locations
Every `module` block **must** include the mandatory `source` meta-argument, which tells `terraform init` where to retrieve the child module's code:

| Module Source Type | Syntax Example | Versioning Mechanism |
| :--- | :--- | :--- |
| **Local Relative Path** | `source = "./modules/vpc"` or `"../network"` | Must start with `./` or `../`; **does NOT support `version` argument** (uses local filesystem symlink) |
| **Public Terraform Registry** | `source = "<NAMESPACE>/<NAME>/<PROVIDER>"` (e.g., `"terraform-aws-modules/vpc/aws"`) | Uses the **`version` argument** (`version = "~> 5.0"`) |
| **HCP Terraform Private Registry** | `source = "app.terraform.io/<ORG>/<NAME>/<PROVIDER>"` | Uses the **`version` argument** (`version = "2.1.0"`) |
| **GitHub / Generic Git Repo** | `source = "github.com/org/repo"` or `"git::https://example.com/vpc.git?ref=v1.4.0"` | Uses URL query string **`?ref=<TAG_BRANCH_OR_SHA>`** (does NOT use `version` argument!) |
| **Amazon S3 / GCS Bucket** | `source = "s3::https://s3-eu-west-1.amazonaws.com/bucket/vpc.zip"` | Object path / S3 version ID |

> [!WARNING]
> **004 Exam Trap:** Whenever you add a new `module` block, modify a module's `source`, or change a Registry module's `version`, you **must run `terraform init`** (or `terraform get`) so Terraform downloads/indexes the module into `.terraform/modules/`.

---

## 2. Describe Variable Scope Within Modules (`5b`)

Terraform enforces strict **lexical encapsulation (scope isolation)** between parent and child modules:
1. **Parent $\rightarrow$ Child Input Flow:** A child module **cannot** automatically read `var.*` or `local.*` defined in the parent root module. To pass data into a child module, the child module must declare `variable "<INPUT_NAME>" {}` in its own `variables.tf`, and the parent must pass that argument inside the `module "<NAME>" { <INPUT_NAME> = ... }` block.
2. **Child $\rightarrow$ Parent Output Flow:** A parent module **cannot** directly reference internal resources inside a child module (e.g., `module.vpc.aws_vpc.main.id` is **illegal syntax**!). The child module must explicitly declare an `output "<OUTPUT_NAME>" { value = ... }` block, which the parent accesses via:
   ```hcl
   module.<MODULE_NAME>.<OUTPUT_NAME>
   ```
3. **Sensitive & Ephemeral Output Propagation:** If a child module marks an output `sensitive = true` (or `ephemeral = true` in TF 1.10+), the root module must also mark any root output that exposes that value as `sensitive = true` (or `ephemeral = true`), otherwise `terraform validate` / `plan` will fail with an error.

---

## 3. Use Modules in Configuration (`5c`) & Manage Module Versions (`5d`)

```hcl
# Root Module: main.tf
module "app_vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.13.0" # Supported ONLY for Public/Private Registry sources!

  name = "prod-vpc"
  cidr = "10.10.0.0/16"

  azs             = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.10.1.0/24", "10.10.2.0/24"]
}

resource "aws_db_subnet_group" "rds" {
  name       = "prod-rds-subnets"
  subnet_ids = module.app_vpc.private_subnets # Accessing child module output
}
```

### 3.1 Public & Private Registry Publishing Standards (`5d`)
To publish a module to the Public Terraform Registry or HCP Terraform Private Registry via VCS:
1. The Git repository name **must** follow the three-part naming convention:
   `terraform-<PROVIDER>-<NAME>` (e.g., `terraform-aws-eks`).
2. Releases **must** use valid **Semantic Versioning (`vX.Y.Z` or `X.Y.Z`) Git tags** (e.g., `v1.0.0`, `v1.2.3`).
3. Standard module file layout (Best Practice):
   - `main.tf` (primary resources and data sources)
   - `variables.tf` (input variable declarations with `type` and `description`)
   - `outputs.tf` (exported values with `description`)
   - `README.md` and `LICENSE`
