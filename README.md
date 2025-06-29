# Kubernetes
## Kubernetes Architecture: Manager & Worker Node Explained
📌 Your Setup

*1x Manager Node (a.k.a. Control Plane)

*1x Worker Node

🔧 Main Components:

1- Manager Node (Control Plane) :

Component	Description :

This node controls and manages the entire Kubernetes cluster.

* kube-apiserver	Central API for communication (kubectl talks to this).

* etcd	Key-value store for cluster state/config.

* kube-scheduler	Assigns new Pods to available nodes.

* kube-controller-manager	Handles replication, node monitoring, etc.

* containerd	Container runtime (runs containers).

✅ Responsibilities:

* Accepts and validates cluster commands (via kubectl).

* Schedules pods to run on Worker nodes.

* Monitors the overall cluster health.

* Stores and updates cluster config/state.

2 - Worker Node :

Component	Description :

 This node runs the actual applications (your containers).

kubelet	Talks to the manager; makes sure pods are running.

kube-proxy	Manages networking for Pods and Services.

containerd	Actually runs the containers.

✅ Responsibilities:

Runs pods assigned by the manager.

Reports node & pod status to the manager.

Manages networking and container lifecycle.
