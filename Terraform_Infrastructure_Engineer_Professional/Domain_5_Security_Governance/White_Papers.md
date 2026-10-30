# Domain 5: Security, Governance & Policy-as-Code — Reference White Papers & Architectural Deep Dives

## 1. Ephemeral Plugin Protocol & Write-Only State Redaction
* **Primary Reference:** *Terraform Plugin Protocol v6 — Ephemeral Resource Lifecycle & Write-Only Schema Specification*
* **Core Architectural Takeaways:**
  * Ephemeral resources introduce three new RPC verbs (`OpenEphemeralResource`, `RenewEphemeralResource`, `CloseEphemeralResource`) that execute outside the state persistence pipeline.
  * Write-only attributes (`WriteOnly: true` in `terraform-plugin-framework`) are stripped by Terraform Core before writing both the saved plan (`tfplan`) and the state snapshot (`terraform.tfstate`), eliminating the historic need to treat `.tfstate` as an unencrypted secret store for supported attributes.

---

## 2. Sentinel vs. Open Policy Agent (OPA) in Enterprise Governance
* **Primary Reference:** *HashiCorp Governance Framework — Policy-as-Code Enforcement Models*
* **Core Architectural Takeaways:**
  * **Enforcement Granularity:** `advisory` (audit/educate without blocking), `soft-mandatory` (blocks apply unless overridden by an authorized Security/Platform team member with an audit log justification), and `hard-mandatory` (unconditional block; zero override path).
  * **Import Selection:** Always match the Sentinel import to the artifact being validated: `tfconfig/v2` for HCL syntax/module sources, `tfplan/v2` for planned diffs and `after` values, `tfstate/v2` for existing state resources, and `tfrun` for workspace metadata and cost estimation.
