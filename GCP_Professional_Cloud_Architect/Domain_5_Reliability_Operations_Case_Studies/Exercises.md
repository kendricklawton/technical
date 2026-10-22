# Domain 5: Reliability, Operations & Official Case Studies — Drills & Exercises

---

## Exercise 1: Multi-Region Load Balancer Failover Simulation (Cymbal Direct)

```bash
# 1. Inspect Global External HTTPS Load Balancer backend health
gcloud compute backend-services get-health web-frontend-backend \
  --global \
  --project=cymbal-direct-prod

# 2. Simulate regional outage by draining primary backend in us-central1
gcloud compute backend-services update-backend web-frontend-backend \
  --instance-group=frontend-mig-us \
  --instance-group-region=us-central1 \
  --max-utilization=0.0 \
  --capacity-scaler=0.0 \
  --global \
  --project=cymbal-direct-prod

# 3. Verify Anycast routing shifts 100% of global traffic to europe-west1
gcloud compute backend-services get-health web-frontend-backend --global
```

---

## Exercise 2: Agones Game Server Fleet Deployment on GKE (Mountkirk Games)

```yaml
# agones-fleet.yaml
apiVersion: "agones.dev/v1"
kind: Fleet
metadata:
  name: mountkirk-battleroyale-fleet
spec:
  replicas: 50
  template:
    spec:
      ports:
      - name: default
        containerPort: 7654
        protocol: UDP
      template:
        spec:
          containers:
          - name: dedicated-game-server
            image: "gcr.io/mountkirk-games/server:v2.4"
            resources:
              requests:
                memory: "1Gi"
                cpu: "1"
```

```bash
# Apply Agones Fleet manifest to GKE gaming cluster
kubectl apply -f agones-fleet.yaml

# Check allocated game servers
kubectl get gameservers
```

---

## Exercise 3: Cloud Spanner Multi-Region Instance Provisioning (EHR Healthcare / Cymbal)

```bash
# Provision a multi-region Cloud Spanner instance (nam-eur-asia1)
gcloud spanner instances create enterprise-spanner-prod \
  --config=nam-eur-asia1 \
  --description="Global Multi-Region Spanner Instance" \
  --processing-units=1000 \
  --project=my-architect-project
```
