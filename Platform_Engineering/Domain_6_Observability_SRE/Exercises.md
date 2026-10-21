# PE Domain 6: Observability & SRE — Hands-On Labs & Exercises

Six production-grade labs drilling PromQL mastery, OpenTelemetry Collector configuration, multi-burn-rate alerting, distributed tracing, and postmortem authoring.

---

## Lab 1: Authoring Complex PromQL Queries and Recording Rules

### Objective
Write production-grade PromQL queries calculating error rates, 99th percentile latencies, Apdex scores, and define a high-performance `PrometheusRule` recording rule.

### Step-by-Step Implementation

1. **Define Recording Rules (`prometheus-recording-rules.yaml`):**
   ```yaml
   apiVersion: monitoring.coreos.com/v1
   kind: PrometheusRule
   metadata:
     name: payment-api-recording-rules
     namespace: monitoring
   spec:
     groups:
       - name: payment-api.metrics
         interval: 30s
         rules:
           # Precompute total request rate per service
           - record: job:http_requests:rate5m
             expr: sum(rate(http_requests_total{job="payments"}[5m])) by (handler, status)

           # Precompute error rate percentage
           - record: job:http_requests:error_rate5m
             expr: |
               sum(rate(http_requests_total{job="payments", status=~"5.."}[5m]))
               /
               sum(rate(http_requests_total{job="payments"}[5m]))

           # Precompute 99th percentile latency
           - record: job:http_latency:p99_5m
             expr: |
               histogram_quantile(
                 0.99,
                 sum(rate(http_request_duration_seconds_bucket{job="payments"}[5m])) by (le)
               )
   ```

2. **Verify PromQL Syntax with `promtool`:**
   ```bash
   promtool check rules prometheus-recording-rules.yaml
   ```

---

## Lab 2: Deploying an OpenTelemetry Collector Pipeline

### Objective
Configure a complete OpenTelemetry Collector pipeline incorporating `memory_limiter`, `batch`, and `transform` processors to ingest OTLP traces and export to Prometheus and Jaeger.

### Implementation Config (`otel-collector-config.yaml`)
```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  # Memory limiter MUST be first processor in pipeline
  memory_limiter:
    check_interval: 1s
    limit_percentage: 75
    spike_limit_percentage: 20

  batch:
    send_batch_size: 1024
    timeout: 5s

  # Strip PII attributes before exporting
  transform:
    error_mode: ignore
    trace_statements:
      - context: span
        statements:
          - delete_key(attributes, "http.request.header.authorization")
          - delete_key(attributes, "user.credit_card")

exporters:
  prometheus:
    endpoint: "0.0.0.0:8889"
    namespace: "platform"

  otlp/tempo:
    endpoint: "tempo:4317"
    tls:
      insecure: true

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch, transform]
      exporters: [otlp/tempo]
    metrics:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [prometheus]
```

---

## Lab 3: Configuring Multi-Window Multi-Burn-Rate Prometheus Alerting

### Objective
Implement the complete Google SRE multi-window multi-burn-rate alerting rules for a 99.9% SLO on a mission-critical platform service.

### Alerting Rules (`slo-burn-rate-rules.yaml`)
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: payment-service-slo-alerts
  namespace: monitoring
spec:
  groups:
    - name: payment-service-slo
      rules:
        # 14.4x burn rate over 1h and 5m -> Critical Page (2% budget consumed in 1h)
        - alert: PaymentServiceHighBurnRatePage
          expr: |
            (
              sum(rate(http_requests_total{job="payments", status=~"5.."}[1h]))
              /
              sum(rate(http_requests_total{job="payments"}[1h]))
            ) > (14.4 * (1 - 0.999))
            and
            (
              sum(rate(http_requests_total{job="payments", status=~"5.."}[5m]))
              /
              sum(rate(http_requests_total{job="payments"}[5m]))
            ) > (14.4 * (1 - 0.999))
          for: 2m
          labels:
            severity: page
            slo: "99.9"
          annotations:
            summary: "Payment service burning 14.4x error budget"
            description: "Error rate is consuming >2% of the 30-day budget within 1 hour."

        # 6x burn rate over 6h and 30m -> Page
        - alert: PaymentServiceMediumBurnRatePage
          expr: |
            (
              sum(rate(http_requests_total{job="payments", status=~"5.."}[6h]))
              /
              sum(rate(http_requests_total{job="payments"}[6h]))
            ) > (6 * (1 - 0.999))
            and
            (
              sum(rate(http_requests_total{job="payments", status=~"5.."}[30m]))
              /
              sum(rate(http_requests_total{job="payments"}[30m]))
            ) > (6 * (1 - 0.999))
          for: 5m
          labels:
            severity: page
            slo: "99.9"
          annotations:
            summary: "Payment service burning 6x error budget"
            description: "Error rate is consuming >5% of the 30-day budget within 6 hours."
```

---

## Lab 4: Instrumenting a Distributed Go Service with OpenTelemetry

### Objective
Write a Go HTTP microservice that initializes the OpenTelemetry TracerProvider, injects trace context into outbound HTTP requests, and records custom span events.

### Implementation (`main.go`)
```go
package main

import (
	"context"
	"fmt"
	"log"
	"net/http"
	"time"

	"go.opentelemetry.io/otel"
	"go.opentelemetry.io/otel/attribute"
	"go.opentelemetry.io/otel/exporters/stdout/stdouttrace"
	"go.opentelemetry.io/otel/propagation"
	"go.opentelemetry.io/otel/sdk/resource"
	sdktrace "go.opentelemetry.io/otel/sdk/trace"
	semconv "go.opentelemetry.io/otel/semconv/v1.24.0"
	"go.opentelemetry.io/otel/trace"
)

func initTracer() (*sdktrace.TracerProvider, error) {
	exporter, err := stdouttrace.New(stdouttrace.WithPrettyPrint())
	if err != nil {
		return nil, err
	}
	res := resource.NewWithAttributes(
		semconv.SchemaURL,
		semconv.ServiceName("payment-gateway"),
	)
	tp := sdktrace.NewTracerProvider(
		sdktrace.WithBatcher(exporter),
		sdktrace.WithResource(res),
	)
	otel.SetTracerProvider(tp)
	otel.SetTextMapPropagator(propagation.TraceContext{})
	return tp, nil
}

func handleCheckout(w http.ResponseWriter, r *http.Request) {
	tr := otel.Tracer("checkout-handler")
	ctx, span := tr.Start(r.Context(), "ProcessCheckout")
	defer span.End()

	span.SetAttributes(
		attribute.String("payment.method", "credit_card"),
		attribute.Float64("order.amount_usd", 99.50),
	)

	// Simulate work
	time.Sleep(100 * time.Millisecond)
	span.AddEvent("ChargeCaptured", trace.WithAttributes(attribute.String("gateway.tx_id", "tx_98765")))

	w.WriteHeader(http.StatusOK)
	fmt.Fprintf(w, "Checkout Complete! TraceID: %s\n", span.SpanContext().TraceID().String())
}

func main() {
	tp, err := initTracer()
	if err != nil {
		log.Fatal(err)
	}
	defer tp.Shutdown(context.Background())

	http.HandleFunc("/checkout", handleCheckout)
	log.Println("Starting service on :8080...")
	http.ListenAndServe(":8080", nil)
}
```

---

## Lab 5: Simulating Service Degradation and Chaos Injection

### Objective
Deploy a Chaos Mesh `NetworkChaos` experiment injecting 200ms of latency and 15% packet loss on an internal payment service to verify circuit breaker trip behavior.

### Implementation Manifest (`network-chaos.yaml`)
```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: payment-network-turbulence
  namespace: default
spec:
  action: delay
  mode: all
  selector:
    namespaces:
      - default
    labelSelectors:
      app: payments-service
  delay:
    latency: '200ms'
    jitter: '50ms'
    correlation: '50'
  loss:
    loss: '15'
    correlation: '25'
  duration: '5m'
  scheduler:
    cron: '@every 2h'
```

---

## Lab 6: Drafting a Complete Blameless Postmortem

### Objective
Author a comprehensive, blameless postmortem for a simulated Sev-1 payment outage, detailing timeline, root cause analysis via the 5 Whys, and SMART action items.

### Postmortem Document (`POSTMORTEM_INC-2026-104.md`)
```markdown
# Blameless Postmortem: INC-2026-104 Payment Checkout Failure

## Summary
- **Date:** 2026-10-18
- **Duration:** 42 minutes (14:12 UTC – 14:54 UTC)
- **Impact:** 18,400 checkout transactions failed (HTTP 500); estimated $240k delayed gross merchandise value.
- **Incident Commander:** Jane Doe (Staff SRE)
- **Communications Lead:** John Smith

## Timeline (UTC)
- **14:10:** Automated deployment of `payments:v2.4.1` completes via Argo CD.
- **14:12:** Multi-burn-rate alert fires: `PaymentServiceHighBurnRatePage` (14.4x burn rate, error rate 12%).
- **14:15:** Incident Commander declares Sev-1; opens incident bridge and creates `#inc-2026-104`.
- **14:22:** Ops Lead identifies connection pool exhaustion in database logs.
- **14:28:** Mitigation decided: rollback to `payments:v2.4.0` via `argocd app rollback`.
- **14:34:** Rollback completes; error rate drops to 0.1% baseline.
- **14:54:** 20 minutes of stability verified; Sev-1 resolved.

## Root Cause Analysis & The 5 Whys
1. *Why did checkouts fail?* All pods ran out of PostgreSQL connections.
2. *Why did pods run out of connections?* In `v2.4.1`, connection pool `max_open_conns` was accidentally increased from 10 to 100 per pod.
3. *Why was it increased?* An engineer intended to increase the worker thread pool and copy-pasted the configuration.
4. *Why did 50 pods connect simultaneously?* Pod autoscaling scaled to 50 replicas during peak, attempting to open 5,000 connections against a database sized for 1,000 connections.
5. *Why was this not caught before production?* Staging environments only run 2 replicas and lacked load tests that exercise connection limits.

## Action Items (SMART)
| Action Item | Type | Owner | Due Date | Ticket |
| :--- | :--- | :--- | :--- | :--- |
| Deploy PgBouncer connection pooler in front of PostgreSQL | Preventative | Infra Team | 2026-11-01 | INFRA-412 |
| Add Cel validation to Helm/Kustomize capping `max_open_conns <= 15` | Detective | Platform | 2026-10-25 | PLAT-890 |
| Add synthetic load testing stage to Argo Rollouts canary analysis | Preventative | Payments | 2026-11-15 | PAY-331 |
```
