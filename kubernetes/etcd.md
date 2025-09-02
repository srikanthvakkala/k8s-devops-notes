# 🧠 etcd – The Brain of Kubernetes

`etcd` is a **distributed, reliable key–value store**.  
In Kubernetes, it acts as the **single source of truth** for all cluster state data.  

Whenever you create or update something in Kubernetes (Pod, Deployment, ConfigMap, etc.), the request is **persisted into `etcd`**.

---

## 📦 What does etcd store?

etcd holds **all critical Kubernetes data**:

### 🔹 Cluster State & Configuration
- `Pods`, `Deployments`, `ReplicaSets`, `Services`  
- `ConfigMaps`, `Secrets`  
- `PersistentVolumes`, `PersistentVolumeClaims`  
- `RBAC`: Roles, ClusterRoles, RoleBindings  

### 🔹 Metadata
- Cluster ID, API versions  
- Bootstrap tokens, certificates  

### 🔹 Networking
- Services, Endpoints, Ingress references  
- CNI configurations  

### 🔹 Events *(short-lived, ephemeral)*

### 🔹 Key Structure (Path-like)
- `/registry/pods/default/nginx-pod` → Pod definition  
- `/registry/secrets/default/db-secret` → Secret data  

📌 Values are stored as **JSON** or **Protobuf-serialized objects**.

---

## ⚙️ How etcd Works Inside Kubernetes

1. **kubectl → API Server**  
   - You run `kubectl apply -f pod.yaml`.  
   - The API server validates and authenticates the request.  

2. **API Server → etcd**  
   - If valid, the API server persists the object definition into etcd.  
   - Example: Pod spec stored at key `/registry/pods/default/nginx-pod`.  

3. **Controllers Watch etcd**  
   - Controllers (Deployment, ReplicaSet, etc.) continuously **watch** etcd for changes.  
   - When they detect new desired state, they reconcile it (e.g., create Pods via kubelet).  

4. **Scheduler & Kubelet**  
   - **Scheduler** queries the API server (not etcd directly) to assign Pods to nodes.  
   - **Kubelet** runs Pods on nodes and reports status → API server → persisted into etcd.  

---

## 📊 Diagram – Pod Creation Flow inside etcd

kubectl apply (Pod manifest) --> API Server (validates request) --> etcd (stores data as key-value pairs)


---

## 🔍 Inside etcd During Pod Creation

- **API Server writes desired state** of the Pod into etcd.  
  - Example key: `/registry/pods/default/nginx-pod`  
  - Example value: full Pod spec (YAML/JSON).  

- **Raft Consensus Algorithm** ensures replication:  
  - Leader node accepts the write.  
  - Follower nodes replicate log entries.  
  - Once quorum acknowledges, the write is **committed**.  

- **Durable storage mechanisms**:  
  - Written first to **WAL (Write Ahead Log)**.  
  - Periodic **snapshots** for recovery.  
  - **Compaction** removes old revisions for optimization.  

- **Revision history**:  
  - Every update creates a new, immutable revision.  
  - Controllers use the `watch API` to track real-time changes.  

- **Strong consistency**:  
  - Any read after commit returns the **latest state**.  
  - If the Leader fails → a new Leader is elected automatically.  

---

