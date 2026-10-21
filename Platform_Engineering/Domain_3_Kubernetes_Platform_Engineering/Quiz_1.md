# PE Domain 3: Kubernetes Platform Engineering — Scenario Quiz 1

Test your knowledge on Kubernetes API machinery, etcd consensus, and CRDs.

---

### Question 1
During a high-concurrency deployment update, an automated pipeline receives an HTTP `409 Conflict` error from `kube-apiserver` while updating a ConfigMap. What mechanism in the Kubernetes API machinery triggered this error?

A) Optimistic Concurrency Control detected that the `metadata.resourceVersion` submitted in the update was stale compared to the version currently stored in etcd.  
B) etcd encountered a disk full condition.  
C) The Pod's network namespace was corrupted.  
D) RBAC permissions were revoked.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Kubernetes uses Optimistic Concurrency Control via `resourceVersion`. When a client updates an object, it must supply the latest known `resourceVersion`. If another controller modified the object in etcd in the interim, the API server rejects the incoming change with `409 Conflict`, forcing the client to re-fetch the latest object and retry.
</details>

---

### Question 2
You have a 5-node etcd cluster. A severe network partition isolates 2 etcd nodes from the remaining 3 nodes. How does this partition impact cluster write and read operations?

A) Both partitions continue processing writes independently, causing a split-brain condition.  
B) The entire etcd cluster shuts down immediately to prevent corruption.  
C) The 3-node partition maintains Raft quorum ($\lfloor 5/2 \rfloor + 1 = 3$) and continues processing writes normally; the 2-node partition cannot achieve quorum and rejects all write operations.  
D) Writes succeed only on the 2-node partition.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Raft requires a strict majority ($\lfloor N/2 \rfloor + 1$) to achieve quorum and commit writes. In a 5-node cluster, 3 nodes constitute quorum. The 3-node partition continues operating seamlessly, while the isolated 2-node partition rejects all write requests.
</details>

---

### Question 3
Why should Custom Resource Definitions (CRDs) always define `.subresources.status`?

A) It encrypts the resource using AES-256.  
B) It is mandatory for the Linux kernel to allocate memory.  
C) It enables the `/status` API subresource, which separates `.spec` changes from `.status` updates, preventing updates to operational status from incrementing `.metadata.generation` and triggering accidental spec reconciliation loops.  
D) It allows unauthenticated users to modify the CRD.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Defining `/status` decouples user intent (`.spec`) from observed state (`.status`). Updating `.status` does not increment `.metadata.generation`, allowing controllers to report health status without triggering an infinite reconcile loop.
</details>

---

### Question 4
When developing a custom Kubernetes controller using `client-go`, why must your controller read objects from an **Indexer cache / Informer** rather than issuing direct `GET` requests to `kube-apiserver`?

A) Informer caches are stored in browser local storage.  
B) Direct `GET` requests to the API server bypass etcd and read from disk.  
C) `client-go` strictly blocks all HTTP network requests.  
D) Direct API server `GET` calls put immense load on `kube-apiserver` and etcd; Informers maintain an in-memory, thread-safe cache synchronized via efficient HTTP/2 watch streams.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** If hundreds of controllers constantly polled the API server with direct `GET` calls, `kube-apiserver` and etcd would collapse under load. Informers maintain a synchronized in-memory cache populated by long-lived watch streams, enabling instant, zero-cost reads.
</details>

---

### Question 5
What is the primary difference between a **level-triggered** architecture and an **edge-triggered** architecture in Kubernetes controller design?

A) Edge-triggered controllers do not use work queues.  
B) Edge-triggered systems react solely to change events, risking state drift if an event is dropped; level-triggered systems reconcile current observed state against desired state, ensuring self-healing even if individual events are lost or coalesced.  
C) Level-triggered systems only react to state transitions; edge-triggered systems do not.  
D) Level-triggered controllers can only run in the cloud.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Kubernetes is level-triggered. Events are treated merely as notifications to enqueue a key. The Reconcile loop inspects the actual current state of the world versus desired state, guaranteeing eventual consistency regardless of lost or reordered events.
</details>

---

### Question 6
An engineer executes `kubectl delete myresource/orders-db`. The command hangs indefinitely, and `kubectl get myresource orders-db` shows `deletionTimestamp` is set, but the resource remains. What is the root cause?

A) The etcd database is frozen.  
B) The resource has one or more entries in `metadata.finalizers` that have not been removed by their respective controllers.  
C) The node running the API server has crashed.  
D) The API server requires a reboot to delete resources.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Finalizers inform the API server to postpone physical deletion until external cleanup tasks complete. When deleted, the API server sets `deletionTimestamp`, but will not purge the object from etcd until all finalizer strings are removed from `metadata.finalizers`.
</details>

---

### Question 7
In Kubernetes 1.29+, what is the purpose of embedding `x-kubernetes-validations` CEL rules in a CRD?

A) It converts the CRD into a virtual machine image.  
B) It automatically installs Prometheus metrics.  
C) It enables declarative, in-tree validation rules evaluated directly by `kube-apiserver` without incurring the latency, complexity, and operational failure risks of dynamic admission webhooks.  
D) It allows running JavaScript scripts inside the Linux kernel.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** In-tree Common Expression Language (CEL) validation allows platform engineers to define complex validation logic directly in CRD schemas, evaluating them within the API server with zero network round-trip overhead.
</details>

---

### Question 8
What happens if a custom controller crashes while processing a key from its `RateLimitingWorkQueue`?

A) The key is re-enqueued with exponential backoff and rate limiting, allowing another worker thread or a restarted controller to retry reconciliation safely.  
B) The key is permanently lost and will never be processed again.  
C) The API server rolls back the cluster state to yesterday.  
D) The entire Kubernetes cluster is placed in read-only mode.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** `client-go`'s `RateLimitingQueue` handles errors gracefully: calling `AddRateLimited(key)` re-enqueues the item with exponential backoff (e.g., 5ms, 10ms, 20ms...) up to a maximum limit, preventing crash-looping controllers from hammering the API server.
</details>

---

### Question 9
What does `metadata.generation` represent on a Kubernetes resource?

A) The Git commit hash of the manifest.  
B) The number of pods currently running.  
C) The number of years the cluster has been running.  
D) A 64-bit integer incremented by the API server whenever the object's `.spec` is modified, allowing controllers to detect whether desired state has changed.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** `metadata.generation` increments only when `.spec` changes. Changes to `.metadata` (like labels) or `.status` do not increment generation, allowing controllers to compare `metadata.generation` against `status.observedGeneration` to determine if work is needed.
</details>

---

### Question 10
What protocol does `kube-apiserver` use to stream resource watch events to controllers?

A) Raw FTP  
B) HTTP/2 with persistent chunked streaming  
C) SMTP  
D) Telnet  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Watches use persistent HTTP/2 connections over TLS, allowing the API server to push serialized JSON or Protobuf watch events to controllers in real time over a single multiplexed TCP connection.
</details>
