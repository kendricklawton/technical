# Certified Cloud Native Platform Engineer (CNPE) — Practice Exams

This directory contains five full-length, 120-minute performance-based practice examinations engineered for the official **Linux Foundation / CNCF Certified Cloud Native Platform Engineer (CNPE)** examination.

---

## 1. Practice Exam Directory & Links

| Exam | Focus & Scope | Tasks | Time Limit | Target Score | Link |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **Practice Exam 1** | Argo CD Auto-Sync, ApplicationSet Matrix, Argo Rollouts Prometheus Canary, Crossplane XRD, Kyverno, ESO | 15 Tasks | 120 mins | $\ge 80\%$ | [**Open `Practice_Exam_1.md`**](Practice_Exam_1.md) |
| **Practice Exam 2** | S3 Managed Bucket Composition, Blue-Green Pre-Promotion Analysis, KEDA ScaledJob, HNC Subnamespaces | 15 Tasks | 120 mins | $\ge 80\%$ | [**Open `Practice_Exam_2.md`**](Practice_Exam_2.md) |
| **Practice Exam 3** | Multi-Cluster DR Failover, Crossplane Function Pipelines, OTel OTTL PII Redaction, Gateway API Traffic Shifting | 15 Tasks | 120 mins | $\ge 80\%$ | [**Open `Practice_Exam_3.md`**](Practice_Exam_3.md) |
| **Practice Exam 4** | PreSync DB Migration Hooks, Crossplane Connection Secrets, PolicyException Exemptions, KEDA Redis Streams | 15 Tasks | 120 mins | $\ge 80\%$ | [**Open `Practice_Exam_4.md`**](Practice_Exam_4.md) |
| **Practice Exam 5** | Crossplane Subresource Statuses, OTel Load-Balancing Exporters, Cilium ClusterMesh, Karpenter NVMe UserData | 15 Tasks | 120 mins | $\ge 80\%$ | [**Open `Practice_Exam_5.md`**](Practice_Exam_5.md) |

---

## 2. CNPE Exam Blueprint & Scoring Matrix

| Domain | Weight | Key Competencies Tested |
| :--- | :---: | :--- |
| **Domain 1: GitOps & Continuous Delivery** | **25%** | Argo CD core sync policies, ApplicationSets (Git, Cluster, Matrix generators), Argo Rollouts (Canary, Blue-Green, AnalysisTemplates, MetricProviders), Flux CD v2. |
| **Domain 2: Self-Service Platform APIs & Automation** | **25%** | Crossplane Composite Resource Definitions (XRDs), Compositions, Composition Functions (P&T), ProviderConfigs, Backstage Software Templates. |
| **Domain 3: Observability & Operations** | **20%** | OpenTelemetry Collector pipelines (Receivers, Processors, Exporters, Connectors), Tail-based sampling, Prometheus Operator (ServiceMonitors, PrometheusRules), SLO burn-rate alerts. |
| **Domain 4: Platform Architecture & Scalability** | **15%** | KEDA event-driven autoscaling (Kafka, RabbitMQ, Redis, SQS), Karpenter v1.0 NodePools and consolidation, Kubernetes Gateway API, vCluster multi-tenancy. |
| **Domain 5: Security, Governance & Policy Enforcement** | **15%** | Kyverno ClusterPolicies (Validate, Mutate, Generate, VerifyImages), External Secrets Operator (ESO), Pod Security Standards (PSS/PSA), Sigstore/Cosign image signing, Cilium L7 NetworkPolicies. |
