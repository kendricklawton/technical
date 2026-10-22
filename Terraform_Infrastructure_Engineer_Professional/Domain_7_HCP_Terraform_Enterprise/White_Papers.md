# Domain 7: HCP Terraform / Enterprise Operations — Authoritative Documentation & White Papers

> **Official Blueprint Weight:** 10% of the 002 Professional Exam

---

## 1. Authoritative References

- **[HCP Terraform Architecture & Concepts](https://developer.hashicorp.com/terraform/cloud-docs):** Overview of Organizations, Projects, Workspaces, and Runs.
- **[Self-Hosted Agents (TFC Agent)](https://developer.hashicorp.com/terraform/cloud-docs/agents):** Deploying private runners inside isolated VPCs or on-premises networks.
- **[Run Tasks Integration](https://developer.hashicorp.com/terraform/cloud-docs/workspaces/settings/run-tasks):** Third-party security and cost validation hooks.
- **[Variable Sets Documentation](https://developer.hashicorp.com/terraform/cloud-docs/workspaces/variables/variable-sets):** Scoping shared configuration and credentials.

---

## 2. Staff SRE Enterprise Governance Architecture

1. **Project-Based Multi-Tenancy:** Structure HCP Terraform with clear Project boundaries reflecting organizational business units (e.g., `Core-Platform`, `Data-Engineering`, `Security-Operations`). Scope OIDC dynamic provider credentials at the Project level.
2. **Private Network Isolation via Agents:** For sensitive environments or on-premises VMware/OpenStack clusters, deploy high-availability pools of `tfc-agent` containers behind forward proxies without exposing internal network subnets to the public internet.
