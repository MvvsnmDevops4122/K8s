# Types of Kubernetes clusters:

1. Self Managed Clusters

2. Cloud Provider Clusters

# 1.Self Managed Cluster:

* A group of servers (physical or virtual) that your team or organization manages directly, instead of using

  an outside service provider or cloud vendor.

## Minikube: \[single node k8s cluster]

* Runs a single-node Kubernetes cluster on your local machine, used mainly for development and testing.

* Kubernetes takes care of pod failures, but node failure recovery is manual—you need to fix node issues yourself.

## Kubeadm: \[multi node k8s cluster]

* Used to create multi-node clusters (e.g., 1 master, 2 worker nodes) for both testing and production.

* Like Minikube, it automatically handles pod failures, but you need to manually fix any failed nodes.

## Note:

* This is the main drawback of self-managed Kubernetes: if a pod fails, Kubernetes will automatically recover,

  but if a node fails, you must fix it manually.

* Many teams choose cloud providers, where node failures are handled automatically by the cloud platform.

# 2. Cloud Provider Clusters
 
EKS (AWS Elastic Kubernetes Service)

AKS (Azure Kubernetes Service)

GKE (Google Kubernetes Engine)

* AWS EKS, Azure AKS, Google GKE

* Fully-managed Kubernetes services provided by cloud vendors.

* Kubernetes automatically restores failed pods.

* Node (virtual machine) failures are handled by the cloud provider—nodes are recreated or replaced automatically without your

  manual intervention.

## Summary:

Application failure: If Pod down inside node is called application failure.

Infra failure: If Entire node down is called Infra failure.

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

### 🔹 Common Setup for All Nodes (Master & Workers) ###

# Step 1: Switch to Root User on All Nodes

sudo -i (or) sudo su -

# Step 2:  Update system

sudo apt update && sudo apt upgrade -y

# Step 3: Disable Swap on All Nodes(Kubernetes does not allow swap)

🔗 Ref:[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) (Taken from here)

swapoff -a
sed -i '/ swap / s/^$.*$\$/#\1/g' /etc/fstab

# Step 4: Enable required kernel modules

🔗 Ref:[https://kubernetes.io/docs/setup/production-environment/container-runtimes/](https://kubernetes.io/docs/setup/production-environment/container-runtimes/) (Taken from here)

cat <\<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br\_netfilter
EOF

modprobe overlay
modprobe br\_netfilter

# Step 5: Set system networking params

cat <\<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip\_forward                 = 1
EOF

sysctl --system

### Install Container Runtime (containerd) ###

# Step 6: Update packages and Install dependencies

🔗 Ref:[https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)

🔗 Ref:[https://github.com/containerd/containerd/blob/main/docs/getting-started.md](https://github.com/containerd/containerd/blob/main/docs/getting-started.md)

🔗 Ref:[https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

apt-get update -y && apt-get install -y ca-certificates curl gnupg lsb-release

# Step 7: Add Docker GPG key

mkdir -p /etc/apt/keyrings
curl -fsSL [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) | gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Step 8: Add Docker repository

echo "deb \[arch=\$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] [https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) \$(lsb\_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Step 9: Install containerd

apt-get update -y
apt-get install -y containerd.io

# Step 10: Configure containerd

containerd config default > /etc/containerd/config.toml

# Step 11: Try to update the configuration cgroup as systemd for containerd

sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

# Step 12: Restart and enable containerd

systemctl restart containerd
systemctl enable containerd
systemctl status containerd

### Install kubelet, kubeadm, kubectl ###

# Step 13: Update packages and install dependencies

apt-get update
apt-get install -y apt-transport-https ca-certificates curl

# Step 14: Add Kubernetes signing key

curl -fsSL [https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key](https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key) | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Step 15: Add Kubernetes repo

echo 'deb \[signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] [https://pkgs.k8s.io/core:/stable:/v1.31/deb/](https://pkgs.k8s.io/core:/stable:/v1.31/deb/) /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Step 16: Update package and install kubelet, kubeadm, kubectl

🔗 Ref:[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports)

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

# Step 17: Prevent them from auto-updating

sudo apt-mark hold kubelet kubeadm kubectl

# Step 18: Enable and start kubelet service

systemctl daemon-reload
systemctl start kubelet
systemctl enable kubelet.service

### Shell Scripting for Common Setup for All Nodes (Master & Workers) ###

File name: k8s-common-setup.sh (chmod +x k8s-common-setup.sh)

## Copy paste:

\#!/bin/bash

set -e

# Step 2: Update system

sudo apt update && sudo apt upgrade -y

# Step 3: Disable Swap on All Nodes (Kubernetes does not allow swap)

swapoff -a
sed -i '/ swap / s/^$.*$\$/#\1/g' /etc/fstab

# Step 4: Enable required kernel modules

cat <\<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br\_netfilter
EOF

modprobe overlay
modprobe br\_netfilter

# Step 5: Set system networking params

cat <\<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip\_forward                 = 1
EOF

sysctl --system

### Install Container Runtime (containerd)###

# Step 6: Update packages and Install dependencies

apt-get update -y && apt-get install -y ca-certificates curl gnupg lsb-release

# Step 7: Add Docker GPG key

mkdir -p /etc/apt/keyrings
curl -fsSL [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) | gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Step 8: Add Docker repository

echo "deb \[arch=\$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] [https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) \$(lsb\_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Step 9: Install containerd

apt-get update -y
apt-get install -y containerd.io

# Step 10: Configure containerd

containerd config default > /etc/containerd/config.toml

# Step 11: Update the configuration cgroup as systemd for containerd

sed -i 's/SystemdCgroup \\= false/SystemdCgroup \\= true/g' /etc/containerd/config.toml

# Step 12: Restart and enable containerd

systemctl restart containerd
systemctl enable containerd
systemctl status containerd

### Install kubelet, kubeadm, kubectl ###

# Step 13: Update packages and install dependencies

apt-get update
apt-get install -y apt-transport-https ca-certificates curl

# Step 14: Add Kubernetes signing key

curl -fsSL [https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key](https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key) | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Step 15: Add Kubernetes repo

echo 'deb \[signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] [https://pkgs.k8s.io/core:/stable:/v1.31/deb/](https://pkgs.k8s.io/core:/stable:/v1.31/deb/) /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Step 16: Update package and install kubelet, kubeadm, kubectl

apt-get update
apt-get install -y kubelet kubeadm kubectl

# Step 17: Prevent them from auto-updating

apt-mark hold kubelet kubeadm kubectl

# Step 18: Enable and start kubelet service

systemctl daemon-reload
systemctl start kubelet
systemctl enable kubelet.service

echo "✅ Common Kubernetes setup completed successfully on this node!"


### 🔹 Master Node Setup ###

### [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

# Step 1: Initialize Kubernetes Master

---

kubeadm init

IF Error
\#sudo kubeadm init --cri-socket /run/containerd/containerd.sock

# Step 2: Configure kubeconfig for kubectl

---

mkdir -p \$HOME/.kube
cp -i /etc/kubernetes/admin.conf \$HOME/.kube/config
chown \$(id -u):\$(id -g) \$HOME/.kube/config

# Step 3: Verify cluster status

---

kubectl get nodes
kubectl get pods -n kube-system

# Step 4: Install CNI plugin (Choose one: weave or calico)

---

# Weave Net (Recommended - simple)

kubectl apply -f [https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml](https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml)

# OR Calico

kubectl apply -f [https://docs.projectcalico.org/manifests/calico.yaml](https://docs.projectcalico.org/manifests/calico.yaml)

# Step 5: Verify network and pods

---

kubectl get nodes
kubectl get pods --all-namespaces

# Step 6: Generate join command for worker nodes

---

kubeadm token create --print-join-command

### Kubernetes Master Node Setup Script ###

#!/bin/bash

echo "🚀 Starting Kubernetes Master Node setup..."

# Step 1: Initialize Kubernetes Master

echo "🔹 Initializing Kubernetes Master..."
kubeadm init --cri-socket /run/containerd/containerd.sock

# Step 2: Configure kubeconfig for kubectl

echo "🔹 Setting up kubeconfig..."
mkdir -p \$HOME/.kube
cp -i /etc/kubernetes/admin.conf \$HOME/.kube/config
chown \$(id -u):\$(id -g) \$HOME/.kube/config

# Step 3: Verify cluster status

echo "🔹 Verifying cluster status..."
kubectl get nodes
kubectl get pods -n kube-system

# Step 4: Install CNI plugin (Weave Net)

echo "🔹 Installing Weave Net CNI..."
kubectl apply -f "[https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml](https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml)"

# Step 5: Verify nodes and podsecho "🔹 Checking nodes and pods..."

kubectl get nodes
kubectl get pods --all-namespaces

# Step 6: Generate join command for worker nodes

echo "🔹 Generating join command for worker nodes..."
kubeadm token create --print-join-command

echo "✅ Master setup completed successfully!"
echo "➡️ Use the join command stored in /root/join-worker.sh on worker nodes."

👉 Save it: master-setup.sh

👉 Make executable: chmod +x master-setup.sh

👉 Run as root: ./master-setup.sh

### 🔹 Worker Node Setup ###

Run the join command from the master on each worker:

kubeadm join 172.31.39.61:6443 --token fum1dp.ifem6w2f3f54fv2m&#x20;
\--discovery-token-ca-cert-hash sha256\:e58fc3ea9fd12f0edfaa3541ba82e3673957a4d3f1c7f2d0ae094a15d53e46e1


###  🔹 Validate the Cluster (from Master) ###

kubectl get nodes   # should show master + workers as "Ready"

kubectl get pods --all-namespaces

## 🔹 Deploy a Sample App ##

# Create test namespace

kubectl create ns test

# Deploy nginx pod in "test" namespace

kubectl run nginx-demo --image=nginx --port=80 -n test

# Verify

kubectl get pods -n test

# 📘 Kubernetes Essentials

###  What is a Cluster?

A Kubernetes cluster is a set of machines (nodes) used to run containerized applications.

### What is a Node?

A node is a single server in the cluster (master or worker) that runs pods.

### Namespace Management

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
