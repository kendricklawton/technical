# Domain 3: Observability & Operations — Quiz 1

**Exam:** Certified Cloud Native Platform Engineer (CNPE)  
**Format:** 10 Hands-On Scenario & Performance Questions with Detailed Rationales

---

### Question 1: OTel Collector Memory Protection
An OpenTelemetry Collector deployment pod is repeatedly getting killed by the Linux kernel Out-Of-Memory (OOM) killer under sudden traffic spikes. Which processor configuration is required at the start of the processing chain to gracefully shed load or drop data?
- A. `batch` with `timeout: 10s`
- B. `memory_limiter` configured with `check_interval`, `limit_percentage`, and `spike_limit_percentage`
- C. `filter` processor dropping all INFO level logs
- D. `tail_sampling` with `decision_wait: 30s`

---

### Question 2: Prometheus Operator Scrape Discovery
A platform engineer created a new `ServiceMonitor` in the `payments` namespace, but Prometheus is not discovering or scraping the target endpoints. The Prometheus instance is configured with `serviceMonitorSelector: { matchLabels: { release: kube-prometheus-stack } }`. What is the most likely cause?
- A. The `ServiceMonitor` resource is missing the label `release: kube-prometheus-stack`.
- B. The `ServiceMonitor` must be in the `kube-system` namespace.
- C. Prometheus requires an imperative reboot to discover new CRDs.
- D. `ServiceMonitor` only works if the pod is running with host networking.

---

### Question 3: Multi-Window Multi-Burn-Rate Alerting
Under a 99.9% SLO over a 30-day window, a service has an error budget of 0.1%. An SRE alert triggers when a $14.4\times$ burn rate is sustained over a 1-hour window. What percentage of the total 30-day error budget is consumed during this 1-hour window?
- A. 0.1%
- B. 2.0%
- C. 14.4%
- D. 50%

---

### Question 4: OpenTelemetry Auto-Instrumentation
When using the OpenTelemetry Operator to auto-instrument a Node.js microservice without modifying the source code or Dockerfile, which mechanism does the operator use?
- A. Recompiling the binary inside a custom mutating admission webhook.
- B. Injecting an init container that copies the OpenTelemetry SDK files into a shared volume, and injecting `NODE_OPTIONS` environment variables into the main container.
- C. Rewriting the container image tag in the deployment spec.
- D. Running an eBPF tracepoint that dynamically edits the V8 bytecode at runtime.

---

### Question 5: Alertmanager Inhibition Rules
A major data center network switch fails, causing all 20 worker nodes in a cluster to become `NotReady`. SREs receive 400 separate alerts for container crashes, pod evictions, and service connection failures, in addition to the 20 node failure alerts. How should Alertmanager be configured to suppress container-level alerts when a node is down?
- A. By disabling Alertmanager during outages.
- B. By configuring an `inhibit_rule` where `source_match` is `alertname: NodeDown` and `target_match` is `severity: critical`, matching on the common `node` label.
- C. By configuring `group_wait: 2h`.
- D. By configuring Slack rate limiting.

---

### Question 6: OTTL (OpenTelemetry Transformation Language)
A security audit reveals that client authorization tokens (`Authorization: Bearer <token>`) are accidentally being logged in span attributes. Which OTel Collector processor and syntax should be used to scrub this key from traces?
- A. `transform` processor with statement `delete_key(attributes, "http.request.header.authorization") where attributes["http.request.header.authorization"] != nil`
- B. `batch` processor with `scrub: true`
- C. `memory_limiter` processor with `filter_auth: enable`
- D. `otlp` receiver with `sanitize: true`

---

### Question 7: High Cardinality TSDB Impact
A developer configures a custom Prometheus metric `checkout_latency_seconds_bucket` with labels `user_id`, `shopping_cart_id`, and `credit_card_bin`. What is the direct operational impact on the Prometheus monitoring platform?
- A. Prometheus automatically deletes old time series to maintain low memory.
- B. The Prometheus TSDB head block will experience a massive cardinality explosion, leading to exponential RAM growth, OOM kills, and unresponsive queries.
- C. Metrics are compressed linearly without any memory impact.
- D. Prometheus routes high-cardinality metrics to etcd.

---

### Question 8: Tail-Based vs Head-Based Sampling
Why is tail-based sampling preferred over head-based sampling for high-throughput distributed tracing architectures?
- A. Head-based sampling makes sampling decisions at span completion when the error status and overall latency are known.
- B. Tail-based sampling buffers the entire trace before making the sampling decision, ensuring 100% of error traces and high-latency outlier traces are retained while sampling down mundane 200 OK spans.
- C. Tail-based sampling requires no memory buffer in the collector.
- D. Head-based sampling captures 100% of traces without overhead.

---

### Question 9: Grafana Sidecar Dashboards
In a GitOps-managed Kubernetes cluster, how does the Grafana sidecar automatically import new dashboards when a developer commits a dashboard JSON file?
- A. The sidecar runs `curl -X POST /api/dashboards` on a scheduled cron job.
- B. The sidecar watches Kubernetes `ConfigMaps` with a specific label (e.g., `grafana_dashboard: "1"`) and dynamically syncs their contents into Grafana's `/tmp/dashboards` directory.
- C. Grafana pulls directly from GitHub using Git SSH keys.
- D. The developer must manually restart the Grafana deployment pod.

---

### Question 10: Spanmetrics Connector Mechanics
How does the OpenTelemetry Collector `spanmetrics` connector improve observability efficiency?
- A. It converts Prometheus metrics into Jaeger trace spans.
- B. It derives RED metrics (request count, error rate, duration histograms) directly from trace spans within the collector, eliminating the need to duplicate telemetry instrumentation in application code.
- C. It compresses JSON trace payloads into gzip archives.
- D. It acts as an external load balancer for gRPC traffic.

---

## Answer Key & Rationales

1. **B** — The `memory_limiter` processor checks memory usage periodically and drops spans/metrics or back-pressures upstream callers when memory usage exceeds configured safety ceilings.
2. **A** — Prometheus Operator uses label selectors (`serviceMonitorSelector`) on the `Prometheus` custom resource to decide which `ServiceMonitor` CRDs to scrape.
3. **B** — A $14.4\times$ burn rate consumes $14.4 \times (1 / 720) \approx 2.0\%$ of the 30-day budget in 1 hour ($720\text{ hours} \text{ in a 30-day month}$).
4. **B** — The OTel Operator injects an init container containing the pre-packaged agent/SDK and sets environment variables (such as `NODE_OPTIONS` or `JAVA_TOOL_OPTIONS`) to attach the SDK during process runtime.
5. **B** — Alertmanager `inhibit_rules` silence a set of target alerts if a matching source alert is active and both share identical specified labels (e.g., `node`).
6. **A** — OTTL statements inside the `transform` processor allow programmatic inspection, redaction, or removal of attributes in trace spans, metrics, and logs.
7. **B** — Including unique identifiers (`user_id`, `cart_id`) in Prometheus metric labels creates millions of unique time series (cardinality explosion), crashing the TSDB.
8. **B** — Head-based sampling decides whether to sample at the start of a trace before knowing whether the trace will fail or succeed; tail-based sampling inspects completed spans and captures all errors.
9. **B** — The `kiwigrid/k8s-sidecar` container runs alongside Grafana, watching `ConfigMaps` matching a label filter and updating local dashboard disk volumes.
10. **B** — The `spanmetrics` connector intercepts trace spans in the collector pipeline and computes RED metrics with latency histograms, decoupling metric generation from application SDK code.
