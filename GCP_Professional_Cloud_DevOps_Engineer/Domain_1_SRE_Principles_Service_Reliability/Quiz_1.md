# Domain 1: Applying SRE Principles to a Service — Quiz 1

**Exam:** Google Cloud Certified: Professional Cloud DevOps Engineer  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
An engineering team is running a customer-facing e-commerce application on Google Kubernetes Engine (GKE). They define an SLI as "the percentage of successful HTTP requests returned with status code `< 500`". The team sets an SLO of $99.9\%$ over a 30-day rolling compliance window. In the past 30 days, the service received $100,000,000$ total valid requests. How many total failed requests are allowed before the error budget is completely exhausted?
- A. 1,000
- B. 10,000
- C. 100,000
- D. 1,000,000

---

### Question 2
An SRE team at a retail platform notices that on-call engineers receive pages at 2:00 AM whenever CPU utilization on Compute Engine instances briefly crosses 90% for 3 minutes, even though the application continues to return sub-50ms responses to all users with zero dropped requests. How should the team adjust their alerting strategy according to Google SRE best practices?
- A. Increase the CPU threshold to 95%.
- B. Eliminate CPU utilization alerts as paging triggers, and configure paging alerts based on Service Level Objective (SLO) error budget burn rates.
- C. Add an automated script to reboot instances when CPU exceeds 90%.
- D. Assign two engineers to on-call duty instead of one.

---

### Question 3
Under Google SRE Multi-Window Multi-Burn-Rate alerting guidelines, what is the purpose of requiring a short window (e.g. 5 minutes) in conjunction with a 1-hour long window when firing a page for a $14.4\times$ burn rate?
- A. To save disk space in Cloud Logging.
- B. To avoid sending false-positive alerts when a transient error spike has already resolved before the on-call engineer wakes up.
- C. To ensure the load balancer retries failed requests.
- D. Because Cloud Monitoring does not support 1-hour windows alone.

---

### Question 4
A development team frequently exhausts its 30-day error budget within the first week of every month due to unstable new feature deployments. The business executives demand that the SRE team relax the SLO from 99.9% to 99.0% so deployments can continue. What is the recommended SRE response according to Google SRE principles?
- A. Agree to relax the SLO immediately.
- B. Enforce the agreed-upon Error Budget Policy: halt feature deployments and redirect engineering effort toward reliability engineering, automated testing, and canary rollouts until the service stabilizes.
- C. Delete the monitoring metrics.
- D. Transfer on-call responsibilities to the product managers.

---

### Question 5
An SRE team spends 30 hours per week per engineer manually provisioning Cloud SQL user credentials, updating firewall rules via tickets, and restarting stuck worker pods. According to Google SRE standards, what is the maximum percentage of an SRE's time that should be spent on this type of operational work (toil)?
- A. 10%
- B. 25%
- C. 50%
- D. 80%

---

### Question 6
Which of the following activities is classified as **Toil** under Google SRE definition?
- A. Writing a reusable Terraform module to automate Cloud SQL provisioning.
- B. Manually executing `gcloud compute instances reset` every morning when a batch job hangs.
- C. Designing an architecture decision record for multi-region GKE failover.
- D. Writing an automated integration test suite in Go.

---

### Question 7
You need to measure user-perceived availability for a web application deployed behind an External HTTPS Load Balancer in Google Cloud. Which metric source provides the most accurate Service Level Indicator (SLI) reflecting actual end-user experience?
- A. CPU utilization metrics from GKE worker nodes.
- B. Request and response count metrics recorded at the Cloud Load Balancing layer (`loadbalancing.googleapis.com/https/request_count`).
- C. Disk write I/O metrics on persistent disks.
- D. Operating system syslog logs from Compute Engine VMs.

---

### Question 8
A service has an availability SLO of $99.95\%$ over a 30-day rolling window ($43,200\text{ minutes}$). What is the total allowable downtime in minutes during this 30-day period?
- A. 4.32 minutes
- B. 21.6 minutes
- C. 43.2 minutes
- D. 120 minutes

---

### Question 9
What is the primary difference between a Service Level Objective (SLO) and a Service Level Agreement (SLA)?
- A. An SLO is a legal contract with financial penalties; an SLA is an internal team objective.
- B. An SLO is an internal target used by SRE and development teams to make engineering and release decisions; an SLA is an external commercial commitment with financial or legal consequences if missed.
- C. An SLO is measured in seconds; an SLA is measured in percentage.
- D. Google Cloud does not support SLOs.

---

### Question 10
An e-commerce service experiences a $14.4\times$ burn rate against its 30-day error budget. If this burn rate continues uninterrupted, how long will it take for the service to consume $100\%$ of its monthly error budget?
- A. 1 hour
- B. 50 hours (approximately 2 days)
- C. 720 hours (30 days)
- D. 15 minutes

---

## Answer Key & Rationales

1. **C** — $100,000,000 \times (1 - 0.999) = 100,000,000 \times 0.001 = 100,000\text{ errors}$.
2. **B** — Alerting on infrastructure resource utilization (CPU) rather than customer-facing SLIs causes alert fatigue. SRE best practice is to page based on SLO error budget burn rates.
3. **B** — Pairing a long window with a short window ensures that if an error burst has already ended, the short-window burn rate drops to zero, resetting the alert and preventing unnecessary pages.
4. **B** — When error budgets are depleted, the Error Budget Policy mandates shifting focus from feature delivery to reliability, test automation, and architectural stabilization.
5. **C** — Google SRE strictly caps operational toil at 50% of an engineer's time, reserving the remaining 50%+ for engineering project work.
6. **B** — Toil is manual, repetitive, tactical work devoid of enduring value that scales linearly with service size (such as manual VM resets).
7. **B** — Measuring SLIs at the External HTTP(S) Load Balancer captures the true user-facing request success and latency before traffic hits internal pods.
8. **B** — $43,200 \times (1 - 0.9995) = 43,200 \times 0.0005 = 21.6\text{ minutes}$.
9. **B** — SLOs are internal engineering metrics that guide release velocity; SLAs are external business contracts with legal/financial repercussions.
10. **B** — $720\text{ hours} / 14.4 = 50\text{ hours}$. At a $14.4\times$ burn rate, the entire 30-day (720h) budget is consumed in 50 hours.
