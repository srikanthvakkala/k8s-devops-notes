# 🌐 kube-apiserver – The Front Door of Kubernetes

The **kube-apiserver** is the **central management component** of Kubernetes.  
It exposes the **Kubernetes API** (REST + gRPC), acting as the **front door** for all interactions with the cluster.  

Every request from `kubectl`, controllers, or external clients must pass through the API server.

---

## 📦 What does kube-apiserver do?

### 🔹 Core Responsibilities
- **Process REST requests** from `kubectl`, controllers, schedulers, kubelets, or external clients.  
- **Validate & authenticate** incoming requests.  
- **Authorize** requests via RBAC or ABAC.  
- **Mutate & validate objects** via admission controllers.  
- **Persist state** into `etcd`.  
- **Expose the Kubernetes API** (`/api`, `/apis`, `/healthz`, `/metrics`).  

### 🔹 Key Features
- **Stateless** → Can run multiple replicas behind a load balancer.  
- **Extensible** → Supports **Custom Resource Definitions (CRDs)**.  
- **Secure** → Communicates only over TLS, enforces authentication & authorization.  

---

## 🔑 Request Flow in kube-apiserver

1. **Client Sends Request**  
   - Example: `kubectl apply -f pod.yaml`.  
   - Request reaches the kube-apiserver (HTTPS).  

2. **Authentication**  
   - API server verifies *who* is making the request.  
   - Methods: Certificates, Bearer tokens, ServiceAccounts, OIDC, etc.  

3. **Authorization**  
   - Once authenticated, checks *what* the user is allowed to do.  
   - Methods: RBAC, ABAC, Node authorization.  

4. **Admission Control**  
   - Request passes through **mutating & validating admission webhooks**.  
   - Example: Inject sidecars, validate resource quotas.  

5. **Validation & Schema Check**  
   - Request payload validated against API schema.  

6. **Persist into etcd**  
   - If valid, kube-apiserver writes the object spec into etcd.  

7. **API Response**  
   - Returns **status & object** back to the client.  

---

## 📊 Diagram – Request Lifecycle in kube-apiserver

kubectl / Client  
   --->  API Server (entry point for all cluster requests)  
   --->  Authentication (verify user identity – certs, tokens, service accounts, OIDC)  
   --->  Authorization (check permissions – RBAC, ABAC, Node authorization)  
   --->  Admission Controllers (apply policies – mutating & validating webhooks, quotas, defaults)  
   --->  Schema Validation (ensure request matches Kubernetes API spec)  
   --->  etcd Cluster (store/update cluster state)  
   --->  Response to Client (send result back to kubectl/user)

---

## ⚙️ Interaction with Other Components

- **etcd** → Stores all cluster data (apiserver is the only component that talks directly to etcd).  
- **kube-scheduler** → Queries API server for unscheduled Pods.  
- **kube-controller-manager** → Watches API server for resource changes.  
- **kubelet** → Interacts with API server to fetch Pod specs & report node/pod status.  
- **kubectl** → CLI client to talk with the API server.  

---

## 🛡️ Security in kube-apiserver

- **TLS everywhere** → All communication is encrypted.  
- **Authentication methods**:  
  - X.509 client certs  
  - Bearer tokens  
  - OpenID Connect (OIDC)  
  - ServiceAccounts  
- **Authorization**:  
  - RBAC (Role-Based Access Control)  
  - ABAC (Attribute-Based Access Control)  
  - Node authorization  
- **Admission Plugins**:  
  - NamespaceLifecycle, ResourceQuota, LimitRanger, MutatingAdmissionWebhook, ValidatingAdmissionWebhook  

---

## 🚦 Scaling kube-apiserver

- **Stateless**: Multiple API servers can run behind a **load balancer**.  
- **HA setup**: Typically deployed as 2–3 replicas in production.  
- All replicas connect to the **same etcd cluster**.  

---

## 🧩 Example API Endpoints

- `GET /api/v1/pods` → List all Pods  
- `POST /api/v1/namespaces/default/pods` → Create a Pod  
- `GET /apis/apps/v1/deployments` → List Deployments  
- `GET /metrics` → Prometheus metrics  
- `GET /healthz` → Health check endpoint  

---

## ✅ Summary

- kube-apiserver is the **gateway** to the Kubernetes control plane.  
- Handles **authentication, authorization, admission control, validation, persistence**.  
- Talks to **etcd** to store cluster state.  
- Ensures **secure, consistent, and scalable** access to Kubernetes.  
