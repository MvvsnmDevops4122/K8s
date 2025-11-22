# 📘 **Kubernetes NFS Volume
---

# ⭐ 1️⃣ What is an NFS Server?

**NFS = Network File System**
           
An NFS server is a machine that shares a folder over the network, allowing other machines (clients) to:

### ✔ What NFS allows:

* Read/write shared files
* Access same data from multiple nodes
* Store application or database data centrally
* Provide persistent storage for Kubernetes Pods

---

# ⭐ 2️⃣ Why NFS is Needed in Kubernetes?

Kubernetes Pods run on **different Nodes**, and each node has its own local filesystem.

### ❌ Problem without NFS:

* Data stored inside a Pod is **lost** when the Pod restarts
* If a Pod moves **Node A → Node B**, the old data is not available
* Databases like MongoDB/MySQL lose data

### ✅ Solution: NFS provides shared storage

* Storage is **outside the Kubernetes cluster**
* Any pod on any node can access the same folder
* Data remains safe even if Pods or Nodes restart
* Perfect for:
  ✔ MongoDB / MySQL / PostgreSQL
  ✔ Jenkins Home
  ✔ Upload folders
  ✔ Log storage
  ✔ Shared config files

---

# ⭐ 3️⃣ NFS Working Diagram (Very Simple)

```
         ┌──────────────────┐
         │   NFS SERVER     │
         │ /mnt/nfs_share   │
         └───────┬──────────┘
                 │
    ┌────────────┼────────────┐
    │            │             │
┌──────────┐┌──────────┐┌──────────┐
│ Node 1   ││ Node 2   ││ Node 3   │
│ (client) ││ (client) ││ (client) │
└──────────┘└──────────┘└──────────┘
```

All Kubernetes nodes mount:

```
172.31.11.218:/mnt/nfs_share → /data/db
```

So **any pod** can read/write the same data.

---

# ⭐ 4️⃣ How to Setup an NFS Server (Simple)

### ✔ Step 1: Launch an Ubuntu EC2 machine

Open NFS port **2049** in Security Group.

### ✔ Step 2: Update packages

```bash
sudo apt update -y
```

### ✔ Step 3: Install NFS Server

```bash
sudo apt install nfs-kernel-server -y
```

### ✔ Step 4: Create shared folder

```bash
sudo mkdir -p /mnt/nfs_share
sudo chown nobody:nogroup -R /mnt/nfs_share
sudo chmod 777 -R /mnt/nfs_share
```

### ✔ Step 5: Configure /etc/exports

```bash
sudo vi /etc/exports
```

Add line:

```
/mnt/nfs_share *(rw,sync,no_subtree_check,no_root_squash)
```

### ✔ Step 6: Apply export settings

```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

### ✔ Step 7: Check NFS running

```bash
ps -ef | grep nfs
```

---

# ⭐ 5️⃣ Setup NFS Client on Kubernetes Nodes

### Install NFS client package:

```bash
sudo apt update -y
sudo apt install nfs-common -y
```

---

# 🟦 Kubernetes Deployment Files

---

# 🟩 **Spring Boot Application (Front-end API Layer)**

### springapplication.yaml

✔ Namespace: `test`
✔ Uses MongoDB Service via ENV variables
✔ NodePort service for browser access

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
# Apply Spring Application

```yaml
kubectl apply -f springapplication.yaml
```
---

# 🟨 **MongoDB Deployment Using NFS

### mongo.yaml

✔ ReplicaSet
✔ NFS mounted to `/data/db`
✔ Stores data permanently on NFS Server
✔ Namespace should be the same as spring (`test-ns`)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mongodb
  namespace: test-ns
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
  namespace: test-ns
spec:
  type: ClusterIP
  selector:
    app: mongodb
  ports:
    - port: 27017
      targetPort: 27017
```
# Apply MongoDB

```yaml
kubectl apply -f mongo.yaml
```
---

# ⭐ 6️⃣ Why This Setup Works Perfectly

### ✔ MongoDB uses NFS → data persists

### ✔ Spring connects to MongoDB via service

### ✔ Both apps are in the same namespace → DNS resolves

### ✔ NFS is outside the cluster → no data loss


✅ NFS-based PV & PVC version
✅ StatefulSet version for MongoDB
✅ Architecture diagram (Visually beautiful)
Just tell me!
