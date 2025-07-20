### `/etc/kubernetes/manifests/` & etcd Configuration Guide

#### 📁 `/etc/kubernetes/manifests/` Directory

#### Purpose
- Static Pod management directory for kubelet
- Automatically runs/manages control plane components (etcd, API server, etc.)
- Primarily used in **kubeadm** setups

#### ✅ Key Facts
- **Format**: YAML manifests (Pod/Deployment specs)
- **Automation**: Kubelet watches directory → creates/manages Pods
- **Control Plane Files**:

/etc/kubernetes/manifests/

├── etcd.yaml

├── kube-apiserver.yaml

├── kube-controller-manager.yaml

└── kube-scheduler.yaml


> ℹ️ Edit these files to reconfigure components (e.g., add etcd flags). Kubelet auto-restarts Pods.

---

#### 🔧 etcd Configuration (kubeadm)

#### Location

`/etc/kubernetes/manifests/etcd.yaml`

**Sample Configuration**

```yaml
spec:
containers:
- command:
  - etcd
  - --name=etcd-manager
  - --data-dir=/var/lib/etcd
  - --listen-client-urls=https://127.0.0.1:2379
  # ... (TLS flags below)
  - --cert-file=/etc/kubernetes/pki/etcd/server.crt
  - --key-file=/etc/kubernetes/pki/etcd/server.key
  - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
```

#### TLS Certificates

| Path                                  | Purpose         |
|---------------------------------------|------------------|
| `/etc/kubernetes/pki/etcd/ca.crt`     | CA Certificate   |
| `/etc/kubernetes/pki/etcd/server.crt` | Server Cert      |
| `/etc/kubernetes/pki/etcd/server.key` | Server Key       |
| `/etc/kubernetes/pki/etcd/peer.crt`   | Peer Cert        |
| `/etc/kubernetes/pki/etcd/peer.key`   | Peer Key         |

**🔍 Accessing etcd Data**

Kubernetes stores all cluster state in etcd. Use `etcdctl` from the etcd Pod:

**1. List Keys (Prefix-Based)**
```bash
kubectl -n kube-system exec etcd-<NODE_NAME> -- \
etcdctl --endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
get / --prefix --keys-only
```
**2. Read Specific Resource**
```bash
# Example: ClusterRoleBinding 'cluster-admin'
kubectl -n kube-system exec etcd-<NODE_NAME> -- \
etcdctl ... get /registry/clusterrolebindings/cluster-admin
```
📌 Common etcd Key Prefixes

| Resource               | etcd Path                             |
|------------------------|----------------------------------------|
| Pods                   | `/registry/pods/`                      |
| Secrets                | `/registry/secrets/`                   |
| ConfigMaps             | `/registry/configmaps/`                |
| ClusterRoles           | `/registry/clusterroles/`              |
| ServiceAccounts        | `/registry/serviceaccounts/`           |
| ClusterRoleBindings    | `/registry/clusterrolebindings/`       |

> ⚠️ Output is in Protobuf format (not human-readable)


⚠️ **Use Cases & Warnings**

✅ Valid Use Cases

**1.Disaster Recovery**

Snapshot etcd when API server is down:
```bash
etcdctl snapshot save backup.db
```
**2.Debugging Corrupted State**

Remove stuck finalizers or orphaned resources

**3.Security Audits**

Inspect raw RBAC bindings and service accounts

**4.Advanced Auditing**

Track resource changes at low level

**❌ Prohibited Uses**

- **Routine operations** (always use kubectl/Kubernetes API)

- **Modifying resources** directly (high corruption risk)

- **Creating/Updating resources** (bypasses API server validation)

- **Editing YAMLs** except for recovery purposes

**🔒 Critical Security Note**

Direct etcd access requires control plane node privileges.

Never modify etcd directly unless absolutely necessary - always prefer Kubernetes API for daily operations.
