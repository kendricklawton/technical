# Domain 6: Monitoring, Logging & Runtime Security — Hands-On Exercises & Solutions

## Exercise 1: Configure Kubernetes API Server Audit Policy
Deploy `/etc/kubernetes/audit/policy.yaml` and configure `--audit-policy-file` on `kube-apiserver`.

## Exercise 2: Write Custom Falco Rule to Detect Terminal Shell Spawning
Add custom rule in `/etc/falco/falco_rules.local.yaml`:
```yaml
- rule: Shell Spawned in Container
  desc: Detect shell spawned inside container
  condition: container.id != host and proc.name in (bash, sh, zsh)
  output: Shell spawned in container (user=%user.name container=%container.id)
  priority: WARNING
```
Restart Falco: `sudo systemctl restart falco`.
