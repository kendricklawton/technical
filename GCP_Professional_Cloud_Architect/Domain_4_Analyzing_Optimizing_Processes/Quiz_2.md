# Domain 4: Analyzing and Optimizing Technical and Business Processes — Quiz 2

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
A retail company expects unpredictable, dynamic workload changes over the next 12 months as they migrate services between Compute Engine, Cloud SQL, and Google Kubernetes Engine across multiple regions. The CTO wants to commit to a baseline dollar spend to receive discounts across ALL of these services flexibly. Which discount program fits this requirement?
- A. Resource-Based Committed Use Discounts
- B. Spend-Based Committed Use Discounts (Flexible CUDs)
- C. Spot VMs
- D. Free Tier

---

### Question 2
An enterprise operates hundreds of Compute Engine VMs. An architect reviews Google Cloud Active Assist and discovers that 40 VMs have an average CPU utilization of less than 3% over the last 30 days. Which Active Assist tool provides automated recommendations to downsize or terminate these idle instances?
- A. Security Health Analytics
- B. Compute Engine Sizing Recommender / Idle VM Recommender
- C. Cloud Profiler
- D. Cloud Trace

---

### Question 3
An architect is designing a CI/CD pipeline using Google Cloud Deploy and Skaffold. The team wants to ensure that 5% of user traffic is initially routed to a new microservice version in production, and if error rates remain below 0.01% for 30 minutes, traffic is automatically incremented to 25%, 50%, and 100%. What deployment methodology is being described?
- A. Big Bang deployment
- B. Progressive Canary Deployment
- C. Recreate deployment
- D. Disaster recovery failover

---

### Question 4
A SaaS company wants to export detailed Google Cloud billing data to perform deep-dive SQL queries, identify cost trends by department tag, and build custom Looker Studio dashboards. What is the recommended practice for analyzing Google Cloud billing data?
- A. Download CSV invoices manually at the end of each month.
- B. Enable Cloud Billing export to BigQuery (standard and detailed usage cost export).
- C. Query the Cloud Logging API.
- D. Review VM serial port logs.

---

### Question 5
A company is refactoring an on-premises monolithic Java application into microservices deployed on Google Cloud. The team wants to identify which classes and methods consume the most CPU and memory in production without introducing noticeable latency overhead. Which observability tool should they use?
- A. Cloud Profiler
- B. Cloud NAT
- C. Cloud DNS
- D. VPC Flow Logs

---

### Question 6
An organization wants to run continuous integration tests for a microservice repository. Each build spins up 20 containerized test runners that execute for 8 minutes and then terminate. What compute model provides the lowest cost for these ephemeral CI test runners?
- A. Bare Metal Compute Engine instances
- B. Cloud Build default pools or GKE node pools backed by Spot VMs
- C. 3-Year Committed Use Discounts
- D. VMware Engine

---

### Question 7
A company is migrating an on-premises MySQL database to Cloud SQL. The source database is 8 Terabytes in size. The network link between on-premises and Google Cloud has a high latency of 80ms. What feature of Database Migration Service accelerates initial data dumping and continuous replication?
- A. Single-threaded mysqldump
- B. Serverless, highly parallel initial data dump combined with native binlog CDC replication over Private Services Access or Cloud VPN
- C. Physical tape shipping
- D. Emailing SQL scripts

---

### Question 8
When optimizing BigQuery storage costs, what automatic feature reduces the price of storage by 50% for tables or partitions that have not been modified for 90 consecutive days?
- A. BigQuery Long-Term Storage pricing
- B. Coldline discount
- C. Table deletion
- D. Spot BigQuery

---

### Question 9
An enterprise has multiple development teams deploying infrastructure using Terraform. The teams frequently experience out-of-band configuration drift when developers modify cloud resources directly through the Google Cloud Console. How should the platform engineering team enforce Infrastructure as Code consistency?
- A. Delete all Google Cloud projects.
- B. Implement automated drift detection pipelines in Cloud Build, and restrict IAM permissions so developers have read-only access to console resources while write permissions are granted exclusively to the CI/CD service account.
- C. Ask developers to promise not to use the console.
- D. Disconnect the corporate network.

---

### Question 10
A media platform generates 100,000 video thumbnails per day. The thumbnails are accessed heavily during the first 14 days after publication, but access drops by 99% after two weeks. The files must remain instantly available if requested. How should the storage bucket be configured?
- A. Store in Archive class from day one.
- B. Store in Standard class initially, with an Object Lifecycle Management rule that transitions objects to Nearline storage class after 14 days.
- C. Store in local VM persistent disks.
- D. Delete the thumbnails after 14 days.

---

## Answer Key & Rationales

1. **B** — Spend-Based CUDs provide dollar-based commitment flexibility across diverse compute services (Compute Engine, GKE, Cloud SQL, Cloud Run) and regions.
2. **B** — Active Assist Idle VM and Rightsizing Recommenders analyze historical telemetry to recommend instance downsizing or termination.
3. **B** — Progressive Canary Deployments incrementally increase traffic weights to new versions while continuously evaluating health metrics.
4. **B** — Continuous Cloud Billing export to BigQuery streams granular resource cost data for advanced SQL analytics and Looker visualization.
5. **A** — Cloud Profiler utilizes statistical sampling to generate call-tree flame graphs of production CPU and memory usage with minimal overhead ($< 1\%$).
6. **B** — Ephemeral CI/CD runners are fault-tolerant and short-lived, making them ideal candidates for Spot VMs or managed Cloud Build pools.
7. **B** — DMS parallelizes snapshot loading and utilizes continuous binlog replication to minimize migration lag and cutover time.
8. **A** — BigQuery automatically halves storage costs for any table or partition unmodified for 90 days, with zero impact on query performance.
9. **B** — Enforcing IaC governance requires restricting direct console write access, forcing all changes through CI/CD pipelines with automated drift detection.
10. **B** — Transitioning objects to Nearline storage after 14 days dramatically reduces storage costs while maintaining sub-second read latency for long-tail requests.
