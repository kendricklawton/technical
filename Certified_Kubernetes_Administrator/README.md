# Certified Kubernetes Administrator (CKA) — Complete Study & Lab Workspace

Welcome to the comprehensive **Certified Kubernetes Administrator (CKA)** preparation repository, aligned with the **2025/2026 CNCF CKA Curriculum** (Kubernetes v1.31 / v1.32+). This workspace contains production-grade study guides, architectural diagrams, imperative command references, 30 hands-on lab scenarios, 180 deep scenario quiz questions, and 3 full-length 17-task performance-based mock exams.

---

## Exam Overview & Format

| Attribute | Specification |
| :--- | :--- |
| **Certification Name** | Certified Kubernetes Administrator (CKA) |
| **Exam Type** | Online, Proctored, Performance-Based (100% Hands-On CLI in Remote Terminal) |
| **Duration** | **120 Minutes** (2 Hours) |
| **Number of Tasks** | **15–20 Tasks** (Typically 17 Performance Tasks across 6 Kubernetes Clusters) |
| **Passing Score** | **66%** (660 / 1000 points) |
| **Kubernetes Version** | Current minor release (e.g., v1.31 / v1.32) |
| **Environment** | Ubuntu Linux Desktop via PSI Bridge Browser (`tmux`, `vim`, `nano`, `kubectl`, `helm`, `etcdctl`, `crictl`, `kubeadm`) |
| **Allowed Documentation** | `https://kubernetes.io/docs/`, `https://kubernetes.io/blog/`, `https://helm.sh/docs/`, `https://gateway-api.sigs.k8s.io/` |

> [!IMPORTANT]
> **2025/2026 CKA Curriculum Updates:**
> The updated CKA exam places heavy emphasis on **Kubernetes Gateway API** (`GatewayClass`, `Gateway`, `HTTPRoute`), **Helm** chart lifecycle management, **Kustomize** overlays and patches, **Custom Resource Definitions (CRDs) & Operators**, **Native Sidecar Containers** (`initContainers` with `restartPolicy: Always`), and **Control Plane / Node Troubleshooting** (30% of the total score).

---

## CKA Curriculum Domains & Weights

```mermaid
flowchart LR
    CKA["CKA Exam (100%)"]
    D1["Domain 1: Cluster Architecture, Installation & Config (25%)"]
    D2["Domain 2: Workloads & Scheduling (15%)"]
    D3["Domain 3: Services & Networking (20%)"]
    D4["Domain 4: Storage (10%)"]
    D5["Domain 5: Troubleshooting (30%)"]

    CKA --> D1
    CKA --> D2
    CKA --> D3
    CKA --> D4
    CKA --> D5
```

| Domain | Weight | Core Competencies | Workspace Directory |
| :--- | :---: | :--- | :--- |
| **Domain 1: Cluster Architecture, Installation & Configuration** | **25%** | RBAC, `kubeadm` install & upgrades, HA Control Plane, `etcd` backup/restore, CRDs, Operators, Helm, Kustomize | [`Domain_1_Cluster_Architecture_Installation_Configuration/`](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_1_Cluster_Architecture_Installation_Configuration) |
| **Domain 2: Workloads & Scheduling** | **15%** | Pod lifecycle, Init & Native Sidecar containers, Deployments, DaemonSets, StatefulSets, Jobs/CronJobs, ConfigMaps/Secrets, Probes, QoS, Taints/Tolerations, Node/Pod Affinity, PriorityClass, PDB, HPA | [`Domain_2_Workloads_Scheduling/`](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_2_Workloads_Scheduling) |
| **Domain 3: Services & Networking** | **20%** | CNI plugins (Calico/Cilium), Services (`ClusterIP`/`NodePort`/`LoadBalancer`/Headless), `EndpointSlices`, CoreDNS, `Ingress`, **Gateway API** (`GatewayClass`/`Gateway`/`HTTPRoute`), `NetworkPolicy` | [`Domain_3_Services_Networking/`](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_3_Services_Networking) |
| **Domain 4: Storage** | **10%** | Volumes (`emptyDir`/`hostPath`/`projected`), PV & PVC lifecycle, AccessModes (`RWO`/`RWX`/`ROX`/`RWOP`), Reclaim Policies, `StorageClass`, CSI `WaitForFirstConsumer`, Online PVC expansion, `VolumeSnapshot` | [`Domain_4_Storage/`](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_4_Storage) |
| **Domain 5: Troubleshooting** | **30%** | Static Pod failures (`/etc/kubernetes/manifests/`), `crictl` debugging when API server is down, `NotReady` worker nodes (`kubelet`/`containerd`/cgroups/certs), Pod `CrashLoopBackOff`/`Pending`/`OOMKilled`, Service/DNS/NetworkPolicy triage, `kubectl debug` | [`Domain_5_Troubleshooting/`](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_5_Troubleshooting) |

---

## Quick Navigation Matrix

### Master Reference & Mock Exams
- **[CKA Master Cheat Sheet](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/CKA_Master_Cheat_Sheet.md)** — Exam-day shell setup, imperative `kubectl` commands, JSONPath templates, `etcdctl` backup/restore, `kubeadm` upgrade sequence, and `crictl`/`journalctl` triage commands.
- **[Practice Exams Overview](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Practice_Exams/README.md)** — Instructions, timing strategies, and cluster topology for the 3 full-length mock exams.
  - **[Practice Exam 1 (Standard Exam Simulation — 17 Tasks)](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Practice_Exams/Practice_Exam_1.md)**
  - **[Practice Exam 2 (Production Scenarios — 17 Tasks)](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Practice_Exams/Practice_Exam_2.md)**
  - **[Practice Exam 3 (Hard/Challenge Mode — 17 Tasks)](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Practice_Exams/Practice_Exam_3.md)**

### Domain Study Guides, Labs & Quizzes

| Domain | Study Guide | Hands-On Labs (6 Scenarios) | Quiz 1 | Quiz 2 | Quiz 3 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Domain 1 (25%)** | [Study Guide](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_1_Cluster_Architecture_Installation_Configuration/Study_Guide.md) | [Exercises](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_1_Cluster_Architecture_Installation_Configuration/Exercises.md) | [Quiz 1](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_1_Cluster_Architecture_Installation_Configuration/Quiz_1.md) | [Quiz 2](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_1_Cluster_Architecture_Installation_Configuration/Quiz_2.md) | [Quiz 3](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_1_Cluster_Architecture_Installation_Configuration/Quiz_3.md) |
| **Domain 2 (15%)** | [Study Guide](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_2_Workloads_Scheduling/Study_Guide.md) | [Exercises](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_2_Workloads_Scheduling/Exercises.md) | [Quiz 1](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_2_Workloads_Scheduling/Quiz_1.md) | [Quiz 2](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_2_Workloads_Scheduling/Quiz_2.md) | [Quiz 3](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_2_Workloads_Scheduling/Quiz_3.md) |
| **Domain 3 (20%)** | [Study Guide](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_3_Services_Networking/Study_Guide.md) | [Exercises](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_3_Services_Networking/Exercises.md) | [Quiz 1](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_3_Services_Networking/Quiz_1.md) | [Quiz 2](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_3_Services_Networking/Quiz_2.md) | [Quiz 3](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_3_Services_Networking/Quiz_3.md) |
| **Domain 4 (10%)** | [Study Guide](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_4_Storage/Study_Guide.md) | [Exercises](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_4_Storage/Exercises.md) | [Quiz 1](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_4_Storage/Quiz_1.md) | [Quiz 2](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_4_Storage/Quiz_2.md) | [Quiz 3](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_4_Storage/Quiz_3.md) |
| **Domain 5 (30%)** | [Study Guide](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_5_Troubleshooting/Study_Guide.md) | [Exercises](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_5_Troubleshooting/Exercises.md) | [Quiz 1](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_5_Troubleshooting/Quiz_1.md) | [Quiz 2](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_5_Troubleshooting/Quiz_2.md) | [Quiz 3](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_5_Troubleshooting/Quiz_3.md) |

---

## Recommended 4-Week Study Plan & Progress Tracker

### Week 1: Architecture, Control Plane & Cluster Operations (Domain 1)
- [ ] Read [Domain 1 Study Guide](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_1_Cluster_Architecture_Installation_Configuration/Study_Guide.md)
- [ ] Complete all 6 labs in [Domain 1 Exercises](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_1_Cluster_Architecture_Installation_Configuration/Exercises.md) (`kubeadm` upgrade, `etcd` snapshot backup/restore, RBAC, Helm, Kustomize, CRDs)
- [ ] Score 85%+ on [Domain 1 Quiz 1](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_1_Cluster_Architecture_Installation_Configuration/Quiz_1.md), [Quiz 2](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_1_Cluster_Architecture_Installation_Configuration/Quiz_2.md), and [Quiz 3](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_1_Cluster_Architecture_Installation_Configuration/Quiz_3.md)

### Week 2: Workloads, Scheduling & Storage (Domains 2 & 4)
- [ ] Read [Domain 2 Study Guide](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_2_Workloads_Scheduling/Study_Guide.md)
- [ ] Complete all 6 labs in [Domain 2 Exercises](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_2_Workloads_Scheduling/Exercises.md)
- [ ] Score 85%+ on [Domain 2 Quiz 1](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_2_Workloads_Scheduling/Quiz_1.md), [Quiz 2](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_2_Workloads_Scheduling/Quiz_2.md), and [Quiz 3](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_2_Workloads_Scheduling/Quiz_3.md)
- [ ] Read [Domain 4 Study Guide](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_4_Storage/Study_Guide.md)
- [ ] Complete all 6 labs in [Domain 4 Exercises](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_4_Storage/Exercises.md)
- [ ] Score 85%+ on [Domain 4 Quiz 1](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_4_Storage/Quiz_1.md), [Quiz 2](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_4_Storage/Quiz_2.md), and [Quiz 3](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_4_Storage/Quiz_3.md)

### Week 3: Services, Gateway API, NetworkPolicies & Troubleshooting (Domains 3 & 5)
- [ ] Read [Domain 3 Study Guide](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_3_Services_Networking/Study_Guide.md)
- [ ] Complete all 6 labs in [Domain 3 Exercises](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_3_Services_Networking/Exercises.md)
- [ ] Score 85%+ on [Domain 3 Quiz 1](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_3_Services_Networking/Quiz_1.md), [Quiz 2](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_3_Services_Networking/Quiz_2.md), and [Quiz 3](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_3_Services_Networking/Quiz_3.md)
- [ ] Read [Domain 5 Study Guide](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_5_Troubleshooting/Study_Guide.md)
- [ ] Complete all 6 labs in [Domain 5 Exercises](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_5_Troubleshooting/Exercises.md)
- [ ] Score 85%+ on [Domain 5 Quiz 1](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_5_Troubleshooting/Quiz_1.md), [Quiz 2](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_5_Troubleshooting/Quiz_2.md), and [Quiz 3](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Domain_5_Troubleshooting/Quiz_3.md)

### Week 4: Speed Drills & Full-Length Mock Exams
- [ ] Memorize all imperative workflows in [CKA Master Cheat Sheet](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/CKA_Master_Cheat_Sheet.md)
- [ ] Complete [Practice Exam 1](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Practice_Exams/Practice_Exam_1.md) under strict 120-minute timer (Target: $\ge 75\%$)
- [ ] Complete [Practice Exam 2](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Practice_Exams/Practice_Exam_2.md) under strict 120-minute timer (Target: $\ge 80\%$)
- [ ] Complete [Practice Exam 3](file:///home/k-henry/repos/technical/Certified_Kubernetes_Administrator/Practice_Exams/Practice_Exam_3.md) under strict 120-minute timer (Target: $\ge 85\%$)

---

## Local Lab Cluster Setup Options

You can run every lab and mock exam in this repository locally using **`kind`** (Kubernetes in Docker), **`minikube`**, or **`kubeadm`** multi-node VMs (via Multipass, Vagrant, or KVM/Libvirt).

### Option A: Multi-Node `kind` Cluster (Fastest for Domains 2, 3, 4, and App Troubleshooting)
```bash
cat <<EOF > kind-cka-cluster.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
networking:
  disableDefaultCNI: false
  podSubnet: "192.168.0.0/16"
EOF

kind create cluster --name cka-lab --config kind-cka-cluster.yaml
```

### Option B: Multi-VM `kubeadm` Cluster with Multipass (Best for Domain 1 & Control Plane/Node Troubleshooting)
```bash
multipass launch --name cka-cp1 --cpus 2 --memory 4G --disk 20G 24.04
multipass launch --name cka-worker1 --cpus 2 --memory 3G --disk 20G 24.04
multipass launch --name cka-worker2 --cpus 2 --memory 3G --disk 20G 24.04
```

---

## Golden Exam-Day Rules

1. **Always Execute the Context Switch First:** Every single question provides a `kubectl config use-context <cluster-name>` command at the top. Run it before typing anything else.
2. **Never Write YAML from Scratch:** Use imperative generators (`kubectl create ... $do > task.yaml` or `kubectl run ... $do > pod.yaml`) and edit only the fields that cannot be generated imperatively.
3. **Use `kubectl explain --recursive`:** Instead of wasting 3 minutes searching browser docs for exact field names or capitalization, run `kubectl explain pod.spec.affinity.nodeAffinity` directly in the terminal.
4. **Return to the Base Jump Host after SSHing:** Whenever you `ssh node01` or `sudo -i` to fix a control plane or worker node, immediately `exit` back to the candidate jump host (`student@base`) before moving to the next question.
5. **Manage Your Clock:** You have ~7 minutes per task across 17 tasks. If a low-weight task (e.g., 4%) takes longer than 6 minutes, flag it, save your manifest in `/root/q<num>.yaml`, and come back after finishing all high-weight tasks (7%–8%).
