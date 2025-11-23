# 📘 ConfigMap and Secrets in Kubernetes

## **ConfigMap**

* A **ConfigMap** is a Kubernetes API object used to store **non-confidential (non-sensitive)** data such as:

  * Environment variables
  * Command-line arguments
  * Configuration files
* Data in a ConfigMap is stored in **plain text**.

---

## **Secrets**

* A **Secret** is a Kubernetes API object used to store **sensitive (confidential)** data such as:

  * Passwords
  * API keys
  * TLS certificates
* Secret data is stored in **base64-encoded format** and **can be encrypted at rest in etcd** (if Kubernetes encryption is enabled).

---

# 📦 MongoDB Deployment with Service, PV, PVC, ConfigMap & Secret

Below is the complete setup to deploy MongoDB using:

* **ReplicaSet**
* **Service**
* **PersistentVolume**
* **PersistentVolumeClaim**
* **ConfigMap**
* **Secret**
* **NFS storage**

All resources are created in the **`test` namespace**.

---

# 1️⃣ Create Namespace

```bash
kubectl create ns test
kubectl get ns
```

---

# 2️⃣ MongoDB ReplicaSet + Service

### **mongodb.yaml**

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
          valueFrom:
            configMapKeyRef:
              name: springappconfig
              key: db_username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: springappsecret
              key: db_password
        volumeMounts:
        - name: mongonfsvol
          mountPath: /data/db
      volumes:
      - name: mongonfsvol
        persistentVolumeClaim:
          claimName: mongodb1-pvc
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

### Apply and check:

```bash
kubectl apply -f mongodb.yaml
```
```bash
kubectl get all -n test
```
<img width="977" height="215" alt="{FEB1E4E4-CE1F-49E9-9C24-7D88F4661493}" src="https://github.com/user-attachments/assets/dfb70da5-22cf-4c3b-9234-52c40ee117de" />

```bash
kubectl describe po mongodb-s4hkt -n test
```
<img width="1878" height="127" alt="{59068FA1-84EA-4672-9C38-4B1D20E628EC}" src="https://github.com/user-attachments/assets/76edcb3f-ce9d-4e3e-88c7-ef8a5f184730" />

Pod is pending state because of PVC is not found.

---

# 3️⃣ PVC (PersistentVolumeClaim)

### **pvc.yaml**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb1-pvc
  namespace: test
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```

Apply PVC:

```bash
kubectl apply -f pvc.yaml
```
<img width="1261" height="125" alt="{B7B86301-C99C-48ED-BC2A-E41DC7F20840}" src="https://github.com/user-attachments/assets/04f0e29f-bacc-4b37-b313-5bbc2c9e1abb" />

```bash
kubectl get pvc -n test
```
```bash
kubectl describe pvc mongodb1-pvc -n test
```
<img width="1785" height="106" alt="{B3FB3F0C-71FA-4AA9-9297-172A99FB4A19}" src="https://github.com/user-attachments/assets/ba332d8f-668f-48d9-8244-36d9f5f6fb03" />

If PVC is **Pending**, it means **no PV available**.

---

# 4️⃣ PV (PersistentVolume)

### **pv.yaml**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 172.31.32.159
    path: /mnt/nfs_share
```

Apply PV:

```bash
kubectl apply -f pv.yaml
```
<img width="1795" height="150" alt="{14214FE7-8A45-4E4D-96F8-4A282E0DE2CC}" src="https://github.com/user-attachments/assets/4d7427a8-df0b-46b8-90f3-3bbff4d8b28d" />

```bash
kubectl get pv
```

You should see:
<img width="1372" height="81" alt="{5280451F-660F-4908-83F6-A3F33976DB05}" src="https://github.com/user-attachments/assets/a4be961d-2b5f-4eb8-b86a-42b532600a10" />

```
STATUS = Bound
```

Then PVC also becomes:
```bash
kubectl get pvc -n test
```
<img width="1343" height="70" alt="{C84011D1-7592-4184-9D03-24AC38DAAB5C}" src="https://github.com/user-attachments/assets/b986ff97-b6f1-4981-84b9-061a8481af79" />

```
STATUS = Bound
```
## Now check pod status

```bash
kubectl get po -n test
```
<img width="898" height="74" alt="{CF45B198-DC3E-4CFC-8AFE-85E29263730E}" src="https://github.com/user-attachments/assets/fb8d9816-3c8d-4ef8-b61b-92a1b883c507" />

<img width="1920" height="373" alt="{80E09FC5-9E60-47C9-B12E-6F5273780BD7}" src="https://github.com/user-attachments/assets/c5b5a046-f7c7-468c-938c-0605ffbc7280" />

STATUS= CreateContainerConfigError

Error: configmap "springappconfig" not found

---

# 5️⃣ ConfigMap

### **configmap.yaml**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: springappconfig
  namespace: test
data:
  db_username: devdb
```

Apply and verify:

```bash
kubectl apply -f configmap.yaml
kubectl get cm -n test
kubectl describe cm springappconfig -n test
```

---

# 6️⃣ Secret

### **secret.yaml**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: springappsecret
  namespace: test
type: Opaque
stringData:
  db_password: devdb@123
```

Apply and verify:

```bash
kubectl apply -f secret.yaml
kubectl get secrets -n test
kubectl describe secret springappsecret -n test
```

---

# 7️⃣ Final Check — Pod Status

```bash
kubectl get po -n test
```
<img width="815" height="80" alt="{413B5DB5-176E-4DA5-9130-5658B87C7BFD}" src="https://github.com/user-attachments/assets/5a43121a-22aa-42c6-99e0-3dd5c937a573" />

You should now see:

```
STATUS = Running
```

If pod was stuck earlier (Pending → ConfigMap error):

* Missing ConfigMap or Secret was the issue
* After creating both in **test namespace**, the pod successfully starts

---

# ✅ Final Output

* **PV = Bound**
* **PVC = Bound**
* **ConfigMap = Available**
* **Secret = Available**
* **ReplicaSet Pod = Running**
* **MongoDB service = Ready**

Your MongoDB is now fully deployed with persistent storage + config + secret management.
