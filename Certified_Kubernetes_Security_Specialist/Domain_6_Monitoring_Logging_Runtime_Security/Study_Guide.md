# Domain 6: Monitoring, Logging & Runtime Security — In-Depth Master Study Guide

**Exam Weight:** 20% of CKS  
**Target Mastery:** Kubernetes API Server Audit Logging, Runtime Threat Detection with Falco, eBPF & Tetragon syscall tracing, ensuring container immutability, and live compromise forensics.

---

## 1. Kubernetes API Server Audit Logging

Audit logs provide a chronological record of calls made to the Kubernetes API server.

### The 4 Audit Stages:
1. `RequestReceived`: Logged immediately when the request is received, before being sent to authorization/admission plugins.
2. `ResponseStarted`: Logged once the response headers are sent (used for long-running streaming requests like `watch` and `exec`).
3. `ResponseComplete`: Logged when the response body is completed.
4. `Panic`: Logged when an unhandled panic occurred during execution.

### The 4 Audit Levels:
- **`None`**: Do not log events matching this rule.
- **`Metadata`**: Log request metadata (requesting user, timestamp, resource, verb, namespace) but NOT request or response body.
- **`Request`**: Log metadata and request body, but NOT response body.
- **`RequestResponse`**: Log metadata, request body, and response body.

### Production Audit Policy (`/etc/kubernetes/audit/policy.yaml`)
```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
  # 1. Do not log read-only queries to noisy endpoints
  - level: None
    resources:
    - group: ""
      resources: ["endpoints", "configmaps"]
    verbs: ["get", "list", "watch"]

  # 2. Never log secret content in plaintext! Log metadata only
  - level: Metadata
    resources:
    - group: ""
      resources: ["secrets"]

  # 3. Log pod exec, attach, and port-forward commands at RequestResponse
  - level: RequestResponse
    resources:
    - group: ""
      resources: ["pods/exec", "pods/attach", "pods/portforward"]

  # 4. Catch-all: Log all other cluster requests at Metadata
  - level: Metadata
    omitStages:
      - RequestReceived
```

### Kube-apiserver Flags for Audit Logging:
```text
--audit-policy-file=/etc/kubernetes/audit/policy.yaml
--audit-log-path=/var/log/kubernetes/audit/audit.log
--audit-log-maxage=30
--audit-log-maxbackup=10
--audit-log-maxsize=100
```

---

## 2. Runtime Threat Detection with Falco

Falco leverages Linux kernel modules or eBPF probes to detect abnormal behavioral patterns in real time.

### Falco Configuration Files:
- Global engine settings: `/etc/falco/falco.yaml`
- Upstream base rules: `/etc/falco/falco_rules.yaml`
- Custom local overrides: `/etc/falco/falco_rules.local.yaml`

### Writing Custom Falco Rules
A Falco rule consists of:
- `rule`: Unique human-readable name.
- `desc`: Detailed description of the threat.
- `condition`: Filtering expression evaluating system calls.
- `output`: Alert message formatted with field extractors (`%user.name`, `%proc.name`, `%container.id`).
- `priority`: `EMERGENCY`, `ALERT`, `CRITICAL`, `ERROR`, `WARNING`, `NOTICE`, `INFO`, `DEBUG`.

```yaml
- rule: Unauthorized Sensitive File Read
  desc: Detect any process reading /etc/shadow or /etc/sudoers inside a container
  condition: >
    open_read and container and
    (fd.name in (/etc/shadow, /etc/sudoers, /etc/pam.d))
  output: >
    Sensitive credential file read (user=%user.name file=%fd.name command=%proc.cmdline container_id=%container.id image=%container.image.repository)
  priority: CRITICAL
  tags: [security, credentials, mitre_credential_access]

- rule: Terminal Shell Spawned in Container
  desc: Detect shell spawned inside container
  condition: >
    spawned_process and container and
    proc.name in (bash, sh, zsh, dash, ksh)
  output: >
    Shell spawned in container (user=%user.name command=%proc.cmdline container=%container.name image=%container.image.repository)
  priority: WARNING
```

### Managing Falco Service:
```bash
# Validate rules syntax
sudo falco -V /etc/falco/falco_rules.local.yaml

# Restart and inspect logs
sudo systemctl restart falco
sudo journalctl -u falco -f
```

---

## 3. Incident Forensics on Compromised Containers

When an alert fires for a compromised pod:
1. **Isolate without terminating:**
   - Modify the pod's labels so the Service stops routing traffic to it:
     `kubectl label pod compromised-pod app=quarantine --overwrite`
2. **Inspect Process Hierarchy:**
   Find the PID on the host node via `crictl inspect`:
   ```bash
   CONTAINER_ID=$(sudo crictl ps --name app -q)
   PID=$(sudo crictl inspect $CONTAINER_ID | jq .info.pid)
   ls -la /proc/$PID/root/
   ```
3. **Capture Volatile Artifacts:**
   Dump `/proc/$PID/cmdline`, `/proc/$PID/environ`, and export memory before deleting the pod.
