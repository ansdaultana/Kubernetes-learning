# Kubernetes-learning

🧩 What is Kubernetes?
Kubernetes (short: K8s) is a system that manages containers (like Docker containers) automatically.
 Think of it as:
🧠 A manager that decides where, when, and how your containers run — even if servers crash.
It handles:
Starting containers


Restarting them if they fail


Spreading them across machines


Scaling them up/down automatically



🏗️ Kubernetes Architecture (Main Components)
1. Cluster
The whole Kubernetes system.
 It’s made of one control plane (brain) and multiple worker nodes (muscles).

2. Node
A machine (real or virtual) that runs your application containers.
Control Plane Node (Master) — makes decisions.


Worker Nodes — actually run your apps.



3. Pod
The smallest unit in Kubernetes.
 A Pod usually holds one container (sometimes more if tightly connected).
It is a wrapper around one or more containers that run together on the same node.
📦 Example:
Pod -> contains your Node.js app container

If your app crashes, Kubernetes restarts the pod.

4. Deployment
A controller that manages multiple copies of your pods.
 If you want 3 pods of your app running:
You create a Deployment


Kubernetes makes sure 3 pods always exist (auto-restart, auto-heal)



5. Service
A stable Network entry point for your pods.
 Pods can die and restart, so their IPs change — Service gives them a fixed name/IP.
Types:
ClusterIP → internal access only


NodePort → access from outside using a port


LoadBalancer → connects to cloud load balancer



6. Namespace
A way to group things inside a cluster.
 Like folders in your computer — helps keep things organized.

7. ConfigMap & Secret
Used to store app settings.
ConfigMap = non-sensitive data (like environment variables)


Secret = passwords or API keys (stored encoded)



8. Volume / PersistentVolume (PV)
Used for storing data permanently (so data doesn’t vanish if a pod restarts).
 For example, MongoDB’s data files.

9. Ingress
A smart router that manages incoming HTTP/HTTPS traffic (like NGINX reverse proxy).

10. kubectl
The command-line tool to talk to your Kubernetes cluster.
 Example:
kubectl get pods
kubectl apply -f deployment.yaml


✅ Deployment: Used for stateless apps where pod identity and data don’t matter.
✅ StatefulSet: Used for stateful apps that need stable pod identity and persistent storage.


🧭 Kubernetes Architecture — How It All Fits Together
Kubernetes is made up of two main parts:
Master Node (Control Plane) — the brain 🧠


Worker Nodes — the hands 💪 (where your apps actually run)



🧠 1. Master Node (Control Plane)
The Master Node is the brain of the cluster — it decides what should happen and manages everything.
It runs these main components 👇



a. API Server
The front door to the cluster.


All commands (kubectl, dashboards, etc.) go through it.


It validates, authenticates, and then forwards requests to other components.
 🗣️ Think: It’s like the receptionist — everything passes through it first.



b. etcd
A key-value database that stores all cluster data.


Keeps track of everything: pods, nodes, configs, secrets, etc.


Acts as the source of truth for the cluster state.
 📚 Think: It’s the memory or brain storage.



c. Scheduler
Watches for new pods that need to run. Decides which node should host each pod (based on CPU, memory, etc.).
 🧩 Think: It’s the planner — assigns work to the right machine.



d. Controller Manager
Continuously checks that the actual state = desired state.


If something breaks (a pod crashes, a node goes down), it takes action to fix it.


Runs different controllers: Node Controller, Deployment Controller, etc.
 ⚙️ Think: It’s the supervisor — keeps things running correctly.



💪 2. Worker Nodes
These are the machines that actually run your applications (pods).
Each worker node has these components 👇

a. Kubelet
An agent running on every node.


Talks to the API server and ensures that the containers (pods) are running as instructed.


Restarts containers if they fail.
 🧍 Think: It’s the worker that actually does the job.



b. Kube Proxy
Manages networking for pods.


Forwards traffic to the right pod, even if IPs change.


Handles load balancing inside the cluster.
 🌐 Think: It’s the network manager that connects everything.



c. Container Runtime
The software that actually runs containers (e.g. Docker, containerd).


Kubernetes doesn’t run containers by itself — it tells the runtime to do it.
 ⚙️ Think: It’s the engine that runs the containers.



🔄 How They Work Together (Step-by-Step Flow)
You run a command:

 kubectl apply -f pod.yaml
 → API Server receives the request.


API Server stores the desired state in etcd.


Scheduler notices a new pod that needs a node → selects the best worker node.


Kubelet on that node gets the instruction → starts the container using the runtime.


Kube Proxy ensures networking is set up so other services can reach this pod.


Controller Manager keeps watching — if a pod dies, it triggers a restart to match the desired state.
🧩 1. Minikube
💡 What it is:
Minikube is a tool that lets you run a Kubernetes cluster on your own computer (like Windows, macOS, or Linux).


It creates a single-node cluster — one master + one worker — inside a virtual machine or container.

🧩 2. kubectl (Kube Control)
💡 What it is:
kubectl is the command-line tool used to talk to Kubernetes clusters (local or remote).


You use it to create, delete, or view resources like pods, deployments, and services.


🚀 Start Kubernetes Cluster (Docker Driver)
🟢 Minikube setup & control
minikube delete                   # Delete any existing cluster
minikube config set driver docker # Set Docker as the driver
minikube start --driver=docker    # Start Minikube cluster
minikube status                   # Check Minikube status
minikube service <service-name>   # Open exposed service in browser



🧱 kubectl basic commands
Check cluster state
kubectl version --client           # Show kubectl version
kubectl get nodes                  # List cluster nodes
kubectl get pods                   # List all pods in current namespace
kubectl get deployments            # List all deployments
kubectl get services               # List all services
kubectl describe pod <pod-name>    # Detailed info about a specific pod
🚀 Deployment commands
kubectl create deployment nginx-deployment --image=nginx:latest --replicas=2
# OR using YAML
kubectl apply -f nginx-deployment.yaml

Update / scale
kubectl scale deployment nginx-deployment --replicas=3  # Change number of pods
kubectl edit deployment nginx-deployment                # Edit config live

Delete resources
kubectl delete deployment nginx-deployment
kubectl delete pod <pod-name>
kubectl delete -f nginx-deployment.yaml


🌐 Expose deployment
kubectl expose deployment nginx-deployment --type=NodePort --port=80
kubectl get svc                                    # Check assigned NodePort
minikube service nginx-deployment                  # Open in browser





