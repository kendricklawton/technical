# Domain 4: Analyzing and Optimizing Technical and Business Processes — Drills & Exercises

---

## Exercise 1: Configure Cloud Storage Lifecycle Transition Policy

```json
// lifecycle.json
{
  "rule": [
    {
      "action": {"type": "SetStorageClass", "storageClass": "NEARLINE"},
      "condition": {"age": 30, "matchesStorageClass": ["STANDARD"]}
    },
    {
      "action": {"type": "SetStorageClass", "storageClass": "COLDLINE"},
      "condition": {"age": 90, "matchesStorageClass": ["NEARLINE"]}
    },
    {
      "action": {"type": "SetStorageClass", "storageClass": "ARCHIVE"},
      "condition": {"age": 365, "matchesStorageClass": ["COLDLINE"]}
    },
    {
      "action": {"type": "Delete"},
      "condition": {"age": 2555}
    }
  ]
}
```

```bash
# Apply the lifecycle policy to an enterprise data lake bucket
gcloud storage buckets update gs://enterprise-datalake-prod \
  --lifecycle-file=lifecycle.json
```

---

## Exercise 2: Automate VM Downsizing with Active Assist Recommender

```bash
# 1. List VM rightsizing recommendations in a project
gcloud recommender recommendations list \
  --recommender=google.compute.instance.MachineTypeRecommender \
  --project=my-architect-project \
  --location=us-central1-a \
  --format="table(name, description, stateInfo.state)"

# 2. Mark recommendation as claimed and apply via gcloud
gcloud compute instances stop worker-vm-01 --zone=us-central1-a
gcloud compute instances set-machine-type worker-vm-01 \
  --machine-type=e2-standard-2 \
  --zone=us-central1-a
gcloud compute instances start worker-vm-01 --zone=us-central1-a
```

---

## Exercise 3: Storage Transfer Service Job for AWS S3 Migration

```bash
# Create an automated daily synchronization job from AWS S3 to Google Cloud Storage
gcloud transfer jobs create s3://my-aws-media-bucket gs://my-gcp-media-bucket \
  --name="s3-to-gcs-sync" \
  --source-creds-file=/tmp/aws_creds.json \
  --schedule-start-date=2026-11-01 \
  --description="Daily automated S3 to GCS asset migration"
```
