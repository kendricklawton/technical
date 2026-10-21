# Domain 5: Troubleshooting — Quiz 3 (Application, CNI, DNS & Service Routing Triage)

**Exam:** Certified Kubernetes Administrator (CKA)  
**Format:** 10 Scenario & Performance Questions with Detailed Explanations

---

### Question 1: Pod Stuck in `CreateContainerConfigError`
A newly deployed Pod `checkout-api` is stuck in `CreateContainerConfigError`. `kubectl describe pod checkout-api` shows: `Error: secret "stripe-api-key" not found`. How do you resolve this without modifying the Pod spec if the Secret is required?
- A. Create the missing Secret `stripe-api-key` in the exact same namespace as `checkout-api` (`kubectl create secret generic stripe-api-key --from-literal=...`); the kubelet will automatically retry and start the container.
- B. Create the Secret in `kube-system`.
- C. Restart `kube-scheduler`.
- D. Delete the node.

---

### Question 2: Retrieving Logs from a Crashed Container (`CrashLoopBackOff`)
A container in Pod `payment-worker` starts, prints a fatal stack trace to `stderr`, and exits after 1 second, entering `CrashLoopBackOff`. Running `kubectl logs payment-worker` while it is restarting returns empty output or only the newly starting attempt. Which flag retrieves the logs from the **previous** terminated container instance?
- A. `kubectl logs payment-worker --previous` (or `-p`)
- B. `kubectl logs payment-worker --all-containers`
- C. `kubectl logs payment-worker --since=1h`
- D. `kubectl describe pod payment-worker`

---

### Question 3: Debugging Distroless or Crashed Pods with `kubectl debug`
A production Pod uses a `gcr.io/distroless/static` image that contains no shell (`/bin/sh`), no `curl`, and no `nslookup`. You need to inspect its network namespace and open sockets without restarting the Pod. Which command attaches an ephemeral troubleshooting container sharing the target container's process and network namespaces?
- A. `kubectl debug -it <pod-name> --image=busybox:1.36 --target=<container-name>`
- B. `kubectl exec -it <pod-name> -- /bin/bash`
- C. `kubectl attach <pod-name>`
- D. `kubectl cp`

---

### Question 4: Service Returning `Connection Refused` (`targetPort` Mismatch)
Pods in Deployment `frontend` (`labels: app=frontend`) are `Running` and listening on container port `8080`. Service `frontend-svc` selects `app=frontend` and exposes `port: 80, targetPort: 80`. Curling `http://frontend-svc` returns `Connection refused`. Which command confirms and fixes the issue?
- A. Inspect `kubectl get endpointslices -l kubernetes.io/service-name=frontend-svc` (or `kubectl get ep frontend-svc`), notice the endpoint ports point to `:80` instead of `:8080`, and patch `frontend-svc` to set `targetPort: 8080`.
- B. Restart `coredns`.
- C. Change the Service type to `LoadBalancer`.
- D. Add an Ingress resource.

---

### Question 5: Service Has Zero Endpoints (`Selector` Typo)
`curl http://backend-svc.default.svc.cluster.local` times out or fails to connect. Running `kubectl get ep backend-svc` shows `ENDPOINTS: <none>`. The backend Pods are `Running` and `1/1 Ready`. What is the root cause?
- A. The Service `spec.selector` labels do not match the `metadata.labels` on the running backend Pods (compare `kubectl get svc backend-svc -o wide` with `kubectl get pods --show-labels`).
- B. `coredns` is crashed.
- C. The container port is TCP instead of UDP.
- D. The node is cordoned.

---

### Question 6: Cluster-Wide DNS Resolution Failure (`CoreDNS` Loop / Upstream Error)
Pods can ping each other by IP address and connect to ClusterIPs directly, but `nslookup kubernetes.default` times out across all namespaces. `kubectl get pods -n kube-system -l k8s-app=kube-dns` shows both CoreDNS pods in `CrashLoopBackOff` with log: `plugin/loop: Loop (127.0.0.1:53 -> :53) detected for zone "."`. What caused this and how is it fixed?
- A. The worker node's `/etc/resolv.conf` points to `127.0.0.1` (e.g., `systemd-resolved` stub listener), which CoreDNS inherited via `forward . /etc/resolv.conf`. Fix by pointing `kubelet`'s `resolvConf` to `/run/systemd/resolve/resolv.conf` or updating the `coredns` ConfigMap (`kubectl edit cm coredns -n kube-system`) to forward to an upstream DNS server like `8.8.8.8` / `1.1.1.1`.
- B. `kube-proxy` is misconfigured.
- C. The Service CIDR is full.
- D. `etcd` is out of disk space.

---

### Question 7: Pods Stuck in `ContainerCreating` (`CNI Plugin Not Initialized`)
On a newly joined worker node, Pods are scheduled onto the node but remain stuck in `ContainerCreating` with event: `networkPlugin cni failed to set up pod network: stat /var/lib/calico/nodename: no such file or directory` or `failed to find plugin "calico" in path [/opt/cni/bin]`. What must you check?
- A. Verify that the CNI DaemonSet (`calico-node` / `cilium` / `flannel`) is running on that node (check if a custom taint blocked the CNI DaemonSet pod), and verify CNI binaries exist in `/opt/cni/bin` and CNI config exists in `/etc/cni/net.d`.
- B. Restart `kube-scheduler`.
- C. Create a `StorageClass`.
- D. Delete `coredns`.

---

### Question 8: `kube-proxy` ConfigMap / `ip_forward` Disabled on Worker Node
Pods on `worker-1` can reach pods on `worker-1`, but cannot reach ClusterIP Services backed by pods on `worker-2`. On `worker-1`, you check `sysctl net.ipv4.ip_forward` and see `net.ipv4.ip_forward = 0`. What command fixes cross-node packet forwarding immediately and persistently?
- A. Run `sysctl -w net.ipv4.ip_forward=1` and add `net.ipv4.ip_forward=1` to `/etc/sysctl.d/k8s.conf` followed by `sysctl --system`.
- B. Run `iptables -F`.
- C. Restart `etcd`.
- D. Change Service to `Headless`.

---

### Question 9: Liveness Probe Killing a Slow-Starting Legacy Application
A Java application takes 75 seconds to initialize its Spring context on startup. Its Pod has a `livenessProbe` with `initialDelaySeconds: 10`, `periodSeconds: 10`, and `failureThreshold: 3`. The container is killed and restarted every 40 seconds (`CrashLoopBackOff`) before it ever finishes starting. What is the best Kubernetes native solution?
- A. Configure a `startupProbe` (e.g., `periodSeconds: 10, failureThreshold: 12`, giving up to 120 seconds for startup) so the `livenessProbe` does not begin evaluating until the `startupProbe` succeeds once.
- B. Remove all probes from the cluster.
- C. Change `restartPolicy: Never`.
- D. Increase CPU limits to 64 cores.

---

### Question 10: Exit Code `137` vs `143` vs `1` vs `127`
During troubleshooting, you inspect `kubectl get pod <name> -o jsonpath='{.status.containerStatuses[0].lastState.terminated}'`. Match the Linux container exit code to its exact meaning:
- A. `137` = `SIGKILL` (9, typically `OOMKilled`), `143` = `SIGTERM` (15, graceful termination), `127` = command/binary not found in `$PATH`, `126` = command found but not executable (permission error).
- B. `137` = syntax error, `143` = OOMKilled, `127` = network timeout.
- C. `137` = DNS failure, `143` = disk full.
- D. `127` = SIGKILL, `137` = command not found.

---

## Answer Key & Detailed Explanations

1. **A** — `CreateContainerConfigError` occurs when the kubelet prepares container configuration prior to runtime start and finds a missing referenced `ConfigMap` or `Secret`.
2. **A** — `kubectl logs <pod> --previous` (`-p`) retrieves stdout/stderr from the terminated container instance prior to the current restart.
3. **A** — `kubectl debug -it <pod> --image=<debug-image> --target=<container>` injects an ephemeral container into the running Pod's namespaces.
4. **A** — A common CKA exam trap is a Service `targetPort` mismatch where the Service forwards traffic to port `80` while the container listens on `8080`.
5. **A** — If `kubectl get endpoints <svc>` is `<none>` while Pods are Ready, the Service's `spec.selector` does not match the Pods' labels.
6. **A** — CoreDNS's `loop` plugin halts if it detects that forwarding queries to `/etc/resolv.conf` loops back to `127.0.0.1:53`.
7. **A** — CNI requires both the CNI binary in `/opt/cni/bin` and the CNI config in `/etc/cni/net.d` populated by the CNI DaemonSet pod on every node.
8. **A** — Linux kernel IPv4 packet forwarding (`net.ipv4.ip_forward = 1`) is mandatory for CNI overlay/BGP routing and `kube-proxy` DNAT forwarding.
9. **A** — `startupProbe` disables `livenessProbe` and `readinessProbe` checks until the slow-starting container passes its startup check for the first time.
10. **A** — Exit codes $>128$ represent `128 + signal_number`: `128 + 9 (SIGKILL) = 137`, `128 + 15 (SIGTERM) = 143`. Exit code `127` means command not found, and `126` means permission denied / non-executable.
