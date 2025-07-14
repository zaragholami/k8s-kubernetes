## Pod Deployment in Kubernetes

You can deploy pods in two ways:

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
