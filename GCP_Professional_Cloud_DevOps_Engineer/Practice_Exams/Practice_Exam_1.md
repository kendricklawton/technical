# GCP Professional Cloud DevOps Engineer — Practice Exam 1

**Difficulty:** Professional / Advanced Production Scenarios  
**Format:** 15 multi-paragraph scenario questions · single answer · four options (A–D)  
**Time Allowed:** 75 minutes  
**Passing Target:** 12 / 15 (80%)

---

### Question 1 (Domain 1: SRE Principles)
An enterprise e-commerce platform hosted on Google Kubernetes Engine (GKE) operates with a monthly 30-day rolling SLO of 99.9% availability. The service receives approximately 100,000,000 requests per month. During a major promotional campaign, an unforeseen database deadlock causes 120,000 requests to fail with HTTP 500 status codes within a 2-hour period before an emergency pod restart restores service. The product manager wants to proceed with a major feature release scheduled for tomorrow morning. According to Google SRE best practices and error budget policies, what should the SRE team do?
- A. Permit the deployment because the remaining 29 days of the month will average out the error rate.
- B. Enforce the agreed-upon Error Budget Policy: freeze new non-emergency feature deployments because the 30-day error budget (100,000 allowed errors) has been exhausted ($120,000 > 100,000$), and require the development team to focus on database connection pooling and deadlock mitigations.
- C. Increase the SLO target to 99.99% to prevent future failures.
- D. Delete the Cloud Monitoring alert logs to reset the calculation.

---

### Question 2 (Domain 2: CI/CD Pipelines)
A software organization uses Cloud Build to compile and containerize a Go microservice. Currently, developers experience 18-minute build times because every build re-downloads all Go modules and recompiles dependencies from scratch in ephemeral Cloud Build worker instances. The engineering lead wants to reduce build durations to under 3 minutes while maintaining immutable, reproducible build outputs. Which approach should be implemented?
- A. Run builds manually on developer workstations.
- B. Utilize Kaniko cache within Cloud Build to cache intermediate container layers in Google Artifact Registry, and configure Go module caching using Cloud Storage volume caching in Cloud Build steps.
- C. Disable all unit tests in the Cloud Build YAML.
- D. Increase the Cloud Build worker machine type to `e2-micro`.

---

### Question 3 (Domain 1: SRE Principles)
An SRE team wants to implement an alerting strategy that balances fast detection of severe outages with protection against false alarms caused by transient error spikes. They implement Google's Multi-Window Multi-Burn-Rate alerting model. For an urgent page condition consuming 2% of the monthly error budget in 1 hour ($14.4	imes$ burn rate), what windows and conditions must be evaluated before dispatching a pager notification to on-call engineers?
- A. Only a 1-minute window evaluating if error rate exceeds 50%.
- B. A 1-hour long window evaluating if burn rate is $\ge 14.4	imes$ AND a 5-minute short window evaluating if burn rate is $\ge 14.4	imes$.
- C. A 30-day window evaluating total request counts.
- D. A CPU utilization alert set to 80% on compute instances.

---

### Question 4 (Domain 2: CI/CD Pipelines)
A fintech application deployed across multi-region GKE clusters requires a canary deployment strategy. The team uses Google Cloud Deploy. They want 10% of user traffic to be directed to the new release candidate, followed by automated verification against Prometheus error rate metrics, before promoting to 100%. If error rates exceed 0.1%, the deployment must roll back automatically. How should this pipeline be configured?
- A. Update the Kubernetes service selector manually via kubectl.
- B. Define a progressive delivery strategy in Cloud Deploy using a `canary` phase with a 10% weight percentage, configure Skaffold to render manifests with Gateway API / Service Mesh routing, and link automated verification hooks.
- C. Deploy two separate GKE clusters and delete the old cluster after 10 minutes.
- D. Use a single deployment and change the replica count from 1 to 2.

---

### Question 5 (Domain 3: Observability)
A platform engineer is writing a Monitoring Query Language (MQL) query in Google Cloud Monitoring to compute the ratio of successful HTTP requests to total HTTP requests for an external HTTPS load balancer over a 30-day window. Which MQL pattern correctly calculates this availability SLI?
- A. `fetch gce_instance | metric 'compute.googleapis.com/instance/cpu/utilization' | filter utilization > 0.8`
- B. `fetch https_lb_rule | metric 'loadbalancing.googleapis.com/https/request_count' | group_by [response_code_class], [count: count()] | filter response_code_class != 500`
- C. Combining `fetch https_lb_rule` with `good_total_ratio` comparing non-5xx responses against total valid request volume.
- D. `fetch logs | grep "error"`

---

### Question 6 (Domain 5: Security & Governance)
A company requires that all containers deployed to production GKE clusters be cryptographically signed by an automated CI/CD pipeline and validated against a Cloud KMS key. Any unsigned image must be rejected at admission time. What combination of services achieves this?
- A. Cloud Armor and VPC Service Controls.
- B. Binary Authorization, Container Analysis (Artifact Analysis), and Cloud KMS.
- C. Cloud NAT and Cloud DNS.
- D. Cloud Run and Compute Engine.

---

### Question 7 (Domain 4: Incident Management)
During an incident where the payment gateway returns HTTP 504 timeouts, the Incident Commander observes that the team is spending 30 minutes reading stack traces in Cloud Logging while thousands of customer transactions continue to fail. What directive should the Incident Commander issue?
- A. "Keep reading logs until we find the exact bug in the code."
- B. "Halt debugging on production; execute immediate mitigation by rolling back the deployment to the last known healthy release or rerouting traffic to the backup region."
- C. "File a bug ticket in Jira for next sprint."
- D. "Shut down the entire payment service permanently."

---

### Question 8 (Domain 2: CI/CD Pipelines)
An enterprise security team mandates that developers cannot deploy directly to GKE production clusters from their laptops. All production deployments must originate from a central Git repository following GitOps principles, with automatic synchronization and drift correction. Which Google Cloud native tool provides this continuous configuration reconciliation?
- A. Cloud Composer
- B. Config Sync (part of Google Kubernetes Engine Enterprise / Anthos Config Management)
- C. Cloud Functions
- D. Compute Engine Startup Scripts

---

### Question 9 (Domain 3: Observability)
An application consists of 20 microservices communicating via gRPC. Users report sporadic 5-second delays on the checkout page, but aggregate CPU and memory metrics on all clusters appear normal. Which Google Cloud observability tool is specifically designed to isolate which microservice in the distributed call graph is causing the latency spike?
- A. Cloud Billing Reports
- B. Cloud Trace (Distributed Tracing with OpenTelemetry)
- C. Cloud Storage Insights
- D. Cloud Deployment Manager

---

### Question 10 (Domain 5: Security & Governance)
A GitHub Actions workflow needs to push container images to Google Artifact Registry in project `my-prod-project`. Security policy forbids exporting or storing JSON service account keys in GitHub Secrets. How should authentication be established?
- A. Commit the private key into a private Git repo.
- B. Configure Workload Identity Federation with Google Cloud IAM and the GitHub OIDC provider, allowing the GitHub runner to assume a minimally privileged Google Service Account via short-lived federated tokens.
- C. Use a public bucket in Cloud Storage.
- D. Grant GitHub the Organization Administrator role.

---

### Question 11 (Domain 1: SRE Principles)
What is the primary characteristic that distinguishes "Toil" from legitimate operational engineering work in Google SRE?
- A. Toil is work that involves writing Python code.
- B. Toil is manual, repetitive, automatable, tactical work devoid of enduring value that scales linearly as the service grows.
- C. Toil is work done only by interns.
- D. Toil is any task that takes longer than 10 minutes.

---

### Question 12 (Domain 4: Incident Management)
An engineer participates in a postmortem review meeting. A team member asks: "Who forgot to run the database migration before deploying the code?" How should this question be reframed to maintain a blameless culture?
- A. "Why wasn't the engineer disciplined for skipping the migration step?"
- B. "What technical safeguards and automated deployment checks were missing that allowed a release to proceed without verifying schema compatibility?"
- C. "Who approved the pull request?"
- D. "Why was the developer working on a Friday?"

---

### Question 13 (Domain 3: Observability)
A high-volume logging service produces 50 TB of debug logs daily in Cloud Logging, resulting in unexpectedly high ingestion costs. The team only needs debug logs during active troubleshooting sessions. What Cloud Logging feature allows filtering out these high-volume logs at ingestion time while allowing production error logs to be ingested normally?
- A. Cloud Storage Lifecycle rules.
- B. Log Router Exclusion Filters on the `_Default` sink.
- C. Deleting the GKE namespace.
- D. Disabling Cloud Audit Logs.

---

### Question 14 (Domain 5: Security & Governance)
An organization wants to guarantee that database passwords used by Cloud Run services are rotated every 90 days without requiring code changes or service redeployments. What architecture fulfills this?
- A. Hardcode passwords in environment variables.
- B. Store passwords in Google Secret Manager, configure automatic rotation using Pub/Sub and a Cloud Function, and have the Cloud Run service mount the secret or retrieve the latest secret version dynamically.
- C. Email the new password to the operations team every quarter.
- D. Store passwords in a public Cloud Storage bucket.

---

### Question 15 (Domain 2: CI/CD Pipelines)
When deploying microservices to GKE using Skaffold and Cloud Deploy, what is the role of Skaffold in the delivery pipeline?
- A. Skaffold replaces Kubernetes master nodes.
- B. Skaffold provides declarative build, tag, and manifest rendering orchestration across different environments (dev, staging, prod) using profiles.
- C. Skaffold is a proprietary database engine.
- D. Skaffold acts as an external DNS provider.

---

## Answer Key & Rationales

1. **B** — The monthly budget is $100,000,000 	imes (1 - 0.999) = 100,000$ errors. With 120,000 errors occurred, the error budget is exhausted ($< 0\%$). The policy dictates halting feature releases to prioritize stability.
2. **B** — Kaniko layer caching in Artifact Registry and Cloud Storage dependency caching dramatically accelerates container build pipelines without sacrificing immutability.
3. **B** — Google's Multi-Window Multi-Burn-Rate alerting requires satisfying both a long window (to detect real budget drain) and a short window (to verify the incident is still active).
4. **B** — Cloud Deploy supports declarative canary deployments and automated verification hooks with Gateway API / Service Mesh routing.
5. **C** — Computing availability SLIs requires evaluating the ratio of good requests (non-5xx) to total valid requests via `good_total_ratio`.
6. **B** — Binary Authorization validates digital signatures created by Container Analysis attestors and KMS keys before allowing pod admission on GKE.
7. **B** — SRE incident response prioritizes *mitigating user impact first* (via rollback or traffic shift) before spending time debugging code on production.
8. **B** — Config Sync provides continuous, declarative GitOps reconciliation, pulling configurations from Git and remediating drift in GKE clusters.
9. **B** — Cloud Trace collects distributed telemetry spans across microservices, pinpointing exact RPC bottlenecks in distributed architectures.
10. **B** — Workload Identity Federation allows external CI systems like GitHub Actions to authenticate via short-lived OIDC tokens without storing static JSON keys.
11. **B** — Toil is manual, repetitive, devoid of enduring value, and scales linearly with service footprint. SRE limits toil to $\le 50\%$ of engineer time.
12. **B** — Blameless reviews look at systemic and architectural flaws (e.g. lack of automated pre-flight checks) rather than assigning human blame.
13. **B** — Log Router exclusion filters drop unnecessary log entries at ingestion, preventing ingestion costs while preserving critical error and audit logs.
14. **B** — Secret Manager automated rotation uses Pub/Sub and Cloud Functions to rotate credentials cleanly, decoupling secrets from application containers.
15. **B** — Skaffold provides standard build and manifest hydration workflows across dev, staging, and production environments for Cloud Deploy.
