# Domain 6: Monitoring, Logging & Runtime Security — Hands-On Exercises & Solutions

## Exercise 1: Multi-Tier Kubernetes API Server Audit Logging
### Scenario
Configure enterprise audit logging on `kube-apiserver` capturing full request/response bodies for security-critical resources while filtering high-volume telemetry noise.

```yaml
# /etc/kubernetes/audit/audit-policy.yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
  # 1. Do not log high-volume endpoints or proxy requests
  - level: None
    users: ["system:kube-proxy"]
  - level: None
    verbs: ["get", "watch", "list"]
    resources:
      - group: ""
        resources: ["endpoints", "endpointslices"]

  # 2. Log full request and response for secrets, configmaps, and token requests
  - level: RequestResponse
    resources:
      - group: ""
        resources: ["secrets", "configmaps", "serviceaccounts/token"]

  # 3. Log metadata for workload mutations
  - level: Metadata
    verbs: ["create", "update", "patch", "delete"]
    resources:
      - group: "apps"
        resources: ["deployments", "daemonsets", "statefulsets"]

  # 4. Catch-all metadata for audit completeness
  - level: Metadata
```

---

## Exercise 2: Real-Time Threat Detection with Falco Custom Rules
### Scenario
Author custom Falco rules in `/etc/falco/falco_rules.local.yaml` to detect rogue interactive shell spawning and crypto-mining binaries inside container runtimes.

```yaml
- rule: Rogue Interactive Shell Spawned in Container
  desc: Detect interactive shell execution inside container
  condition: >
    spawned_process and container and
    shell_procs and proc.tty != 0 and
    not user_known_shell
  output: >
    ALERT: Shell spawned in container (user=%user.name pod=%k8s.pod.name
    container=%k8s.container.name cmd=%proc.cmdline tty=%proc.tty)
  priority: CRITICAL
  tags: [mitre_execution, container, shell]

- rule: Cryptocurrency Mining Process Detected
  desc: Detect common cryptocurrency mining binary execution
  condition: >
    spawned_process and container and
    (proc.name in (xmrig, minerd, cpuminer) or proc.cmdline contains "stratum+tcp")
  output: >
    CRITICAL: Miner process executed (user=%user.name pod=%k8s.pod.name cmd=%proc.cmdline)
  priority: EMERGENCY
  tags: [malware, cryptomining]
```

---

## Exercise 3: In-Kernel Runtime Enforcement with Tetragon eBPF
### Scenario
Deploy a Cilium Tetragon `TracingPolicy` to monitor and terminate (`Sigkill`) any process executing binary `/usr/bin/nsenter` or calling `sys_ptrace` to inspect other processes.

```yaml
apiVersion: cilium.io/v1alpha1
kind: TracingPolicy
metadata:
  name: block-nsenter-and-ptrace
spec:
  kprobes:
    - call: "sys_execve"
      syscall: true
      args:
        - index: 0
          type: "string"
      selectors:
        - matchArgs:
            - index: 0
              operator: "Equal"
              values: ["/usr/bin/nsenter", "/bin/nsenter"]
          matchActions:
            - action: Sigkill
    - call: "sys_ptrace"
      syscall: true
      matchActions:
        - action: Sigkill
```

---

## Exercise 4: Automated Pod Threat Containment & Network Quarantine
### Scenario
When a container is flagged for intrusion, execute immediate network quarantine using an ingress/egress deny-all NetworkPolicy without killing the pod to preserve volatile evidence.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: quarantine-compromised-workload
  namespace: prod
spec:
  podSelector:
    matchLabels:
      security.quarantine: "true"
  policyTypes:
  - Ingress
  - Egress
```
```bash
# Label the compromised pod to trigger the quarantine immediately
kubectl label pod compromised-pod-xyz -n prod security.quarantine=true --overwrite
```

---

## Exercise 5: Live Container Forensics & Volatile Memory Preservation
### Scenario
Extract forensic evidence from a running, isolated container without triggering anti-analysis or self-deletion scripts.

```bash
# 1. Identify container runtime ID
CONTAINER_ID=$(crictl ps --name compromised-app -q)

# 2. Inspect container process PID on host
HOST_PID=$(crictl inspect --output go-template --template '{{.info.pid}}' ${CONTAINER_ID})

# 3. Preserve process environment and memory maps
cp /proc/${HOST_PID}/environ /opt/forensics/environ.txt
cp /proc/${HOST_PID}/maps /opt/forensics/maps.txt
cp /proc/${HOST_PID}/status /opt/forensics/status.txt

# 4. Copy container root filesystem
tar -czf /opt/forensics/container-root.tar.gz -C /proc/${HOST_PID}/root .
```

---

## Exercise 6: Audit Log Analysis with jq
### Scenario
Query raw JSON audit logs from `/var/log/kubernetes/audit/audit.log` to identify unauthorized access attempts, anonymous queries, and secret extractions.

```bash
# 1. Find all 403 Forbidden requests
cat /var/log/kubernetes/audit/audit.log | jq 'select(.responseStatus.code == 403) | {timestamp: .requestReceivedTimestamp, user: .user.username, verb: .verb, uri: .requestURI}'

# 2. Find all secret reads by non-system users
cat /var/log/kubernetes/audit/audit.log | jq 'select(.objectRef.resource == "secrets" and .verb == "get" and (.user.username | startswith("system:") | not)) | {user: .user.username, secret: .objectRef.name, ns: .objectRef.namespace}'
```
