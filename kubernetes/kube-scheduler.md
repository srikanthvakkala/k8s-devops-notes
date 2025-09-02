# 📌 kube-scheduler – The Matchmaker of Kubernetes

The **kube-scheduler** is a control plane component responsible for **assigning Pods to Nodes**.  
It doesn’t create or run Pods itself—rather, it decides *where* a Pod should run, based on cluster state and scheduling policies.  

Think of it as the air traffic controller of Kubernetes: it decides which runway (node) each plane (Pod) will land on.

---

## ⚙️ What kube-scheduler Does

1. **Listens for unscheduled Pods**  
   - Watches the API server for Pods that have no assigned node (`.spec.nodeName` is empty).  

2. **Selects a Node**  
   - Evaluates available nodes based on filters (can the Pod run here?) and scoring (which node is best?).  

3. **Binds the Pod**  
   - Writes the chosen Node back to the API server.  
   - API server updates etcd with the Pod’s node assignment.  
   - Kubelet on that node then starts the Pod.  

---

## 🔹 Scheduling Workflow

1. **Pending Pod** → created by Deployment/ReplicaSet/Job.  
2. **API Server** → stores Pod spec (unscheduled) in etcd.  
3. **kube-scheduler** → notices the Pod with no node.  
4. **Filtering** → eliminates nodes that don’t meet requirements.  
5. **Scoring** → ranks the remaining nodes and picks the best one.  
6. **Binding** → updates Pod with selected node → API server → etcd.  
7. **kubelet** → on that node runs the Pod.  

---

## 🔍 Filtering (Predicates)

Nodes must pass basic checks:
- **Resource requirements**: Enough CPU & memory for the Pod.  
- **Affinity/Anti-Affinity rules**: e.g., Pod must run on a node with a certain label.  
- **Taints and tolerations**: Pod must tolerate any taints on the node.  
- **Node conditions**: Node must be healthy and ready.  

---

## 🏆 Scoring (Priorities)

Among the valid nodes, scheduler ranks them based on:  
- **Least resource usage** (spread Pods evenly).  
- **Topology preferences** (zone/region balance).  
- **Pod affinity/anti-affinity weights**.  
- **Custom scheduling plugins**.  

Highest scoring node wins.

---

## 🔄 Example: Scheduling a Pod

- User creates a Deployment with `replicas: 3`.  
- API server persists the Pod specs in etcd, but they have no assigned nodes.  
- kube-scheduler sees 3 unscheduled Pods:  
  - Filters nodes for CPU/memory availability.  
  - Scores nodes to balance load.  
  - Picks NodeA for Pod1, NodeB for Pod2, NodeC for Pod3.  
- Scheduler updates the API server with bindings.  
- Kubelets on NodeA, NodeB, NodeC start the Pods.  

---

## 🗂️ Key Points

- **Scheduler doesn’t run Pods**—it only assigns them.  
- **API server is the middleman** (scheduler never talks directly to etcd).  
- **Extensible:** Supports custom scheduling via plugins or external schedulers.  
- **Core role:** Ensures Pods are efficiently and fairly distributed across the cluster.  
