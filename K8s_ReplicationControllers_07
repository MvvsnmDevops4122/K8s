# 📘 Kubernetes ReplicationController (RC)

---
## 1. What Is a ReplicationController?

* A ReplicationController (RC) is a Kubernetes object that ensures a fixed number of identical Pods are always running.
* It acts as a controller, continuously monitoring the cluster state.
* It takes corrective actions to maintain the desired number of Pods.

**Key Features:**

1. Self-healing: If a Pod crashes or is deleted, RC creates a new one.
2. No extras allowed: If someone manually creates extra Pods with the same labels, RC deletes them to maintain the correct count.
3. Label-based tracking: RC uses labels, not Pod names, to find and manage Pods.

---

## 2. Desired vs Observed State

* Desired state: How many Pods you want (e.g., 3).
* Observed state: How many Pods are currently running with the correct labels.

**Reconciliation:** RC constantly compares desired vs observed state and adjusts Pods to match.

**Example:**

* If desired = 3, but only 2 running → RC creates 1 more.
* If desired = 3, but 5 running (extra Pods created manually) → RC deletes 2.

---

## 3. How ReplicationController Works

* The RC uses a continuous control loop to make sure the desired number of Pods are always running.

### 1. Watches All Pods

* RC constantly monitors all Pods in the cluster.
* It doesn’t track Pods by name — it uses labels to group and manage them.

### 2. Filters by Label Selector

* RC uses spec.selector to find Pods with matching labels.
* Only Pods that have these labels are considered part of the RC group.

### 3. Counts Matching Pods

* RC checks how many Pods match the selector.
* If the count is less than desired → RC creates new Pods using the spec.template.
* If the count is more than desired → RC deletes the extra Pods to bring the number back to the desired state.

### 4. Adopts Orphan Pods

* If Pods already exist with the same labels but no controller managing them → RC can adopt those Pods.

---

### RC YAML Example

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: javawebapprc
  namespace: test-ns
spec:
  replicas: 2
  selector:
    app: javawebapp
  template:
    metadata:
      labels:
        app: javawebapp
    spec:
      containers:
      - name: javawebappcon
        image: kkducationb2/java-webapp:1.1
        ports:
        - containerPort: 8080
````

## 4. Working with ReplicationControllers

### 1. Create and Inspect

```bash
kubectl apply -f rc.yaml
kubectl get all -n test-ns
kubectl get rc
```

### 2. Watch Self-Healing

```bash
kubectl delete pod javawebapprc-44v9d -n test-ns
# RC notices and creates a new Pod, e.g. javawebapprc-dpkn2
```

### 3. See Who Owns a Pod

```bash
kubectl describe pod javawebapprc-dpkn2 -n test-ns | grep -i 'Controlled By'
```

### 4. Scaling Pods (Best Practice)

**Initial State:**

2 Pods: javawebapprc-b5bfp, javawebapprc-dpkn2

**Scale Up to 3 Pods:**

```bash
kubectl scale rc javawebapprc --replicas=3 -n test-ns
```

**Now running:**

* javawebapprc-b5bfp
* javawebapprc-dpkn2
* javawebapprc-d2kq6 (newly created)

**To Scale Down (e.g., Maintenance):**

```bash
kubectl scale rc javawebapprc --replicas=0 -n test-ns
# All Pods removed, RC remains with 0 replicas
```

**Scale Up Again:**

```bash
kubectl scale rc javawebapprc --replicas=3 -n test-ns
# RC recreates 3 Pods
```

### 5. Deleting RC (What Happens to Pods?)

```bash
kubectl delete rc javawebapprc -n test-ns
# Output: replicationcontroller "javawebapprc" deleted
kubectl get all -n test-ns
# Output: No resources found in test-ns namespace
# RC and its managed Pods were deleted
```

**Keep Pods but Delete RC**

```bash
kubectl delete rc javawebapprc -n test-ns --cascade=orphan
# RC is gone, Pods remain running, but no longer managed
```

---

## ReplicationController Summary

* Replication Controller is a key Kubernetes feature responsible for managing the pod lifecycle.
* It ensures that the specified number of pod replicas are running at any time.
* RC replaces crashed Pods automatically and manages Pods via labels.
* Creating an RC with count of 1 ensures a Pod is always available.

**Example YAML:**

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: javawebapprc
  namespace: test-ns
spec:
  replicas: 2
  selector:
    app: javawebapp
  template:
    metadata: 
      name: javawebapprcpod
      labels:
        app: javawebapp
    spec: 
      containers:
      - name: javawebapprccon
        image: kkeducationb2/java-webapp:1.1
        ports:
        - containerPort: 8080
```

---

## Command Summary

| Command                                               | Purpose                                    |
| ----------------------------------------------------- | ------------------------------------------ |
| `kubectl apply -f rc.yaml --dry-run=client`           | Test RC creation without applying          |
| `kubectl apply -f rc.yaml`                            | Create RC from YAML                        |
| `kubectl get rc -n <NS>`                              | List RCs in namespace                      |
| `kubectl get all -n <NS>`                             | List all resources in namespace            |
| `kubectl describe pod <pod-name> -n <NS>`             | Inspect Pod details                        |
| `kubectl scale rc <rcName> --replicas=N -n <NS>`      | Scale Pods up/down                         |
| `kubectl delete rc <rcName> -n <NS>`                  | Delete RC and its Pods                     |
| `kubectl delete rc <rcName> -n <NS> --cascade=orphan` | Delete RC but keep Pods                    |
| `kubectl delete pod <pod-name> -n <NS>`               | Delete a Pod manually to test self-healing |

```
