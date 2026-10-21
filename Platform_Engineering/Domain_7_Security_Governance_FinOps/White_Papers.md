# Domain 7: Security, Governance & FinOps — Essential Technical Documentation & Guide

> **Core Focus:** Zero Trust Architecture, Workload Identity (SPIFFE/SPIRE), Kubernetes Security Posture, Policy-as-Code (Kyverno / Gatekeeper), Supply Chain Security, and Cloud FinOps Framework.

---

## 1. Authoritative Frameworks & Security Standards

### 1.1 [NIST SP 800-207: Zero Trust Architecture](https://csrc.nist.gov/publications/detail/sp/800-207/final)
* **Canonical Scope:** Core cybersecurity architecture moving away from perimeter-based defenses to continuous verification.
* **Core Tenets:**
  * Never trust, always verify.
  * All communication is secured regardless of network location.
  * Access to resources is determined by dynamic policy (client identity, device posture, workload context).

### 1.2 [FinOps Foundation: FinOps Framework](https://www.finops.org/framework/)
* **Canonical Scope:** The operating model for cloud financial management.
* **The Three FinOps Phases:**
  1. **Inform:** Visibility, allocation, benchmarking, and unit economics (cost per transaction/customer).
  2. **Optimize:** Resource right-sizing, anomaly detection, purchasing commitments (Savings Plans, Spot).
  3. **Operate:** Continuous measurement, automated governance, aligning platform teams with finance.

### 1.3 [SPIFFE / SPIRE: Workload Identity Architecture](https://spiffe.io/docs/latest/spiffe-about/overview/)
* **Canonical Scope:** Secure Production Identity Framework for Everyone.
* **Architecture:**
  * `SPIFFE ID`: URI uniquely identifying a workload (e.g. `spiffe://example.com/ns/prod/sa/payment-service`).
  * `SVID` (SPIFFE Verifiable Identity Document): Short-lived X.509 certificate or JWT automatically rotated and issued by SPIRE agents to workloads for cryptographic mTLS authentication without passwords.

---

## 2. Policy-as-Code & Cloud Native Governance

### 2.1 [Kyverno & OPA Gatekeeper](https://kyverno.io/docs/)
* **Canonical Scope:** Enforcing cluster-wide Kubernetes admission policies.
* **Capabilities:** Validating, mutating, and generating Kubernetes resources (e.g. automatically injecting standard labels, enforcing non-root users, blocking public load balancers).

### 2.2 [Kubernetes Cost Allocation: OpenCost & Kubecost](https://www.opencost.io/docs/)
* Allocating multi-tenant Kubernetes cluster costs accurately down to specific Namespaces, Deployments, and Labels by correlating pod CPU/memory requests with cloud provider billing data.

---

## 3. Staff-Level Interview Angle & Architectural Reflexes

1. **"FinOps: Allocating Multi-Tenant Cluster Costs":**
   * *Problem:* A single \$50,000/month Kubernetes cluster runs 200 microservices across 15 engineering teams; nobody takes responsibility for compute waste.
   * *Staff Reflex:* Deploy OpenCost to measure allocated vs idle resources per namespace. Attribute idle cluster overhead proportionally to teams based on their requested capacity. Publish showback/chargeback dashboards to engineering leaders.
2. **Eliminating Long-Lived Cloud Credentials:**
   * *Staff Reflex:* Zero tolerance for static IAM user access keys. Enforce cloud provider workload identity federation (OIDC) across GitHub Actions, GitLab runners, and Kubernetes clusters.

---

## 4. Reading & Study Checklist

- [ ] Read *NIST SP 800-207: Zero Trust Architecture*
- [ ] Read *FinOps Foundation Framework Overview*
- [ ] Read *SPIFFE/SPIRE Architecture & Workload Attestation Guide*
- [ ] Study Kyverno vs OPA Gatekeeper policy syntax and generation capabilities
- [ ] Review OpenCost / Kubecost allocation algorithms for idle capacity
