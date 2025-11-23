# 📘 Liveness Probe & Readiness Probe in Kubernetes

## Overview

Liveness and Readiness Probes are mechanisms in Kubernetes used to check the health and status of containers inside the Pod.

These probes ensure:

* Containers stay healthy
* Unhealthy containers get restarted
* Only ready containers receive traffic

---

## 🔍 What is a Liveness Probe?

A **Liveness Probe** checks *whether the application inside the container is still running properly*.

Example situations:

* Memory leak
* High CPU usage
* Deadlock
* Application becomes unresponsive

👉 If the **liveness probe fails**, Kubernetes **restarts the container** automatically.

---

## 🔍 What is a Readiness Probe?

A **Readiness Probe** checks *whether the container is ready to receive traffic*.

Important points:

* Pods failing readiness will **NOT receive traffic**
* The Pod stays **in Service**, but **removed from load balancer endpoints**
* Useful during application startup or slow initialization

👉 If readiness fails, **traffic stops** until probe passes again.

---

# 🧪 Example: Deployment with Liveness & Readiness Probes

### `live_readyness.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: javawebappdep
  namespace: test
spec:
  replicas: 2
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: javawebapp
  template:
    metadata:
      name: javawebapp
      labels:
        app: javawebapp
    spec:
      containers:
      - name: javawebapp
        image: kkeducation123456/maven-web-app:1.2
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /maven-web-application
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /maven-web-application
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 5
          failureThreshold: 3
---
apiVersion: v1
kind: Service
metadata:
  name: javawebappsvc
  namespace: test
spec:
  type: NodePort
  selector:
    app: javawebapp
  ports:
  - port: 80
    targetPort: 8080
```

### Commands

```bash
kubectl apply -f live_readyness.yaml
kubectl get all -n test
```

---

# 🔁 Same YAML, but Probe Paths Changed (Incorrect Path)

Here, we intentionally use a **wrong URL path**:

```yaml
livenessProbe:
  httpGet:
    path: /maven-web-application123
    port: 8080

readinessProbe:
  httpGet:
    path: /maven-web-application123
    port: 8080
```

### Apply & Check

```bash
kubectl apply -f live_readyness.yaml
```
kubectl get all -n test
kubectl describe pod <pod-name> -n test
```

### Expected Events

```
Warning  Unhealthy  Readiness probe failed: HTTP probe failed with statuscode: 404
Warning  Unhealthy  Liveness probe failed: HTTP probe failed with statuscode: 404
Normal   Killing    Container failed liveness probe, will be restarted
```

## ✔ Reason

Kubernetes checks `/maven-web-application123` but your application exposes:

```
/maven-web-application
```

Because the paths do not match → **404** → probe fails → Pod restarts.

---

# 📝 Summary

| Probe Type          | Purpose                                  | What Happens When It Fails?                            |
| ------------------- | ---------------------------------------- | ------------------------------------------------------ |
| **Liveness Probe**  | Checks if container is alive             | Kubernetes **restarts** the container                  |
| **Readiness Probe** | Checks if container is ready for traffic | Pod is **removed from Service endpoints** (no traffic) |

---

# 🎯 Real-Time Access

If your service is NodePort:

```
http://<NodeIP>:<NodePort>/maven-web-application
```
