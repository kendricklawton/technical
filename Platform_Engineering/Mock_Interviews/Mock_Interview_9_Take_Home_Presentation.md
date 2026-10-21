# Mock Interview 9: Take-Home & Architecture Defense Presentation

Simulates a 60-minute take-home project defense and Architecture Review Board (ARB) presentation. Tests written architectural rigor, live presentation skills, and the ability to defend technical decisions under hostile questioning.

---

## Candidate Brief (Safe to read in advance)

- **Duration:** 60 minutes (20 min presentation, 35 min cross-examination, 5 min wrap-up).
- **Format:** You will present an Architecture Decision Record (ADR) and platform design for an enterprise scenario, followed by rigorous technical probing by the architecture committee.
- **Deliverables Required:**
  1. Executive Summary & Problem Framing.
  2. High-Level Architecture Diagram (C4 model or Mermaid).
  3. Formal ADR (Context, Decision, Trade-offs, Consequences).
  4. Migration & Rollout Plan with DORA success metrics.

---

## Interviewer Packet (Confidential — Do not read beforehand)

### Scenario A: Enterprise Microservices Platform Modernization
- **Scenario:** A legacy fintech enterprise running 200 monolithic Java applications on VMware VMs must migrate to an Internal Developer Platform on Kubernetes within 12 months.
- **Defense Probing Attacks:**
  - *Security Grill:* "You selected GitOps with Argo CD. How do you prevent a malicious developer from opening a PR modifying production cluster RBAC or exposing an internal service to the public internet?"
  - *FinOps Grill:* "Your architecture uses multi-tenant EKS clusters with Karpenter. How do you provide exact, auditable cost attribution per business unit down to the penny for finance?"
  - *Disaster Recovery Grill:* "If the AWS control plane goes down in our primary region, how do our applications failover without database split-brain?"

### Scenario B: Ephemeral Preview Environments at Scale
- **Scenario:** An e-commerce engineering org with 250 engineers experiences a massive bottleneck where PRs take 6 days to merge because staging environments are constantly broken or shared.
- **Defense Probing Attacks:**
  - *Data Isolation Grill:* "How do your ephemeral preview environments handle state? If an engineer needs a 500GB Postgres database to test a schema change, how do you provide that in under 60 seconds without ballooning cloud storage bills?"
    - *Expected answer:* Database branch virtualization (e.g., Neon / Neon-like copy-on-write snapshots, or sanitized synthetic seed databases).
  - *Cost Control Grill:* "What prevents developers from leaving 50 ephemeral environments running over the weekend and burning $50,000?"
    - *Expected answer:* Aggressive TTL sweeps (Kube-downscaler / automated lifecycle controllers terminating clusters after 4 hours of inactivity).

---

## Scoring Rubric

| Score | Anchor | Behavioral Criteria |
| :---: | :--- | :--- |
| **1** | Strong No Hire | Slides are disorganized; cannot defend basic architecture choices; becomes defensive or hostile under questioning. |
| **2** | No Hire | Good presentation, but falls apart when probed on data consistency, cost reality, or enterprise compliance. |
| **3** | Hire | Clear, structured presentation; acknowledges trade-offs openly; defends decisions with sound engineering rationale and business ROI. |
| **4** | Strong Hire | Masterclass presentation; anticipates interviewer objections before they are spoken; provides battle-tested production figures and contingency plans. |

---

## Self-Debrief Template
- **Presentation Timing:** ___ min (Target: 18-20 min)
- **Hardest Question Encountered:**
- **How Well Trade-offs Were Defended (1-4):**
- **Action Items:**
