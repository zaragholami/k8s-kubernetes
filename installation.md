## Pre-installation Requirements

Set hostname

manager node --> mnf

worker node --> wnf

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
## enable Cgroup Driver
```
vi /etc/containerd/config.toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
SystemdCgroup = true
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
SystemdCgroup = true
```
```
systemctl restart containerd
```
## Forwarding IPV4 and letting iptables see traffic
Load br_netfilter module
```
modprobe br_netfilter
```
## Install kubelet,kubeadm,kubectl
Add Kubernetes Repository & Update
```
apt update
apt install -y apt-transport-https ca-certificates curl
# Add Kubernetes GPG key
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
# Add Kubernetes APT repository
```
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
apt update
```
# Install Kubernetes Components
Install specific version (recommended for stability)
```
sudo apt install -y kubelet=1.29.* kubeadm=1.29.* kubectl=1.29.*
# OR install latest version
# sudo apt install -y kubelet kubeadm kubectl
```
# Prevent Auto-Upgrades (Optional but Recommended)
```
apt-mark hold kubelet kubeadm kubectl
```
# Verify Installation
```
kubectl version --client --output=yaml
kubeadm version
```


