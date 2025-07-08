#### ✅ What is nerdctl?
nerdctl is a Docker-compatible CLI for containerd — a container runtime used by Kubernetes and lightweight distributions like k3s. It lets you manage containers, images, and networks — just like Docker — but using containerd directly
It’s commonly used when working with:
k3s clusters
Minimal, secure environments where Docker is not installed
`containerd` as the runtime

#### ✅ Installation
```
# Go to /usr/local/bin or another directory in your $PATH
cd /usr/local/bin

# Download the latest nerdctl release (adjust version if needed)
wget https://github.com/containerd/nerdctl/releases/download/v1.7.5/nerdctl-full-1.7.5-linux-amd64.tar.gz

# Extract the archive
tar zxvf nerdctl-full-1.7.5-linux-amd64.tar.gz

# Move nerdctl to /usr/local/bin if not already there
cp nerdctl /usr/local/bin/

# Optional: verify
nerdctl --version
```

#### Basic and Important nerdctl Commands

-n k8s.io option in nerdctl refers to the containerd namespace that Kubernetes uses to manage its containers.

List all Kubernetes containers:
```
nerdctl -n k8s.io ps -a
```
Exec into a running Kubernetes container:
```
nerdctl -n k8s.io exec -it <container-id> sh
```
View images used by Kubernetes:
```
nerdctl -n k8s.io images
```

#### Documentation

GitHub Repository:

🔗 https://github.com/containerd/nerdctl

Docs & Examples:

🔗 https://github.com/containerd/nerdctl#readme





