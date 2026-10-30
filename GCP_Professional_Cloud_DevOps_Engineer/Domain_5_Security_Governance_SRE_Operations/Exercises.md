# Domain 5: Security, Governance & SRE Operations — Drills & Exercises

---

## Exercise 1: Configure Workload Identity Federation for GitHub Actions

```bash
# 1. Create Workload Identity Pool
gcloud iam workload-identity-pools create "github-pool"   --project="acme-prod"   --location="global"   --display-name="GitHub Actions Pool"

# 2. Create OIDC Provider
gcloud iam workload-identity-pools providers create-oidc "github-provider"   --project="acme-prod"   --location="global"   --workload-identity-pool="github-pool"   --display-name="GitHub Provider"   --issuer-uri="https://token.actions.githubusercontent.com"   --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository"   --attribute-condition="assertion.repository=='acme-org/checkout-service'"

# 3. Bind IAM Service Account Impersonation
gcloud iam service-accounts add-iam-policy-binding "deployer-sa@acme-prod.iam.gserviceaccount.com"   --project="acme-prod"   --role="roles/iam.workloadIdentityUser"   --member="principalSet://iam.googleapis.com/projects/PROJECT_NUMBER/locations/global/workloadIdentityPools/github-pool/attribute.repository/acme-org/checkout-service"
```

---

## Exercise 2: Binary Authorization Policy Enforcement on GKE

```yaml
# binary-auth-policy.yaml
admissionWhitelistPatterns:
- matchedPathExpression: gcr.io/google-containers/*
- matchedPathExpression: k8s.gcr.io/*
defaultAdmissionRule:
  enforcementMode: ENFORCING
  evaluationMode: REQUIRE_ATTESTATION
  requireAttestationsBy:
  - projects/acme-prod/attestors/build-attestor
globalPolicyEvaluationMode: ENABLE
```

```bash
# Apply Binary Authorization Policy
gcloud container binauthz policy import binary-auth-policy.yaml --project=acme-prod

# Verify GKE cluster has Binary Authorization enabled
gcloud container clusters update prod-cluster   --enable-binauthz   --zone=us-central1-a   --project=acme-prod
```

---

## Exercise 3: Secret Manager Secret Creation and GKE Mounting

```bash
# 1. Create a production database password secret
echo -n "super-secret-db-pass-2026!" | gcloud secrets create prod-db-password   --data-file=-   --replication-policy="automatic"   --project=acme-prod

# 2. Grant Secret Accessor role to the GKE Workload Identity service account
gcloud secrets add-iam-policy-binding prod-db-password   --member="serviceAccount:gke-workload-sa@acme-prod.iam.gserviceaccount.com"   --role="roles/secretmanager.secretAccessor"   --project=acme-prod
```
