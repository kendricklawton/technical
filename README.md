# Technical

A collection of technical concepts, hands-on lab architectures, certifications, and engineering references.

---

## Technical Tracks

### 1. [Certified Cloud Native Platform Engineer (CNPE)](./Certified_Cloud_Native_Platform_Engineer/README.md)
- **Profile:** 100% Performance-Based Terminal Exam (2 Hours).
- **Core Stack:** Kubernetes v1.31+, Argo CD, Argo Rollouts, Crossplane, Backstage, OpenTelemetry, Kyverno, External Secrets Operator (ESO), KEDA, Karpenter, Gateway API, Cilium eBPF, vCluster.
- **Key Resources:**
  - [CNPE Master Cheat Sheet](./Certified_Cloud_Native_Platform_Engineer/CNPE_Master_Cheat_Sheet.md)
  - [DigitalOcean Droplet Lab Bootstrap Guide](./Certified_Cloud_Native_Platform_Engineer/DigitalOcean_Droplet_Lab_Guide.md)
  - [Domain 1: GitOps & Continuous Delivery (25%)](./Certified_Cloud_Native_Platform_Engineer/Domain_1_GitOps_Continuous_Delivery/Study_Guide.md)
  - [Domain 2: Self-Service Platform APIs & Automation (25%)](./Certified_Cloud_Native_Platform_Engineer/Domain_2_Self_Service_APIs_Automation/Study_Guide.md)
  - [Domain 3: Observability & Operations (20%)](./Certified_Cloud_Native_Platform_Engineer/Domain_3_Observability_Operations/Study_Guide.md)
  - [Domain 4: Platform Architecture & Scalability (15%)](./Certified_Cloud_Native_Platform_Engineer/Domain_4_Platform_Architecture_Scalability/Study_Guide.md)
  - [Domain 5: Security, Governance & Policy Enforcement (15%)](./Certified_Cloud_Native_Platform_Engineer/Domain_5_Security_Governance_Policy/Study_Guide.md)
  - [Full-Length Practice Exams Suite (5 Mock Exams)](./Certified_Cloud_Native_Platform_Engineer/Practice_Exams/README.md)

---

### 2. [Terraform Infrastructure Engineer Professional (002)](./Terraform_Infrastructure_Engineer_Professional/README.md)
- **Profile:** Scenario exam launching at IBM TechXchange 2026 in Atlanta.
- **Key Resources:**
  - [Terraform Master Cheat Sheet](./Terraform_Infrastructure_Engineer_Professional/Terraform_Master_Cheat_Sheet.md)
  - [Domain 1: Scalable Configuration Design](./Terraform_Infrastructure_Engineer_Professional/Domain_1_Scalable_Configuration_Design/Study_Guide.md)
  - [Domain 2: State & Backend Operations](./Terraform_Infrastructure_Engineer_Professional/Domain_2_State_Backend_Operations/Study_Guide.md)
  - [Domain 3: Lifecycle Operations](./Terraform_Infrastructure_Engineer_Professional/Domain_3_Lifecycle_Operations/Study_Guide.md)
  - [Domain 4: CI/CD & Automation Workflows](./Terraform_Infrastructure_Engineer_Professional/Domain_4_CICD_Automation_Workflows/Study_Guide.md)
  - [Domain 5: Security, Governance & Compliance](./Terraform_Infrastructure_Engineer_Professional/Domain_5_Security_Governance/Study_Guide.md)
  - [Domain 6: Advanced Modules & Testing Frameworks](./Terraform_Infrastructure_Engineer_Professional/Domain_6_Advanced_Modules_Testing/Study_Guide.md)
  - [Domain 7: HCP Terraform & Terraform Enterprise](./Terraform_Infrastructure_Engineer_Professional/Domain_7_HCP_Terraform_Enterprise/Study_Guide.md)
  - [Full-Length Practice Exams Suite (4 Comprehensive Exams)](./Terraform_Infrastructure_Engineer_Professional/Practice_Exams/README.md)

---

### 3. [Certified Kubernetes Administrator (CKA)](./Certified_Kubernetes_Administrator/README.md)
- **Profile:** 100% Performance-based CLI Exam (2 Hours).
- **Core Coverage:** Cluster Architecture, Installation, Workloads & Scheduling, Networking, Storage, and Troubleshooting.
- **Lab Infrastructure:** DigitalOcean Droplets multi-node architecture (VPC private networking on `eth1`).
- **Practice Suite:** [5 Complete Performance Exams](./Certified_Kubernetes_Administrator/Practice_Exams/README.md) (85 real-world cluster tasks).

---

### 4. [Certified Kubernetes Security Specialist (CKS)](./Certified_Kubernetes_Security_Specialist/README.md)
- **Profile:** 100% Performance-based Security Hardening Exam (2 Hours).
- **Core Coverage:** Cluster Setup, Hardening, System Hardening (AppArmor, Seccomp), Supply Chain Security (Cosign, Trivy), Monitoring & Logging (Falco, audit logs, Tetragon eBPF).
- **Lab Infrastructure:** DigitalOcean Droplets multi-node architecture (`cks-cp1`, `cks-worker1`, `cks-worker2`).
- **Practice Suite:** [5 Complete Performance Exams](./Certified_Kubernetes_Security_Specialist/Practice_Exams/README.md) (80 real-world security tasks).

---

### 5. [Platform Engineering & Staff SRE Track](./Platform_Engineering/README.md)
- **Target Role:** Staff SRE / Staff Platform Engineer / Principal Cloud Architect.
- **Core Coverage:** Internal Developer Platforms (Backstage, Port), Golden Paths, Multi-cluster Fleet Management, Service Meshes (Cilium, Istio), Advanced GitOps, FinOps, SLO Engineering.
- **Staff Artifacts & War Rooms:**
  - [Staff SRE Production Playbook](./Platform_Engineering/Staff_Track/Staff_SRE_Playbook.md)
  - [Incident War Room Postmortems (7 SEV-1 Outages Analyzed)](./Platform_Engineering/Incident_War_Room_Postmortems.md)
  - [Enterprise Architecture Decision Records (7 Production ADRs)](./Platform_Engineering/Architecture_Decision_Records.md)
  - [Staff Career Track & Evidence Log](./Platform_Engineering/Staff_Track/README.md)

---

### 6. [AWS Certified Solutions Architect Professional (SAP-C02)](./AWS_Solutions_Architect_Professional/README.md)
- **Profile:** Multi-account architectures, hybrid cloud, disaster recovery, migration strategies, and complex networking.
- **Practice Suite:** [6 Complete Scenario Practice Exams](./AWS_Solutions_Architect_Professional/Practice_Exams/README.md) (120 multi-paragraph scenario questions with full distractor analysis).

---

### 7. [AWS Certified DevOps Engineer Professional (DOP-C02)](./AWS_DevOps_Professional/README.md)
- **Profile:** SDLC automation, configuration management, infrastructure-as-code, monitoring, policies automation, incident response.
- **Practice Suite:** [6 Complete Scenario Practice Exams](./AWS_DevOps_Professional/Practice_Exams/Practice_Exam_1.md) (150 scenario questions).

---

## Multi-Disciplinary & Non-Technical Foundations

Staff and Principal leadership requires driving impact beyond pure code and infrastructure. This section outlines the non-technical foundations integrated throughout this workspace:

1. **Executive Communication & Influence Without Authority:** Writing clear Architecture Decision Records (ADRs) and RFCs, communicating business impact to leadership, and leading blameless postmortems.
   - *References:* [Staff SRE Playbook](./Platform_Engineering/Staff_Track/Staff_SRE_Playbook.md) | [Incident War Rooms](./Platform_Engineering/Incident_War_Room_Postmortems.md) | [Domain 10 Study Guide](./Platform_Engineering/Domain_10_Behavioral_Career/Study_Guide.md)
2. **Business Acumen & Cloud FinOps:** Connecting latency and reliability to business revenue, designing cost allocation models, and optimizing unit economics.
   - *Reference:* [FinOps & Governance](./Platform_Engineering/Domain_7_Security_Governance_FinOps/Study_Guide.md)
3. **Organizational Design & Team Topologies:** Applying Conway's law, managing team cognitive load, and building self-service internal developer platforms.
   - *Reference:* [Platform Foundations & IDP](./Platform_Engineering/Domain_1_Platform_Foundations_IDP/Study_Guide.md)
4. **Decision Science & Risk Modeling:** Evaluating reversible vs. irreversible decisions (Type 1 vs. Type 2), modeling second-order effects, and managing statistical error budgets.
   - *Reference:* [Architecture Decision Records](./Platform_Engineering/Architecture_Decision_Records.md)
5. **Behavioral Dynamics & Career Strategy:** Structuring communication with the STAR method, managing upward alignment, and accumulating verifiable impact evidence.
   - *References:* [Staff Evidence Log](./Platform_Engineering/Staff_Track/README.md) | [Mock Interview Protocol](./Platform_Engineering/Mock_Interviews/Interviewer_Protocol.md)

---

## Lab Architecture & Operational Standards

- **Zero-Trust Baseline:** Default-deny network policies, mutual TLS encryption, ephemeral credentials, and least-privilege RBAC.
- **Infrastructure-as-Code & GitOps:** Automated provisioning with Terraform and declarative continuous delivery via Argo CD.
- **DigitalOcean Droplet Compatibility:** Low-cost multi-node lab environments running Ubuntu 24.04 LTS on private VPC interfaces (`eth1`).
