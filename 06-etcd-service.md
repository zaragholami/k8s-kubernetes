#### 📁 /etc/kubernetes/manifests/ – What Is It?
<small>This directory is used by the kubelet to auto-manage static Pods on the node (especially the control plane components like etcd, kube-apiserver, etc.).<small>
#### ✅ Key Points:
<small>Files in `/etc/kubernetes/manifests/` are YAML manifests (like Deployments, Pods).<small>

<small>The kubelet watches this directory.<small>

<small>If you place a valid pod manifest here, kubelet will automatically create and manage the pod.<small>

<small>Used in **kubeadm-based** Kubernetes setups.<small>

#### Typical files found there on a control plane node:
```
/etc/kubernetes/manifests/
├── etcd.yaml
├── kube-apiserver.yaml
├── kube-controller-manager.yaml
└── kube-scheduler.yaml
```
<small>So if you want to modify etcd configuration (e.g., add new flags), you'd edit etcd.yaml, and kubelet will restart the etcd Pod with the new config.<small>
#### 🔍 Accessing etcd via kubectl
<small>You can access etcd running as a static pod like this:<small>
```
kubectl -n kube-system exec -it etcd-<NODE_NAME> -- etcdctl get / --prefix --keys-only
```
*Replace:*
*etcd-<NODE_NAME> → The actual pod name, usually something like etcd-mnf if your hostname is mnf.*
