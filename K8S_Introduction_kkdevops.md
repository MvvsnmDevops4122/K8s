# 🚀 Kubernetes — Introduction & Features

**Kubernetes (often abbreviated as K8s)** is an open-source container orchestration platform/engine  
that automates the deployment, scaling, and management of containerized applications.

---

## 🧭 Container Orchestration Engine

Kubernetes manages clusters of containers, ensuring the application runs consistently across different environments.

### ⚙️ Key Responsibilities

- Container deployment  
- Scaling & descaling  
- Load balancing of containers  

---

## 📜 History

- Developed by Google  
- Written in Go/Golang  
- Donated to CNCF (Cloud Native Computing Foundation) in 2014  
- Kubernetes v1.0 was released on July 21, 2015  
- Current stable release: **v1.31**

---

# ✨ Kubernetes Key Features

## 1. Automated Scheduling

Kubernetes provides an advanced scheduler to launch containers on cluster nodes. It performs resource optimization.

## 2. Self-Healing Capabilities

It provides rescheduling, replacing, and restarting of containers that are dead or unresponsive.

## 3. Automated Rollouts and Rollbacks

Supports rollouts and rollbacks to maintain the desired state of containerized applications.

## 4. Horizontal Scaling

Kubernetes can scale applications up or down based on demand.

## 5. Service Discovery & Load Balancing

Exposes containers using DNS names or IPs and distributes traffic across them.

## 6. Multi-Cloud & Hybrid Cloud Support

Can be deployed across various cloud platforms and supports hybrid cloud environments.

## 7. Storage Orchestration

- Supports mounting local or cloud provider storage (e.g., AWS EBS, GCP Persistent Disks)  
- Compatible with NFS, iSCSI, and more

## 8. Community Support

Large and active community with frequent updates, bug fixes, and feature enhancements.

---

# 🏗️ Kubernetes Architecture Overview

## Two Main Components

---

## 🛠️ 1. Control Plane (Master)

### `kube-apiserver`

- Front-end for the Kubernetes control plane  
- Accepts REST API commands (from `kubectl`, UIs, or other tools) and validates requests

### `etcd`

- Key-value store that holds all cluster data

### `kube-scheduler`

- Assigns unscheduled Pods to Nodes based on resource requirements, constraints, and policies

### `kube-controller-manager`

- Runs core controllers (Node, Replication, Endpoint, Namespace, ServiceAccount, etc.)  
- Ensures desired state matches actual state

### `cloud-controller-manager`

- Integrates with cloud provider APIs (AWS, Azure, GCP)  
- Manages cloud-specific resources like load balancers and storage

---

## ⚙️ 2. Node Components (Workers of the Cluster)

### `kubelet`

- Agent running on each node  
- Communicates with the control plane

### `kube-proxy`

- Maintains network rules  
- Enables service-to-pod communication inside and outside the cluster

### Pods

- Smallest deployable units in Kubernetes  
- Contain one or more containers

### CRI (Container Runtime Interface)

- Interface between `kubelet` and container runtime (e.g., `containerd`, Docker)
