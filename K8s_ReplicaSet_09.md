# ReplicaSet
---
## 1. What Is a ReplicaSet?

A ReplicaSet (RS) is a Kubernetes object, makes sure the desired number of Pods are always running in a cluster.

### How It Works:

- You define `spec.replicas` (eg. 3).
- RS ensures exactly 3 Pods with matching labels are running.
- If a Pod crashes or is deleted → RS creates a new one.
- If extra Pods appear as Pods → RS deletes them.

### Features of ReplicaSet

- **Self-healing:** Crashed or deleted Pods are recreated.
- **Prevents extra Pods:** If more Pods exist than required, ReplicaSet deletes them.
- **Label-based management:** Pods are tracked by labels, not by name.
- **Advanced selectors:** Can use `matchLabels` (simple) or `matchExpressions` (advanced).
- **Deployment integration:** Deployments use ReplicaSets internally to manage rolling updates.

---

## 2. Desired vs Observed State

This is the heart of Kubernetes controllers.

- **Desired state:** How many Pods you want (eg. 3).
- **Observed state:** How many Pods are currently running with the correct labels.

RS constantly compares these two and adjusts the number of Pods to match your desired state.

ReplicaSet continuously compares these states:

- If the observed count is less than the desired count, it creates new Pods until both match.
- If the observed count is greater than the desired count, it deletes the extra Pods.
- If the observed count equals the desired count, it does nothing.

<img width="968" height="694" alt="image" src="https://github.com/user-attachments/assets/07e836b2-43af-456b-94f6-a9f26759f26e" />

---

## 3. How ReplicaSet Uses Label Selectors

ReplicaSet doesn’t care about Pod names. It uses labels to decide which Pods to manage.

### Types of selectors:

#### 1. matchLabels (Equality-based)

Simple key-value match.

```yaml
selector:
  matchLabels:
    app: myapp
````

* This ReplicaSet will only control Pods with the label `app=myapp`.
* If a Pod doesn’t have this label, RS ignores it.

<img width="904" height="382" alt="image" src="https://github.com/user-attachments/assets/7fced8f8-e8be-42f0-bf03-41d51403c19d" />

#### 2. MatchExpressions (Set-based, Advanced)

Pods often have multiple labels:

```yaml
labels:
  env: prod
  tier: frontend
  app: myapp
```

**Why use matchExpressions?**

For complex conditions, we use `matchExpressions`. It lets you define rules like:

* Label value must be in a set.
* Label value must not be in a set.
* Label must exist / not exist.

**Example:**

```yaml
EX2 set based
==============


apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: javawebapprs
  namespace: test-ns
spec:
  replicas: 3
  selector:
    matchExpressions:
      - key: app
        operator: In
        values:
          - javawebapp1
          - javawebapp2
          - javawebapp
  template:
    metadata:
      labels:
        app: javawebapp1  # or javawebapp2 or javawebapp, make sure this matches your selector
    spec:
      containers:
        - name: javawebapp
          image: kkeducationb2/java-webapp:1.1
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: javawebappsvc
  namespace: test-ns
spec:
  type: NodePort
  selector:
    app: javawebapp1  # or javawebapp2 or javawebapp, make sure this matches your ReplicaSet's pods
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

## 4. Operators in matchExpressions

### 1. In

```yaml
- key: app
  operator: In
  values:
    - nginx
    - apache
```

* Pod must have the label `app`, and its value must be either `nginx` or `apache`.

### 2. NotIn

```yaml
- key: app
  operator: NotIn
  values:
    - mysql
```

* Pod must have the label `app`, but its value must not be `mysql`.

### 3. Exists

```yaml
- key: release
  operator: Exists
```

* Pod must have a label with the key `release`.
* Value does not matter as long as the key exists.

### 4. DoesNotExist

```yaml
- key: debug
  operator: DoesNotExist
```

* Pod must not have the label `debug`.

**Summary:**

* `matchLabels` → simple key=value.
* `matchExpressions` → advanced operators (`In`, `NotIn`, `Exists`, `DoesNotExist`).
* Both can be used together for precise Pod selection.
* This gives fine control over which Pods a ReplicaSet manages.

---

## 5. Difference Between RC and RS Selector

**ReplicationController (RC)**

* Selector is optional. If a selector isn’t given, RC will automatically use the labels from the Pod template.

**ReplicaSet (RS)**

* Selector is mandatory. A selector must be defined explicitly; otherwise RS won’t know which Pods to manage.
* Pod template labels must match the selector; otherwise RS will create Pods but won’t track them.

---

## 6. ReplicaSet with Service Example

**YAML Example:**

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata: 
  name: javawebapprs
  namespace: test-ns
spec:
  replicas: 3
  selector:
    matchLabels:
      app: javawebapp
  template:
    metadata: 
      name: javawebapprcpod
      labels:
        app: javawebapp
    spec: 
      containers:
      - name: javawebapprccon
        image: satyamolleti4599/maven-web-app:1.0.0
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: javawebapprs-service
  namespace: test-ns
spec:
  type: NodePort
  selector:
    app: javawebapp
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30007
```

### Understanding spec in ReplicaSet

A ReplicaSet YAML has two main spec sections:

1. **Outer spec → belongs to the ReplicaSet**

* `replicas: 3` → number of Pods RS should run.
* `selector` → how RS finds/manages Pods (based on labels).
  Without selector → RS won’t know which Pods to manage.

2. **Inner spec → belongs to the Pod template**

* `template` → defines how Pods are created.
* Includes labels, containers, image, ports, etc.
* Labels here must match the RS selector.

---

### Why Selector is Crucial in RS

```yaml
selector:
  matchLabels:
    app: javawebapp
```

* RS manages only Pods with label `app=javawebapp`.
* If labels don’t match → RS won’t manage those Pods (even if it created them).

---

### Common Mistake: Missing Labels in Pod Template

❌ Wrong Example (no labels):

```yaml
template:
  metadata:
    # Labels missing here
```

* RS will create Pods but won’t recognize them later.
* Reason: RS uses labels to track Pods.

### Correct Way: Add Matching Labels

```yaml
template:
  metadata:
    labels:
      app: javawebapp
```

* Now RS knows these Pods belong to it.
* Ensures Pods are tracked properly.
* Keeps Pod count at the desired level.
* Enables self-healing (recreates Pod if it crashes).

---

## 7. Working with ReplicaSets

### (1) Create & Inspect

```bash
kubectl apply -f rs.yaml
kubectl get rs
kubectl get all
```

* `kubectl get all` → shows all common resources in the cluster.

**Explanation:**

* 2 Pods are running.
* Pod names start with `javawebapprs` (ReplicaSet name).
* The suffixes (-abc12, -def34) are random strings generated by Kubernetes.
* These suffixes ensure Pod names are unique, even if created by the same ReplicaSet.
* Both Pods belong to the same ReplicaSet but have different names.

---

### (2) Pod Deletion & Auto-Healing

**Steps:**

1. Check Pods before delete:

```bash
kubectl get pods
```

2. Delete one Pod:

```bash
kubectl delete pod <podName>
```

3. Check again after a few seconds:

```bash
kubectl get pods
```

**Explanation in simple words:**

* ReplicaSet always maintains DESIRED = 2 Pods.
* When you delete one Pod (`javawebapprs-abc12`), RS notices only 1 Pod is left.
* ReplicaSet notices Pod count dropped and creates a new Pod.
* So it immediately creates a new Pod (`javawebapprs-dpkzn`) with a new random suffix.

---

### (3) Check Pod Ownership

```bash
kubectl describe pod <pod-name>
```

or filter ownership:

```bash
kubectl describe pod <pod-name> | grep -i "Controlled By"
```

---

## 8. ReplicaSet Update with New Image Version

### 1. Initial Setup:

* ReplicaSet created with `image: kkeducation12345/spring-app:1.0.0`
* RS launched 2 Pods with this image.
* Everything is working fine.

### 2. Update Attempt

* Update the container image in `rs.yaml`:

```yaml
containers:
  - name: javawebapp
    image: kkeducation12345/spring-app:1.0.1
```

Apply the changes:

```bash
kubectl apply -f rs.yaml
```

Output:

```
replicaset.apps/javawebapprs configured
```

* RS definition updated in Kubernetes (etcd store).

### 3. Check pods again

```bash
kubectl get pods
```

* Still the same old Pods are running.
* No new Pods created because RS only makes sure Pod count = desired (2).
* It does not restart existing Pods when the template changes.

### 4. Verify Pod Image

```bash
kubectl describe pod <podName>
```

* Pod is still running with: Image: 1.0.0
* Existing Pods keep running with the old image.

### 5. Why Didn’t the Pods Update?

* ReplicaSet’s job = keep the number of Pods equal to the desired count.
* It does NOT track Pod template changes (like new image versions).
* If Pod count is already correct, RS does nothing even if the template has been updated.
* ReplicaSet is not designed for rolling updates or version management.

---

## 9. How to Update Pods in a ReplicaSet

### 1. Manual way

* Delete old Pods → RS will create new ones using the updated template.

```bash
kubectl delete pod <pod-name>
```

* New Pods will come with the new image.

### 2. Scaling trick

* Increase replicas → newly created Pods will use the new image.

```bash
kubectl scale rs javawebapprs --replicas=3
```

* Then scale back to original.

### 3. Recreate RS

* Delete and re-apply the ReplicaSet with the new image.

### 4. Best practice

* Use a Deployment instead of RS for updates.
* Deployment handles rolling updates, rollbacks, and version history automatically.

---

## 10. Summary

* ReplicaSet = modern replacement for ReplicationController.
* Maintains the desired number of Pods at all times.
* Works by comparing desired vs observed state in a control loop.
* Uses label selectors (`matchLabels` + `matchExpressions`) to manage Pods.
* Provides self-healing, scaling, adoption of orphan Pods.
* In real-world production, always managed via Deployments.

```
