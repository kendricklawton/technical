# Domain 4: Analyzing and Optimizing Technical and Business Processes — Quiz 1

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
An enterprise has steady, predictable baseline workloads running on 500 Compute Engine instances in `us-central1` that will run 24/7 for the next three years. The CFO wants to achieve the maximum possible discount on compute costs without altering the VM configurations. Which purchasing option should the architect recommend?
- A. Spot VMs
- B. 3-Year Resource-Based Committed Use Discounts (CUDs)
- C. On-demand billing with monthly invoicing
- D. 1-Year Spend-Based CUDs

---

### Question 2
A data engineering team runs a batch Apache Spark job every night that takes 4 hours to process raw event logs. The job has built-in checkpointing; if a worker node disappears, the Spark driver automatically redistributes tasks to remaining workers without failing the job. How can the architect reduce compute costs for the worker nodes by up to 90%?
- A. Run workers on high-memory instances.
- B. Configure Dataproc secondary worker node pools using Spot VMs.
- C. Run the workers on local laptops.
- D. Purchase a 3-year CUD for batch workers.

---

### Question 3
An enterprise needs to migrate 500 Terabytes of static image assets from an on-premises NAS storage system to Google Cloud Storage. The company has a 10 Gbps Dedicated Interconnect with ample available bandwidth. The migration must support automated synchronization, data integrity validation, and minimal manual operational overhead. Which service is best suited?
- A. Shipping USB hard drives via postal courier
- B. Storage Transfer Service with on-premises agents installed on the local network
- C. Manual `curl` scripts running in bash
- D. Transfer Appliance

---

### Question 4
A legacy retail database running on self-managed PostgreSQL on a physical server in an on-premises datacenter needs to be migrated to Cloud SQL for PostgreSQL. The business requires near-zero downtime, allowing live transactions to continue on-premises until a brief 5-minute DNS cutover window. Which Google Cloud migration tool natively supports this workflow?
- A. Storage Transfer Service
- B. Database Migration Service (DMS) using continuous Change Data Capture (CDC)
- C. Google Transfer Appliance
- D. Exporting database dumps to CSV files

---

### Question 5
A company wants to migrate 200 VMware virtual machines running Windows Server and Linux from an on-premises data center to Compute Engine. The team needs to test the migration by running non-disruptive test clones in GCP before executing the final cutover. Which tool provides this end-to-end VM replication and testing workflow?
- A. Migrate to Virtual Machines
- B. Google Cloud Deploy
- C. BigQuery Data Transfer Service
- D. Cloud Functions

---

### Question 6
An architect is reviewing the monthly cloud invoice for an e-commerce platform. Cloud Storage costs have steadily grown because millions of customer transaction PDF receipts uploaded 5 years ago are still stored in the Standard storage class. Regulatory compliance mandates receipts be kept for 7 years, but they are accessed fewer than once a year. What is the most cost-effective architectural solution?
- A. Delete all receipts immediately.
- B. Implement a Cloud Storage Object Lifecycle Management rule that transitions objects older than 365 days from Standard to Archive storage class.
- C. Download all PDFs to local office hard drives.
- D. Compress every PDF manually.

---

### Question 7
A company has development teams deploying experimental workloads across 20 sandbox Google Cloud projects. The finance director wants to ensure that if a sandbox project's monthly spend exceeds $1,000, all compute instances in that specific sandbox project are stopped automatically to prevent unexpected charges. How can this automation be achieved?
- A. Check the console manually every morning.
- B. Configure a Cloud Billing Budget alert for the sandbox projects that publishes budget notifications to a Pub/Sub topic, which triggers a Cloud Function that programmatically disables billing or stops running Compute Engine instances.
- C. Cancel the company credit card.
- D. Block all developer email accounts.

---

### Question 8
An analytics team runs ad-hoc SQL queries on BigQuery. A junior analyst accidentally ran an unpartitioned query that scanned 50 Terabytes of data, incurring an unexpected \$312 charge. What BigQuery administrative guardrail prevents analysts from accidentally launching queries that scan massive amounts of data?
- A. Deleting the BigQuery dataset
- B. Setting the "Maximum bytes billed" limit on query configurations or setting project-level custom query quotas
- C. Requiring queries to be written in Python only
- D. Disabling BigQuery for all analysts

---

### Question 9
In the "6 Rs" cloud migration framework, what is the definition of "Replatforming" (Lift and Optimize)?
- A. Rewriting the entire application from scratch using serverless microservices.
- B. Moving an application to the cloud with minor optimizations to take advantage of managed cloud services (e.g., migrating a self-hosted database to Cloud SQL) without fundamentally changing the core application architecture.
- C. Decommissioning an application.
- D. Keeping the application in the on-premises data center permanently.

---

### Question 10
An architect needs to migrate 800 Terabytes of seismic research data from an isolated field laboratory located in a remote desert location with only a 10 Mbps satellite internet link to Cloud Storage. The migration must be completed within 30 days. Which migration solution should be chosen?
- A. Transfer the data over the 10 Mbps satellite link using `gsutil`.
- B. Order a Google Cloud Transfer Appliance, copy the data locally over high-speed 10GbE network interfaces, ship the appliance back to Google, and have Google upload the data directly to the target Cloud Storage bucket.
- C. Print the data on paper and scan it.
- D. Upload the data via mobile 4G hotspot.

---

## Answer Key & Rationales

1. **B** — 3-Year Resource-Based CUDs deliver the maximum discount (up to 57%) for stable, predictable VM capacity in a known region and machine family.
2. **B** — Dataproc secondary workers run on Spot VMs at up to 90% discount; since Spark handles worker loss gracefully, this provides massive cost efficiency.
3. **B** — Storage Transfer Service with on-premises agents provides high-speed, managed, automated migration of local NAS data over existing Dedicated Interconnect.
4. **B** — Database Migration Service (DMS) utilizes continuous Change Data Capture (CDC) to maintain real-time sync with on-prem databases, enabling minimal downtime cutovers.
5. **A** — Migrate to Virtual Machines provides non-disruptive agentless VM replication, continuous background syncing, and test cloning capabilities.
6. **B** — Storage Lifecycle rules automatically transition infrequently accessed compliance records to the low-cost Archive storage class.
7. **B** — Billing budget alerts published to Pub/Sub invoke serverless functions to programmatically stop instances or cap resources when budgets are breached.
8. **B** — Setting "Maximum bytes billed" on BigQuery queries aborts any query whose dry-run estimate exceeds the allowed scan threshold.
9. **B** — Replatforming adapts workloads to use managed platforms (like Cloud SQL or GKE) without rewriting application business logic.
10. **B** — Transfer Appliance overcomes severe bandwidth limitations by physically shipping high-capacity storage hardware to remote sites.
