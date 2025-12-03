# EKS Cluster Autoscaler

## BEFORE EKS CLUSTER AUTOSCALER

### 1. Create Namespace

```bash
kubectl create ns prod
````

### 2. Sample Maven Web App Deployment + Service

Create a file `deploy.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mavenwebappdep
  namespace: prod
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mavenwebapp
  template:
    metadata:
      name: mavenwebapp
      labels:
        app: mavenwebapp
    spec:
      containers:
        - name: mavenwebapp
          image: kkeducation123456/maven-web-app:1.2
          ports:
            - containerPort: 8080
          resources:
            requests:
              memory: "512Mi"   # Minimum memory required per Pod
              cpu: "900m"       # Minimum CPU required per Pod
            limits:
              memory: "1Gi"     # Maximum memory allowed per Pod
              cpu: "1"          # Maximum CPU allowed per Pod
---
apiVersion: v1
kind: Service
metadata:
  name: mavenwebapp-svc
  namespace: prod
spec:
  type: LoadBalancer
  selector:
    app: mavenwebapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

Apply the manifest:

```bash
kubectl apply -f deploy.yaml
```

### 3. Verify Resources

```bash
# Check nodes
kubectl get nodes

# Check objects in prod namespace
kubectl get all -n prod
```

### 4. Scale Deployment to Simulate Resource Pressure

Increase replicas so that one Pod goes into `Pending` (no enough resources):

```bash
kubectl scale deployment mavenwebappdep -n prod --replicas=10
```

Now check:

```bash
kubectl get pods -n prod
```

You will see some Pods in `Pending` state because **cluster does not have enough CPU/Memory** to schedule all Pods.

> **Note:** Even if you defined min and max nodes in your Managed Node Group / ASG, new nodes **won’t** automatically come up **without** Cluster Autoscaler.
> That’s why we configure **EKS Cluster Autoscaler**.

---

## EKS CLUSTER AUTOSCALER

The Cluster Autoscaler in Amazon EKS automatically adjusts the size of your cluster by adding nodes when resources are insufficient for pending pods and removing nodes when they are underutilized.

---

## Setup Steps

> Reference:
> [https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md)

### Step 1: Create IAM Policy for Cluster Autoscaler

Create an IAM policy with the following JSON:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "autoscaling:DescribeAutoScalingGroups",
        "autoscaling:DescribeAutoScalingInstances",
        "autoscaling:DescribeLaunchConfigurations",
        "autoscaling:DescribeScalingActivities",
        "ec2:DescribeImages",
        "ec2:DescribeInstanceTypes",
        "ec2:DescribeLaunchTemplateVersions",
        "ec2:GetInstanceTypesFromInstanceRequirements",
        "eks:DescribeNodegroup"
      ],
      "Resource": ["*"]
    },
    {
      "Effect": "Allow",
      "Action": [
        "autoscaling:SetDesiredCapacity",
        "autoscaling:TerminateInstanceInAutoScalingGroup"
      ],
      "Resource": ["*"]
    }
  ]
}
```

**AWS Console steps:**

1. Go to **IAM → Policies → Create policy**.
2. Select **JSON** tab and paste the above policy.
3. Click **Next**.
4. Give a name, for example: `ClusterAutoScalerPolicy`.
5. Click **Create policy**.

---

### Step 2: Attach Policy to EKS Node IAM Role

1. Go to **IAM → Roles**.
2. Find and select your **EKS node group IAM role** (not the control plane role).
3. Go to **Permissions → Add permissions → Attach policies**.
4. Search and select `ClusterAutoScalerPolicy`.
5. Click **Add permissions**.

---

### Step 3: Install the Cluster Autoscaler in EKS

> Reference:
> [https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md)

Create a file `cluster-autoscaler.yaml` with the following content:

> ⚠️ **Important:**
> In `--node-group-auto-discovery` argument, **replace `my-demo-cluster` with your actual EKS cluster name**.

```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
  name: cluster-autoscaler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["events", "endpoints"]
    verbs: ["create", "patch"]
  - apiGroups: [""]
    resources: ["pods/eviction"]
    verbs: ["create"]
  - apiGroups: [""]
    resources: ["pods/status"]
    verbs: ["update"]
  - apiGroups: [""]
    resources: ["endpoints"]
    resourceNames: ["cluster-autoscaler"]
    verbs: ["get", "update"]
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["watch", "list", "get", "update"]
  - apiGroups: [""]
    resources: ["namespaces", "pods", "services", "replicationcontrollers", "persistentvolumeclaims", "persistentvolumes"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["extensions"]
    resources: ["replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["policy"]
    resources: ["poddisruptionbudgets"]
    verbs: ["watch", "list"]
  - apiGroups: ["apps"]
    resources: ["statefulsets", "replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses", "csinodes", "csidrivers", "csistoragecapacities"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["batch", "extensions"]
    resources: ["jobs"]
    verbs: ["get", "list", "watch", "patch"]
  - apiGroups: ["coordination.k8s.io"]
    resources: ["leases"]
    verbs: ["create"]
  - apiGroups: ["coordination.k8s.io"]
    resources: ["leases"]
    resourceNames: ["cluster-autoscaler"]
    verbs: ["get", "update"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["create", "list", "watch"]
  - apiGroups: [""]
    resources: ["configmaps"]
    resourceNames: ["cluster-autoscaler-status", "cluster-autoscaler-priority-expander"]
    verbs: ["delete", "get", "update", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    app: cluster-autoscaler
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8085"
    spec:
      priorityClassName: system-cluster-critical
      securityContext:
        runAsNonRoot: true
        runAsUser: 65534
        fsGroup: 65534
        seccompProfile:
          type: RuntimeDefault
      serviceAccountName: cluster-autoscaler
      containers:
        - name: cluster-autoscaler
          image: registry.k8s.io/autoscaling/cluster-autoscaler:v1.26.2
          imagePullPolicy: Always
          resources:
            limits:
              cpu: 100m
              memory: 600Mi
            requests:
              cpu: 100m
              memory: 600Mi
          command:
            - ./cluster-autoscaler
            - --v=4
            - --stderrthreshold=info
            - --cloud-provider=aws
            - --skip-nodes-with-local-storage=false
            - --expander=least-waste
            - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/my-demo-cluster  # change cluster name here
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop: ["ALL"]
          volumeMounts:
            - name: ssl-certs
              mountPath: /etc/ssl/certs/ca-certificates.crt
              readOnly: true
      volumes:
        - name: ssl-certs
          hostPath:
            # For Amazon Linux, path is:
            path: /etc/ssl/certs/ca-bundle.crt
```

> 🔁 **Replace `my-demo-cluster`** with your actual EKS cluster name
> Example: `k8s.io/cluster-autoscaler/prod-eks-cluster`

Apply manifest:

```bash
kubectl apply -f cluster-autoscaler.yaml
```

---

## Testing the Autoscaler

1. **Check autoscaler Pod:**

```bash
kubectl get pods -n kube-system | grep cluster-autoscaler
```

2. **Scale your Deployment again to create Pending Pods:**

```bash
kubectl scale deployment mavenwebappdep -n prod --replicas=15
kubectl get pods -n prod
```

3. **Watch nodes scaling up:**

```bash
kubectl get nodes -w
```

4. **Check autoscaler logs:**

```bash
kubectl logs -f -n kube-system deploy/cluster-autoscaler
```

You should see logs showing that new nodes are being added to handle Pending Pods.

---
