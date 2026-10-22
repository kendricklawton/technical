# Domain 3: Implementing Service Monitoring Strategies — Authoritative White Papers & SRE Standards

> **Exam Focus:** Observability Best Practices, Distributed Tracing, and Log Cost Optimization.

---

## 1. Authoritative References & Standards

- **[Google Cloud Architecture Framework — Operational Excellence](https://cloud.google.com/architecture/framework/operational-excellence):** Logging and monitoring best practices, centralized observability.
- **[Google SRE Book — Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/):** The philosophy of monitoring, symptoms vs causes, and actionable paging.
- **[Google Cloud Managed Service for Prometheus Guide](https://cloud.google.com/stackdriver/docs/managed-prometheus):** Architecture, PodMonitoring CRDs, and PromQL integration.
- **[Google Cloud Logging Best Practices](https://cloud.google.com/logging/docs/best-practices):** Structured logging in JSON, log exclusion filters, and cost containment.

---

## 2. Staff SRE Architectural Patterns

### 2.1 Structured Logging Standards
Unstructured string logs (`log.Printf("User %s failed", user)`) prevent automated querying and metric extraction. All production microservices must emit structured JSON payloads:
```json
{
  "severity": "ERROR",
  "message": "Payment processing failed",
  "httpRequest": {
    "requestMethod": "POST",
    "requestUrl": "/v1/charge",
    "status": 500,
    "latency": "0.450s"
  },
  "logging.googleapis.com/trace": "projects/my-proj/traces/0af7651916cd43dd8448eb211c80319c",
  "logging.googleapis.com/spanId": "b7ad6b7169203331",
  "user_id": "cust_12345",
  "error_code": "INSUFFICIENT_FUNDS"
}
```
*Injecting `logging.googleapis.com/trace` enables 1-click correlation between Cloud Logging and Cloud Trace spans.*

### 2.2 Log Ingestion Cost Management
Log volume can easily spiral out of control. Staff SREs implement Log Router **exclusion filters** to drop redundant debug/healthcheck logs before storage:
```text
resource.type="k8s_container" AND jsonPayload.path="/healthz" AND httpRequest.status=200
```
*Excluding non-critical logs reduces Cloud Logging monthly bills by 40–70% while preserving diagnostic fidelity.*

---

## 3. Production Failure Modes & Anti-Patterns

| Anti-Pattern | Operational Risk | Production Resolution |
| :--- | :--- | :--- |
| **Paging on High Memory Utilization Alone** | Paging engineers because a Java container's memory is 95% full when Java heaps are designed to fill memory before GC. | Alert on Container OOM Kills (`kubernetes.io/container/restart_count` with termination reason `OOMKilled`) rather than steady-state JVM heap usage. |
| **High Cardinality Custom Metrics** | Emitting custom metrics with user IDs or order IDs in labels blows past Cloud Monitoring time series quotas, resulting in massive metric drops and bills. | Keep metric labels low cardinality (region, cluster, service, status_code). Route high-cardinality metadata exclusively to distributed trace spans or logs. |
| **Unlinked Logs and Traces** | SREs cannot correlate an error log with the specific distributed trace span that triggered it, lengthening incident triage. | Inject the W3C trace ID into the structured JSON log entry's `logging.googleapis.com/trace` field. |
