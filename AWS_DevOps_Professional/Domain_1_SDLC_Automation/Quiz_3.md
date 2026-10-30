# Domain 1 (SDLC Automation) — Quiz 3: CodeDeploy Deep Dive

**Topic:** AWS CodeDeploy Deployment Configurations, AppSpec Lifecycles & Multi-Compute Support  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** 9 Staff-Level Scenario-Based Questions with In-Depth Architectural Explanations  
**Design Standard:** Balanced option lengths, rigorous constraints, and zero obvious giveaway distractors.

---

### Question 1: CodeDeploy Host Agent Diagnostic Logging and Script Failure Analysis

An in-place deployment to an Auto Scaling Group of Amazon EC2 instances fails during the `ApplicationStart` lifecycle event hook. In the AWS CodeDeploy Management Console, the instance deployment status displays `ScriptFailed` with an exit code of `1`, but the console transcript truncates the underlying stdout and stderr diagnostic traces.

**On an Amazon Linux 2023 EC2 instance, where are the CodeDeploy agent daemon logs and individual lifecycle hook script execution logs located on disk?**
- A. The agent daemon operations log is at `/var/log/aws/codedeploy-agent/codedeploy-agent.log`, and the lifecycle hook script execution outputs are at `/opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-deployments.log`.
- B. The agent operational messages are written to `/var/log/messages`, and the user script execution outputs are captured in `/var/log/cloud-init-output.log`.
- C. The agent daemon logs are streamed directly to the Amazon CloudWatch log group `/aws/ec2/codedeploy`, and local script output files are deleted upon hook failure.
- D. Both daemon operations and lifecycle script outputs are concatenated into a single flat file located at `/etc/aws/codedeploy/agent-execution.log`.

---

### Question 2: Custom Deployment Configurations for Large-Scale Production Fleets

An enterprise platform runs 250 Amazon EC2 instances inside an Auto Scaling Group. Standard predefined deployment configurations (`CodeDeployDefault.OneAtATime` and `CodeDeployDefault.HalfAtATime`) fail business requirements: `OneAtATime` is too slow (taking over 4 hours), while `HalfAtATime` drops 50% of production capacity, risking request throttling. The architectural board mandates that deployments must update the fleet in batches where **AT LEAST 80% OF CAPACITY (200 INSTANCES)** remains healthy and serving traffic at all times.

**How should the DevOps Engineer configure CodeDeploy to enforce this deployment constraint?**
- A. Create a **Custom Deployment Configuration** using `aws deploy create-deployment-config --deployment-config-name TwentyPercentAtATime --minimum-healthy-hosts "type=FLEET_PERCENT,value=80"`.
- B. Configure the Auto Scaling Group `UpdatePolicy` with `AutoScalingRollingUpdate` specifying `MinInstancesInService: 200` and `MaxBatchSize: 50`.
- C. Split the fleet of 250 instances across 5 separate CodeDeploy Deployment Groups, and orchestrate them sequentially using an AWS Step Functions state machine.
- D. Create an Amazon EventBridge rule that intercepts deployment lifecycle events and dynamically updates the ASG `DesiredCapacity` to maintain 80% available instances.

---

### Question 3: CodeDeploy AppSpec Execution Context for Application Teardown Hooks

An in-place deployment to an EC2 Auto Scaling Group fails during the `ApplicationStop` lifecycle event hook. An investigation reveals that the bash script specified under `ApplicationStop` was renamed and moved into a new directory structure in the latest Git commit. The CodeDeploy agent reports that it cannot find the file at the old filepath.

**Why does CodeDeploy reference the old filepath during `ApplicationStop`, and how does its lifecycle sequencing function?**
- A. `ApplicationStop`, `BeforeBlockTraffic`, `BlockTraffic`, and `AfterBlockTraffic` execute scripts from the **previously installed application revision**, because the new revision bundle has not yet been downloaded (`DownloadBundle` occurs afterward).
- B. The CodeDeploy agent caches the local filesystem state from the initial EC2 launch and requires an instance reboot to invalidate the AppSpec file cache.
- C. The `appspec.yml` parser parses the files section in reverse alphabetical order, executing scripts in the parent directory before evaluating child directory paths.
- D. CodeDeploy executes `ApplicationStop` on the deployment host by cloning the Git repository directly from the remote origin rather than reading the deployment bundle.

---

### Question 4: ECS Blue/Green Deployments with Synthetic Test Traffic Shifting

An enterprise microservice deploys to Amazon ECS on AWS Fargate using AWS CodeDeploy (`ECSBlueGreen`). The Application Load Balancer is provisioned with two listeners: a Production Listener on Port 443 and a Test Listener on Port 8443. The DevOps team needs an automated integration test suite to validate database read/write integrity against the replacement task set over Port 8443 *before* any production user traffic shifts to the new version.

**Which AppSpec lifecycle hook must be configured to invoke the test suite Lambda function?**
- A. `AfterAllowTestTraffic`: CodeDeploy routes traffic from the test listener (Port 8443) to the replacement task set, invokes the hook Lambda function, and waits for `PutLifecycleEventHookExecutionStatus` before shifting production traffic.
- B. `BeforeInstall`: CodeDeploy invokes the hook Lambda function to test the ECS task definition schema before provisioning replacement tasks.
- C. `AfterInstall`: CodeDeploy provisions the replacement tasks and runs the hook Lambda function before any ALB listeners are associated.
- D. `BeforeAllowTraffic`: CodeDeploy shifts production traffic to 100% and invokes the hook Lambda function to verify live application error rates.

---

### Question 5: CodeDeploy Tag Filtering and Complex Deployment Group Scoping

An enterprise operations team uses AWS CodeDeploy to update standalone EC2 instances across multiple business units. A specific production deployment group must deploy software only to EC2 instances that satisfy two simultaneous tagging conditions: the instance must have the tag `Environment=Production` AND the tag `Role=PaymentGateway`.

**How must the CodeDeploy deployment group tag filters be declared?**
- A. Configure **Multiple Tag Groups**: Define `Environment=Production` in Tag Group 1, and define `Role=PaymentGateway` in Tag Group 2, enforcing logical **AND** evaluation across groups.
- B. Add both tags to a single Tag Group in the deployment group configuration, which evaluates tags with native logical **AND** condition logic.
- C. Tag filtering in CodeDeploy only supports logical **OR** matching; instances must be moved into a dedicated Auto Scaling Group to filter by multiple tags.
- D. Attach an IAM condition to the CodeDeploy Service Role that evaluates `aws:ResourceTag/Environment` and `aws:ResourceTag/Role` during deployment execution.

---

### Question 6: Declarative Lambda AppSpec Specification Syntax & Version Mapping

An engineer is automating AWS Lambda serverless deployments with traffic shifting using AWS CodeDeploy and the AWS CLI. The deployment must shift traffic from Version 1 to Version 2 on the alias `live`.

**Which syntax accurately represents a valid `appspec.yaml` declaration for an AWS Lambda deployment?**
- A.
  ```yaml
  version: 0.0
  Resources:
    - TargetFunction:
        Type: AWS::Lambda::Function
        Properties:
          Name: "paymentProcessor"
          Alias: "live"
          CurrentVersion: "1"
          TargetVersion: "2"
  ```
- B.
  ```yaml
  version: 1.0
  Deployment:
    Function:
      Name: "paymentProcessor"
      SourceVersion: "1"
      DestinationVersion: "2"
      RoutingConfig:
        Alias: "live"
  ```
- C.
  ```yaml
  version: 0.0
  Resources:
    - Type: AWS::Serverless::Function
      Properties:
        FunctionName: "paymentProcessor"
        AutoPublishAlias: "live"
        DeploymentPreference: "Canary10Percent5Minutes"
  ```
- D.
  ```yaml
  version: 2.0
  ComputePlatform: Lambda
  Functions:
    - FunctionName: "paymentProcessor"
      TargetAlias: "live"
      ShiftPercentage: 10
  ```

---

### Question 7: CodeDeploy Automated Rollback Triggers with CloudWatch Alarms

A critical microservice is deployed across an Auto Scaling Group using AWS CodeDeploy. The platform team configures automated rollbacks to protect against subtle software regressions that pass systemd service checks but cause high 5xx error rates on the Application Load Balancer.

**How does AWS CodeDeploy execute automated rollbacks when configured with CloudWatch Alarms?**
- A. CodeDeploy continuously polls the configured CloudWatch Alarms; if any alarm enters the `ALARM` state during deployment or while traffic is shifting, CodeDeploy immediately halts forward progression, marks the deployment as failed, and deploys the last known good revision bundle to all targets.
- B. CodeDeploy sends an SNS notification to the Auto Scaling Group, which immediately terminates all instances and rebuilds the fleet from the base Launch Template.
- C. CodeDeploy stops sending new HTTP requests to the Application Load Balancer and prompts the operator in the AWS Console to manually select a prior commit SHA.
- D. CodeDeploy pauses the deployment for a 60-minute bake time, and if alarms remain active, it executes an inverse rolling deployment that rolls back one instance every 24 hours.

---

### Question 8: CodeDeploy Agent Security Architecture in Private Isolated Subnets

An enterprise deploys EC2 instances into private isolated subnets with zero internet access, no NAT Gateway, and no public IP addresses. An automated CodeDeploy in-place deployment fails because the CodeDeploy agent installed on the instances cannot communicate with the AWS CodeDeploy control plane.

**Which architectural components are required to allow the CodeDeploy agent to operate within a private isolated subnet?**
- A. Provision **AWS PrivateLink VPC Endpoints** for `com.amazonaws.<region>.codedeploy` and `com.amazonaws.<region>.codedeploy-commands-secure`, alongside VPC endpoints for Amazon S3 (Gateway Endpoint) and Amazon CloudWatch Logs (Interface Endpoint).
- B. Configure an egress-only Internet Gateway in the private subnet route table, and allow outbound traffic on TCP port 22 in the EC2 Security Group.
- C. Open inbound TCP port 80 and port 443 in the EC2 instance Security Group to allow the CodeDeploy control plane to push deployment bundles directly into the host.
- D. Attach an IAM User access key with administrator permissions directly to the CodeDeploy agent configuration file at `/etc/codedeploy/credentials`.

---

### Question 9: Overriding Broken ApplicationStop Scripts via AWS CLI

An in-place deployment to an EC2 fleet fails at the `ApplicationStop` lifecycle event hook because the previous deployment revision contained a syntax error in its stop script. Because `ApplicationStop` runs the script from the *previous* revision, every subsequent deployment attempt fails immediately at `ApplicationStop`, blocking engineers from shipping the corrected codebase.

**How can the DevOps Engineer force CodeDeploy to deploy the fixed application revision without terminating the EC2 instances?**
- A. Trigger the new deployment via the AWS CLI using `aws deploy create-deployment` with the **`--ignore-application-stop-failures`** parameter enabled.
- B. Stop the CodeDeploy agent service on all EC2 instances and restart it using `systemctl restart codedeploy-agent --clean-cache`.
- C. Delete the previous deployment archive from the Amazon S3 artifact bucket and re-upload the new bundle with the identical S3 key.
- D. Manually terminate all EC2 instances in the Auto Scaling Group and allow Auto Scaling to launch fresh instances running the new revision.

---

## Answer Key & Detailed Explanations

### Question 1: CodeDeploy Host Agent Diagnostic Logging and Script Failure Analysis
**Answer:** A  
**Explanation:**  
On Linux instances running the AWS CodeDeploy agent, log files are separated into two distinct locations:  
1. `/var/log/aws/codedeploy-agent/codedeploy-agent.log`: Captures the daemon's operational activity, registration with the CodeDeploy control plane, and lifecycle polling state.  
2. `/opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-deployments.log`: Captures standard output (stdout) and standard error (stderr) generated by user-defined scripts executed during lifecycle event hooks (`ApplicationStart`, `ValidateService`, etc.).

---

### Question 2: Custom Deployment Configurations for Large-Scale Production Fleets
**Answer:** A  
**Explanation:**  
AWS CodeDeploy supports **Custom Deployment Configurations** via `aws deploy create-deployment-config`. By defining `minimum-healthy-hosts` with `type=FLEET_PERCENT,value=80`, CodeDeploy automatically calculates batch sizes to ensure that at least 80% of the fleet remains healthy and active throughout the deployment, preventing throughput degradation without manual Step Functions orchestration.

---

### Question 3: CodeDeploy AppSpec Execution Context for Application Teardown Hooks
**Answer:** A  
**Explanation:**  
In CodeDeploy in-place deployments, before new code files can be extracted onto the host, the currently running application process must be stopped. Therefore, the pre-download hooks (`ApplicationStop`, `BeforeBlockTraffic`, `BlockTraffic`, `AfterBlockTraffic`) execute using the `appspec.yml` and scripts from the **previously deployed application revision**. The new revision bundle is only downloaded during the subsequent `DownloadBundle` step.

---

### Question 4: ECS Blue/Green Deployments with Synthetic Test Traffic Shifting
**Answer:** A  
**Explanation:**  
In Amazon ECS blue/green deployments, CodeDeploy provides the **`AfterAllowTestTraffic`** lifecycle hook. After CodeDeploy associates the ALB Test Listener (Port 8443) with the replacement task set, it triggers the Lambda function attached to `AfterAllowTestTraffic`. The test function executes integration suites against the test port. Once tests pass, the Lambda function calls `codedeploy:PutLifecycleEventHookExecutionStatus` with status `Succeeded`, allowing CodeDeploy to proceed to shift production traffic.

---

### Question 5: CodeDeploy Tag Filtering and Complex Deployment Group Scoping
**Answer:** A  
**Explanation:**  
In CodeDeploy Deployment Groups:  
- Tags defined within the **same** Tag Group are evaluated with logical **OR** logic.  
- Tags defined across **multiple** Tag Groups are evaluated with logical **AND** logic.  
To ensure that target instances possess both `Environment=Production` AND `Role=PaymentGateway`, the deployment group must be configured with two separate Tag Groups.

---

### Question 6: Declarative Lambda AppSpec Specification Syntax & Version Mapping
**Answer:** A  
**Explanation:**  
For AWS Lambda deployments, the CodeDeploy `appspec.yaml` requires `version: 0.0` and a `Resources` section containing a `TargetFunction` of type `AWS::Lambda::Function`. The `Properties` mapping must declare `Name`, `Alias`, `CurrentVersion`, and `TargetVersion`. Option C represents an AWS SAM template, which synthesizes this AppSpec under the hood, but is not the AppSpec file itself.

---

### Question 7: CodeDeploy Automated Rollback Triggers with CloudWatch Alarms
**Answer:** A  
**Explanation:**  
CodeDeploy features native **Deployment Alarms**. When CloudWatch Alarms are associated with a Deployment Group, CodeDeploy monitors their status during the deployment. If an alarm transitions to `ALARM`, CodeDeploy automatically stops forward execution and triggers an automated rollback, deploying the last known good revision to restore service stability immediately.

---

### Question 8: CodeDeploy Agent Security Architecture in Private Isolated Subnets
**Answer:** A  
**Explanation:**  
The CodeDeploy agent uses an outbound polling model, querying AWS endpoints over HTTPS (port 443). In private isolated subnets without internet access, the VPC must have **VPC Interface Endpoints** for `codedeploy` and `codedeploy-commands-secure`, alongside an **S3 Gateway Endpoint** (to download deployment bundles) and CloudWatch Logs endpoints. CodeDeploy never requires inbound open ports.

---

### Question 9: Overriding Broken ApplicationStop Scripts via AWS CLI
**Answer:** A  
**Explanation:**  
Because `ApplicationStop` runs the script from the *previous* deployment bundle, a bug in that old script will cause every subsequent deployment attempt to fail at `ApplicationStop`. To overcome this deadlock without terminating instances, engineers can create a deployment using the AWS CLI or SDK with the **`--ignore-application-stop-failures`** flag, instructing CodeDeploy to ignore script errors in `ApplicationStop` and proceed with the deployment.
