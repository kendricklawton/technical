# Domain 1 (SDLC Automation) — Quiz 2: CodeBuild Deep Dive

**Topic:** AWS CodeBuild Projects, Buildspec Specifications, VPC Networking & Caching  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: CodeBuild Environment Variables Precedence

An AWS CodeBuild project compiles a microservice. An engineer notices that the build output uses a staging database URL instead of the production URL. The parameter is defined in multiple places:
1. Hardcoded in `buildspec.yml` under `env: variables`.
2. Passed as an environment variable override in the AWS CodePipeline action configuration.
3. Defined as an environment variable in the CodeBuild project settings.
4. Passed via the AWS CLI command `start-build --environment-variables-override`.

**Which value takes the HIGHEST precedence during a CodeBuild execution?**
- A. The value in `buildspec.yml` under `env: variables`.
- B. The value passed via the AWS CLI or CodePipeline `start-build` API call override.
- C. The value configured in the CodeBuild Project definition.
- D. Parameter Store defaults.

---

### Question 2: CodeBuild Phase Failure and Finally Blocks

A CodeBuild project runs automated integration tests. In the `build` phase, a shell script executes unit tests. If any test fails, the build must still execute a cleanup command in the `post_build` phase to terminate temporary test resources before CodeBuild exits.

**How must the `buildspec.yml` be structured to guarantee cleanup execution even if tests fail?**
- A. Place the cleanup commands inside the `finally` block of the `build` or `post_build` phase.
- B. Set `continue-on-error: true` at the top of the buildspec.
- C. Use `|| true` on every test command.
- D. CodeBuild always runs all phases regardless of failures.

---

### Question 3: CodeBuild VPC Subnet Routing and Public Internet Access

A CodeBuild project is configured to run inside a VPC to access an internal Amazon DocumentDB cluster. During the build, `npm install` times out attempting to reach `registry.npmjs.org`. The VPC has an Internet Gateway attached, and the CodeBuild project is configured with a public subnet that has a route `0.0.0.0/0 -> igw-xxxx`.

**Why does CodeBuild fail to reach the internet, and what is the fix?**
- A. CodeBuild does not support Node.js inside a VPC.
- B. CodeBuild ENIs assigned to VPC subnets **never receive public IPv4 addresses**. Even if placed in a public subnet, traffic cannot reach the Internet Gateway directly. The CodeBuild ENIs must be placed in **private subnets** that route `0.0.0.0/0` through a **NAT Gateway**.
- C. The security group must allow inbound traffic on port 80.
- D. AWS DocumentDB must be moved to a public subnet.

---

### Question 4: Multi-Architecture Docker Builds with Buildx in CodeBuild

A company needs to build Docker container images for both `x86_64` (Intel/AMD) and `arm64` (AWS Graviton) architectures in a single CI build and push a multi-arch manifest list to Amazon ECR.

**How should the CodeBuild `buildspec.yml` be configured?**
- A. CodeBuild can only build `x86_64` images; Graviton builds must run on EC2.
- B. Enable the `privileged-mode: true` flag in the CodeBuild project. In `buildspec.yml`, initialize Docker Buildx, register QEMU emulators (`docker run --privileged --rm tonistiigi/binfmt --install all`), and build with `docker buildx build --platform linux/amd64,linux/arm64 -t $REPO:tag --push .`.
- C. Run two separate CodePipeline pipelines in parallel.
- D. Compile code into an AMI instead of a Docker image.

---

### Question 5: CodeBuild Batch Builds for Matrix Testing

A Python library must be tested against Python 3.9, 3.10, 3.11, and 3.12 across both Linux and Windows environments before publication. Writing separate CodeBuild projects for each permutation creates massive maintenance overhead.

**Which AWS CodeBuild feature enables running these combinatorial tests concurrently from a single build specification?**
- A. CodeBuild **Batch Builds** with a `build-matrix` defined in `buildspec.yml`.
- B. AWS Step Functions invoking CodeBuild sequentially.
- C. Running a bash `for` loop inside a single container.
- D. S3 event notifications.

---

### Question 6: Securely Passing Build Artifacts Between Buildspec Phases

A build script generates a dynamic authorization token in the `pre_build` phase that must be available as an environment variable to scripts executing in the subsequent `build` and `post_build` phases.

**How can data generated in one phase be shared with later phases in CodeBuild?**
- A. CodeBuild phases run in completely separate operating system containers, so data cannot be shared.
- B. Export the variable using a shell script, or write the value to a file (e.g., `/tmp/token.txt`) on the shared ephemeral file system, which persists across all phases of a single build execution.
- C. Upload the token to public S3.
- D. Call the CodeBuild API to update project environment variables.

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

### Question 1: CodeBuild Environment Variables Precedence

**Answer:** B  
**Explanation:** 
In AWS CodeBuild, the order of precedence for environment variables from highest to lowest is:
1. Values passed in the `start-build` API call (CLI or CodePipeline action override).
2. Values defined in the CodeBuild project settings.
3. Values defined in the `buildspec.yml` file.
Therefore, any parameter passed dynamically at start-build overrides project and buildspec settings.

---

### Question 2: CodeBuild Phase Failure and Finally Blocks

**Answer:** A  
**Explanation:** 
By default, if any command in a phase fails with a non-zero exit code, CodeBuild skips subsequent commands in that phase and transitions to the `post_build` phase. However, within any phase, commands placed in a `finally` block are **guaranteed to run** even if commands in the preceding command list fail, ensuring resource cleanup.

---

### Question 3: CodeBuild VPC Subnet Routing and Public Internet Access

**Answer:** B  
**Explanation:** 
AWS CodeBuild does not assign public IPv4 addresses to the Elastic Network Interfaces (ENIs) it provisions in a customer VPC. Because Internet Gateways require endpoints to possess public IP addresses to route packets, placing CodeBuild in a public subnet fails. The only supported architecture for VPC-enabled CodeBuild requiring internet access is placement in **private subnets** routed to an active **NAT Gateway** in a public subnet.

---

### Question 4: Multi-Architecture Docker Builds with Buildx in CodeBuild

**Answer:** B  
**Explanation:** 
AWS CodeBuild supports Docker Buildx. To run Docker inside CodeBuild, `privileged-mode` must be enabled in the project settings. Developers can register QEMU binary format emulators and use `docker buildx build --platform linux/amd64,linux/arm64` to build multi-platform images and push a single multi-arch manifest list directly to Amazon ECR.

---

### Question 5: CodeBuild Batch Builds for Matrix Testing

**Answer:** A  
**Explanation:** 
AWS CodeBuild supports **Batch Builds**. Using the `batch` stanza in `buildspec.yml`, engineers can define a `build-matrix` specifying dimensions (e.g. `runtime-versions: python: [3.9, 3.10, 3.11, 3.12]` and `image: [linux, windows]`). CodeBuild automatically fans out and provisions independent concurrent build instances for every permutation in the matrix and aggregates the results into a single report.

---

### Question 6: Securely Passing Build Artifacts Between Buildspec Phases

**Answer:** B  
**Explanation:** 
All phases of a single CodeBuild run execute within the **same container and shared filesystem**. Files written to disk (e.g., `/tmp/token.txt` or `.env`) in the `pre_build` phase remain immediately accessible to scripts in `build` and `post_build`. Alternatively, variables can be exported to `$BASH_ENV` or sourced across scripts.

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

