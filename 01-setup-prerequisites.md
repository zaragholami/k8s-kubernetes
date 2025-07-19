### Minimum System Requirements per Kubernetes Node
| Component        | Minimum (Small cluster/test)                          | Recommended (Production)               |
| ---------------- | ----------------------------------------------------- | -------------------------------------- |
| **CPU**          | 1 CPU core                                            | 2+ CPU cores                           |
| **Memory (RAM)** | 2 GB RAM                                              | 4+ GB RAM                              |
| **Disk Storage** | 10 GB free disk                                       | 20+ GB free SSD recommended            |
| **Network**      | Reliable network connectivity                         | Low latency, reliable, and redundant   |
| **OS**           | Linux (Ubuntu 20.04+, CentOS 7+, Debian 10+, RHEL 7+) | Same + kernel optimized for containers |
| **Swap**         | Disabled (must disable swap)                          | Disabled (must disable swap)           |

### Software Requirements

OS: Linux distributions (Ubuntu(22.04 LTS	Jammy-20.04 LTS	Focal), CentOS, Debian, RHEL)

Docker or container runtime: Containerd, CRI-O, Docker (deprecated in latest Kubernetes)

kubeadm, kubelet, kubectl: Match Kubernetes version

Disable swap: swapoff -a and remove or comment swap entries from /etc/fstab

Firewall: Open required ports (e.g., 6443 for API server, 10250 for kubelet, 2379-2380 for etcd)

### Pre-installation Requirements

🖥️ Set Hostname

Set unique and descriptive hostnames for each node to simplify cluster management.

 Apply Hostname
```
hostnamectl set-hostname Manager
hostnamectl set-hostname Worker1
```
Then verify ;
```
bash -l;
hostnamectl
```
🔧 Set TimeZone
```
timedatectl set-timezone Asia/Tehran;timedatectl
```

🔧 Ensure Unique MAC Address & UUID 
```
ip link            # Check MAC address
cat /sys/class/dmi/id/product_uuid    # Check UUID
```
🔧 Disable swap
Kubernetes requires swap to be disabled.
```
sudo sed -i '/swap/s/^\//\#\//g' /etc/fstab
sudo swapoff -a
```
```
sudo reboot

```
Verify Swap is Disabled
```
free -m
```
🔧 disable and stop ufw
```
service ufw stop && service ufw disable 
```
🔧 or Configure UFW

✅ Manager (Control Plane) Node
```
sudo ufw allow 6443/tcp    # Kubernetes API server
sudo ufw allow 2379:2380/tcp  # etcd server client API
sudo ufw allow 10250/tcp   # Kubelet API
sudo ufw allow 10259/tcp   # kube-scheduler
sudo ufw allow 10257/tcp   # kube-controller-manager
sudo ufw allow from <worker-node-ip>/32 to any port 10250 proto tcp  # Allow workers to connect to kubelet
```
✅ Worker Node
```
sudo ufw allow 10250/tcp   # Kubelet API
sudo ufw allow 30000:32767/tcp  # NodePort range for services
sudo ufw allow 10255/tcp   # Read-only kubelet API (optional; often disabled)
sudo ufw allow 6783/tcp    # Optional: Weave/Flannel CNI (if using)
sudo ufw allow from <manager-node-ip>/32 to any port 10250 proto tcp  # Manager access to kubelet
```

Update System
```
sudo apt update && sudo apt upgrade -y
```
🔧 Set Static DNS

To ensure consistent name resolution, configure your system to use a static DNS server

📌 Recommended DNS: electro,openDNS,shecan,....

for set permanently :
```
rm -rf /etc/resolv.conf;echo "nameserver 78.157.42.100  # (Electro)" | sudo tee -a /etc/resolv.conf
```

#### Install containerd (Kubernetes Container Runtime)
Step 1: Install containerd
```
cd /usr/local/
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
mv containerd.service /usr/lib/systemd/system/;\
systemctl daemon-reload;\
systemctl enable --now containerd.service
```
Step 3: Install runc (for namespaces & cgroups)
```
wget https://github.com/opencontainers/runc/releases/download/v1.1.12/runc.amd64
```
```
install -m 755 runc.amd64 /usr/local/sbin/runc;\
runc --version
```
Step 4: Configure containerd
```
mkdir -p /etc/containerd;\
cd /etc/containerd;\
containerd config default | sudo tee /etc/containerd/config.toml;\
systemctl daemon-reload;\
systemctl restart containerd
```
#### Install CNI Plugins (Container Network Interface)
```
wget https://github.com/containernetworking/plugins/releases/download/v1.4.0/cni-plugins-linux-amd64-v1.4.0.tgz
```
```
mkdir -p /opt/cni/bin;\
tar -xzvf cni-plugins-linux-amd64-v1.4.0.tgz -C /opt/cni/bin
```
#### enable Cgroup Driver
```toml
# /etc/containerd/config.toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
SystemdCgroup = true
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
SystemdCgroup = true
```
```
systemctl restart containerd
```
#### Forwarding IPV4 and letting iptables see traffic
Load br_netfilter module
```
modprobe br_netfilter;\
echo 1 > /proc/sys/net/ipv4/ip_forward
```
Make it persistent:
```
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf;
sysctl -p
```
