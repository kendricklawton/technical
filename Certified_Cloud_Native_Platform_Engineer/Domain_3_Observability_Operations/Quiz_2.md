# Domain 3: Observability & Operations — Quiz 2

**Exam:** Certified Cloud Native Platform Engineer (CNPE)  
**Format:** 10 Hands-On Scenario & Performance Questions with Detailed Rationales

---

### Question 1: PrometheusRule Metric Alert Expression
You need to write a PromQL alerting rule expression that fires when the 99th percentile request duration for `service="checkout"` exceeds 750 milliseconds over a 5-minute rate window. Which PromQL expression is correct?
- A. `histogram_quantile(0.99, sum by (le) (rate(http_request_duration_seconds_bucket{service="checkout"}[5m]))) > 0.75`
- B. `avg(http_request_duration_seconds_bucket[5m]) > 0.75`
- C. `rate(http_request_duration_seconds_count[5m]) > 750`
- D. `quantile(0.99, http_request_duration_seconds{service="checkout"}) > 750`

---

### Question 2: OpenTelemetry OTLP Port Standards
Which network ports are designated by the OpenTelemetry standard for receiving OTLP data over gRPC and HTTP, respectively?
- A. gRPC on port 4317, HTTP on port 4318
- B. gRPC on port 14250, HTTP on port 14268
- C. gRPC on port 9090, HTTP on port 9091
- D. gRPC on port 8080, HTTP on port 80

---

### Question 3: Prometheus Operator Probe Custom Resource
To perform blackbox synthetic probing (HTTP status code checks and TLS certificate expiration checks) of external partner APIs from within a Kubernetes cluster, which Prometheus Operator CRD is designed for this task?
- A. `PrometheusRule`
- B. `Probe` (leveraging Blackbox Exporter)
- C. `PodMonitor`
- D. `ServiceMonitor`

---

### Question 4: Alertmanager Repeat Interval Tuning
A non-critical informational alert (`DiskUsageWarning`) fires and sends an email. However, the engineering team receives a duplicate email every 5 minutes until the issue is resolved 3 days later, causing severe alert fatigue. Which Alertmanager parameter must be increased?
- A. `group_interval`
- B. `scrape_interval`
- C. `group_wait`
- D. `repeat_interval`

---

### Question 5: W3C Trace Context Propagation
When an auto-instrumented frontend service calls an auto-instrumented backend service over HTTP, which standard HTTP headers propagate the trace identifier and parent span identifier?
- A. `X-Correlation-ID` and `X-Span-ID`
- B. `Authorization` and `Cookie`
- C. `traceparent` and `tracestate`
- D. `X-Request-ID` and `X-B3-TraceId`

---

### Question 6: Relabeling vs Metric Relabeling
In Prometheus Operator configuration, what is the architectural difference between `relabelings` and `metricRelabelings` on a `ServiceMonitor`?
- A. `metricRelabelings` modifies DNS records; `relabelings` changes HTTP ports.
- B. There is no difference; they are aliases.
- C. `relabelings` is for logs; `metricRelabelings` is for traces.
- D. `relabelings` applies to scrape target discovery before the scrape occurs; `metricRelabelings` applies to individual scraped time series after the scrape but before storage in the TSDB.

---

### Question 7: Thanos Sidecar vs Prometheus Remote Write
When scaling Prometheus for long-term historical retention (1+ years) in multi-cluster environments, what is the primary operational trade-off between the Thanos Sidecar architecture and Prometheus Remote Write?
- A. Thanos Sidecar does not support PromQL.
- B. Remote Write can only target local Docker daemons.
- C. Remote Write stores data only on local SSDs.
- D. Thanos Sidecar uploads 2-hour TSDB blocks directly to S3/GCS object storage and queries them via Thanos Store Gateway, minimizing local memory; Remote Write streams samples point-by-point in real time, increasing Prometheus CPU/network overhead.

---

### Question 8: Pyroscope Continuous eBPF Profiling
A platform engineer notices periodic CPU spikes across a Java microservice cluster, but CPU metrics and traces only indicate high overall utilization without showing which function or line of code is consuming CPU. Which continuous profiling technology captures stack traces across the kernel and user space without recompiling code?
- A. Prometheus Node Exporter
- B. eBPF continuous profiling agent (Pyroscope / Parca)
- C. OpenTelemetry HTTP logger
- D. Fluentbit JSON parser

---

### Question 9: Multi-Cluster Alertmanager Clustering
To ensure high availability for Alertmanager without duplicate notifications being sent to PagerDuty or Slack, how must multiple Alertmanager replicas communicate?
- A. By sharing a ReadWriteMany NFS volume.
- B. Through an external PostgreSQL database.
- C. Using the Gossip protocol (Weave Mesh / HashiCorp Memberlist) over TCP/UDP port 9094 to exchange notification state and silence data.
- D. Through an active-passive DNS failover record.

---

### Question 10: OTel Collector Batch Processor Timeout
An OTel Collector is configured with `send_batch_size: 10000` and `timeout: 10s`. Under low traffic (only 50 spans per minute), what happens to incoming trace spans?
- A. Spans are dropped because the batch size was not reached.
- B. Spans are held in memory for up to 10 seconds, after which the batch processor flushes the available spans to the exporter.
- C. Spans bypass the exporter and write directly to disk.
- D. Spans trigger an immediate collector crash.

---

## Answer Key & Rationales

1. **A** — `histogram_quantile(0.99, sum by (le) (rate(...[5m])))` calculates the 99th percentile from a Prometheus histogram bucket series. `0.75` corresponds to 750 milliseconds.
2. **A** — The official standard OTLP ports are `4317` for gRPC and `4318` for HTTP.
3. **B** — The `Probe` CRD in Prometheus Operator instructs Prometheus to scrape an instance of Blackbox Exporter to probe external targets via ICMP, TCP, or HTTP.
4. **D** — `repeat_interval` specifies how long Alertmanager waits before re-sending an already active, unresolved alert notification to receivers.
5. **C** — W3C Trace Context specifies the `traceparent` (version, trace ID, parent ID, trace flags) and `tracestate` headers for vendor-neutral distributed context propagation.
6. **D** — `relabelings` filters and rewrites target metadata before Prometheus scrapes the endpoint; `metricRelabelings` inspects and modifies or drops the scraped metric samples before ingestion into the TSDB.
7. **D** — Thanos Sidecar reads immutable TSDB blocks every 2 hours and ships them to cheap object storage (S3/GCS), whereas Remote Write streams live metrics point-by-point to an ingestion API (Mimir/Cortex).
8. **B** — eBPF profilers (such as Pyroscope or Parca) sample kernel and user-space call stacks at low overhead (1-2% CPU) without application modification, generating actionable flame graphs.
9. **C** — Alertmanager instances form a mesh cluster using the Memberlist gossip protocol on port 9094 to deduplicate notifications and synchronize silence records.
10. **B** — The `batch` processor flushes either when `send_batch_size` is reached or when `timeout` expires, whichever happens first.
