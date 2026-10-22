# Domain 6: Advanced Modules & Testing Frameworks — Quiz 3

**Exam:** HashiCorp Certified: Terraform Infrastructure Engineer Professional (002)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
In the native `terraform test` framework (Terraform v1.6+), what is the difference between `command = plan` and `command = apply` within a `run` block?
- A. `command = plan` executes static syntax checks only; `command = apply` deploys code to production.
- B. `command = plan` generates an execution plan and evaluates assertions without creating real cloud infrastructure; `command = apply` provisions ephemeral test resources, asserts conditions, and destroys them upon completion.
- C. `command = apply` cannot use assertions.
- D. There is no difference.

---

### Question 2
You are authoring a `.tftest.hcl` file and want to verify that an output `bucket_arn` matches a specific regex pattern. Which block and syntax is used?
- A.
```hcl
run "verify_arn" {
  command = plan
  assert {
    condition     = can(regex("^arn:aws:s3:::[a-z0-9.-]+$", output.bucket_arn))
    error_message = "Output bucket_arn is invalid"
  }
}
```
- B.
```hcl
test "arn" {
  expect = output.bucket_arn != null
}
```
- C.
```hcl
check "arn" {
  rule = "regex"
}
```
- D.
```hcl
verify {
  match = output.bucket_arn
}
```

---

### Question 3
How does Terratest differ from the native `terraform test` framework?
- A. Terratest is written in Go and provisions real infrastructure while executing real-world end-to-end integration tests (such as sending HTTP requests or SSHing into VMs) using cloud SDKs.
- B. Terratest is built into the Terraform binary.
- C. Terratest does not support AWS.
- D. Terratest is a static analysis linter.

---

### Question 4
In Terraform v1.5+, what is the purpose of the top-level `check` block in configuration files?
- A. It blocks `terraform apply` if any syntax error exists.
- B. It declares continuous assertions and health checks about infrastructure (e.g. verifying an HTTPS endpoint returns status 200) that emit warnings during plan/apply without failing the deployment.
- C. It checks Git commit signatures.
- D. It verifies cloud billing limits.

---

### Question 5
When structuring an enterprise Terraform module repository, where should example configurations be located according to HashiCorp best practices?
- A. In the `tests/` directory only.
- B. In an `examples/` subfolder (e.g. `examples/complete/`, `examples/basic/`) containing functional, deployable root modules demonstrating usage patterns.
- C. In the root directory mixed with module code.
- D. In the `.terraform` folder.

---

### Question 6
You are writing a unit test with `terraform test` and want to test a module that depends on an external provider without making real AWS API calls or requiring cloud credentials. What feature introduced in Terraform v1.7+ enables this?
- A. Mock providers (`mock_provider "aws" {}`)
- B. Local state emulation
- C. Dryrun CLI flag
- D. Dummy backend

---

### Question 7
In a reusable root module, how can you publish a module to a private HCP Terraform / Terraform Enterprise registry?
- A. By uploading a zip file via FTP.
- B. By tagging a Git repository following the naming convention `terraform-<PROVIDER>-<NAME>` (e.g., `terraform-aws-vpc`) with semantic version tags (e.g., `v1.0.0`).
- C. By emailing the code to HashiCorp support.
- D. By compiling the code with Go.

---

### Question 8
What is the function of the `terraform-docs` utility in modern module engineering workflows?
- A. It compiles Markdown files into PDF manuals.
- B. It automatically parses module inputs, outputs, providers, and requirements, generating standardized Markdown documentation tables in `README.md`.
- C. It audits security CVEs.
- D. It executes integration tests.

---

### Question 9
In `terraform test`, how can variables be passed to a specific `run` block without affecting subsequent `run` blocks?
- A. By setting environment variables in bash.
- B. By declaring a nested `variables {}` block inside the specific `run` block.
- C. By modifying `terraform.tfvars`.
- D. By using `-var` on the command line.

---

### Question 10
When using `postcondition` blocks inside a resource lifecycle, what keyword represents the resource's newly created or planned attributes within the condition expression?
- A. `each.value`
- B. `self`
- C. `this`
- D. `resource`

---

## Answer Key & Explanations

1. **B** — In `terraform test`, `command = plan` runs in-memory evaluation; `command = apply` creates physical resources and tears them down at the end of the test run.
2. **A** — `assert { condition = ... error_message = ... }` inside a `run` block tests planned outputs against logical conditions.
3. **A** — Terratest is a Go testing library that leverages Go's native testing framework and cloud client libraries to validate functional end-to-end behavior.
4. **B** — `check` blocks run validation assertions outside the critical path; failures produce warnings rather than halting execution.
5. **B** — The standard HashiCorp module layout specifies an `examples/` directory containing complete, runnable example roots.
6. **A** — Terraform 1.7 introduced `mock_provider`, allowing test authors to synthesize provider responses without making external network calls.
7. **B** — HCP Terraform and Terraform Enterprise registry modules must follow the three-part repository naming convention `terraform-<PROVIDER>-<NAME>`.
8. **B** — `terraform-docs` inspects HCL ASTs and maintains synchronized documentation for module interfaces.
9. **B** — Variables defined inside a `run` block scope apply exclusively to that execution step.
10. **B** — In `lifecycle { postcondition {} }`, `self` refers to the attributes of the resource being created or updated.
