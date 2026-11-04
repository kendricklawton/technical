# Terraform Associate (004) — Practice Exam 3

**Focus:** HCP Terraform Projects & Workspaces (`8a`–`8d`), Governance, Custom Conditions (`4g`), Ephemeral Secrets (`4h`), and Full 004 Blueprint Synthesis  
**Tested Version:** Terraform 1.12+  
**Format:** 25 Scenario-Based Questions with In-Depth Technical Explanations  
**Passing Score:** 80% (20 / 25)

---

### Question 1 (Objective 8c — New in 004)
An organization uses HCP Terraform to manage 80 workspaces across four business units (`Finance`, `Retail`, `Logistics`, and `Core-IT`). How should the administrator organize these workspaces so that each business unit's team automatically gets `Plan` and `Apply` permissions and shared cloud credentials for all workspaces in their business unit, without having to reconfigure permissions every time a new workspace is created?
- A. Create four separate GitHub accounts.
- B. Store credentials in a public module on the Terraform Registry.
- C. Create four separate HCP Terraform **Projects** (one per business unit), assign team permissions at the Project level, and attach Project-scoped **Variable Sets** to each Project.
- D. Create four local CLI workspaces using `terraform workspace new`.

---

### Question 2 (Objective 8a & 8c)
Examine the following `cloud` block inside `main.tf`:
```hcl
terraform {
  cloud {
    organization = "finserv-global"

    workspaces {
      project = "Core Banking"
      tags    = ["networking", "prod"]
    }
  }
}
```
What is the behavior when an engineer runs `terraform init` with this configuration?
- A. It runs `terraform destroy` on all untagged workspaces.
- B. It connects to the `finserv-global` organization in HCP Terraform and filters/selects workspaces belonging to the `"Core Banking"` Project that carry both the `"networking"` and `"prod"` workspace tags.
- C. It fails because `project` and `tags` cannot be used inside a `cloud` block.
- D. It creates an AWS S3 bucket named `finserv-global`.

---

### Question 3 (Objective 8b)
An organization wants to enforce a policy in HCP Terraform that blocks any workspace run if an `aws_s3_bucket` resource is missing a `CostCenter` tag, with **zero exceptions** allowed—not even Organization Owners should be able to bypass the failure. Which Sentinel enforcement level must be configured?
- A. `Hard Mandatory`
- B. `Soft Mandatory`
- C. `Audit-Only`
- D. `Advisory`

---

### Question 4 (Objective 8b)
In addition to HashiCorp Sentinel, which open-source, CNCF-graduated policy engine is natively supported in HCP Terraform for writing policy-as-code rules in **Rego**?
- A. Kubernetes Admission Webhook
- B. HashiCorp Consul
- C. Ansible Lint
- D. Open Policy Agent (OPA)

---

### Question 5 (Objective 8b)
How does HCP Terraform's **Dynamic Provider Credentials** feature authenticate to AWS, Microsoft Azure, Google Cloud, or HashiCorp Vault during a run?
- A. By emailing a one-time passcode to the workspace administrator on every `terraform plan`.
- B. By storing long-lived IAM secret keys in plaintext inside the Git repository.
- C. By reading static root keys from `/etc/passwd`.
- D. By generating a short-lived OpenID Connect (OIDC) JSON Web Token (JWT) unique to that specific workspace run and exchanging it with the cloud provider's identity federation endpoint (e.g., AWS STS `AssumeRoleWithWebIdentity`) for temporary credentials.

---

### Question 6 (Objective 4g — New in 004)
An engineer writes a module that outputs a database connection endpoint. Before exposing the output to the caller, the module author wants to verify that the database instance status is `"available"`, halting execution with an error if it is not. Which block can be placed directly inside the `output` block's `lifecycle` block?
- A. `postcondition` *(Trap: `output` blocks do not have a `self` object, so they support `precondition`, not `postcondition`!)*
- B. `check`
- C. `precondition`
- D. `validation`

---

### Question 7 (Objective 4h — New in 004)
Which of the following block types can be marked with `ephemeral = true` in Terraform 1.10+ / 1.12+ so that their values are never persisted in state or plan files?
- A. `backend` blocks and `provider` blocks
- B. `variable` blocks and `output` blocks
- C. `resource` blocks and `data` blocks
- D. `moved` blocks and `removed` blocks

---

### Question 8 (Objective 8b)
What happens when **Drift Detection** (part of HCP Terraform **Health Assessments**) is enabled on a workspace?
- A. HCP Terraform automatically runs `terraform apply -auto-approve` every 5 minutes to overwrite manual changes without human approval.
- B. HCP Terraform deletes drifted resources immediately.
- C. HCP Terraform periodically executes background `-refresh-only` checks on a schedule and alerts the team in the UI / via notifications if real-world cloud resources have diverged from the workspace's state file.
- D. HCP Terraform locks the cloud provider's web console so administrators cannot log in.

---

### Question 9 (Objective 8c)
You have two workspaces in HCP Terraform: `infra-network` and `infra-k8s`. `infra-k8s` needs to read the `vpc_id` and `private_subnet_ids` outputs from `infra-network`. Which data source is recommended by HashiCorp in HCP Terraform because it reads **only** the upstream workspace's root outputs without requiring access to the entire raw upstream state file?
- A. `data "external"`
- B. `data "http"`
- C. `data "local_file"`
- D. `data "tfe_outputs"`

---

### Question 10 (Objective 4e)
Examine the following HCL expression:
```hcl
locals {
  admin_arns = [for user in var.iam_users : user.arn if user.is_admin]
}
```
What type of HCL expression is `[for user in var.iam_users : ...]`?
- A. A `dynamic` block
- B. A `for` expression (list comprehension with an `if` filter clause)
- C. A ternary conditional expression
- D. A `splat` expression

---

### Question 11 (Objective 3d)
In a CI/CD script, an engineer runs `terraform plan -detailed-exitcode`. The command finishes and returns **exit code `2`**. What does exit code `2` signify?
- A. The state file is locked by another user.
- B. The plan succeeded and detected **zero changes** (infrastructure matches configuration).
- C. A fatal syntax or authentication error occurred.
- D. The plan succeeded and detected **pending changes** (a non-empty diff exists).

---

### Question 12 (Objective 6c)
When using a remote backend (such as `s3` or HCP Terraform `cloud`), where are sensitive input variables (`sensitive = true`) that are passed to standard resource attributes stored after `terraform apply`?
- A. They are hashed with one-way bcrypt so Terraform can never read them again.
- B. They are stored in plaintext JSON inside the remote state file (which is why remote state storage must be encrypted at rest and protected by strict IAM/RBAC access controls, or replaced with `ephemeral` / `_wo` write-only arguments).
- C. They are written to `.terraform.lock.hcl`.
- D. They are never stored in remote state files; only local state files store sensitive values.

---

### Question 13 (Objective 7a)
After running `terraform plan -generate-config-out=imported.tf` and `terraform apply` using an `import` block (`import { to = aws_vpc.main id = "vpc-12345" }`), what happens if you later remove the `import` block from your `.tf` files while keeping the `resource "aws_vpc" "main"` block?
- A. `terraform validate` fails unless `import` blocks are kept forever.
- B. Terraform removes `aws_vpc.main` from the state file.
- C. Terraform destroys `aws_vpc.main` on the next apply.
- D. Nothing happens to the VPC; once a resource has been imported into `terraform.tfstate` and bound to its `resource` block, the `import` block has fulfilled its purpose and can either be removed or left in place as historical documentation.

---

### Question 14 (Objective 2b)
What is the correct precedence order when Terraform searches for provider plugins during `terraform init`?
- A. Terraform compiles the Go source code of the provider on the fly during `init`.
- B. If a local filesystem mirror or `plugin_cache_dir` is configured, Terraform checks the local mirror/cache according to CLI configuration rules before downloading from the upstream Terraform Registry over HTTPS.
- C. Terraform always downloads from GitHub Releases directly, ignoring `registry.terraform.io`.
- D. Terraform reads provider binaries from `terraform.tfstate`.

---

### Question 15 (Objective 4c)
An engineer wants to provide values for declared input variables automatically without passing any `-var` or `-var-file` flags on the CLI and without setting `TF_VAR_*` environment variables. Which two filenames does Terraform load **automatically** if present in the root working directory?
- A. `terraform.tfvars` (or `.json`) and any file ending in `*.auto.tfvars` (or `.json`)
- B. `defaults.tfvars` and `config.hcl`
- C. `variables.tf` and `outputs.tf`
- D. `main.tfvars` and `values.tfvars`

---

### Question 16 (Objective 5c)
If a root module calls `module "app_server" { source = "./modules/server" count = 3 }`, how do you reference the `public_ip` output of the **first** module instance in the root module?
- A. `module[0].app_server.public_ip`
- B. `app_server[0].module.public_ip`
- C. `module.app_server.public_ip[0]`
- D. `module.app_server[0].public_ip`

---

### Question 17 (Objective 8d)
What is the purpose of **Run Tasks** in HCP Terraform?
- A. To format `.tf` files in the browser.
- B. To integrate external third-party tools and services (such as Snyk security scanners, ServiceNow change tickets, or Infracost) into the HCP Terraform run lifecycle at `Pre-plan`, `Post-plan`, `Pre-apply`, or `Post-apply` stages via webhooks.
- C. To schedule Linux cron jobs inside target EC2 instances.
- D. To replace `terraform init`.

---

### Question 18 (Objective 4e)
An engineer writes the following string in HCL:
```hcl
user_data = <<-EOT
  #!/bin/bash
  echo "Hello World"
EOT
```
What is the purpose of the hyphen in the indented heredoc syntax `<<-EOT` compared to standard `<<EOT`?
- A. `<<-EOT` marks the string as `ephemeral`.
- B. `<<-EOT` executes the bash script on the local laptop during `terraform plan`.
- C. `<<-EOT` base64-encodes the string automatically.
- D. `<<-EOT` automatically strips leading indentation whitespace up to the indentation level of the closing `EOT` marker, producing a clean script without leading spaces.

---

### Question 19 (Objective 6b)
Which of the following operations does **NOT** acquire a state lock when executed?
- A. `terraform validate`
- B. `terraform destroy`
- C. `terraform plan` (default mode)
- D. `terraform apply`

---

### Question 20 (Objective 4h)
When integrating **HashiCorp Vault** with Terraform to inject short-lived secrets into infrastructure, what is a key security limitation if you use a standard `data "vault_generic_secret"` data source instead of an `ephemeral "vault_kv_secret_v2"` resource (TF 1.10+)?
- A. Standard `data` sources cache all read attributes inside `terraform.tfstate` in plaintext JSON, whereas `ephemeral` resources fetch the secret from Vault at runtime without ever writing it to `terraform.tfstate`.
- B. Vault can only be used with GCP.
- C. `data "vault_generic_secret"` deletes the secret from Vault after reading it.
- D. Standard `data` sources cannot authenticate to Vault.

---

### Question 21 (Objective 3e)
Why does HashiCorp warn against using `terraform apply -target=<RESOURCE>` as part of normal daily workflows?
- A. Because `-target` disables state encryption.
- B. Because `-target` requires a paid enterprise license.
- C. Because targeting only a subset of resources skips downstream dependents in the dependency graph, leaving the overall environment in an inconsistent, partially updated state that diverges from the full `.tf` configuration in Git.
- D. Because `-target` deletes all untargeted resources from the cloud account.

---

### Question 22 (Objective 7b)
You run `terraform state list` and see `module.vpc.aws_nat_gateway.this[0]`. You want to view the exact `public_ip` and `allocation_id` attributes recorded in state for that NAT Gateway. Which command should you run?
- A. `terraform state show 'module.vpc.aws_nat_gateway.this[0]'`
- B. `terraform output 'module.vpc.aws_nat_gateway.this[0]'`
- C. `terraform plan -show 'module.vpc.aws_nat_gateway.this[0]'`
- D. `terraform validate 'module.vpc.aws_nat_gateway.this[0]'`

---

### Question 23 (Objective 8a)
What command creates an API token via your web browser and saves it to the local CLI credentials file so your terminal can authenticate to HCP Terraform (or a private Terraform Enterprise instance)?
- A. `terraform auth`
- B. `terraform init -cloud`
- C. `terraform login`
- D. `terraform connect`

---

### Question 24 (Objective 4d)
If you define a variable with `type = map(number)`, what happens if a user passes `{ web = "3", api = 5 }`?
- A. Terraform converts `5` into `"five"`.
- B. Terraform drops the `web` key from the map.
- C. Terraform automatically converts `"3"` to the number `3` because Terraform supports automatic type conversion between primitive types (`string`, `number`, `bool`) when they can be cleanly coerced.
- D. Terraform crashes with a type mismatch error.

---

### Question 25 (Objective 6d & 7b)
Compare the declarative `moved` block (`moved { from = A to = B }`) with the imperative CLI command `terraform state mv A B`. When is the `moved` block strongly preferred?
- A. When refactoring a reusable module or collaborative Git repository, because `moved` blocks are committed to version control, previewed in `terraform plan` / Pull Requests, and automatically applied across every workspace or consumer that calls the module.
- B. When you want to destroy the resource at address `A`.
- C. Never; `moved` blocks were removed in Terraform 1.12.
- D. When you want to move a resource without running `terraform apply`.

---

## Answer Key & Explanations

1. **C** — **HCP Terraform Projects** (004 Objective `8c`) provide a logical grouping layer above workspaces for Project-level RBAC team permissions and Project-scoped **Variable Sets**.
2. **B** — Inside the `cloud { workspaces { ... } }` block, specifying `project` and `tags` binds the directory to the matching set of tagged workspaces within that HCP Terraform Project.
3. **A** — **Hard Mandatory** blocks any run that fails the policy and **cannot be overridden** by any user or owner.
4. **D** — HCP Terraform natively supports both **HashiCorp Sentinel** and **Open Policy Agent (OPA)** written in Rego.
5. **D** — **Dynamic Provider Credentials** use OIDC workload identity federation to exchange a per-run signed JWT for temporary cloud credentials.
6. **C** — `output` blocks (as well as `resource` and `data` blocks) support `precondition` blocks inside `lifecycle`. (`postcondition` requires `self`, which only exists on `resource` and `data` blocks.)
7. **B** — In addition to `ephemeral "<TYPE>" "<NAME>"` resource blocks, both `variable` blocks and `output` blocks support `ephemeral = true`.
8. **C** — **Drift Detection** in HCP Terraform Health Assessments runs scheduled background `-refresh-only` checks and notifies teams when drift is detected.
9. **D** — The `tfe_outputs` data source securely fetches only the root outputs of another HCP Terraform workspace without exposing the entire raw state file.
10. **B** — `[for ... in ... : ... if ...]` is a **for expression** (list comprehension) that transforms and filters a collection.
11. **D** — With `-detailed-exitcode`: `0` = succeeded with no changes; `1` = error; `2` = succeeded with changes present (diff non-empty).
12. **B** — `sensitive = true` does not strip values from `terraform.tfstate`; only `ephemeral` resources/variables and write-only (`_wo`) arguments omit values from state.
13. **D** — Once `terraform apply` has imported the resource into `terraform.tfstate`, the resource is tracked normally by its `resource` block. Removing the `import` block later does not affect or destroy the resource.
14. **B** — Terraform checks local filesystem mirrors / `TF_PLUGIN_CACHE_DIR` (if configured) before downloading from the Registry over HTTPS.
15. **A** — Terraform automatically loads `terraform.tfvars`, `terraform.tfvars.json`, `*.auto.tfvars`, and `*.auto.tfvars.json` from the current working directory.
16. **D** — When a `module` block uses `count` or `for_each`, the index or key is placed on the module name: `module.<MODULE_NAME>[<INDEX>].<OUTPUT_NAME>` (`module.app_server[0].public_ip`).
17. **B** — **Run Tasks** invoke external webhook services at `Pre-plan`, `Post-plan`, `Pre-apply`, or `Post-apply` stages.
18. **D** — `<<-EOT` (indented heredoc) strips leading indentation whitespace matching the closing delimiter's indentation.
19. **A** — `terraform validate` runs completely offline against local `.tf` files and schemas without reading or locking state.
20. **A** — Standard `data` sources persist all fetched attributes in `terraform.tfstate`, whereas `ephemeral` blocks keep fetched Vault secrets exclusively in memory during the run.
21. **C** — `-target` bypasses full graph evaluation and can leave dependent resources out of sync with the `.tf` configuration; it should be reserved for emergency troubleshooting only.
22. **A** — `terraform state show '<ADDRESS>'` prints all recorded state attributes for a specific resource instance.
23. **C** — `terraform login` authenticates the CLI to HCP Terraform or Terraform Enterprise and stores the API token in `credentials.tfrc.json`.
24. **C** — Terraform automatically coerces primitive types (`"3"` $\rightarrow$ `3`, `"true"` $\rightarrow$ `true`) when a target type constraint (`map(number)`) is known.
25. **A** — Declarative `moved` blocks allow state refactoring to travel through Git Pull Requests and apply automatically across all workspaces and downstream module consumers.
