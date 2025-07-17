
#### Install kubelet,kubeadm,kubectl
Add Kubernetes Repository & Update
```
apt update
```
```
apt install -y apt-transport-https ca-certificates curl
```
#### Add Kubernetes GPG key
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key\
| sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
#### Add Kubernetes APT repository
```
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /'\
| sudo tee /etc/apt/sources.list.d/kubernetes.list
```
```
apt update
```
#### Install Kubernetes Components
To install a stable version of kubectl, kubeadm, and kubelet, you should select a version that:

Is not the very latest, as those may still have bugs

Has received multiple patch updates

Matches across all three components (kubectl, kubeadm, kubelet)


check version with this command
```
apt list -a kubelet  # or kubectl ,kubeadm
```
then ;
```
sudo apt install -y kubelet=1.29.13-1.1 kubeadm=1.29.13-1.1 kubectl=1.29.13-1.1
```

#### Prevent Auto-Upgrades
```
apt-mark hold kubelet kubeadm kubectl
```
#### Verify Installation
```
kubectl version --client --output=yaml
```
```
kubeadm version
```
#### kubectl command auto-completion

To enable kubectl command auto-completion and make it persistent by adding it to your .bashrc, use the following commands:
```
# Enable bash completion (install if not already installed)
sudo apt install -y bash-completion

# Enable kubectl autocomplete for the current shell
source <(kubectl completion bash)

# Add kubectl autocomplete permanently to .bashrc
echo 'source <(kubectl completion bash)' >> ~/.bashrc

# Optionally, add alias and enable completion for it
echo 'alias k=kubectl' >> ~/.bashrc
echo 'complete -F __start_kubectl k' >> ~/.bashrc

# Reload .bashrc
source ~/.bashrc
```

