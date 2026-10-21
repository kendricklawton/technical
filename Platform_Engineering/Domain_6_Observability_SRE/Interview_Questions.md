# PE Domain 6: Observability & SRE — 25 Core Interview Questions

Master these questions across Technical Screening, Live Troubleshooting (Mock 2), and Staff Architecture rounds.

---

## Tier 1: Core / Screening Questions (Questions 1–10)

### Q1: What are the "Three Pillars of Observability", and what are their respective limitations?
- **Short Answer:** Metrics, Logs, and Traces.
- **Deep Dive:**
  - *Metrics:* Aggregatable numeric time series. Pros: ultra-fast, cheap to store, ideal for alerting. Cons: lack fine-grained context; cannot isolate individual user request paths.
  - *Logs:* Detailed timestamped event records. Pros: rich debugging context. Cons: expensive to ingest, store, and index; high volume leads to noise.
  - *Traces:* Request lifecycle graphs across distributed systems. Pros: pinpoint exact latency bottlenecks and downstream call hierarchies. Cons: complex instrumentation; high data volume requires sampling.
- **Trade-offs & Realities:** Modern observability rejects treating them as separate silos; they must be correlated via shared identifiers (`trace_id`, `span_id`).
- **Follow-up Probes:** "What is an exemplar in Prometheus?" (A direct link embedding a specific `trace_id` inside a Prometheus metric bucket).

### Q2: Explain the difference between `rate()` and `irate()` in PromQL.
- **Short Answer:** `rate()` calculates the average per-second increase over the entire range window (smoothing out noise); `irate()` calculates the instant rate between the last two data points in the window.
- **Deep Dive:** `rate()` must always be used for alerts and SLO calculations to prevent false alarms on transient spikes. `irate()` is used during live investigation of rapidly changing metrics. Both handle counter resets.
- **Trade-offs & Realities:** Using `irate()` in alerting rules causes alert flapping and noisy false alarms.
- **Follow-up Probes:** "What happens if a counter resets during a `rate()` window?" (Prometheus detects the drop and adjusts calculations assuming monotonicity).

### Q3: What is the difference between an SLI, an SLO, and an SLA?
- **Short Answer:** An SLI is what you measure (metric: 99.8% good requests). An SLO is what you target internally (objective: 99.9% availability). An SLA is what you promise customers legally with financial consequences (agreement: 99.5% or refund).
- **Deep Dive:** SLOs should always be stricter than SLAs. If an SLA is 99.5%, set the internal SLO at 99.9% so engineering reacts and remediates before contractual penalties occur.
- **Trade-offs & Realities:** Setting an SLO to 100% is an anti-pattern: 100% reliability is impossible, economically wasteful, and halts all product innovation.
- **Follow-up Probes:** "What is an Error Budget?" ($1 - \text{SLO}$; the allowable margin of failure).

### Q4: What is high cardinality in metric systems, and why is it dangerous?
- **Short Answer:** High cardinality occurs when a metric label has thousands or millions of unique values (e.g., `user_id`, `email`, `ip_address`).
- **Deep Dive:** Prometheus creates an independent time series in RAM for every unique label combination. High cardinality causes memory explosion, crashes the TSDB with OOM, and degrades query performance.
- **Trade-offs & Realities:** Keep Prometheus labels low cardinality (HTTP status, method, service name). Push high-cardinality data into distributed tracing spans or structured logs.
- **Follow-up Probes:** "How do you identify high-cardinality metrics in Prometheus?" (Query the TSDB status page `/api/v1/status/tsdb` for top series counts).

### Q5: How does the W3C Trace Context standard propagate distributed traces?
- **Short Answer:** Through standardized HTTP/gRPC headers: `traceparent` (version, 16-byte Trace ID, 8-byte Parent Span ID, sampling flags) and `tracestate` (vendor-specific metadata).
- **Deep Dive:** When Service A calls Service B, it injects the `traceparent` header into the HTTP request. Service B extracts the header, creates a child span with the same Trace ID and setting its Parent ID to Service A's span ID, maintaining the call tree.
- **Trade-offs & Realities:** If a legacy proxy or gateway strips custom headers, the trace context breaks, splitting the distributed trace into disconnected fragments.
- **Follow-up Probes:** "What does the `01` flag at the end of `traceparent` mean?" (The trace was sampled and should be recorded).

### Q6: What is the role of an Incident Commander (IC)?
- **Short Answer:** The leader who coordinates the incident response process, assigns roles, facilitates decisions, and manages communications, intentionally avoiding technical debugging to maintain overall situational awareness.
- **Deep Dive:** The IC establishes the incident bridge, directs investigative hypotheses, approves high-risk actions (e.g., rolling back a database), and ensures regular stakeholder updates.
- **Trade-offs & Realities:** If the IC dives into code, no one is managing the big picture or coordinating cross-team response.
- **Follow-up Probes:** "What roles does the IC typically delegate?" (Operations Lead and Communications Lead).

### Q7: What is the difference between Head-Based and Tail-Based sampling in tracing?
- **Short Answer:** Head-based sampling makes the sampling decision at the root service when the request starts. Tail-based sampling buffers spans until the trace finishes, evaluating errors and latency before deciding whether to keep it.
- **Deep Dive:** Head-based sampling is cheap and simple, but frequently drops traces that encounter downstream errors. Tail-based sampling guarantees 100% retention of errors and slow requests, but requires memory buffers on the collector.
- **Trade-offs & Realities:** Tail-based sampling requires routing all spans for a specific Trace ID to the same collector instance using load-balancing exporters.
- **Follow-up Probes:** "What component handles tail-based sampling in OpenTelemetry?" (The OpenTelemetry Collector `tail_sampling` processor).

### Q8: What does a "Blameless Postmortem" mean?
- **Short Answer:** A retrospective focusing on systemic, procedural, and technological root causes of an outage rather than blaming individuals, operating on the premise that engineers act in good faith with the tools they had.
- **Deep Dive:** Blameless culture encourages engineers to report mistakes openly without fear of punishment. Investigations focus on why the system permitted a mistake to cause an outage (e.g., missing guardrails, lack of canaries).
- **Trade-offs & Realities:** Blameless does not mean lack of accountability; engineers are accountable for executing agreed-upon remediation action items.
- **Follow-up Probes:** "What happens when an organization runs blame-heavy postmortems?" (Engineers hide incidents, avoid touching risky legacy systems, and morale drops).

### Q9: Why is `by (le)` mandatory when using `histogram_quantile()` in PromQL?
- **Short Answer:** Because `histogram_quantile()` mathematically interpolates percentiles using the bucket upper-bound label `le`. Aggregating with `sum()` without `by (le)` strips this label, producing invalid calculations.
- **Deep Dive:** Correct syntax: `histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))`.
- **Trade-offs & Realities:** A common interview filter question; omitting `by (le)` demonstrates a lack of real-world PromQL production experience.
- **Follow-up Probes:** "What is the difference between a histogram and a summary in Prometheus?" (Histograms aggregate across instances via buckets on the server; summaries calculate quantiles client-side and cannot be aggregated across instances).

### Q10: What is Chaos Engineering?
- **Short Answer:** The discipline of experimenting on a system by intentionally injecting controlled failures (network delays, packet loss, node crashes) to build confidence in the system's ability to withstand turbulent conditions.
- **Deep Dive:** Validates that circuit breakers trip, pods fail over gracefully, autoscalers trigger, and alert notifications fire before outages occur in production.
- **Trade-offs & Realities:** Chaos engineering should only be introduced once baseline observability and automated alerting are mature; injecting chaos into an un-monitored system is just causing downtime.
- **Follow-up Probes:** "Name two Kubernetes chaos engineering tools." (Chaos Mesh and LitmusChaos).

---

## Tier 2: Deep Dive & Mid-Senior Questions (Questions 11–20)

### Q11: Explain the mathematics of Multi-Window Multi-Burn-Rate alerting.
- **Short Answer:** Measures the rate at which error budget is consumed using paired short (e.g., 5m) and long (e.g., 1h) windows across various burn rates (14.4x, 6x, 1x) to eliminate false alarms and detect leaks rapidly.
- **Deep Dive:**
  - Standard budget consumption over 30 days (720h) is $1\times$.
  - A $14.4\times$ burn rate consumes $2\%$ of the budget in 1 hour ($14.4 / 720 = 2\%/\text{hr}$).
  - Rule condition: `ErrorRate(1h) > 14.4 * (1 - SLO) AND ErrorRate(5m) > 14.4 * (1 - SLO)`.
  - The long window guarantees significant budget loss; the short window guarantees the incident is active right now.
- **Trade-offs & Realities:** Eliminates 95% of alert fatigue compared to naive static threshold alerting.
- **Follow-up Probes:** "What burn rate corresponds to burning 5% of budget in 6 hours?" ($6\times$ burn rate).

### Q12: Walk through the OpenTelemetry Collector architecture and the mandatory ordering of processors.
- **Short Answer:** Receivers -> Processors -> Exporters. The `memory_limiter` processor must always be placed first in the processing pipeline, followed by `batch`, and then transformation/filtering processors.
- **Deep Dive:** The `memory_limiter` continuously samples process memory against percentage thresholds. If memory usage exceeds `limit_percentage`, it drops data or signals backpressure to upstream receivers before downstream batching or transform operations allocate more RAM.
- **Trade-offs & Realities:** If `batch` is placed before `memory_limiter`, large incoming bursts can trigger OOM kills before the limiter can shed load.
- **Follow-up Probes:** "What protocol does the OTel Collector use internally?" (Internal OpenTelemetry transformation data models).

### Q13: How do you design an alert fatigue reduction program for an engineering organization drowning in pages?
- **Short Answer:** Audit all existing alerts; delete non-actionable alerts; replace raw threshold alerts with SLO burn rate alerts; convert warning alerts into tickets; establish on-call review rotations.
- **Deep Dive:**
  1. *Rule of Paging:* An alert should only wake an engineer if it is **urgent, actionable, and customer-impacting**.
  2. *Categorization:* Move internal metrics (e.g., "Pod CPU > 85%") to automated autoscaling. Page only on user-facing symptoms (high error rate, latency breaching SLO).
  3. *Weekly On-Call Handover:* Review every page that fired. If a page required no action, delete the alert or adjust thresholds immediately.
- **Trade-offs & Realities:** Engineers fear deleting alerts; leadership must reinforce that noisy alerts cause more outages than missing alerts due to apathy.
- **Follow-up Probes:** "What is an acceptable number of pages per 24-hour on-call shift?" (Google SRE recommends a maximum of 2 actionable pages per shift).

### Q14: How does Prometheus TSDB compress floating-point time series data?
- **Short Answer:** Using Gorilla compression: timestamps are compressed using delta-of-delta encoding; values are compressed using XOR floating-point encoding.
- **Deep Dive:**
  - *Timestamps:* Most scrapes occur at regular intervals (e.g., every 15s). The difference between consecutive deltas is usually 0, requiring only 1 bit to store.
  - *Values:* XORing consecutive IEEE 754 floats produces leading and trailing zeroes when values change smoothly, compressing 8-byte floats down to an average of ~1.37 bytes per sample.
- **Trade-offs & Realities:** Highly volatile or random float values compress poorly compared to smoothly changing metrics.
- **Follow-up Probes:** "What is a chunk in Prometheus TSDB?" (A 120-sample compressed block of time series data).

### Q15: How do you design an OpenTelemetry tail-based sampling architecture across multiple collector replicas?
- **Short Answer:** Deploy a two-tier collector architecture: Tier 1 (Agents) receives traffic from applications and uses the `loadbalancingexporter` to route spans by `trace_id` hash; Tier 2 (Collectors) receives all spans for a given trace and executes the `tail_sampling` processor.
- **Deep Dive:** Because a single distributed trace spans multiple services and hosts, different spans arrive at different collectors. Tail-based sampling requires all spans of a `trace_id` to reside in the same memory space. The load-balancing exporter hashes `trace_id` to ensure deterministic routing to Tier 2 collectors.
- **Trade-offs & Realities:** Requires managing two collector tiers and allocating sufficient RAM on Tier 2 collectors to buffer traces for the evaluation window (e.g., 30s).
- **Follow-up Probes:** "What happens if a Tier 2 collector crashes?" (In-flight buffered traces currently in memory are lost).

### Q16: How do you establish an Error Budget Policy between SRE and Product teams?
- **Short Answer:** A signed agreement specifying that when a service exhausts its error budget, product feature launches are halted and engineering capacity is redirected 100% to reliability, bug fixes, and platform guardrails.
- **Deep Dive:** The policy establishes clear triggers:
  - *Budget > 20% remaining:* Normal feature delivery.
  - *Budget between 0% and 20%:* Warnings emitted; riskier architectural changes postponed.
  - *Budget < 0% (Exhausted):* Code freeze on non-essential features; team focuses on postmortems, performance tuning, and technical debt until the rolling window recovers.
- **Trade-offs & Realities:** Without executive sponsorship (VP of Engineering), product managers will ignore error budget exhaustion and continue demanding feature launches.
- **Follow-up Probes:** "What happens if a product team consistently ignores the error budget?" (Escalate to leadership or mandate that SRE hands the operational pager back to the product team).

### Q17: What is the difference between Apdex and percentile latency (p95/p99)?
- **Short Answer:** Apdex is an index (0 to 1) comparing requests categorized as Satisfied, Tolerating, and Frustrated against a target threshold $T$. Percentile latency measures the exact response time below which a given percentage of requests fall.
- **Deep Dive:**
  $$\text{Apdex} = \frac{\text{Satisfied Count} + (\text{Tolerating Count} / 2)}{\text{Total Count}}$$
  While Apdex provides a single composite executive score, percentile latencies (p99) expose true long-tail performance degradation that Apdex averages can obscure.
- **Trade-offs & Realities:** A system with a good Apdex score can still have an atrocious p99 latency that severely affects high-value enterprise customers.
- **Follow-up Probes:** "Why is average (mean) latency a misleading metric in distributed systems?" (Averages hide outliers; 1% of users experiencing a 10-second hang is completely invisible in average latency).

### Q18: What is distributed context propagation, and how does it work across asynchronous messaging queues (Kafka / SQS)?
- **Short Answer:** Extracting the W3C trace context from the originating thread, serializing it into message headers, and injecting it into the Kafka/SQS record. The consumer extracts headers from the message to link its processing span as a child of the producer span.
- **Deep Dive:**
  - *Producer:* `otel.GetTextMapPropagator().Inject(ctx, propagation.HeaderCarrier(msg.Headers))`.
  - *Consumer:* `ctx := otel.GetTextMapPropagator().Extract(context.Background(), propagation.HeaderCarrier(msg.Headers))`.
  - A new span is started with `trace.WithSpanKind(trace.SpanKindConsumer)`, preserving the asynchronous link without blocking.
- **Trade-offs & Realities:** If the queue message schema does not support metadata headers, the trace context cannot be propagated.
- **Follow-up Probes:** "What span relationship is used for async messaging?" (`trace.Link` or child span).

### Q19: How do you configure Prometheus to scrape 10,000 targets without crashing?
- **Short Answer:** Implement horizontal sharding using Prometheus Agent mode or Thanos/Mimir; utilize `metric_relabel_configs` to drop unused metrics at scrape time; tune scrape intervals; distribute targets across nodes.
- **Deep Dive:**
  - *Drop Unused Metrics:* Most exporters emit hundreds of useless metrics. Use `metric_relabel_configs` with `action: keep` or `drop` to discard unneeded series before they enter the TSDB.
  - *Prometheus Agent Mode:* Runs lightweight Prometheus instances that scrape targets and stream data via remote-write to central long-term storage (Mimir, Thanos), bypassing local TSDB storage overhead.
- **Trade-offs & Realities:** Dropping metrics at scrape time means those metrics cannot be recovered retroactively.
- **Follow-up Probes:** "What is the difference between `relabel_configs` and `metric_relabel_configs`?" (`relabel_configs` modifies scrape target labels *before* the scrape; `metric_relabel_configs` filters individual metric samples *after* the scrape).

### Q20: Explain the "Five Whys" methodology using a real-world platform infrastructure outage.
- **Candidate Example:**
  - *Incident:* Production API gateway dropped 30% of requests for 15 minutes.
  - *Why 1:* Envoy proxy ran out of file descriptors.
  - *Why 2:* Worker pods experienced high connection buildup in `CLOSE_WAIT`.
  - *Why 3:* Application HTTP client did not close response bodies on non-200 responses.
  - *Why 4:* A new third-party payment vendor API began returning HTTP 429 rate limit errors.
  - *Why 5:* The third-party client integration lacked connection timeouts and automated circuit breaking in the platform SDK.
  - *Systemic Fix:* Patch platform HTTP client library to enforce automatic body closure and connection timeouts; deploy Envoy circuit breakers.

---

## Tier 3: Staff & Principal Architect Scenarios (Questions 21–25)

### Q21: Architect an enterprise-wide observability platform ingesting 10 million metric samples/sec, 5 TB of logs/day, and 1 billion traces/day across multi-cloud environments.
- **Candidate Framework:**
  - *Telemetry Collection:* Deploy OpenTelemetry Collector DaemonSets on all Kubernetes nodes to collect host/pod telemetry; instrument microservices with OTel SDKs exporting OTLP over gRPC.
  - *Metrics Plane:* Deploy Prometheus in Agent Mode scraping local pods and remote-writing to a centralized, horizontally scalable backend (Grafana Mimir or VictoriaMetrics) backed by S3/GCS object storage.
  - *Logs Plane:* Deploy Vector DaemonSets streaming structured JSON logs directly to ClickHouse or Grafana Loki with strict label cardinality limits.
  - *Tracing Plane:* Tier 1 OTel Collectors route traces via `loadbalancingexporter` to Tier 2 Collectors running tail-based sampling (100% errors, 100% latency outliers, 1% baseline), exporting to Grafana Tempo or Jaeger backed by object storage.
  - *Correlation:* Standardize on W3C Trace Context and OpenTelemetry Semantic Conventions; link logs to traces via `trace_id` injection and metrics via Prometheus exemplars.
- **Red Flags:** Recommending a single massive standalone Prometheus or Elasticsearch instance with no sharding or retention policies.

### Q22: You are Incident Commander for a Sev-1 outage where the primary customer database is down and engineers are panicking and arguing over conflicting fixes. How do you lead?
- **Candidate Framework:**
  - *Take Control & Set Tone:* Establish authority calmly: "I am Jane Doe, Incident Commander. All actions must be coordinated through me. Please pause all manual terminal commands immediately."
  - *Establish Roles:* Designate Lead DBA as Operations Lead; appoint Senior Engineer as Scribe; assign Support Lead as Comms Lead.
  - *Formulate & Test Hypotheses:* Direct the Ops Lead to evaluate hypotheses one at a time. Demand evidence before executing actions: "What data proves this will fix it? What is the rollback plan if this makes it worse?"
  - *Time-Box Actions:* Give 10 minutes to evaluate hypothesis A; if no progress, pivot to hypothesis B (e.g., failover to read replica).
  - *Maintain Cadence:* Instruct Comms Lead to publish internal and external status updates every 15 minutes.
- **Red Flags:** Panicking, joining the technical argument, or letting multiple engineers execute uncoordinated changes on production databases simultaneously.

### Q23: How would you design a Chaos Engineering GameDay program to validate zero-downtime multi-region failover?
- **Candidate Framework:**
  - *Define Steady State:* Quantify normal business metrics (e.g., checkout success rate 99.95%, p99 latency < 200ms).
  - *Formulate Hypothesis:* "Severing all network connectivity to AWS Region `us-east-1` will cause Route53 ARC (Application Recovery Controller) to detect health failure and shift 100% of global traffic to `us-west-2` within 60 seconds with zero data corruption."
  - *Staged Rollout:*
    - Phase 1: Test in pre-production staging environment during business hours.
    - Phase 2: Execute GameDay in production with executive approval, cross-functional teams present on an active bridge, and explicit abort triggers (e.g., abort if gross error rate exceeds 1%).
  - *Inject Chaos:* Trigger network black-hole on ingress gateways in Region 1.
  - *Measure & Verify:* Measure actual failover latency, DNS propagation time, and database replication lag.
  - *Retrospective:* Document gaps and file priority engineering action items.
- **Red Flags:** Running chaos experiments in production without clear steady-state metrics or immediate abort triggers.

### Q24: A CTO complains: "We spend $250,000 a month on Datadog, but our developers still can't diagnose outages quickly." How do you evaluate and solve this?
- **Candidate Framework:**
  - *Audit Current Spend:* Analyze Datadog cost breakdown: typically 70% of cost is driven by custom metrics (high cardinality) and un-sampled log ingestion.
  - *Identify Diagnostic Gaps:* Developers struggle because telemetry is noisy, alerts lack actionable runbooks, and logs are unstructured plaintext.
  - *Cost Optimization & Architecture Migration:*
    - Deploy in-cluster OpenTelemetry Collectors to filter, aggregate, and drop unneeded metrics and debug logs before egress.
    - Implement tail-based sampling on traces.
    - Evaluate migrating to an open-source, object-storage-backed stack (Mimir, Loki, Tempo, Grafana) managed by the platform team or a managed cloud alternative.
  - *Developer Enablement:* Clean up alert rules, establish standardized Golden Path dashboards, and embed runbook links in every alert notification.
- **Red Flags:** Canceling the monitoring tool immediately without understanding the diagnostic workflow gaps.

### Q25: How do you design an automated SLO and Error Budget reporting engine integrated with the Developer Portal?
- **Candidate Framework:**
  - *Service Catalog Integration:* In Backstage/Port, every service entity's `catalog-info.yaml` defines its SLO reference:
    ```yaml
    metadata:
      annotations:
        platform.example.com/slo-target: "99.9"
        platform.example.com/prometheus-query: "job:http_requests:error_rate5m"
    ```
  - *Telemetry Ingestion:* A backend platform cron queries Prometheus / Thanos daily, calculating 7-day and 30-day budget burn percentages.
  - *Portal Scorecard:* Displays a visual health indicator on the service dashboard:
    - *Green:* Error budget > 50%.
    - *Yellow:* Error budget between 10% and 50%.
    - *Red:* Error budget exhausted (<0%).
  - *Deployment Gate:* Integrate with CI/CD: if a service has exhausted its 30-day error budget, automated GitOps promotion to production is blocked, requiring an explicit override from the Engineering Director.
- **Red Flags:** Treating SLOs as static documents on Confluence rather than living telemetry integrated into deployment pipelines.
