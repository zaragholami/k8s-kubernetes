## Kubernetes Architecture: Manager & Worker Node Explained
📌 Your Setup
1x Manager Node (a.k.a. Control Plane)
1x Worker Node

Manager Node (Control Plane)
This node controls and manages the entire Kubernetes cluster.

🔧 Main Components:
Component	Description
kube-apiserver	Central API for communication (kubectl talks to this).
etcd	Key-value store for cluster state/config.
kube-scheduler	Assigns new Pods to available nodes.
kube-controller-manager	Handles replication, node monitoring, etc.
containerd	Container runtime (runs containers).

✅ Responsibilities:
Accepts and validates cluster commands (via kubectl).

Schedules pods to run on Worker nodes.

Monitors the overall cluster health.

Stores and updates cluster config/state.

🛠️ Worker Node
This node runs the actual applications (your containers).

🔧 Main Components:
Component	Description
kubelet	Talks to the manager; makes sure pods are running.
kube-proxy	Manages networking for Pods and Services.
containerd	Actually runs the containers.

✅ Responsibilities:
Runs pods assigned by the manager.

Reports node & pod status to the manager.

Manages networking and container lifecycle.


## Pre-installation Requirements
Ensure Unique MAC Address & UUID 
```
ip link            # Check MAC address
cat /sys/class/dmi/id/product_uuid    # Check UUID
```
Disable swap
Kubernetes requires swap to be disabled.
```
sudo sed -i.bak '/ swap / s/^/#/' /etc/fstab
sudo swapoff -a
sudo reboot

```
Verify Swap is Disabled
```
free -m
```
Update System
```
sudo apt update && sudo apt upgrade -y

```
Set Static DNS
Use DNS 78.157.42.100 (Electro).

## Install containerd (Kubernetes Container Runtime)
Step 1: Install containerd
```
cd /usr/local/
wget https://github.com/containerd/containerd/releases/download/v1.6.30/containerd-1.6.30-linux-amd64.tar.gz
tar xzf containerd-1.6.30-linux-amd64.tar.gz
```
Step 2: Setup systemd service for containerd
```
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
mv containerd.service /usr/lib/systemd/system/
systemctl daemon-reload
systemctl enable --now containerd.service
```
Step 3: Install runc (for namespaces & cgroups)
```
wget https://github.com/opencontainers/runc/releases/download/v1.1.12/runc.amd64
install -m 755 runc.amd64 /usr/local/sbin/runc
runc --version
```
Step 4: Configure containerd
```
mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl daemon-reload
systemctl restart containerd
```
## Install CNI Plugins (Container Network Interface)
```
wget https://github.com/containernetworking/plugins/releases/download/v1.4.0/cni-plugins-linux-amd64-v1.4.0.tgz
mkdir -p /opt/cni/bin
tar -xzvf cni-plugins-linux-amd64-v1.4.0.tgz -C /opt/cni/bin
```


