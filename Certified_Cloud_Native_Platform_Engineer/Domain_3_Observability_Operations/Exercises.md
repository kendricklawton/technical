# Domain 3: Observability & Operations — Performance Labs & Drills

---

## Lab 1: Deploy OpenTelemetry Collector with Tail-Based Sampling & Prometheus Exporter

### Objective
Deploy an OpenTelemetry Collector instance that:
1. Receives OTLP telemetry on gRPC `:4317` and HTTP `:4318`.
2. Uses `memory_limiter` and `batch` processors.
3. Implements `tail_sampling` to retain 100% of error spans and 1% of normal 200 OK spans.
4. Exports metrics to Prometheus via an in-cluster scrape endpoint `:8889`.

```yaml
apiVersion: opentelemetry.io/v1alpha1
kind: OpenTelemetryCollector
metadata:
  name: platform-otel-collector
  namespace: observability
spec:
  mode: deployment
  replicas: 2
  config: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
          http:
            endpoint: 0.0.0.0:4318

    processors:
      memory_limiter:
        check_interval: 1s
        limit_percentage: 75
        spike_limit_percentage: 20
      batch:
        send_batch_size: 8192
        timeout: 5s
      tail_sampling:
        decision_wait: 10s
        num_traces: 10000
        expected_new_traces_per_sec: 2000
        policies:
          [
            {
              name: error-policy,
              type: status_code,
              status_code: { status_codes: [ ERROR ] }
            },
            {
              name: probabilistic-policy,
              type: probabilistic,
              probabilistic: { sampling_percentage: 1.0 }
            }
          ]

    exporters:
      prometheus:
        endpoint: 0.0.0.0:8889
      otlp/tempo:
        endpoint: tempo.observability.svc.cluster.local:4317
        tls:
          insecure: true

    service:
      pipelines:
        traces:
          receivers: [otlp]
          processors: [memory_limiter, tail_sampling, batch]
          exporters: [otlp/tempo]
        metrics:
          receivers: [otlp]
          processors: [memory_limiter, batch]
          exporters: [prometheus]
```

### Verification
```bash
kubectl apply -f otel-collector.yaml
kubectl get pods -n observability -l app.kubernetes.io/name=platform-otel-collector
kubectl logs -n observability -l app.kubernetes.io/name=platform-otel-collector --tail=50
```

---

## Lab 2: Prometheus Operator ServiceMonitor & Alerting Rule Configuration

### Objective
Create a `ServiceMonitor` targeting an internal platform microservice (`order-api`) and a `PrometheusRule` triggering a high-priority alert if the 5xx error rate exceeds 5% over a 5-minute sliding window.

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: order-api-monitor
  namespace: e-commerce
  labels:
    release: prometheus-stack
spec:
  selector:
    matchLabels:
      app: order-api
  endpoints:
    - port: metrics
      path: /metrics
      interval: 15s
      scrapeTimeout: 10s
---
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: order-api-alerts
  namespace: e-commerce
  labels:
    role: alert-rules
    release: prometheus-stack
spec:
  groups:
    - name: order-api.rules
      rules:
        - alert: OrderApiHighErrorRate
          expr: |
            (
              sum(rate(http_requests_total{job="order-api", status=~"5.."}[5m]))
              /
              sum(rate(http_requests_total{job="order-api"}[5m]))
            ) * 100 > 5
          for: 5m
          labels:
            severity: critical
            team: platform-core
          annotations:
            summary: "High 5xx error rate detected on order-api: {{ $value | printf '%.2f' }}%"
            description: "order-api is failing more than 5% of requests over the last 5 minutes. Check pod logs and upstream database latency."
            runbook_url: "https://wiki.internal/ops/runbooks/order-api-errors"
```

### Verification
```bash
kubectl apply -f service-monitor-alert.yaml
kubectl get servicemonitors -n e-commerce
kubectl get prometheusrules -n e-commerce
```

---

## Lab 3: Alertmanager Routing Tree & Notification Silencing

### Objective
Configure an `AlertmanagerConfig` CRD that routes alerts with `severity: critical` to PagerDuty and all other alerts to Slack, with an inhibition rule silencing pod alerts when an entire node is down.

```yaml
apiVersion: monitoring.coreos.com/v1alpha1
kind: AlertmanagerConfig
metadata:
  name: platform-routing
  namespace: observability
spec:
  route:
    groupBy: ['alertname', 'cluster', 'service']
    groupWait: 30s
    groupInterval: 5m
    repeatInterval: 4h
    receiver: 'slack-notifications'
    routes:
      - matchers:
          - name: severity
            value: critical
        receiver: 'pagerduty-urgent'
  inhibitRules:
    - sourceMatch:
        - name: alertname
          value: NodeDown
      targetMatch:
        - name: severity
          value: critical
      equal: ['node', 'instance']
  receivers:
    - name: 'slack-notifications'
      slackConfigs:
        - channel: '#platform-ops'
          sendResolved: true
          apiURL:
            name: alert-secrets
            key: slack-webhook-url
    - name: 'pagerduty-urgent'
      pagerdutyConfigs:
        - routingKey:
            name: alert-secrets
            key: pagerduty-service-key
          sendResolved: true
```

### Verification
```bash
kubectl apply -f alertmanager-config.yaml
kubectl get alertmanagerconfigs -n observability
```
