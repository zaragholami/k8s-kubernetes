#### Pre-Network Setup: Verify Kubernetes Components

Before installing the networking layer (CNI), verify the core Kubernetes components:

```
# View all namespaces
kubectl get namespaces

# View pods in the default namespace
kubectl get pods

# View system-level pods (control plane components)
kubectl get pods -n kube-system

# View detailed pod info (includes Node, IP, etc.)
kubectl get pods -n kube-system -o wide

```
🔍 Example Output Before Networking Is Applied

```
NAME                                    READY   STATUS    RESTARTS   AGE     IP       NODE            NOMINATED NODE   READINESS GATES
coredns-558bd4d5db-8xlfz                0/1     Pending   0          3m      <none>   control-plane   <none>           <none>
kube-proxy-fhx29                        1/1     Running   0          3m      <none>   control-plane   <none>           <none>
```

📌 coredns will be in Pending state until a network plugin like Calico is installed.

✅ `kubectl apply`
Purpose: Creates or updates Kubernetes resources based on a YAML/JSON file.
Use Case: When you want to deploy or update apps, configs, services, etc.

❌ `kubectl delete`
Purpose: Deletes Kubernetes resources defined in a YAML file or by name.
Use Case: When you want to remove resources from the cluster

#### Installing Calico (Recommended CNI)

To enable pod-to-pod networking, install Calico in Manager Node :
```
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
```
 Why Calico?
 
Calico is chosen because:

🔒 Security-first design: Built-in support for Kubernetes NetworkPolicies and eBPF.

🚀 High performance: Uses efficient Linux kernel routing (or eBPF mode) for fast data paths.

📡 Flexible networking: Supports both BGP and IP-in-IP modes.

☁️ Cloud & on-prem friendly: Works seamlessly in bare metal and cloud environments.

🔧 Active community and long-term support with extensive documentation.

Calico is widely adopted in production-grade Kubernetes clusters due to its balance of simplicity, performance, and security.

🔄 Example: Immediately After Applying Calico

```
kubectl get pods -n kube-system
```
```
NAME                                       READY   STATUS              RESTARTS   AGE
calico-kube-controllers-7c579b9bcd-dm9r4   0/1     ContainerCreating   0          15s
calico-node-gmcqf                          0/1     ContainerCreating   0          15s
coredns-558bd4d5db-8xlfz                   0/1     Pending             0          3m
coredns-558bd4d5db-lrkdn                   0/1     Pending             0          3m
etcd-control-plane                         1/1     Running             0          3m
kube-apiserver-control-plane               1/1     Running             0          3m
kube-controller-manager-control-plane      1/1     Running             0          3m
kube-proxy-btx8c                           1/1     Running             0          3m
kube-scheduler-control-plane               1/1     Running             0          3m
```
⏳ What to Expect
ContainerCreating: The node is pulling images, setting up volumes, networking, etc.

This should transition to Running within 1–2 minutes if everything is correct.

✅ To Monitor Progress:

You can run:
```
watch kubectl get pods -n kube-system
```
or check logs/events if it takes too long:
```
kubectl describe pod <pod-name> -n kube-system
```
Remove Taint from Control Plane Node (for Single-Node Clusters)
```
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```
🔍 Why Do This?

In single-node clusters, the control-plane node is tainted to prevent regular workloads from being scheduled on it. If you don’t remove the taint, any pod that isn’t “tolerating” it will stay in Pending state

Example Problem:
```
$ kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
nginx       0/1     Pending   0          1m
```




