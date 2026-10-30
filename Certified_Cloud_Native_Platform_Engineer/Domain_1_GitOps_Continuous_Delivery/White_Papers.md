# Domain 1: GitOps & Continuous Delivery — Authoritative White Papers & CNCF Standards

> **CNPE Exam Weight:** 25% of Certified Cloud Native Platform Engineer

---

## 1. Authoritative References & Standards

- **[OpenGitOps Principles & Standards (CNCF)](https://opengitops.net/):** The 4 canonical principles of declarative cloud-native reconciliation.
- **[Argo CD Core Architecture](https://argo-cd.readthedocs.io/):** Repository management, multi-cluster topology, diffing strategies, and ApplicationSets.
- **[Argo Rollouts Progressive Delivery](https://argo-rollouts.readthedocs.io/):** Canary, Blue-Green, AnalysisTemplates, MetricProviders, and Traffic Routers (NGINX, Istio, Envoy, Gateway API).
- **[Flux CD v2 Architecture](https://fluxcd.io/flux/components/):** Source, Kustomize, Helm, and Image Automation controller mechanics.

---

## 2. Staff SRE Architecture Patterns

1. **Self-Healing vs Drift Loops:** Never let manual interventions linger in production. Always configure `automated: { prune: true, selfHeal: true }` in Argo CD. If a developer runs an imperative `kubectl edit`, Argo CD must revert the drift within 3 minutes.
2. **HPA / KEDA Drift Mitigation:** When pairing autoscalers with GitOps, always configure `ignoreDifferences` on `/spec/replicas` so that dynamic scaling events are not overwritten by Git commits.
3. **Automated Canary Gates:** Never promote production code based on arbitrary wait timers alone (`pause: { duration: 1h }`). Pair pause intervals with automated `AnalysisTemplate` metrics querying live error rates and latency $P99$.
