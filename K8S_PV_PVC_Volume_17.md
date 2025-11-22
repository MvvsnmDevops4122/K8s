## PV & PVC ##
---

# ✅ Persistent Volume (PV)

* PV is actual storage available in the Kubernetes cluster.

* It is provisioned by an administrator or dynamically by a StorageClass.

* It is a cluster-scoped resource, It is independent of any specific pod or namespace.

* PV defines storage size, access modes (ReadWriteOnce / ReadOnlyMany / ReadWriteMany) and reclaim policy (Retain / Recycle / Delete).

* Supports multiple storage backends like NFS, EBS, iSCSI, HostPath, etc.

---

# 🟩 Persistent Volume Claim (PVC)

* PVC is a request for storage made by a pod or application.

* It is a namespaced resource, so it belongs to a specific namespace.

* PVC specifies required size, access mode, and optional StorageClass.

* When a PVC is created, Kubernetes attempts to find a suitable PV that matches the PVC's requirements and binds the PVC to that PV.

---

## How they work:

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

# 🟦 MongoDB Deployment with PVC

## 🟩 MongoDB Deployment YAML

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

# 🟩 PVC YAML

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

---

# Commands to Check PVC & PV

### Check PVC in `test` namespace

```bash
kubectl get pvc -n test
```

### Check all PVs

```bash
kubectl get pv
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
