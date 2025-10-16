### 🧱 Static Pods in Kubernetes

---

## 1. What Is a Static Pod in Kubernetes?

* A Static Pod is a special type of Pod managed directly by the kubelet on a specific node  
  not by the Kubernetes API Server or the control plane.

---

## 2. Key Characteristics of Static Pods

### 1. Node-Specific
* Static Pods are tied to a specific node and cannot be scheduled or moved to other nodes

### 2. Managed by Kubelet
* The kubelet watches a directory (commonly `/etc/kubernetes/manifests/`) and creates pods from the manifest files found there.
  
### 3. Visible but Not Controllable
* Static Pods appear when you run `kubectl get pods`, but you cannot update, scale, or delete them from the master.

### 4. Self-Healing
* If a Static Pod crashes, kubelet will restart it automatically.

### 5. No Controllers
* They are not managed by higher-level controllers like Deployments, ReplicaSets, or StatefulSets.

---

## 3. How Static Pods Work

* You place a pod manifest file (YAML or JSON) in the kubelet’s watched directory: `/etc/kubernetes/manifests/`
* The kubelet detects the file and starts the pod on that node.
* If the file is updated, the kubelet will restart the pod with the new configuration.
* If the file is deleted, the kubelet will terminate the pod.

---

## 4. How Static Pods Work in Kubernetes Step by Step

* Static Pods are managed directly by the kubelet on a node and are not controlled by the Kubernetes API server.  
  Here’s how to create, observe, and delete a Static Pod:

### Step 1: SSH into the Node

```bash
ssh root@<node-ip>
````

### Step 2: Go to the Manifest Directory

```bash
cd /etc/kubernetes/manifests
```

### Step 3: Create a YAML File for the Pod

Example: **nginx-static.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-static
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

### Step 4: Verify from Master

```bash
kubectl get pods -o wide
```

👉 You will see the static pod running on the node.

---

## 5. Steps to Delete a Static Pod

### Step 1: SSH into the Node

```bash
ssh root@<node-ip>
```

### Step 2: Remove the Pod Manifest File

```bash
rm /etc/kubernetes/manifests/nginx-static.yaml
```

### Step 3: Confirm from Master

```bash
kubectl get pods
```

* The pod will no longer appear.
* Kubelet detects the file is gone → it will stop and delete the pod from that node.

---

## 6. Advantages of Static Pods

* Self-healing → If a static pod crashes or is deleted, the Kubelet will automatically restart it.
* Node-specific → A Static Pod belongs to one specific node.

---

## 7. Limitations of Static Pod

* Not scalable (you must copy files to each node manually).
* No features like Deployments/ReplicaSets.
* To update or delete → you must directly edit/remove the file on the node.
* No rolling updates.
* Harder to manage (requires SSH for edits/removals).
* Not suited for regular apps.

```
