# Domain 3: Observability & Operations — Authoritative White Papers & CNCF Standards

> **CNPE Exam Weight:** 20% of Certified Cloud Native Platform Engineer

---

## 1. Authoritative References & Standards

- **[OpenTelemetry Specification (CNCF)](https://opentelemetry.io/docs/specs/):** OTLP protocol (gRPC port 4317, HTTP port 4318), semantic conventions, resource attributes, and W3C Trace Context propagation.
- **[Prometheus Operator Architecture](https://prometheus-operator.dev/):** Declarative monitoring via CRDs (`ServiceMonitor`, `PodMonitor`, `PrometheusRule`, `Probe`, `AlertmanagerConfig`).
- **[Google SRE Book — Service Level Objectives](https://sre.google/sre-book/service-level-objectives/):** SLI, SLO, SLA, Multi-Window Multi-Burn-Rate alerting strategies.
- **[Grafana As Code & Declarative Dashboard Provisioning](https://grafana.com/docs/grafana/latest/administration/provisioning/):** ConfigMaps, sidecar synchronization, and GitOps-managed dashboards.
- **[Continuous Profiling with Pyroscope / Parca (CNCF)](https://www.cncf.io/):** CPU and memory flame graphs in production environments with eBPF.

---

## 2. Staff SRE Architecture Patterns

### 2.1 Multi-Window Multi-Burn-Rate Alerting
Traditional threshold alerting (e.g., "Error rate > 1% for 5m") generates either false alerts during brief spikes or alert fatigue during minor slow burns. SRE best practice uses burn-rate alerts based on error budget consumption:

$$\text{Burn Rate} = \frac{\text{Observed Error Rate}}{\text{Allowed Budget Error Rate}}$$

For a 99.9% SLO ($0.1\%$ error budget over 30 days / 720 hours):
1. **$14.4\times$ Burn Rate (2% budget consumed in 1 hour):** Triggers PagerDuty immediate page (1h and 5m short-window confirmation).
2. **$6\times$ Burn Rate (5% budget consumed in 6 hours):** Triggers PagerDuty page within 6 hours.
3. **$1\times$ Burn Rate (100% budget consumed in 30 days):** Ticket / low-urgency alert.

### 2.2 OpenTelemetry Collector Pipeline Optimization
When deploying OTel Collector at platform scale (100,000+ spans/sec):
- **Memory Limiter Processor MUST BE FIRST:** Always place `memory_limiter` at the very beginning of the `processors` chain to enforce hard heap ceiling limits and prevent kernel OOM-killer termination.
- **Batch Processor MUST BE SECOND:** Group data across time (`timeout: 5s`) and size (`send_batch_size: 8192`) before calling expensive transformations or network exporters.
- **OTTL (OpenTelemetry Transformation Language) Filtering:** Scrub sensitive user data (PII, tokens, authorization headers) at the collector edge before exporting to cloud vendors:
  ```yaml
  transform:
    error_mode: ignore
    trace_statements:
      - context: span
        statements:
          - delete_key(attributes, "http.request.header.authorization")
          - set(attributes["user.id"], "REDACTED") where attributes["user.id"] != nil
  ```

### 2.3 Span Metrics Generation
Instead of dual-instrumenting applications for metrics and traces, platform engineers deploy the OTel Collector `spanmetrics` connector. The connector parses incoming trace spans and automatically emits Prometheus RED metrics (Rate, Errors, Duration) aggregated by service, operation, and HTTP status code.

---

## 3. Production Failure Modes & Anti-Patterns

| Anti-Pattern | Operational Risk | Production Resolution |
| :--- | :--- | :--- |
| **High Cardinality Metrics Explosion** | Putting dynamic IDs (`user_id`, `uuid`, `order_id`) into Prometheus metric labels causes TSDB memory exhaustion and query timeouts. | Enforce metric relabeling rules in `ServiceMonitor` to drop high-cardinality labels or route them exclusively to distributed tracing spans. |
| **Collector OOM without `memory_limiter`** | Bursts of application traffic saturate collector pod memory, leading to pod restarts and telemetry blackout. | Configure `memory_limiter` with `check_interval: 1s`, `limit_percentage: 80`, and `spike_limit_percentage: 20`. |
| **Unrouted Alertmanager Traps** | Default Alertmanager configurations dropping alerts or routing production SEV-1 alerts to silent Slack channels. | Define explicit `route` trees with `matchers`, fallback routes, and `inhibit_rules` so node outages silence container alerts. |
| **Scrape Storms on Control Plane** | Multiple Prometheus replicas independently scraping thousands of endpoints at identical intervals. | Stagger `scrapeInterval` and utilize Prometheus Agent mode forwarding to remote storage (Thanos/Cortex/Mimir). |
