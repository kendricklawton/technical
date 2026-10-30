# Domain 1 (SDLC Automation) — Quiz 2: CodeBuild Deep Dive

**Topic:** AWS CodeBuild Projects, Buildspec Specifications, VPC Networking, Caching & Test Reporting  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: CodeBuild Environment Variable Resolution Precedence Hierarchy

An AWS CodeBuild project compiles a critical financial microservice. An engineer observes that during pipeline execution, the build output connects to a staging database endpoint instead of the expected production endpoint. The database URL parameter is configured in four distinct locations:
1. In `buildspec.yml` under `env.variables.DB_URL`.
2. As an environment variable override passed in the AWS CodePipeline build action configuration.
3. As a project-level environment variable in the AWS CodeBuild project settings.
4. In AWS Systems Manager Parameter Store, referenced by the build environment.

**Which configuration source takes the HIGHEST evaluation precedence during a CodeBuild execution?**
- A. The value passed dynamically via the AWS CLI or CodePipeline `start-build` action environment variable override.
- B. The value hardcoded directly within the source repository's `buildspec.yml` file under the `env.variables` mapping.
- C. The static environment variable value configured directly within the AWS CodeBuild project definition settings.
- D. The dynamic value resolved from AWS Systems Manager Parameter Store via the `env.parameter-store` mapping.

---

### Question 2: Guaranteed Cleanup Execution in Buildspec Lifecycle Phases

A continuous integration pipeline executes end-to-end integration tests in AWS CodeBuild. During the `build` phase, a shell script provisions temporary mock services and database records. If any test assertion fails, the container must execute cleanup commands in the `post_build` phase to destroy temporary mock resources before CodeBuild exits, preventing resource leakage.

**How must the `buildspec.yml` be structured to guarantee that cleanup commands execute even when test commands fail?**
- A. Place the cleanup commands inside the `finally` block of the `build` or `post_build` phase within the `buildspec.yml`.
- B. Append `|| true` to the end of every individual test execution command in the `build` phase commands list.
- C. Add `continue-on-error: true` at the root level of the `buildspec.yml` specification to suppress container exits.
- D. Configure an Amazon EventBridge rule that detects CodeBuild `BUILD_STATE_CHANGE: FAILED` and triggers an AWS Lambda cleanup function.

---

### Question 3: CodeBuild VPC Subnet Routing and Public Internet Access

An enterprise AWS CodeBuild project is attached to an Amazon VPC to execute database migrations against an internal Amazon Aurora PostgreSQL DB cluster deployed in private subnets. During the build execution, `npm install` and `pip install` commands hang indefinitely and time out attempting to connect to public package repositories (`registry.npmjs.org`). The VPC has an Internet Gateway attached, and the CodeBuild project is configured with subnets that have an active route `0.0.0.0/0 -> igw-xxxx`.

**Why does CodeBuild fail to reach the internet, and what architectural reconfiguration resolves the issue?**
- A. CodeBuild Elastic Network Interfaces (ENIs) deployed into VPC subnets **never receive public IPv4 addresses**; placing CodeBuild in a public subnet cannot route traffic to an Internet Gateway. The CodeBuild ENIs must be deployed into **private subnets** that route `0.0.0.0/0` outbound traffic through a **NAT Gateway** located in a public subnet.
- B. The security group attached to the CodeBuild project lacks an inbound rule permitting traffic from the Internet Gateway on port 443; an inbound rule for `0.0.0.0/0` on HTTPS must be added to allow return traffic.
- C. AWS CodeBuild does not support outbound internet access when attached to a VPC; language dependencies must be bundled as compressed tarballs and uploaded to Amazon S3 prior to build execution.
- D. The VPC route table must be updated to enable IPv6 egress-only internet gateway routing, and the CodeBuild project environment must be configured with a dual-stack network interface.

---

### Question 4: Multi-Architecture Container Compilation with Docker Buildx in CodeBuild

An organization is modernizing its container workloads to run on both Intel/AMD (`x86_64`) and AWS Graviton (`arm64`) Amazon ECS clusters. A single CI build in AWS CodeBuild must compile the application binary for both architectures, build multi-architecture Docker container images, and publish a unified multi-arch manifest list to Amazon ECR.

**How should the CodeBuild project and `buildspec.yml` be configured to achieve this?**
- A. Enable `privileged-mode: true` in the CodeBuild project settings; in `buildspec.yml`, initialize Docker Buildx, register QEMU emulators (`tonistiigi/binfmt`), and build with `docker buildx build --platform linux/amd64,linux/arm64 -t $REPO:tag --push .`.
- B. Configure CodeBuild to use the `ARM_CONTAINER` environment type, and execute standard `docker build` commands with the `--target-arch all` compilation flag.
- C. Provision two independent AWS CodePipeline pipelines running concurrently, using an Amazon EventBridge rule to merge the independent images into a manifest list via AWS Lambda.
- D. Pre-bake an Amazon Linux 2023 EC2 AMI with cross-compilation toolchains, and configure CodeBuild to launch an EC2 build fleet using custom launch templates.

---

### Question 5: High-Concurrency Combinatorial Testing via CodeBuild Batch Builds

An open-source Python SDK maintained in AWS CodeBuild must be validated against Python 3.9, 3.10, 3.11, and 3.12 across both Amazon Linux and Ubuntu container images. Testing each permutation sequentially in a single build takes over 90 minutes. Creating 8 separate CodeBuild projects creates severe administrative overhead and fragmented build metrics.

**Which native CodeBuild capability allows running all 8 permutations concurrently from a single build configuration?**
- A. Configure CodeBuild **Batch Builds** with a `build-matrix` in `buildspec.yml`, defining dimensions for `runtime-versions: python` and `image`, allowing CodeBuild to fan out 8 parallel build instances automatically.
- B. Orchestrate 8 parallel CodeBuild projects using an AWS Step Functions state machine with a `Distributed Map` state running in `Inline` mode.
- C. Write a shell script in the `buildspec.yml` `build` phase that uses GNU Parallel to launch background container processes inside the single build container.
- D. Configure an Amazon EventBridge rule that intercepts GitHub pull request events and triggers 8 concurrent CodePipeline pipeline executions with environment overrides.

---

### Question 6: Sharing Dynamic State and Credentials Across Buildspec Lifecycle Phases

A complex continuous delivery build script executes an authentication handshake in the `pre_build` phase, obtaining a temporary short-lived JWT token. This token must be accessible to integration test runners and deployment utilities executing in the subsequent `build` and `post_build` phases.

**How can dynamic runtime variables generated during `pre_build` be shared with later phases in CodeBuild?**
- A. Export the variable into the shell environment using a script, or write the token to a file on the local filesystem (e.g. `/tmp/token.txt`); all buildspec phases execute within the **same container and shared filesystem**.
- B. Execute the `aws codebuild update-project` API call during the `pre_build` phase to update the project's default environment variables before the `build` phase begins.
- C. Store the generated token in an AWS Systems Manager Parameter Store parameter during `pre_build`, and re-query Parameter Store at the start of each subsequent phase.
- D. CodeBuild executes each phase in an isolated ephemeral Docker container; shared data must be uploaded to an Amazon S3 staging bucket and downloaded in each phase.

---

### Question 7: Reducing Cold Start Latency with AWS CodeBuild Lambda Compute

A development team has hundreds of microservices built with lightweight Node.js and Go binaries. The average build compilation takes only 35 seconds, but standard EC2-based CodeBuild containers take 45 to 60 seconds just to provision and boot the execution environment, more than doubling total CI pipeline latency. The team requires the **LOWEST CONTAINER PROVISIONING LATENCY** for rapid CI feedback.

**Which CodeBuild compute configuration satisfies this requirement?**
- A. Configure the CodeBuild project to use **AWS Lambda compute**, which provisions execution environments with near-instantaneous startup times (under 5 seconds) for fast, short-duration build jobs.
- B. Pre-warm dedicated Amazon EC2 build instances using an Auto Scaling Group Warm Pool configured with pre-initialized Docker daemons.
- C. Deploy an Amazon Elastic Kubernetes Service (EKS) cluster with Karpenter, and run CodeBuild self-hosted runners on EC2 Spot Instances.
- D. Enable local caching with `CUSTOM` mode on the existing EC2-based CodeBuild project to eliminate Docker image pull latency.

---

### Question 8: Secondary Source Repositories & Multi-Repo Ingestion in CodeBuild

An enterprise build process requires source code from a primary application repository in GitHub, alongside shared proprietary security libraries and configuration profiles stored in a secondary private AWS CodeCommit repository. The build must checkout both repositories into distinct local paths during the `install` phase.

**How should the CodeBuild project be configured to ingest multiple source repositories natively?**
- A. Define a **Secondary Source** in the CodeBuild project configuration with a custom source identifier (e.g. `shared_libs`); CodeBuild automatically clones the secondary repository to the directory specified by `$CODEBUILD_SRC_DIR_shared_libs`.
- B. Use Git submodules in the primary repository and execute `git submodule update --init --recursive` inside the `buildspec.yml` `install` phase using hardcoded personal access tokens.
- C. Configure an AWS Lambda function in CodePipeline that clones both repositories, zips their contents together into a single composite archive, and uploads it to S3.
- D. CodeBuild strictly supports only one source repository per project; the secondary library must be compiled and published to AWS CodeArtifact prior to the build.

---

### Question 9: Native Test Reporting and Code Coverage Integration in CodeBuild

An enterprise QA governance policy requires that all pull requests built by AWS CodeBuild publish standard JUnit test execution results and Cobertura code coverage reports. Engineering leads must be able to view pass/fail trends, execution duration, and test failure stack traces directly within the AWS CodeBuild console without downloading raw log files.

**How should this reporting requirement be declared in `buildspec.yml`?**
- A. Declare a **`reports:`** stanza in `buildspec.yml` specifying the report group ARN, report format (e.g. `JunitXml` or `CoberturaXml`), and the relative file paths to the generated test output XML files.
- B. Upload the raw XML test reports to an Amazon S3 bucket and configure an S3 Event Notification that triggers Amazon QuickSight to render test metrics.
- C. Parse the test output XML files in the `post_build` phase using `grep` and `awk`, and emit custom metrics to Amazon CloudWatch using `PutMetricData`.
- D. Configure an Amazon CloudWatch Logs metric filter on the CodeBuild log group to match test failure regex patterns and publish failure counts to SNS.

---

## Answer Key & Detailed Explanations

### Question 1: CodeBuild Environment Variable Resolution Precedence Hierarchy
**Answer:** A  
**Explanation:**  
In AWS CodeBuild, environment variables follow a strict precedence hierarchy (from highest to lowest priority):  
1. **API / CLI / Pipeline Overrides:** Parameters passed dynamically via `start-build --environment-variables-override` or through CodePipeline action configuration overrides.  
2. **Project Definition:** Static environment variables defined directly in the CodeBuild project settings.  
3. **Buildspec Definition:** Variables defined inside the `buildspec.yml` file under `env: variables`.  
4. **Parameter Store / Secrets Manager:** Dynamic references defined in `env: parameter-store` or `env: secrets-manager`.  
Therefore, dynamically passed action overrides take highest precedence.

---

### Question 2: Guaranteed Cleanup Execution in Buildspec Lifecycle Phases
**Answer:** A  
**Explanation:**  
In `buildspec.yml`, each phase (`install`, `pre_build`, `build`, `post_build`) supports an optional **`finally`** block. Commands placed within the `finally` block are guaranteed to execute even if preceding commands in that phase fail with non-zero exit codes. Placing cleanup logic in `build.finally` or `post_build.finally` ensures resources are reliably torn down before the build environment terminates.

---

### Question 3: CodeBuild VPC Subnet Routing and Public Internet Access
**Answer:** A  
**Explanation:**  
When CodeBuild attaches to a VPC, its Elastic Network Interfaces (ENIs) do not receive public IPv4 addresses. In AWS networking, an Internet Gateway requires attached resources to possess public IPv4 addresses to perform internet routability. Placing CodeBuild in a public subnet causes all outbound internet traffic to time out. The only valid architecture for VPC-enabled CodeBuild requiring outbound internet access is placement in **private subnets** whose route table points default traffic (`0.0.0.0/0`) to an active **NAT Gateway** in a public subnet.

---

### Question 4: Multi-Architecture Container Compilation with Docker Buildx in CodeBuild
**Answer:** A  
**Explanation:**  
AWS CodeBuild supports multi-architecture container builds using **Docker Buildx**. The project must have `privileged-mode: true` enabled to run the Docker daemon. In `buildspec.yml`, registering QEMU binary format emulators (`tonistiigi/binfmt`) allows cross-compilation across architectures. Executing `docker buildx build --platform linux/amd64,linux/arm64 --push` builds the respective binary architectures and publishes a unified OCI multi-arch image manifest list directly to Amazon ECR.

---

### Question 5: High-Concurrency Combinatorial Testing via CodeBuild Batch Builds
**Answer:** A  
**Explanation:**  
AWS CodeBuild **Batch Builds** natively support a `build-matrix` defined directly in `buildspec.yml`. By declaring matrix dimensions (such as multiple runtime language versions and container images), CodeBuild automatically provisions parallel, isolated build runners for every combinatorial permutation and aggregates the results into a single consolidated batch execution report.

---

### Question 6: Sharing Dynamic State and Credentials Across Buildspec Lifecycle Phases
**Answer:** A  
**Explanation:**  
All lifecycle phases of an individual AWS CodeBuild execution (`install`, `pre_build`, `build`, `post_build`) run sequentially inside the **same container on a shared filesystem**. Files written to disk (e.g. `/tmp/token.txt`) or exported to persistent shell startup files (like `$BASH_ENV`) in `pre_build` remain immediately accessible to commands executing in `build` and `post_build`.

---

### Question 7: Reducing Cold Start Latency with AWS CodeBuild Lambda Compute
**Answer:** A  
**Explanation:**  
AWS CodeBuild supports **Lambda compute**. For fast, short-duration build jobs (such as unit tests, linters, or Go/Node.js builds), Lambda compute provisions execution environments in under 5 seconds, virtually eliminating the 45–60 second cold-start provisioning delays associated with standard EC2-based container environments.

---

### Question 8: Secondary Source Repositories & Multi-Repo Ingestion in CodeBuild
**Answer:** A  
**Explanation:**  
AWS CodeBuild supports **Multiple Input Sources and Secondary Artifacts**. In the project definition, secondary sources can be added with unique source identifiers (e.g. `shared_libs`). During execution, CodeBuild clones the secondary repository to a dedicated directory accessible via the environment variable `$CODEBUILD_SRC_DIR_<source_identifier>`, allowing multi-repository ingestion without custom git scripts.

---

### Question 9: Native Test Reporting and Code Coverage Integration in CodeBuild
**Answer:** A  
**Explanation:**  
CodeBuild provides native **Test Reporting** via the `reports:` stanza in `buildspec.yml`. By specifying report group ARNs, report formats (such as `JunitXml`, `NunitXml`, `CucumberJson`, or `CoberturaXml`), and the file paths to test output files, CodeBuild automatically parses test execution results and code coverage metrics, displaying visual trends, execution durations, and failure stack traces directly in the AWS Console.
