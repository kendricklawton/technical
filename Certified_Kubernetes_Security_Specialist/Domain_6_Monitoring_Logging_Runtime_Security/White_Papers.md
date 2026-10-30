# Domain 6: Monitoring, Logging and Runtime Security — Essential Security Benchmarks & Technical Documentation Guide

> **Blueprint Weight:** 20% of Certified Kubernetes Security Specialist (CKS)  
> **Core Objective:** Detect and respond to runtime security threats, analyze Kubernetes API audit logs, configure Falco rules to alert on malicious behavior, and perform active threat containment and forensics.

---

## 1. Authoritative Security Standards & Documentation

### 1.1 [Kubernetes Auditing & Audit Policy Rules](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/)
* **Canonical Scope:** Recording chronological sequences of actions affecting cluster state.
* **Audit Stages:**
  * `RequestReceived`, `ResponseStarted`, `ResponseComplete`, `Panic`.
* **Audit Levels:**
  * `None`: Do not log events.
  * `Metadata`: Log request metadata (user, timestamp, resource, namespace), but no request or response body.
  * `Request`: Log metadata and request body, but not response body.
  * `RequestResponse`: Log metadata, request body, and response body.
* **Sample Audit Policy (`/etc/kubernetes/audit-policy.yaml`):**
  ```yaml
  apiVersion: audit.k8s.io/v1
  kind: Policy
  rules:
  - level: Metadata
    resources:
    - group: ""
      resources: ["secrets", "configmaps"]
  - level: RequestResponse
    users: ["admin"]
    resources:
    - group: ""
      resources: ["pods"]
  - level: None
    users: ["system:kube-proxy"]
  ```

### 1.2 [Falco Runtime Security Engine](https://falco.org/docs/)
* **Canonical Scope:** Behavioral activity monitoring and threat detection at the Linux kernel level via eBPF.
* **Architecture & Rules:**
  * Configuration in `/etc/falco/falco.yaml`.
  * Custom rules in `/etc/falco/falco_rules.local.yaml`.
  * Rule Structure:
    ```yaml
    - rule: Terminal Shell Spawned in Container
      desc: Detect shell execution inside container
      condition: >
        spawned_process and container
        and shell_procs and not user_known_shell
      output: >
        Shell spawned in container (user=%user.name pod=%k8s.pod.name
        container=%k8s.container.name process=%proc.name cmdline=%proc.cmdline)
      priority: WARNING
      tags: [container, shell, mitre_execution]
    ```

### 1.3 [Tetragon eBPF Security Observability & Enforcement](https://tetragon.io/docs/)
* **Canonical Scope:** In-kernel threat detection and immediate process kill capabilities using Cilium eBPF.

---

## 2. High-Frequency Exam Traps & Practical Reflexes

1. **Audit Policy Flag Configuration:**
   * Flags required in `/etc/kubernetes/manifests/kube-apiserver.yaml`:
     * `--audit-policy-file=/etc/kubernetes/audit/policy.yaml`
     * `--audit-log-path=/var/log/kubernetes/audit/audit.log`
     * `--audit-log-maxage=30`
     * `--audit-log-maxbackup=10`
     * `--audit-log-maxsize=100`
   * *Critical Trap:* You MUST add volume mounts for BOTH the policy file and the audit log directory into the API server pod spec!
2. **Falco Rule Testing & Restart:**
   * After modifying `/etc/falco/falco_rules.local.yaml`, always restart the service:
     `systemctl restart falco` and verify with `journalctl -u falco -e`.
3. **Pod Threat Containment:**
   * When a compromised container is detected in real time:
     1. Isolate the pod via an egress/ingress deny-all NetworkPolicy.
     2. Do not immediately delete the pod if memory forensics are needed; capture container logs and dump processes.

---

## 3. Reading & Study Checklist

- [ ] Read *Kubernetes Auditing Documentation*
- [ ] Read *Falco Rule Syntax & Configuration Guide*
- [ ] Practice creating Kubernetes audit policies with multiple audit levels
- [ ] Practice writing custom Falco rules and testing threat detection triggers
- [ ] Study eBPF runtime observability architecture with Tetragon
