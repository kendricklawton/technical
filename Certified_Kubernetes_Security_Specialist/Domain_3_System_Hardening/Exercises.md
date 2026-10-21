# Domain 3: System Hardening — Hands-On Exercises & Solutions

## Exercise 1: AppArmor Profile Enforcement in Kubernetes
Load an AppArmor profile `k8s-deny-write` onto worker node `node-01` and apply it to a Pod.

```text
# Profile definition at /etc/apparmor.d/k8s-deny-write
#include <tunables/global>

profile k8s-deny-write flags=(attach_disconnected) {
  #include <abstractions/base>
  file,
  deny /** w,
}
```

```bash
# Load profile on worker node
sudo apparmor_parser -q /etc/apparmor.d/k8s-deny-write
sudo aa-status | grep k8s-deny-write
```

Pod manifest (`secure-app.yaml`):
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secured-pod
spec:
  securityContext:
    appArmorProfile:
      type: Localhost
      localhostProfile: k8s-deny-write
  containers:
  - name: app
    image: busybox:1.28
    command: ["sh", "-c", "touch /tmp/test || sleep 3600"]
```

---

## Exercise 2: Seccomp Profile Implementation
Deploy a container with Seccomp profile `RuntimeDefault` and a custom localhost profile blocking `chmod`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: seccomp-pod
spec:
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: test
    image: alpine:3.18
    command: ["sleep", "3600"]
```
