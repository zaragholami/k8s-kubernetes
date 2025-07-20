### Kubernetes Architecture: Control Plane & Worker Nodes

#### 🧠 Control Plane (Manager Node)
Controls and manages the entire Kubernetes cluster

| Component                 | Description                                               |
| ------------------------- | --------------------------------------------------------- |
| `kube-apiserver`          | Central API server for communication (kubectl uses this) |
| `etcd`                    | Key-value store for cluster state/configuration          |
| `kube-scheduler`          | Assigns new Pods to available nodes                      |
| `kube-controller-manager` | Handles replication, node health, and controllers        |
| `containerd`              | Container runtime that runs control plane containers     |

**✅ Control Plane Responsibilities**
- Accepts and validates cluster commands (kubectl)
- Schedules Pods to Worker nodes
- Monitors cluster health
- Stores/updates cluster state in etcd

**⚙️ Worker Nodes**

Run application workloads in containers

| Component    | Description                                                       |
| ------------ | ----------------------------------------------------------------- |
| `kubelet`    | Ensures containers (pods) are running as specified               |
| `kube-proxy` | Manages networking for Pods and Services                         |
| `containerd` | Executes and manages application containers                      |

#### ✅ Worker Node Responsibilities
- Runs Pods assigned by Control Plane
- Reports node/pod status to Control Plane
- Manages container lifecycle
- Handles pod networking

---

#### 📊 Cluster Architecture Overview
```mermaid
flowchart TB
    title["Kubernetes Cluster Architecture"]:::title
    
    subgraph CLUSTER["CLUSTER"]
        subgraph CONTROL_PLANE["Control Plane"]
            direction TB
            api["kube-apiserver"]:::important
            etcd["etcd"]:::important
            sched["kube-scheduler"]
            ctrl_mgr["kube-controller-manager"]
            cloud_ctrl["cloud-controller-manager"]
        end
        
        subgraph Worker1["Worker Node 1"]
            direction TB
            kubelet1["kubelet"]:::important
            kproxy1["kube-proxy"]
            pod1_1["pod"]
            pod1_2["pod"]
            pod1_3["pod"]
            containerd1["containerd"]
        end
        
        subgraph Worker2["Worker Node 2"]
            direction TB
            kubelet2["kubelet"]:::important
            kproxy2["kube-proxy"]
            pod2_1["pod"]
            containerd2["containerd"]
        end
    end
    
    %% Connections
    CONTROL_PLANE <-.-> Worker1
    CONTROL_PLANE <-.-> Worker2
    
    %% Styling
    classDef title fill:transparent,stroke:transparent,font-size:18px,font-weight:bold
    classDef controlPlane fill:#e6f7ff,stroke:#1890ff,stroke-width:2px;
    classDef worker fill:#f6ffed,stroke:#52c41a,stroke-width:2px;
    classDef important fill:#fff1f0,stroke:#f5222d,stroke-width:2px;
    classDef component fill:#fff,stroke:#ddd;
    
    class title title;
    class CONTROL_PLANE controlPlane;
    class Worker1,Worker2 worker;
    class api,etcd,kubelet1,kubelet2 important;
    class sched,ctrl_mgr,cloud_ctrl,kproxy1,kproxy2,pod1_1,pod1_2,pod1_3,containerd1,pod2_1,containerd2 component;
```
-------------------------------------------

#### 🔄 API Server Request Flow

```mermaid
sequenceDiagram
    autonumber
    participant User as User (kubectl/UI/API)
    participant API as API Server
    participant Auth as Auth Services
    participant etcd as etcd
    participant Scheduler
    participant Kubelet
    
    User->>API: ① API Request
    API->>Auth: ② Authenticate
    API->>Auth: ③ Authorize (RBAC)
    API->>Auth: ④ Admission Control
    API->>API: ⑤ Validate & Parse
    API->>etcd: ⑥ Persist to Storage
    API->>Scheduler: ⑦ Notify Scheduler
    Scheduler->>API: ⑧ Assign Node
    API->>Kubelet: ⑨ Create Pod
    Kubelet->>Kubelet: ⑩ Start Containers
    Kubelet->>API: ⑪ Report Status
```
--------------------------------------

#### 🛠️ kubelet Responsibilities

```mermaid
flowchart LR
    kubelet["kubelet"] --> Pods["Run Assigned Pods"]
    kubelet --> Reporting["Report Node/Pod Status"]
    kubelet --> Networking["Manage Pod Networking"]
    kubelet --> Lifecycle["Manage Container Lifecycle"]
    kubelet --> Resources["Monitor Resource Usage"]
```
