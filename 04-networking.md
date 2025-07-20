# Kubernetes Networking Setup Guide

## Pre-Network Verification
Before installing CNI, verify cluster components:

```bash
# Check core namespaces
kubectl get namespaces

# Verify system pods
kubectl get pods -n kube-system -o wide
```
**Expected Output (Before CNI):**
```text
NAME                                    READY   STATUS    RESTARTS   AGE     IP       NODE     
coredns-558bd4d5db-8xlfz                0/1     Pending   0          3m      <none>   control-plane
kube-proxy-fhx29                        1/1     Running   0          3m      <none>   control-plane
```
🔍 CoreDNS will show `Pending` status until CNI is installed

### `Calico CNI Installation`

**Why Calico?**

- 🔒 Built-in network policies for security

- 🚀 High-performance eBPF dataplane

- ☁️ Cloud/on-prem hybrid support

- 📡 BGP/IP-in-IP networking flexibility

- 🔧 Production-grade stability

#### Install Calico
```bash
# Install latest stable version
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
```
**Verify Installation**
``bash
watch kubectl get pods -n kube-system
```
**Expected Progression:**
```text
NAME                                       READY   STATUS              RESTARTS   AGE
calico-kube-controllers-7c579b9bcd-dm9r4   0/1     ContainerCreating   0          15s
calico-node-gmcqf                          0/1     Init:0/1            0          15s
```
→ Within 1-2 minutes →
```text
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-7c579b9bcd-dm9r4   1/1     Running   0          2m
calico-node-gmcqf                          1/1     Running   0          2m
coredns-558bd4d5db-8xlfz                   1/1     Running   0          5m
```
**Post-Install Checks**
```bash
# Verify all system pods
kubectl get pods -n kube-system

# Check network components
kubectl get daemonsets -n kube-system
```
#### Single-Node Cluster Configuration

Remove taint to allow workload scheduling:
```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```
⚠️ Only required for single-node clusters. Production clusters should maintain taints.

#### Network Verification Tests

**1. DNS Resolution Test**
```bash
kubectl run dns-test --image=busybox:1.28 --restart=Never --rm -it -- \
  sh -c "nslookup kubernetes.default"
```
**2. Pod-to-Pod Connectivity**
```bash
kubectl run ping-test --image=alpine --restart=Never -- \
  sh -c "ping -c 4 google.com"
```
**3. Network Policy Validation**
```bash
# Create test policy
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
EOF

# Verify policy enforcement
kubectl describe networkpolicy test-deny-all
```
**Troubleshooting Guide**

**Common Issues & Solutions**

## Common Issues & Solutions

| Symptom                         | Solution                                                                 |
|---------------------------------|--------------------------------------------------------------------------|
| CoreDNS pending                 | Verify Calico installation logs using `kubectl logs -n kube-system -l k8s-app=calico-node` |
| Calico pods stuck in Init      | Check required kernel modules: `lsmod | grep ip_tables`                  |
| Inter-pod connectivity fails   | Validate IP pools with: `kubectl get ippools.crd.projectcalico.org`     |
| Network policies not enforced  | Confirm Calico is in eBPF mode: `calicoctl get felixconfiguration default -o yaml` |

**Diagnostic Commands**
```bash
# Check Calico node status
kubectl exec -it <calico-pod> -n kube-system -- calico-node -status

# View CNI configuration
cat /etc/cni/net.d/10-calico.conflist

# Inspect kubelet logs
journalctl -u kubelet -f | grep -i cni
```
**Log Inspection**
```bash
# Calico controller logs
kubectl logs -n kube-system -l k8s-app=calico-kube-controllers

# Calico node logs
kubectl logs -n kube-system -l k8s-app=calico-node
```




