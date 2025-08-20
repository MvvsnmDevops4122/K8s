# 🚀 Kubernetes — Introduction & Features

**Kubernetes (often abbreviated as K8s)** is an open-source container orchestration platform/Engine that automates  
the deployment, scaling, and management of containerized applications.

## Container Orchestration Engine
Kubernetes manages clusters of containers, ensuring the application runs consistently across different environments.

### ⚙️ Key Responsibilities
- Container deployment  
- Scaling & descaling  
- Load balancing of containers  

## 📜 History
- Developed by Google  
- Written in Go/Golang  
- Donated to CNCF (Cloud Native Computing Foundation) in 2014  
- Kubernetes v1.0 was released on July 21, 2015  
- Current stable release: v1.31  

# ✨ Kubernetes Key Features

## 1. Automated Scheduling
Kubernetes provides an advanced scheduler to launch containers on cluster nodes. It performs resource optimization.

## 2. Self-Healing Capabilities
It provides rescheduling, replacing, and restarting the containers that are dead.

## 3. Automated Rollouts and Rollbacks
It supports rollouts and rollbacks for the desired state of the containerized application.

## 4. Horizontal Scaling
Kubernetes can scale up and scale down the application as per the requirements.

## 5. Service Discovery & Load Balancing
Exposes containers using DNS names or IPs and distributes traffic.

## 6. Support for Multiple Clouds and Hybrid Clouds
Kubernetes can be deployed on different cloud platforms and run containerized applications across multiple clouds.

## 7. Storage Orchestration
- Supports mounting local storage or cloud provider storage (e.g., AWS EBS, GCP Persistent Disks)  
- Supports NFS, iSCSI, etc.

## 8. Community Support
Kubernetes has a large and active community with frequent updates, bug fixes, and new features being added.

# 🏗️ Kubernetes Architecture Overview

## Two Main Components:

### 🛠️ 1. Control Plane (Master)

#### kube-apiserver
- The front-end for the Kubernetes control plane. Accepts REST API commands (from kubectl, UIs, or other tools) and validates requests.

#### etcd
- Key-value store that holds all cluster data

#### kube-scheduler
- Assigns unscheduled Pods to Nodes based on resource requirements, constraints, and policies.

#### kube-controller-manager
- Runs core controllers (Node, Replication, Endpoint, Namespace, ServiceAccount, etc.); ensures desired state matches actual state.

#### cloud-controller-manager
- Integrates with cloud provider APIs (AWS, Azure, GCP) to manage cloud-specific resources (load balancers, storage, etc.).

### 2. Node Components (Workers of the cluster)

#### kubelet
- Agent running on each node; communicates with the control plane

#### kube-proxy
- Maintains network rules and enables service-to-pod communication inside and outside the cluster.

#### Pods
- Smallest deployable units in Kubernetes; contain one or more containers

#### CRI (Container Runtime Interface)
- Interface between kubelet and container runtime (e.g., containerd, Docker)

