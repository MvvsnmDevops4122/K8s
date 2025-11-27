# **Reclaim Policies**

======================

## **1. Retain (Default)**

---

* When a PVC is deleted, the PV is not removed; it moves into a **Released** state with all data still preserved.
* It allows you to manually reclaim, inspect and manage the data.
* This is the **default reclaim policy**.
* This policy is used when **data must not be lost**.

---

## **2. Delete**

---

* When a PVC is deleted, the PV is automatically deleted along with its underlying storage (disk/NFS/EBS/etc.).
* All data is permanently lost.
* This policy is used when data is **temporary and not required** after PVC deletion.

---

## **Example:**

---

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv
spec:
  capacity:
    storage: 1Gi  # Adjust as necessary
  accessModes:
    - ReadWriteMany  # NFS supports this
  nfs:
    server: 172.31.40.109  # Your NFS server IP
    path: /mnt/nfs_share    # Path on your NFS server
  persistentVolumeReclaimPolicy: Delete  # Retain or Delete based on use case
```

---

# **Dynamic Volumes**

=====================

* Dynamic Volume Provisioning allows Kubernetes to automatically create PersistentVolumes (PVs) whenever a PersistentVolumeClaim (PVC) is created.
* Storage is provisioned automatically based on the PVC request through a StorageClass.
* No manual PV YAML creation is required.
* **PVC requests storage → StorageClass provisions PV → Pod consumes the volume.**

---

# **NFS Dynamic Provisioner – Full YAML Setup**

==============================================

```yaml
kind: ServiceAccount
apiVersion: v1
metadata:
  name: nfs-pod-provisioner-sa
  namespace: kube-system
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-provisioner-clusterRole
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-provisioner-rolebinding
subjects:
  - kind: ServiceAccount
    name: nfs-pod-provisioner-sa
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: nfs-provisioner-clusterRole
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-pod-provisioner-otherRoles
  namespace: kube-system
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-pod-provisioner-otherRoles
  namespace: kube-system
subjects:
  - kind: ServiceAccount
    name: nfs-pod-provisioner-sa
    namespace: kube-system
roleRef:
  kind: Role
  name: nfs-pod-provisioner-otherRoles
  apiGroup: rbac.authorization.k8s.io
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nfs-pod-provisioner
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nfs-pod-provisioner
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nfs-pod-provisioner
    spec:
      serviceAccountName: nfs-pod-provisioner-sa
      containers:
        - name: nfs-pod-provisioner
          image: rkevin/nfs-subdir-external-provisioner:fix-k8s-1.20
          volumeMounts:
            - name: nfs-provisioner-v
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: nfs-provisioner
            - name: NFS_SERVER
              value: 172.31.4.196
            - name: NFS_PATH
              value: /mnt/nfs_share
      volumes:
       - name: nfs-provisioner-v
         nfs:
           server: 172.31.4.196
           path: /mnt/nfs_share
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-storageclass
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: nfs-provisioner
parameters:
  archiveOnDelete: "false"
```

---

### **Apply the file**

```
kubectl apply -f nfsProviser.yaml
```

---

### **NOTE**

To understand the dynamic provisioning flow:
➡ Delete **ReplicaSet, PVC, and PV** in the system and test again.

---

# **MongoDB PVC + ReplicaSet + Service**

========================================

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
  namespace: prod
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
---
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

### Application Access
---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: sprinapp
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
        resources:
          requests:
            cpu: 300m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
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
