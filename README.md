# 🧩 Kubernetes Learning Notes

## 🧠 What is Kubernetes?

Kubernetes (**K8s**) is a system that manages containers (like Docker containers) automatically.

Think of it as:
> A manager that decides **where, when, and how** your containers run — even if servers crash.

It handles:
- Starting containers  
- Restarting them if they fail  
- Spreading them across machines  
- Scaling them up/down automatically  

---

## 🏗️ Kubernetes Architecture (Main Components)

### 1. Cluster
The whole Kubernetes system — made of one **Control Plane (brain)** and multiple **Worker Nodes (muscles)**.

### 2. Node
A machine (real or virtual) that runs your application containers.
- **Control Plane Node (Master)** — makes decisions  
- **Worker Nodes** — actually run your apps  

### 3. Pod
The smallest unit in Kubernetes.  
A Pod usually holds one container (sometimes more if tightly connected).  
It’s a wrapper around containers that run together on the same node.

> 📦 Example:  
> Pod → contains your Node.js app container  
> If your app crashes, Kubernetes restarts the pod.

### 4. Deployment
A controller that manages multiple copies of Pods.  
If you want 3 Pods of your app running:
- You create a Deployment  
- Kubernetes ensures 3 Pods always exist (auto-restart, auto-heal)

### 5. Service
A stable network entry point for your Pods.  
Since Pods restart and change IPs, a **Service** provides a fixed name/IP.

**Types:**
- `ClusterIP` → internal access only  
- `NodePort` → external access using a port  
- `LoadBalancer` → connects to a cloud load balancer  

### 6. Namespace
A way to group resources inside a cluster — like folders in your computer.

### 7. ConfigMap & Secret
Used to store app settings:
- **ConfigMap:** non-sensitive data (like environment variables)
- **Secret:** sensitive data (passwords or API keys, stored encoded)

### 8. Volume / PersistentVolume (PV)
Used for storing data permanently (e.g., MongoDB’s data files).

### 9. Ingress
A smart router managing incoming HTTP/HTTPS traffic (like an NGINX reverse proxy).

### 10. kubectl
The command-line tool to talk to your Kubernetes cluster.

Examples:
```bash
kubectl get pods
kubectl apply -f deployment.yaml
```

✅ **Deployment:** For stateless apps where pod identity/data don’t matter  
✅ **StatefulSet:** For stateful apps that need stable pod identity & storage  

---

## 🧭 Kubernetes Architecture — How It All Fits Together

Kubernetes consists of two main parts:
- **Master Node (Control Plane)** — the brain 🧠  
- **Worker Nodes** — the hands 💪 (where apps run)

---

### 🧠 Master Node (Control Plane)

The Control Plane decides what happens and manages everything.

#### a. API Server
- The front door of the cluster  
- All commands (kubectl, dashboard, etc.) go through it  
- Validates, authenticates, and forwards requests  

> 🗣️ Think: The receptionist — everything passes through it first.

#### b. etcd
- Key-value database storing all cluster data  
- Source of truth for cluster state  

> 📚 Think: The cluster’s memory or brain storage.

#### c. Scheduler
- Watches for new pods and assigns them to nodes based on CPU, memory, etc.  

> 🧩 Think: The planner assigning work to the right machine.

#### d. Controller Manager
- Continuously checks that **actual state = desired state**  
- Fixes issues like crashed pods or down nodes  

> ⚙️ Think: The supervisor keeping things running correctly.

---

### 💪 Worker Nodes

Machines that actually run your applications (Pods).

#### a. Kubelet
- Agent running on each node  
- Talks to the API server and ensures containers are running  

> 🧍 Think: The worker that actually does the job.

#### b. Kube Proxy
- Manages networking and forwards traffic to correct Pods  
- Handles load balancing  

> 🌐 Think: The network manager connecting everything.

#### c. Container Runtime
- Software that actually runs containers (e.g., Docker, containerd)  

> ⚙️ Think: The engine running containers.

---

## 🔄 How They Work Together (Step-by-Step Flow)

1. Run command:  
   ```bash
   kubectl apply -f pod.yaml
   ```
2. API Server receives request  
3. etcd stores desired state  
4. Scheduler assigns Pod to a Node  
5. Kubelet on that Node starts the container  
6. Kube Proxy sets up networking  
7. Controller Manager monitors and restarts if needed  

---

## 🧩 Tools

### 1. Minikube
> Run a Kubernetes cluster locally (on Windows, macOS, or Linux).  
Creates a **single-node cluster** (1 master + 1 worker) inside a VM or container.

### 2. kubectl (Kube Control)
> CLI tool to manage Kubernetes clusters (local or remote).

---

## 🚀 Start Kubernetes Cluster (Using Docker Driver)

```bash
minikube delete                   # Delete existing cluster
minikube config set driver docker # Use Docker driver
minikube start --driver=docker    # Start Minikube
minikube status                   # Check status
minikube service <service-name>   # Open exposed service
```

---

## 🧱 kubectl Basic Commands

### Check cluster state
```bash
kubectl version --client
kubectl get nodes
kubectl get pods
kubectl get deployments
kubectl get services
kubectl describe pod <pod-name>
```

### Deployments
```bash
kubectl create deployment nginx-deployment --image=nginx:latest --replicas=2
# OR using YAML
kubectl apply -f nginx-deployment.yaml
```

### Update / Scale
```bash
kubectl scale deployment nginx-deployment --replicas=3
kubectl edit deployment nginx-deployment
```

### Delete
```bash
kubectl delete deployment nginx-deployment
kubectl delete pod <pod-name>
kubectl delete -f nginx-deployment.yaml
```

### Expose Deployment
```bash
kubectl expose deployment nginx-deployment --type=NodePort --port=80
kubectl get svc
minikube service nginx-deployment
```

---

## 🚀 Kubernetes Deployment & Service — Core Notes

### 🏷️ Labels & Selectors
Labels are key–value pairs attached to resources (e.g. `app: nginx`).

Selectors in a Deployment or Service match these labels to identify Pods.

Example:
```yaml
# Pod label
labels:
  app: nginx

# Service selector
selector:
  app: nginx
```

> Service knows which Pods to send traffic to.  
Think of labels as “tags” that connect Kubernetes components.

---

### 🌐 Service

Exposes Pods (which change dynamically) using a stable endpoint (`ClusterIP`, `NodePort`, etc.).

**Types:**
- **ClusterIP** → internal communication only  
- **NodePort** → external access via Node’s IP (ports 30000–32767)  
- **LoadBalancer** → cloud-based external access  

---

### 🔁 Ports Explained

| Field       | Meaning                          | Example |
|--------------|----------------------------------|----------|
| `port`       | Service’s internal port          | 80       |
| `targetPort` | Pod container’s port             | 80       |
| `nodePort`   | External Node port (for NodePort)| 30007    |

➡️ **Flow:**  
External Request → NodePort (30007) → Service Port (80) → Pod TargetPort (80)

---

### 🧩 NodePort in Kubernetes

**NodePort** exposes your app outside the cluster.  
It opens a port (`30000–32767`) on every node.  
Traffic hitting that port gets forwarded to the right Service → Pod.

---

## 🧠 Kubernetes – MongoDB & Mongo Express Setup

1️⃣ **MongoDB Secret**
- Stores username/password (base64 encoded).  
- Accessed via `secretKeyRef`.

2️⃣ **MongoDB Deployment**
- Runs MongoDB container  
- Uses env vars from Secret  
- Port: `27017`

3️⃣ **MongoDB Service**
- Type: `ClusterIP` (internal)  
- Name: `mongodb-service` → used by Mongo Express to connect

4️⃣ **Mongo Express Deployment**
- UI for MongoDB  
- Uses env vars:  
  - `ME_CONFIG_MONGODB_SERVER = mongodb-service`  
  - Credentials from Secret  
- Port: `8081`

5️⃣ **Mongo Express Service**
- Type: `LoadBalancer` (external)  
- NodePort (e.g., 30001) exposes it  

Access using:
```bash
minikube service mongo-express-service
```
→ Opens tunnel and gives local URL (e.g., `http://127.0.0.1:33679`)

6️⃣ **Notes**
- `ClusterIP` → internal only  
- `LoadBalancer` (in Minikube) → use `minikube service`  
- Manual NodePort may not work without tunnel  

---

## ✅ Summary

- **Kubernetes** manages containerized apps automatically.  
- **Pods** are smallest deployable units.  
- **Deployments** ensure Pods stay healthy.  
- **Services** give stable access to Pods.  
- **Minikube** lets you test locally.  
- **kubectl** is your main control tool.  

Keep learning, keep automating! 🚀
