# 📘 Kubernetes NFS, PV, PVC & MongoDB–Spring App Setup Guide

## 1️⃣ What is an NFS Server?

**NFS = Network File System**

An NFS server is a machine that shares a folder over the network, allowing other machines (clients) to:

✔ Access files stored on the server
✔ Read/write data
✔ Share the same data across multiple nodes

---

## 2️⃣ Why NFS is Important in DevOps & Kubernetes?

Kubernetes nodes are distributed, so each node has its own filesystem.
If a Pod moves from **Node A → Node B**, the data gets lost.

### ✅ NFS solves this:

* Storage is outside the cluster
* Any Pod on any Node can use the same data
* Data is persistent
* Perfect for DBs, logs, uploads, CI/CD agents, etc.

---

## 3️⃣ How NFS Works (Simple Diagram)

```
           ┌──────────────────┐
           │   NFS SERVER     │
           │ /mnt/nfs_share   │
           └───────┬──────────┘
                   │
      ┌────────────┼────────────┐
      │            │             │
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Node 1   │  │ Node 2   │  │ Node 3   │
│ (client) │  │ (client) │  │ (client) │
└──────────┘  └──────────┘  └──────────┘
```

All nodes mount:

```
172.31.11.218:/mnt/nfs_share → /data/db
```

---

## 4️⃣ Setup the NFS Server

### ✔ Step 1: Launch an EC2 machine

Use Ubuntu EC2 instance.

### ✔ Step 2: Update packages

```bash
sudo apt update -y
```

### ✔ Step 3: Allow NFS port

Open **2049** in EC2 Security Group.

### ✔ Step 4: Install NFS Server

```bash
sudo apt install nfs-kernel-server -y
```

### ✔ Step 5: Create the shared directory

```bash
sudo mkdir -p /mnt/nfs_share
sudo chown nobody:nogroup -R /mnt/nfs_share/
sudo chmod 777 -R /mnt/nfs_share/
```

### ✔ Step 6: Configure NFS exports

```bash
sudo vi /etc/exports
```

Add:

```
/mnt/nfs_share *(rw,sync,no_subtree_check,no_root_squash)
```

### ✔ Step 7: Export directory

```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

### ✔ Step 8: Verify NFS running

```bash
ps -ef | grep -i nfs
```

---

## 5️⃣ Configure NFS Client Machines

### ✔ Step 1: Update machine

```bash
sudo apt update -y
```

### ✔ Step 2: Install NFS client

```bash
sudo apt install nfs-common -y
```

---

# 🟦 Kubernetes Deployment Files

---

## 🟩 Spring Application (Deployment + Service)

### **springapplication.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springapp
  namespace: test-ns 
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
  namespace: test-ns
spec:
  type: NodePort
  selector:
    app: springapp
  ports:
  - port: 80
    targetPort: 8080
```

---

# 🟨 MongoDB Deployment Using NFS (Without PVC)

### **mongo.yaml**

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mongodb
  namespace: test
spec:
  selector:
    matchLabels:
      app: mongodb
  replicas: 1
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
        - name: mongonfsvol
          mountPath: /data/db
      volumes:
      - name: mongonfsvol
        nfs:
          server: 172.31.11.218
          path: /mnt/nfs_share
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

# 🟥 Understanding PV & PVC

## 6️⃣ PV — PersistentVolume

PV = Actual storage available in the cluster.

## 7️⃣ PVC — PersistentVolumeClaim

PVC = Request/claim for storage made by pods/applications.

### How they work:

1. PV provides storage
2. PVC claims storage
3. Kubernetes binds PVC → PV
4. Pods mount PVC

---

# Springapp with service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springapp
  namespace: test-ns 
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
  namespace: test-ns
spec:
  type: NodePort
  selector:
    app: springapp
  ports:
  - port: 80
    targetPort: 8080
```
---
## 🟩 MongoDB Deployment using PVC

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mongodb
  namespace: prod
spec:
  selector:
    matchLabels:
      app: mongodb
  replicas: 1
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongocon
        image: mongo
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value: devdb
        - name: MONGO_INITDB_ROOT_PASSWORD
          value: devdb@123
        volumeMounts:
        - name: mongonfsvol
          mountPath: /data/db
      volumes:
      - name: mongonfsvol
        persistentVolumeClaim:
          claimName: mongodb-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: mongosvc
  namespace: prod
spec:
  type: ClusterIP
  selector:
    app: mongodb
  ports:
    - port: 27017
      targetPort: 27017
```
---
# 🟦 MongoDB Deployment with PVC 

## 🟩 PV YAML

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 13.127.245.46
    path: /mnt/nfs_share
```

---

## 🟩 PVC YAML

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
  namespace: test
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```
# Commands to Check PVC & PV

### Check PVC in `test` namespace

```bash
kubectl get pvc -n test
```

### Check all PVs

```bash
kubectl get pv
```
