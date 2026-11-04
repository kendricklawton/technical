# Domain 3: Implementing Service Monitoring Strategies — Quiz 2

**Exam:** Google Cloud Certified: Professional Cloud DevOps Engineer  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
In Google Cloud Monitoring, what is the effect of configuring an **Incident Autoclose** duration (e.g. 7 days) on an alerting policy?
- A. If an alerting condition stops being evaluated or metrics cease reporting for the specified duration, the incident is automatically marked closed to prevent orphaned incidents from lingering indefinitely.
- B. It deletes the project.
- C. It deletes the logs.
- D. It automatically restarts the failing pods after 7 days.

---

### Question 2
When creating a Cloud Monitoring alert for a metric that experiences predictable daily seasonality (e.g. high traffic during business hours and low traffic at night), which condition type avoids false alerts during expected dips?
- A. Metric Absence condition
- B. **Forecast / Anomaly Detection condition** (or time-shifted baseline comparison evaluating deviations from historical trends)
- C. Log count condition
- D. Static Threshold condition

---

### Question 3
An application runs on Google Cloud Run. The platform team needs to receive an alert if the count of HTTP 5xx errors returned by Cloud Run exceeds 50 errors per minute for 3 consecutive minutes. Which metric should be evaluated in the alerting policy?
- A. `container.googleapis.com/node/memory/used`
- B. `compute.googleapis.com/instance/cpu/utilization`
- C. `logging.googleapis.com/byte_count`
- D. `run.googleapis.com/request_count` filtered by `response_code_class="5xx"`

---

### Question 4
In Cloud Logging, what is a **Log Scope**?
- A. A physical camera inside the Google data center.
- B. A tool to format Python logs.
- C. A resource that groups one or more log buckets, allowing users to query, analyze, and restrict access across multiple project log buckets through a single query interface.
- D. A network firewall rule.

---

### Question 5
Why should an SRE avoid putting high-cardinality values (such as customer email addresses or unique transaction UUIDs) into metric labels in Cloud Monitoring?
- A. Metric labels cannot contain text.
- B. Each unique combination of label values creates a distinct new time series; high-cardinality values cause a time-series cardinality explosion that exceeds quota limits, slows dashboard rendering, and incurs massive API costs.
- C. Cloud Monitoring automatically converts labels into SQL tables.
- D. Google Cloud charges per character in labels.

---

### Question 6
You need to visualize the end-to-end latency of a distributed transaction that hops through an API Gateway, three microservices on GKE, and a Cloud SQL database. What W3C header must be passed in HTTP requests between these services to preserve the distributed trace context?
- A. `Cookie`
- B. `X-Custom-Host`
- C. `Authorization`
- D. `traceparent`

---

### Question 7
In Cloud Profiler, what does a wide horizontal bar in a flame graph represent?
- A. The function is running on a wide-screen monitor.
- B. The function (and its callees) accounted for a large proportion of the total resource consumed (such as CPU execution time or allocated heap bytes) during the profiling period.
- C. The function was executed 100 times.
- D. The function failed with an exception.

---

### Question 8
Which Cloud Monitoring feature allows an operations team to define automated synthetic blackbox HTTP, HTTPS, or TCP checks against a public endpoint from multiple geographic locations worldwide?
- A. **Uptime Checks**
- B. Cloud Armor security policies
- C. Cloud DNS private zones
- D. Compute Engine health checks

---

### Question 9
In Google Cloud Managed Service for Prometheus, how can engineers query collected Prometheus metrics alongside native Google Cloud metrics on the same dashboard?
- A. Export metrics to Excel.
- B. It is impossible to view them on the same dashboard.
- C. Recompile the dashboard in C++.
- D. Use Cloud Monitoring Dashboards with PromQL support enabled in the query editor.

---

### Question 10
What is the function of the **Ops Agent** on Google Compute Engine instances?
- A. It manages user SSH keys.
- B. It reboots VMs during kernel panics.
- C. It unifies logging and metric collection into a single high-performance agent running Fluent Bit (for structured logs) and OpenTelemetry (for system and third-party application metrics).
- D. It automatically pays the cloud bill.

---

## Answer Key & Rationales

1. **A** — Incident Autoclose ensures that if a resource is deleted or ceases sending metrics while in an alert state, the open incident will automatically resolve after the specified timeout.
2. **B** — Anomaly detection and baseline comparison alert conditions track dynamic statistical deviations from historical patterns rather than static fixed numbers.
3. **D** — `run.googleapis.com/request_count` filtered by response code class `5xx` accurately tracks application server errors on Cloud Run.
4. **C** — Log Scopes provide an abstraction layer aggregating log buckets across projects and organizations, facilitating centralized security and operational log querying.
5. **B** — High-cardinality metric labels generate millions of active time series, exhausting Google Cloud time series limits and corrupting metric ingestion.
6. **D** — The W3C Trace Context standard specifies the `traceparent` header to propagate trace ID, parent span ID, and trace flags across distributed HTTP boundaries.
7. **B** — In a flame graph, the horizontal width of a box is proportional to the fraction of CPU time or memory allocated by that function and its children.
8. **A** — Cloud Monitoring Uptime Checks perform external synthetic probes from multiple locations across the globe to verify external reachability and TLS certificate validity.
9. **D** — Google Cloud Monitoring natively supports PromQL in dashboard charts, alerting policies, and Metrics Explorer, providing a unified single pane of glass.
10. **C** — The Google Cloud Ops Agent combines Fluent Bit and OpenTelemetry Collector into an optimized agent for Compute Engine virtual machines.
