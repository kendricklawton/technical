# Domain 2 (Configuration Management & IaC) — Quiz 5: Elastic Beanstalk Architecture Deep Dive

**Topic:** Elastic Beanstalk Deployment Strategies, `.ebextensions`, Platform Hooks, Worker Tiers & Environment Decoupling  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: Automated Canary Traffic Shifting with Elastic Beanstalk Traffic Splitting

An e-commerce platform hosted on AWS Elastic Beanstalk needs to deploy a major application revision. The engineering director mandates an automated canary rollout: exactly 5% of incoming live customer traffic must be routed to the new application version for 20 minutes while monitoring CloudWatch 5xx error alarms. If error thresholds are breached, traffic must automatically shift back to the original version with zero impact on running instances. If clean, 100% of traffic must transition to the replacement fleet.

**Which Elastic Beanstalk deployment policy provides this automated canary traffic shifting?**
- A. `Traffic Splitting`: Creates a temporary Auto Scaling Group containing a full replacement fleet, routes 5% of incoming production traffic through the Application Load Balancer to the new version for the 20-minute evaluation window, and automatically rolls back if CloudWatch alarms trigger.
- B. `Rolling with Additional Batch`: Launches a 5% batch of new instances into the existing Auto Scaling Group, routes traffic, and begins updating remaining instances in sequential batches.
- C. `Immutable`: Creates a temporary Auto Scaling Group alongside the existing one, shifts 100% of traffic immediately once health checks pass, and terminates the old fleet.
- D. `All at Once`: Updates all instances in the existing Auto Scaling Group simultaneously, relying on ALB health checks to mark unhealthy targets.

---

### Question 2: Coordinated Database Migrations via `.ebextensions` Execution Semantics

An enterprise application running on AWS Elastic Beanstalk spans 10 Amazon EC2 instances inside an Auto Scaling Group. The application revision includes a database schema migration script. If all 10 instances execute the migration script simultaneously upon deployment, database table lock contention and race conditions will corrupt database state and crash the deployment.

**How should the migration script be declared in `.ebextensions` to ensure execution on exactly one instance?**
- A. Declare the migration command under the `container_commands` section in an `.ebextensions/*.config` file and enable `leader_only: true`, ensuring execution occurs on only a single instance after code extraction.
- B. Declare the migration command under the `commands` section in an `.ebextensions/*.config` file with `single_instance: true`, running before application code is staged.
- C. Configure a custom CloudWatch Alarm that triggers an AWS Lambda function to execute the migration script over SSH prior to the Elastic Beanstalk deployment.
- D. Temporarily scale the Auto Scaling Group `DesiredCapacity` down to 1 instance, deploy the application revision, and scale the capacity back to 10 instances.

---

### Question 3: Zero-Downtime Maintenance via Elastic Beanstalk Managed Platform Updates

Corporate security compliance requires that all underlying operating system security patches and runtime platform updates on AWS Elastic Beanstalk environments must be applied automatically during off-peak weekend hours without causing application downtime or capacity loss.

**How should Managed Platform Updates be configured in Elastic Beanstalk to satisfy these requirements?**
- A. Enable **Managed Platform Updates** in the environment settings, schedule an automated weekly **Maintenance Window** during off-peak hours, select the update level for **Patch and Minor** releases, and configure the update mechanism to use an **Immutable** deployment strategy.
- B. Write an AWS Systems Manager Automation runbook that executes `eb upgrade` across all Elastic Beanstalk environments every Sunday at 02:00 UTC via EventBridge Scheduler.
- C. Configure an AWS Lambda function that triggers an environment clone weekly, redeploying the application to the new environment and swapping DNS CNAMEs.
- D. Enable Auto Scaling Scheduled Actions to terminate instances sequentially during the weekend, forcing Elastic Beanstalk to launch instances with updated AMIs.

---

### Question 4: Architectural Lifecycle Decoupling for Stateful Databases in Elastic Beanstalk

A startup previously provisioned an Amazon RDS PostgreSQL DB instance directly inside their Elastic Beanstalk environment template using the `AWS::RDS::DBInstance` resource. During a routine environment cleanup exercise, an engineer deleted the Elastic Beanstalk environment, which automatically destroyed the embedded database and resulted in catastrophic data loss.

**What is the architectural best practice for managing stateful databases with AWS Elastic Beanstalk?**
- A. Decouple the database from the Elastic Beanstalk environment lifecycle: Provision the Amazon RDS database independently (via standalone CloudFormation, AWS CDK, or Terraform), and inject the database connection string and credentials into the Elastic Beanstalk environment as environment properties.
- B. Add `DeletionPolicy: Retain` directly to the `AWS::RDS::DBInstance` resource inside the `.ebextensions` configuration file to prevent deletion when the environment terminates.
- C. Place the database inside an Amazon Elastic File System (EFS) volume mounted to the Elastic Beanstalk EC2 instances using an `.ebextensions` script.
- D. Enable Termination Protection on the Elastic Beanstalk environment to permanently block all environment deletion API requests.

---

### Question 5: Zero-Downtime Architectural Migrations via Elastic Beanstalk CNAME Swaps

An operations team needs to perform an architectural upgrade on an Elastic Beanstalk environment, migrating from an existing Application Load Balancer (ALB) to a Network Load Balancer (NLB) to support static IP addresses and non-HTTP protocols. Elastic Beanstalk does not support altering load balancer types on an existing live environment in-place.

**How should the DevOps Engineer execute this migration with ZERO customer downtime?**
- A. Provision a **new, independent Elastic Beanstalk environment** configured with the Network Load Balancer, deploy and test the application on the new environment, and execute an **Elastic Beanstalk Swap Environment CNAMEs** operation to swap DNS routing instantaneously.
- B. Delete the existing Application Load Balancer from the EC2 console and attach a newly provisioned Network Load Balancer directly to the Elastic Beanstalk Auto Scaling Group.
- C. Update the environment configuration settings via the AWS CLI with `--options-to-update "Namespace=aws:elasticbeanstalk:environment,OptionName=LoadBalancerType,Value=network"`.
- D. Terminate the existing Elastic Beanstalk environment and deploy the new environment with the same environment name during a scheduled weekend maintenance window.

---

### Question 6: NGINX Reverse Proxy Customization: `.ebextensions` vs. `.platform/nginx`

An application running on Elastic Beanstalk on the Amazon Linux 2023 platform requires custom reverse proxy routing: incoming client requests with payload sizes up to 100 MB must be accepted, requiring the NGINX configuration parameter `client_max_body_size` to be increased from the default 1 MB to 100 MB.

**What is the AWS-recommended method on Amazon Linux 2 / AL2023 platforms to override NGINX proxy configurations?**
- A. Add a configuration file in the application source bundle at `.platform/nginx/conf.d/proxy.conf` containing the directive `client_max_body_size 100M;`.
- B. Add a script in `.ebextensions/nginx.config` that executes `sed` to edit `/etc/nginx/nginx.conf` directly and restarts the NGINX service in `commands:`.
- C. Upload a custom pre-compiled NGINX binary inside the application source zip file and update the systemd service unit file.
- D. Define an environment variable `NGINX_MAX_BODY_SIZE=100M` under the Elastic Beanstalk environment configuration settings.

---

### Question 7: Lifecycle Execution Sequences: `.platform/hooks/` on Modern Elastic Beanstalk Platforms

On modern Amazon Linux 2 and Amazon Linux 2023 Elastic Beanstalk platforms, developers can place executable shell scripts within specialized directories under `.platform/hooks/` to customize deployment phases.

**What is the exact chronological execution order of hook directories during an application deployment?**
- A. `.platform/hooks/prebuild/` > `.platform/hooks/predeploy/` > `.platform/hooks/postdeploy/`
- B. `.platform/hooks/predeploy/` > `.platform/hooks/postdeploy/` > `.platform/hooks/prebuild/`
- C. `.platform/hooks/postdeploy/` > `.platform/hooks/predeploy/` > `.platform/hooks/prebuild/`
- D. `.platform/confighooks/` > `.platform/hooks/postdeploy/` > `.platform/hooks/predeploy/`

---

### Question 8: Asynchronous Background Processing with Elastic Beanstalk Worker Tiers

An enterprise web application processes asynchronous video encoding tasks. The web tier runs in an Elastic Beanstalk Web Server Environment. Long-running encoding tasks take up to 45 minutes and must not block web tier HTTP worker threads.

**How should this asynchronous processing architecture be designed using Elastic Beanstalk?**
- A. Deploy an **Elastic Beanstalk Worker Environment**: The web tier publishes video encoding jobs to an Amazon SQS queue; the SQS daemon (`sqsd`) running on worker instances polls the queue, translates messages into local HTTP POST requests against the local application, and manages message acknowledgment.
- B. Configure the Web Server Environment with a larger EC2 instance type and execute encoding tasks in background Python sub-processes.
- C. Deploy an AWS Lambda function with a 60-minute execution timeout to process video files directly from Amazon S3.
- D. Attach an Amazon Kinesis Data Stream to the Web Server Auto Scaling Group to buffer video encoding tasks in memory.

---

### Question 9: Eliminating In-Place Configuration Drift with Elastic Beanstalk Immutable Updates

A production environment experiences intermittent deployment failures when using the `Rolling` deployment policy. Over time, manual package installations and failed rolling deployments left different EC2 instances running mismatched library versions and altered configuration files.

**Why does selecting the `Immutable` deployment policy permanently resolve in-place configuration drift?**
- A. The `Immutable` deployment policy **never modifies existing production EC2 instances in-place**; it provisions a brand-new temporary Auto Scaling Group with fresh instances, deploys the new revision, verifies health checks, shifts traffic cleanly, and terminates the old instances completely.
- B. The `Immutable` policy locks the Linux root filesystem as read-only (`ro`), physically preventing administrators from editing configuration files over SSH.
- C. The `Immutable` policy automatically converts the application into a Docker container and deploys it to an Amazon EKS cluster.
- D. The `Immutable` policy takes automated Amazon EBS snapshots of every instance before deploying changes and automatically rolls back if drift is detected.

---

## Answer Key & Detailed Explanations

### Question 1: Automated Canary Traffic Shifting with Elastic Beanstalk Traffic Splitting
**Answer:** A  
**Explanation:**  
The **`Traffic Splitting`** deployment policy in AWS Elastic Beanstalk provides automated canary testing. It provisions a full replacement fleet in a temporary Auto Scaling Group behind the Application Load Balancer. It routes a designated percentage of incoming live traffic (e.g. 5%) to the replacement fleet for a defined evaluation window (e.g. 20 minutes) while monitoring CloudWatch Alarms. If alarms breach, traffic is shifted back immediately; if clean, 100% of traffic shifts and the old instances are terminated.

---

### Question 2: Coordinated Database Migrations via `.ebextensions` Execution Semantics
**Answer:** A  
**Explanation:**  
In `.ebextensions/*.config`, commands declared under `container_commands` execute from the application staging directory *after* the application source code has been unpacked, but *before* the application server starts. Setting **`leader_only: true`** ensures that Elastic Beanstalk chooses exactly one EC2 instance in the environment to execute the command, eliminating database migration concurrency conflicts.

---

### Question 3: Zero-Downtime Maintenance via Elastic Beanstalk Managed Platform Updates
**Answer:** A  
**Explanation:**  
**Managed Platform Updates** automatically apply operating system patches and platform runtime updates during a scheduled weekly maintenance window. Configuring the update level to **Patch and Minor** ensures only backward-compatible updates are applied. By selecting the **Immutable** deployment mechanism, Elastic Beanstalk creates a temporary ASG to validate the new platform version before shifting traffic, guaranteeing zero application downtime or capacity reduction.

---

### Question 4: Architectural Lifecycle Decoupling for Stateful Databases in Elastic Beanstalk
**Answer:** A  
**Explanation:**  
Stateful data stores (such as Amazon RDS, Amazon Aurora, or Amazon ElastiCache) should **never be coupled directly to the lifecycle of an Elastic Beanstalk environment**. If an environment is terminated or cloned, embedded databases are terminated with it. The architectural best practice is to provision databases independently using dedicated CloudFormation, CDK, or Terraform, passing the database endpoints and credentials into the Elastic Beanstalk environment as environment properties.

---

### Question 5: Zero-Downtime Architectural Migrations via Elastic Beanstalk CNAME Swaps
**Answer:** A  
**Explanation:**  
Elastic Beanstalk does not support altering load balancer types (e.g. ALB to NLB) on an active environment in-place. The zero-downtime migration path is to provision a **new Elastic Beanstalk environment** configured with the desired Network Load Balancer, deploy and validate the application, and execute a **Swap Environment CNAMEs** operation. This switches the Route 53 DNS records between the two environments seamlessly.

---

### Question 6: NGINX Reverse Proxy Customization: `.ebextensions` vs. `.platform/nginx`
**Answer:** A  
**Explanation:**  
On Amazon Linux 2 and Amazon Linux 2023 platforms, Elastic Beanstalk provides native configuration directories for NGINX: `.platform/nginx/conf.d/`. Placing a `.conf` file (e.g. `proxy.conf`) containing `client_max_body_size 100M;` inside `.platform/nginx/conf.d/` automatically includes it in the NGINX `http` context without manual script execution or restarting services via `commands:`.

---

### Question 7: Lifecycle Execution Sequences: `.platform/hooks/` on Modern Elastic Beanstalk Platforms
**Answer:** A  
**Explanation:**  
Modern Elastic Beanstalk platforms execute deployment hooks in a fixed chronological order:  
1. `.platform/hooks/prebuild/`: Executes before the platform builds and stages the application.  
2. `.platform/hooks/predeploy/`: Executes after the application is built and staged, but before it is deployed to the runtime server.  
3. `.platform/hooks/postdeploy/`: Executes after the application is deployed and the service daemon is running.

---

### Question 8: Asynchronous Background Processing with Elastic Beanstalk Worker Tiers
**Answer:** A  
**Explanation:**  
An **Elastic Beanstalk Worker Environment** is specifically designed for asynchronous background job processing. It provisions an Auto Scaling Group of EC2 instances running an AWS-managed SQS daemon (`sqsd`). The `sqsd` daemon polls an Amazon SQS queue for incoming messages, formats them as HTTP POST requests, and delivers them to the local application listening on localhost. If the task returns HTTP 200, `sqsd` deletes the message from the queue.

---

### Question 9: Eliminating In-Place Configuration Drift with Elastic Beanstalk Immutable Updates
**Answer:** A  
**Explanation:**  
The **Immutable** deployment policy guarantees that existing production instances are never altered in-place. Elastic Beanstalk launches an entirely separate, temporary Auto Scaling Group containing fresh EC2 instances running the new code revision. Once replacement instances pass health checks, traffic is shifted and the old instances are terminated. This ensures zero lingering file artifacts, zero dependency drift, and guaranteed identical fleet state across all nodes.
