## Kubernetes Architecture: Manager & Worker Node

#### 1️⃣ Manager Node (Control Plane)

----------------------------

This node controls and manages the entire Kubernetes cluster.

| Component                 | Description                                               |
| ------------------------- | --------------------------------------------------------- |
| `kube-apiserver`          | Central API server for communication (kubectl uses this). |
| `etcd`                    | Key-value store for cluster state/configuration.          |
| `kube-scheduler`          | Assigns new Pods to available nodes.                      |
| `kube-controller-manager` | Handles replication, node health, and other controllers.  |
| `containerd`              | Container runtime that actually runs the containers.      |

##### ✅ Responsibilities

Accepts and validates cluster commands (kubectl)

Schedules Pods to Worker nodes

Monitors cluster health

Stores and updates cluster state/config in etcd
#### 2️⃣ Worker Node

------------------------

This node runs your actual applications (containers).

| Component    | Description                                                       |
| ------------ | ----------------------------------------------------------------- |
| `kubelet`    | Talks to the Manager node; ensures containers (pods) are running. |
| `kube-proxy` | Manages networking for Pods and Services.                         |
| `containerd` | Executes and manages containers.                                  |

##### ✅ Responsibilities

Runs Pods assigned by the Manager

Reports node and pod status to the Control Plane

Manages networking and container lifecycle

![Kubernetes Architecture](k8s-architecture.PNG)

------------------------------------------------------


### 🔄 Kubernetes API Server Request Sequence

```mermaid
sequenceDiagram
    participant User as User (kubectl/UI/API)
    participant API_Server as API Server
    participant Auth as Auth Services
    participant Backend as Backend (etcd/SQLite)
    participant Control as Control Plane
    participant Kubelet as kubelet

    User->>API_Server: API Request
    API_Server->>Auth: 1. Authenticate (token/cert/webhook)
    API_Server->>Auth: 2. Authorize (RBAC/ABAC)
    API_Server->>Auth: 3. Admission Control
    API_Server->>API_Server: 4. Validate & Parse Object
    API_Server->>Backend: 5. Persist Object
    API_Server->>Control: 6. Notify Controllers
    alt Scheduling Needed
        Control->>Control: Scheduler: Assign Pod to Node
        Control->>API_Server: Update Pod Status
        API_Server->>Kubelet: 7. Notify Assigned Pod
        Kubelet->>Kubelet: Create Pod/Containers
        Kubelet->>API_Server: Report Pod Status
    else Direct Action
        Control->>Control: Controller Logic
    end
```

-----------------------------------------------

#### 🔄 Responsibilities Summary:

```mermaid
flowchart LR
    kubelet[kubelet] --> Runs[Runs Assigned Pods]
    kubelet --> Reports[Reports Node/Pod Status]
    kubelet --> Manages[Manages Networking]
    kubelet --> Lifecycle[Manages Container Lifecycle]
```

📘 ***Documentation***

For complete Kubernetes guides and reference materials, visit the [official Kubernetes documentation](https://kubernetes.io/docs/).






