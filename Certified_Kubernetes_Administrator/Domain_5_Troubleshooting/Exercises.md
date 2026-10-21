# Domain 5: Troubleshooting — Production Hands-On Exercises & Solutions

## Exercise 1: Triage Control Plane kube-apiserver Failure
### Scenario
`kubectl` returns `The connection to the server 192.168.1.10:6443 was refused`.
### Diagnostic Steps
1. SSH into control plane node.
2. Check kubelet: `sudo systemctl status kubelet`.
3. Check running containers: `sudo crictl ps -a | grep apiserver`.
4. Check logs: `sudo crictl logs <container-id>`.
5. Fix typo in `/etc/kubernetes/manifests/kube-apiserver.yaml`.
6. Watch container start: `sudo crictl ps | grep apiserver`.

---

## Exercise 2: Worker Node in `NotReady` State Diagnosis
### Scenario
`worker-02` shows `NotReady`.
### Diagnostic Steps
1. `kubectl describe node worker-02` (Check `Conditions` for DiskPressure or MemoryPressure).
2. SSH to `worker-02`.
3. Run `journalctl -u kubelet -e -n 50`.
4. If error indicates swap is active:
   ```bash
   sudo swapoff -a
   sudo systemctl restart kubelet
   ```
5. Check `kubectl get nodes` to confirm `Ready` status.

---

## Exercise 3: Pod in `CrashLoopBackOff` & Exit Code Analysis
### Scenario
A pod `payment-processor` crashes continuously with `CrashLoopBackOff`.
```bash
# Check previous logs
kubectl logs payment-processor -n payment --previous

# Check exit code
kubectl describe pod payment-processor -n payment | grep "Exit Code"
# Exit Code 137 -> OOMKilled! Increase resources.limits.memory
```

---

## Exercise 4: Resolving `CreateContainerConfigError`
A pod is stuck in `CreateContainerConfigError`.
```bash
kubectl describe pod api-worker -n default | grep -A 5 "Warning"
# Error: configmap "app-config" not found
# Fix: Create missing configmap
kubectl create cm app-config --from-literal=ENV=production
```

---

## Exercise 5: Live Debugging of Scratch / Distroless Container
Debug a network issue inside a container that has no shell or curl:
```bash
kubectl debug -it backend-pod --image=nicolaka/netshoot --target=app-container
# Inside netshoot:
ping db-service
nc -zvw3 db-service 5432
```

---

## Exercise 6: CoreDNS CrashLoopBackOff Loop Detection Fix
Fix CoreDNS infinite forwarding loops caused by systemd-resolved:
```bash
kubectl -n kube-system edit cm coredns
# Replace forward . /etc/resolv.conf with forward . 8.8.8.8
kubectl -n kube-system rollout restart deployment/coredns
```
