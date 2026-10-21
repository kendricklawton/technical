# Domain 5: Troubleshooting — Essential Documentation & Technical Reference Guide

> **Blueprint Weight:** 30% of Certified Kubernetes Administrator (CKA)  
> **Core Objective:** Systematically diagnose and remediate broken control planes, crashed worker nodes, failing kubelet services, unresponsive static pods, networking/DNS outages, and crashing applications using command-line triage tools (`kubectl`, `crictl`, `journalctl`, `systemctl`).

---

## 1. Official Documentation & Authoritative References

### 1.1 [Troubleshooting Clusters and Control Planes](https://kubernetes.io/docs/tasks/debug/debug-cluster/)
* **Canonical Scope:** Diagnosing catastrophic cluster failures.
* **Control Plane Diagnostic Sequence:**
  1. Inspect static pod manifests in `/etc/kubernetes/manifests/` for syntax errors or invalid flags.
  2. If API server is down, `kubectl` will fail. Use `crictl` directly:
     * `crictl ps -a` (Check stopped or crashing containers).
     * `crictl logs <container-id>` (Read raw container stdout/stderr).
  3. Inspect control plane log files directly in `/var/log/pods/` and `/var/log/containers/`.

### 1.2 [Troubleshooting Worker Nodes & Kubelet](https://kubernetes.io/docs/tasks/debug/debug-cluster/monitor-node-health/)
* **Canonical Scope:** Resolving `NotReady` nodes and unresponsive container runtimes.
* **Node Triage Sequence:**
  1. `kubectl describe node <node-name>` (Check Conditions: `MemoryPressure`, `DiskPressure`, `PIDPressure`, `Ready`).
  2. SSH to node: `systemctl status kubelet` and `systemctl status containerd`.
  3. Inspect system logs: `journalctl -u kubelet -e --no-pager -n 100`.
  4. Verify certificate validity in `/var/lib/kubelet/pki/` (`kubelet-client-current.pem`).

### 1.3 [Troubleshooting Applications & Debugging with Ephemeral Containers](https://kubernetes.io/docs/tasks/debug/debug-application-cluster/debug-running-pod/)
* **Canonical Scope:** Debugging running or failing application pods.
* **Core Commands:**
  * **Examine Events:** `kubectl describe pod <pod-name>` (Check `LastState`, `ExitCode`, and event history).
  * **Ephemeral Debug Container:**
    ```bash
    kubectl debug -it <pod-name> --image=busybox:1.28 --target=<container-name>
    ```
  * **Copy and Modify Pod:**
    ```bash
    kubectl debug <pod-name> -it --copy-to=my-debugger --container=app -- sh
    ```

---

## 2. High-Frequency Exam Traps & Practical Reflexes

1. **Static Pod Manifest Typos:**
   * A typo in `/etc/kubernetes/manifests/kube-apiserver.yaml` causes the kubelet to fail to start the container.
   * *Symptom:* `The connection to the server <ip>:6443 was refused - did you specify the right host or port?`
   * *Resolution:* Inspect `/var/log/pods/` or run `crictl logs` on the stopped container to pinpoint the invalid flag.
2. **Kubelet Wrong Cgroup Driver:**
   * Container runtime is configured with `systemd` cgroup driver, but kubelet is configured with `cgroupfs`.
   * *Resolution:* Update `/var/lib/kubelet/config.yaml` to set `cgroupDriver: systemd`, reload daemon, and restart kubelet.
3. **Application `CrashLoopBackOff` vs `OOMKilled`:**
   * Check `kubectl describe pod`:
     * If `Exit Code: 137` and `Reason: OOMKilled` -> Increase memory limit in pod spec.
     * If `Exit Code: 1` or `2` -> Application runtime crash; check `kubectl logs <pod> --previous`.

---

## 3. Reading & Study Checklist

- [ ] Read *Troubleshooting Clusters and Control Planes*
- [ ] Read *Troubleshooting Worker Nodes & Kubelet*
- [ ] Practice debugging crashed static pods using `crictl ps` and `crictl logs`
- [ ] Practice kubelet service debugging using `journalctl -u kubelet`
- [ ] Practice interactive debugging using `kubectl debug` with ephemeral containers
