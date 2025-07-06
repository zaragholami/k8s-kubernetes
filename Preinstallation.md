## Minimum System Requirements per Kubernetes Node
| Component        | Minimum (Small cluster/test)                          | Recommended (Production)               |
| ---------------- | ----------------------------------------------------- | -------------------------------------- |
| **CPU**          | 1 CPU core                                            | 2+ CPU cores                           |
| **Memory (RAM)** | 2 GB RAM                                              | 4+ GB RAM                              |
| **Disk Storage** | 10 GB free disk                                       | 20+ GB free SSD recommended            |
| **Network**      | Reliable network connectivity                         | Low latency, reliable, and redundant   |
| **OS**           | Linux (Ubuntu 20.04+, CentOS 7+, Debian 10+, RHEL 7+) | Same + kernel optimized for containers |
| **Swap**         | Disabled (must disable swap)                          | Disabled (must disable swap)           |

## Software Requirements

OS: Linux distributions (Ubuntu, CentOS, Debian, RHEL)

Docker or container runtime: Containerd, CRI-O, Docker (deprecated in latest Kubernetes)

kubeadm, kubelet, kubectl: Match Kubernetes version

Disable swap: swapoff -a and remove or comment swap entries from /etc/fstab

Firewall: Open required ports (e.g., 6443 for API server, 10250 for kubelet, 2379-2380 for etcd)

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
```
```
sudo reboot

```
Verify Swap is Disabled
```
free -m
```
Update System
```
disable and stop ufw
```
service ufw stop && service ufw disable 
```
```
sudo apt update && sudo apt upgrade -y

```
Set Static DNS
Use DNS 78.157.42.100 (Electro).

## Install containerd (Kubernetes Container Runtime)
Step 1: Install containerd
```
cd /usr/local/
```
```
wget https://github.com/containerd/containerd/releases/download/v1.6.30/containerd-1.6.30-linux-amd64.tar.gz
```
```
tar xzf containerd-1.6.30-linux-amd64.tar.gz
```
Step 2: Setup systemd service for containerd
```
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
```
```
mv containerd.service /usr/lib/systemd/system/
```
```
systemctl daemon-reload
```
```
systemctl enable --now containerd.service
```
Step 3: Install runc (for namespaces & cgroups)
```
wget https://github.com/opencontainers/runc/releases/download/v1.1.12/runc.amd64
```
```
install -m 755 runc.amd64 /usr/local/sbin/runc
```
```
runc --version
```
Step 4: Configure containerd
```
mkdir -p /etc/containerd
```
```
cd /etc/containerd
```
```
containerd config default | sudo tee /etc/containerd/config.toml
```
```
sudo systemctl daemon-reload
```
```
systemctl restart containerd
```
## Install CNI Plugins (Container Network Interface)
```
wget https://github.com/containernetworking/plugins/releases/download/v1.4.0/cni-plugins-linux-amd64-v1.4.0.tgz
```
```
mkdir -p /opt/cni/bin
```
```
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
