# Mock Interview 5: System Design — Internal Developer Platform (IDP)

Simulates a 60-minute open-ended system design interview. Design a self-service IDP for an organization whose specifics (growth, compliance or consolidation pressures) are revealed only when the candidate asks.

---

## Candidate Brief (Safe to read in advance)

- **Duration:** 60 minutes.
- **Prompt:** Given at the start of the round: design an Internal Developer Platform for an engineering organization facing a specific scaling, compliance or consolidation problem.
- **Expected Deliverables:** High-level architecture diagram, API interface, developer onboarding workflow, Golden Path definition, and adoption/migration strategy.

---

## Interviewer Packet (Confidential — Do not read beforehand)

### Variant A: High-Growth Enterprise on AWS (80 → 400 Developers)
- **Prompt:** "Design an Internal Developer Platform for an engineering organization scaling from 80 to 400 developers over the next two years. Currently, provisioning a new microservice takes 2 weeks of manual tickets across database, security, and infrastructure teams."
- **Hidden Company Fact Sheet (Reveal ONLY when asked):**
  - *Workloads:* 85% stateless HTTP/gRPC services (Go, Java/Spring); 15% stateful data pipelines (Kafka, Spark).
  - *Infrastructure:* AWS (EKS, RDS Postgres, S3, SQS).
  - *Current Pain Points:* 14-day lead time to provision RDS and IAM roles; developers bypass security with hand-rolled Terraform; shadow ops on product teams.
  - *Compliance:* SOC 2 Type II audit in 6 months; all deployments must have audit trails and signed images.
  - *Portal Budget:* Only 2 platform engineers available initially; cannot afford 3 full-time engineers to maintain self-hosted Backstage.
- **Curveballs to Throw Mid-Interview:**
  1. *Minute 25:* "A product team wants to run a high-throughput C++ video engine that doesn't fit your standard Go/Java Golden Path. How does your platform handle this without blocking them?" (Evaluates **Escape Hatch** architecture).
  2. *Minute 40:* "The CFO announces cloud spend must be cut by 20% next quarter. How does your platform provide cost visibility and eliminate waste?" (Evaluates **FinOps / Karpenter consolidation**).

### Variant B: Regulated Fintech, Hybrid On-Prem + AWS (PCI DSS)
- **Prompt:** "Design an Internal Developer Platform for a payments company with about 250 engineers. Card processing runs in two on-prem data centers, and new services are supposed to go to AWS. Every production change goes through a Change Advisory Board and takes 9 business days on average. Leadership wants teams deploying daily without failing the next PCI DSS assessment."
- **Hidden Company Fact Sheet (Reveal ONLY when asked):**
  - *Workloads:* About 120 services. 30 are in PCI scope (the cardholder data environment) and run on VMware VMs and two self-managed Kubernetes clusters on-prem, three minor versions behind. The other 90 run on 3 EKS clusters in an AWS landing zone.
  - *Compliance:* PCI DSS v4.0.1. Every production change needs documented approval, test evidence and a rollback plan; no one may deploy their own unreviewed code; audit logs are kept for 12 months. The company's assessor (QSA) has confirmed that automated, pre-approved "standard changes" are acceptable if the pipeline captures that evidence. Nothing requires a CAB meeting.
  - *Network:* The cardholder data environment has no internet egress, so images and packages must come from internal mirrors. Firewall changes go through the network team with a 5-business-day SLA.
  - *Teams:* Platform team of 6. A security team of 4, who also run the CAB. Developers reach production through a shared break-glass account with standing `kubectl` access.
  - *Current Pain Points:* Change tickets are filled in by hand after the fact; nobody trusts the CMDB; the last assessment raised 14 findings about missing change evidence.
- **Curveballs to Throw Mid-Interview:**
  1. *Minute 25:* "The assessor flags that developers can `kubectl exec` into production pods in the PCI clusters. Fix it without making production impossible to debug." (Evaluates **just-in-time access, break-glass and audit**).
  2. *Minute 40:* "A critical OpenSSL CVE is published on a Friday afternoon. Under your design, how fast can every PCI-scope service be patched, and what evidence does the assessor see?" (Evaluates **base-image ownership, automated rebuilds, SBOM inventory and an emergency change path**).

### Variant C: Post-Acquisition Platform Merger (GCP Cloud Run + AWS EKS)
- **Prompt:** "Your company (AWS and EKS, about 350 engineers) just acquired a competitor (GCP and Cloud Run, about 250 engineers). The CTO has promised the board 'one platform' within 18 months. Design the target platform and the path to it without stalling either organization's product roadmap."
- **Hidden Company Fact Sheet (Reveal ONLY when asked):**
  - *Acquirer:* 6 EKS clusters, Argo CD, Terraform, a self-hosted Backstage portal maintained by 2 engineers, Datadog, Okta.
  - *Acquired company:* About 140 services on Cloud Run, deployed by Cloud Build triggers, using Pub/Sub and Cloud SQL, with Google Workspace identity. Their developers love it ("merge and it's live in 4 minutes"), and their DORA metrics are better than the acquirer's.
  - *Contracts:* A GCP committed-use discount runs for another 30 months; leaving GCP early wastes committed spend.
  - *People:* The acquired platform team had 5 engineers, and 2 have already resigned. Retention bonuses vest in 12 months.
  - *What leadership actually needs:* One security and audit posture, one identity system and on-call model, and lower combined cost. "One platform" was shorthand in a board deck, not a requirement to run a single runtime.
- **Curveballs to Throw Mid-Interview:**
  1. *Minute 25:* "The acquired company's strongest platform engineer tells you privately they'll quit if Cloud Run is replaced with Kubernetes. What do you do?" (Evaluates **people judgment and whether the design depends on a single runtime**).
  2. *Minute 40:* "Finance confirms the GCP commitment can't be broken, and now wants a 15% cost reduction across both clouds within 12 months. How does that change your sequencing?" (Evaluates **migration economics and FinOps sequencing**).

---

## Scoring Rubric

| Score | Anchor | Behavioral Criteria |
| :---: | :--- | :--- |
| **1** | Strong No Hire | Immediately jumps to installing Backstage without asking clarifying questions; ignores cloud automation. |
| **2** | No Hire | Designs shared infrastructure but fails to make it self-service; requires tickets. |
| **3** | Hire | Clarifies requirements; designs clean 5-plane IDP; handles the variant's curveballs (escape hatch and FinOps in A, production access and patch evidence in B, people and migration sequencing in C). |
| **4** | Strong Hire | Masterful product-led platform design; balances Thinnest Viable Platform (TVP) with long-term enterprise scale. In B, reframes change control as evidence the pipeline produces; in C, unifies the developer interface and controls before touching runtimes. |

---

## Self-Debrief Template
- **Variant:** A / B / C
- **Clarifying Questions Asked:**
- **Pacing Across Phases:**
- **Curveball Handling Score (1-4):**
- **Action Items:**
