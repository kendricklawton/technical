# Domain 2 (Configuration Management & IaC) — Quiz 5: Elastic Beanstalk Architecture

**Topic:** Elastic Beanstalk Deployment Strategies, `.ebextensions`, Traffic Splitting & Environment Governance  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: Traffic Splitting Deployments in Elastic Beanstalk

A retail platform hosted on AWS Elastic Beanstalk needs to deploy a major application release. The company requires a canary testing phase where 5% of real user traffic is routed to the new version for 20 minutes while evaluating CloudWatch error alarms. If error thresholds are breached, traffic must automatically revert. If clean, 100% of traffic must shift to the new version.

**Which Elastic Beanstalk deployment policy provides this automated canary traffic shifting?**
- A. `All at Once`
- B. `Rolling with Additional Batch`
- C. `Immutable`
- D. **`Traffic Splitting`**

---

### Question 2: Running Database Migrations with `leader_only` in `.ebextensions`

An Elastic Beanstalk deployment contains multiple EC2 instances in an Auto Scaling Group. The application bundle contains a database migration command defined in `.ebextensions/db.config`. If all instances execute the migration script simultaneously, database table lock conflicts will crash the deployment.

**How can the migration script be restricted to run on only one instance in the environment?**
- A. Run the script manually via SSH on one instance before deployment.
- B. Define the command under `container_commands` in `.ebextensions` and set **`leader_only: true`**.
- C. Reduce the Auto Scaling Group size to 1 during deployments.
- D. Place the script under `commands` with `single_instance: true`.

---

### Question 3: Elastic Beanstalk Managed Platform Updates

An enterprise security policy mandates that all operating system security patches and runtime platform updates on AWS Elastic Beanstalk environments must be applied automatically during off-peak hours without causing application downtime.

**How should Managed Platform Updates be configured in Elastic Beanstalk?**
- A. Write an AWS Lambda function triggered on a weekly cron schedule to execute `eb upgrade`.
- B. Enable **Managed Platform Updates** in the Elastic Beanstalk environment configuration. Set an automated weekly **Maintenance Window** during off-peak hours, configure the update level to **Patch and Minor**, and ensure the update uses an **Immutable** deployment mechanism to guarantee zero downtime.
- C. Managed updates cannot be performed without terminating the environment.
- D. Disable Auto Scaling.

---

### Question 4: Externalizing Stateful Resources from Elastic Beanstalk

A junior engineer uses `.ebextensions` to provision an Amazon RDS database inside the Elastic Beanstalk environment (`AWS::RDS::DBInstance`). When the environment is terminated during a cleanup exercise, the database is deleted along with all customer production data.

**What is the architectural best practice for managing stateful databases with Elastic Beanstalk?**
- A. Set `DeletionPolicy: Retain` in `.ebextensions`.
- B. **Decouple the database from the Elastic Beanstalk environment lifecycle**: provision the Amazon RDS database cluster independently (via standalone CloudFormation, Terraform, or CDK), and pass the database connection string and credentials into the Elastic Beanstalk environment as environment properties.
- C. Elastic Beanstalk environments must never be deleted.
- D. Store database files on the EC2 root EBS volume.

---

### Question 5: Environment Clones vs. Blue/Green CNAME Swaps

A team needs to perform an architectural upgrade on an Elastic Beanstalk environment, migrating from an Application Load Balancer to a Network Load Balancer. Elastic Beanstalk does not support changing load balancer types on an existing live environment in-place.

**How should the DevOps Engineer execute this migration with zero downtime?**
- A. Delete the environment and recreate it with the NLB.
- B. **Clone or create a new Elastic Beanstalk environment** configured with the Network Load Balancer. Deploy and test the application on the new environment. Once verified, perform an **Elastic Beanstalk Swap Environment CNAMEs** operation to swap DNS routing between the two environments with zero downtime.
- C. Modify Route 53 DNS records manually with a 24-hour TTL.
- D. Elastic Beanstalk does not support NLBs.

---

### Question 6: Overriding Software Configurations via `.ebextensions` Option Settings

An application deployed on Elastic Beanstalk needs to increase the web server upload size limit (`client_max_body_size`) on NGINX from 1 MB to 50 MB.

**What is the correct way to apply this configuration via `.ebextensions`?**
- A. Manually edit `/etc/nginx/nginx.conf` over SSH on every instance.
- B. Add a configuration file in `.ebextensions/nginx.config` that creates `/etc/nginx/conf.d/proxy.conf` containing `client_max_body_size 50M;` using the `files` section of the configuration file.
- C. Upload a custom NGINX binary in the application zip file.
- D. Set an environment variable `NGINX_MAX_SIZE=50M`.

---

### Question 7: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap

A centralized CI/CD pipeline in Account 111111111111 deploys an ECS service in Account 222222222222. The deployment stage fails with: 'The ciphertext refers to a customer master key that does not exist, does not allow access, or is invalid.' The S3 bucket is encrypted with the default AWS-managed key 'aws/s3'. How should the engineer resolve this?

- A. Update the S3 bucket policy to grant read access to Account 222222222222.
- B. Reconfigure the S3 artifact bucket to use a Customer Managed Key (CMK) and grant `kms:Decrypt` and `kms:DescribeKey` permissions to Account 222222222222's execution role in the KMS Key Policy.
- C. Create a VPC Endpoint for S3 in Account 222222222222.
- D. Enable S3 Object Ownership with Bucket Owner Preferred.

---

### Question 8: CodeDeploy In-Place ApplicationStop Lifecycle Failure

During an in-place deployment to an EC2 Auto Scaling Group, the deployment fails at the ApplicationStop lifecycle event hook because a previous buggy script had been deployed in the older revision. How can the engineer force CodeDeploy to deploy the new fixed revision without terminating the instances?

- A. Delete the CodeDeploy agent from all EC2 instances.
- B. Trigger a new deployment via the AWS CLI with the `--ignore-application-stop-failures` flag enabled.
- C. Manually stop the EC2 instances in the AWS Management Console.
- D. Reboot the instances and rerun the same deployment ID.

---

### Question 9: CloudFormation Custom Resource 60-Minute Stack Freeze

A CloudFormation stack creation hangs for exactly 1 hour in `CREATE_IN_PROGRESS` on a Lambda-backed custom resource before failing and rolling back. What was the root cause?

- A. The Lambda function exceeded its maximum execution timeout of 15 minutes.
- B. An unhandled exception crashed the Lambda function before it could send an HTTP PUT response with status SUCCESS or FAILED to the pre-signed Amazon S3 ResponseURL.
- C. CloudFormation lost IAM permissions to invoke Lambda.
- D. The S3 bucket holding the CloudFormation template was deleted.

---

## Answer Key & Detailed Explanations

### Question 1: Traffic Splitting Deployments in Elastic Beanstalk

**Answer:** D  
**Explanation:** 
AWS Elastic Beanstalk provides the **Traffic Splitting** deployment policy specifically for canary testing. It creates an independent temporary Auto Scaling Group running the new application version behind the existing load balancer and routes a configurable percentage of incoming traffic (e.g. 5%) to the new version for a set evaluation period (e.g. 20 minutes). If health checks or CloudWatch alarms fail, traffic reverts instantly. If healthy, traffic cutover completes automatically.

---

### Question 2: Running Database Migrations with `leader_only` in `.ebextensions`

**Answer:** B  
**Explanation:** 
In Elastic Beanstalk `.ebextensions`, commands placed in the `container_commands` block support the `leader_only: true` attribute. When configured, Elastic Beanstalk elects a single instance in the Auto Scaling Group as the "leader" to execute the command. All other instances skip the command, ensuring database migrations execute exactly once per deployment without concurrency contention.

---

### Question 3: Elastic Beanstalk Managed Platform Updates

**Answer:** B  
**Explanation:** 
Elastic Beanstalk **Managed Platform Updates** automatically applies patch and minor version platform updates (OS, web server, and application runtime patches) on a weekly maintenance schedule chosen by the customer. Managed updates use an **Immutable** deployment process to create a new fleet of instances, verify health, and swap traffic, ensuring zero downtime and automated rollback if the new platform version fails health checks.

---

### Question 4: Externalizing Stateful Resources from Elastic Beanstalk

**Answer:** B  
**Explanation:** 
AWS strongly recommends decoupling persistent data stores (RDS, ElastiCache, S3) from the Elastic Beanstalk environment lifecycle. Databases created inside an Elastic Beanstalk environment bundle are tied directly to the environment's lifecycle and are destroyed when the environment is terminated. Creating the database independently ensures its lifecycle is insulated from application environment teardowns and blue/green environment swaps.

---

### Question 5: Environment Clones vs. Blue/Green CNAME Swaps

**Answer:** B  
**Explanation:** 
When an infrastructure property cannot be modified in-place (such as changing the load balancer type, VPC subnet tier, or major platform generation), the standard Elastic Beanstalk technique is a **Blue/Green deployment via Swap Environment CNAMEs**. The new environment is launched and fully validated independently. Executing a CNAME swap redirects traffic instantly by exchanging the DNS prefixes between the two environments.

---

### Question 6: Overriding Software Configurations via `.ebextensions` Option Settings

**Answer:** B  
**Explanation:** 
In Elastic Beanstalk Linux platforms (such as AL2 and AL2023), NGINX includes all configuration files located in `/etc/nginx/conf.d/`. Using the `files` block in an `.ebextensions` configuration file, developers can deploy `/etc/nginx/conf.d/proxy.conf` with `client_max_body_size 50M;`. Elastic Beanstalk writes the file and reloads NGINX automatically during deployment.

---


### Question 7: Cross-Account CodePipeline S3 Artifact KMS Decrypt Trap
**Answer:** B
**Explanation:** Default AWS-managed keys like `aws/s3` cannot have their key policies modified and CANNOT be accessed across AWS accounts. Cross-account CodePipeline deployments strictly require an S3 bucket encrypted with a Customer Managed Key (CMK) whose key policy explicitly grants cross-account decrypt rights.

---

### Question 8: CodeDeploy In-Place ApplicationStop Lifecycle Failure
**Answer:** B
**Explanation:** CodeDeploy executes the `ApplicationStop` hook from the PREVIOUS revision's AppSpec file. If that script fails, all future deployments fail at that step unless you specify `--ignore-application-stop-failures`.

---

### Question 9: CloudFormation Custom Resource 60-Minute Stack Freeze
**Answer:** B
**Explanation:** CloudFormation Custom Resources expect a signed JSON payload uploaded to the pre-signed S3 `ResponseURL`. If the Lambda function crashes or exits without sending this signal, CloudFormation waits 1 hour before timing out.

---

