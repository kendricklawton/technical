# Domain 3: Implementing Service Monitoring Strategies — Performance Labs & Drills

---

## Lab 1: Configure PodMonitoring for Managed Service for Prometheus (GMP)

### Objective
Deploy a Google Cloud Managed Service for Prometheus `PodMonitoring` resource to scrape Prometheus metrics from an application pod on GKE without managing Prometheus servers.

```yaml
# podmonitoring.yaml
apiVersion: monitoring.googleapis.com/v1
kind: PodMonitoring
metadata:
  name: order-service-prometheus
  namespace: e-commerce
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: order-api
  endpoints:
    - port: metrics
      path: /metrics
      interval: 15s
      metricRelabeling:
        - action: keep
          regex: http_requests_total|http_request_duration_seconds.*
          sourceLabels: [__name__]
```

### Verification
```bash
# 1. Apply PodMonitoring manifest
kubectl apply -f podmonitoring.yaml

# 2. Query metric in Cloud Monitoring using PromQL
# PromQL: sum(rate(http_requests_total{namespace="e-commerce"}[5m]))
```

---

## Lab 2: Create a Log-Based Metric with Metric Extraction

### Objective
Create a Cloud Logging distribution metric extracting response latency from JSON logs to measure P95 and P99 API response times.

```bash
cat << 'EOF' > /tmp/latency-metric.json
{
  "name": "api_request_latency",
  "description": "Measures API request latency in milliseconds from JSON logs",
  "filter": "resource.type=\"k8s_container\" AND jsonPayload.latency_ms > 0",
  "valueExtractor": "EXTRACT(jsonPayload.latency_ms)",
  "bucketOptions": {
    "exponentialBuckets": {
      "numFiniteBuckets": 64,
      "growthFactor": 1.4,
      "scale": 1.0
    }
  },
  "metricDescriptor": {
    "metricKind": "DELTA",
    "valueType": "DISTRIBUTION",
    "unit": "ms"
  }
}
EOF

gcloud logging metrics create api_request_latency \
  --config-from-file=/tmp/latency-metric.json
```

---

## Lab 3: Log Router Cost Optimization Sink & Exclusion Rule

### Objective
Route all high-severity audit logs to BigQuery for long-term analytics, while excluding noisy HTTP 200 healthcheck logs from default ingestion.

```bash
# 1. Create a log sink forwarding all security and error logs to BigQuery
gcloud logging sinks create bq-audit-sink \
  bigquery.googleapis.com/projects/${PROJECT_ID}/datasets/security_audit_logs \
  --log-filter='severity>=WARNING OR protoPayload.methodName=~".*create.*|.*delete.*|.*update.*"'

# 2. Add an exclusion filter to the _Default log bucket to drop healthcheck logs
gcloud logging sinks update _Default \
  --add-exclusion="name=exclude-healthchecks,filter=resource.type=\"k8s_container\" AND httpRequest.requestUrl=~\"/healthz|/livez\" AND httpRequest.status=200"
```
