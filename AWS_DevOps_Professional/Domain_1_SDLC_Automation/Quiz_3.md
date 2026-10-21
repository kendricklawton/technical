# Domain 1 (SDLC Automation) — Quiz 3: CodeDeploy Deep Dive

**Topic:** AWS CodeDeploy Deployment Configurations, AppSpec Lifecycles & Multi-Compute Support  
**Exam:** AWS Certified DevOps Engineer - Professional (DOP-C02)  
**Format:** Scenario-Based Multiple Choice Questions with Explanations

---

### Question 1: CodeDeploy Agent Logging on Linux Hosts

An in-place deployment to an Amazon EC2 instance fails during the `ApplicationStart` lifecycle hook. The AWS Management Console displays `ScriptFailed` with no additional diagnostic details.

**On the Linux EC2 host, where are the CodeDeploy agent logs and specific script execution logs located?**
- A. `/var/log/messages` and `/var/log/cloud-init.log`
- B. `/var/log/aws/codedeploy-agent/codedeploy-agent.log` for agent operations, and `/opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-deployments.log` for script outputs.
- C. `/etc/codedeploy/logs.txt`
- D. In the Amazon CloudWatch `/aws/lambda/codedeploy` log group.

---

### Question 2: Custom Deployment Configurations for High-Capacity Fleets

A company runs 200 EC2 instances in an Auto Scaling Group. Standard predefined configurations (`CodeDeployDefault.OneAtATime` and `CodeDeployDefault.HalfAtATime`) do not meet business requirements: the company needs deployments to update exactly 20% of the fleet at a time, ensuring that at least 80% of capacity remains available throughout the deployment.

**How can the DevOps Engineer configure this custom deployment rate?**
- A. Update the Auto Scaling Group minimum size to 80%.
- B. Create a **Custom Deployment Configuration** using `aws deploy create-deployment-config --deployment-config-name TwentyPercentAtATime --minimum-healthy-hosts "type=FLEET_PERCENT,value=80"`.
- C. Split the fleet into 5 separate deployment groups.
- D. Write an AWS Lambda function to terminate 20 instances sequentially.

---

### Question 3: CodeDeploy AppSpec Hook for Stopping Old Versions

An in-place deployment fails during the `ApplicationStop` hook on an EC2 instance. The engineer inspects the host and finds that the script referenced in `ApplicationStop` does not exist because the file was renamed in the new source revision.

**Why does `ApplicationStop` reference scripts from the old revision rather than the new revision?**
- A. CodeDeploy has a bug where files are mixed up.
- B. By design, the `ApplicationStop`, `BeforeBlockTraffic`, `BlockTraffic`, and `AfterBlockTraffic` lifecycle hooks execute scripts from the **previously installed application revision**, because the new revision bundle has not yet been downloaded (`DownloadBundle` occurs afterward).
- C. The CodeDeploy agent must be restarted to download the new revision.
- D. `ApplicationStop` only runs on Windows instances.

---

### Question 4: ECS Blue/Green Deployments with Test Traffic Routing

An engineer configures an Amazon ECS Blue/Green deployment using AWS CodeDeploy. The ALB has two listeners: Port 443 (Production) and Port 8443 (Test). The engineer wants automated integration tests to run against the replacement task set over Port 8443 before production users receive any traffic.

**Which AppSpec lifecycle hook executes test scripts against the test listener?**
- A. `BeforeInstall`
- B. `AfterAllowTestTraffic`
- C. `AfterAllowTraffic`
- D. `ValidateService`

---

### Question 5: CodeDeploy Tag Filtering and Deployment Group Scoping

An operations team manages EC2 instances shared across multiple teams. A deployment group must target only instances that have both the tag `Environment=Production` AND the tag `Role=API-Server`.

**How should the CodeDeploy deployment group tag filters be configured?**
- A. Add both tags to a single tag group in the deployment group configuration.
- B. Add `Environment=Production` in Tag Group 1, and add `Role=API-Server` in Tag Group 2 (**Multiple Tag Groups**).
- C. Tag filtering only supports OR logic, so instances must be grouped into separate ASGs.
- D. Use Name tags only.

---

### Question 6: Lambda Deployment AppSpec Specification Format

When deploying an AWS Lambda function using AWS CodeDeploy, how is the target function revision specified in the `appspec.yaml`?

**Which snippet accurately reflects a valid Lambda AppSpec definition?**
- A.
  ```yaml
  version: 0.0
  Resources:
    - TargetFunction:
        Type: AWS::Lambda::Function
        Properties:
          Name: "myFunction"
          Alias: "live"
          CurrentVersion: "1"
          TargetVersion: "2"
  ```
- B.
  ```yaml
  version: 1.0
  Lambda:
    FunctionName: "myFunction"
    Version: "2"
  ```
- C. Lambda does not use AppSpec files; it uses SAM templates exclusively.
- D.
  ```yaml
  hooks:
    - RunLambda: myFunction:2
  ```

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

### Question 1: CodeDeploy Agent Logging on Linux Hosts

**Answer:** B  
**Explanation:** 
On Linux instances running the CodeDeploy agent:
- `/var/log/aws/codedeploy-agent/codedeploy-agent.log`: Records agent lifecycle events, daemon status, and AWS API polling.
- `/opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-deployments.log`: Contains standard output and standard error from scripts defined in `appspec.yml` hooks.

---

### Question 2: Custom Deployment Configurations for High-Capacity Fleets

**Answer:** B  
**Explanation:** 
AWS CodeDeploy allows creating custom deployment configurations using `create-deployment-config`. Specifying `minimum-healthy-hosts` with `type=FLEET_PERCENT,value=80` (or `type=HOST_COUNT,value=160`) ensures CodeDeploy updates instances in batches while guaranteeing that 80% of the fleet remains healthy and active.

---

### Question 3: CodeDeploy AppSpec Hook for Stopping Old Versions

**Answer:** B  
**Explanation:** 
In CodeDeploy in-place deployments, before new code can be installed, the currently running application must be stopped. Therefore, `ApplicationStop` executes using the scripts from the **previous successful deployment's AppSpec**. If an earlier deployment left a broken or missing stop script, subsequent deployments will fail at `ApplicationStop` until fixed or overridden.

---

### Question 4: ECS Blue/Green Deployments with Test Traffic Routing

**Answer:** B  
**Explanation:** 
For Amazon ECS blue/green deployments, CodeDeploy defines specific hooks:
1. `BeforeInstall`
2. `AfterInstall`
3. `AfterAllowTestTraffic`: Invoked after test traffic is enabled on the test listener (Port 8443) but *before* production traffic (Port 443) is shifted. An AWS Lambda function attached to this hook runs automated tests against the replacement task set.
4. `BeforeAllowTraffic`
5. `AfterAllowTraffic`

---

### Question 5: CodeDeploy Tag Filtering and Deployment Group Scoping

**Answer:** B  
**Explanation:** 
In CodeDeploy deployment groups:
- Tags within the **same** tag group are evaluated with **OR** logic.
- Tags across **different** tag groups are evaluated with **AND** logic.
To require that an instance possesses both `Environment=Production` AND `Role=API-Server`, the deployment group must define two separate Tag Groups.

---

### Question 6: Lambda Deployment AppSpec Specification Format

**Answer:** A  
**Explanation:** 
For AWS Lambda deployments, the CodeDeploy AppSpec file requires `version: 0.0` and a `Resources` block with `TargetFunction` of type `AWS::Lambda::Function`. Properties must specify the function `Name`, `Alias` (e.g. `live`), `CurrentVersion`, and `TargetVersion`.

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

