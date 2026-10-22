# Certified Cloud Native Platform Engineer (CNPE) — DigitalOcean Droplet Lab Provisioning Guide

> **Architecture:** 3-Node Kubernetes v1.31+ Cluster on DigitalOcean Droplets  
> **Host OS:** Ubuntu 24.04 LTS (x86_64)  
> **Droplet Spec:** `s-2vcpu-4gb` ($24/mo per node, ~$0.036/hr — under $0.50 for a full 4-hour practice exam!)  
> **Platform Stack:** Cilium CNI, Argo CD v2.12+, Argo Rollouts v1.7+, Crossplane v1.16+, Kyverno v1.12+, External Secrets Operator v0.10+, OpenTelemetry Collector, and KEDA v2.15+.

---

## 1. Automated Provisioning with `doctl` & `cloud-init`

### Step 1: Create the Cloud-Init Automation Script (`cloud-init-cnpe.yaml`)
Save the following configuration as `/tmp/cloud-init-cnpe.yaml`:

```yaml
#cloud-config
package_update: true
package_upgrade: true
packages:
  - apt-transport-https
  - ca-certificates
  - curl
  - gnupg
  - lsb-release
  - jq
  - git
  - socat
  - conntrack

runcmd:
  # 1. Disable Swap
  - swapoff -a
  - sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

  # 2. Kernel modules & sysctl for Kubernetes networking
  - modprobe overlay
  - modprobe br_netfilter
  - echo -e "overlay\nbr_netfilter" > /etc/modules-load.d/k8s.conf
  - echo -e "net.bridge.bridge-nf-call-iptables = 1\nnet.bridge.bridge-nf-call-ip6tables = 1\nnet.ipv4.ip_forward = 1" > /etc/sysctl.d/k8s.conf
  - sysctl --system

  # 3. Install containerd runtime
  - install -m 0755 -d /etc/apt/keyrings
  - curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
  - chmod a+r /etc/apt/keyrings/docker.asc
  - echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
  - apt-get update
  - apt-get install -y containerd.io
  - mkdir -p /etc/containerd
  - containerd config default > /etc/containerd/config.toml
  - sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
  - systemctl restart containerd
  - systemctl enable containerd

  # 4. Install kubeadm, kubelet, kubectl (v1.31)
  - curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
  - echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /" > /etc/apt/sources.list.d/kubernetes.list
  - apt-get update
  - apt-get install -y kubelet=1.31.2-1.1 kubeadm=1.31.2-1.1 kubectl=1.31.2-1.1 helm
  - apt-mark hold kubelet kubeadm kubectl

  # 5. Bind Kubelet to DigitalOcean VPC Private IP (eth1)
  - VPC_IP=$(ip -4 addr show eth1 | awk '/inet / {print $2}' | cut -d/ -f1)
  - echo "KUBELET_EXTRA_ARGS=--node-ip=${VPC_IP}" > /etc/default/kubelet
  - systemctl daemon-reload
  - systemctl restart kubelet
```

### Step 2: Spin Up the 3 Droplets via `doctl`
```bash
# Set your DigitalOcean SSH Key ID
SSH_KEY_ID=$(doctl compute ssh-key list --format ID --no-header | head -n 1)

# Provision Control Plane
doctl compute droplet create cnpe-cp1 \
  --region nyc3 \
  --size s-2vcpu-4gb \
  --image ubuntu-24-04-x64 \
  --enable-private-networking \
  --ssh-keys "${SSH_KEY_ID}" \
  --user-data-file /tmp/cloud-init-cnpe.yaml \
  --tag-name cnpe-lab \
  --wait

# Provision Worker Droplets
doctl compute droplet create cnpe-worker1 cnpe-worker2 \
  --region nyc3 \
  --size s-2vcpu-4gb \
  --image ubuntu-24-04-x64 \
  --enable-private-networking \
  --ssh-keys "${SSH_KEY_ID}" \
  --user-data-file /tmp/cloud-init-cnpe.yaml \
  --tag-name cnpe-lab \
  --wait
```

---

## 2. Cluster Initialization & Platform Bootstrap

### Step 3: Initialize Control Plane & Install Cilium CNI on `eth1`
SSH into `cnpe-cp1`:
```bash
# 1. Grab Private VPC IP on eth1
CP_VPC_IP=$(ip -4 addr show eth1 | awk '/inet / {print $2}' | cut -d/ -f1)

# 2. Initialize kubeadm
sudo kubeadm init \
  --apiserver-advertise-address="${CP_VPC_IP}" \
  --apiserver-cert-extra-sans="${CP_VPC_IP}" \
  --control-plane-endpoint="${CP_VPC_IP}:6443" \
  --pod-network-cidr=192.168.0.0/16 \
  --kubernetes-version=v1.31.2

# Configure kubectl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 3. Install Cilium bound to eth1
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
curl -L --fail --remote-name-all "https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-amd64.tar.gz"
sudo tar xzvfC cilium-linux-amd64.tar.gz /usr/local/bin
rm cilium-linux-amd64.tar.gz

cilium install --version 1.16.3 --set devices=eth1
```

### Step 4: Join Worker Droplets
Run the `kubeadm join` command printed by `kubeadm init` on `cnpe-worker1` and `cnpe-worker2`.

---

## 3. Install the Complete CNPE Platform Stack

Execute these Helm commands on `cnpe-cp1` to prepare your lab for all 5 exam domains:

```bash
# 1. Install Argo CD & Argo Rollouts
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.12.3/manifests/install.yaml

kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

# 2. Install Crossplane
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
helm install crossplane crossplane-stable/crossplane --namespace crossplane-system --create-namespace

# 3. Install Kyverno (Policy as Code)
helm repo add kyverno https://kyverno.github.io/kyverno/
helm repo update
helm install kyverno kyverno/kyverno -n kyverno --create-namespace

# 4. Install External Secrets Operator (ESO)
helm repo add external-secrets https://charts.external-secrets.io
helm repo update
helm install external-secrets external-secrets/external-secrets -n external-secrets --create-namespace

# 5. Install KEDA (Event-Driven Autoscaling)
helm repo add kedacore https://kedacore.github.io/charts
helm repo update
helm install keda kedacore/keda --namespace keda --create-namespace
```

---

## 4. One-Command Teardown
When you finish your practice session:
```bash
doctl compute droplet delete --tag-name cnpe-lab -f
```
