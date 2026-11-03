# Domain 5: Terraform Modules — Quiz 1 (004)

**Exam:** HashiCorp Certified: Terraform Associate (004)  
**Format:** 10 Scenario-Based Questions with In-Depth Explanations

---

### Question 1 (Objective 5a & 5d)
An engineer writes the following `module` block sourcing a module from an internal Git repository:
```hcl
module "security_baseline" {
  source  = "git::https://git.corp.example.com/infra/terraform-aws-baseline.git"
  version = "1.4.0"
}
```
Why does `terraform validate` return an error on this configuration, and how should it be fixed?
- A. Git repositories cannot host Terraform modules.
- B. The `version` argument is only supported for modules fetched from a Terraform Registry (public or private); for Git sources, version pinning must be specified in the `source` URL query parameter (`?ref=v1.4.0`).
- C. The `version` argument must use the pessimistic `~>` operator.
- D. The `source` attribute must start with `./`.

---

### Question 2 (Objective 5b)
A child module `./modules/rds` contains `resource "aws_db_instance" "postgres" { ... }`, but the author of the child module did not define any `output` blocks in `./modules/rds/outputs.tf`. How can the calling root module access the database endpoint (`aws_db_instance.postgres.endpoint`)?
- A. By referencing `module.rds.aws_db_instance.postgres.endpoint` directly in the root module.
- B. It cannot; because of module scope encapsulation, the child module must first declare an `output` block exporting `aws_db_instance.postgres.endpoint`, and the root module then references `module.rds.<OUTPUT_NAME>`.
- C. By setting `export_all = true` inside the root `module "rds"` block.
- D. By reading `.terraform/modules/rds/main.tf` using the `file()` function.

---

### Question 3 (Objective 5b)
In your root module, you declare `variable "environment" { default = "production" }`. You also call a child module `module "compute" { source = "./modules/compute" }`. Will resources inside `./modules/compute` automatically have access to the root module's `var.environment`?
- A. Yes, all root variables are globally inherited by all child modules automatically.
- B. No; variables are scoped strictly to the module in which they are declared. To pass `environment` to `module.compute`, the child module must declare its own `variable "environment" {}` and the root module must pass `environment = var.environment` inside the `module "compute"` block.
- C. Yes, if the child module uses `root.var.environment`.
- D. Only if running in HCP Terraform.

---

### Question 4 (Objective 5a)
How does Terraform distinguish a **local path** module source from a **Public Terraform Registry** module source?
- A. Local module paths must start with either `./` or `../`, whereas Registry module addresses use the three-part `<NAMESPACE>/<NAME>/<PROVIDER>` format without a leading dot-slash.
- B. Local module paths require the `local::` prefix.
- C. Registry modules must end with `.git`.
- D. Local modules must be zipped into a `.tar.gz` archive first.

---

### Question 5 (Objective 5c)
You are using a local child module (`source = "./modules/network"`). You edit `./modules/network/main.tf` to add a new `aws_nat_gateway` resource. Do you need to re-run `terraform init` before running `terraform plan`?
- A. Yes, every edit to a local `.tf` file requires re-running `terraform init`.
- B. No; for local path modules already initialized, Terraform references the directory directly, so edits inside `./modules/network/main.tf` are picked up immediately by `terraform plan` (you only need `terraform init` when adding a new `module` block or changing `source`/`version`).
- C. Yes, and you must delete `.terraform.lock.hcl`.
- D. Local modules cannot be edited after initialization.

---

### Question 6 (Objective 5d)
What repository naming convention and Git tagging requirement must be met to publish a module to the Public Terraform Registry or HCP Terraform VCS-backed Private Registry?
- A. Repo named `module-<NAME>-<PROVIDER>` with branch names matching dates.
- B. Repo named `terraform-<PROVIDER>-<NAME>` (e.g., `terraform-aws-vpc`) with Semantic Versioning Git tags (e.g., `v1.0.0`).
- C. Repo named `hashicorp-<NAME>` with a `Dockerfile` in the root.
- D. Any repository name is accepted as long as it contains `terraform.tfstate`.

---

### Question 7 (Objective 5c)
Can a `module` block in Terraform 1.12 use the `count`, `for_each`, and `depends_on` meta-arguments?
- A. No, meta-arguments are only valid on `resource` blocks.
- B. Yes, since Terraform 0.13+, `module` blocks natively support `count`, `for_each`, `depends_on`, and `providers`.
- C. Only `count` is supported on modules; `for_each` is unsupported.
- D. Only if the module is sourced from the Public Registry.

---

### Question 8 (Objective 5b)
A child module exports an output `output "db_connection_string" { value = ... sensitive = true }`. The root module wants to re-export this value in its own `outputs.tf` so a CI script can read it via `terraform output -raw`. What requirement does Terraform enforce on the root module's `output` block?
- A. The root module must also set `sensitive = true` on its `output` block (or explicitly unwrap it with `nonsensitive()`), otherwise Terraform raises an error refusing to expose a sensitive value through a non-sensitive output.
- B. Sensitive outputs cannot be passed from child modules to root modules under any circumstances.
- C. The root module must encrypt the output with AWS KMS first.
- D. Nothing; sensitive flags are stripped automatically at module boundaries.

---

### Question 9 (Objective 5a)
Where does `terraform init` store remote child modules downloaded from the Terraform Registry or Git repositories?
- A. `.terraform/providers/`
- B. `.terraform/modules/`
- C. `~/.ssh/modules/`
- D. Inside `terraform.tfstate`

---

### Question 10 (Objective 5d)
Why is it a critical production best practice to pin the `version` of modules sourced from the Public Terraform Registry (e.g., `version = "~> 5.1.0"`) rather than omitting the `version` argument?
- A. Because omitting `version` causes `terraform init` to fail with a syntax error.
- B. Because omitting `version` downloads the latest available release on a fresh `terraform init`, which could pull in a new major version (`6.0.0`) containing breaking schema changes or resource replacements.
- C. Because unpinned modules cannot use variables.
- D. Because `.terraform.lock.hcl` automatically locks Registry module versions even without a `version` attribute. *(Trap: `.terraform.lock.hcl` only locks providers, NOT modules!)*

---

## Answer Key & Rationales

1. **B** — The `version` argument inside a `module` block is **only** valid for Registry sources (Public or Private Registry). Git sources pin versions via the `?ref=` query string in `source`.
2. **B** — Child module resources are encapsulated. A parent module can only read values that the child module explicitly exports via an `output` block (`module.<MODULE_NAME>.<OUTPUT_NAME>`).
3. **B** — Variables are never implicitly inherited across module boundaries. They must be declared in the child module's `variables.tf` and explicitly passed in the parent's `module` block.
4. **A** — Local module paths must begin with `./` or `../` to distinguish them from three-segment Registry addresses (`<NAMESPACE>/<NAME>/<PROVIDER>`).
5. **B** — Once a local module is linked during `terraform init`, edits to files inside that local directory are read directly by `plan`/`apply`. Remote modules (Registry/Git), however, are cached in `.terraform/modules/` and require `terraform init -upgrade` (or `terraform get -update`) when upstream changes occur.
6. **B** — Registry modules require the `terraform-<PROVIDER>-<NAME>` repository naming format and `vX.Y.Z` semantic version tags.
7. **B** — Modern Terraform supports `count`, `for_each`, `depends_on`, and `providers` directly on `module` blocks.
8. **A** — Terraform prevents accidental leakage of `sensitive = true` (and `ephemeral = true`) values by requiring any downstream root `output` block to also declare `sensitive = true` (or `ephemeral = true`).
9. **B** — Downloaded child modules are stored in `.terraform/modules/`.
10. **B** — Unlike providers, **module versions are NOT recorded in `.terraform.lock.hcl`**. Therefore, pinning `version` inside the `module` block is essential to prevent unexpected breaking changes on fresh `terraform init` runs.
