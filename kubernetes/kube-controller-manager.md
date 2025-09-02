# 🛠️ kube-controller-manager – The Operators of Kubernetes

The **kube-controller-manager** is a core control plane component in Kubernetes.  
It runs a collection of **controllers**, each of which watches the cluster state (via the API server) and tries to move the actual state toward the desired state stored in **etcd**.  

Think of it as the “operators” in a factory—always watching the machines and making adjustments when something drifts from the plan.

---

## 📦 What are Controllers?

Controllers are **control loops**:  
- They continuously **observe** cluster state (through the API server).  
- Compare it with the **desired state** (stored in etcd).  
- Take corrective actions if the two differ.  

---

## 🔹 Major Controllers Inside kube-controller-manager

- **Node Controller**  
  - Detects and responds when nodes go down.  
  - If a node is unreachable for too long, Pods are rescheduled onto healthy nodes.  

- **Replication / Deployment Controller**  
  - Ensures the specified number of Pod replicas are always running.  
  - If one Pod crashes, it spins up another.  

- **Job & CronJob Controller**  
  - Runs Pods to completion for Jobs.  
  - Triggers Jobs on schedule for CronJobs.  

- **Service Account & Token Controller**  
  - Manages default accounts and API tokens for Pods.  

- **EndpointSlice Controller**  
  - Creates and updates EndpointSlices that link Services → Pods.  

- **PersistentVolume Controller**  
  - Handles binding of PersistentVolumes (PVs) and PersistentVolumeClaims (PVCs).  

---

## ⚙️ How kube-controller-manager Works

1. **Watch Loop**  
   - Controllers “watch” for changes via the API server (which pulls from etcd).  
   - Example: Desired replicas for a Deployment = 3.  

2. **Detect Drift**  
   - Actual state differs (e.g., only 2 Pods running).  

3. **Reconcile State**  
   - Controller creates a new Pod to bring count back to 3.  

4. **Update Status**  
   - Controller updates cluster state back through the API server.  
   - API server persists changes into etcd.  

---

## 🔄 Example: Deployment Controller

- You define a Deployment with `replicas: 3`.  
- API server stores this spec in etcd.  
- Deployment Controller notices desired = 3 but actual = 2.  
- It creates one more Pod via the API server.  
- New Pod spec → persisted in etcd → scheduled → run by kubelet.  

**Result:** Desired and actual states match again.  

---

## 🗂️ Key Points

- Runs as a **single process** (`kube-controller-manager`) but hosts many controllers.  
- Communicates only with the **API server** (never directly with etcd).  
- Ensures Kubernetes is **self-healing**: drift between desired and actual state is automatically corrected.  
