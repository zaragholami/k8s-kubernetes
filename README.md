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


```mermaid
flowchart TB
    subgraph CLUSTER["CLUSTER"]
        subgraph CONTROL_PLANE["CONTROL PLANE"]
            subgraph Cloud_Components[" "]
                ccm[cloud-controller-manager]
                cp[cloud-player]
            end
        end
        
        subgraph Node_app_server["Node app server"]
            s[scheduler]
            ks[kube-scheduler]
            cm[controller-manager]
            kcm[kube-controller-manager]
        end
        
        subgraph Node1["Node 1"]
            k1[kubelet]
            kp1[kube-proxy]
            p1_1[pod]
            p1_2[pod]
            p1_3[pod]
            cri1[CRI]
        end
        
        subgraph Node2["Node 2"]
            k2[kubelet]
            kp2[kube-proxy]
            p2_1[pod]
            cri2[CRI]
        end
    end
    
    classDef controlPlane fill:#f9f,stroke:#333,stroke-width:2px;
    classDef node fill:#eef,stroke:#333,stroke-width:2px;
    classDef component fill:#fff,stroke:#aaa;
    
    class CONTROL_PLANE controlPlane;
    class Node1,Node2,Node_app_server node;
    class ccm,cp,s,ks,cm,kcm,k1,kp1,p1_1,p1_2,p1_3,cri1,k2,kp2,p2_1,cri2 component;
```


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

**Key Components Explained:**

1.Authentication: Verifies user credentials

2.Authorization: Checks permissions (RBAC/ABAC)

3.Admission Control: Validates/modifies requests

4.Validation: Ensures object schema correctness

5.Persistence: Stores state in etcd/SQLite

6.Control Plane: Handles controllers/scheduling

7.kubelet: Creates containers and reports status

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






