# Domain 4: CI/CD and Automation Workflows — Complete Study Guide

**Exam Weight:** 15% of the Terraform Infrastructure Engineer Professional (002) Exam  
**Target Mastery:** Automated GitOps pipelines, OpenID Connect (OIDC) dynamic provider credentials (zero static secrets), variable evaluation precedence, VCS-driven vs API-driven workflows, and concurrency control.

---

## 1. Dynamic Provider Credentials via OpenID Connect (OIDC)

Storing static cloud credentials (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, or GCP Service Account JSON keys) in CI/CD secret stores is a critical security vulnerability. If a pipeline runner or repository is compromised, static keys can be exfiltrated and abused indefinitely.

### 1.1 The OIDC Authentication Flow
```mermaid
sequenceDiagram
    participant CI as CI Runner (GitHub Actions / GitLab / HCP)
    participant OIDC as CI OIDC Provider
    participant STS as Cloud STS / Vault
    participant TF as Terraform Engine

    CI->>OIDC: 1. Request signed JSON Web Token (JWT) with claims (repo, branch, commit)
    OIDC-->>CI: 2. Return cryptographically signed JWT
    CI->>STS: 3. Exchange JWT via AssumeRoleWithWebIdentity (validates audience & subject)
    STS-->>CI: 4. Issue temporary (15 min - 1 hr) session credentials
    CI->>TF: 5. Execute terraform plan/apply with ephemeral credentials
```

### 1.2 GitHub Actions OIDC Configuration for AWS
```yaml
name: "Terraform Production Deployment"
on:
  push:
    branches: [ main ]

permissions:
  id-token: write # Required for requesting the JWT
  contents: read

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS Credentials via OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GitHubTerraformDeploymentRole
          aws-region: us-east-1

      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.15.0

      - run: terraform init
      - run: terraform plan -out=tfplan
      - run: terraform apply -auto-approve tfplan
```

---

## 2. Variable Evaluation Precedence in CI/CD

When debugging pipelines, you must know exactly which variable definition wins when variables are declared in multiple locations:

| Rank | Source | Mechanism / Scope |
| :---: | :--- | :--- |
| **1 (Lowest)** | Default in block | `variable "name" { default = "val" }` inside `.tf` code. |
| **2** | Environment Variables | `TF_VAR_name="val"` set in runner environment or shell. |
| **3** | `terraform.tfvars` | File in working directory automatically loaded. |
| **4** | `terraform.tfvars.json` | JSON format automatically loaded. |
| **5** | `*.auto.tfvars` / `.json` | Automatically loaded; if multiple exist, evaluated in **alphabetical filename order**. |
| **6** | `-var-file="path.tfvars"` | Explicitly passed CLI flag (later flags override earlier flags). |
| **7 (Highest)** | `-var="name=val"` | Explicit command-line override; overrides all other sources. |

---

## 3. Deployment Strategies: Blue/Green & Canary in Infrastructure

### 3.1 Blue/Green with DNS or Target Group Switching
```mermaid
flowchart LR
    Route53["Route 53 DNS / DigitalOcean Load Balancer"]
    Route53 -->|"Active 100% Traffic"| BlueCluster["Blue Environment (v1.0 Droplets/ASG)"]
    Route53 -.->|"Pending Validation"| GreenCluster["Green Environment (v2.0 Droplets/ASG)"]
```

1. Deploy the new (Green) environment alongside the existing (Blue) environment using distinct variable identifiers.
2. Run smoke tests and integration assertions against Green.
3. Switch the Load Balancer forwarding rule or DNS alias to point to Green.
4. Decommission Blue by running `terraform destroy` against the Blue workspace or by setting its replica count to 0.

### 3.2 Automated Speculative Plans on Pull Requests
Every pull request targeting the infrastructure repository should trigger an automated CI job that:
- Executes `terraform fmt -check`.
- Executes `terraform validate`.
- Executes `tflint` and security linters (Trivy / Checkov).
- Executes `terraform test` unit tests.
- Executes `terraform plan` (speculative plan) and posts the formatted plan summary as a comment on the PR.
