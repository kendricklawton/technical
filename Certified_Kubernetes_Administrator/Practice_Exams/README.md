# Certified Kubernetes Administrator (CKA) — Full-Length Performance Practice Exams

Welcome to the CKA Practice Exam Simulation Hub. These three full-length, 120-minute performance-based mock exams replicate the exact task density, cluster topology, domain weights, and troubleshooting complexity of the live CNCF Certified Kubernetes Administrator exam.

---

## 1. Exam Blueprint & Scoring Matrix

| Parameter | Official CNCF CKA Standard | This Practice Suite |
| :--- | :--- | :--- |
| **Duration** | 120 Minutes (2 Hours) | 120 Minutes per Exam |
| **Number of Tasks** | 15–20 Performance Tasks | **17 Tasks per Exam (51 Total Tasks)** |
| **Passing Score** | **66%** | Target **80%+ (80/100 points)** before booking |
| **Environment** | Remote Linux Desktop (XFCE), Multi-Cluster `kubectl` contexts | 6 Cluster Contexts (`k8s-c1-H`, `k8s-c2-AC`, `k8s-c3-CCC`, `k8s-efk`, `k8s-stor`, `k8s-net`) |

### Domain Weight Breakdown

| Domain | Weight | Points per Exam | Core Skills Tested |
| :--- | :---: | :---: | :--- |
| **1. Cluster Architecture, Installation & Configuration** | **25%** | 25 pts | `kubeadm` upgrades, `etcd` snapshot backup/restore, RBAC (`Role`/`ClusterRole`), Kubeconfig contexts |
| **2. Workloads & Scheduling** | **15%** | 15 pts | Deployments, DaemonSets, Taints/Tolerations, NodeAffinity, Probes, Multi-Container Pods (`sidecar`/`init`) |
| **3. Services & Networking** | **20%** | 20 pts | ClusterIP/NodePort, NetworkPolicies (AND/OR logic), Ingress TLS, **Gateway API** (`Gateway`/`HTTPRoute`), CoreDNS |
| **4. Storage** | **10%** | 10 pts | PVs, PVCs, StorageClasses (`WaitForFirstConsumer`), CSI Volume Expansion, StatefulSet volumeClaimTemplates |
| **5. Troubleshooting** | **30%** | 30 pts | Broken `kube-apiserver` static pods, `NotReady` worker nodes (`kubelet`/`containerd`), `CrashLoopBackOff`, CNI/Service triage |

---

## 2. Practice Exam Suite

| Exam | Focus Profile | Tasks | Time Limit | Target Score |
| :--- | :--- | :---: | :---: | :---: |
| **[Practice Exam 1: Core Cluster Operations & Recovery](Practice_Exam_1.md)** | Control-plane static pod recovery, `kubeadm` v1.30 $\to$ v1.31 upgrade, `etcd` backup/restore, Gateway API `HTTPRoute`, RWO/RWX PVCs, and Node `NotReady` triage | 17 Tasks | 120 mins | $\ge 80 / 100$ |
| **[Practice Exam 2: Advanced Scheduling, Networking & Multi-Node Triage](Practice_Exam_2.md)** | Zero-downtime node drain, native sidecar containers (`restartPolicy: Always`), NetworkPolicy egress/ingress lockdown, CSI online volume expansion, and `kube-controller-manager` recovery | 17 Tasks | 120 mins | $\ge 80 / 100$ |
| **[Practice Exam 3: Chaos Simulation & High-Pressure Troubleshooting](Practice_Exam_3.md)** | Expired control-plane PKI certs, `etcd` database space quota alarm, CoreDNS loop resolution, Gateway API canary traffic splitting (`90/10`), and `ip_forward` / CNI repair | 17 Tasks | 120 mins | $\ge 80 / 100$ |

---

## 3. First-60-Seconds Terminal Setup (Run at the Start of Every Exam)

Execute this block immediately when your exam terminal opens to maximize speed and prevent YAML indentation errors:

```bash
# 1. Enable kubectl autocompletion and alias 'k'
source <(kubectl completion bash)
alias k=kubectl
complete -o default -F __start_kubectl k

# 2. Export rapid imperative generation variables
export do="--dry-run=client -o yaml"
export now="--force --grace-period=0"

# 3. Configure ~/.vimrc for clean 2-space YAML indentation
cat << 'EOF' > ~/.vimrc
set expandtab
set tabstop=2
set shiftwidth=2
set softtabstop=2
set autoindent
set smartindent
set number
EOF
```

---

## 4. Golden Exam Execution Rules

1. **Always Run the Context Switch Command First:** Every task begins with `kubectl config use-context <cluster>`. Skipping this scores **0 points** even if your solution is technically flawless.
2. **Exit SSH Sessions Immediately After Host Tasks:** When you `ssh cp-01` or `ssh worker-02` to fix `kubelet` or `etcd`, run `exit` as soon as you finish so you don't accidentally run the next task inside a worker node shell.
3. **Never Edit Static Pod Manifests Without Backing Them Up First:**
   ```bash
   sudo cp /etc/kubernetes/manifests/kube-apiserver.yaml /root/kube-apiserver.yaml.bak
   ```
   Make backups in `/root/`, **never** inside `/etc/kubernetes/manifests/` (or the kubelet will attempt to run both files as duplicate static pods!).
4. **Enforce the 7-Minute Timebox:** If a 4% task takes longer than 7 minutes, flag it and move immediately to high-weight 7%–8% tasks (`etcd` backup, `kubeadm` upgrade, `NotReady` node troubleshooting).

---

## 5. Personal Score Tracker

| Attempt Date | Exam # | Score (/100) | Time Used | Missed Tasks / Weak Domains |
| :--- | :---: | :---: | :---: | :--- |
| `YYYY-MM-DD` | Exam 1 | `___ / 100` | `___ mins` | |
| `YYYY-MM-DD` | Exam 2 | `___ / 100` | `___ mins` | |
| `YYYY-MM-DD` | Exam 3 | `___ / 100` | `___ mins` | |
