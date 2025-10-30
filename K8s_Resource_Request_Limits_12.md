## 1️⃣ What Are Resource Requests?

**Definition:**  
Resource requests define the **minimum amount of CPU and memory** a container needs to run smoothly.

When defining a Pod, you are telling Kubernetes:

> “My container needs at least this much CPU and memory to function properly.”

Kubernetes uses these values to decide **where to schedule** the Pod.  
It will place the Pod only on Nodes that have at least these resources available.

**Example:**
```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "500m"
````

**Meaning:**

* `memory: 256Mi` → Container needs at least **256 MiB of memory**
* `cpu: 500m` → Container needs **0.5 CPU core**

**Scheduling Behavior:**
Kubernetes checks for Nodes with **≥ 0.5 CPU and 256Mi memory** before scheduling the Pod.

---

## 2️⃣ What Are Resource Limits?

**Definition:**
Resource limits define the **maximum CPU and memory** a container can use.

When you define limits, you’re telling Kubernetes:

> “Don’t allow this container to use more than this much CPU or memory.”

**Example:**

```yaml
resources:
  limits:
    memory: "512Mi"
    cpu: "1"
```

**Meaning:**

* `memory: 512Mi` → Can use up to **512 MiB memory**
* `cpu: 1` → Can use up to **1 CPU core**

**If Limits Are Exceeded:**

* **Memory** → Pod gets terminated (`OOMKilled`)
* **CPU** → Pod’s CPU usage is throttled (slowed down)

---

## 3️⃣ Deploying with Requests Only (No Limits)

If you set **only requests** (and no limits), Pods are guaranteed their requested resources but can use more if available.

**Example YAML:**
`resourcerequest.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: javawebappdeploy
  namespace: test-ns
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
  selector:
    matchLabels:
      app: javawebapp
  template:
    metadata:
      name: javawebapppod 
      labels:
        app: javawebapp
    spec:
      containers:
      - name: javawebappcon
        image: satyamolleti4599/maven-web-app:1.0.0
        resources:
          requests:
            memory: "256Mi"
            cpu: "600m"
        ports:
        - containerPort: 8080
```

**Commands:**

```bash
# Deploy
kubectl apply -f reqonly.yaml

# Check pods
kubectl get pods -n <namespace>

# Inspect node resource allocation
kubectl describe node <node-name>
```

🧠 **Observation:**
Requests appear under node allocation; limits show as **zero** if not set.

---

## 4️⃣ Scaling the Deployment

Increase the number of Pod replicas:

```bash
kubectl scale deploy javawebapp --namespace test-ns --replicas=3
```

If there isn’t enough CPU or memory, new Pods will stay in **Pending** state.

**Why Pending?**
Kubernetes can’t schedule Pods when Nodes lack available resources.

**Clean Up:**

```bash
kubectl delete -f reqonly.yaml
```

---

## 5️⃣ Adding Limits

Adding limits restricts the **maximum** resources a Pod can use.

If a container exceeds its **memory limit**, it’s **killed (OOMKilled)** and restarted.
Repeated restarts cause **CrashLoopBackOff**.

**Example:**

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "600m"
  limits:
    memory: "256Mi"
    cpu: "900m"
```

**Effect:**

* Over 256Mi memory → Container killed
* Over 900m CPU → CPU usage throttled

---

## 6️⃣ Fixing CrashLoopBackOff by Increasing Limits

If Pods crash because of low limits, increase them in the YAML.

**Steps:**

1. Edit YAML to set higher limits:

   ```yaml
   limits:
     memory: "1Gi"
     cpu: "1"
   ```
2. Apply the update:

   ```bash
   kubectl apply -f req-limits.yaml
   ```
3. Check Pod status:

   ```bash
   kubectl get po -n test-ns
   ```

Pods should now start successfully without crashing.

---

## 🧠 Key Takeaways

| Concept              | Description                                      |
| -------------------- | ------------------------------------------------ |
| **Requests**         | Minimum guaranteed CPU and memory for scheduling |
| **Limits**           | Maximum CPU and memory allowed for a container   |
| **Too Low Requests** | Pod may not schedule (Pending)                   |
| **Too Low Limits**   | Pod may crash (OOMKilled or CrashLoopBackOff)    |
| **Too High Values**  | Wastes cluster resources                         |

✅ **Best Practice:**
Set realistic **requests and limits** for balanced performance and resource utilization.
