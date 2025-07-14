## Manager Node Configuration (Mnf)
---------------------------------

 #### kubeadm init Options – CLI vs Config File
 | Option                       | CLI Flag                        | Config File Path (YAML)                                        | Example Value    |
| ---------------------------- | ------------------------------- | -------------------------------------------------------------- | ---------------- |
| API Server Advertise Address | `--apiserver-advertise-address` | Not in YAML directly — set by the node’s network configuration | `192.168.0.100` |
| Kubernetes Version           | `--kubernetes-version`          | `ClusterConfiguration.kubernetesVersion`                       | `v1.29.13`       |
| Pod Network CIDR             | `--pod-network-cidr`            | `ClusterConfiguration.networking.podSubnet`                    | `10.10.0.0/16`   |

✅ Notes:

--apiserver-advertise-address: should be the manager node’s IP address.

--pod-network-cidr: internal network range for pods (used by the CNI plugin).

--ignore-preflight-errors=NumCPU,Mem: skips low CPU/RAM errors (helpful in test/lab environments).

--v=5: sets verbosity to level 5 (for debug output).

```
kubeadm init \
  --kubernetes-version=v1.29.13 \
  --pod-network-cidr=10.10.0.0/16 \
  --apiserver-advertise-address=192.168.0.100 \
  --v=5
```
After Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:
```
  mkdir -p $HOME/.kube;
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config;
  sudo chown $(id -u):$(id -g) $HOME/.kube/config;
```
```
export KUBECONFIG=/etc/kubernetes/admin.conf
```

```
kubectl get node
```
Example output :
```
kubectl get nodes
NAME   STATUS     ROLES           AGE   VERSION
mnf    NotReady   control-plane   58m   v1.29.13
```

-----------------------------------

## Worker Node Configuration (Wnf) :

✅ 1. Get the Worker Join Command

Right after running kubeadm init in Mnf, it usually prints a kubeadm join command like this:
```
kubeadm join <CONTROL_PLANE_IP>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```
If you didn’t save it, no problem — you can regenerate it:

🔁 Re-generate the join command in Manager Node :
```
kubeadm token create --print-join-command
```
Example output:
```
kubeadm join 192.168.0.100:6443 --token abcdef.0123456789abcdef \
  --discovery-token-ca-cert-hash sha256:0123456789abcdef...1234
```
and use this for join worker node to cluster

🔍 for verify in Mnf :

```
kubectl get nodes
```
Example Output;
```
NAME   STATUS     ROLES           AGE   VERSION
mnf    NotReady   control-plane   58m   v1.29.13
wnf    NotReady   <none>          57s   v1.29.13
```
Label Node wnf as worker1 in Mnager Node :

Helps identify or select the node in deployments, node selectors, and dashboards

```
kubectl label node wnf node-role.kubernetes.io/worker=worker1
```
then:
```
kubectl get nodes
```
Will output:
```
NAME   STATUS   ROLES           AGE   VERSION
mnf    Ready    control-plane   20m   v1.29.3
wnf    Ready    worker1          18m   v1.29.3
```
Then for see Pods Status ;
```
kubectl get pod -n kube-system  #-n or--namespace
```
Example output;

```
NAME                          READY   STATUS    RESTARTS   AGE
coredns-76f75df574-g8wwj      0/1     Pending   0          145m
coredns-76f75df574-t9jtf      0/1     Pending   0          145m
etcd-mnf                      1/1     Running   0          145m
kube-apiserver-mnf            1/1     Running   0          145m
kube-controller-manager-mnf   1/1     Running   0          145m
kube-proxy-25m7s              1/1     Running   0          88m
kube-proxy-6mps4              1/1     Running   0          145m
kube-scheduler-mnf            1/1     Running   0          145m
```
🔍 Verify Cluster Info

After setting up your Kubernetes cluster, you can verify its status using:
```
kubectl cluster-info
```
Example Output:
```
Kubernetes control plane is running at https://192.168.0.100:6443
CoreDNS is running at https://192.168.0.100:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```
This confirms that your control plane and DNS services are up and reachable





