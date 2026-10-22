# Domain 7: HCP Terraform / Enterprise Operations — Production Exercises & Labs

---

## Lab 1: Deploying and Registering a Self-Hosted `tfc-agent` in a Private VPC

### Objective
Deploy a containerized `tfc-agent` runner inside an isolated private subnet with no public IP address, register it with an HCP Terraform Agent Pool, and configure a workspace to execute plans and applies using this agent.

```bash
# 1. Pull the official HashiCorp agent container:
docker pull hashicorp/tfc-agent:latest

# 2. Run the agent with registration token and pool name:
docker run -d \
  --name tfc-agent-primary \
  -e TFC_AGENT_TOKEN="your-agent-pool-token-here" \
  -e TFC_AGENT_NAME="internal-vpc-agent-01" \
  hashicorp/tfc-agent:latest

# 3. Check agent logs to verify registration:
docker logs tfc-agent-primary
# Output: Successfully registered agent with pool: private-vpc-pool
```

### Workspace Configuration in HCL (`main.tf`)
```hcl
terraform {
  cloud {
    organization = "enterprise-platform"

    workspaces {
      name = "private-database-tier"
    }
  }
}
```
*(In HCP Terraform workspace settings, set **Execution Mode** to **Agent** and select `private-vpc-pool`.)*

---

## Lab 2: Managing Shared Variables with Project-Scoped Variable Sets

### Objective
Create a Variable Set containing common AWS region, corporate billing tags, and dynamic OIDC credentials, and attach it to an entire Project so all current and future child workspaces inherit the configuration automatically.
