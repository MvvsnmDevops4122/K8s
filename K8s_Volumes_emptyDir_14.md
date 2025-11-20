# ✅ Kubernetes Volumes

## 1. Why Do We Need Kubernetes Volumes?

By default, **data stored inside a container is temporary**.

This means:

* If a **container restarts → data is lost**
* If a **pod is rescheduled to another node → data is lost**

➡️ To solve this, Kubernetes introduces **Volumes** to store data outside the container filesystem.

---

## 2. What Is a Volume in Kubernetes?

Kubernetes Volumes allow Pods/containers to store and access data persistently. Since containers are ephemeral (deleted → data lost), 
Volumes solve the problem of data persistence, especially for databases like MongoDB, MySQL, PostgreSQL, etc. It allowing to

✔ Data persistence
✔ Data sharing between containers
✔ Storage managed by cluster/node/cloud

📌 **Important:**
A Volume is attached to a **Pod**, *not to a container.*
However, containers inside the same pod can mount and use it.

---

# 3. Types of Kubernetes Volumes

## 1️⃣ emptyDir

`emptyDir` is a **temporary storage space** created when the Pod starts.

### Features:

* Created when Pod starts
* Deleted when Pod is removed
* Suitable for **temporary data**
* Good for **sharing files between containers in the same Pod**

### Common Use Cases:

* Logs
* Cache
* Temporary processing files
* Shared data between containers in same Pod

### YAML Example

```yaml
volumes:
  - name: cache-data
    emptyDir: {}
```

---

# 🟦 Spring Application Deployment (Example)

### `springapp.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springapp
  namespace: test
spec:
  replicas: 2
  selector:
    matchLabels:
      app: springapp
  template:
    metadata:
      labels:
        app: springapp
    spec:
      containers:
      - name: springapp
        image: kkeducation12345/spring-app:1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: MONGO_DB_HOSTNAME
          value: mongosvc
        - name: MONGO_DB_USERNAME
          value: devdb
        - name: MONGO_DB_PASSWORD
          value: devdb@123
---
apiVersion: v1
kind: Service
metadata:
  name: springappsvc
  namespace: test
spec:
  type: NodePort
  selector:
    app: springapp
  ports:
  - port: 80
    targetPort: 8080
```

---

# 🟩 MongoDB ReplicaSet with emptyDir Volume

### `mongo_db_with_emptyDir.yaml`

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata: 
  name: mongodb-emptydir-rs
  namespace: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongocon
        image: mongo:8.0.9-noble
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value: devdb
        - name: MONGO_INITDB_ROOT_PASSWORD
          value: devdb@123
        volumeMounts:
        - name: mongo-emptydir-storage
          mountPath: /data/db
      volumes:
      - name: mongo-emptydir-storage
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: mongosvc
  namespace: test
spec:
  type: ClusterIP
  selector:
    app: mongodb
  ports:
  - port: 27017
    targetPort: 27017
```

---

# 🚀 Commands to Deploy

```bash
kubectl apply -f mongo_db_with_emptyDir.yaml
kubectl apply -f springapp.yaml
kubectl get pods -n test
kubectl get rs -n test
kubectl get svc -n test
```

If pod errors:

```bash
kubectl describe pod <pod-name> -n test
```

---

# 🔍 How to Check MongoDB Data is Stored

<img width="1916" height="967" alt="image" src="https://github.com/user-attachments/assets/d06b2053-d15f-4238-9ca9-28bf285c9aff" />


## 1️⃣ Get Pod Name

```bash
kubectl get pods -n test
```

Find pod like:

```
mongodb-emptydir-rs-xxxxx
```

## 2️⃣ Enter into MongoDB Pod

```bash
kubectl exec -it <pod-name> -n test -- bash
```

## 3️⃣ Check MongoDB Data Folder

```bash
ls -l /data/db
```

Expected files:

* WiredTiger
* WiredTiger.wt
* journal
* *.wt files
* metadata file

This confirms MongoDB is writing data to the emptyDir volume.

---

# 🧪 Test: Delete Pod and Check Data

## Step 1 — Delete Pod

```bash
kubectl delete pod <pod-name> -n test
```

ReplicaSet will create a new pod automatically.

## Step 2 — Check New Pod

```bash
kubectl get pods -n test
```

## Step 3 — Enter the New Pod

```bash
kubectl exec -it <new-pod-name> -n test -- bash
```

## Step 4 — Check Folder

```bash
ls /data/db
```

### ❗ RESULT:

* Folder `/data/db` exists
* **BUT old data is gone**

### 🔥 Why?

Because:

> **emptyDir data is deleted whenever the Pod is deleted.**
