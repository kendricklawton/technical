# Domain 2: State and Backend Operations — Authoritative Documentation & White Papers

> **Official Blueprint Weight:** 18% of the 002 Professional Exam

---

## 1. Authoritative References

- **[Terraform State Internals](https://developer.hashicorp.com/terraform/language/state):** Why state exists, metadata mapping, performance caching, and concurrency locks.
- **[Remote State Backends](https://developer.hashicorp.com/terraform/language/settings/backends):** S3, GCS, AzureRM, Kubernetes Secrets, and HCP Terraform.
- **[Drift Detection and Reconciliation](https://developer.hashicorp.com/terraform/tutorials/state/refresh):** `-refresh-only` execution workflows.
- **[State Surgery Commands](https://developer.hashicorp.com/terraform/cli/commands/state):** `state mv`, `state rm`, `state pull`, `state push`, and `force-unlock`.

---

## 2. Staff SRE Production Playbook

1. **State Isolation by Blast Radius:** Never store an entire organization's cloud estate in a single state file. Partition state into discrete layers:
   - `core-network/` (VPC, subnets, route tables, peering)
   - `shared-services/` (DNS, IAM roles, KMS keys, Bastion hosts)
   - `workloads/` (Kubernetes clusters, autoscaling groups)
   - `applications/` (Microservices, database instances)
2. **State Storage Encryption:** State files can contain sensitive plaintext strings (passwords, TLS private keys). Always enable client-side or server-side encryption with strict IAM policies restricting `s3:GetObject` or GCS bucket access.
