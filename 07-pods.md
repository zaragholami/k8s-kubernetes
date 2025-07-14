## Pod Deployment in Kubernetes

You can deploy pods in two ways:

#### 📌 1. Deploy a Pod Directly with Command

This is quick and doesn't require a YAML file. Use it for simple or temporary testing.

✅ Example: Run an Nginx Pod

```
kubectl run nginx --image=nginx
```
🔎 Verify:
```
kubectl get pods
```
🧹 Delete:
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
Apply:
```
kubectl apply -f nginx-pod.yaml
```
🔎 Verify:
```
kubectl get pods
```
🧹 Delete:
```
kubectl delete -f nginx-pod.yaml
```
#### 📝 Summary Table

| Method     | Command Format                    | Use Case                          |
| ---------- | --------------------------------- | --------------------------------- |
| Direct CLI | `kubectl run nginx --image=nginx` | Quick testing                     |
| YAML File  | `kubectl apply -f nginx-pod.yaml` | Reusable, version-controlled code |
