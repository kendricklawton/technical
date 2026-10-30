# Domain 4: Analyzing and Optimizing Technical and Business Processes — Quiz 3

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
A financial company with 500 projects wants to enforce that all Google Cloud storage buckets have uniform bucket-level access enabled and that no public buckets exist across the entire organization. How should this compliance policy be enforced continuously at scale?
- A. Writing a manual script run by the security admin every Friday.
- B. Implementing Organization Policy constraints (`constraints/storage.uniformBucketLevelAccess` and `constraints/storage.publicAccessPrevention`) at the root Organization node.
- C. Manually clicking the settings on all 500 projects.
- D. Sending an email reminder to developers.

---

### Question 2
An architect is designing a migration strategy for a monolithic on-premises enterprise application. The application consists of tightly coupled components that share local file systems and low-latency IPC. What is the recommended first phase migration strategy according to Google Cloud best practices?
- A. Immediately rewrite the entire system into 50 microservices on Cloud Run.
- B. Rehost (Lift and Shift) the application onto Compute Engine VMs with minimal architectural alterations to validate operational stability before refactoring.
- C. Discard the application and buy off-the-shelf software.
- D. Move the application to BigQuery.

---

### Question 3
A media processing pipeline processes 50,000 images per hour using Cloud Functions. The team notices costs are higher than expected. Profiling reveals that each function invocation spends 80% of its execution time waiting for an external third-party API response over the network. How can the team optimize this architecture for cost?
- A. Increase the memory of each Cloud Function to 32 GB.
- B. Switch to Cloud Run with high concurrency (`concurrency > 50`), allowing a single container instance to process multiple simultaneous requests concurrently while waiting for external I/O.
- C. Run the workload on bare-metal servers.
- D. Disable the third-party API.

---

### Question 4
An architect needs to select a data transfer mechanism to migrate 2 Petabytes of historical log files from an on-premises EMC Isilon storage array to Google Cloud Storage. The company has a 1 Gbps internet connection that cannot be fully dedicated to migration. What is the fastest and most practical migration path?
- A. Stream over the 1 Gbps link (will take over 6 months).
- B. Use multiple Google Cloud Transfer Appliances shipped to the datacenter to copy data locally at multi-gigabit speeds and transfer offline.
- C. Upload via developer laptops over Wi-Fi.
- D. Recreate the logs manually.

---

### Question 5
A company wants to optimize BigQuery query performance and reduce compute costs. They have a 10 TB `web_traffic` table queried daily with clauses like `WHERE event_date = '2026-10-21' AND country = 'US' AND device_type = 'mobile'`. How should the table be designed?
- A. Single flat table with no indexing.
- B. Partitioned by `event_date` and clustered by `country` and `device_type`.
- C. Split into 365 separate tables for each day of the year.
- D. Converted to JSON files on Compute Engine.

---

### Question 6
During a cloud readiness assessment, an architect identifies an on-premises application that is slated for complete retirement in 4 months when a new SaaS vendor contract begins. What migration action should be taken for this application under the 6 Rs framework?
- A. Refactor to Cloud Spanner immediately.
- B. Retain on-premises until retirement; do not invest engineering resources into cloud migration.
- C. Migrate to VMware Engine.
- D. Rewrite in Go.

---

### Question 7
A company has configured automated alerts for anomalous billing spikes. An alert triggers indicating that a test GKE cluster created by an intern generated $5,000 in costs over the weekend because it was configured with 50 high-memory GPU nodes. What architectural control prevents unbudgeted high-cost instance provisioning in development environments?
- A. Organization Policy constraints restricting allowed VM machine types (`constraints/compute.vmMachineType`) in development folders.
- B. Firing the intern.
- C. Turning off the office Wi-Fi.
- D. Deleting the billing account.

---

### Question 8
An architect is designing an automated deployment pipeline for an e-commerce microservice. The release process requires executing automated end-to-end integration tests in a dedicated staging environment, followed by a manual approval gate before promoting the container image to production. Which Google Cloud service natively orchestrates this multi-target pipeline?
- A. Google Cloud Deploy
- B. Compute Engine startup scripts
- C. Cloud DNS
- D. Cloud Storage FUSE

---

### Question 9
A company wants to minimize compute licensing costs for Windows Server workloads on Google Cloud. What licensing option allows them to leverage their existing on-premises Windows Server licenses with Software Assurance on Compute Engine?
- A. Standard Google on-demand licensing
- B. Sole-Tenant Nodes with Bring Your Own License (BYOL)
- C. Spot VMs without licenses
- D. Linux VM conversion

---

### Question 10
When using Storage Transfer Service to migrate data from an Amazon S3 bucket to a Cloud Storage bucket, what credential mechanism ensures secure authentication without generating long-lived AWS IAM access keys?
- A. Sharing AWS root account passwords.
- B. Workload Identity Federation / AWS IAM Role assumption using Google's Storage Transfer Service service agent identity.
- C. Making the S3 bucket public.
- D. Running transfer jobs over unauthenticated HTTP.

---

## Answer Key & Rationales

1. **B** — Org Policies provide declarative, non-bypassable guardrails enforcing bucket-level access and preventing public storage exposure organization-wide.
2. **B** — Rehosting (Lift and Shift) establishes a rapid cloud footprint with minimal risk, providing a baseline for subsequent iterative refactoring.
3. **B** — Cloud Run concurrency allows a single container instance to handle multiple I/O-bound requests simultaneously, drastically reducing instance hours.
4. **B** — 2 Petabytes over a constrained 1 Gbps link would take months; Google Transfer Appliances provide physical offline bulk data ingestion in days.
5. **B** — Partitioning by date eliminates full-table scans, and clustering by frequently filtered dimensions (country, device) optimizes query execution and cost.
6. **B** — The "Retain" strategy avoids wasteful migration expenditures on workloads scheduled for near-term decommissioning.
7. **A** — `constraints/compute.vmMachineType` allows organizations to restrict expensive instance types (such as GPU or M-series VMs) in dev/sandbox environments.
8. **A** — Google Cloud Deploy natively manages multi-target promotion pipelines with automated verification phases and explicit approval gates.
9. **B** — Sole-Tenant Nodes provide dedicated physical hardware instances necessary to satisfy Microsoft BYOL licensing terms.
10. **B** — Assuming AWS IAM roles via federated trust establishes secure cross-cloud authorization without storing long-lived AWS access keys.
