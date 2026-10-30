# Domain 2 — Quiz 1: Pods, Native Sidecar Containers, Deployments, DaemonSets, StatefulSets & Jobs

Test your mastery of Kubernetes workload controllers, Pod lifecycle, and Native Sidecar Containers.

---

### Question 1
In Kubernetes 1.29+, how do you configure a container named `log-collector` so that it runs as a **Native Sidecar Container**—meaning it starts **before** the main application container, runs continuously for the entire lifecycle of the Pod, and terminates **after** the main application container exits?

A. Place `log-collector` in `spec.containers[]` and add `sidecar.kubernetes.io/inject: "true"` to the Pod's annotations.  
B. Place `log-collector` inside `spec.initContainers[]` and set `restartPolicy: Always` on that specific initContainer entry.  
C. Place `log-collector` inside `spec.sidecarContainers[]`.  
D. Set the Pod-level `spec.restartPolicy: Always` and place `log-collector` first in `spec.containers[]`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
Kubernetes implements Native Sidecar Containers by allowing `restartPolicy: Always` on individual entries inside `spec.initContainers[]`. Unlike standard init containers (which must exit `0` before the next container starts), an init container with `restartPolicy: Always` stays running while subsequent init containers and main `spec.containers[]` start, supports `startupProbe`/`readinessProbe`/`livenessProbe`, does not block `Job` completion, and terminates last during Pod shutdown.
</details>

---

### Question 2
A Pod has **two** standard `initContainers` (`init-1` and `init-2` without `restartPolicy: Always`) and **two** main `containers` (`app-1` and `app-2`). What is the exact execution behavior when the Pod starts?

A. `init-1` and `init-2` run in parallel; once both succeed, `app-1` and `app-2` start sequentially.  
B. `init-1` runs to completion (`Exit 0`), then `init-2` runs to completion (`Exit 0`), and only after `init-2` succeeds do `app-1` and `app-2` start in parallel.  
C. All 4 containers start simultaneously.  
D. If `init-1` fails with a non-zero exit code and the Pod has `restartPolicy: Never`, `init-2` still runs.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
Standard `initContainers` always execute **sequentially one at a time** in the order defined in `spec.initContainers[]`. Each must exit `0` before the next starts. Once all `initContainers` have succeeded (or started, for sidecars), all main containers in `spec.containers[]` are started concurrently.
</details>

---

### Question 3
A `Deployment` has `replicas: 8` and uses a `RollingUpdate` strategy with `maxUnavailable: 25%` and `maxSurge: 25%`. During a rolling update to a new container image, what are the **minimum number of available Pods** and the **maximum total number of Pods** that can exist at any point during the rollout?

A. Minimum available: `6`, Maximum total: `10`  
B. Minimum available: `8`, Maximum total: `10`  
C. Minimum available: `6`, Maximum total: `8`  
D. Minimum available: `4`, Maximum total: `12`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: A**

**Explanation:**  
- `maxUnavailable: 25%` of `8` = $8 \times 0.25 = 2$. Minimum available Pods = $8 - 2 = 6$.  
- `maxSurge: 25%` of `8` = $8 \times 0.25 = 2$. Maximum total Pods = $8 + 2 = 10$.
</details>

---

### Question 4
You need to update a `Deployment` where two different versions of the application **must never run at the same time** because the database schema only supports a single version connected at once. Which Deployment strategy should you configure?

A. `type: RollingUpdate` with `maxSurge: 1` and `maxUnavailable: 0`  
B. `type: Recreate`  
C. `type: BlueGreen`  
D. `type: OrderedReady`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
`spec.strategy.type: Recreate` terminates **all** existing Pods of the old ReplicaSet completely before creating any Pods in the new ReplicaSet, guaranteeing that old and new versions never run simultaneously.
</details>

---

### Question 5
You have a `StatefulSet` named `kafka` with `3` replicas (`kafka-0`, `kafka-1`, `kafka-2`) in the `messaging` namespace, backed by a Headless Service (`clusterIP: None`) named `kafka-headless`. What is the exact internal DNS A-record FQDN for reaching Pod `kafka-1` directly?

A. `kafka-1.messaging.svc.cluster.local`  
B. `kafka-1.kafka.messaging.svc.cluster.local`  
C. `kafka-1.kafka-headless.messaging.svc.cluster.local`  
D. `kafka-headless.kafka-1.messaging.pod.cluster.local`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**  
Every Pod in a `StatefulSet` paired with a governing Headless Service (`spec.serviceName: kafka-headless`) gets a deterministic DNS hostname following the pattern:
`<pod-name>.<headless-service-name>.<namespace>.svc.cluster.local`
Here: `kafka-1.kafka-headless.messaging.svc.cluster.local`.
</details>

---

### Question 6
What happens to the PersistentVolumeClaims (PVCs) created by a `StatefulSet`'s `volumeClaimTemplates` when you scale the `StatefulSet` down from `3` replicas to `1` replica, or when you delete the `StatefulSet` entirely?

A. The PVCs for `ordinal-1` and `ordinal-2` are immediately deleted.  
B. The PVCs are **retained** (never automatically deleted by scaling down or deleting the `StatefulSet` in default configuration) to prevent accidental data loss.  
C. The PVCs are converted into `emptyDir` volumes.  
D. The PVCs are detached and bound to `ordinal-0`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
By default (`persistentVolumeClaimRetentionPolicy.whenScaled: Retain` and `whenDeleted: Retain`), scaling down or deleting a `StatefulSet` only deletes the Pods; the associated `PersistentVolumeClaims` remain bound to preserve stateful data. If you later scale the `StatefulSet` back up to `3`, `kafka-1` and `kafka-2` automatically reattach to their existing PVCs.
</details>

---

### Question 7
A `Job` named `report-generator` is configured with `completions: 6`, `parallelism: 2`, and `backoffLimit: 3`. What does this configuration do?

A. It runs 6 Pods simultaneously and restarts each up to 2 times.  
B. It runs up to 2 Pods concurrently at a time until a total of 6 Pods have exited with code `0`, and marks the Job as `Failed` if more than 3 Pod failures occur.  
C. It runs 2 Pods every 6 minutes.  
D. It terminates the Job after 6 seconds.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
- `completions: 6`: The Job succeeds only when 6 total Pods have terminated cleanly (`Exit 0`).
- `parallelism: 2`: At most 2 worker Pods run at the same time.
- `backoffLimit: 3`: Maximum number of failed Pod attempts before the Job controller stops retrying and marks the Job `Failed`.
</details>

---

### Question 8
A `CronJob` runs a database synchronization script every 10 minutes (`*/10 * * * *`). Occasionally, the synchronization takes 14 minutes to complete, and you must guarantee that a second synchronization Job is **never** started while the previous Job is still running. Which field and value in `CronJob.spec` enforces this?

A. `parallelism: 1`  
B. `concurrencyPolicy: Forbid`  
C. `concurrencyPolicy: Replace`  
D. `restartPolicy: Never`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
- `concurrencyPolicy: Allow` (default) allows overlapping Jobs to run concurrently.
- `concurrencyPolicy: Forbid` skips the new scheduled run if the previous Job has not yet finished.
- `concurrencyPolicy: Replace` kills the currently running Job and starts a new one.
</details>

---

### Question 9
When a Pod is deleted (e.g., during a rolling update or `kubectl delete pod`), in what order do the termination steps occur on the node?

A. `SIGKILL` is sent immediately, then `preStop` hook runs.  
B. Simultaneously: Pod is marked `Terminating` (EndpointSlice controller removes IP from Services) AND `kubelet` runs any container `preStop` hook $\to$ sends `SIGTERM` to PID 1 in the container $\to$ waits up to remaining `terminationGracePeriodSeconds` (default 30s) $\to$ sends `SIGKILL` if still running.  
C. `SIGTERM` is sent first, wait 30s, then `preStop` hook runs.  
D. The Pod is removed from `etcd` before containers stop.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
During graceful Pod termination:
1. Pod gets a `deletionTimestamp` (`Terminating` state) and `EndpointSlice` controller removes it from active Service load-balancing endpoints.
2. Concurrently, `kubelet` executes the `lifecycle.preStop` hook (if configured).
3. After `preStop` finishes, `kubelet` sends `SIGTERM` to PID 1 of each container.
4. If containers haven't exited when `terminationGracePeriodSeconds` (default `30` seconds, which includes `preStop` execution time) expires, `kubelet` sends `SIGKILL`.
</details>

---

### Question 10
You want to deploy a node-level security agent Pod on **every** node in your cluster, and ensure that whenever a new worker node joins the cluster in the future, it automatically gets a copy of this Pod. However, `kubectl create` does not have a `kubectl create daemonset` subcommand. What is the fastest imperative way to generate a valid `DaemonSet` YAML template during the CKA exam?

A. Run `kubectl create deployment sec-agent --image=security/agent:v1 --dry-run=client -o yaml > ds.yaml`, then edit `ds.yaml` to change `kind: Deployment` to `kind: DaemonSet` and remove `replicas` and `strategy`.  
B. Run `kubectl run sec-agent --image=security/agent:v1 --schedule="* * * * *"`  
C. Create a `ReplicaSet` with `replicas: 100`.  
D. Use `kubeadm init phase addon`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: A**

**Explanation:**  
A `Deployment` and a `DaemonSet` share the exact same `apiVersion: apps/v1`, `spec.selector.matchLabels`, and `spec.template` structure! The only difference is that a `DaemonSet` does not have `spec.replicas` or `spec.strategy` (it uses `spec.updateStrategy` instead). Generating a Deployment with `$do` and deleting those 2 fields takes less than 15 seconds.
</details>

---

### Question 11
A Pod is stuck in `CrashLoopBackOff`. Its container executes a one-line shell script `echo "Backup finished"` that exits with status code `0` after 1 second, and the Pod has the default `restartPolicy: Always`. Why is it entering `CrashLoopBackOff` even though the command succeeded (`Exit 0`)?

A. `echo` is not a valid container entrypoint.  
B. With `restartPolicy: Always` (the default for Pods and mandatory for Deployments), `kubelet` restarts any container that exits—even with exit code `0`! Repeatedly exiting immediately triggers exponential backoff (`CrashLoopBackOff`).  
C. The container ran out of memory.  
D. `kube-scheduler` is restarting the Pod.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
A container stays `Running` only as long as its PID 1 foreground process stays alive. With `restartPolicy: Always`, whenever PID 1 exits (even with `0`), `kubelet` restarts it with exponential delay (`10s`, `20s`, `40s`... up to `5m`), displaying `CrashLoopBackOff`. One-shot tasks that exit `0` should use a `Job` with `restartPolicy: OnFailure` or `Never`.
</details>

---

### Question 12
You pause a Deployment (`kubectl rollout pause deploy/web`) to apply multiple updates (updating the container image AND updating CPU/memory resource limits) before triggering a single rollout. How many new `ReplicaSets` are created while the Deployment is paused, and how do you start the rollout?

A. Two new `ReplicaSets` are created immediately; run `kubectl rollout restart`.  
B. **Zero** new rollouts are triggered while paused; all changes are batched and a **single** new `ReplicaSet` rollout begins only when you run `kubectl rollout resume deploy/web`.  
C. Pausing a Deployment deletes all existing Pods until resumed.  
D. `kubectl rollout undo` must be used to unpause.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
`kubectl rollout pause` freezes the Deployment controller from reconciling `spec.template` mutations into new `ReplicaSets`, allowing you to make multiple `kubectl set image`, `kubectl set env`, and `kubectl set resources` edits and roll them out in a single revision once you run `kubectl rollout resume`.
</details>
