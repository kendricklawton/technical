# Certified Cloud Native Platform Engineer (CNPE) — Master Preparation Workspace

[![Exam Code](https://img.shields.io/badge/Certification-CNCF_CNPE-0086FF.svg?logo=cncf&logoColor=white)](https://www.cncf.io/certification/cnpe/)
[![Linux Foundation](https://img.shields.io/badge/Authority-The_Linux_Foundation-003764.svg?logo=linuxfoundation&logoColor=white)](https://training.linuxfoundation.org/)
[![Exam Format](https://img.shields.io/badge/Format-100%25_Performance_Based_(2_Hours)-success.svg)](#exam-format--details)
[![Target Engine](https://img.shields.io/badge/Kubernetes-v1.31+-326CE5.svg?logo=kubernetes&logoColor=white)](https://kubernetes.io/)

A comprehensive, production-grade study curriculum and performance-based preparation workspace for the **Cloud Native Computing Foundation (CNCF)** and **The Linux Foundation's** premier hands-on certification: **Certified Cloud Native Platform Engineer (CNPE)**.

This certification validates senior and staff-level expertise in architecting, automating, operating, and securing cloud-native internal developer platforms (IDP) utilizing the CNCF ecosystem: **Argo CD & Rollouts, Flux v2, Crossplane, Backstage, OpenTelemetry, Prometheus Operator, Kyverno, Gatekeeper, External Secrets Operator (ESO), vCluster, KEDA, and Karpenter**.

---

## 1. Exam Blueprint & Domain Weightings

| Domain | Weight | Core Exam Competencies | Direct Links |
| :--- | :---: | :--- | :--- |
| **Domain 1: GitOps & Continuous Delivery** | **25%** | Argo CD core architecture, ApplicationSet generators (Git, Matrix, Cluster, List, PR), sync policies, diffing/ignoreExtraneous, Progressive Delivery with Argo Rollouts (Canary, Blue/Green, AnalysisTemplate, MetricProvider), Flux CD v2 (Kustomize/Helm controllers, multi-tenancy) | [Study Guide](Domain_1_GitOps_Continuous_Delivery/Study_Guide.md) \| [White Papers](Domain_1_GitOps_Continuous_Delivery/White_Papers.md) \| [Exercises](Domain_1_GitOps_Continuous_Delivery/Exercises.md) \| [Quiz 1](Domain_1_GitOps_Continuous_Delivery/Quiz_1.md) \| [Quiz 2](Domain_1_GitOps_Continuous_Delivery/Quiz_2.md) \| [Quiz 3](Domain_1_GitOps_Continuous_Delivery/Quiz_3.md) |
| **Domain 2: Self-Service Platform APIs & Automation** | **25%** | Crossplane architecture (Managed Resources, CompositeResourceDefinitions / XRDs, Compositions, Composition Functions, Environment Configs, Claims), Kubernetes Operator Pattern (CRD schema validation, controller-runtime reconciliation), Backstage Catalog entity models & Software Templates (`fetch:template`, `publish:github`) | [Study Guide](Domain_2_Self_Service_APIs_Automation/Study_Guide.md) \| [White Papers](Domain_2_Self_Service_APIs_Automation/White_Papers.md) \| [Exercises](Domain_2_Self_Service_APIs_Automation/Exercises.md) \| [Quiz 1](Domain_2_Self_Service_APIs_Automation/Quiz_1.md) \| [Quiz 2](Domain_2_Self_Service_APIs_Automation/Quiz_2.md) \| [Quiz 3](Domain_2_Self_Service_APIs_Automation/Quiz_3.md) |
| **Domain 3: Observability & Operations** | **20%** | OpenTelemetry (OTel) Collector pipelines (Receivers, Processors: memory_limiter, batch, transform; Exporters: OTLP), OTel Operator auto-instrumentation, Prometheus Operator (ServiceMonitor, PodMonitor, Probe, PrometheusRule alerts, Alertmanager configs), Grafana dashboard provisioning, SLO calculation & self-healing | [Study Guide](Domain_3_Observability_Operations/Study_Guide.md) \| [White Papers](Domain_3_Observability_Operations/White_Papers.md) \| [Exercises](Domain_3_Observability_Operations/Exercises.md) \| [Quiz 1](Domain_3_Observability_Operations/Quiz_1.md) \| [Quiz 2](Domain_3_Observability_Operations/Quiz_2.md) \| [Quiz 3](Domain_3_Observability_Operations/Quiz_3.md) |
| **Domain 4: Platform Architecture & Scalability** | **15%** | Multi-tenancy models (Hard vs soft isolation, virtual clusters with vCluster, Hierarchical Namespace Controller - HNC, ResourceQuotas, LimitRanges), Workload & Cluster Autoscaling (HPA v2, VPA, Event-driven KEDA ScaledObjects/ScaledJobs, Karpenter NodePool/NodeClass consolidation vs Cluster Autoscaler), Cluster API (CAPI) multi-cluster topology | [Study Guide](Domain_4_Platform_Architecture_Scalability/Study_Guide.md) \| [White Papers](Domain_4_Platform_Architecture_Scalability/White_Papers.md) \| [Exercises](Domain_4_Platform_Architecture_Scalability/Exercises.md) \| [Quiz 1](Domain_4_Platform_Architecture_Scalability/Quiz_1.md) \| [Quiz 2](Domain_4_Platform_Architecture_Scalability/Quiz_2.md) \| [Quiz 3](Domain_4_Platform_Architecture_Scalability/Quiz_3.md) |
| **Domain 5: Security, Governance & Policy Enforcement** | **15%** | Policy-as-Code in Kubernetes (Kyverno ClusterPolicies: validate, mutate, generate, verifyImages, CEL-based validation vs OPA/Gatekeeper ConstraintTemplates/Constraints), Secret Management (External Secrets Operator - ESO SecretStore, ClusterSecretStore, ExternalSecret, PushSecret, HashiCorp Vault integration, Sealed Secrets), Workload Identity (SPIFFE/SPIRE), supply chain security | [Study Guide](Domain_5_Security_Governance_Policy/Study_Guide.md) \| [White Papers](Domain_5_Security_Governance_Policy/White_Papers.md) \| [Exercises](Domain_5_Security_Governance_Policy/Exercises.md) \| [Quiz 1](Domain_5_Security_Governance_Policy/Quiz_1.md) \| [Quiz 2](Domain_5_Security_Governance_Policy/Quiz_2.md) \| [Quiz 3](Domain_5_Security_Governance_Policy/Quiz_3.md) |

---

## 2. Exam Format & Details

- **Exam Name:** Certified Cloud Native Platform Engineer (CNPE)
- **Credential Issuers:** Cloud Native Computing Foundation (CNCF) & The Linux Foundation
- **Exam Delivery:** 100% Performance-Based, Live Cluster Terminal Exam (Killer.sh style simulator included)
- **Duration:** 120 Minutes (2 Hours)
- **Passing Score:** 66% or higher
- **Retake Policy:** 1 Free Retake included with Linux Foundation registration
- **Target Platform Stack:**
  - Kubernetes v1.31+
  - GitOps: Argo CD v2.12+, Argo Rollouts v1.7+, Flux v2.3+
  - Control Planes & IDP: Crossplane v1.16+, Backstage v1.28+
  - Observability: OpenTelemetry Collector v0.108+, Prometheus Operator v0.76+
  - Autoscaling: KEDA v2.15+, Karpenter v1.0+, vCluster v0.20+
  - Policy & Secrets: Kyverno v1.12+, OPA Gatekeeper v3.17+, External Secrets Operator v0.10+

---

## 3. High-Value Resources & Lab Guides

- **Master Cheat Sheet:** [CNPE_Master_Cheat_Sheet.md](CNPE_Master_Cheat_Sheet.md)
- **DigitalOcean Droplet Lab Provisioning Guide:** [DigitalOcean_Droplet_Lab_Guide.md](DigitalOcean_Droplet_Lab_Guide.md) *(Automated `doctl` + Kubeadm + Argo CD + Crossplane + Kyverno + OTel installation)*
- **Performance Practice Exams:**
  - [Practice Exam 1 (2-Hour Performance Simulation)](Practice_Exams/Practice_Exam_1.md)
  - [Practice Exam 2 (2-Hour Performance Simulation)](Practice_Exams/Practice_Exam_2.md)
  - [Practice Exam 3 (2-Hour Performance Simulation)](Practice_Exams/Practice_Exam_3.md)
