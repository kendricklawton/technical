# Domain 2 — Quiz 3: ConfigMaps, Secrets, Probes, Resource Limits, QoS Classes & HPA

Test your mastery of application configuration, container health probes, Quality of Service (QoS) classes, namespace resource governance, and horizontal pod autoscaling.

---

### Question 1
A Pod mounts a `ConfigMap` named `nginx-conf` into `/etc/nginx/nginx.conf` using `volumeMounts[].subPath: nginx.conf` so that the other files in `/etc/nginx/` are not overwritten. You update the `nginx-conf` `ConfigMap` using `kubectl edit cm nginx-conf` and wait 5 minutes, but `/etc/nginx/nginx.conf` inside the running container still has the old content. Why?

A. `ConfigMaps` are immutable by default.  
B. Container volume mounts that use **`subPath`** (as well as environment variables) **never** receive automatic live updates when the referenced `ConfigMap` or `Secret` is updated; the Pod must be recreated/restarted.  
C. `kube-proxy` caches `ConfigMaps` for 24 hours.  
D. Only `Secrets` auto-update inside containers.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
When a `ConfigMap` or `Secret` is mounted as a whole directory (without `subPath`), `kubelet` uses atomic symlink swaps (`..data`) to update the mounted files within ~1 minute. However, when **`subPath`** is used to mount a single file, the Linux bind mount locks onto the original inode and **never receives live updates** when the `ConfigMap`/`Secret` changes.
</details>

---

### Question 2
An application takes anywhere from 45 to 90 seconds to load machine-learning weights into memory at startup. Once started, you want `kubelet` to detect any process deadlock within 15 seconds (`periodSeconds: 5`, `failureThreshold: 3`) and restart the container. Which probe configuration prevents `kubelet` from prematurely killing the container during its 90-second startup phase?

A. Increase `livenessProbe.initialDelaySeconds` to `5`.  
B. Configure a **`startupProbe`** with `periodSeconds: 5` and `failureThreshold: 24` (allowing up to $5 \times 24 = 120\text{s}$ to start), paired with a fast **`livenessProbe`** (`periodSeconds: 5`, `failureThreshold: 3`).  
C. Configure only a `readinessProbe` (`readinessProbe` restarts deadlocked containers).  
D. Set `terminationGracePeriodSeconds: 120`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
When a `startupProbe` is defined, `kubelet` disables `livenessProbe` and `readinessProbe` checks until the `startupProbe` succeeds for the first time. Once it succeeds, `startupProbe` stops running for the remainder of the container's life, handing over monitoring to the fast `livenessProbe`.
</details>

---

### Question 3
A Pod in a 3-replica Deployment begins failing its **`readinessProbe`**, while its **`livenessProbe`** continues to succeed. What action does Kubernetes take?

A. `kubelet` immediately kills and restarts the container.  
B. The container continues running without being restarted, `kubelet` sets the Pod's `Ready` condition to `False`, and the `EndpointSlice` controller removes the Pod's IP from all matching `Services` so no new traffic is routed to it.  
C. The Pod is evicted from the node.  
D. The `ReplicaSet` creates a 4th replacement Pod immediately.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
- **`livenessProbe` failure** $\to$ Kills and restarts the container.
- **`readinessProbe` failure** $\to$ **Never restarts** the container; instead, it marks the Pod `Ready=False` and removes the Pod from Service `Endpoints`/`EndpointSlices` until the `readinessProbe` passes again.
</details>

---

### Question 4
A Pod contains **two** containers: `app` and `metrics-exporter`.
- Container `app` specifies `requests: { cpu: 500m, memory: 512Mi }` and `limits: { cpu: 500m, memory: 512Mi }`.
- Container `metrics-exporter` specifies `requests: { cpu: 100m, memory: 128Mi }` and `limits: { cpu: 200m, memory: 128Mi }`.

What Quality of Service (`status.qosClass`) class does Kubernetes assign to this Pod?

A. `Guaranteed`  
B. `Burstable`  
C. `BestEffort`  
D. `Critical`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
For a Pod to receive the **`Guaranteed`** QoS class, **every** container in the Pod must have both CPU and memory requests and limits set, and for **every** container `requests.cpu == limits.cpu` and `requests.memory == limits.memory`. Because `metrics-exporter` has `requests.cpu: 100m` $\neq$ `limits.cpu: 200m`, the entire Pod is classified as **`Burstable`**.
</details>

---

### Question 5
What happens inside a running container on a Linux worker node when:
1. The container tries to use **more CPU** than its `resources.limits.cpu`?
2. The container tries to allocate **more Memory** than its `resources.limits.memory`?

A. Both CPU and Memory violations cause `kubelet` to kill the container with `OOMKilled`.  
B. Exceeding `limits.cpu` causes the Linux kernel CFS scheduler to **throttle** the container's CPU cycles (without killing it), whereas exceeding `limits.memory` causes the kernel OOMKiller to **terminate** the process (`OOMKilled`, Exit Code `137`).  
C. Exceeding `limits.cpu` kills the container with Exit Code `143`, while exceeding `limits.memory` pages to swap.  
D. Resource limits are only used by `kube-scheduler` and are not enforced at runtime.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
- **CPU is compressible:** Enforced via Linux cgroup CFS bandwidth quota (`cpu.cfs_quota_us` / `cpu.max`). The container is throttled (slowed down), never killed.
- **Memory is incompressible:** Enforced via Linux cgroup memory max. Exceeding the limit invokes the kernel OOMKiller, sending `SIGKILL` (`128 + 9 = 137`) and marking the container state `Terminated` with `Reason: OOMKilled`.
</details>

---

### Question 6
An administrator creates a `ResourceQuota` in the `analytics` namespace with `hard: { requests.cpu: "4", limits.cpu: "8", requests.memory: 8Gi, limits.memory: 16Gi }`. No `LimitRange` exists in the `analytics` namespace. A developer then runs:
```bash
kubectl run test-pod --image=nginx:1.27 -n analytics
```
What happens?

A. `test-pod` is created with `BestEffort` QoS.  
B. `kube-apiserver` **rejects** the Pod creation with `Error from server (Forbidden): pods "test-pod" is forbidden: failed quota: ... must specify limits.cpu,limits.memory,requests.cpu,requests.memory`.  
C. `test-pod` remains `Pending` until a `LimitRange` is created.  
D. `test-pod` consumes all 4 CPUs and 8Gi memory by default.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
When a namespace has an active `ResourceQuota` tracking CPU or memory (`requests.cpu`, `limits.cpu`, `requests.memory`, `limits.memory`), the `ResourceQuota` admission controller **rejects any Pod that does not explicitly define those requests and limits**—unless a **`LimitRange`** is also present in the namespace to automatically inject `default` and `defaultRequest` values during mutating admission!
</details>

---

### Question 7
You create a `Pod` manifest where the container specifies **only** `resources.limits: { cpu: "1", memory: "1Gi" }` and omits `resources.requests` entirely. What will `resources.requests` and the Pod's `qosClass` be once admitted by `kube-apiserver`?

A. `requests` will be `0`, and `qosClass` will be `Burstable`.  
B. `kube-apiserver` automatically defaults `requests` to equal `limits` (`cpu: "1", memory: "1Gi"`), and if it is the only container in the Pod, its `qosClass` will be **`Guaranteed`**.  
C. `kube-apiserver` rejects the Pod because `requests` cannot be omitted when `limits` are set.  
D. `qosClass` will be `BestEffort`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
In Kubernetes, if a container specifies `limits` for CPU or memory without specifying `requests`, `kube-apiserver` automatically copies the `limits` values into `requests`. Since `requests == limits` for both CPU and memory, the Pod satisfies the exact rule for **`Guaranteed`** QoS!
</details>

---

### Question 8
In a Pod manifest, you mount a `Secret` named `ssh-private-key` as a volume at `/root/.ssh` and want the mounted file permissions to be read-only for the file owner (`0400` in octal). When writing `defaultMode` in YAML, which two values are equivalent and valid?

A. `defaultMode: 0400` (octal with leading zero) or `defaultMode: 256` (decimal)  
B. `defaultMode: "400"` or `defaultMode: "r--------"`  
C. `defaultMode: 400` (without leading zero)  
D. `chmod: 0400`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: A**

**Explanation:**  
In YAML, `defaultMode` is an integer. If you write `defaultMode: 0400` (with a leading `0`), YAML parses it as an octal number ($4 \times 8^2 = 256$ in decimal). If you write `defaultMode: 400` *without* a leading zero, YAML parses it as decimal `400` (which is octal `0620`, giving unexpected permissions!).
</details>

---

### Question 9
A `HorizontalPodAutoscaler` (`HPA`) is configured to scale a `Deployment` between `minReplicas: 2` and `maxReplicas: 10` based on `averageUtilization: 50` for CPU. However, when you run `kubectl get hpa`, the `TARGETS` column displays `<unknown>/50%` and the HPA fails to scale. Which two issues are the most common root causes of `<unknown>/50%`?

A. `kube-proxy` is stopped or `CoreDNS` is down.  
B. Either the `metrics-server` addon is not running/healthy in the cluster, **OR** the Pods in the target `Deployment` do not have `resources.requests.cpu` defined!  
C. The `Deployment` uses `Recreate` strategy.  
D. The namespace is missing a `ResourceQuota`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
CPU percentage utilization is calculated as:
$$\text{Utilization \%} = \frac{\text{Current Pod CPU Usage (from metrics-server)}}{\text{Pod } \texttt{resources.requests.cpu}} \times 100$$
If **either** `metrics-server` is missing/broken **or** the Deployment's Pods omit `resources.requests.cpu` (zero denominator!), the HPA cannot compute a percentage and displays `<unknown>/50%`.
</details>

---

### Question 10
When a worker node runs critically low on available memory (`MemoryPressure`), `kubelet` begins evicting Pods to reclaim memory. In what order are Pods evicted based on their Quality of Service (`qosClass`)?

A. `Guaranteed` $\to$ `Burstable` $\to$ `BestEffort`  
B. `BestEffort` $\to$ `Burstable` $\to$ `Guaranteed`  
C. `Burstable` $\to$ `BestEffort` $\to$ `Guaranteed`  
D. Alphabetical order by Pod name

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
Under node `MemoryPressure`, `kubelet` evicts Pods in the following strict order:
1. **`BestEffort`** Pods (and any `Burstable` Pods using more memory than requested) are evicted **first**.
2. **`Burstable`** Pods using less memory than their request are evicted **second**.
3. **`Guaranteed`** Pods and `system-node-critical` static pods are protected and never evicted unless system daemons themselves are starving for memory.
</details>

---

### Question 11
How can you prevent a `ConfigMap` or `Secret` containing static production configuration from ever being accidentally modified, while also reducing watch load on `kubelet` and `kube-apiserver`?

A. Add `readOnly: true` to the `ConfigMap` metadata.  
B. Set `immutable: true` at the top level of the `ConfigMap` or `Secret` manifest.  
C. Remove `update` from the `system:nodes` ClusterRole.  
D. Add a finalizer `kubernetes.io/immutable`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
Setting `immutable: true` on a `ConfigMap` or `Secret` prohibits any future modifications to `data` or `binaryData` (the object can only be deleted and recreated). It also improves cluster scalability because `kubelet` stops polling/watching `kube-apiserver` for updates to that `ConfigMap`/`Secret`.
</details>

---

### Question 12
You need to pull an image from a private registry `registry.internal.io` for a Pod. After creating a `kubernetes.io/dockerconfigjson` Secret named `private-reg-cred` using `kubectl create secret docker-registry`, where in the Pod YAML do you reference `private-reg-cred`?

A. Inside `spec.containers[].envFrom`  
B. At the Pod spec level under `spec.imagePullSecrets: [{ name: private-reg-cred }]` (or attached to the Pod's `ServiceAccount`)  
C. Inside `metadata.annotations`  
D. Inside `spec.volumes[]`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
Docker registry pull secrets are referenced either at the Pod spec level (`spec.imagePullSecrets: - name: private-reg-cred`) or added to the `ServiceAccount`'s `imagePullSecrets` field (which automatically injects `imagePullSecrets` into every Pod created with that `ServiceAccount`).
</details>
