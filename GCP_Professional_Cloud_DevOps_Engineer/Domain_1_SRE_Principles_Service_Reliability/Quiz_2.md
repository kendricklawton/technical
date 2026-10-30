# Domain 1: Applying SRE Principles to a Service — Quiz 2

**Exam:** Google Cloud Certified: Professional Cloud DevOps Engineer  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
You are defining latency Service Level Indicators (SLIs) for an interactive API. Which of the following is the most statistically sound metric to measure user satisfaction?
- A. The mean (average) latency of all requests over 24 hours.
- B. The 99th percentile ($P99$) latency of successful requests measured over a sliding 5-minute window.
- C. The minimum latency recorded by any user.
- D. The maximum latency recorded in the last 30 days.

---

### Question 2
Why does Google SRE caution against using the arithmetic mean (average) to evaluate system request latency?
- A. The arithmetic mean requires too much CPU to calculate.
- B. The arithmetic mean masks severe long-tail outliers; 5% of users could experience catastrophic 10-second delays while the average appears healthy at 50ms.
- C. Google Cloud Monitoring does not support average metrics.
- D. The mean cannot be represented in JSON.

---

### Question 3
An engineering team sets an internal SLO of $99.9\%$ for their microservice, while the legal department publishes an external customer SLA of $99.0\%$. Why is this discrepancy considered an SRE best practice?
- A. It gives the SRE team a 10x safety buffer to detect and remediate internal outages before any contractual SLA breach or customer penalty occurs.
- B. It is an error in calculation that should be corrected.
- C. It reduces Google Cloud billing costs.
- D. Cloud Load Balancing requires mismatched SLOs.

---

### Question 4
In Google Cloud Monitoring, which tool allows SREs to declaratively define custom services, calculate good-vs-total ratios, and visualize remaining error budgets over rolling windows?
- A. Google Cloud Deployment Manager
- B. Service Monitoring (SLO Monitoring)
- C. Cloud Trace Profiler
- D. Cloud NAT Gateway

---

### Question 5
When calculating capacity planning for an upcoming regional marketing campaign expected to drive $3\times$ baseline traffic, which SRE technique should be used to verify that the application and database tiers scale linearly without hitting saturation cliffs?
- A. Manual code review.
- B. Load and stress testing in a staging environment mirroring production data schemas and query profiles.
- C. Relying entirely on reactive autoscaling without testing.
- D. Disabling Cloud Armor rate limiting.

---

### Question 6
An organization implements an Error Budget Policy. During a major deployment, a database schema migration locks the primary table, consuming $80\%$ of the monthly error budget in 45 minutes. What is the appropriate immediate governance action?
- A. Fire the database administrator.
- B. Lower the SLO threshold for the remainder of the month.
- C. Restrict non-essential feature deployments, require all upcoming releases to undergo automated canary analysis, and prioritize database zero-downtime migration tooling.
- D. Re-architect the application in Python.

---

### Question 7
Which of the Four Golden Signals measures how close a system is to reaching its maximum theoretical capacity limit (such as connection pools, memory headroom, or disk I/O channels)?
- A. Latency
- B. Traffic
- C. Errors
- D. Saturation

---

### Question 8
You have an asynchronous message processing worker pool reading tasks from a Cloud Pub/Sub subscription. Which metric is the most effective SLI for measuring the health and performance of this asynchronous pipeline?
- A. Worker VM CPU utilization.
- B. The age of the oldest unacknowledged message (`subscription/oldest_unacked_message_age`).
- C. Total number of Google Cloud Service Accounts.
- D. Worker VM disk read operations.

---

### Question 9
In Chaos Engineering, what is the primary objective of running controlled chaos experiments (such as killing random GKE pods or injecting artificial network latency)?
- A. To test whether the security operations team is watching the monitors.
- B. To proactively uncover hidden architectural vulnerabilities, cascading failure modes, and recovery deficiencies before they cause real-world outages.
- C. To exhaust the error budget early in the month.
- D. To benchmark hardware CPU speeds.

---

### Question 10
Under what condition should a service's SLO be made more stringent (e.g. upgraded from $99.0\%$ to $99.9\%$)?
- A. When developers have extra free time.
- B. When users demand higher reliability and the business determines that the commercial value of increased uptime outweighs the slowed feature velocity and added infrastructure cost.
- C. Because 99.9% is always mandatory for all software.
- D. Whenever cloud compute instances are upgraded.

---

## Answer Key & Rationales

1. **B** — Percentiles (such as $P95$ or $P99$) accurately represent long-tail user experience without being diluted by high volumes of trivial fast requests.
2. **B** — Averages obscure tail latency; a fast 10ms average can easily hide the fact that the slowest 1% of users are experiencing 15-second timeouts.
3. **A** — Maintaining an internal SLO that is significantly tighter than the contractual SLA ensures teams receive alerts and remediate incidents before incurring legal or financial breach penalties.
4. **B** — Cloud Monitoring Service Monitoring provides first-class constructs for defining Request-based and Windows-based SLIs, SLOs, and visualizing error budget consumption.
5. **B** — Rigorous load and stress testing under realistic production conditions validates that autoscaling policies, database connection pools, and backend dependencies handle surges without bottlenecking.
6. **C** — When the error budget is severely depleted, the error budget policy dictates throttling feature delivery in favor of hardening migrations, resilience, and automated testing.
7. **D** — **Saturation** measures the fraction of the most constrained system resource in use (e.g., 90% memory used, 95% thread pool occupied).
8. **B** — For asynchronous queue processing, oldest unacknowledged message age measures queue delay and freshness directly reflecting pipeline SLA.
9. **B** — Chaos engineering proactively identifies weaknesses (e.g. missing timeouts, circuit breaker failures, missing retries) under safe, controlled conditions.
10. **B** — Upgrading an SLO requires balancing product velocity against the exponential engineering and infrastructure costs required to deliver additional nines of availability.
