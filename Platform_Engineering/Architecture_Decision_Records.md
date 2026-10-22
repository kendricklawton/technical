# Enterprise Architecture Decision Records (ADRs) — Platform Engineering & SRE

A collection of five production-grade Architecture Decision Records (ADRs) capturing architectural governance, trade-off analyses, and technology selections across enterprise platforms.

---

## ADR-001: Internal Developer Platform (IDP) Architecture
* **Status:** Accepted  
* **Date:** 2026-10-15  
* **Deciders:** Staff Platform Architect, VP of Engineering, Security Principal  
* **Technical Area:** Developer Experience (DevEx) & Software Catalog  

### Context and Problem Statement
Our engineering organization has grown to 120 stream-aligned teams operating 800+ microservices across 40 AWS accounts. Teams experience excessive extraneous cognitive load: onboarding a new microservice takes 3.5 weeks due to disparate manual ticket requests (requesting S3 buckets, IAM roles, DNS records, CI pipelines). Knowledge silos prevent security teams from identifying microservice ownership during critical CVE remediation.

### Evaluated Options
1. **Option A: Deploy and customize Spotify Backstage (Open Source)**
2. **Option B: Adopt a Commercial SaaS Developer Portal (Port / Cortex)**
3. **Option C: Build a Bespoke In-House CLI & Web Portal**

### Decision Outcome
**Chosen Option:** **Option A: Deploy Spotify Backstage** combined with Backstage Software Templates and custom plugins.

### Positive Consequences
- **Extensible Ecosystem:** Backstage provides hundreds of community-maintained open-source plugins (Kubernetes, TechDocs, GitHub Actions, PagerDuty, Argo CD).
- **Decentralized Ownership:** Metadata is maintained as code in application repositories (`catalog-info.yaml`), enabling stream-aligned teams to own service definitions without a centralized database.
- **Golden Path Automation:** Backstage Scaffolder templates automate repository creation, CI/CD pipeline instantiation, and cloud infrastructure bootstrapping in under 5 minutes.

### Negative Consequences & Trade-offs
- Requires dedicated platform team engineers (Node.js/React/TypeScript) to maintain, upgrade, and secure the Backstage host instances.
- High initial configuration complexity compared to commercial SaaS portals.

---

## ADR-002: Multi-Tenancy Strategy for Production Kubernetes Fleets
* **Status:** Accepted  
* **Date:** 2026-10-18  
* **Deciders:** Staff SRE, Principal Security Architect, Infrastructure Director  
* **Technical Area:** Kubernetes Infrastructure Architecture  

### Context and Problem Statement
Our cloud infrastructure incurs $180,000/month in Kubernetes control plane fees and worker node underutilization due to a legacy policy of provisioning dedicated EKS clusters for every individual microservice team (over 75 clusters active). However, running untrusted or high-compliance multi-tenant workloads in a single shared cluster risks noisy-neighbor resource starvation and security boundary breaches.

### Evaluated Options
1. **Option A: Soft Multi-Tenancy via Namespaces in Mega-Clusters**
2. **Option B: Virtual Clusters via vCluster on Shared Worker Fleets**
3. **Option C: Maintain Dedicated Multi-Cluster Fleet per Team**

### Decision Outcome
**Chosen Option:** **Hybrid Strategy: Option B (vCluster) for Non-Production & Option A (Hardened Namespaces) per Compliance Domain.**
- Group microservices by compliance boundary: General Workloads, PCI/Payment Workloads, and Data Analytics.
- Inside each boundary cluster, isolate teams via Namespaces reinforced with Pod Security Standards (`restricted`), Calico NetworkPolicies, and ResourceQuotas.
- In Development/Staging: deploy **vCluster** virtual control planes, giving each developer team full ClusterRole and CRD freedom on shared compute infrastructure.

### Positive Consequences
- Consolidates 75 clusters down to 6 regional clusters, reducing monthly cloud infrastructure spend by 58% ($104,000/month savings).
- Karpenter consolidation bin-packs worker nodes efficiently across teams.
- Preserves control plane isolation for dev teams without physical cluster overhead.

---

## ADR-003: Cloud Infrastructure as Code & Control Plane Modernization
* **Status:** Accepted  
* **Date:** 2026-10-19  
* **Deciders:** Staff Infrastructure Engineer, Staff Platform Engineer  
* **Technical Area:** Infrastructure Automation  

### Context and Problem Statement
Developers frequently request cloud backing services (Amazon S3 buckets, SQS queues, DynamoDB tables, RDS databases). Currently, developers submit requests via Jira tickets to the Platform Team, who run Terraform pipelines manually or via PRs. This creates a 4-day ticket bottleneck.

### Evaluated Options
1. **Option A: Crossplane Composable Control Planes in Kubernetes**
2. **Option B: Self-Service Terraform / OpenTofu PR Automation (Atlantis / Terraform Cloud)**
3. **Option C: AWS Service Catalog Console Portals**

### Decision Outcome
**Chosen Option:** **Hybrid Composable Architecture:**
- **OpenTofu / Terraform:** Used exclusively by the core Platform Team to provision foundational networking (VPCs, Transit Gateways, Direct Connect) and base EKS clusters.
- **Crossplane:** Deployed inside the EKS platform clusters to expose Composite Resource Definitions (XRDs) for application backing services (e.g. `CompositeDatabase`, `CompositeQueue`). Developers request infrastructure by committing standard Kubernetes manifests into Git.

### Positive Consequences
- **Continuous Reconciliation:** Crossplane continuously detects and corrects infrastructure drift automatically, unlike batch-run Terraform.
- **Developer Familiarity:** Developers configure cloud resources using the same YAML syntax and GitOps pipelines they use for Deployments and Services.
- **Zero Ticket Lead Time:** Microservice infrastructure is provisioned automatically alongside application deployment.

---

## ADR-004: Service Mesh vs Ingress & Gateway API Modernization
* **Status:** Accepted  
* **Date:** 2026-10-20  
* **Deciders:** Staff Network Architect, Principal SRE  
* **Technical Area:** Service-to-Service Networking & Traffic Management  

### Context and Problem Statement
Our microservice fleet requires mutual TLS (mTLS) encryption in transit to satisfy SOC 2 Type II compliance, along with canary traffic shifting (splitting 5% of traffic to new revisions based on HTTP headers) and distributed tracing. The team previously evaluated Istio with sidecar injection, but experienced severe memory overhead (each sidecar consumed 128 MB RAM, totaling 120 GB RAM across the cluster) and complex debugging cycles.

### Evaluated Options
1. **Option A: Heavyweight Service Mesh with Sidecars (Istio / Linkerd)**
2. **Option B: eBPF-Based CNI (Cilium) with In-Kernel mTLS & Kubernetes Gateway API**
3. **Option C: Application Load Balancers with Custom NGINX Ingress Controllers**

### Decision Outcome
**Chosen Option:** **Option B: Cilium eBPF + Kubernetes Gateway API.**
- Deploy **Cilium** as the CNI provider utilizing eBPF for in-kernel network routing, replacing kube-proxy.
- Enforce WireGuard transparent encryption at the CNI layer for automated node-to-node and pod-to-pod encryption without sidecar proxies.
- Use **Kubernetes Gateway API** (`Gateway` and `HTTPRoute`) implemented via Cilium Envoy integration for L7 canary traffic splitting and header routing at the ingress perimeter.

### Positive Consequences
- **Zero Sidecar Resource Tax:** Eliminates 120 GB of RAM wasted on sidecar proxies across worker fleets.
- **Superior Latency:** eBPF socket layer enforcement bypasses the TCP/IP stack on the host, reducing inter-pod latency by up to 30%.
- Transparent cryptographic compliance without developer involvement or Dockerfile sidecar injection.

---

## ADR-005: Enterprise Observability Pipeline Standardization
* **Status:** Accepted  
* **Date:** 2026-10-21  
* **Deciders:** Staff SRE, Principal Data Engineer  
* **Technical Area:** Observability, Metrics & Distributed Tracing  

### Context and Problem Statement
Our telemetry vendor bills (Datadog) have escalated to $140,000/month. Analysis shows that 82% of ingested traces are redundant HTTP 200 health checks, and debug logs are being streamed unthrottled. Direct proprietary SDKs in application code create vendor lock-in, making migration or multi-backend routing impossible.

### Evaluated Options
1. **Option A: Direct Proprietary Vendor SDKs (Datadog / New Relic Agents)**
2. **Option B: Unified OpenTelemetry Collector Pipeline with Tail-Based Sampling**
3. **Option C: In-House Elastic / Prometheus / Jaeger Stack**

### Decision Outcome
**Chosen Option:** **Option B: Standardize on OpenTelemetry (OTel) Collector with Tail-Based Sampling.**
- Mandate OpenTelemetry SDKs across all application codebases.
- Deploy OpenTelemetry Collector DaemonSets on worker nodes for local telemetry batching.
- Route traces to an OpenTelemetry Collector gateway cluster configured with **Tail-Based Sampling**:
  - Sample 100% of traces containing HTTP status codes $\ge 400$ or errors.
  - Sample 100% of traces exceeding P99 latency threshold ($>500$ ms).
  - Sample only 1% of normal, successful health checks and fast transactions.
- Filter, scrub PII, and route metrics to Prometheus/Mimir and traces to vendor/Tempo.

### Positive Consequences
- Reduces telemetry volume sent to cloud vendors by 78%, resulting in an immediate $95,000/month cost reduction.
- Complete vendor neutrality: switching telemetry backends requires only an OTel Collector configuration update, zero application code changes.
- Eliminates sensitive customer PII before telemetry leaves internal VPC boundaries.

---

## ADR-006: Kubernetes Node Autoscaling Architecture (Karpenter v1.0 vs Cluster Autoscaler)
* **Status:** Accepted  
* **Date:** 2026-10-18  
* **Deciders:** Principal Infrastructure Architect, Staff Platform Engineer, FinOps Director  
* **Technical Area:** Compute Orchestration, Capacity Management & Cloud Spend Optimization  

### Context and Problem Statement
Our Kubernetes clusters operate 450+ nodes across AWS and hybrid environments. Under Cluster Autoscaler, scaling response times average 4 to 6 minutes due to rigid Auto Scaling Group (ASG) instance sizing, leading to pod startup delays during flash traffic surges. Furthermore, static ASGs result in significant compute fragmentation and over-provisioning waste (\$38,000/month in idle CPU/memory).

### Evaluated Options
1. **Option A: Retain Kubernetes Cluster Autoscaler with Overprovisioning Pods**
   - *Pros:* Battle-tested, simple integration with AWS ASGs.
   - *Cons:* Slow spin-up latency (3–5 min); requires paying for permanent dummy "pause" pods to buffer capacity.
2. **Option B: Adopt Karpenter v1.0+ (Just-In-Time Provisioning)**
   - *Pros:* Direct interaction with cloud EC2 Fleet APIs (spins up nodes in 35–45 seconds); dynamically evaluates pending pod constraints and launches right-sized instances; automated consolidation (`WhenEmptyOrUnderutilized`) packs workloads and reclaims idle spend; native Spot diversification.
   - *Cons:* Deeper IAM role requirements; controller failure risk requires multi-replica HA.

### Decision Outcome
**Chosen Option:** **Option B: Migrate all EKS compute pools to Karpenter v1.0+**.

### Consequences
- **Positive:** Node provisioning latency drops from 300s to 40s; automated consolidation reduces monthly EC2 spend by 32%; eliminates management of dozens of static CloudFormation ASGs.
- **Negative / Risks:** Requires enforcing strict Pod Disruption Budgets (PDBs) across all application deployments to prevent consolidation disruption during business hours.

---

## ADR-007: Kubernetes Ingress & Service Networking (Gateway API vs Ingress NGINX vs AWS VPC Lattice)
* **Status:** Accepted  
* **Date:** 2026-10-19  
* **Deciders:** Staff Network Engineer, Platform Architect, AppSec Lead  
* **Technical Area:** L7 Ingress, Cross-Namespace Routing & Service Mesh Interconnect  

### Context and Problem Statement
Our application teams require canary traffic splitting, header-based routing, and cross-namespace ingress routing. Traditional Ingress v1 is constrained by non-standard vendor annotations (e.g. `nginx.ingress.kubernetes.io/canary`) that cannot be validated programmatically, and single-namespace coupling creates RBAC conflicts between infrastructure and product teams.

### Evaluated Options
1. **Option A: Ingress NGINX Controller with Custom Lua / Annotations**
   - *Pros:* Ubiquitous, widely understood.
   - *Cons:* Fragile annotation chaining; security vulnerabilities in ingress-nginx admission controllers; no role-oriented separation.
2. **Option B: Kubernetes Gateway API (implemented via Cilium / Envoy)**
   - *Pros:* Official Kubernetes standard; clean separation of roles (`GatewayClass` for infra, `Gateway` for cluster ops, `HTTPRoute` for app teams); native canary traffic weighting; cross-namespace routing secured by `ReferenceGrant`.
   - *Cons:* Requires upgrading cluster CRDs to Gateway API v1.1+.
3. **Option C: AWS VPC Lattice Service Network**
   - *Pros:* Fully managed AWS service-to-service networking across VPCs.
   - *Cons:* AWS-proprietary lock-in; does not run on on-premises or multi-cloud clusters.

### Decision Outcome
**Chosen Option:** **Option B: Adopt Kubernetes Gateway API** implemented via Cilium eBPF and Envoy.

### Consequences
- **Positive:** Standardizes declarative, portable traffic splitting; allows application teams to independently manage `HTTPRoutes` in their own namespaces without cluster-admin permissions; eliminates brittle ingress annotations.
- **Negative / Risks:** Platform team must provide internal developer training on Gateway API resource models and `parentRefs` conventions.
