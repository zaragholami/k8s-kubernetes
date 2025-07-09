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

#### Use Cases for Reading or Modifying etcd in Kubernetes

**1. 🔥 Disaster Recovery / Backup**

Why? If the API server is down or corrupted and you can't use kubectl, etcd is the only source of truth.
What you do: Access etcd directly to recover data or create a snapshot of the current state.
Example:
```
etcdctl snapshot save /backup/etcd-backup.db
```
**2. 🧪 Debugging Corrupted Cluster State**

Why? Sometimes resources are stuck or broken (e.g., stale finalizers, orphaned pods), and can't be deleted through kubectl.
What you do: Read or delete the raw keys manually from etcd.

⚠️ Dangerous if used incorrectly.

**3. 🔒 Investigating Security & RBAC Configuration**

Why? You need to inspect or verify critical RBAC bindings (like cluster-admin) or service account tokens.
What you do: Read keys like:
```
/registry/clusterrolebindings/
/registry/serviceaccounts/
```
**4. ⚙️ Advanced Automation & Auditing**

Why? You want to track or audit raw Kubernetes resource changes at a lower level than kubectl.
What you do: Query etcd with a prefix to find patterns or perform consistency checks.

**❌ When You Should Not Use etcd Directly**

**Routine operations**: Always use kubectl, kustomize, helm, etc.

**Creating or updating resources**: Use Kubernetes APIs, not etcd writes.

**Editing YAML files**: Never write manually into etcd unless you're restoring.



