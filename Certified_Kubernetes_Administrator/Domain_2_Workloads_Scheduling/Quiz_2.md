# Domain 2 — Quiz 2: Scheduling, Taints & Tolerations, Node/Pod Affinity, PriorityClass & PDB

Test your mastery of Kubernetes scheduling mechanics, taints and tolerations, affinity rules, preemption, and voluntary disruption budgets.

---

### Question 1
You apply the taint `dedicated=ml:NoSchedule` to `worker-03` and add a matching `toleration` to your `ml-training` Deployment. However, when you scale `ml-training` to 3 replicas, 2 of the Pods get scheduled onto `worker-01` and `worker-02` instead of `worker-03`. Why did this happen, and how do you fix it?

A. `NoSchedule` only applies to DaemonSets; change the taint effect to `NoExecute`.  
B. A **Toleration** only *permits* a Pod to schedule onto a tainted node—it does **not** attract or restrict the Pod to that node. To force `ml-training` Pods onto `worker-03`, you must **also** label `worker-03` and configure `nodeSelector` or `nodeAffinity` (`requiredDuringSchedulingIgnoredDuringExecution`) on the Deployment.  
C. You must restart `kube-scheduler` after adding a taint.  
D. `worker-01` and `worker-02` must also be tainted with `dedicated=ml:NoSchedule`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
- **Taints + Tolerations** repel *other* Pods from `worker-03`, and allow `ml-training` to land on `worker-03`.
- **Node Affinity / `nodeSelector`** attracts/restricts `ml-training` so it can *only* land on `worker-03`.
To dedicate a node exclusively to a specific workload, you must combine **both** Taints/Tolerations and Node Affinity/`nodeSelector`.
</details>

---

### Question 2
A worker node `worker-02` currently has 5 running Pods (`pod-a`, `pod-b`, `pod-c`, `pod-d`, `pod-e`), none of which have any custom `tolerations`. You execute:
```bash
kubectl taint nodes worker-02 maintenance=true:NoSchedule
```
What happens to the 5 Pods already running on `worker-02`?

A. All 5 Pods are immediately evicted from `worker-02`.  
B. All 5 Pods are evicted after 300 seconds.  
C. **Nothing** happens to the 5 already-running Pods; they continue running normally on `worker-02`, and only **new** unscheduled Pods without a matching toleration are prevented from scheduling onto `worker-02`.  
D. Only `BestEffort` Pods are evicted.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**  
- `NoSchedule` only affects the scheduling of **new** Pods; it never evicts Pods already running on the node.
- `NoExecute` both prevents new Pods from scheduling **and** immediately evicts already-running Pods that do not tolerate the taint (or evicts them after `tolerationSeconds` if they have a time-bounded toleration).
</details>

---

### Question 3
In a Pod's `nodeAffinity` configuration under `requiredDuringSchedulingIgnoredDuringExecution`, you see **two** entries under `nodeSelectorTerms[]`, and the first `nodeSelectorTerm` contains **two** entries under `matchExpressions[]`:

```yaml
nodeSelectorTerms:
- matchExpressions:
  - key: zone
    operator: In
    values: ["us-east-1a"]
  - key: disk
    operator: In
    values: ["ssd"]
- matchExpressions:
  - key: gpu
    operator: Exists
```
How does `kube-scheduler` evaluate multiple `nodeSelectorTerms` versus multiple `matchExpressions`?

A. Both `nodeSelectorTerms` and `matchExpressions` are evaluated with logical **AND**.  
B. Multiple `nodeSelectorTerms` are evaluated with logical **OR**, while multiple `matchExpressions` within a single `nodeSelectorTerm` are evaluated with logical **AND**.  
C. Multiple `nodeSelectorTerms` are evaluated with logical **AND**, while multiple `matchExpressions` are evaluated with logical **OR**.  
D. Both are evaluated with logical **OR**.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
- **Multiple `nodeSelectorTerms[]` = Logical OR:** A node matches if it satisfies *either* Term 1 *or* Term 2.
- **Multiple `matchExpressions[]` inside one term = Logical AND:** To satisfy Term 1, a node must have `zone=us-east-1a` **AND** `disk=ssd`.
Thus, the Pod above will schedule onto a node that has (`zone=us-east-1a` AND `disk=ssd`) **OR** (`gpu` label exists).
</details>

---

### Question 4
What does the `IgnoredDuringExecution` suffix in `requiredDuringSchedulingIgnoredDuringExecution` mean?

A. `kubelet` ignores container exit codes after scheduling.  
B. If a node's labels are changed or removed **after** a Pod is already scheduled and running on that node, the Pod will **continue running** on the node and will not be evicted.  
C. Taints are ignored when the Pod is running.  
D. Resource limits are ignored during scheduling.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
`requiredDuringScheduling` means the affinity rule must be met at the time `kube-scheduler` binds the Pod to a node. `IgnoredDuringExecution` means that once bound, removing the matching label from the node will not cause the running Pod to be evicted.
</details>

---

### Question 5
You have a 3-node cluster (`worker-01`, `worker-02`, `worker-03`) and deploy a `redis-cluster` Deployment with **`4` replicas** configured with the following `podAntiAffinity`:
```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app: redis-cluster
      topologyKey: "kubernetes.io/hostname"
```
What will happen when this Deployment is applied?

A. All 4 Pods will be `Running` (2 on `worker-01`, 1 on `worker-02`, 1 on `worker-03`).  
B. 3 Pods will be `Running` (1 on each of the 3 worker nodes), and the **4th Pod will remain stuck in `Pending`** indefinitely because no 4th node with a distinct `kubernetes.io/hostname` exists.  
C. The 4th Pod will run on the control-plane node regardless of its `NoSchedule` taint.  
D. The Deployment will be rejected by `kube-apiserver`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
Because the anti-affinity rule uses `requiredDuringSchedulingIgnoredDuringExecution` (hard requirement) with `topologyKey: kubernetes.io/hostname`, no two Pods with `app: redis-cluster` can ever share the same node hostname. Once 3 Pods occupy the 3 worker nodes, the 4th replica cannot satisfy the hard anti-affinity constraint and remains `Pending`.
</details>

---

### Question 6
How can a Pod be scheduled onto a worker node `worker-01` if the `kube-scheduler` static pod in `kube-system` has crashed or been deleted?

A. It is impossible to run new Pods while `kube-scheduler` is down.  
B. Set `spec.nodeName: worker-01` directly in the Pod manifest, which bypasses `kube-scheduler` so `kubelet` on `worker-01` starts the Pod immediately.  
C. Set `spec.nodeSelector: { "kubernetes.io/hostname": "worker-01" }`.  
D. Set `spec.schedulerName: none`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
`nodeSelector` and `nodeAffinity` still require `kube-scheduler` to evaluate the rules and write `spec.nodeName`. However, setting `spec.nodeName: worker-01` directly in the Pod spec bypasses `kube-scheduler` entirely—the `kubelet` on `worker-01` sees `spec.nodeName == worker-01` in `kube-apiserver` and starts the containers immediately.
</details>

---

### Question 7
You want to write a `toleration` in a Pod spec that matches **any** taint on a node that has `key: "special-hardware"` and `effect: "NoSchedule"`, **regardless of the taint's `value`**. Which toleration configuration is valid?

A. `key: "special-hardware"`, `operator: "Equal"`, `value: "*"`, `effect: "NoSchedule"`  
B. `key: "special-hardware"`, `operator: "Exists"`, `effect: "NoSchedule"` (with `value` omitted)  
C. `key: "special-hardware"`, `operator: "In"`, `effect: "NoSchedule"`  
D. `key: "*"`, `operator: "Equal"`, `effect: "NoSchedule"`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
Toleration operators are only `Equal` (default) or `Exists`. When using `operator: "Exists"`, you **must omit** the `value` field, and it matches any taint with the specified `key` and `effect` (or if `key` and `effect` are also omitted with `operator: "Exists"`, it tolerates every taint in the cluster!).
</details>

---

### Question 8
What is the command to **remove** the taint `dedicated=experimental:NoSchedule` from node `worker-02`?

A. `kubectl untaint nodes worker-02 dedicated=experimental:NoSchedule`  
B. `kubectl taint nodes worker-02 dedicated=experimental:NoSchedule-`  
C. `kubectl taint nodes worker-02 dedicated=false`  
D. `kubectl label nodes worker-02 dedicated-`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
Just like `kubectl label`, removing a taint with `kubectl taint` uses a trailing minus sign (`-`) at the end of the taint specification:
`kubectl taint nodes worker-02 dedicated=experimental:NoSchedule-` (or `kubectl taint nodes worker-02 dedicated-` to remove all taints with key `dedicated`).
</details>

---

### Question 9
A cluster has a `Deployment` named `etcd-gateway` with `3` replicas (`3/3 Ready`) and a `PodDisruptionBudget` (`PDB`) configured with `minAvailable: 3`. What happens when an administrator runs `kubectl drain worker-01 --ignore-daemonsets`, where one of the `etcd-gateway` Pods is currently running?

A. `kubectl drain` ignores the PDB and immediately evicts the Pod.  
B. `kubectl drain` blocks and repeatedly logs `error when evicting pods/"etcd-gateway-..." - Cannot evict pod as it would violate the pod's disruption budget` because terminating 1 of the 3 running Pods would drop available Pods to 2 (violating `minAvailable: 3`).  
C. The PDB automatically scales the Deployment up to 4 replicas first.  
D. The node is Rebooted instead.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
If a Deployment has `replicas: 3` and its `PodDisruptionBudget` requires `minAvailable: 3` (or `maxUnavailable: 0`), then `ALLOWED DISRUPTIONS` is `0`! Because a Deployment replica replacement is only created *after* or during eviction, evicting 1 Pod drops ready Pods to 2, so the Eviction API rejects `kubectl drain`. Either lower `minAvailable: 2` on the PDB or scale the Deployment to `4` replicas before draining.
</details>

---

### Question 10
You create a `PriorityClass` named `high-priority-non-preempting` with `value: 500000` and `preemptionPolicy: Never`. How does `kube-scheduler` treat a `Pending` Pod that uses this `PriorityClass` when cluster nodes are full?

A. The Pod is placed at the **front of the scheduling queue** ahead of lower-priority `Pending` Pods, but `kube-scheduler` will **not** evict (preempt) any currently running lower-priority Pods to make room for it.  
B. The Pod immediately evicts `BestEffort` Pods.  
C. The `PriorityClass` value is ignored completely.  
D. The Pod bypasses `ResourceQuota`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: A**

**Explanation:**  
Setting `preemptionPolicy: Never` on a `PriorityClass` gives the Pod priority ordering in the scheduling queue (so it gets the next available resources before lower-priority `Pending` Pods) without disrupting/evicting already-running workloads.
</details>

---

### Question 11
Which built-in `PriorityClass` has the highest priority (`value: 2000001000`) and is used by Kubernetes to ensure that critical node-level pods like `kube-apiserver`, `etcd`, and `calico-node` are never preempted?

A. `system-cluster-critical`  
B. `system-node-critical`  
C. `default-global`  
D. `kube-system-high`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
Kubernetes includes two built-in system `PriorityClasses`:
- `system-node-critical` (`value: 2000001000` — highest possible priority, used for static control-plane pods and CNI/kube-proxy).
- `system-cluster-critical` (`value: 2000000000` — used for cluster add-ons like `CoreDNS` and `metrics-server`).
</details>

---

### Question 12
Why are regular workload Pods not scheduled onto the Control Plane node in a default `kubeadm` cluster, and how could you allow Pods to schedule onto a single-node `kubeadm` cluster?

A. Control plane nodes have `cordon` enabled by default; run `kubectl uncordon`.  
B. Control plane nodes have the taint `node-role.kubernetes.io/control-plane:NoSchedule`; remove it with `kubectl taint nodes <cp-node> node-role.kubernetes.io/control-plane:NoSchedule-`.  
C. Control plane nodes do not run `kubelet` or `containerd`.  
D. Control plane nodes have `maxPods: 4` in `/var/lib/kubelet/config.yaml`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
`kubeadm init` automatically applies the taint `node-role.kubernetes.io/control-plane:NoSchedule` to the control plane node. Removing that taint (or adding a matching toleration to a Pod) allows standard workloads to schedule onto the control plane node.
</details>
