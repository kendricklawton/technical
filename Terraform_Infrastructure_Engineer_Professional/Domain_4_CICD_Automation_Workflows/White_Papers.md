# Domain 4: CI/CD & Automation Workflows — Reference White Papers & Architectural Deep Dives

## 1. Cryptographic Supply Chain Integrity: `.terraform.lock.hcl` & GPG Provider Signing
* **Primary Reference:** *HashiCorp Security Architecture — Provider Checksum Verification & Lockfile Modes*
* **Core Architectural Takeaways:**
  * Using `terraform init -lockfile=readonly` in CI/CD pipelines guarantees that the runner will never silently modify `.terraform.lock.hcl` or accept an unverified provider binary if a platform hash is missing.
  * When using an internal `filesystem_mirror` or `network_mirror`, the mirror strips the original registry GPG signature check, making `.terraform.lock.hcl` `h1:` package directory hashes the primary cryptographic defense against tampered provider binaries.

---

## 2. Dynamic Provider Credentials via OpenID Connect (OIDC)
* **Primary Reference:** *HCP Terraform & Cloud Provider Workload Identity Federation Reference Architecture*
* **Core Architectural Takeaways:**
  * Static cloud credentials stored in CI variables remain valid even when no pipeline is running. OIDC JWT tokens are minted just-in-time for a single run and carry granular claims (`organization`, `project`, `workspace`, `run_phase`).
  * Scoping IAM trust policies by `run_phase:plan` vs. `run_phase:apply` enforces cryptographically verified privilege separation between pull-request validation and post-merge execution.
