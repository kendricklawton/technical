# Domain 6: Monitoring, Logging & Runtime Security — Quiz 2 (Falco & Tetragon eBPF Runtime Security)

**Exam:** Certified Kubernetes Security Specialist (CKS)  
**Format:** 10 Performance Scenario & Multiple Choice Security Questions with Explanations

---

### Question 1: Customizing Falco Rules Without Being Overwritten on Upgrade
On a worker node running Falco, `/etc/falco/falco_rules.yaml` contains the default upstream ruleset. If a CKS task asks you to add a new custom rule or override an existing rule, which file should you edit so package upgrades do not overwrite your changes?
- A. `/etc/falco/falco_rules.local.yaml`
- B. `/etc/falco/falco_rules.yaml`
- C. `/var/log/falco.yaml`
- D. `/etc/kubernetes/falco.conf`

---

### Question 2: Mandatory Keys in a Falco Rule Definition
Every Falco rule block in YAML must contain five required top-level keys. What are those five keys?
- A. `rule`, `desc`, `condition`, `output`, and `priority` (e.g., `EMERGENCY`, `ALERT`, `CRITICAL`, `ERROR`, `WARNING`, `NOTICE`, `INFORMATIONAL`, `DEBUG`).
- B. `name`, `match`, `action`, `log`, `level`.
- C. `apiVersion`, `kind`, `metadata`, `spec`, `status`.
- D. `syscall`, `container`, `user`, `file`, `severity`.

---

### Question 3: Falco Filter Fields for Container & Process Telemetry
In a Falco rule `condition` or `output` string, what do the filter fields `container.id != host`, `%container.name`, `%proc.name`, `%proc.cmdline`, `%user.name`, and `%fd.name` represent?
- A. `container.id != host` matches only events occurring inside a container (excluding host OS processes); `%container.name` is the container name, `%proc.name` is the binary name (e.g., `bash`), `%proc.cmdline` is the full command line, `%user.name` is the executing user, and `%fd.name` is the file descriptor path or socket address accessed.
- B. They are Prometheus metric labels.
- C. They are AppArmor macro variables.
- D. They only work on Windows nodes.

---

### Question 4: Writing a Falco Rule to Detect Interactive Shells Spawned in Containers
Which Falco `condition` expression detects whenever a shell (`bash`, `sh`, `zsh`, `ash`) is spawned inside any container?
- A. `evt.type = execve and evt.dir = < and container.id != host and proc.name in (bash, sh, zsh, ash)` (or using default macro `spawned_process and container and shell_procs`)
- B. `container.shell == true`
- C. `syscall == bash`
- D. `pod.exec == 1`

---

### Question 5: Reloading Falco and Viewing Live Falco Alerts on a Worker Node
After editing `/etc/falco/falco_rules.local.yaml`, how do you validate the rule syntax, reload the Falco systemd daemon, and inspect the emitted runtime alerts?
- A. Run `falco --validate /etc/falco/falco_rules.local.yaml`, then `systemctl restart falco` (or `kill -SIGHUP $(pidof falco)`), and inspect `journalctl -u falco -f` or `/var/log/syslog`.
- B. Run `kubectl rollout restart falco`.
- C. Run `aa-enforce /etc/falco/falco_rules.local.yaml`.
- D. Reboot the cluster.

---

### Question 6: Falco Detection vs Inline Kernel Enforcement (Cilium Tetragon)
What is the fundamental architectural difference between **Falco** and **Cilium Tetragon** when an attacker inside a container attempts to execute a forbidden binary or read `/etc/shadow`?
- A. **Falco** operates **asynchronously** in user space by reading kernel/eBPF ring-buffer events to generate alerts (and optionally trigger external response webhooks like Falcosidekick after the syscall has already executed), whereas **Cilium Tetragon** attaches eBPF programs directly in-kernel and can enforce **synchronous inline `Sigkill` (`SIGKILL`) or `Override` (`-EPERM`)** before the system call completes or returns data to user space.
- B. Falco runs inside the container, while Tetragon runs in the cloud console.
- C. Tetragon cannot inspect container processes.
- D. Falco blocks network packets inline.

---

### Question 7: Cilium Tetragon `TracingPolicy` / `TracingPolicyNamespaced` CRD
In Cilium Tetragon (`cilium.io/v1alpha1`), you want to immediately kill (`SIGKILL`) any container process in namespace `production` that attempts to execute `/usr/bin/curl` or `/usr/bin/wget`. Which Tetragon resource and action accomplish this?
- A. A `TracingPolicyNamespaced` in `production` hooking `kprobes` (`sys_execve`) with a `matchArgs` selector on `/usr/bin/curl` and `/usr/bin/wget` and `matchActions: [{ action: Sigkill }]`.
- B. A `NetworkPolicy` with `action: Sigkill`.
- C. An `Ingress` rule.
- D. A `PersistentVolumeClaim`.

---

### Question 8: Overriding a Existing Default Falco Rule Using `append: true` or `override:`
In `/etc/falco/falco_rules.local.yaml`, you want to modify the built-in `Write below etc` rule to change its `priority` from `ERROR` to `CRITICAL` and append `and not proc.name = confd` to its `condition` without copying the entire original rule. What syntax does Falco 0.35+ support?
- A. Specify the `rule: Write below etc` with an `override:` block (`condition: append`, `priority: replace`) or `append: true`.
- B. Delete `/etc/falco/falco_rules.yaml`.
- C. Rename the rule to `Write below etc 2`.
- D. Falco rules cannot be overridden.

---

### Question 9: Detecting Package Management Execution (`apt`, `apk`, `yum`, `pip`) at Runtime
Why is launching `apt-get`, `apk`, `yum`, or `pip` inside a running production container a high-priority Falco runtime alert?
- A. Containers should be immutable artifacts built in CI/CD; running package managers at runtime indicates either severe operational drift or an attacker installing post-exploitation tools (`ncat`, `nmap`, `xmrig`, `socat`) inside a compromised container.
- B. Package managers corrupt `etcd`.
- C. Package managers disable `kube-proxy`.
- D. `apt-get` deletes the container image.

---

### Question 10: Detecting Container Escape Attempts (`Mount` in Host Namespace / Reading `/proc/1/root`)
Which Falco condition indicators reveal an active container escape attempt by a privileged or `CAP_SYS_ADMIN` container?
- A. A container process (`container.id != host`) opening files under `/proc/1/root` (the host `init`/`systemd` root filesystem), loading kernel modules (`init_module` / `finit_module`), invoking `unshare`/`setns`, or writing to `/sys/fs/cgroup/.../release_agent`.
- B. A container reading `/etc/resolv.conf`.
- C. A container binding to port `8080`.
- D. A container writing to `/tmp`.

---

## Answer Key & Detailed Explanations

1. **A** — `/etc/falco/falco_rules.local.yaml` is loaded after `falco_rules.yaml` and is preserved across package upgrades.
2. **A** — Every Falco rule requires `rule`, `desc`, `condition`, `output`, and `priority`.
3. **A** — Mastering Falco filter fields (`%container.id`, `%container.name`, `%k8s.ns.name`, `%k8s.pod.name`, `%proc.name`, `%user.uid`, `%fd.name`) is essential for formatting custom CKS alert outputs.
4. **A** — Checking `evt.type = execve` (or macro `spawned_process`), `container.id != host` (`container`), and `proc.name in (bash, sh, ...)` detects interactive shell execution inside containers.
5. **A** — Always run `falco --validate <file>` before restarting `falco` so syntax typos don't prevent the daemon from starting.
6. **A** — Falco alerts out-of-band via userspace event consumption, whereas Tetragon executes eBPF enforcement actions (`Sigkill`, `Override`) synchronously inside the kernel syscall path.
7. **A** — Tetragon uses `TracingPolicy` (cluster-wide) and `TracingPolicyNamespaced` (namespace-scoped) CRDs to hook kernel functions and apply `Sigkill` actions.
8. **A** — Falco supports `override:` blocks (`condition: append`, `output: replace`, `priority: replace`) in `falco_rules.local.yaml`.
9. **A** — Executing package managers at runtime violates container immutability and is a classic indicator of post-compromise tool installation.
10. **A** — Accessing `/proc/1/root` (when `hostPID: true`), invoking `finit_module`, or abusing cgroup `release_agent` are primary container escape techniques.
