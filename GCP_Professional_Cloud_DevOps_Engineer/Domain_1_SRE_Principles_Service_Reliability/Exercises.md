# Domain 1: Applying SRE Principles to a Service — Performance Labs & Drills

---

## Lab 1: Define and Deploy an SLO via Cloud Monitoring API

### Objective
Create an Availability SLO for an external HTTP service using the Cloud Monitoring Service Monitoring API.
- Service Name: `checkout-service`
- SLO: 99.9% availability over a rolling 30-day compliance period
- SLI: Ratio of valid HTTP requests returning non-5xx response codes.

```bash
# 1. Create a custom service definition in Cloud Monitoring
gcloud monitoring services create \
  --service-id="checkout-service" \
  --display-name="Checkout Microservice"

# 2. Define the SLO JSON payload
cat << 'EOF' > /tmp/availability-slo.json
{
  "displayName": "99.9% - Availability - Rolling 30 Days",
  "goal": 0.999,
  "rollingPeriod": "2592000s",
  "serviceLevelIndicator": {
    "requestBased": {
      "goodTotalRatio": {
        "goodServiceFilter": "metric.type=\"loadbalancing.googleapis.com/https/request_count\" resource.type=\"https_lb_rule\" metric.labels.response_code_class=\"200\"",
        "totalServiceFilter": "metric.type=\"loadbalancing.googleapis.com/https/request_count\" resource.type=\"https_lb_rule\""
      }
    }
  }
}
EOF

# 3. Apply the SLO to the service
gcloud monitoring services slos create \
  --service="checkout-service" \
  --config-from-file=/tmp/availability-slo.json
```

---

## Lab 2: Configure a Multi-Burn-Rate Alerting Policy

### Objective
Configure a Cloud Monitoring Alerting Policy that triggers when a $14.4\times$ burn rate is detected over a 1-hour window and confirmed by a 5-minute short window.

```bash
cat << 'EOF' > /tmp/burn-rate-alert.json
{
  "displayName": "Urgent: 14.4x Burn Rate on Checkout Service (1h / 5m)",
  "combiner": "AND",
  "conditions": [
    {
      "displayName": "14.4x burn rate over 1h",
      "conditionThreshold": {
        "filter": "select_slo_burn_rate(\"projects/${PROJECT_ID}/services/checkout-service/serviceLevelObjectives/availability-slo\", 3600s)",
        "comparison": "COMPARISON_GT",
        "thresholdValue": 14.4,
        "duration": "0s"
      }
    },
    {
      "displayName": "14.4x burn rate over 5m",
      "conditionThreshold": {
        "filter": "select_slo_burn_rate(\"projects/${PROJECT_ID}/services/checkout-service/serviceLevelObjectives/availability-slo\", 300s)",
        "comparison": "COMPARISON_GT",
        "thresholdValue": 14.4,
        "duration": "0s"
      }
    }
  ],
  "notificationChannels": [
    "projects/${PROJECT_ID}/notificationChannels/pagerduty-urgent"
  ],
  "documentation": {
    "content": "The checkout service has consumed 2% of its 30-day error budget in the last hour. Runbook: https://wiki.corp/ops/checkout-latency",
    "mimeType": "text/markdown"
  }
}
EOF

gcloud alpha monitoring policies create --policy-from-file=/tmp/burn-rate-alert.json
```

---

## Lab 3: Toil Accounting & Capacity Planning Analysis

### Objective
Calculate operational toil and project compute capacity requirements for an upcoming 5x traffic event.

```python
# Toil calculation script: verify 50% threshold
def audit_toil(team_hours_per_week, toil_hours_per_week):
    toil_percentage = (toil_hours_per_week / team_hours_per_week) * 100
    print(f"Current Toil Percentage: {toil_percentage:.1f}%")
    if toil_percentage > 50.0:
        excess_hours = toil_hours_per_week - (team_hours_per_week * 0.5)
        print(f"ALERT: Team exceeds 50% toil limit by {excess_hours:.1f} hours/week! Initiate automation projects.")
    else:
        print("Toil within safe SRE operating budget.")

audit_toil(team_hours_per_week=200, toil_hours_per_week=125)
```
