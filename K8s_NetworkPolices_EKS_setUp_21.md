# Network Policies in Kubernetes

Network Policies in Kubernetes are used to control communication between pods and external endpoints.

They define rules that determine how pods can connect to each other and to other network endpoints.

They help secure your cluster by allowing only authorized and required traffic.

---

## 🔑 Key Concepts of Network Policies

### **1. Namespaces**

Network Policies apply **within a specific namespace**.
Different namespaces can have different rules.

### **2. Selectors**

Used to choose which pods or namespaces the policy applies to.

* **Pod Selector** → selects specific pods
* **Namespace Selector** → selects pods based on namespace labels

### **3. Rule Types**

* **Ingress Rules** → control **incoming** traffic
* **Egress Rules** → control **outgoing** traffic

### **4. Policy Types**

A policy can include:

* **Ingress**
* **Egress**
* **Both**

---

# Example: Without Network Policies

All applications can communicate freely within the cluster.

---

# 1. MongoDB ReplicaSet + Service

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mongodb
  namespace: prod
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
        image: mongo
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
          path: /mongodata
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

# 2. Spring App + Service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sprinapp
  namespace: prod
spec:
  replicas: 1
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
  namespace: prod
spec:
  type: NodePort
  selector:
    app: springapp
  ports:
  - port: 80
    targetPort: 8080
```

---

# 3. Maven Web App + Service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: javawebappdep
  namespace: prod
spec:
  replicas: 1
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
        image: kkeducation123456/maven-web-app:1.2
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: javawebappsvc
  namespace: prod
spec:
  type: NodePort
  selector:
    app: javawebapp
  ports:
  - port: 80
    targetPort: 8080
```

---

# Connectivity Testing (Before Network Policies)

## Step 1: From Java Web App

```sh
kubectl exec -it javawebappdep-xxxx -n test-ns -- bash
curl -v telnet://mongosvc:27017   # Connected
```

## Step 2: From Spring App

```sh
kubectl exec -it sprinapp-xxxx -n test-ns -- sh
apk add curl
curl -v telnet://mongosvc:27017   # Connected
```

## Step 3: From Nginx Pod (Default Namespace)

Create pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

Test:

```sh
kubectl exec -it nginx -- bash
curl -v telnet://mongosvc.prod.svc.cluster.local:27017   # Connected
```

---

# Applying Network Policies

---

# Example 1: General Network Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: prod
spec:
  podSelector:
    matchLabels:
      app: springapp
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

---

# Example 2: Network Policy for Our App

Allow only **Spring App** to access **MongoDB**.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: mongodb-policy
  namespace: prod
spec:
  podSelector:
    matchLabels:
      app: mongodb
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: springapp
    ports:
    - protocol: TCP
      port: 27017
```

Apply:

```sh
kubectl apply -f networkpol.yaml
kubectl get netpol -n prod
```

---

# Troubleshooting

### ✔️ Spring App → MongoDB

```sh
kubectl exec -it sprinapp-xxxx -n test-ns -- sh
curl -v telnet://mongosvc:27017   # SHOULD WORK
```

### ❌ Java Web App → MongoDB

```sh
kubectl exec -it javawebappdep-xxxx -n test-ns -- bash
curl -v telnet://mongosvc:27017   # BLOCKED (Expected)
```

Because only **springapp** is allowed.

---

# Allow Java Web App Also

Add additional podSelector:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: mongodb-policy
  namespace: test-ns
spec:
  podSelector:
    matchLabels:
      app: mongodb
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: springapp
    - podSelector:
        matchLabels:
          app: javawebapp
    ports:
    - protocol: TCP
      port: 27017
```

Apply:

```sh
kubectl apply -f networkpol.yaml
kubectl get netpol -n test-ns
```

---

# Add NamespaceSelector

Example:

```yaml
- namespaceSelector:
    matchLabels:
      ns: test-ns
```
