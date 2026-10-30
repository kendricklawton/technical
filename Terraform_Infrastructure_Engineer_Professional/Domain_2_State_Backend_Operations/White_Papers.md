# Domain 2: State and Backend Operations — Reference White Papers & Architectural Deep Dives

## 1. Native S3 Conditional-Write State Locking (`use_lockfile = true`)
* **Primary Reference:** *HashiCorp Terraform 1.10+ Architecture Specification — S3 Native State Locking*
* **Core Architectural Takeaways:**
  * Amazon S3 added strong read-after-write consistency and conditional `PutObject` (`If-None-Match: "*"`) headers, allowing Terraform 1.10+ to write `<state_key>.tflock` atomically inside the S3 bucket itself.
  * Eliminating DynamoDB reduces attack surface, cross-service IAM complexity, and DynamoDB table provisioning dependencies during bootstrap ("chicken-and-egg" state bootstrapping).
  * IAM policies must distinguish between `*.tfstate` (where `s3:DeleteObject` should be denied to prevent state deletion) and `*.tflock` (where `s3:DeleteObject` **must be allowed** so locks can be released).

---

## 2. State Lineage, Serial Monotonicity & Split-Brain Prevention
* **Primary Reference:** *Terraform Internals — State Storage, Lineage Verification & Optimistic Concurrency Control*
* **Core Architectural Takeaways:**
  * Terraform uses a two-layer safety mechanism on state writes: **Active Distributed Locking** (preventing concurrent runs) and **Optimistic Concurrency Verification** via `lineage` (UUID) and `serial` (integer).
  * Even if a lock is forcefully broken via `terraform force-unlock`, if the crashed runner suddenly wakes up and attempts to flush its in-memory state, the backend write check detects if the remote `serial` has already been incremented by a newer run.

---

## 3. Cross-Stack Decoupling: Moving Beyond `terraform_remote_state`
* **Primary Reference:** *Enterprise Terraform Operating Model — Blast Radius Isolation & Output Sharing*
* **Core Architectural Takeaways:**
  * `data "terraform_remote_state"` violates least privilege by requiring read access to the entire producer `.tfstate` blob.
  * Enterprise architectures replace `terraform_remote_state` with either **HCP Terraform `tfe_outputs`** (API-scoped output sharing) or **Cloud-Native Parameter Registries** (AWS SSM Parameter Store, GCP Secret Manager/Runtime Config, Azure App Configuration) paired with read-only `data` source filters.
