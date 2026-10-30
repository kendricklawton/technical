# Domain 7: HCP Terraform & Terraform Enterprise Administration — Hands-On Lab Exercises

## Lab 1: Multi-Region Orchestration with Terraform Stacks (`*.tfstack.hcl` & `*.tfdeploy.hcl`)

### Scenario
Author a Terraform Stack that dynamically provisions an AWS VPC and EKS component across a configurable set of AWS regions (`["us-east-1", "us-west-2"]`) using `for_each` on the Stack `provider` block, and configure an automated `orchestrate` approval rule for the `development` deployment.

### Step-by-Step Implementation
1. **Define Stack Variables & Dynamic Regional Providers (`providers.tfstack.hcl`):**
   ```hcl
   variable "regions" {
     type = set(string)
   }

   variable "identity_token" {
     type      = string
     ephemeral = true
   }

   required_providers {
     aws = {
       source  = "hashicorp/aws"
       version = "~> 5.0"
     }
   }

   # In Terraform Stacks (*.tfstack.hcl), provider blocks natively support for_each!
   provider "aws" "regional" {
     for_each = var.regions

     config {
       region = each.value
       assume_role_with_web_identity {
         role_arn           = "arn:aws:iam::123456789012:role/HCPTerraformStackRole"
         web_identity_token = var.identity_token
       }
     }
   }
   ```
2. **Define Stack Components (`components.tfstack.hcl`):**
   ```hcl
   component "vpc" {
     for_each = var.regions
     source   = "./modules/vpc"

     inputs = {
       region    = each.value
       cidr_block = "10.${index(tolist(var.regions), each.value) + 10}.0.0/16"
     }

     providers = {
       aws = provider.aws.regional[each.key]
     }
   }
   ```
3. **Define Deployments & Auto-Approval Orchestration (`deployments.tfdeploy.hcl`):**
   ```hcl
   identity_token "aws" {
     audience = ["aws.workload.identity"]
   }

   deployment "development" {
     inputs = {
       regions        = ["us-east-1"]
       identity_token = identity_token.aws.jwt
     }
   }

   deployment "production" {
     inputs = {
       regions        = ["us-east-1", "us-west-2"]
       identity_token = identity_token.aws.jwt
     }
   }

   # Auto-approve development plans ONLY if zero resources are being destroyed:
   orchestrate "auto_approve" "safe_dev_changes" {
     check {
       condition = context.plan.deployment == deployment.development
       reason    = "Auto-approval is restricted to the development deployment."
     }
     check {
       condition = context.plan.changes.remove == 0
       reason    = "Plans that remove resources require manual human approval."
     }
   }
   ```

---

## Lab 2: Deploying Self-Hosted `tfc-agent` on Kubernetes for Private Network Execution

### Scenario
Run a self-hosted HCP Terraform Agent pool inside a private Kubernetes cluster so HCP Terraform can provision internal-only resources without exposing internal endpoints to the public internet.

### Step-by-Step Implementation (`tfc-agent-deployment.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tfc-agent
  namespace: terraform-agents
spec:
  replicas: 2
  selector:
    matchLabels:
      app: tfc-agent
  template:
    metadata:
      labels:
        app: tfc-agent
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 999
      containers:
        - name: tfc-agent
          image: hashicorp/tfc-agent:1.18.0
          env:
            - name: TFC_AGENT_TOKEN
              valueFrom:
                secretKeyRef:
                  name: tfc-agent-Pool-secret
                  key: token
            - name: TFC_AGENT_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: TFC_AGENT_SINGLE
              value: "true" # Ephemeral single-job execution per pod lifecycle
          resources:
            requests:
              cpu: "500m"
              memory: "1Gi"
            limits:
              cpu: "2000m"
              memory: "4Gi"
```
