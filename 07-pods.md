### Kubernetes Pod Deployment Guide

### 📌 Pod Deployment Methods

#### 1️⃣ Direct CLI Command (Quick Testing)
```bash
kubectl run nginx --image=nginx
```
**Pros:**

- Fast for temporary testing

- No YAML required

**Verification & Cleanup:**
```bash
kubectl get pods
kubectl delete pod nginx
```
⚠️ Limitations: No support for volumes, probes, or replica sets

#### 2️⃣ YAML Manifest (Recommended)
```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
    type: backend
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
```
**Operations:**
```bash
# Apply manifest
kubectl apply -f nginx-pod.yaml

# Verify
kubectl get pods -o wide

# Delete
kubectl delete -f nginx-pod.yaml
```
**YAML Structure Explained**


| Section      | Key           | Value/Example       | Purpose                             |
|--------------|---------------|---------------------|-------------------------------------|
| `apiVersion` | -             | `v1`                | Required for Pods; API compatibility |
| `kind`       | -             | `Pod`               | Defines object type (e.g., Pod, Deployment, Job) |
| `metadata`   | `name`        | `nginx`             | Unique Pod identifier               |
|              | `labels`      | `app: nginx`        | For grouping/selection              |
| `spec`       | `containers`  | List of containers  | Core Pod configuration              |
| `containers` | `name`        | `nginx-container`   | Container identifier                |
|              | `image`       | `nginx:latest`      | Container image source              |
|              | `ports`       | `containerPort: 80` | Exposed port (for internal use)     |

**Advanced YAML Features**
```yml
# Example additions:
env:
  - name: ENV_VAR
    value: "production"
    
volumeMounts:
  - name: config-vol
    mountPath: /etc/config

resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
    
livenessProbe:
  httpGet:
    path: /health
    port: 8080
```
#### 3️⃣ Controller-Based Deployment (Production)

**🔁 Deployment (Stateless Apps)**
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.23
        ports:
        - containerPort: 80
```
**Job (One-Time Tasks)**
```yml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-processor
spec:
  template:
    spec:
      containers:
      - name: processor
        image: python:3.9
        command: ["python", "process.py"]
      restartPolicy: OnFailure
```
**StatefulSet (Stateful Applications)**
```yml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: "mysql"
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
```
**DaemonSet (Node-Level Agents)**
```yml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector
spec:
  selector:
    matchLabels:
      name: log-collector
  template:
    metadata:
      labels:
        name: log-collector
    spec:
      containers:
      - name: fluentd
        image: fluentd:latest
```
**Controller Comparison**

#### Kubernetes Controllers Comparison

| Controller  | Use Case                 | Scaling | Storage | Network Identity | Node Affinity |
|-------------|--------------------------|---------|---------|------------------|----------------|
| Deployment  | Web apps, APIs           | ✅      | ❌      | ❌               | ❌             |
| Job         | Batch processing         | ❌      | ❌      | ❌               | ❌             |
| StatefulSet | Databases (MySQL, Redis) | ✅      | ✅      | ✅               | ❌             |
| DaemonSet   | Monitoring, logging      | Auto    | Optional| ❌               | ✅ (usually)   |
**Static Pods**

**Key Characteristics**

- **Location**: /etc/kubernetes/manifests/ (node-specific)

- **Management**: Managed directly by kubelet (not API server)

- **Use Case**: Critical system pods (etcd, kube-apiserver, etc.)

- **Behavior**: Auto-created when manifest appears, auto-deleted when removed

💡 Essential for control plane components that must run before API server is available

#### 🔄 Updating Running Pods

| Change Type        | Command                              | Behavior                                  |
|--------------------|---------------------------------------|-------------------------------------------|
| Initial Creation   | `kubectl create -f pod.yaml`          | Creates pod if it does not exist          |
| Minor Update       | `kubectl apply -f pod.yaml`           | Incremental update (e.g., image change)   |
| Major Replacement  | `kubectl replace -f pod.yaml`         | Full spec replacement                     |
| Force Recreate     | `kubectl replace --force -f pod.yaml` | Delete and recreate the pod (causes downtime) |

**⚠️ Critical Note**: Pods are immutable. Most updates require recreation. Use controllers for zero-downtime updates.

#### 📋 Deployment Method Comparison


| Method         | Command                            | Use Case              | Production Safe |
|----------------|-------------------------------------|------------------------|------------------|
| Direct CLI     | `kubectl run`                      | Quick testing          | ❌               |
| YAML Manifest  | `kubectl apply -f pod.yaml`        | Single-pod scenarios   | ⚠️ (Limited)     |
| Deployment     | `kubectl apply -f deploy.yaml`     | Web applications       | ✅               |
| StatefulSet    | `kubectl apply -f stateful.yaml`   | Stateful services      | ✅               |
| DaemonSet      | `kubectl apply -f daemonset.yaml`  | Node-level agents      | ✅               |


