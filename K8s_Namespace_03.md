# Kubernetes Namespace

---

## 1. What is a Kubernetes Namespace?

- A **Namespace** in Kubernetes is a **logical partition of the cluster**.  
- It is used to **organize, isolate, and manage resources**.  
- Within a namespace, you can deploy resources such as:
  - Pods
  - Services
  - Deployments
  - ConfigMaps
  - Secrets
- It acts like a **virtual cluster inside the main cluster**.  
- Namespaces divide a cluster into **multiple virtual workspaces**.  
- This makes it easier to manage resources for different teams, projects, or environments (e.g., Dev, Test, Prod).  

---

## 2. What Namespaces Provide

1. **Isolation**  
   Keeps resources separate so different teams or environments don’t affect each other.  

2. **Access Control (RBAC – Role-Based Access Control)**  
   - Restrict access to certain namespaces using RBAC policies.  
   - Only authorized users/teams can view or manage resources in that namespace.  

3. **Resource Quotas**  
   Set CPU and memory limits for each namespace to prevent overuse.  

4. **Environment Segregation**  
   Create separate namespaces for different environments:  
   - `dev` – Development  
   - `staging` – Testing / Pre-production  
   - `prod` – Production  

   Each environment gets its own namespace, keeping workloads and settings isolated.  

---

## 3. Types of Namespaces in Kubernetes

### 1. `default`
- Used if no namespace is specified when creating a resource.  
- All resources go here by default.  

### 2. `kube-system`
- Contains Kubernetes system components and internal services.  
- Don’t modify unless required.  

### 3. `kube-public`
- Visible to everyone (even without login).  
- Mostly used for public cluster information.  

### 4. `kube-node-lease`
- Manages node lease objects.  
- Helps track node health and status.  
- Reduces the load on the Kubernetes control plane.  

### 5. **User-defined**
- Namespaces you create for your own use.  
- Example: `dev`, `test`, `prod`.  

---

## 4. How to Work with Namespaces

### Creating a Namespace

**Using YAML:**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
  labels:
    name: my-namespace
````

Apply the file:

```bash
kubectl apply -f namespace.yaml
```

**Using Command Line:**

```bash
kubectl create namespace test
```

---

### 1. Listing Namespaces

```bash
kubectl get namespaces
# OR
kubectl get ns
```

### 2. Viewing Namespace Details

```bash
kubectl describe ns my-namespace
```

### 3. Deleting a Namespace

⚠️ Warning: Deletes all resources inside the namespace.

```bash
kubectl delete namespace my-namespace
```

---

## 5. Namespace Resource Quotas

* Limit resources (CPU, memory, pods) in a namespace.
* Example YAML to limit pods to 10:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
  namespace: my-namespace
spec:
  hard:
    pods: "10"
```

Apply the quota:

```bash
kubectl apply -f resource-quota.yaml
```

Check quota:

```bash
kubectl describe ns my-namespace
```

---

### Creating a Pod in the Namespace to Test Quota

```bash
kubectl run nginx --image=nginx --port=80 -n my-namespace
```

Check quota again:

```bash
kubectl describe ns my-namespace
```

Edit the quota:

```bash
kubectl edit quota my-resource-quota -n my-namespace
```
**Example output:**

```
Pods: 1/10 — 1 pod used, 9 left.
```

---

## 6. Switching Namespaces in kubectl Context

Set your current namespace context so commands default to it:

```bash
kubectl config set-context --current --namespace=my-namespace
```
---
