# Kubernetes Node Setup Guide

## Minimum System Requirements
| Component        | Minimum (Test/Small)        | Recommended (Production)     |
|------------------|-----------------------------|------------------------------|
| **CPU**          | 1 CPU core                  | 2+ CPU cores                 |
| **RAM**          | 2 GB                        | 4+ GB                        |
| **Storage**      | 10 GB free                  | 20+ GB SSD                   |
| **Network**      | Reliable connection         | Low latency + redundancy     |
| **OS**           | Linux (Ubuntu 20.04+, RHEL 7+, etc) | OS + optimized kernel        |
| **Swap**         | Disabled                    | Disabled                     |

### Pre-installation Setup

#### 1. Host Configuration

```bash
# Set unique hostnames
hostnamectl set-hostname <node-type>  # e.g., Manager or Worker1

# Set timezone
timedatectl set-timezone Asia/Tehran
timedatectl

# Verify unique identifiers
ip link show                 # Check MAC
cat /sys/class/dmi/id/product_uuid  # Check UUID
```
#### 2. System Optimization

```bash
# Disable swap permanently
sudo swapoff -a
sudo sed -i '/swap/s/^\(.*\)$/#\1/' /etc/fstab

# Disable UFW
sudo ufw disable

# Update system
sudo apt update && sudo apt upgrade -y
```
#### 3. Network Configuration
```bash
# Set static DNS
echo "nameserver 78.157.42.100" | sudo tee /etc/resolv.conf

# Enable IP forwarding
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```
### Container Runtime Installation (containerd)

#### 1. Install containerd
```bash
# Download and extract
wget https://github.com/containerd/containerd/releases/download/v1.6.30/containerd-1.6.30-linux-amd64.tar.gz -P /tmp
sudo tar Cxzvf /usr/local /tmp/containerd-1.6.30-linux-amd64.tar.gz

# Configure systemd service
sudo wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service -O /etc/systemd/system/containerd.service
sudo systemctl daemon-reload
sudo systemctl enable --now containerd
```
#### 2. Install runc
```bash
wget https://github.com/opencontainers/runc/releases/download/v1.1.12/runc.amd64 -P /tmp
sudo install -m 755 /tmp/runc.amd64 /usr/local/sbin/runc
```
#### 3. Install CNI Plugins
```bash
wget https://github.com/containernetworking/plugins/releases/download/v1.4.0/cni-plugins-linux-amd64-v1.4.0.tgz -P /tmp
sudo mkdir -p /opt/cni/bin
sudo tar -xzvf /tmp/cni-plugins-linux-amd64-v1.4.0.tgz -C /opt/cni/bin
```
#### 4. Configure containerd
```bash
# Generate default config
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# Enable systemd cgroup driver
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
```
### Firewall Configuration

#### Control Plane (Manager)
```bash
sudo ufw allow 6443/tcp     # API Server
sudo ufw allow 2379:2380/tcp # etcd
sudo ufw allow 10250/tcp    # Kubelet
sudo ufw allow 10259/tcp    # kube-scheduler
sudo ufw allow 10257/tcp    # kube-controller-manager
```
#### Worker Nodes
```bash
sudo ufw allow 10250/tcp
sudo ufw allow 30000:32767/tcp  # NodePort Services
```
#### Verification Commands
```bash
# Check services
systemctl status containerd

# Verify swap
free -h | grep Swap

# Confirm IP forwarding
cat /proc/sys/net/ipv4/ip_forward
```
#### Post-Installation Notes

- Reboot after initial setup: sudo reboot

- Kubernetes components (kubeadm, kubelet, kubectl) must be installed next
