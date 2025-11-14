# **Kubernetes Horizontal Pod Autoscaler (HPA)**

## **1. Introduction**

Kubernetes **Horizontal Pod Autoscaler (HPA)** automatically scales your application (adds/removes Pods) based on real-time CPU, Memory, or custom metrics.

Without HPA, you must manually scale Pods:

```bash
kubectl scale deployment <DEPLOYMENT_NAME> --replicas=<NUMBER>
```

With HPA:

* Pods **increase** when traffic/load increases
* Pods **decrease** when load reduces
* No need to monitor or manually adjust replicas
* Kubernetes ensures the app stays responsive

---

## **2. What is HPA?**

Before HPA → scaling was **manual**.

Example:

```bash
kubectl scale deployment <DEPLOYMENT_NAME> --replicas=<NUM>
```

Problems with manual scaling:

* Must continuously monitor CPU/memory
* Must scale up/down by hand

With HPA:

* Kubernetes auto-scales based on CPU/Memory
* No manual intervention
* Helps maintain performance

**Horizontal scaling** → Add/remove Pods
**Vertical scaling** → Increase CPU/RAM inside a Pod

HPA adjusts the number of Pods in:

* Deployment
* ReplicaSet
* StatefulSet

Kubernetes compares Pod usage to the **target utilization**.

Example (minReplicas: 2, maxReplicas: 5):

* Low traffic → 2 Pods
* High traffic → 5 Pods
* Normal → scales down to 2 Pods

---

## **3. HPA vs VPA**

### **Horizontal Pod Autoscaler (HPA)**

* Scales **number of Pods**
* Works on **CPU, memory, or custom metrics**
* Best for **variable traffic** (web apps, APIs)
* Needs **Metrics Server**

### **Vertical Pod Autoscaler (VPA)**

* Scales **resources inside the Pod** (CPU/RAM)
* Does **NOT** increase pod count
* Best for apps with stable traffic but unknown resource needs
* Can run in **recommendation** or **auto-update** mode

---

# **4. Kubernetes HPA Demo (Step-by-Step)**

## **4.1 Check Metrics Availability**

```bash
kubectl top nodes
kubectl top po -A
```

If you see:

```
error: Metrics API not available
```

→ Metrics Server is missing.

---

## **4.2 Install Metrics Server**

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Check status:

```bash
kubectl get all -n kube-system
```

If Metrics Server Pod shows **0/1** → TLS issue.

---

## **4.3 Fix Metrics Server TLS Issue**

Edit deployment:

```bash
kubectl edit deploy metrics-server -n kube-system
```

Under:
`spec.containers[].args:`

Add:

```
--cert-dir=/tmp
--secure-port=10250
--kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
--kubelet-use-node-status-port
--metric-resolution=15s
--kubelet-insecure-tls
```

Apply and check:

```bash
kubectl get all -n kube-system
```

---

## **4.4 Verify Metrics**

```
kubectl top nodes
kubectl top po -A
```

Now CPU and Memory usage should appear.

---

# **5. Deploy HPA Demo Application**

Create `hpa-demo.yaml` containing Deployment, HPA, and Service.

---

## **5.1 Deployment**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpadeployment
  labels:
    name: hpadeployment
spec:
  replicas: 1
  selector:
    matchLabels:
      name: hpapod
  template:
    metadata:
      labels:
        name: hpapod
    spec:
      containers:
      - name: hpacontainer
        image: k8s.gcr.io/hpa-example
        ports:
        - name: http
          containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "64Mi"
          limits:
            cpu: "100m"
            memory: "128Mi"
```

Important:

* Creates **1 initial Pod**
* Pod label → `name=hpapod`
* CPU/Memory requests & limits required for HPA

---

## **5.2 HPA Configuration**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpadeploymentautoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpadeployment
  minReplicas: 2
  maxReplicas: 4
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 30
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

Key Points:

* **minReplicas = 2** → overrides Deployment replicas
* **maxReplicas = 4** → cannot scale beyond this
* Scales when:

  * CPU > 30%
  * Memory > 80%

---

## **5.3 Service**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hpaclusterservice
  labels:
    name: hpaservice
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    name: hpapod
  type: ClusterIP
```

This routes internal traffic to Pods with label `name=hpapod`.

Apply the file:

```bash
kubectl apply -f hpa-demo.yaml
```

Check HPA:

```bash
kubectl get hpa
```

---

# **6. When Will Scaling Happen?**

* **CPU > 30%** → Scale up
* **Memory > 80%** → Scale up

Example:

* 2 Pods → 3 Pods → 4 Pods (max)

---

# **7. Watch Autoscaling Live**

Open two terminals:

```
watch kubectl get po
watch kubectl get hpa
```

Initially:

* 2 Pods running
* CPU/Memory low → No scaling yet

---

# **8. Generate Load**

Run BusyBox load generator:

```bash
kubectl run -i --tty load-generator --rm --image=busybox -- /bin/sh
```

Inside BusyBox:

```bash
while true; do wget -q -O- http://hpaclusterservice; done
```

What happens:

* BusyBox continuously hits the service
* Pods get busy → usage increases
* HPA scales Pods up (2 → 3 → 4)

---

# **9. After Load Reduces**

When you exit BusyBox:

* load-generator Pod auto-deletes
* No more traffic → Pod CPU/Memory drop
* HPA gradually scales down (4 → 3 → 2)

Kubernetes scales down slowly to avoid sudden outages.
