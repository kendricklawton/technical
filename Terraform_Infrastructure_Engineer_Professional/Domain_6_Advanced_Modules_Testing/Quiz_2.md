# Domain 6: Advanced Module Usage and Testing — Quiz 2

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Advanced Module Refactoring & Testing Architecture Questions

---

### Question 1: Testing Nested Child Modules with `terraform test`
Can a `run` block in `tests/main.tftest.hcl` execute a test against a nested sub-module located in `./modules/subnet`?
- A. No, tests can only evaluate root modules.
- B. Yes, by specifying `module { source = "./modules/subnet" }` inside the `run` block.
- C. Only if the submodule is hosted on GitHub.
- D. Only if using Terratest in Go.

---

### Question 2: Mocking Provider Data Sources
When using `mock_provider` in `terraform test`, what values are returned for data sources by default?
- A. `null` or empty strings unless overridden with `mock_data` blocks.
- B. Real data from the internet.
- C. Terraform crashes if a data source is queried under a mock provider.
- D. Random UUIDs.

---

### Question 3: Providing Custom Mock Data
How can you provide static mock return values for a data source (e.g. `data.aws_ami.ubuntu`) in a `mock_provider` block?
- A. By creating a `.json` fixture in `/tmp`.
- B. By using the `mock_data` nested block inside `mock_provider` matching the data source name and defining returned attributes.
- C. By intercepting DNS requests.
- D. By compiling a custom Go plugin.

---

### Question 4: Module Versioning & Breaking Changes
An engineer needs to remove an input variable `db_password` and replace it with an ephemeral Vault integration in a shared module. Under Semantic Versioning rules, what version increment is required?
- A. Patch version increment (`v1.2.0` -> `v1.2.1`).
- B. Minor version increment (`v1.2.0` -> `v1.3.0`).
- C. Major version increment (`v1.2.0` -> `v2.0.0`) because removing a required input variable is a breaking change for existing module callers.
- D. No version change is needed.

---

### Question 5: Module Pinning Strategy in Production
Why should production root modules pin module versions with exact (`= 2.4.1`) or pessimistic (`~> 2.4.0`) constraints rather than pointing to `main`?
- A. Git branches do not support submodules.
- B. Pointing to `main` exposes production to untested breaking changes or regressions pushed to the module repository without code review.
- C. Terraform cannot download code from `main`.
- D. Registries prohibit git branches.

---

### Question 6: Documenting Outputs in Enterprise Modules
Why is documenting the `description` and `type` of every output in `outputs.tf` critical for scalable module consumption?
- A. Automated documentation generators (like `terraform-docs`) inspect these attributes to render module reference tables and downstream developers rely on them for schema compliance.
- B. The Terraform compiler requires descriptions or it refuses to initialize.
- C. Descriptions encrypt outputs.
- D. Descriptions reduce state file size.

---

### Question 7: Submodule vs Standalone Module
When should a module be authored as a nested submodule (`./modules/cache`) rather than a standalone repository?
- A. When the submodule is tightly coupled to the parent module's domain and is rarely consumed independently.
- B. Always; standalone module repos are deprecated.
- C. Only when using Docker.
- D. When the module contains secrets.

---

### Question 8: Testing State Assertions
In a `terraform test` assertion:
```hcl
assert {
  condition     = length(output.subnet_ids) == 3
  error_message = "Module must produce exactly 3 subnet IDs."
}
```
If the module produces only 2 subnet IDs, what does `terraform test` output?
- A. A warning in stdout.
- B. The test run fails with exit code 1, printing the specified `error_message` and the failed expression details.
- C. It re-runs the test 3 times.
- D. It adds a 3rd subnet automatically.

---

### Question 9: Pre-commit Linting for Module Formatting
What command formats all HCL files recursively in a module directory tree?
- A. `terraform fmt -recursive`
- B. `terraform clean`
- C. `terraform tidy`
- D. `terraform lint --fix`

---

### Question 10: Testing Provider Upgrades with `terraform test`
How can a platform engineering team verify that upgrading the AWS or DigitalOcean provider from v4 to v5 will not cause unplanned resource destruction across their module library?
- A. Deploy to production immediately.
- B. Run `terraform test` across all module test suites using the new provider version constraint; inspect assertion outcomes and plan diffs for unexpected replacements.
- C. Delete all state files.
- D. Provider upgrades never cause destruction.

---

## Answer Key & Explanations

1. **B** — A `run` block can target nested child modules by setting `module { source = "..." }`.
2. **A** — Mock providers return zero-values unless explicitly given fixtures via `mock_data`.
3. **B** — The `mock_data` block allows deterministic mocking of data source outputs for comprehensive unit testing.
4. **C** — SemVer rules require major version increments (`X.y.z`) whenever an API's backward compatibility is broken.
5. **B** — Pinning prevents breaking changes in upstream modules from unexpectedly cascading into production deployments.
6. **A** — `description` and `type` provide essential developer ergonomics and enable automated documentation pipelines.
7. **A** — Submodules represent specialized, tightly coupled sub-capabilities packaged within a larger module ecosystem.
8. **B** — Failed assertions cause `terraform test` to exit non-zero, immediately failing CI pipelines.
9. **A** — `terraform fmt -recursive` scans and formats subdirectories and files in-place.
10. **B** — Comprehensive automated unit testing harnesses validate that provider schema changes do not trigger unintended resource replacements.
