# Domain 1: Applying SRE Principles to a Service — Authoritative White Papers & SRE Standards

> **Exam Focus:** Google SRE Book, SRE Workbook, and Enterprise Case Studies.

---

## 1. Authoritative References & Standards

- **[Google SRE Book — Service Level Objectives](https://sre.google/sre-book/service-level-objectives/):** The canonical principles for formulating SLIs, SLOs, and Error Budgets.
- **[Google SRE Book — Eliminating Toil](https://sre.google/sre-book/eliminating-toil/):** Definition, taxonomy, measurement, and the 50% cap on operational overhead.
- **[Google SRE Workbook — Implementing SLOs](https://sre.google/workbook/implementing-slos/):** Practical implementations, Cloud Monitoring service-oriented architecture, and multi-window burn rate math.
- **[Google Cloud Architecture Framework — Reliability](https://cloud.google.com/architecture/framework/reliability):** Designing high-availability topologies across zones and regions.

---

## 2. Staff SRE Architectural Patterns

### 2.1 The Four Golden Signals
Google SRE categorizes user-facing service health into four fundamental metrics:
1. **Latency:** The time it takes to service a request. Important: Distinguish between successful request latency and failed request latency (e.g. fast HTTP 500s).
2. **Traffic:** A measure of how much demand is being placed on your system (e.g. HTTP requests per second, I/O network bandwidth).
3. **Errors:** The rate of requests that fail, either explicitly (HTTP 500s), implicitly (wrong content returned), or by policy (HTTP 403 authorization failures).
4. **Saturation:** How "full" your service is, measuring the most constrained system resource (CPU, memory, database connection pool, disk I/O bandwidth).

### 2.2 Error Budget Policies in Practice
An error budget policy is ineffective if leadership does not back it up. In high-performing engineering organizations:
- The error budget policy is signed off by both the VP of Product and VP of Engineering.
- When an error budget is depleted, the product roadmap is explicitly put on hold. SREs and software developers swarm to fix technical debt, flaky integration tests, and infrastructure bottlenecks.

---

## 3. Production Failure Modes & Anti-Patterns

| Anti-Pattern | Operational Risk | Production Resolution |
| :--- | :--- | :--- |
| **Alerting on CPU Utilization instead of User SLI** | Paging engineers because CPU reached 85% when users experience fast 30ms latency causes severe alert fatigue and burnout. | Alert exclusively on user-impacting symptoms (SLI degradation, latency P99, error rate) and burn rates, using resource utilization purely for diagnostic dashboards. |
| **Setting 100% Availability SLO** | Demanding 100% uptime halts all software innovation and creates an impossible engineering standard. | Set SLOs based on what users actually perceive. No external user benefits from $100\%$ uptime if their local ISP or mobile carrier operates at $99.5\%$. |
| **Manual Ticket-Driven Operations** | SRE team acts as manual gatekeepers approving VM creations or firewall rules, leading to toil explosion. | Enforce a strict 50% toil cap. Automate repetitive requests into self-service APIs and GitOps declarative pipelines. |
