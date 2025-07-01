
### Install kubelet,kubeadm,kubectl
Add Kubernetes Repository & Update
```
apt update
apt install -y apt-transport-https ca-certificates curl
#### Add Kubernetes GPG key
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
## Add Kubernetes APT repository
```
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
apt update
```
## Install Kubernetes Components
Install specific version (recommended for stability)
```
sudo apt install -y kubelet=1.29.* kubeadm=1.29.* kubectl=1.29.*
# OR install latest version
# sudo apt install -y kubelet kubeadm kubectl
```
## Prevent Auto-Upgrades (Optional but Recommended)
```
apt-mark hold kubelet kubeadm kubectl
```
## Verify Installation
```
kubectl version --client --output=yaml
kubeadm version
```


