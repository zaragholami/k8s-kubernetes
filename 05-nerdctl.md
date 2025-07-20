# nerdctl: Containerd CLI for Kubernetes

## What is nerdctl?
`nerdctl` is a Docker-compatible CLI for containerd, the container runtime used by Kubernetes. It provides familiar container management commands while interacting directly with containerd instead of Docker.

**Key Features**:
- 🐳 Docker-compatible commands (`run`, `exec`, `ps`, etc.)
- ☸️ Native Kubernetes integration via `k8s.io` namespace
- 🔒 Secure alternative to Docker in production environments
- ⚡ Lightweight with no additional daemon required

**Common Use Cases**:
- Managing containers in k3s clusters
- Troubleshooting Kubernetes pods/containers
- Container operations in Docker-free environments
- CI/CD pipelines using containerd runtime

## Installation

### Download and Install
```bash
# Download latest stable release
curl -sLO https://github.com/containerd/nerdctl/releases/download/v1.7.5/nerdctl-full-1.7.5-linux-amd64.tar.gz

# Extract to /usr/local/bin
sudo tar Cxzvf /usr/local/bin nerdctl-full-1.7.5-linux-amd64.tar.gz

# Verify installation
nerdctl --version
```
**Verify Installation**
```bash
nerdctl version
```
Output:
```text
Client:
 Version:	v1.7.5
 Git commit:	1a2b3c4d5e6f7g8h9i0j
```
#### Kubernetes Integration

**Essential Commands**
```bash
# List all Kubernetes containers
nerdctl -n k8s.io ps -a

# Access shell in Kubernetes container
nerdctl -n k8s.io exec -it <container-id> sh

# List images used by Kubernetes
nerdctl -n k8s.io images

# Inspect container details
nerdctl -n k8s.io inspect <container-id>
```
**Permanent Configuration**

Add to `~/.bashrc` to avoid repeating `-n k8s.io`:
```bash
echo 'export CONTAINERD_ADDRESS=/run/containerd/containerd.sock' >> ~/.bashrc
echo 'export CONTAINERD_NAMESPACE=k8s.io' >> ~/.bashrc
source ~/.bashrc
```
After configuration:
```bash
# Simplified commands
nerdctl ps -a          # Show Kubernetes containers
nerdctl images         # Show Kubernetes images
```
**Common Workflows**

1. Troubleshoot Failing Pod
```bash
# Find container ID
nerdctl ps -a | grep "my-pod"

# Access logs
nerdctl logs <container-id>

# Execute diagnostic tools
nerdctl exec -it <container-id> sh -c "nslookup kubernetes.default"
```
2. Inspect Container Resources
```bash
nerdctl inspect <container-id> | jq '.[0].Spec.linux.resources'
```
3. Copy Files to/from Container
```bash
# Copy from container
nerdctl cp <container-id>:/path/to/file ./local-copy

# Copy to container
nerdctl cp ./local-file <container-id>:/destination/
```
**Command Comparison**

| Docker Command     | nerdctl Equivalent |
|--------------------|--------------------|
| `docker ps`        | `nerdctl ps`       |
| `docker exec`      | `nerdctl exec`     |
| `docker logs`      | `nerdctl logs`     |
| `docker inspect`   | `nerdctl inspect`  |
| `docker run`       | `nerdctl run`      |

**Advanced Usage**

Build Images
```bash
nerdctl build -t my-image:latest .
```
Manage Networks
```
# Create network
nerdctl network create my-net

# List networks
nerdctl network ls
```
Run Container with GPU
```bash
nerdctl run -it --gpus all nvidia/cuda:11.0-base nvidia-smi
```
**Troubleshooting**

Common Issues

Permission denied errors

Solution: Add user to `containerd` group:
```bash
sudo usermod -aG containerd $USER
newgrp containerd
```
Connection refused

Verify containerd socket:
```bash
ls -l /run/containerd/containerd.sock
```
Debugging Tips
```bash
# Show detailed container info
nerdctl inspect <container-id> | jq .

# View containerd namespaces
nerdctl namespace ls

# Check containerd version
nerdctl info | grep containerd
```

