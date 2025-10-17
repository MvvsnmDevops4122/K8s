# 📘 Kubernetes Pods

----

## 1. What is a Pod?

* A Pod is the smallest deployable unit (or building block) in Kubernetes.

* A Pod can contain one or more containers, which share the same network IP, storage, and configuration.

* Pods are always created inside a namespace for logical separation.

* Pods are the actual running instances of your application.

* Each Pod is assigned a unique IP address within the cluster.

* But some system pods (like DNS, networking, API server helpers, etc.) may run on the master node 
  because Kubernetes needs these pods for management and cluster functionality. These are part of the control plane or system namespaces (like `kube-system`). 

Note: Application Pods usually run on worker nodes, not on the master (control plane) node.

----

## 2. Where Pods Fit in the Kubernetes World

1. **Master Node / Control Plane**

   * The brain of Kubernetes.

   * Contains the scheduler, which decides which worker node should run your Pod.

2. **Worker Nodes**

   * The machines (VMs or physical servers) that actually run the Pods assigned by the master.

3. **Kubelet**

   * Kubelet agent that runs on every worker node.

   * It communicates with the master, pulls container images, starts containers, and ensures Pods keep running.

4. **Container Runtime**

   * The software (e.g., Docker, containerd) that actually runs the containers inside each Pod.

----

## 3. Pod Characteristics

1. Smallest Unit          : A Pod is the smallest deployable unit (or building block) in Kubernetes.

2. Contains Containers    : A Pod can contain one or more containers, which share the same network IP, storage, and configuration.

3. Unique IP Address      : Each Pod is assigned a unique IP address within the cluster.

4. Shared Network         : Containers inside a Pod communicate with each other using localhost.

5. Shared Storage         : Containers inside a Pod can share the same storage volumes.

6. Ephemeral (Temporary)  : Pods are not permanent. If a Pod dies, Kubernetes creates a new one with a new IP.

7. Namespace Scoped       : Pods are always created inside a namespace for logical separation.

----
## 4. Types of Pods

### Single-Container Pod

✅ This model is the most popular in Kubernetes.

✅ Pod contains only one container.

Example: Running just a web server.

----

**Pod YAML Example**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-first-pod
  namespace: default
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
````

**Commands**

```bash
kubectl apply -f pod.yaml        # Apply the YAML
kubectl get pods                 # Verify Pods
kubectl get pods -n <namespace>
```
----

### Multi-container Pod (Sidecar containers)

✅ A Multi-Container Pod runs two or more containers together in the same Pod.

   All containers inside a Pod share:

      * Network         → They can talk to each other using localhost and share ports.

      * Storage Volumes → They can use the same mounted volumes.

      * Lifecycle       → They start, stop, and restart together.


## Common Use Case: Sidecar Pattern

* The Sidecar Pattern adds a helper container to extend or enhance the functionality of the main container, without modifying it.

* Example: A log-collector container that gathers logs from the main app-container.

----

## Example

Main App Container
------------------

* Runs your primary app (e.g., web server, API).

Sidecar Container
----------------

Handles supporting tasks like:

Logging (collecting logs)

Monitoring (exporting metrics)

Proxying (service mesh, security, etc.)


**Pod YAML Example**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
    - name: app-container
      image: nginx:latest
      ports:
        - containerPort: 80
    - name: log-collector
      image: fluentd:latest
```

**Sidecar Use Case:** log collection, monitoring, proxying.

----

## 5. Pod-to-Pod Communication

Same Node:

* Pods can directly talk to each other via their IP addresses (no extra steps needed).

Different Nodes:

* Kubernetes uses a Cluster Network (e.g., Calico, Flannel, Cilium).

* This ensures Pods can communicate across different worker nodes, even if they are on separate machines.


----

## 6. Pod Storage

By default, containers are ephemeral → data is lost if the Pod restarts or dies.

Solution: Use Volumes to persist data.

Common Volume Types:

1. emptyDir → Temporary storage, erased when the Pod stops.

2. hostPath → Uses a folder from the host machine.

3. Persistent Volume (PV) → Storage that remains even if the Pod is deleted.

📌 Tip: For databases or applications storing important data, always use Persistent Volumes (PV).

----

## 7. Pod Lifecycle

* Make a Pod request to the API server using a local Pod definition file.

* The API server saves the pod information in ETCD

* The scheduler identifies the unscheduled pod and assigns it to a node.

* The Kubelet running on the node detects the scheduled pod and starts the container runtime.

* The entire lifecycle state of the pod is stored in ETCD.

----


## Kubernetes Pod Commands (Copy-Paste)

```bash
kubectl api-resources                             # List all Kubernetes objects

kubectl api-resources --namespaced=true           # List namespace-level objects

kubectl api-resources --namespaced=false          # List cluster-level objects

kubectl get po -n <namespace>                     # Get pods in namespace

kubectl get po -o wide -n <namespace>             # Get pods with details (IP, Node)

watch kubectl get po -n <namespace>               # Watch pods (auto refresh)

kubectl get po -n <namespace> --show-labels       # Get pod with labels

kubectl describe po <podName> -n <namespace>      # Describe a pod

kubectl logs <pod-name> -n <namespace>            # Check logs of a pod 

kubectl exec -it <pod-name> -n <namespace> -- /bin/sh   # Exec into a pod (debug inside)  

kubectl get events -n <namespace>                       # Check events in namespace (troubleshoot)

```
----

## Pod Examples

----

### Example 1: Nginx Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginxpod
  namespace: test
spec:
  containers:
  - name: nginx-cont
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

```bash
kubectl apply -f nginx.yaml --dry-run
kubectl apply -f nginx.yaml --dry-run=client
kubectl apply -f nginx.yaml --dry-run=server
kubectl apply -f nginx.yaml
```
----

### Example 2: Nginx Pod with ImagePullBackOff

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginxpod1
  namespace: test
spec:
  containers:
  - name: nginx-cont
    image: kkdevopsnginx:1.14.2
    ports:
    - containerPort: 80
```

```bash
kubectl apply -f nginx1.yaml
kubectl describe pod nginxpod1 -n test
```
----

### Example 3: Java WebApp Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: javawebapp
  labels:
    app: javawebapp
  namespace: test-ns
spec:
  containers:
  - name: java-container
    image: satyamolleti4599/maven-web-app:1.0.0
    ports:
    - containerPort: 8080
```

```bash
kubectl describe po javawebapp -n test
```
----

### Example 4: Mongo Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mongo-pod
  namespace: db-ns  
  labels:
    app: mongo
  annotations:
    description: "A MongoD"
spec:
  containers:
  - name: mongo-container
    image: mongo:latest
    ports:
    - containerPort: 27017
```
----

### Namespace creation:

```bash
kubectl create ns my-namespace
kubectl apply -f dbns.yaml
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: db-ns
  labels:
    purpose: database-resources
```
----

### Example 5: Tomcat Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tomcatpod
  namespace: test
spec:
  containers:
  - name: tomcatcontainer
    image: tomcat:9.0
    ports:
    - containerPort: 8080
```

```bash
kubectl apply -f tomcat-pod.yaml -n test
```
----
