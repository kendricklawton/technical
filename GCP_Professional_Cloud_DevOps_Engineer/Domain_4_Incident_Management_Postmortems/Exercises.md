# Domain 4: Incident Management & SRE Postmortems — Drills & Exercises

---

## Exercise 1: Incident Command System (ICS) Escalation & Role Assignment Drill

### Context
Your multi-region GKE payment processing service experiences a cascading database connection exhaustion event during Black Friday. Latency spikes from 45ms to 12,000ms, and 503 errors jump to 18%. You are the first responder.

### Terminal Drill: Declare and Organize Incident
```bash
# 1. Post to #incidents Slack / Chat via Webhook
curl -X POST -H 'Content-type: application/json'   --data '{"text":"🚨 [SEV-1] Incident Declared: Payment API 503 spike. War room link: https://meet.google.com/abc-defg-hij. Incident Commander: @k-henry"}'   https://chat.googleapis.com/v1/spaces/INCIDENTS/messages?key=$WEBHOOK_KEY

# 2. Assign ICS Roles in Incident Document
# Role Assignments:
# - Incident Commander (IC): Leads the response, delegates tasks, maintains state.
# - Operations Lead (OL): Executes technical commands and mitigations.
# - Communications Lead (CL): Interfaces with executive stakeholders and customers via status page.
```

### Mitigation Priority: Mitigate First, Debug Later
```bash
# Execute emergency mitigation: Shed non-critical traffic and scale read replicas
kubectl scale deployment/recommendation-engine -n prod --replicas=0
kubectl scale deployment/payment-api -n prod --replicas=150

# Verify Cloud SQL connection pool utilization
gcloud sql instances describe prod-payments-db   --project=fintech-prod   --format="json(settings.tier, state)"
```

---

## Exercise 2: Chaos Engineering Drill with Chaos Mesh / Litmus on GKE

### Objective
Simulate a pod failure and network latency injection in staging to test automatic failover and multi-burn-rate alerting.

```yaml
# chaos-network-delay.yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: gke-latency-injection
  namespace: staging
spec:
  action: delay
  mode: fixed
  value: '25%'
  delay:
    latency: '800ms'
    jitter: '100ms'
  selector:
    namespaces:
      - staging
    labelSelectors:
      'app': 'order-service'
  duration: '10m'
```

```bash
# Apply Chaos experiment
kubectl apply -f chaos-network-delay.yaml

# Monitor SLI response via MQL query in Cloud Monitoring
gcloud monitoring dashboards create --config-from-file=/tmp/chaos-dashboard.json
```

---

## Exercise 3: Postmortem Drafting & Blameless Root Cause Analysis (5 Whys)

### Scenario
An automated CI/CD pipeline deployed a faulty database migration script that locked the orders table for 42 minutes.

### 5 Whys Analysis Template
1. **Why did the payment service fail?**  
   *Because the orders database table was locked in exclusive mode.*
2. **Why was the orders table locked?**  
   *Because a column rename migration script executed `ALTER TABLE` without `CONCURRENTLY`.*
3. **Why did the migration execute without `CONCURRENTLY`?**  
   *Because the ORM auto-generated migration syntax was not reviewed for production locking behaviors.*
4. **Why was the locking migration not caught before production?**  
   *Because staging databases contain only 1,000 rows, where the lock lasted under 2ms, masking the table lock.*
5. **Why was there no synthetic data load test in staging?**  
   *Because load testing was excluded from staging pipelines to keep CI runs under 5 minutes.*

### Preventive Action Items (SMART)
- [P0] Add a CI linter check (`pg-lint`) to block non-concurrent index/table modifications in Cloud Build.
- [P1] Deploy a synthetic 10,000,000 row read/write staging environment for pre-flight migration validations.
- [P2] Configure Cloud SQL query lock timeout to auto-kill long-running lock statements after 10 seconds.
