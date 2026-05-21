# Types of Kubernetes Clusters

---

Kubernetes clusters are mainly divided into two types:

1. Self-Managed Clusters
2. Cloud-Managed Clusters

---

# 1. Self-Managed Clusters

- In self-managed clusters, the organization manages the Kubernetes infrastructure and nodes manually.

- The team is responsible for:
  - Installation
  - Configuration
  - Upgrades
  - Maintenance
  - Troubleshooting

## Minikube

- Single-node Kubernetes cluster.

- Mainly used for:
  - Learning
  - Development
  - Testing

- Runs on a local machine.

---

## kubeadm

- Used to create multi-node Kubernetes clusters.

Example:
- 1 Control Plane Node
- Multiple Worker Nodes

- Used for:
  - Testing
  - Production environments

---

## Drawback of Self-Managed Clusters

- Kubernetes automatically handles Pod failures.

- But if a node fails, it must be fixed manually by the administrator.

---

# 2. Cloud-Managed Clusters

Cloud providers offer fully managed Kubernetes services.

Examples:
- Amazon EKS
- Azure Kubernetes Service
- Google Kubernetes Engine

---

# Features of Cloud-Managed Clusters

- Kubernetes automatically handles Pod failures.

- Cloud providers automatically manage:
  - Node failures
  - Infrastructure maintenance
  - Scaling
  - High availability

- If a node fails, the cloud provider automatically recreates or replaces it.
---

### Lab Setup: Kubeadm on AWS EC2 (Ubuntu 22.04) ###


## Infrastructure Requirements:

1 Master Node: t2.medium (2 vCPU, 4 GB RAM)

2 Worker Nodes: t2.micro (1 vCPU, 1 GB RAM)

## Create a Security Group in AWS

* Open AWS Console → Go to VPC service → Select "Security Groups".

* Click "Create Security Group".

* Name it (e.g., kube-cluster-sg).

* Allow all inbound traffic (temporarily for setup) from 0.0.0.0/0 for all protocols and ports.

* Click "Create security group".

## Launch Master Node EC2 Instance

* Go to EC2 → Launch Instance.

* Name: kube-master.

* AMI: Ubuntu 22.04 LTS (or latest supported).

* Instance Type: t2.medium (2 vCPU, 4 GB RAM).

* Key Pair: Select or create a key pair.

* Network: Default VPC.

* Security Group: Select the one created in Step 1.

* Click "Launch Instance".

## Launch Worker Node EC2 Instances

* Repeat Step 2.

* Name: kube-node-1 and kube-node-2.

* Instance Type: t2.micro(1 vCPU, 1 GB RAM)

* Number of Instances: 2.

* Same key pair and security group as master.

* Click "Launch Instances".

---

### 🔹 Common Setup for All Nodes (Master & Workers) ###

# Step 1: Switch to Root User on All Nodes

sudo -i (or) sudo su -

# Step 2:  Update system

sudo apt update && sudo apt upgrade -y

# Step 3: Disable Swap on All Nodes(Kubernetes does not allow swap)

🔗 Ref:[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) (Taken from here)

```bash
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab 
```

# Step 4: Enable required kernel modules

🔗 Ref:[https://kubernetes.io/docs/setup/production-environment/container-runtimes/](https://kubernetes.io/docs/setup/production-environment/container-runtimes/) (Taken from here)

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter
```

# Step 5: Set system networking params

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sysctl --system
```

### Install Container Runtime (containerd) ###

# Step 6: Update packages and Install dependencies

🔗 Ref:[https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)

🔗 Ref:[https://github.com/containerd/containerd/blob/main/docs/getting-started.md](https://github.com/containerd/containerd/blob/main/docs/getting-started.md)

🔗 Ref:[https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

```bash
apt-get update -y && apt-get install -y ca-certificates curl gnupg lsb-release
```

# Step 7: Add Docker GPG key

```bash
mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
# Step 8: Add Docker repository

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list


```
# Step 9: Install containerd

```bash
apt-get update -y
apt-get install -y containerd.io
```

# Step 10: Configure containerd

```bash
containerd config default > /etc/containerd/config.toml
```

# Step 11: Try to update the configuration cgroup as systemd for containerd

```bash
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

```
# Step 12: Restart and enable containerd

```bash

systemctl restart containerd
systemctl enable containerd
systemctl status containerd

```
### Install kubelet, kubeadm, kubectl ###

# Step 13: Update packages and install dependencies

```bash

apt-get update
apt-get install -y apt-transport-https ca-certificates curl
```
# Step 14: Add Kubernetes signing key

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
# Step 15: Add Kubernetes repo

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
# Step 16: Update package and install kubelet, kubeadm, kubectl

🔗 Ref:[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports)

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

```
# Step 17: Prevent them from auto-updating

```bash

sudo apt-mark hold kubelet kubeadm kubectl

```
# Step 18: Enable and start kubelet service

```bash
systemctl daemon-reload
systemctl start kubelet
systemctl enable kubelet.service
```
---

### Kubernetes Common Setup Script

---

```bash id="masterscript1"

#!/bin/bash

set -e

echo "========== Updating System =========="

apt update -y
apt upgrade -y

echo "========== Disable Swap =========="

swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

echo "========== Enable Kernel Modules =========="

cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

echo "========== Configure Sysctl =========="

cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sysctl --system

echo "========== Install Required Packages =========="

apt-get install -y \
curl \
gnupg \
ca-certificates \
apt-transport-https \
software-properties-common \
conntrack

echo "========== Install Containerd =========="

mkdir -p /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | \
tee /etc/apt/sources.list.d/docker.list > /dev/null

apt update -y

apt install -y containerd.io

echo "========== Configure Containerd =========="

mkdir -p /etc/containerd

containerd config default | tee /etc/containerd/config.toml > /dev/null

sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

systemctl restart containerd
systemctl enable containerd

echo "========== Install Kubernetes Packages =========="

mkdir -p /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | \
gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /" | \
tee /etc/apt/sources.list.d/kubernetes.list

apt update -y

apt install -y kubelet kubeadm kubectl

apt-mark hold kubelet kubeadm kubectl

echo "========== Enable Kubelet =========="

systemctl daemon-reload
systemctl enable kubelet
systemctl restart kubelet

echo "========== Verify Services =========="

systemctl status containerd --no-pager
systemctl status kubelet --no-pager

echo "========== Kubernetes Common Setup Completed Successfully =========="

```.

---
# Master Node Setup Notes
---

# Step 1: Initialize Kubernetes Master

Use this recommended command:

```bash id="master1"
kubeadm init --pod-network-cidr=192.168.0.0/16
```

---

# If You Get CRI Socket Error

Use:

```bash id="master2"
kubeadm init --cri-socket unix:///run/containerd/containerd.sock --pod-network-cidr=192.168.0.0/16
```

---

# If conntrack Error Comes

Install:

```bash id="master3"
apt-get update

apt-get install -y conntrack
```

Verify:

```bash id="master4"
which conntrack
```

---

# Step 2: Configure kubectl

```bash id="master5"
mkdir -p $HOME/.kube

cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

chown $(id -u):$(id -g) $HOME/.kube/config
```

---

# Step 3: Verify Cluster

```bash id="master6"
kubectl get nodes

kubectl get pods -n kube-system
```

---

# Step 4: Install CNI Plugin

## IMPORTANT

Do NOT use Weave Net with Kubernetes v1.31.

You faced:

* `weave-net CrashLoopBackOff`
* `Node NotReady`
* `API server connection refused`

So use ONLY Calico.

---

# Install Calico

```bash id="master7"
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.3/manifests/calico.yaml
```

---

# Step 5: Verify Network

```bash id="master8"
kubectl get pods -A
```

Wait until all:

* calico pods
* coredns
* kube-system pods

show:

```text id="master9"
Running
```

---

# Step 6: Verify Node Ready

```bash id="master10"
kubectl get nodes
```

Expected:

```text id="master11"
STATUS = Ready
```

---

# Step 7: Generate Worker Join Command

```bash id="master12"
kubeadm token create --print-join-command
```

Example:

```bash id="master13"
kubeadm join 172.31.17.111:6443 --token TOKEN --discovery-token-ca-cert-hash sha256:HASH
```

---

# Worker Node Important Fix

Before joining worker node:

Install conntrack:

```bash id="master14"
apt-get update

apt-get install -y conntrack
```

Then run join command.

---

# Final Verification

On master node:

```bash id="master15"
kubectl get nodes
```

Expected:

```text id="master16"
master-node   Ready
worker-node   Ready
```


---

Your script is good, but based on the issues you faced, these corrections are required:

* Add pod CIDR
* Use Calico instead of Weave
* Fix escaped `$`
* Add wait/check steps
* Store join command properly
* Add error handling

---

# Updated Stable Kubernetes Master Setup Script

```bash id="masterscript1"
#!/bin/bash

set -e

echo "🚀 Starting Kubernetes Master Node Setup..."

echo "=============================================="
echo "Step 1: Initialize Kubernetes Master"
echo "=============================================="

kubeadm init \
--cri-socket unix:///run/containerd/containerd.sock \
--pod-network-cidr=192.168.0.0/16

echo "=============================================="
echo "Step 2: Configure kubeconfig"
echo "=============================================="

mkdir -p $HOME/.kube

cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

chown $(id -u):$(id -g) $HOME/.kube/config

echo "=============================================="
echo "Step 3: Verify Cluster Status"
echo "=============================================="

kubectl get nodes

kubectl get pods -n kube-system

echo "=============================================="
echo "Step 4: Install Calico Network Plugin"
echo "=============================================="

kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.3/manifests/calico.yaml

echo "=============================================="
echo "Waiting 60 Seconds for Cluster Networking..."
echo "=============================================="

sleep 60

echo "=============================================="
echo "Step 5: Verify Nodes and Pods"
echo "=============================================="

kubectl get nodes

kubectl get pods -A

echo "=============================================="
echo "Step 6: Generate Worker Join Command"
echo "=============================================="

kubeadm token create --print-join-command | tee /root/join-worker.sh

chmod +x /root/join-worker.sh

echo "=============================================="
echo "✅ Kubernetes Master Setup Completed Successfully!"
echo "=============================================="

echo "➡️ Worker Join Command saved in:"
echo "/root/join-worker.sh"
```

---

# Save Script

```bash id="masterscript2"
vi master-setup.sh
```

Paste script.

---

# Give Execute Permission

```bash id="masterscript3"
chmod +x master-setup.sh
```

---

# Run Script

```bash id="masterscript4"
./master-setup.sh
```
---

## 🔹 Deploy a Sample App ##

# Create test namespace
```bash
kubectl create ns test
```
# Deploy nginx pod in "test" namespace
```bash
kubectl run nginx-demo --image=nginx --port=80 -n test
```
# Verify
```bash
kubectl get pods -n test
```
# 📘 Kubernetes Essentials

###  What is a Cluster?

A Kubernetes cluster is a set of machines (nodes) used to run containerized applications.

### What is a Node?

A node is a single server in the cluster (master or worker) that runs pods.

### Namespace Management

```bash

kubectl create ns my-namespace                            # Create namespace

kubectl get ns                                            # List namespaces

kubectl describe ns my-namespace                          # View details

kubectl delete ns my-namespace                            # Delete namespace

kubectl config set-context --current --namespace=test     # Switch context

kubectl get pods -A                                       # List all pods across namespaces

kubectl get pods -n my-namespace                          # Pods in specific namespace

kubectl run nginx-pod --image=nginx --port=80 -n test     # Create pod in namespace

kubectl delete pod nginx-pod -n test                      # Delete pods

kubectl delete pod nginx-pod                              # default namespace

kubectl api-resources                                     # List all resource types

kubectl config set-context --current --namespace=test     # Switch namespace

```

# ✅ Resource Quotas

quota.yaml

apiVersion: v1
kind: ResourceQuota
metadata:
name: my-resource-quota
namespace: test
spec:
hard:
pods: "10"

Apply quota: kubectl apply -f quota.yaml
