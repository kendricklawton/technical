# Domain 6: Observability & SRE — Essential Technical Documentation & Guide

> **Core Focus:** Site Reliability Engineering (SRE), SLIs/SLOs/Error Budgets, Toil Reduction, OpenTelemetry (OTel) Architecture, Prometheus & Mimir, Distributed Tracing, High-Cardinality Logging, and Alert Fatigue Mitigation.

---

## 1. Authoritative SRE & Observability Doctrines

### 1.1 [Google Site Reliability Engineering (SRE) Book](https://sre.google/sre-book/table-of-contents/)
* **Canonical Scope:** The foundational canon of modern production systems reliability.
* **Core Principles:**
  * **Embracing Risk & Error Budgets:** $100\%$ availability is the wrong target for almost any system. Use Error Budgets ($100\% - \text{SLO}$) as a balancing mechanism between feature velocity and system stability. When budget is exhausted, halt feature releases and dedicate sprints to reliability.
  * **Service Level Objectives (SLOs):** Measurable targets for Service Level Indicators (SLIs). Formula:
    $$\text{SLI} = \frac{\text{Good Events}}{\text{Total Valid Events}} \times 100\%$$
  * **Toil Elimination:** Cap operational toil (manual, repetitive, automatable work devoid of enduring value) at $<50\%$ of an engineer's time; spend the remainder on software engineering.

### 1.2 [OpenTelemetry (OTel) Specification](https://opentelemetry.io/docs/specs/otel/)
* **Canonical Scope:** Unified vendor-neutral telemetry collection framework.
* **OTel Architecture:**
  * **API vs SDK:** API provides instrumentation hooks in application code; SDK provides concrete implementation, sampling, and routing.
  * **OpenTelemetry Collector:** Pipelines composed of `receivers` -> `processors` (batching, memory limiter, PII scrubbing) -> `exporters` (OTLP, Prometheus, Jaeger).
  * **Trace Context Propagation:** W3C `traceparent` headers (`00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01`).

### 1.3 [Prometheus Monitoring & Alerting Architecture](https://prometheus.io/docs/introduction/overview/)
* **Canonical Scope:** Pull-based multi-dimensional time-series data model.
* **Core Mechanisms:**
  * PromQL rate calculations: `rate()` vs `irate()` (use `rate()` for alerts and SLOs; `irate()` for instant graph spikes).
  * Alertmanager: Grouping, inhibition, and silencing to prevent alert flooding during upstream network/cluster failures.

---

## 2. Production Telemetry at Scale

* **High-Cardinality Metrics:** Storing user IDs or IP addresses as Prometheus labels leads to TSDB memory exhaustion. Move high-cardinality metadata into distributed tracing tags or structured logs.
* **Distributed Tracing Sampling:** Head-based sampling (probabilistic at start of trace) vs Tail-based sampling (sampling decided at collector after full trace completes, guaranteeing $100\%$ capture of errors and P99 latency anomalies).

---

## 3. Staff-Level Interview Angle & Architectural Reflexes

1. **"Burn Rate Alerts vs Static Threshold Alerts":**
   * *Problem:* Alerting on CPU $>80\%$ or error rate $>1\%$ causes massive false positives during low-traffic hours or brief spikes.
   * *Staff Reflex:* Implement **Multi-Window Multi-Burn-Rate Alerts** (Google SRE standard). Trigger high-priority pages only when the service is consuming its 30-day error budget at a rate that will exhaust it within 1 hour (14.4x burn rate) or 6 hours (6x burn rate).
2. **Centralized Observability Cost Control:**
   * Telemetry ingestion bills exploding.
   * *Resolution:* Deploy OTel collector edge processors to drop health check traces, scrub debug logs, aggregate metrics via recording rules, and enforce tail-based sampling.

---

## 4. Reading & Study Checklist

- [ ] Read *Google SRE Book (Chapters on Service Level Objectives and Eliminating Toil)*
- [ ] Read *OpenTelemetry Specification & Collector Architecture*
- [ ] Master Multi-Window Multi-Burn-Rate alerting formulas
- [ ] Study Tail-based sampling configuration in OTel Collector
- [ ] Review Prometheus TSDB storage engine and memory compaction mechanics
