# 🚀 Kubernetes — Introduction & Features

* Kubernetes (K8s) is an open-source container orchestration platform/Engine/tool.
* It automates the deployment, scaling, and management of containerized applications.

---

## 🧭 Container Orchestration Engine

* Container orchestration means managing containers automatically — like when to start, stop, scale, and 
  handle traffic between them.

* Kubernetes manages clusters of containers, ensuring the application runs consistently across different environments.

### ⚙️ Key Responsibilities

* Container deployment : Runs containers automatically across available servers.

* Scaling & descaling : Increases containers when traffic is high and decreases them when traffic is low.

* Load balancing of containers: Distributes the incoming traffic equally to all containers, so no single one is overloaded.

---

### Kubernetes is not a replacement for Docker

* Docker helps to create containers.

* Kubernetes helps to manage those containers across multiple servers.

* Kubernetes does not replace Docker, it replaces Docker Swarm (another orchestration tool).

* So, Docker and Kubernetes work together, not against each other.

---

## 📜 History

- Developed by Google  
- Written in Go/Golang  
- Donated to CNCF (Cloud Native Computing Foundation) in 2014  
- Kubernetes v1.0 was released on July 21, 2015  
- Current stable release: **v1.32**

---

# ✨ Kubernetes Key Features

## Auto‑Scheduling

* Kubernetes provides an advanced scheduler to launch containers on cluster nodes.

* The Kubernetes Scheduler automatically assigns Pods to available nodes.

* It checks resources (CPU, memory), node conditions, and policies/constraints.

## Self-Healing Capabilities

* Kubernetes provides rescheduling, replacing, and restarting the containers that are dead.

## Automated Rollouts and Rollbacks

* Allows changes in app configuration or container image to be rolled out gradually
  
* Supports rollback in case of failure

## Horizontal Scaling 

* Kubernetes can automatically scale your application up or down based on metrics like CPU usage, memory, or custom thresholds.

## Service Discovery & Load Balancing:

* In Kubernetes, you don’t have to manually assign IPs to communication between Pods/containers.

* Kubernetes automatically assigns an IP address to each container.

* Exposes containers using DNS names or IPs and distributes traffic.

* Distributes the incoming traffic equally to all containers, so no single one is overloaded.

## Storage Orchestration

* Kubernetes supports mounting local storage or cloud provider storage (e.g., AWS EBS, GCP Persistent Disks)

* Supports NFS, iSCSI, etc.
---

#  Kubernetes Architecture Overview

<img width="1402" height="882" alt="image" src="https://github.com/user-attachments/assets/4b67947d-7fa8-48f0-83a3-1b891e0bcbb1" />

* Kubernetes has two main components:

  1. Control Plane (Master)

  2. Nodes (Workers/Minions/Slaves)

A group of nodes (Control Plane + Workers) together form a Kubernetes Cluster.

## 🛠️ 1. Control Plane (Master)

* This is the brain of the cluster, handling decisions and maintaining the overall cluster state.

# API Server: 

* The API Server is the front-end of the Kubernetes control plane and the heart of the cluster.

* It acts as the central hub for communication.

* It accepts REST API requests (from kubectl, UIs, or other tools), validates them.

* It retrieves or stores data in etcd, and manages authentication and authorization to control access.

# etcd:

*  etcd is a Key-value store.

* etcd holds the current state of all Kubernetes resources, including Pods, Secrets (for sensitive information), 

  ConfigMaps (for non-sensitive configuration data), Deployments, Services, and more.

* When a request is made through the API Server, etcd provides the necessary information, 

  such as the list of Pods and their statuses.

# Scheduler:

* The scheduler identifies the unscheduled pod and assigns it to a node.  

* It selects the best node for each pod based on available CPU, memory, and other criteria.

# kube-controller-manager

* The Controller Manager monitors the state of all nodes and resources in the cluster.

* It ensure everything is working properly and matches the desired state.
 
* If something goes wrong, like a Pod crash, it automatically takes action to fix it by restarting or rescheduling the Pod.

# cloud-controller-manager:

* Integrates with cloud provider APIs (AWS, Azure, GCP) to manage cloud-specific resources (load balancers, storage, etc.).

## 2. Node Components (Workers of the cluster)

# kubelet: 

* The Kubelet is an agent on each worker node that makes sure Pods are running properly.

* The Kubelet running on the node detects the scheduled pod and starts the container runtime.

* If a Pod on the node crashes or has a problem, the Kubelet informs the API Server.

* The API Server passes this info to the Controller Manager, which then fixes it (like restarting or rescheduling the Pod).

👉 In short: Kubelet → API Server → Controller Manager → Fix the Pod ✅

# kube-proxy: 
 
* Maintains network rules and enables service-to-pod communication inside and outside the cluster.

# Pods: 

* Smallest deployable units in Kubernetes; contain one or more containers
 
# CRI (Container Runtime Interface): 

* Interface between kubelet and container runtime (e.g., containerd, Docker)

* The Container Runtime is the software on each worker node that actually runs the containers.

---
