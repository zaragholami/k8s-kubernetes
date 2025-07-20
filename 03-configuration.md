# Kubernetes Cluster Configuration Guide

## Manager Node Setup

### kubeadm Init Options
| Option                       | CLI Flag                        | Config File Path (YAML)             | Example Value      |
|------------------------------|---------------------------------|-------------------------------------|--------------------|
| API Server Advertise Address | `--apiserver-advertise-address` | Node network configuration          | `192.168.0.100`    |
| Kubernetes Version           | `--kubernetes-version`          | `ClusterConfiguration.kubernetesVersion` | `v1.29.13`         |
| Pod Network CIDR             | `--pod-network-cidr`            | `ClusterConfiguration.networking.podSubnet` | `10.10.0.0/16`     |

**Key Notes**:
- `--apiserver-advertise-address`: Manager node's IP accessible by workers
- `--pod-network-cidr`: Must match your CNI plugin's requirements
- `--ignore-preflight-errors=NumCPU,Mem`: Bypasses resource checks for lab environments
- `--v=5`: Enables verbose debugging output

### Initialize Control Plane

```bash
kubeadm init \
  --kubernetes-version=v1.29.13 \
  --pod-network-cidr=10.10.0.0/16 \
  --apiserver-advertise-address=192.168.0.100 \
  --v=5
```
#### Configure kubectl Access

After successful initialization:
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Optional: Persist configuration
echo 'export KUBECONFIG=/etc/kubernetes/admin.conf' | sudo tee -a /etc/profile
source /etc/profile
```
#### Verify Manager Node
```bash
kubectl get nodes
```
Example output:

```bash
NAME      STATUS     ROLES           AGE   VERSION
manager   NotReady   control-plane   58m   v1.29.13
```
#### Worker Node Setup

**Join Worker to Cluster**

1.Get join command from manager node:
```bash
kubeadm token create --print-join-command
```
Example output:
```text
kubeadm join 192.168.0.100:6443 --token abcdef.0123456789abcdef \
  --discovery-token-ca-cert-hash sha256:0123456789abcdef...1234
```
2.Run join command on worker node

**Verify Node Join**

On manager node:
```bash
watch kubectl get nodes  # Monitor join progress
```
Example output:
```text
NAME      STATUS     ROLES           AGE   VERSION
manager   NotReady   control-plane   58m   v1.29.13
worker1   NotReady   <none>          57s   v1.29.13
```
**Label Worker Node**
```bash
kubectl label node worker1 node-role.kubernetes.io/worker=worker1
```
Verify:
```bash
kubectl get nodes
```
```text
NAME      STATUS   ROLES           AGE   VERSION
manager   Ready    control-plane   20m   v1.29.3
worker1   Ready    worker1         18m   v1.29.3
```
#### Cluster Verification

**Check System Pods**
```bash
kubectl get pods -n kube-system
```
Expected once CNI is installed:
```text
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-5f6546844f-2j6kd   1/1     Running   0          5m
calico-node-abcde                          1/1     Running   0          5m
coredns-76f75df574-g8wwj                   1/1     Running   0          145m
etcd-manager                               1/1     Running   0          145m
```
**Verify Cluster Health**
```bash
kubectl cluster-info
```
```text
Kubernetes control plane is running at https://192.168.0.100:6443
CoreDNS is running at https://192.168.0.100:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```
**Check Component Status**
```bash
kubectl get componentstatuses
```

