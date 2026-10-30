# Domain 6: Monitoring, Logging & Runtime Security — Quiz 1 (Kubernetes API Audit Logging)

**Exam:** Certified Kubernetes Security Specialist (CKS)  
**Format:** 10 Performance Scenario & Multiple Choice Security Questions with Explanations

---

### Question 1: Four Kubernetes Audit Policy Logging Levels (`None`, `Metadata`, `Request`, `RequestResponse`)
In a Kubernetes `Policy` (`audit.k8s.io/v1`), what is the exact difference between the four audit levels?
- A. `None` (do not log matching events), `Metadata` (log request metadata: requesting user, timestamp, resource, verb, namespace, but NOT request/response bodies), `Request` (log metadata + request body, but NOT response body), and `RequestResponse` (log metadata + both request and response bodies).
- B. `Info`, `Warning`, `Error`, `Fatal`.
- C. `Debug`, `Verbose`, `Trace`, `Silent`.
- D. `Read`, `Write`, `Delete`, `Admin`.

---

### Question 2: Why `Secrets` Should Be Audited at `Metadata` Level Instead of `RequestResponse`
Why do CKS security benchmarks recommend auditing `secrets` and `tokenreviews` at level `Metadata` rather than `Request` or `RequestResponse`?
- A. Auditing `secrets` at `Request` or `RequestResponse` writes the complete JSON payload—including the base64-encoded Secret data—into plaintext audit log files (`/var/log/kubernetes/audit/audit.log`) on disk.
- B. `kube-apiserver` crashes if `RequestResponse` is used on `secrets`.
- C. `Secrets` do not have request or response bodies.
- D. `Metadata` encrypts the audit log file.

---

### Question 3: First-Match-Wins Evaluation Order in Audit `Policy` Rules
In a Kubernetes Audit `Policy` file (`rules:`), how does `kube-apiserver` evaluate incoming API requests against the list of rules?
- A. **Top-to-bottom, first-match-wins**: as soon as an event matches a rule, that rule's `level` is applied and subsequent rules are ignored—therefore, specific exclusions (`level: None`) and sensitive resource rules (`secrets` at `Metadata`) must be placed **above** broad catch-all rules.
- B. Bottom-to-top, last-match-wins.
- C. All matching rules are merged and the highest level wins.
- D. Rules are evaluated in alphabetical order.

---

### Question 4: Four Audit Execution Stages (`RequestReceived`, `ResponseStarted`, `ResponseComplete`, `Panic`)
Which audit stage fires exclusively for long-running streaming requests (such as `kubectl logs -f` or `kubectl get pods --watch`) before the HTTP response body finishes streaming, and which stage do you typically put in `omitStages` to reduce duplicate log volume?
- A. `ResponseStarted` fires when response headers are sent for long-running `watch`/`logs` requests; `RequestReceived` (which fires immediately before the handler processes the request) is commonly listed in `omitStages: ["RequestReceived"]` so every normal request produces one final audit entry at `ResponseComplete` instead of two.
- B. `Panic` fires on every request.
- C. `RequestReceived` fires only on `watch` requests.
- D. `ResponseComplete` fires before authentication.

---

### Question 5: Required `kube-apiserver` Flags for Audit Log File Backend & Rotation
On a CKS exam task, you must enable audit logging using policy `/etc/kubernetes/audit/policy.yaml`, writing logs to `/var/log/kubernetes/audit/audit.log`, keeping at most `10` old log files (`maxbackup`), rotating at `100` megabytes (`maxsize`), and retaining logs for `30` days (`maxage`). What are the exact `kube-apiserver` flags?
- A.
  - `--audit-policy-file=/etc/kubernetes/audit/policy.yaml`
  - `--audit-log-path=/var/log/kubernetes/audit/audit.log`
  - `--audit-log-maxbackup=10`
  - `--audit-log-maxsize=100`
  - `--audit-log-maxage=30`
- B. `--enable-audit=true` and `--audit-dir=/var/log/kubernetes/audit`
- C. `--audit-config=/etc/kubernetes/audit/policy.yaml`
- D. `--log-file=/var/log/kubernetes/audit/audit.log`

---

### Question 6: Mandatory `volumes` and `volumeMounts` in `kube-apiserver.yaml` for Audit Logging
You added the five `--audit-*` flags in Question 5 to `/etc/kubernetes/manifests/kube-apiserver.yaml`, but `/var/log/kubernetes/audit/audit.log` is not appearing on the host node or `kube-apiserver` is crashing. Why?
- A. You must mount **both** `/etc/kubernetes/audit` (read-only `hostPath` `DirectoryOrCreate`/`File`) AND `/var/log/kubernetes/audit` (read-write `hostPath` `DirectoryOrCreate`) into the `kube-apiserver` container via `volumes` and `volumeMounts`.
- B. Only `etcd` mounts `/var/log/kubernetes/audit`.
- C. You must mount `/var/log/kubernetes/audit` as `readOnly: true`.
- D. `kubelet` writes the audit log directly without volume mounts.

---

### Question 7: Writing an Audit Policy Rule for `namespaces` and `deployments` in `apps` API Group
In an Audit `Policy` (`audit.k8s.io/v1`), what `group:` string is used for core resources (`pods`, `secrets`, `configmaps`, `namespaces`) versus `deployments` and `daemonsets`?
- A. Core resources use an empty string `group: ""` (`resources: ["pods", "secrets"]`), while `deployments` and `daemonsets` use `group: "apps"`.
- B. Core resources use `group: "core/v1"`.
- C. Both use `group: "*"` (wildcards are not valid in `group`).
- D. `deployments` use `group: "extensions"`.

---

### Question 8: Filtering Audit Events by `verbs` and `nonResourceURLs`
You want to suppress (`level: None`) all read-only `get`, `list`, and `watch` requests by `system:kube-proxy`, as well as unauthenticated health probes to `/healthz`, `/readyz`, and `/livez`. How are health endpoints matched in an Audit Policy?
- A. Using `nonResourceURLs: ["/healthz*", "/readyz*", "/livez*"]` with `level: None`, and a rule matching `users: ["system:kube-proxy"]`, `verbs: ["get", "list", "watch"]` with `level: None`.
- B. Using `resources: ["healthz"]`.
- C. Using `namespaces: ["healthz"]`.
- D. Health probes cannot be filtered in Audit Policy.

---

### Question 9: Hunting Compromised ServiceAccounts in `/var/log/kubernetes/audit/audit.log` with `jq`
An attacker used a stolen ServiceAccount token inside a Pod to list Secrets across the cluster (`403 Forbidden`). Which `jq` command filters `/var/log/kubernetes/audit/audit.log` to identify the exact username (`user.username`), source IP (`sourceIPs`), and target resource of all `403` responses?
- A. `jq 'select(.responseStatus.code == 403) | {user: .user.username, ip: .sourceIPs, verb: .verb, resource: .objectRef.resource, ns: .objectRef.namespace}' /var/log/kubernetes/audit/audit.log`
- B. `grep 403 /var/log/syslog`
- C. `kubectl get events -A`
- D. `crictl logs kube-apiserver`

---

### Question 10: Recovering When `kube-apiserver` Crashes After Editing Audit Policy
After modifying `/etc/kubernetes/audit/policy.yaml` and `/etc/kubernetes/manifests/kube-apiserver.yaml`, `kubectl get pods` fails with `connection refused`. How do you immediately diagnose why `kube-apiserver` refused to start?
- A. Check `/var/log/pods/kube-system_kube-apiserver-*/kube-apiserver/*.log` (or `crictl ps -a | grep kube-apiserver` $\to$ `crictl logs <id>`) for messages like `unknown field` or `failed to load audit policy file` (e.g., using `apiVersion: audit.k8s.io/v1beta1` instead of `audit.k8s.io/v1` or plural/singular typos in `resources`).
- B. Run `kubectl describe node`.
- C. Reboot the worker nodes.
- D. Delete `/etc/kubernetes/pki`.

---

## Answer Key & Detailed Explanations

1. **A** — `None` drops the event; `Metadata` logs headers/user/verb/objectRef; `Request` adds the request payload; `RequestResponse` adds both request and response payloads.
2. **A** — Logging `secrets` at `Request` or `RequestResponse` dumps secret contents into the audit log file in plaintext.
3. **A** — Audit rules are evaluated sequentially from top to bottom; the first matching rule wins.
4. **A** — Omitting `RequestReceived` prevents duplicate audit entries while `ResponseStarted` captures the start of long-lived `watch` and `exec`/`logs` streams.
5. **A** — These five `--audit-*` flags configure the file backend and built-in `lumberjack` log rotation in `kube-apiserver`.
6. **A** — Because `kube-apiserver` is a static pod container, both the policy file path and the writable log directory path must be mounted from the host via `volumes` and `volumeMounts`.
7. **A** — Core API group is `group: ""` and resource names are always lowercase plural (`pods`, `secrets`, `namespaces`, `configmaps`).
8. **A** — API server HTTP endpoints that are not Kubernetes API objects (`/healthz`, `/metrics`, `/version`, `/openapi/*`) are matched via `nonResourceURLs`.
9. **A** — Kubernetes audit logs are JSON Lines (`.jsonl`), making `jq` the ideal CLI tool to filter by `.responseStatus.code`, `.user.username`, `.verb`, and `.objectRef.resource`.
10. **A** — Inspecting `/var/log/pods/kube-system_kube-apiserver-*/kube-apiserver/*.log` immediately prints the exact line error in `policy.yaml` or flag typo.
