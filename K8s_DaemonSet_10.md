# DaemonSet

## 1. What Is a DaemonSet?

- A **DaemonSet** makes sure that one Pod runs on every Node in the cluster or only on selected Nodes.  
- It’s perfect for background services that need to be available everywhere.

---

## Key Features of DaemonSets

### One Pod per Node
- A DaemonSet makes sure exactly **one Pod** is scheduled on each Node.

### Automatic Pod Management
- When a new Node is added, the DaemonSet automatically creates a Pod on it.  
- When a Node is removed, the Pod running on that Node is also terminated.

### Selective Node Deployment
A DaemonSet can be configured to run only on specific Nodes using:  
- Node selectors  
- Affinity rules  
- Taints and tolerations  

---

## 2. Common Use Cases

### Monitoring
- Run tools like **Prometheus Node Exporter** or **Datadog Agent** on every Node.  
- DaemonSet ensures one monitoring Pod runs on each Node, even if new Nodes are added.

### Logging
- Run log collectors like **Fluentd, Logstash, or Filebeat** on every Node.  
- They collect logs from containers and the system, then send them to a central place.

### Networking Services
- Run networking tools like **kube-proxy, Calico, Flannel, or VPN clients** on all Nodes.  
- DaemonSet ensures every Node has these tools for **network connections and rules**.

---

## 3. Understanding DaemonSet Pods in the Cluster

When you run:

```bash
kubectl get all -A -o wide
````

* You will see many system Pods running inside the cluster, especially in the **kube-system** namespace.
* Some Pods run only on the **Master Node**.
* Other Pods run on **all Nodes** (Master + Workers).

Each Pod is always linked to a specific Node:

* Master Node → `ip-172-31-34-191`
* Worker 1 → `ip-172-31-5-194`
* Worker 2 → `ip-172-31-3-215`

### 1. Pods Running in the Cluster

Examples of Pods:

* `coredns`
* `etcd`
* `kube-apiserver`
* `kube-controller-manager`
* `kube-scheduler`
* `kube-proxy`
* `weave-net` (CNI plugin)

### 2. Control Plane Pods (Only on Master Node)

Run only on Master Node (`ip-172-31-34-191`):

* `etcd`
* `kube-apiserver`
* `kube-controller-manager`
* `kube-scheduler`

### 3. DaemonSet Pods (On All Nodes)

Managed by DaemonSets → one Pod per Node:

* `kube-proxy`
* `weave-net`

So you see them on **every Node**:

* kube-proxy → 3 Pods (1 on Master + 2 Workers)
* weave-net → 3 Pods (1 on Master + 2 Workers)

They run on:

* Master → `ip-172-31-34-191`
* Worker 1 → `ip-172-31-5-194`
* Worker 2 → `ip-172-31-3-215`

### 4. Key Point

* Some Pods run only on **Master Node** → e.g., `etcd`, `kube-apiserver`.
* Some Pods run on **all Nodes** → e.g., `kube-proxy`, `weave-net`.

👉 Controllers difference:

* **ReplicaSet/Deployment** = runs multiple Pods based on replicas.
* **DaemonSet** = runs one Pod on every Node for system services (networking, logging, monitoring).

---

## 4. Basic DaemonSet

### Step 1: Create a DaemonSet

This runs an **nginx Pod** on every Worker Node.
You don’t need to set replicas → one Pod per Node automatically.

Example `ds.yaml`:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

Apply:

```bash
kubectl apply -f ds.yaml
```

---

### Step 2: Check Cluster Nodes

```bash
kubectl get nodes
```

Output:

* Master → `ip-172-31-34-191`
* Worker 1 → `ip-172-31-5-194`
* Worker 2 → `ip-172-31-3-215`

---

### Step 3: Verify DaemonSet Pods

```bash
kubectl get all -o wide
```

* Shows all resources with Pod IPs, Node names, images.
* You will see **2 nginx Pods** → one on each Worker Node.
* No Pod on Master (`ip-172-31-34-191`).

---

### Step 4: Check Taints on Nodes

```bash
kubectl describe node ip-172-31-34-191   # Master
kubectl describe node ip-172-31-5-194   # Worker
```

* Worker Nodes → **No taints** → Pods scheduled normally.
* Master Node → Has a **NoSchedule taint** → DaemonSet Pods not placed there.

---

## 5. Deploy DaemonSet on Master Node Also

### Step 1: Delete Old DaemonSet

```bash
kubectl delete -f ds.yaml
```

### Step 2: Add Tolerations in YAML

Update `ds.yaml` with tolerations:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      tolerations:
      - key: "node-role.kubernetes.io/control-plane"
        operator: "Exists"
        effect: "NoSchedule"
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

### Step 3: Apply Updated DaemonSet

```bash
kubectl apply -f ds.yaml
```

### Step 4: Verify Pods

```bash
kubectl get pods -o wide
```

Now you should see **3 nginx Pods** → one on Master + one on each Worker Node.

---

## 6. Key Takeaways

* **DaemonSet = One Pod per Node** → Kubernetes ensures this.
* By default → Pods run only on **Worker Nodes**.
* To include **Master Nodes** → add toleration for `NoSchedule` taint.
* No replicas needed → Pod count always equals **Node count** (unlike Deployments).
