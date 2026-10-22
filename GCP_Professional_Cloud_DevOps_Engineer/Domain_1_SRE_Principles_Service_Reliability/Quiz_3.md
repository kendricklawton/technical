# Domain 1: Applying SRE Principles to a Service — Quiz 3

**Exam:** Google Cloud Certified: Professional Cloud DevOps Engineer  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
An SRE team is designing an alert for a $6\times$ burn rate against a $99.9\%$ availability SLO over a 30-day compliance window ($720\text{ hours}$). What percentage of the total error budget is consumed if a $6\times$ burn rate persists for 6 hours?
- A. 1%
- B. 5%
- C. 15%
- D. 50%

---

### Question 2
When implementing Service Level Objectives in Google Cloud Monitoring, what is the fundamental difference between a **Request-based SLI** and a **Windows-based SLI**?
- A. Request-based calculates the ratio of good requests to total valid requests; Windows-based calculates the percentage of time intervals (e.g. 1-minute or 5-minute windows) where the service met an operational quality threshold.
- B. Request-based is only for Google Compute Engine; Windows-based is only for GKE.
- C. Windows-based requires the Microsoft Windows operating system.
- D. There is no difference.

---

### Question 3
A high-throughput API processes $20,000$ requests per second. The team sets an SLO that $99.9\%$ of requests must complete in $< 100\text{ ms}$. If the application returns HTTP 500 errors in $2\text{ ms}$, how should the latency SLI filter be structured to avoid skewing the latency measurement with fast errors?
- A. Calculate latency across all requests regardless of status code.
- B. Filter the latency SLI calculation to include ONLY successful requests (e.g. HTTP status $< 500$).
- C. Delete the latency SLO and keep only availability.
- D. Add artificial delay to error responses.

---

### Question 4
In the Google SRE framework, what is the concept of a **Reliability Tax**?
- A. A federal corporate tax on high-uptime cloud infrastructure.
- B. The inevitable engineering and velocity cost paid to build redundancy, automated failover, and fault-tolerant architecture to achieve higher levels of availability.
- C. The cost of Google Cloud Support Premier tier.
- D. A fee charged by Cloud Monitoring for custom metrics.

---

### Question 5
An organization has an application deployed across multiple regions in Google Cloud. An unexpected network partition isolates `us-east1`. What SRE design pattern ensures that clients in `us-east1` are gracefully routed to `us-central1` without operator intervention?
- A. Manual DNS update in Cloud DNS.
- B. Global External HTTPS Load Balancing configured with multi-region backend services and automated health checking.
- C. Rebooting worker nodes in `us-east1`.
- D. Increasing the GKE pod replicas.

---

### Question 6
Which of the following metrics best indicates **Errors** in an asynchronous data processing pipeline using Cloud Dataflow?
- A. Worker CPU utilization.
- B. Total bytes written to Cloud Storage.
- C. Number of failed work items or unhandled exceptions captured in Error Reporting and dead-letter topics.
- D. Total number of Cloud Logging sinks.

---

### Question 7
Under what circumstances is an SRE justified in turning off automated alerts during an ongoing major incident?
- A. Never; all alerts must stay on at all times.
- B. When an established Incident Commander explicitly decides to silence redundant cascading alerts (e.g. via Alertmanager or Cloud Monitoring silences) to reduce noise while the incident response team executes mitigation.
- C. When the on-call engineer wants to go to sleep.
- D. Because alerts consume too much network bandwidth.

---

### Question 8
What is the primary danger of setting an SLO that is excessively high and unnecessarily strict for a non-critical internal dashboard tool (e.g. demanding 99.99% availability)?
- A. Cloud Monitoring will crash.
- B. It severely restricts feature release velocity, forces unnecessary engineering spending on multi-region active-active architectures, and provides zero tangible user benefit.
- C. Cloud Load Balancing requires 99.99% for all services.
- D. It prevents developers from using Python.

---

### Question 9
In Google Cloud Monitoring, what query language is used to express complex statistical metric transformations, ratios, and multi-window burn rate calculations?
- A. SQL
- B. Monitoring Query Language (MQL)
- C. Python
- D. Bash

---

### Question 10
When calculating an error budget over a rolling 30-day window, how does the passage of time affect past outages?
- A. Past outages remain in the calculation forever.
- B. Once an outage moves beyond the 30-day rolling horizon (e.g. on day 31), its failed events fall out of the evaluation window, replenishing the error budget.
- C. The error budget resets to 0% at midnight every Sunday.
- D. Google Cloud automatically bills the team for used budget.

---

## Answer Key & Rationales

1. **B** — A $6\times$ burn rate consumes $6 / 720 = 1/120$ of the monthly budget per hour. Over 6 hours, it consumes $6 \times (6 / 720) = 36 / 720 = 5.0\%$ of the budget.
2. **A** — Request-based SLIs evaluate individual request events; Windows-based SLIs evaluate whether discrete time intervals (e.g. minutes) met the required health threshold.
3. **B** — Failed requests often terminate abnormally fast; calculating latency across failed requests creates misleadingly optimistic latency percentiles. Latency SLIs must filter for successful responses.
4. **B** — Every additional "nine" of reliability incurs an exponential reliability tax in terms of engineering complexity, infrastructure redundancy, and release speed constraints.
5. **B** — Google Cloud Global External HTTP(S) Load Balancing provides single anycast IP routing and seamlessly redirects traffic to healthy backends in alternative regions when health checks fail.
6. **C** — For batch and streaming Dataflow jobs, error metrics measure unhandled job exceptions, element drop rates, and dead-letter queue records.
7. **B** — Suppressing or silencing redundant dependent alerts during active incident mitigation prevents alert floods and cognitive overload for responders.
8. **B** — Over-specifying SLOs wastes valuable engineering resources building complex high-availability infrastructure for services that do not require it.
9. **B** — Monitoring Query Language (MQL) is Google Cloud Monitoring's purpose-built query language for evaluating complex time series transformations and SLO ratios.
10. **B** — In a rolling window model, bad events from an outage age out as time progresses past the 30-day window length, dynamically recovering the error budget.
