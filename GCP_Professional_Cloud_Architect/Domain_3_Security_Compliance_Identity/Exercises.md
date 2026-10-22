# Domain 3: Designing for Security, Identity, and Compliance — Drills & Exercises

---

## Exercise 1: Establish a VPC Service Controls Perimeter

```bash
# 1. Create an Access Policy at the Organization level
gcloud access-context-manager policies create \
  --organization=ORGANIZATION_ID \
  --title="Corporate Access Policy"

# 2. Create an Access Level specifying approved corporate IP ranges
gcloud access-context-manager levels create corporate_office \
  --policy=POLICY_ID \
  --title="Corporate Office IPs" \
  --basic-level-spec=/tmp/corp_ip_rule.yaml

# 3. Create a Service Perimeter protecting BigQuery and Cloud Storage
gcloud access-context-manager perimeters create secure_data_perimeter \
  --policy=POLICY_ID \
  --title="Secure Data Perimeter" \
  --resources="projects/PROJECT_NUMBER" \
  --restricted-services="bigquery.googleapis.com,storage.googleapis.com"
```

---

## Exercise 2: Customer-Managed Encryption Keys (CMEK) with Cloud KMS

```bash
# 1. Create a KMS Key Ring and asymmetric/symmetric key
gcloud kms keyrings create prod-keyring \
  --location=us-central1 \
  --project=my-security-project

gcloud kms keys create data-lake-key \
  --location=us-central1 \
  --keyring=prod-keyring \
  --purpose=encryption \
  --rotation-period=7776000s \
  --next-rotation-time=2026-12-01T00:00:00Z

# 2. Grant Cloud Storage Service Agent access to the key
STORAGE_SA=$(gcloud storage service-agent --project=my-security-project)

gcloud kms keys add-iam-policy-binding data-lake-key \
  --location=us-central1 \
  --keyring=prod-keyring \
  --member="serviceAccount:${STORAGE_SA}" \
  --role="roles/cloudkms.cryptoKeyEncrypterDecrypter"

# 3. Create a Cloud Storage bucket encrypted with CMEK
gcloud storage buckets create gs://secure-vault-bucket \
  --location=us-central1 \
  --default-encryption-key=projects/my-security-project/locations/us-central1/keyRings/prod-keyring/cryptoKeys/data-lake-key
```

---

## Exercise 3: Masking Sensitive PII via Cloud Data Loss Prevention (DLP)

```bash
# Inspect and de-identify a string containing SSN and credit card data
gcloud dlp text deidentify \
  --content="Customer John Doe with SSN 000-12-3456 and CC 4111-1111-1111-1111" \
  --info-types="US_SOCIAL_SECURITY_NUMBER,CREDIT_CARD_NUMBER" \
  --mask-character="*" \
  --number-to-mask=0
```
