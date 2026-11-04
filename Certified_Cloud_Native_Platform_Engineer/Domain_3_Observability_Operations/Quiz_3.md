# Domain 3: Observability & Operations — Quiz 3

**Exam:** Certified Cloud Native Platform Engineer (CNPE)  
**Format:** 10 Hands-On Scenario & Performance Questions with Detailed Rationales

---

### Question 1: Kubernetes Pod Eviction Alert PromQL
You want to detect if pods in the `production` namespace are being evicted due to node memory pressure or disk pressure. Which PromQL metric indicates the rate of pod evictions?
- A. `node_memory_MemTotal_bytes`
- B. `kube_deployment_status_replicas_unavailable`
- C. `container_cpu_usage_seconds_total`
- D. `kube_pod_status_phase{namespace="production", phase="Failed"}` and `kube_pod_status_reason{reason="Evicted"}`

---

### Question 2: OpenTelemetry Resource Attributes
Which semantic convention resource attribute is required to identify the logical service name across traces, metrics, and logs in OpenTelemetry?
- A. `cloud.zone`
- B. `service.name`
- C. `k8s.pod.tag`
- D. `application.title`

---

### Question 3: Prometheus Alertmanager Inhibition Loop Prevention
Why must Alertmanager inhibition rules never contain circular dependencies (e.g., Alert A inhibits Alert B, and Alert B inhibits Alert A)?
- A. Prometheus will crash on startup.
- B. Inhibit rules only work on HTTP routes.
- C. Alertmanager converts alerts to JSON automatically.
- D. Circular inhibition rules can create deadlocks or non-deterministic suppression states where neither alert fires.

---

### Question 4: OpenTelemetry Collector Load-Balancing Exporter
When deploying an OpenTelemetry Collector gateway tier behind an external load balancer, trace spans from the same distributed trace often get routed to different collector instances, breaking tail-based sampling decisions. How does the `loadbalancing` exporter resolve this?
- A. By saving spans to SQLite on the local node.
- B. By rejecting traces that have more than 5 spans.
- C. By hashing the `trace_id` and routing all spans belonging to the same trace to the identical downstream backend collector instance.
- D. By converting gRPC to WebSocket.

---

### Question 5: Prometheus Scrape Timeout Configuration
If Prometheus is configured with `scrape_interval: 15s` and `scrape_timeout: 30s`, what error will the Prometheus server log during target evaluation?
- A. "Port 9090 is occupied."
- B. "Scrape timeout cannot be greater than scrape interval."
- C. "TSDB corrupted."
- D. "Scrape timeout is valid."

---

### Question 6: SLO Error Budget Calculation
A service has an SLO of 99.95% availability over a 30-day window (43,200 total minutes). How many minutes of total downtime or error budget are permitted during this 30-day period?
- A. 120 minutes
- B. 43.2 minutes
- C. 4.32 minutes
- D. 21.6 minutes

---

### Question 7: Automated Canary Analysis Metric Provider
When configuring Argo Rollouts with Prometheus as the `metricProvider` in an `AnalysisTemplate`, which field defines the PromQL query to evaluate?
- A. `template.rules.promql`
- B. `provider.prometheus.address` and `provider.prometheus.query`
- C. `spec.metrics.sql`
- D. `eval.http.get`

---

### Question 8: Loki Promtail vs Fluentbit vs OTel Collector Logs
What is the primary architectural advantage of using the OpenTelemetry Collector for container log shipping compared to running standalone Promtail or Fluentbit?
- A. It consolidates tracing, metrics, and log pipelines into a single unified agent architecture, applying uniform resource attributes (`service.name`, `k8s.pod.name`) across all telemetry signals.
- B. OpenTelemetry collector does not require root privileges on Linux.
- C. It reduces log size by 99% using lossy compression.
- D. OpenTelemetry collector only supports Windows.

---

### Question 9: Prometheus Remote Write WAL (Write-Ahead Log)
What role does the local Write-Ahead Log (WAL) play when Prometheus Remote Write is forwarding metrics to an external storage cluster (such as Thanos Receive or Cortex)?
- A. It buffers samples locally on disk so that transient network interruptions to the remote endpoint do not cause data loss.
- B. It converts metrics into Parquet files.
- C. It generates grafana dashboards automatically.
- D. It encrypts payloads with GPG.

---

### Question 10: SLO Burn Rate Multi-Window Alerting Benefit
Why does Google SRE recommend pairing a long window (e.g., 1 hour) with a short window (e.g., 5 minutes) when configuring burn-rate alerts?
- A. To save disk space on Prometheus servers.
- B. To prevent Slack webhooks from timing out.
- C. To avoid sending false-positive alerts when an error spike ends immediately (short-window reset) while still alerting promptly when a major failure occurs (long-window sustained burn).
- D. Because Alertmanager only supports 2 windows.

---

## Answer Key & Rationales

1. **D** — `kube_pod_status_phase{phase="Failed"}` combined with `kube_pod_status_reason{reason="Evicted"}` from `kube-state-metrics` tracks evicted pods.
2. **B** — In the OpenTelemetry semantic conventions standard, `service.name` is the core attribute identifying the logical service emitting the telemetry.
3. **D** — Circular inhibition rules create indeterminate dependency graphs where Alertmanager cannot evaluate whether to silence or deliver notifications.
4. **C** — The OpenTelemetry `loadbalancing` exporter hashes incoming trace IDs to ensure that all spans sharing the same `trace_id` are consistently routed to the same backend collector for stateful tail-sampling.
5. **B** — Prometheus strictly enforces that `scrape_timeout` must be less than or equal to `scrape_interval`.
6. **D** — $43,200 \times (1 - 0.9995) = 43,200 \times 0.0005 = 21.6 \text{ minutes}$.
7. **B** — In an Argo Rollouts `AnalysisTemplate`, the Prometheus provider block specifies `address` (Prometheus URL) and `query` (PromQL statement).
8. **A** — OTel Collector unifies logs, metrics, and traces under a single pipeline and semantic data model, correlating logs with traces using identical resource attributes.
9. **A** — The Prometheus remote write subsystem reads from the local TSDB WAL, providing resilient disk-backed buffering during remote storage outages.
10. **C** — Multi-window alerting ensures that a brief spike that has already cleared does not wake up an on-call engineer (since the short window is no longer burning), preventing unnecessary pages.
