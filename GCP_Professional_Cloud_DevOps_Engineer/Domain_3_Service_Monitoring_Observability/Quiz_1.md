# Domain 3: Implementing Service Monitoring Strategies — Quiz 1

**Exam:** Google Cloud Certified: Professional Cloud DevOps Engineer  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
An SRE team notices their monthly Google Cloud bill for Cloud Logging has tripled. Analysis reveals that three high-volume microservices emit millions of HTTP 200 healthcheck logs every minute (`httpRequest.requestUrl="/healthz"`). The logs provide no operational value. How should the team eliminate the ingestion and storage costs for these specific log entries without stopping the application from emitting them?
- A. Terminate the GKE cluster.
- B. Rewrite the healthcheck probe in Python.
- C. Delete the Cloud Monitoring workspace.
- D. Configure a **Log exclusion filter** on the `_Default` log sink in Cloud Logging matching the healthcheck URL and 200 status code.

---

### Question 2
You want to collect custom Prometheus metrics from application pods running in a GKE cluster without deploying, patching, or managing Prometheus servers, Alertmanager instances, or stateful disks. Which Google Cloud service natively handles this?
- A. Google Cloud Dataproc
- B. Google Cloud Managed Service for Prometheus (GMP) using the `PodMonitoring` custom resource.
- C. Google Cloud Deployment Manager
- D. Google Cloud Bigtable

---

### Question 3
An application deployed across 20 microservices experiences intermittent 3-second latency spikes during checkout transactions. Logs show no error messages, and CPU utilization across all pods remains under 30%. Which Google Cloud tool should an SRE use to inspect the end-to-end distributed call chain and pinpoint which specific remote procedure call (RPC) is causing the delay?
- A. Cloud Trace
- B. Cloud Filestore
- C. Cloud Storage Transfer Service
- D. Cloud Armor

---

### Question 4
An SRE needs to inspect which functions inside a Go production microservice are consuming the most CPU cycles during peak trading hours, without recompiling the binary with debug flags or causing noticeable performance degradation. Which tool provides continuous production call-stack flame graphs?
- A. VPC Flow Logs
- B. Cloud NAT
- C. Cloud Profiler
- D. Compute Engine OS Login

---

### Question 5
What is the structural difference between a Cloud Logging **Counter metric** and a **Distribution metric**?
- A. Counter metrics count the number of matching log entries over time; Distribution metrics extract a numeric value (e.g. latency, payload size) from structured log entries and group values into statistical histogram buckets to calculate percentiles.
- B. Counter metrics cost \$100 per query; Distribution metrics are free.
- C. Counter metrics are only for Linux; Distribution metrics are for Windows.
- D. There is no difference.

---

### Question 6
You are creating a Cloud Monitoring Alerting Policy to notify engineers when a critical backend service is down. Which notification channel configuration ensures the team is paged immediately 24/7 with automated escalation policies?
- A. Writing an entry to Cloud Storage.
- B. A weekly Slack digest.
- C. PagerDuty or Opsgenie webhook integration configured in Cloud Monitoring notification channels.
- D. Personal email address of the developer.

---

### Question 7
In Cloud Logging, what is the default retention period for log entries stored in the `_Default` log bucket?
- A. Indefinitely
- B. 365 days
- C. 7 days
- D. 30 days

---

### Question 8
You need to retain security audit logs for 7 years to comply with financial industry regulations, and you want to be able to run fast, complex SQL queries across billions of historical log entries when auditors request reports. Where should you route these logs using a Cloud Logging sink?
- A. Cloud Pub/Sub
- B. BigQuery
- C. Compute Engine local scratch disk
- D. Cloud Run memory

---

### Question 9
How can developers ensure that log entries in Cloud Logging are automatically grouped with corresponding distributed trace spans in the Cloud Trace UI?
- A. By using plain text log strings.
- B. Traces and logs cannot be linked in Google Cloud.
- C. By outputting structured JSON logs that include the `logging.googleapis.com/trace` field populated with the project's trace ID: `projects/<PROJECT_ID>/traces/<TRACE_ID>`.
- D. By restarting the GKE node agent.

---

### Question 10
Which Google Cloud Operations tool automatically parses unhandled exceptions and stack traces from application logs, deduplicates them into distinct issue groups, and sends alerts when a new crash signature appears in production?
- A. Error Reporting
- B. Cloud Memorystore
- C. Cloud Profiler
- D. Cloud Key Management Service

---

## Answer Key & Rationales

1. **D** — Log exclusion filters in the Log Router drop non-essential logs before ingestion, preventing both ingestion and storage charges while allowing other logs to flow unimpeded.
2. **B** — Google Cloud Managed Service for Prometheus (GMP) offers a fully managed, serverless Prometheus ingestion engine that uses declarative `PodMonitoring` CRDs.
3. **A** — Cloud Trace collects distributed trace contexts across microservices, visualizing the timeline of RPC calls, database queries, and external API requests in a single waterfall graph.
4. **C** — Cloud Profiler continuously profiles CPU and memory allocations in production with negligible overhead ($< 1\%$), rendering actionable flame graphs.
5. **A** — Counter metrics count matching log events; Distribution metrics extract continuous numerical values to compute statistical distributions, percentiles ($P50$, $P95$, $P99$), and averages.
6. **C** — PagerDuty/Opsgenie notification channels provide incident paging, on-call schedules, and automated multi-tier escalation paths.
7. **D** — The `_Default` log bucket in Google Cloud Logging retains logs for 30 days free of charge before automatic expiration.
8. **B** — BigQuery provides serverless, highly optimized SQL query engines over petabytes of structured log data, ideal for multi-year compliance analytics.
9. **C** — Including the `logging.googleapis.com/trace` attribute in structured JSON log entries enables native 1-click correlation between Cloud Logging and Cloud Trace spans.
10. **A** — Error Reporting automatically aggregates, de-duplicates, and categorizes application stack traces (Java, Python, Go, Node.js, .NET), alerting teams to new runtime crash patterns.
