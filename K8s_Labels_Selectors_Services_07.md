# 📘 Kubernetes Labels, Selectors & Services

---

## 1. Labels:

* Labels are key/value pairs attached to Kubernetes objects (Pod, Service, Deployment, etc.).
* They are mainly used for organization and grouping of resources.
* You can create your own labels and assign them to any object.

Example:

```yaml
labels:
  app: javawebapp
  environment: production
````

👉 Think of labels like tags in Kubernetes.

---

## 2. Selectors

* A selector is used to find/match  objects by their labels.
* Services, ReplicaSets, and Deployments use selectors to find and connect to the right Pods.

Example:

```yaml
selector:
  app: javawebapp
```

---

## 3. Why Are They Important?

* Labels & selectors are crucial for creating Services.
* They ensure traffic is routed to the right Pods.
* Without labels → Services can’t find Pods → traffic won’t work.
* Using labels, you can easily group Pods (like dev, test, prod).

### Simple workflow:

* Add labels to Pods.
* Create a Service with a selector.
* Kubernetes will automatically send traffic from the Service → to Pods with matching labels.

---

## 4. How Do Labels Form Groups?

* Labels act like group tags.
* Pods with app: javawebapp → form one group.
* Pods with environment: production → form production group.

This way you can create logical groups across your cluster.

---

## 5. What If Labels Were Missing?

❌ Services won’t know which Pods to connect.
❌ Traffic won’t reach the right Pods.
❌ Hard to manage environments (chaos).

👉 Labels are essential for smooth functioning of Kubernetes.

---

## 6. Creating Pods with Labels

* In Kubernetes, labels are key–value pairs attached to objects (like Pods).

### One Javawebapp Pod with a label.

**javawebapp.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: javawebapp
  labels:
    app: javawebapp   # This is the label we are creating
  namespace: test
spec:
  containers:
    - name: javawebappcontainer
      image: satyamolleti4599/maven-web-app:1.0.0
      ports:
        - containerPort: 8080
```

👉 Here you are creating a pod called javawebapp in namespace test with a label app: javawebapp.

### ▶️ Run:

```bash
kubectl apply -f javawebapp.yaml --dry-run=client   # Check before apply
kubectl apply -f javawebapp.yaml
```

### Verify the Pod:

```bash
kubectl get pods -n test
```

---

## 7. Create Another Pod with Labels

### Mangodb Pod with a label.

**mongo.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mongopod
  namespace: test
  labels:
    app: mongo   # This is the label we are creating
spec:
  containers:
  - name: mongodbcontainer
    image: mongo
    ports:
    - containerPort: 27017
```

### Run:

```bash
kubectl apply -f mongo.yaml
```

### Verify the pod

```bash
kubectl get pods -n test
```

---

## 8. Check Labels of Pods

### Detailed labels & info:

```bash
kubectl describe pod javawebapp -n test
kubectl describe pod mongopod -n test
```

### To see only labels:

```bash
kubectl get po -n test --show-labels
```

---

## 9. Check Pod IPs

```bash
kubectl get pods -n test -o wide
```

* Shows Pod IP, Node, Status. Pod IPs are internal and can be used for pod-to-pod communication.

---

## 10. Verifying Pod-to-Pod Communication Ping from One Pod to Another

1. Go inside javawebapp Pod:

```bash
kubectl exec -it javawebapptest -n test -- sh
```

2. Install ping (if missing):

```bash
apt-get update && apt-get install -y iputils-ping
```

3. Ping MongoDB Pod using its IP:

```bash
ping <MongoPod-IP>
```

* If successful → Pods can communicate across nodes.
* But Pod IPs are temporary → in real projects, use Services instead of Pod IPs.

---

## 11. Why Service is Needed?

* Pods are ephemeral → Pod IP changes when pod restarts.
* Service gives a stable DNS name + virtual IP to access pods.
* Services can distribute traffic across multiple pods with the same label.

---

## 12. Service in Kubernetes

* Provides stable communication between Pods.
* Uses labels & selectors to connect to Pods.
* Creates an Endpoint object that stores Pod IPs & ports.

---

## 13. Why Services Need Labels

* Labels are key-value pairs attached to Pods.
* A Service in Kubernetes always uses selectors to find or match Pods by their labels.
* Without labels, a Service cannot find or connect to Pods.
* Kubernetes automatically creates an Endpoint object for each Service, storing the IP addresses and ports of the matching Pods.

---

## 14. Endpoints

* Endpoints contain network information (Pod IPs + Ports).
* They ensure that traffic from the Service is always routed to the correct Pods.
* If Pods change, Endpoints are updated automatically.

---

## 15. Types of Kubernetes Services

ClusterIP | NodePort | LoadBalancer | ExternalName

---

### 1. ClusterIP Service (Default)

* This is the default Service type in Kubernetes.
* Kubernetes assigns a cluster-internal IP address to the Service.
* Exposes the service on a cluster-internal IP. Service is accessible only within the cluster.
* Not accessible from outside the cluster unless exposed with NodePort/LoadBalancer/Ingress.

Optional: you can set clusterIP in the service YAML.

**Flow:** Pod ➝ Label ➝ Service ➝ Endpoint ➝ Traffic Routing

---

#### Example: Java Web App Pod + Service

**javawebapp-service.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: javawebapp-pod
  namespace: test
  labels:
    app: javawebapp   # Label added here
spec:
  containers:
  - name: javawebapp-cont
    image: satyamolleti4599/maven-web-app:1.0.0
    ports:
    - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: javawebapp-service
  namespace: test
spec:
  selector:              # Service will target pods with this label
    app: javawebapp      # must match the pod label
  ports:
  - protocol: TCP
    port: 8080           # Service port (cluster-internal)
    targetPort: 8080     # Container port inside the pod
  type: ClusterIP        # Default
```

### Step 1: Apply the Pod + Service

```bash
kubectl apply -f javawebapp-service.yaml
```

### Step 2: Check All Pods & Check All Resources

```bash
kubectl get all -n test
```

### Step 3: Verify the Service

```bash
kubectl get svc -n test
```

### Step 4: Check Service Details

**Describe Service:**

```bash
kubectl describe svc javawebapp-service -n test
```

**Verify Pod IP**

```bash
kubectl get pods -n test -o wide
```

* Endpoints → point to the Pod IP(s) (from kubectl get pods -o wide).
* Matches the Endpoints IP.
* This confirms that traffic routing is correct.

### Step 5: Directly Get Endpoints

```bash
kubectl get ep javawebapp-service -n test        # Endpoints of a specific Service
kubectl get ep -n test                           # All Endpoints in a namespace
```

* Shows the list of Pod IPs the Service will forward to.
* If multiple replicas exist, you’ll see multiple IPs here (load balancing).
* If ENDPOINTS is empty → Service exists, but no matching pods are ready.

---


## 2. NodePort

* Exposes the Service on a static port (range: 30000–32767) on each node’s IP.
* Extension of ClusterIP.
* Accessible from outside the cluster using:<NodeIP>:<NodePort>.
* Each Node forwards traffic to the Service → Pods.

### Use Cases

* External access without a cloud load balancer.
* Useful for testing / small setups.

**Example: Java Web App Pod + NodePort Service**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: javawebapp-nodeport
  namespace: test
  labels:
    app: java
spec:
  containers:
  - name: javawebapp1
    image: satyamolleti4599/maven-web-app:1.0.0
    ports:
    - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: javawebapp-nodeport-service
  namespace: test
spec:
  selector:
    app: java          # Must match Pod label
  ports:
  - port: 8080           # ClusterIP service port
    targetPort: 8080     # Pod container port
    nodePort: 30080      # NodePort (must be 30000-32767 if specified)
  type: NodePort
```

### Apply:

```bash
kubectl apply -f javawebapp-nodeport.yaml
```

### Verify Service:

```bash
kubectl get svc -n test
```

8080 → ClusterIP port (inside cluster)
30080 → NodePort (exposed externally on all nodes)

**Service Description**

```bash
kubectl describe svc javawebapp-nodeport-service -n test
```

Pod IP: 10.36.0.1
Port: 8080 (containerPort)

---

### NodePort Service Traffic Flow (with Labels & Selectors)

Client (outside cluster)
↓
Example: EC2-Public-IP:30080
↓
Node → forwards request to NodePort Service
↓
Service (selector: app=java)
↓
Finds Pods with label app=java
↓
Resolves to Pod IP(s) via Endpoints
↓
Traffic routed to Pod containerPort (8080)
↓
Java Web App responds back to client

---

### Test Access (NodePort Service)

Once the Pod + Service are applied and running:

Get your Node’s Public IP (for example, EC2 instance public IP).

```
http://<NodeIP>:30080
```

You should reach your Java Web App pod via the NodePort.

<img width="1324" height="683" alt="image" src="https://github.com/user-attachments/assets/9b4ca3e4-681e-4fb6-b7ab-2549b8209bf9" />

---

## 3. LoadBalancer

* Extension of NodePort.
* Integrates with cloud provider’s load balancer (AWS ELB, GCP LB, Azure LB).
* Exposes service to the internet with public IP.
* Mostly used in production deployments.
* The cloud load balancer automatically routes traffic → Service → Pods.

### Use Cases

* Production-grade external exposure.
* When running Kubernetes on cloud providers.

---

## 4. ExternalName

* Maps a Service to an external DNS name instead of Pod IPs.
* No selectors, no proxying.
* Returns a CNAME record to redirect traffic.

### Use Cases

* Connect Kubernetes Pods to external databases/services.

```
