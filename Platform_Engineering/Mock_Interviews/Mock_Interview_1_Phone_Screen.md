# Mock Interview 1: Technical Phone Screen

Simulates a 45-minute rapid-fire technical phone screen testing breadth across Linux, networking, Kubernetes, IaC, CI/CD, and SRE, with calibrated depth probes.

---

## Candidate Brief (Safe to read in advance)

- **Duration:** 45 minutes.
- **Format:** Fast-paced conversational screen. The interviewer will ask 8–12 questions across core domains, followed by drill-down probes depending on your depth.
- **Target Pace:** 2–3 minutes per question. Lead with the core concept, then explain the mechanism, and name the trade-off.
- **Evaluation Criteria:** Breadth of knowledge, communication clarity, depth under probing, avoiding bluffing.

---

## Interviewer Packet (Confidential — Do not read beforehand)

### Variant A
- **Q1 (Linux/Memory):** What is the difference between Virtual Memory (VIRT) and Resident Memory (RES)? What causes an OOM kill?
  - *Probe:* How does Kubernetes `oom_score_adj` prioritize Guaranteed vs BestEffort pods?
- **Q2 (Networking/DNS):** What causes the `ndots:5` latency penalty in Kubernetes?
  - *Probe:* How does NodeLocal DNSCache resolve this?
- **Q3 (Kubernetes/API):** What is Optimistic Concurrency Control, and what does HTTP 409 Conflict mean in `kube-apiserver`?
- **Q4 (IaC/State):** How do declarative `moved` blocks prevent resource destruction during Terraform refactoring?
- **Q5 (CI/CD/GitOps):** Why is pull-based GitOps more secure than push-based CI deployment?
- **Q6 (Observability/PromQL):** Why should you use `rate()` instead of `irate()` for alerting rules?
- **Q7 (Security/Identity):** How does AWS EKS Pod Identity work compared to IRSA?
- **Q8 (Platform/Topologies):** What is the difference between a Stream-aligned team and a Platform team in Team Topologies?

### Variant B
- **Q1 (Linux/CPU):** Explain CFS CPU quotas and how multi-threaded applications experience CFS throttling.
- **Q2 (Networking/TCP):** What is the purpose of the `TIME_WAIT` state, and how does it differ from `CLOSE_WAIT`?
- **Q3 (Kubernetes/Machinery):** What is a Finalizer, and what happens when an object with a finalizer is deleted?
- **Q4 (IaC/Testing):** How does `terraform test` work natively with mock providers?
- **Q5 (CI/CD/SupplyChain):** How does Sigstore Cosign perform keyless image signing using OIDC?
- **Q6 (Observability/SLO):** Explain the mathematical formula for a 14.4x multi-burn-rate SLO alert.
- **Q7 (Security/Secrets):** How does the External Secrets Operator decouple secret storage from GitOps?
- **Q8 (Platform/Product):** What is a Golden Path, and how do you prevent it from becoming a Golden Cage?

### Variant C
- **Q1 (Linux/Processes):** What is PID 1 in a container, and why must it reap zombie processes?
- **Q2 (Networking/TLS):** How does TLS 1.3 achieve a 1-RTT handshake compared to TLS 1.2?
- **Q3 (Kubernetes/Policy):** Why does `ValidatingAdmissionPolicy` replace dynamic admission webhooks in K8s 1.30+?
- **Q4 (IaC/Architecture):** How does Crossplane continuous reconciliation compare to Terraform pipeline execution?
- **Q5 (CI/CD/Canary):** How does Argo Rollouts evaluate automated canary analysis using Prometheus?
- **Q6 (Observability/Tracing):** What is W3C Trace Context and what is encoded in `traceparent`?
- **Q7 (FinOps/Cost):** What is the "Idle Capacity Gap" between Kubernetes CPU requests and utilization?
- **Q8 (Platform/Metrics):** What are the 4 core DORA metrics, and what constitutes elite delivery?

### Hint Ladders
- *Level 1 (Nudge):* "Think about how the Linux kernel allocates resources..."
- *Level 2 (Guiding Question):* "How does `resolv.conf` evaluate relative search domains?"
- *Level 3 (Explicit Cue):* "Remember that in Kubernetes, `ndots` defaults to 5 dots."

---

## Scoring Rubric

| Score | Anchor | Behavioral Criteria |
| :---: | :--- | :--- |
| **1** | Strong No Hire | Inaccurate fundamentals; cannot explain basic processes, memory, or GitOps. |
| **2** | No Hire | Answers surface questions correctly but struggles under depth probing; superficial recall. |
| **3** | Hire | Clear, structured answers across all domains; handles probes fluently; names trade-offs unprompted. |
| **4** | Strong Hire | Exceptional depth; explains kernel primitives and distributed systems nuances; teaches the interviewer. |

---

## Self-Debrief Template
- **Elapsed Time:** ___ min
- **Questions Answered:** ___ / 8
- **Top 2 Strengths:**
- **Top 2 Knowledge Gaps:**
- **Action Items:**
