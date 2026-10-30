# Domain 2: Self-Service Platform APIs & Automation — Authoritative White Papers & Standards

> **CNPE Exam Weight:** 25% of Certified Cloud Native Platform Engineer

---

## 1. Authoritative References & Standards

- **[CNCF Platforms White Paper](https://tag-app-delivery.cncf.io/whitepapers/platforms/):** The foundational document defining platform engineering, internal developer platforms (IDP), cognitive load reduction, and platform as a product.
- **[Crossplane Architecture Specifications](https://docs.crossplane.io/):** Composite Resource Definitions (XRDs), Compositions, Composition Functions, Environment Configs, and Managed Resources.
- **[Kubernetes Operator Best Practices](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/):** Level-triggered reconciliation, status subresources, and admission webhooks.
- **[Backstage Software Templates & Catalog Specifications](https://backstage.io/docs/features/software-templates/):** Scaffolder actions, entity model taxonomy (System, Domain, Component, API, Resource).

---

## 2. Staff SRE Design Patterns

1. **Abstraction Level Invariants:** Never expose raw cloud primitives (e.g. `subnet_ids`, `vpc_peering_connections`, `security_group_ids`) directly to developers in self-service claims. Provide high-level business abstractions (e.g. `tier: internal-standard`, `storageGB: 50`) and let Compositions bind those claims to hardened, compliant cloud resources.
2. **Composition Functions:** In modern Crossplane v1.14+, prefer **Composition Functions** (written in Go, Python, or KCL) over massive YAML patch-and-transform files for complex conditionals and loops.
3. **Golden Paths over Golden Cages:** Provide fully automated self-service templates in Backstage for standard stacks, but design the underlying Crossplane APIs so advanced teams can consume them directly via `kubectl` or GitOps.
