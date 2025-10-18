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

## 🧩 Kubernetes Namespace

### 📘 What is a Namespace?
- A **Namespace** is a way to organize and isolate resources within a Kubernetes cluster.
- Think of it as a **folder** inside the cluster — each namespace can contain its own pods, services, and configurations.

---

### 🧱 Default Namespaces
- **default** → for resources without a specified namespace.  
- **kube-system** → for system components (like kube-dns).  
- **kube-public** → readable by all users (even unauthenticated).  
- **kube-node-lease** → stores node heartbeat leases.

A namespace is a “folder” — Ingress can only see Services inside the same folder.


## 🗂️ Namespace (MongoDB Example)

Namespaces help **organize and isolate** resources like dev, staging, and prod.

### Example

**Create Namespace**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev


apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
  namespace: dev
spec:

apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
  namespace: dev
---

### ⚙️ Basic Commands
```bash
# List all namespaces
kubectl get namespaces

# Create a new namespace
kubectl create namespace dev

# Apply manifest to specific namespace
kubectl apply -f app.yaml -n dev

# Switch default namespace (using context)
kubectl config set-context --current --namespace=dev


### 🧾 Add Namespace in YAML
Add this under `metadata` in any resource:
```yaml
metadata:
  name: my-app
  namespace: mongodb
```


# 🧭 Kubernetes Ingress Notes

## 🌐 What is Ingress?
Ingress is a Kubernetes object that manages **external HTTP/HTTPS access** to services inside a cluster.  
It acts as a **smart router (reverse proxy)** — routing requests based on hostnames or paths.

---

## ⚙️ How It Works
1. **Ingress Controller**  
   - A pod running Nginx (or other controller) that listens on ports 80/443.
   - Must be installed first:  
     ```bash
     minikube addons enable ingress
     ```
2. **Ingress Resource (YAML)**  
   - Defines routing rules.
   - Example:
     ```yaml
     apiVersion: networking.k8s.io/v1
     kind: Ingress
     metadata:
       name: node-app-ingress
     spec:
       rules:
         - host: nginx.local
           http:
             paths:
               - path: /
                 pathType: Prefix
                 backend:
                   service:
                     name: node-service
                     port:
                       number: 3000
     ```

---

## 🧩 How Ingress Works Internally
1. Browser → DNS → `192.168.49.2` (Minikube IP)  
2. Ingress Controller listens on port 80/443.  
3. Based on the host/path rule, it routes traffic to the correct **Service → Pod**.

---

## 🧱 Common Setup Steps
1. Enable Ingress in Minikube:
   ```bash
   minikube addons enable ingress

kubectl get ingress
kubectl describe ingress
kubectl get pods -n kube-system | grep ingress
minikube ip

**Browser → nginx.local → Ingress Controller → Service → Pod**


# 🌐 Kubernetes Load Balancer & Domain Routing

## 🚪 Load Balancer (in Kubernetes)
- A **LoadBalancer** is a Kubernetes **Service type** that exposes your cluster to the **outside world**.
- It gets a **public IP address** (from your cloud provider or via Minikube tunnel).
- All external traffic first enters through this LoadBalancer, then goes to your **Ingress Controller**.
- The LoadBalancer can distribute traffic among multiple Ingress Controller pods for reliability.

### 🔁 Traffic Flow
Browser → Domain (DNS)
↓
LoadBalancer (Public IP)
↓
Ingress Controller
↓
Ingress Rules (host/path)
↓
Service → Pods

## 🌍 Domain and Ingress Relationship
- Your **domain** (e.g. `myapp.example.com`) only needs to point to the **external IP** of the **Ingress Controller’s LoadBalancer**.
- The **Ingress Controller** reads the Ingress YAML rules and decides:
  - Which **namespace**
  - Which **Service**
  - Which **path**  
  the traffic should go to.


# 🌐 Node App and MongoDB — Service Communication

## 🧩 Overview
This setup shows how a Node.js application connects to MongoDB inside Kubernetes using internal and external services.

---
# 🧩 Ingress, Controller & Load Balancer Overview

## 1. Roles Overview

![alt text](./pics/image.png)

---

## 2. How They Work Together

![alt text](./pics/image-1.png)


## 🔹 Minikube (Minikube has no real “cloud” load balancer.)

## 🔹 Internal Communication
- The **Node.js app** connects to **MongoDB** through an internal **ClusterIP Service**.
- Example connection string:
  ```env
  MONGO_URL=mongodb://admin:pass@mongodb-service:27017/user-accounts?authSource=admin
> 🧩 **App–Service–Database Flow (Node.js + MongoDB)**  
>
> 🔹 **Internal Communication**  
> The Node.js app connects to MongoDB using an internal **ClusterIP** service.  
>
> **Example:**  
> ```env
> MONGO_URL=mongodb://admin:pass@mongodb-service:27017/user-accounts?authSource=admin
> ```
>
> `mongodb-service` is only accessible **inside the cluster** and routes to the MongoDB pods.  
>
> 🔹 **External Communication**  
> The Node.js app is exposed to the outside world using a **LoadBalancer** or **Ingress**.  
> This allows users to access the app via a browser or external network.  
>

> ```
>
> ✅ **Key Points**  
> - **ClusterIP** → Used for internal communication (Node app → MongoDB).  
> - **Service names** act as DNS inside the cluster to connect components easily.  

- **Ingress** = Rules for routing (YAML config).
- **Ingress Controller** = Reverse proxy that enforces those rules.
- **LoadBalancer** = Entry point from the internet to the cluster. Used to expose the Node.js app externally.  
- **Domain (DNS)** = Maps your hostname to the LoadBalancer’s public IP.



# 📦 Helm — Kubernetes Package Manager

## 🧩 What is Helm
- **Helm** is the **package manager for Kubernetes**, similar to `apt`, `npm`, or `pip`.
- It helps you **package, configure, and deploy** applications easily.
- A Helm package is called a **Chart** — a reusable template for deploying an app.

---

## ⚙️ What Helm Does
- Deploys and manages complex Kubernetes apps using a single command.
- Combines multiple YAML files into one **Chart**.
- Supports **versioning**, **upgrades**, and **rollbacks**.
- Allows you to customize deployments via a single `values.yaml` file.

---

## 🌟 Key Features
| Feature | Description |
|----------|-------------|
| **Charts** | Pre-packaged templates for apps (e.g., Nginx, MongoDB, Grafana) |
| **Templating** | Parameterized YAMLs with reusable logic |
| **Releases** | Versioned deployments that can be upgraded or rolled back |
| **Repositories** | Online libraries of charts (like DockerHub for Helm) |
| **Values File** | Custom configuration without editing manifests |

---

## 💡 Why Helm is Used
| Purpose | Benefit |
|----------|----------|
| 🧠 Simplifies deployments | One command can create all related resources |
| 🔁 Consistency | Same chart used across dev, staging, and prod |
| ⚙️ Easy customization | Override defaults in `values.yaml` |
| 🕒 Version control | Rollback to older releases easily |
| 📦 Reusability | Share and reuse common charts across teams |

---

```
mychart/
 ├── Chart.yaml      # Metadata (name, version, description)
 ├── values.yaml     # Default configuration values
 ├── templates/      # Kubernetes manifest templates
 └── charts/         # Dependencies (other charts)
```