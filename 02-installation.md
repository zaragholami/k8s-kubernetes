# Kubernetes Components Installation Guide

## Install kubelet, kubeadm, and kubectl

#### 1. Add Kubernetes Repository
```bash
# Update package index
sudo apt update

# Install prerequisite packages
sudo apt install -y apt-transport-https ca-certificates curl
```
#### 2. Add Kubernetes GPG Key
```bash
# Create keyrings directory
sudo mkdir -p /etc/apt/keyrings

# Download and install GPG key
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key |
sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
#### 3. Add Kubernetes APT Repository
```bash
# Add stable repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' |
sudo tee /etc/apt/sources.list.d/kubernetes.list

# Update package lists
sudo apt update
```
#### 4. Install Kubernetes Components
```bash
# Check available versions
apt list -a kubelet kubeadm kubectl

# Install specific stable version (example: 1.29.13)
sudo apt install -y \
    kubelet=1.29.13-1.1 \
    kubeadm=1.29.13-1.1 \
    kubectl=1.29.13-1.1

# Prevent automatic updates
sudo apt-mark hold kubelet kubeadm kubectl
```
#### 5. Verify Installation
```bash
# Verify kubectl installation
kubectl version --client --output=yaml

# Verify kubeadm version
kubeadm version

# Check kubelet service status
sudo systemctl status kubelet
```
#### 6. Configure kubectl Autocompletion
```bash
# Install bash-completion
sudo apt install -y bash-completion

# Enable autocompletion in current shell
source <(kubectl completion bash)

# Add to .bashrc for persistence
cat <<EOF >> ~/.bashrc
# kubectl autocompletion
source <(kubectl completion bash)

# kubectl alias with autocompletion
alias k=kubectl
complete -F __start_kubectl k
EOF

# Reload configuration
source ~/.bashrc
```
**Version Selection Best Practices**

When choosing versions:

1.**Avoid latest**: Skip x.x.0 releases (e.g., 1.30.0)

2.**Prefer patched versions**: Use versions with multiple patches (e.g., 1.29.3+)

3.**Consistency**: Use same version across all nodes

4.**Check compatibility**: Verify with Kubernetes release notes
