# GCP Professional Cloud DevOps Engineer — Practice Exam 2

**Difficulty:** Professional / Advanced Production Scenarios  
**Format:** 15 multi-paragraph scenario questions · single answer · four options (A–D)  
**Time Allowed:** 75 minutes  
**Passing Target:** 12 / 15 (80%)

---

### Question 1 (Domain 3: Observability & Monitoring)
An SRE team monitors a critical payment gateway service. They have an SLO stating that 99.5% of successful requests must respond in less than 250 milliseconds over a 30-day rolling window. During peak hours, request volume increases fivefold. Why is measuring latency using an average (mean) considered an SRE antipattern, and what metric should be used instead?
- A. Average is preferred because it smooths out outliers; no change is needed.
- B. Averages hide long-tail latency spikes that affect significant numbers of users; percentiles (e.g. 95th, 99th, or 99.9th percentile) must be used to measure the true customer experience.
- C. SREs should only measure CPU utilization, not latency.
- D. Averages consume too much CPU in Cloud Monitoring.

---

### Question 2 (Domain 1: SRE Principles)
A multi-region web application on Compute Engine uses an external HTTPS Load Balancer with backends in `us-central1` and `europe-west1`. The SRE team wants to simulate the complete failure of the `us-central1` datacenter to verify that global Anycast routing automatically shifts traffic to `europe-west1` without dropping active sessions. What is the safest way to execute this Chaos Engineering game day test?
- A. Delete the `us-central1` VPC network.
- B. Drain traffic gracefully by setting the `us-central1` backend service capacity / max utilization to 0 in Cloud Load Balancing, observe latency and error rate SLIs, and then restore capacity.
- C. Terminate all Compute Engine instances via a bash script simultaneously.
- D. Change DNS records at the public registrar.

---

### Question 3 (Domain 2: CI/CD Pipelines)
A software delivery team needs to deploy an updated version of a backend microservice. They want to ensure zero downtime and immediate instant rollback capability in the event of an undetected bug, with zero dual-version traffic mixing. Which deployment pattern best matches this requirement?
- A. Rolling update with `maxUnavailable: 50%`
- B. Blue-Green deployment where the green environment is fully deployed and tested before traffic is switched atomically at the load balancer or service router.
- C. Recreate deployment (delete old pods, then create new pods).
- D. In-place binary replacement on running VMs.

---

### Question 4 (Domain 3: Observability & Monitoring)
A GKE cluster hosts multiple tenant applications across different namespaces. The operations team wants to collect custom Prometheus metrics from application pods without managing Prometheus server infrastructure, scraping agents, or storage backends manually. Which Google Cloud native feature should they enable?
- A. Google Cloud Managed Service for Prometheus (GMP) with `PodMonitoring` custom resources.
- B. Deploying an unmanaged single-replica Prometheus container on an `e2-micro` node.
- C. Writing metrics to text files on disk.
- D. Streaming logs to Cloud Storage.

---

### Question 5 (Domain 4: Incident Management)
During an incident, the Operations Lead discovers that an automated database vacuum process is causing high disk I/O, slowing down customer queries. The developer suggests killing the PostgreSQL process abruptly with `kill -9`. What should the Incident Commander consider before approving this action?
- A. Whether the developer has more seniority than the Operations Lead.
- B. The risk of database catalog corruption or extended recovery time (WAL replay), preferring a controlled cancellation of the vacuum job or graceful connection throttling instead.
- C. The time of day.
- D. Whether the database is running on Linux or Windows.

---

### Question 6 (Domain 5: Security & Governance)
A company must ensure that all Google Cloud storage buckets containing healthcare data comply with HIPAA audit retention. Which configuration ensures that Cloud Storage audit logs cannot be modified, tampered with, or deleted by any administrator for 7 years?
- A. Cloud Storage Object Lifecycle management deleting objects after 30 days.
- B. Exporting audit logs via a Log Sink to a dedicated, locked Cloud Storage bucket with a Bucket Retention Policy locked under Retention Policy Lock (WORM).
- C. Storing logs on developer workstations.
- D. Creating a weekly zip archive.

---

### Question 7 (Domain 2: CI/CD Pipelines)
An enterprise builds container images using Cloud Build. Security policy requires that all builds execute in an isolated network environment with no access to the public internet, but with access to private repositories in an on-premises datacenter via Cloud Interconnect. Which Cloud Build configuration is required?
- A. Default shared worker pool.
- B. Cloud Build Private Pools configured with private peering to the company VPC network.
- C. Running builds inside a public GKE cluster.
- D. Disabling Cloud Build and running builds over SSH.

---

### Question 8 (Domain 3: Observability & Monitoring)
An application running on Cloud Run occasionally experiences extreme latency spikes. An engineer inspects Cloud Logging and finds no error messages. Which Google Cloud tool allows profiling CPU consumption and memory allocations directly at the function/call-stack level with low overhead in production?
- A. Cloud Profiler
- B. Cloud Armor
- C. Cloud DNS
- D. Secret Manager

---

### Question 9 (Domain 1: SRE Principles)
An engineering manager calculates that their team spent 70% of their engineering hours last sprint manually patching VMs, responding to un-actionable alerts, and executing manual database cleanups. According to Google SRE operational guidelines, what problem does this team have?
- A. The team is over-performing on SRE metrics.
- B. The team is experiencing an unsustainable toil level exceeding the 50% SRE maximum threshold, leaving inadequate time for engineering projects that eliminate root causes.
- C. The team needs more managers.
- D. The team should stop using Google Cloud.

---

### Question 10 (Domain 4: Incident Management)
What is the function of the "Timeline" section in a standard Google SRE postmortem document?
- A. To list the vacation schedules of the engineers involved.
- B. To chronologically log every significant event, alert trigger, mitigation action, and status update from inception to resolution with exact UTC timestamps.
- C. To predict the company's stock price next quarter.
- D. To record the hardware manufacturing dates of servers.

---

### Question 11 (Domain 5: Security & Governance)
A microservice running on GKE needs to read customer records from a private BigQuery dataset. Which authentication method conforms to Google Cloud security best practices?
- A. Embedding an administrative API key in the application configuration file.
- B. Using GKE Workload Identity to bind the Kubernetes Service Account used by the pod to a Google Cloud Service Account that has `roles/bigquery.dataViewer` on that specific dataset.
- C. Making the BigQuery dataset publicly readable.
- D. Granting the GKE worker node service account `roles/bigquery.admin`.

---

### Question 12 (Domain 3: Observability & Monitoring)
When using Cloud Trace with microservices running on GKE, what HTTP header standard is used to propagate trace contexts between distributed services?
- A. `Cookie`
- B. W3C Trace Context (`traceparent` / `tracestate`)
- C. `User-Agent`
- D. `Authorization: Basic`

---

### Question 13 (Domain 2: CI/CD Pipelines)
A development team wants to implement a deployment strategy where 1% of live user traffic is mirrored to a new release candidate without returning the candidate's response to the client, allowing performance verification on real production traffic with zero user risk. What strategy is this?
- A. Canary deployment
- B. Blue-Green deployment
- C. Shadow / Traffic Mirroring deployment
- D. Rolling deployment

---

### Question 14 (Domain 1: SRE Principles)
If a service has an availability SLO of 99.95% over a 30-day period (43,200 total minutes), what is the maximum total allowed downtime during that 30-day window?
- A. 4.32 minutes
- B. 21.6 minutes
- C. 43.2 minutes
- D. 120 minutes

---

### Question 15 (Domain 4: Incident Management)
During an incident debrief, an executive asks why the postmortem does not name the specific developer whose configuration change caused the outage. What is the core SRE justification for blameless postmortems?
- A. To protect developers from being fired.
- B. Blame culture encourages concealment of mistakes, while a blameless culture fosters psychological safety, transparent reporting, and deep exploration of systemic failure modes.
- C. Because Google Cloud does not record user identities in audit logs.
- D. Because it is illegal to name employees in technical reports.

---

## Answer Key & Rationales

1. **B** — Averages obscure the experiences of unhappy tail-latency users. High-percentile SLIs (p95, p99) accurately capture latency degradation.
2. **B** — Gracefully setting backend capacity to 0 at the load balancer allows safe, controlled validation of global failover routing without destructive operations.
3. **B** — Blue-Green deployments provide isolated verification of the green environment followed by an instant atomic cutover with zero downtime and immediate rollback.
4. **A** — Google Cloud Managed Service for Prometheus (GMP) manages Prometheus scraping and storage automatically using Kubernetes `PodMonitoring` custom resources.
5. **B** — Responders must balance mitigation speed with the risk of compounding the disaster; abrupt process termination could trigger prolonged database recovery.
6. **B** — Log sinks routing to locked Cloud Storage buckets with Bucket Retention Lock ensure tamper-proof WORM compliance.
7. **B** — Cloud Build Private Pools peer with your VPC, enabling access to internal corporate subnets and Cloud Interconnect without exposing traffic publicly.
8. **A** — Cloud Profiler continuously profiles CPU and memory allocation call trees in production with negligible overhead ($< 1\%$).
9. **B** — Google SRE mandates that toil must not exceed 50% of an engineer's time; exceeding this indicates an operational crisis requiring structural engineering intervention.
10. **B** — The timeline provides an objective, chronological sequence of events used to analyze Time to Detect (TTD) and Time to Mitigate (TTM).
11. **B** — GKE Workload Identity enables fine-grained pod-level IAM authentication without mounting static service account keys.
12. **B** — Modern distributed tracing frameworks (OpenTelemetry) standardize on the W3C Trace Context standard (`traceparent`).
13. **C** — Traffic Mirroring (Shadowing) copies production traffic to the new service asynchronously without affecting live client responses.
14. **B** — $43,200 	ext{ minutes} 	imes (1 - 0.9995) = 43,200 	imes 0.0005 = 21.6 	ext{ minutes}$.
15. **B** — Blameless postmortems ensure engineers report incidents openly, allowing teams to identify and remediate systemic and automated vulnerabilities.
