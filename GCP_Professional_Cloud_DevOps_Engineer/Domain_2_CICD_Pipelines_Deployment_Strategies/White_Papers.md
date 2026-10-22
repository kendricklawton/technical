# Domain 2: Building & Implementing CI/CD Pipelines — Authoritative White Papers & SRE Standards

> **Exam Focus:** Continuous Delivery Best Practices, Progressive Rollouts, and Secure Supply Chains.

---

## 1. Authoritative References & Standards

- **[Google Cloud Deploy Architecture & Delivery Pipelines](https://cloud.google.com/deploy/docs):** Target declarations, releases, rollouts, canary strategy, and verification jobs.
- **[Google SRE Book — Release Engineering](https://sre.google/sre-book/release-engineering/):** Self-service releases, hermetic builds, continuous deployment, and enforcement of policies.
- **[DORA (DevOps Research and Assessment) Core Capabilities](https://cloud.google.com/devops):** The 4 Key Metrics (Deployment Frequency, Lead Time for Changes, Change Failure Rate, Time to Restore Service).
- **[Binary Authorization on GKE Architecture](https://cloud.google.com/binary-authorization/docs):** Attestors, KMS asymmetric keys, and deploy-time policy enforcement.

---

## 2. Staff SRE Architectural Patterns

### 2.1 The DORA Core Metrics
High-performing software delivery teams optimize for four research-backed metrics:
1. **Deployment Frequency:** How often an organization successfully releases to production (elite: multiple deploys per day).
2. **Lead Time for Changes:** The amount of time it takes a commit to get into production (elite: less than one hour).
3. **Change Failure Rate:** The percentage of deployments causing a failure in production (elite: 0–15%).
4. **Time to Restore Service (MTTR):** How long it takes an organization to recover from a failure in production (elite: less than one hour).

### 2.2 Immutable Release Snapshots
Never build once for development and rebuild from source for production. A container image must be built and tested **once** in CI, cryptographically signed, and promoted across dev $\to$ staging $\to$ prod without rebuilding code. Only runtime environment variables (ConfigMaps/Secrets) should differ between environments.

---

## 3. Production Failure Modes & Anti-Patterns

| Anti-Pattern | Operational Risk | Production Resolution |
| :--- | :--- | :--- |
| **Using Mutable `:latest` Container Tags** | Deploying `:latest` makes rollbacks non-deterministic and prevents verifying what code is currently running in production. | Strictly pin container images by their immutable cryptographic digest: `image@sha256:4b2...` or immutable commit SHA tag. |
| **Manual GKE Deployment via `kubectl apply` from Laptops** | Uncontrolled configuration drift, lack of audit trails, and security credential sprawl on developer machines. | Restrict direct developer `cluster-admin` access. Force all deployments through Google Cloud Deploy pipelines or GitOps Config Sync. |
| **Big-Bang Cutover Deployments** | Switching 100% of user traffic to an unverified release causes massive customer-facing outages if an edge case triggers. | Enforce automated Canary rollouts with 10% traffic evaluation and automated health check rollback gates. |
