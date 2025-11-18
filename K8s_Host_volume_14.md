# Kubernetes Volumes

Kubernetes Volumes allow Pods/containers to store and access data persistently.
Since containers are ephemeral (deleted → data lost), Volumes solve the problem of data persistence, especially for databases like MongoDB, MySQL, PostgreSQL, etc.

---

## 1. Why do we need Volumes?

Containers are temporary → Data inside container filesystem disappears when container restarts.

Databases need persistent storage.

Logs or uploaded files may need to be stored safely.

Volumes ensure:

✔ Data is not lost on Pod restart
✔ Data is shared between containers
✔ Data exists outside the Pod lifecycle

---

## 2. Types of Volumes in Kubernetes

### 1. hostPath

Stores data on the Kubernetes node filesystem.

Good for testing.

Not recommended for production (node failure = data loss).

---

## Without host path volume

### Example: MongoDB and service without Volumes

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata: 
  name: mongodb
  namespace: test-ns
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

---

### Example: Spring Boot app and service without Volumes

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

### Check MongoDB directory inside Pod

```bash
kubectl exec -it <pod-Name> -n <NS> -- ls /data/db
```

---

## After adding host path volumes

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata: 
  name: mongodb
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
        - name: mongovol
          mountPath: /data/db
      volumes:
      - name: mongovol
        hostPath:
          path: /mongobkp
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
