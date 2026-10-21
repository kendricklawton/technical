# Domain 1: Platform Foundations & IDP — Essential White Papers & Technical Reference Guide

> **Core Focus:** Platform Product Management, Internal Developer Platforms (IDPs), Team Topologies, Cognitive Load Reduction, Golden Paths, Backstage Architecture, and Developer Experience (DevEx).

---

## 1. Authoritative Industry White Papers & Frameworks

### 1.1 [CNCF TAG App Delivery: Platforms White Paper](https://tag-app-delivery.cncf.io/whitepapers/platforms/)
* **Canonical Scope:** Definitive industry blueprint defining what cloud-native platforms are, how they are constructed, and how they deliver value.
* **Core Principles:**
  * **Platform as a Product:** Treating developers as customers, establishing feedback loops, conducting user research, and measuring adoption rather than enforcing mandates.
  * **Thinnest Viable Platform (TVP):** Delivering the smallest possible layer of abstraction that solves immediate pain points without over-engineering speculative infrastructure.
  * **Self-Service Capabilities:** Moving from ticket-based ops to API-driven, declarative self-service workflows with built-in guardrails.

### 1.2 [CNCF TAG App Delivery: Platform Engineering Maturity Model](https://tag-app-delivery.cncf.io/whitepapers/platform-eng-maturity-model/)
* **Canonical Scope:** Assessment framework across 5 core aspects: Investment, Adoption, Interfaces, Operations, and Measurement across 4 maturity levels (Provisional, Operational, Scalable, Optimized).
* **Maturity Highlights:**
  * *Interfaces:* Moving from ad-hoc scripts -> documented templates -> unified developer portals (Backstage) -> contextual AI/declarative intent.
  * *Operations:* Transitioning from heroic manual intervention -> scheduled automation -> GitOps & self-healing -> autonomic platform policies.

### 1.3 [Team Topologies (Skelton & Pais) Framework](https://teamtopologies.com/)
* **Canonical Scope:** Organizational design optimizing team interactions for fast flow and minimal cognitive load.
* **The 4 Fundamental Team Types:**
  1. **Stream-Aligned Team:** Aligned to a single flow of business work; responsible for end-to-end delivery.
  2. **Platform Team:** Provides underlying domain-agnostic capabilities to enable stream-aligned teams to deliver without friction.
  3. **Enabling Team:** Specialists who cross-skill and uplevel stream-aligned teams on new tools or practices.
  4. **Complicated-Subsystem Team:** Rare specialists responsible for deeply specialized mathematical, algorithmic, or networking systems.
* **The 3 Interaction Modes:** Collaboration, X-as-a-Service, Facilitating.

### 1.4 [DORA State of DevOps & SPACE Framework](https://dora.dev/)
* **Canonical Scope:** Industry benchmarks for measuring software delivery velocity and developer productivity.
* **DORA Four Keys:** Deployment Frequency, Lead Time for Changes, Change Failure Rate, Time to Restore Service (plus Operational Performance).
* **SPACE Framework:** Satisfaction & Well-being, Performance, Activity, Communication & Collaboration, Efficiency & Flow.

---

## 2. Technical Documentation & Architectures to Master

### 2.1 [Backstage.io Architecture & Core Plugins](https://backstage.io/docs/overview/what-is-backstage)
* **Architecture:** Node.js/React frontend catalog aggregating metadata from Git repositories via `catalog-info.yaml`.
* **Core Components:**
  * **Software Catalog:** Central source of truth for microservice ownership, system dependencies, and API specs.
  * **Software Templates (Scaffolder):** Golden path repository bootstrapping using cookiecutter/nunjucks templates with pre-configured CI/CD, linting, and security scanners.
  * **TechDocs:** Markdown documentation co-located with code, rendered centrally via MkDocs.

---

## 3. Staff-Level Interview Angle & Architectural Reflexes

1. **"Platform Mandate vs Organic Adoption":**
   * *Trap:* Forcing developers to use the platform by leadership decree.
   * *Staff Reflex:* Platform mandates create shadow IT and resentment. Drive adoption by creating "Golden Paths" that are genuinely easier, faster, and more reliable than manual alternatives. Measure Net Promoter Score (NPS) and self-service conversion rates.
2. **Cognitive Load Optimization:**
   * Distinguish between *extraneous cognitive load* (fighting Docker, IAM syntax, YAML boilerplate) and *intrinsic/germane cognitive load* (solving business domain logic). The platform's sole purpose is to ruthlessly eliminate extraneous load.

---

## 4. Reading & Study Checklist

- [ ] Read *CNCF Platforms White Paper*
- [ ] Read *CNCF Platform Engineering Maturity Model*
- [ ] Review *Team Topologies* summary and interaction modes
- [ ] Study Backstage.io architecture and `catalog-info.yaml` spec
- [ ] Master DORA metrics calculation and SPACE productivity dimensions
