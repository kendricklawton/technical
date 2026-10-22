# Domain 6: Advanced Module Usage and Testing — Quiz 1

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
What file extension is recognized by the `terraform test` command for test scenario suites?
- A. `*.tfspec`
- B. `*.tftest.hcl` (or `*.tftest.json`)
- C. `*.test.tf`
- D. `*.unit.tf`

---

### Question 2
In a `*.tftest.hcl` file, what is the effect of setting `command = plan` inside a `run` block?
- A. It provisions live cloud resources and then plans their deletion.
- B. It runs in unit test mode, generating an in-memory execution plan and checking assertions without making any write calls to remote cloud APIs.
- C. It outputs a `.tfplan` file to the root directory.
- D. It tests network connectivity to the cloud provider.

---

### Question 3
How do you test that a module's variable `validation` block correctly rejects invalid inputs using `terraform test`?
- A. By catching the exception in Python.
- B. By adding an `expect_failures` argument listing the variable that should fail validation within that `run` block.
- C. By setting `ignore_errors = true`.
- D. Terraform test cannot test negative failures.

---

### Question 4
What is the purpose of the `mock_provider` block in `terraform test`?
- A. It deploys mock resources into a local Docker container.
- B. It mocks provider interactions in memory, allowing `command = plan` runs to execute completely offline without real cloud credentials or API access.
- C. It generates fake billing invoices.
- D. It replaces AWS with LocalStack.

---

### Question 5
When refactoring a monolithic Terraform configuration containing 50 resources into reusable modules, what is the primary architectural criteria for determining module boundaries?
- A. Group all resources created on the same day into one module.
- B. Group resources that share a single cohesive lifecycle, common failure domain, or logical business capability (e.g. `networking`, `database`, `compute`).
- C. Place every individual resource into its own separate module.
- D. Group resources strictly alphabetically.

---

### Question 6
Why should reusable child modules NEVER hardcode `provider` blocks?
- A. Child modules do not support providers.
- B. Hardcoding providers inside child modules prevents module consumers from overriding regions, credentials, or aliases, breaks module reusability, and can lead to provider initialization conflicts.
- C. It causes syntax errors during `terraform fmt`.
- D. HashiCorp charges fees for child module providers.

---

### Question 7
What is the required Git repository naming format for a module published to the public Terraform Registry?
- A. `tf-module-<NAME>`
- B. `terraform-<PROVIDER>-<NAME>`
- C. `hashicorp-<NAME>-module`
- D. `<NAME>-terraform`

---

### Question 8
In a module repository, where should self-contained runnable examples demonstrating how to use the module be placed?
- A. In the `/src` folder.
- B. In an `/examples` directory at the repository root.
- C. Inside `/tests/fixtures`.
- D. In the `.gitignore` file.

---

### Question 9
How does a consumer reference a module hosted in a private HCP Terraform registry?
- A. `source = "git::https://github.com/..."`
- B. `source = "<HOSTNAME>/<ORGANIZATION-NAME>/<MODULE-NAME>/<PROVIDER>"` (e.g., `app.terraform.io/enterprise/vpc/aws`)
- C. `source = "local::/modules/vpc"`
- D. `source = "s3::https://..."`

---

### Question 10
What happens at the end of a `terraform test` execution that used `command = apply`?
- A. The created test infrastructure remains running indefinitely until manually destroyed.
- B. Terraform automatically destroys all temporary test resources created during the test run.
- C. The test resources are migrated to production.
- D. The state file is saved to `/tmp`.

---

## Answer Key & Explanations

1. **B** — The native test runner parses `*.tftest.hcl` and `*.tftest.json` files located in the root or `tests/` directory.
2. **B** — `command = plan` evaluates HCL logic and assertions against the planned state without mutating cloud resources.
3. **B** — `expect_failures` asserts that planned operations fail for the expected variables or preconditions.
4. **B** — `mock_provider` eliminates external cloud dependencies for fast, cost-free unit testing.
5. **B** — Cohesion, shared lifecycle, and bounded context determine optimal module architecture.
6. **B** — Providers must be configured at the root level and passed into child modules to preserve flexibility and maintainability.
7. **B** — Public registry indexing requires the 3-part naming format: `terraform-<PROVIDER>-<NAME>`.
8. **B** — The `/examples` directory is the canonical location for reference implementations.
9. **B** — Private registry source strings follow the 4-segment convention `<host>/<org>/<name>/<provider>`.
10. **B** — `terraform test` guarantees automated cleanup of resources provisioned during integration testing.
