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

#### 🔍 Reading etcd Content in Kubernetes

Kubernetes stores its entire cluster state in etcd, including resources like Pods, ConfigMaps, Secrets, RBAC, and more. You can directly read this data using `etcdctl` from inside the etcd Pod (usually running as a static Pod on the control plane node).
Prerequisites:
You must have access to the control plane node.
etcd is secured with TLS, so you need to provide certificate paths.

`kubectl` access with sufficient privileges.
Example: Get list of keys in etcd
```
kubectl -n kube-system exec -it etcd-<NODE_NAME> -- \
etcdctl \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
get / --prefix --keys-only
```
📝 ***What this does***:

`--prefix` gets all keys under `/`.

`--keys-only` shows only key names (no values).

Helps explore etcd structure.
Read specific value (e.g. cluster-admin binding)
```
kubectl -n kube-system exec -it etcd-<NODE_NAME> -- \
etcdctl \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
get /registry/clusterrolebindings/cluster-admin
```
📝 ***Notes***:

The output is stored in Protobuf format and not human-readable.

This is the raw form of what Kubernetes uses internally.

**Common etcd Key Prefixes in Kubernetes :**
| Resource Type       | etcd Key Prefix                  |
| ------------------- | -------------------------------- |
| Pods                | `/registry/pods/`                |
| ConfigMaps          | `/registry/configmaps/`          |
| Secrets             | `/registry/secrets/`             |
| ClusterRoles        | `/registry/clusterroles/`        |
| ClusterRoleBindings | `/registry/clusterrolebindings/` |


