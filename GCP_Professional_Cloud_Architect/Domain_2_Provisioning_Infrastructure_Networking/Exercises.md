# Domain 2: Managing and Provisioning Infrastructure & Networking — Drills & Exercises

---

## Exercise 1: Provision a Shared VPC Host and Attach Service Projects

```bash
# 1. Enable Shared VPC in the Host Project
gcloud compute shared-vpc enable host-project-123

# 2. Attach a Service Project to the Shared VPC
gcloud compute shared-vpc associated-projects add service-project-456 \
  --host-project=host-project-123

# 3. Grant subnet-level access to Service Project developers
gcloud compute networks subnets add-iam-policy-binding prod-subnet-us \
  --project=host-project-123 \
  --region=us-central1 \
  --member="serviceAccount:service-project-456-sa@service-project-456.iam.gserviceaccount.com" \
  --role="roles/compute.networkUser"
```

---

## Exercise 2: Configure Cloud HA VPN with Dynamic BGP Routing

```bash
# 1. Create an HA VPN Gateway (allocates two public interfaces)
gcloud compute vpn-gateways create ha-vpn-gw \
  --network=corp-vpc \
  --region=us-central1 \
  --project=my-architect-project

# 2. Create Cloud Router for dynamic BGP peering
gcloud compute routers create corp-cloud-router \
  --network=corp-vpc \
  --region=us-central1 \
  --asn=65001 \
  --project=my-architect-project

# 3. Configure External VPN Gateway representing on-premises device
gcloud compute external-vpn-gateways create onprem-peer-gw \
  --interfaces 0=203.0.113.1,1=203.0.113.2 \
  --project=my-architect-project
```

---

## Exercise 3: Deploy an External HTTPS Load Balancer with Cloud Armor WAF

```bash
# 1. Create a Cloud Armor security policy with rate limiting
gcloud compute security-policies create edge-waf-policy \
  --description="Edge security policy with rate limiting and OWASP rules"

# 2. Add a rate limiting rule (limit 100 requests per minute per IP)
gcloud compute security-policies rules create 1000 \
  --security-policy=edge-waf-policy \
  --expression="true" \
  --action="rate-based-ban" \
  --rate-limit-threshold-count=100 \
  --rate-limit-threshold-interval-sec=60 \
  --ban-duration-sec=300 \
  --conform-action="allow" \
  --exceed-action="deny(429)"

# 3. Attach security policy to backend service
gcloud compute backend-services update web-backend-service \
  --security-policy=edge-waf-policy \
  --global
```
