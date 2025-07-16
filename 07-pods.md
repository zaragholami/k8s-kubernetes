## Pod Deployment in Kubernetes

#### 📌 1. Deploy a Pod Directly with Command

This is quick and doesn't require a YAML file. Use it for simple or temporary testing.

✅ Example: Run an Nginx Pod

```
kubectl run nginx --image=nginx
```
**🔎 Verify:**
```
kubectl get pods
```
**🧹 Delete:**
```
kubectl delete pod nginx
```
This creates a basic pod. It does not support advanced configurations like volumes, probes, or replica sets.

#### 📄 2. Deploy a Pod Using a YAML File
Recommended for production or reusable configuration. You define all settings declaratively.

✅ Example: `nginx-pod.yaml`
```
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
**✅ Apply:**
```
kubectl apply -f nginx-pod.yaml
```
**🔎 Verify:**
```
kubectl get pods
```
**🧹 Delete:**
```
kubectl delete -f nginx-pod.yaml
```
**YAML Sections Explained**

**🔢 apiVersion**
```
apiVersion: v1
```
**What it does**: Specifies the Kubernetes API version used for the object.

**Why it's important**: Ensures compatibility with your cluster.

✅ For Pods, always use `v1`

**`kind`**
```
kind: Pod
```
**What it does**: Defines the type of object you're creating.

**Other kinds**: `Deployment`, `Service`, `ConfigMap`, etc

**`metadata`**
```
metadata:
  name: nginx
  labels:
    app: nginx
    type: backend
```
**name**: The unique name for the pod in the namespace

**labels**: Key-value pairs used to group/select objects (e.g., for services or monitoring)

**`spec`**
```
spec:
  containers:
```
**What it does:** Defines the desired state of the pod.

**Main part:** The list of containers to run

**`containers`**
```
containers:
- name: nginx-container
  image: nginx:latest
```
**name:** A unique name for the container inside the pod

**image:** The container image to run (from Docker Hub or a registry)

**`ports`**
```
ports:
- containerPort: 80
```
**containerPort**: The port your application listens on inside the container.

This doesn’t expose it externally; it just documents it for internal communication (like with Services)

**Optional Additions (Advanced)**

you can extend your pod YAML with:

| Feature       | Field Example                              |
| ------------- | ------------------------------------------ |
| Environment   | `env:` – define variables                  |
| Volume Mounts | `volumeMounts:` + `volumes:`               |
| Probes        | `livenessProbe:`, `readinessProbe:`        |
| Resources     | `resources:` – requests/limits for CPU/mem |
| Command       | `command:` and `args:`                     |


#### 📝 Summary Table

| Method     | Command Format                    | Use Case                          |
| ---------- | --------------------------------- | --------------------------------- |
| Direct CLI | `kubectl run nginx --image=nginx` | Quick testing                     |
| YAML File  | `kubectl apply -f nginx-pod.yaml` | Reusable, version-controlled code |

**✅ 3.Create Pods Using Controllers (Recommended in Production)**

While you can deploy a single pod directly, it's better to use Kubernetes controllers to manage pod lifecycle automatically. Controllers offer:

-✓ Self-healing (restart on failure)

-✓ Scaling

-✓ Rolling updates

-✓ Resource management

**🔁 3-1. Deployment (Web Apps, APIs)**

Use when you want multiple **replicas** and **auto-recovery**.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
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
        image: nginx:latest
        ports:
        - containerPort: 80
```
**Commands:**
```
kubectl apply -f nginx-deployment.yaml
kubectl get deployments
kubectl get pods
kubectl delete -f nginx-deployment.yaml
```
**🧹 3-2. Job (Run Once)**

Use for one-time tasks, scripts, or batch jobs
```
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-job
spec:
  template:
    spec:
      containers:
      - name: hello
        image: busybox
        command: ["echo", "Hello from job"]
      restartPolicy: Never
```
**💾 3-3. StatefulSet (Databases)**

Use for apps that need persistent storage and stable identity, like databases.

**🌍 3-4. DaemonSet (One Pod Per Node)**

Use for node-level agents, like log collectors or monitoring tools

**📋 Summary Table**

| Controller  | Use Case                  | Pod Management | Scaling | Auto Healing |
| ----------- | ------------------------- | -------------- | ------- | ------------ |
| Deployment  | Web apps, APIs            | ✅              | ✅       | ✅            |
| Job         | One-time scripts          | ✅              | ❌       | N/A          |
| StatefulSet | Databases, message queues | ✅              | ✅       | ✅            |
| DaemonSet   | Node-level agents         | ✅ (1/node)     | Auto    | ✅            |

#### 🧱 What Is a Static Pod?

**Node-local**: A static Pod is defined by a `YAML/JSON` manifest placed directly on a specific node (typically under `/etc/kubernetes/manifests/`) and is exclusively managed by that node’s kubelet—not by the API server or scheduler 

**Control-plane independence**: Ideal for bootstrapping or running essential components even if the API server is down—commonly used for critical system Pods like `kube-apiserver`, `etcd`, `kube-scheduler`, etc. 

#### 🔧 When a Pod is running & you change its YAML

| Desired Action                        | Command                               | Behavior                                       |
| ------------------------------------- | ------------------------------------- | ---------------------------------------------- |
| Initial creation                      | `kubectl create -f pod.yaml`          | Creates only if not already present            |
| Small change (e.g., image, resources) | `kubectl apply -f pod.yaml`           | Incrementally updates only modified fields     |
| Big change requiring full replacement | `kubectl replace -f pod.yaml`         | Overwrites entire spec; must include full YAML |
| Force recreate (wipe & recreate)      | `kubectl replace --force -f pod.yaml` | Deletes and recreates Pod (use cautiously)     |

