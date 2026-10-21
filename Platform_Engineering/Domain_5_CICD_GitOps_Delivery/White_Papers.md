# Domain 5: CI/CD, GitOps & Delivery — Essential Technical Documentation & Guide

> **Core Focus:** Enterprise CI/CD at Scale, Distributed Build Caching, Runner Fleet Autoscaling, GitOps Principles (OpenGitOps, Argo CD, Flux v2), Progressive Delivery (Argo Rollouts), and Supply Chain Security (SLSA, Sigstore).

---

## 1. Authoritative Frameworks & Documentation

### 1.1 [OpenGitOps Principles & CNCF GitOps Standards](https://opengitops.dev/)
* **Canonical Scope:** The four foundational principles of GitOps:
  1. **Declarative:** A system's desired state must be expressed declaratively.
  2. **Versioned and Immutable:** Desired state is stored in a way that enforces immutability and version history (Git).
  3. **Pulled Automatically:** Software agents automatically pull desired state from the source.
  4. **Continuously Reconciled:** Software agents continuously observe actual system state and reconcile it back to desired state.

### 1.2 [Argo CD & Flux v2 Enterprise Architecture](https://argo-cd.readthedocs.io/en/stable/)
* **Architecture Patterns:**
  * **App-of-Apps Pattern:** A master Argo CD Application managing child Applications to orchestrate cluster bootstrap.
  * **ApplicationSets:** Multi-cluster and multi-tenant generator automatically discovering Git branches, folders, or cluster lists to generate application instances.
  * **Sync Waves & Resource Hooks:** Ordering resource deployments (`PreSync`, `Sync`, `PostSync`) with configurable waves (e.g., CRDs at wave 0, DB schema migration at wave 1, Deployment at wave 2).

### 1.3 [Progressive Delivery with Argo Rollouts](https://argoproj.github.io/argo-rollouts/)
* **Canonical Scope:** Advanced deployment controllers replacing standard Kubernetes Deployments.
* **Capabilities:**
  * Blue/Green deployments with preview service verification and manual/automated promotion.
  * Canary deployments with fine-grained traffic routing (ALB, Istio, Linkerd, Nginx).
  * Automated metric analysis (`AnalysisTemplate`) querying Prometheus, Datadog, or CloudWatch; aborting rollout if error rate $> 1\%$.

### 1.4 [Supply Chain Levels for Software Artifacts (SLSA)](https://slsa.dev/)
* **Canonical Scope:** Security framework against tampering, source compromise, and build contamination.
* **SLSA Levels (1-3):** Build service generation of non-falsifiable provenance, hermetic build environments, and cryptographically verified pipelines.

---

## 2. High-Performance Build Pipeline Engineering

* **Distributed Caching:** Optimizing Docker build times using BuildKit (`--cache-to=type=registry`, `--cache-from=type=registry`) and Bazel/Turbo cache stores.
* **Self-Hosted Runner Fleets:** Operating ephemeral Kubernetes-based runner fleets using Actions Runner Controller (ARC) autoscaling based on GitHub webhook backlogs.

---

## 3. Staff-Level Interview Angle & Architectural Reflexes

1. **"Monorepo vs Polyrepo Delivery Pipelines":**
   * *Staff Perspective:* Monorepos simplify atomic cross-service refactors and shared libraries, but require sophisticated path-filtering triggers and distributed build caching to avoid quadratic CI times. Polyrepos provide isolated ownership and blast radiuses, but complicate dependency upgrades across fleets.
2. **GitOps Drift Remediation Conflicts:**
   * Production issue requires an emergency hotfix via `kubectl edit`.
   * *Resolution:* Argo CD automated sync will immediately overwrite manual changes. Teams must either update Git directly (the proper GitOps path) or temporarily toggle `auto-sync: false` while resolving the Sev-1 incident, followed by codifying the fix into Git.

---

## 4. Reading & Study Checklist

- [ ] Read *OpenGitOps Principles*
- [ ] Read *Argo CD Architecture & ApplicationSet Guide*
- [ ] Study Argo Rollouts Canary AnalysisTemplate with Prometheus metrics
- [ ] Review SLSA Framework requirements and Sigstore signing pipelines
- [ ] Master Actions Runner Controller (ARC) ephemeral runner architecture
