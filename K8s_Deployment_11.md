# Kubernetes Deployment

## 1. What is a Deployment?

* Deployments are the backbone of Kubernetes applications.
* They manage the lifecycle of Pods and provide features like scaling, rolling updates, and rollback.

👉 When you define and use a Deployment:
* The Deployment automatically creates a ReplicaSet in the background.

---

## ReplicaSet Management

* The ReplicaSet ensures that the desired number of Pods are running at all times.
* The ReplicaSet creates and manages Pods.
* Inside each Pod, there can be one or more containers running your application.

📌 **Flow:** Deployment → ReplicaSets (RS) → Pods → Containers

---

## Self-Healing in Deployments

* Kubernetes automatically replaces failed or unhealthy Pods to maintain the desired state.
* The controller manager constantly watches the Pods and recreates them if they fail.

👉 Once the Deployment is set up:
* The only manual task is updating configuration (e.g., changing image version).

After that, Kubernetes handles:
* Rolling updates (gradually replacing old Pods with new ones).
* Rollbacks (if an update causes issues, Kubernetes rolls back to the previous stable version).

---

## 2. Key Features of Deployments

* **Replica Management** – Ensures the specified number of Pods are always running.  
* **Rolling Updates** – Updates applications gradually with minimal or no downtime.  
* **Rollback Capability** – Easily revert to a previous version if a rollout fails.  
* **Declarative Scaling** – Adjust the number of replicas by changing a single field in the YAML file.  

---

## 3. Deployment Strategies in Kubernetes

When you release a new version of your application, Kubernetes offers different strategies to control how Pods are updated.  

The two main strategies are:

1. **Recreate**  
2. **RollingUpdate**

---

## 1. Recreate Strategy

* In this strategy, all the old Pods are terminated first, and then new Pods are created with the updated version.
* This means there will be downtime during the update because, for a short period, no Pods are available to serve requests.

### When to use Recreate?

* When downtime is acceptable.  
* When running two versions of the app at the same time could cause conflicts (e.g., database schema changes).  

### Example Flow

```

Step:1 Existing Pods → \[v1]\[v1]\[v1]
Step:2 Delete all Pods
Step:3 Start new Pods → \[v2]\[v2]\[v2]

````

⚠️ **Notice:** There’s a gap between step 2 and step 3 → downtime.

---

### YAML Example (Recreate Strategy)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: javawebapp
  namespace: test-ns
spec:
  replicas: 2
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: javawebapp
  template:
    metadata:
      labels:
        app: javawebapp
    spec:
      containers:
      - name: javawebapp
        image: k8seducation12345/spring-app:1.0.0
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: javawebappsvc
  namespace: test-ns
spec:
  type: NodePort
  selector:
    app: javawebapp
  ports:
    - port: 80
      targetPort: 8080
````

---

### Step 1: Apply Deployment

```bash
kubectl apply -f deploy.yaml
kubectl get all -n test-ns
```

* Creates Deployment, ReplicaSet, Pods, Service.
* Shows all resources (Pods with image 1.0.0).

Pods running:

```
pod/javawebappdeploy-59466fe944-2xskd
pod/javawebappdeploy-59466fe944-dnrcp
pod/javawebappdeploy-59466fe944-c6f8f
```

Deployment details:

* READY: 3/3
* UP-TO-DATE: 3
* AVAILABLE: 3

ReplicaSet details:

* DESIRED = 3
* CURRENT = 3
* READY = 3

📌 **Deployment → ReplicaSet → Pods**

---

### Checking Rollout Status

```bash
kubectl rollout status deployment javawebappdep -n test-ns
```

Output:

```
deployment "javawebappdep" successfully rolled out
```

---

### Step 2: Update to Version 1.0.1

* Update image in YAML:

  ```yaml
  image: kkeducation12345/spring-app:1.0.1
  ```

```bash
kubectl apply -f deploy.yaml
kubectl get all -n test-ns
```

Because strategy is **RollingUpdate**:

* Old Pods are deleted in batches.
* New Pods are created gradually → ✅ No downtime.

---

## Step 3: Checking Deployment Rollout History

* Each update creates a new ReplicaSet and a new revision number.
* You can confirm which image was deployed and what configuration was applied.

```bash
kubectl rollout history deployment javawebappdep -n test-ns --revision=2
```

---

### Step 4: Update Deployment to Version 1.0.2

Change YAML:

```yaml
image: kkeducation12345/spring-app:1.0.2
```

Apply update:

```bash
kubectl apply -f deploy.yaml
kubectl get all -n test-ns
```

---

### Step 5: Update to Version 1.0.3

Change YAML:

```yaml
image: kkeducation12345/spring-app:1.0.3
```

Apply and check rollout:

```bash
kubectl apply -f deploy.yaml
kubectl rollout status deployment javawebappdeploy -n test-ns
```

Rollout process shows:

* 0 of 3 available → downtime
* 1/3, 2/3 available → partial availability
* 3/3 available → rollout completed

---

### Step 6: Kubernetes Deployment Rollout History and Rollback

1. **Check rollout history**

   ```bash
   kubectl rollout history deployment javawebappdeploy -n test-ns
   ```

2. **Verify state**

   ```bash
   kubectl get all -n test-ns
   ```

3. **Perform rollback**

   ```bash
   kubectl rollout undo deployment javawebappdeploy -n test-ns --to-revision=3
   ```

👉 Kubernetes scales down the current RS and scales up the older RS.

4. **After rollback**
   Check again → old ReplicaSet becomes active.

5. **Revision numbers keep increasing**
   Even rollbacks get a new revision number.
   Rollback to revision 3 → logged as revision 5.

6. **Rolling back to Revision 1**

   ```bash
   kubectl rollout undo deployment javawebappdeploy -n test-ns --to-revision=1
   ```

---

## 2. RollingUpdate Strategy (Default in Kubernetes)

### Overview

* Pods are updated gradually, one by one (or in small batches).
* Some old Pods keep running while new Pods are created.
* Ensures minimal or zero downtime.

When to use:

* Application must be **available 24/7**
* Safe for old and new versions to run together

### Example Flow

```
Step 1: [v1][v1][v1]
Step 2: [v2][v1][v1]
Step 3: [v2][v2][v1]
Step 4: [v2][v2][v2]
```

---

### YAML Example (RollingUpdate Strategy)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: javawebappdep
  namespace: test-ns
spec:
  replicas: 3
  selector:
    matchLabels:
      app: javawebapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: javawebapp
    spec:
      containers:
      - name: javawebapp
        image: kkeducation12345/spring-app:1.0.1
        ports:
        - containerPort: 8080
```

---

### RollingUpdate Process

1. Apply Deployment:

   ```bash
   kubectl apply -f deploy.yaml
   ```

2. Check Pods:

   ```bash
   kubectl get po -n test-ns
   ```

   * New Pods → `ContainerCreating`
   * Old Pods → `Terminating`
   * New Pods eventually reach `Running` state.

3. Final State:

   * Old ReplicaSet → scaled down to 0.
   * New ReplicaSet → scaled up to desired count.

---

### Why RollingUpdate is Preferred?

* ✅ **No downtime** – At least one Pod always running.
* ✅ **Safer upgrades** – Stops rollout if new Pods fail.
* ✅ **Easy rollback** – Switch back to older ReplicaSet.
* ✅ **Production standard** – Default strategy for high availability clusters.

```

---

Do you want me to **split this into multiple smaller `.md` files** (like `01-deployment.md`, `02-strategies.md`, `03-rollout.md`) for better readability, or keep it as one long file?
```
