# Domain 4: CI/CD and Automation Workflows — Production Exercises & Labs

---

## Lab 1: End-to-End GitHub Actions Pipeline with OIDC and Speculative Plans

### Objective
Create an automated GitHub Actions workflow that executes speculative plans on Pull Requests and applies changes on merge to `main` using short-lived AWS OIDC tokens.

```yaml
# .github/workflows/terraform.yml
name: "Terraform CI/CD"

on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]

permissions:
  id-token: write
  contents: read
  pull-requests: write

jobs:
  plan:
    name: "Terraform Plan & Validate"
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.15.0

      - name: Terraform Format Check
        run: terraform fmt -check

      - name: Terraform Init
        run: terraform init

      - name: Terraform Validate
        run: terraform validate

      - name: Terraform Plan
        id: plan
        run: terraform plan -no-color -out=tfplan
        continue-on-error: false

      - name: Comment Plan on PR
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const output = `#### Terraform Format and Style 🖌\`${{ steps.plan.outcome }}\`
            #### Terraform Initialization ⚙️\`success\`
            #### Terraform Plan 📖\`${{ steps.plan.outcome }}\`

            *Pushed by: @${{ github.actor }}*`;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: output
            })

  apply:
    name: "Terraform Apply"
    needs: plan
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.15.0

      - run: terraform init
      - run: terraform apply -auto-approve
```
