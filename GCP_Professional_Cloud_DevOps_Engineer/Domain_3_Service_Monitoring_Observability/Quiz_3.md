# Domain 3: Implementing Service Monitoring Strategies — Quiz 3

**Exam:** Google Cloud Certified: Professional Cloud DevOps Engineer  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
You have an alerting policy configured on an external HTTPS endpoint using a Cloud Monitoring Uptime Check. You want to alert the on-call engineer ONLY if the endpoint fails health checks from at least 2 distinct geographic regions simultaneously. Why is this multi-region threshold recommended?
- A. To avoid sending false-positive alerts caused by localized, transient Internet routing disruptions or ISP anomalies between a single probing location and the target.
- B. To reduce the cost of Cloud Monitoring.
- C. Because Google Cloud requires at least 2 regions for all alerts.
- D. To verify that all Google data centers are operational.

---

### Question 2
In Monitoring Query Language (MQL), which operation calculates the per-second rate of change of a cumulative counter metric (such as total network bytes or request count)?
- A. `sum()`
- B. `rate()`
- C. `count()`
- D. `average()`

---

### Question 3
An engineering team wants to monitor database query latency extracted from log messages. The log messages contain JSON field `jsonPayload.db_query_time_ms: 45.2`. What type of Cloud Logging metric must be created to compute the 99th percentile ($P99$) query latency?
- A. Counter metric
- B. Text search metric
- C. Boolean metric
- D. Distribution metric with exponential or linear bucket options

---

### Question 4
In Cloud Logging, what happens if an incoming log entry matches an active **Exclusion filter** configured on a log sink?
- A. The log entry is written to an external file on local disk.
- B. The log entry triggers a Cloud Function.
- C. The log entry is dropped before being ingested into the destination log bucket, preventing both ingestion and storage charges.
- D. The application pod is terminated.

---

### Question 5
How does **Cloud Trace** determine sampling rates for high-throughput microservices (e.g. 50,000 requests per second) to prevent trace ingestion overhead and excessive costs?
- A. It uses configurable head-based probabilistic sampling (e.g. sampling 1 out of every 1,000 requests) or adaptive rate-limiting (e.g. max 10 traces per second per container).
- B. It captures 100% of all requests without sampling.
- C. It only samples requests during business hours.
- D. It drops all traces that take longer than 1 second.

---

### Question 6
An SRE wants to configure an alert that fires if an application pod stops sending heartbeat metrics entirely for 10 minutes (indicating the pod has hung or crashed silently). Which Cloud Monitoring condition type should be selected?
- A. Anomaly condition
- B. Metric Absence condition
- C. Log query condition
- D. Metric Threshold condition

---

### Question 7
In Google Cloud Error Reporting, what criteria does the service use to group stack traces into the same issue?
- A. The IP address of the worker node.
- B. The user ID of the client making the request.
- C. The exception type, stack trace call frame signatures, and root-cause code location.
- D. The time of day the crash occurred.

---

### Question 8
What is the function of the `_Required` log bucket in Google Cloud Logging?
- A. It holds Admin Activity audit logs, system event audit logs, and Access Transparency logs; it cannot be modified, deleted, or disabled, and retains logs for 400 days free of charge.
- B. It stores temporary cache files.
- C. It hosts container images.
- D. It stores developer debug logs.

---

### Question 9
In Cloud Monitoring Dashboards, which chart type is most appropriate for visualizing the correlation between CPU utilization and request latency across 50 GKE pods?
- A. Pie chart
- B. Single-value scorecard
- C. Text widget
- D. Scatter plot or Heatmap

---

### Question 10
When using Google Cloud Managed Service for Prometheus (GMP), how does an SRE scrape metrics from pods across multiple namespaces using a single cluster-scoped resource?
- A. By configuring an SSH tunnel to each node.
- B. Deploying a `ClusterPodMonitoring` custom resource that matches pods cluster-wide regardless of namespace.
- C. Using multiple PodMonitoring CRDs in every namespace.
- D. By disabling Kubernetes RBAC.

---

## Answer Key & Rationales

1. **A** — Requiring failures across multiple distinct geographic prober regions (e.g. 2 of 3) filters out regional transit hiccups and ISP-specific peering issues, ensuring only genuine outages trigger pages.
2. **B** — In MQL, `rate(val())` computes the per-second rate of change of cumulative time series values over specified alignment windows.
3. **D** — Computing percentiles ($P95$, $P99$) requires a Distribution metric that bins extracted continuous numerical values into histogram buckets.
4. **C** — Log exclusion filters drop matching log entries in the Log Router before they are stored in log buckets, saving both ingestion and retention costs.
5. **A** — Distributed tracing uses probabilistic or rate-limited sampling to maintain statistical significance while keeping network and storage overhead negligible.
6. **B** — Metric Absence conditions trigger an alert when a time series stops reporting data points for a specified duration, ideal for detecting silent process deadlocks.
7. **C** — Error Reporting groups exceptions by analyzing stack trace frames, function names, and exception types to deduplicate identical errors into a single actionable issue.
8. **A** — The `_Required` log bucket is an immutable, non-configurable security audit bucket that retains Admin Activity logs for 400 days at no cost.
9. **D** — Heatmaps and scatter plots allow visual correlation of multi-dimensional distributions across hundreds of pods simultaneously, revealing clusters and outliers.
10. **B** — `ClusterPodMonitoring` is a cluster-scoped CRD in GMP that scrapes endpoints matching label selectors across all namespaces in the cluster.
