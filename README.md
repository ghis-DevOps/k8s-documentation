# k8s-documentation



Kubernetes
1 Why kubernetes?
1.1 The Pain Before K8s
You have 50 containers running in production
    10 of them keep crashing. You SSH around to restart them. Every few hours.
    Traffic spikes. You manually launch more EC2s, copy the docker run command, hope it works.
    One server dies at 3am. Customers see errors until you wake up.

What was missing
    Self-healing: 'if a container dies, start a new one automatically.'
    Auto-scaling: 'add more containers when traffic goes up.'
    Service discovery: 'how do my 50 containers find each other when their IPs change?'
    Rolling deploys: 'replace 10 containers with new versions without downtime.'

Kubernetes solves all four
    Google built the design (Borg) over 15 years. Open-sourced as K8s in 2014.
    Around 92 percent of orgs run containers in production. K8s is the default orchestrator. K8s is the default orchestrator.

1.2 Who runs K8s in production
Tech leaders on K8s (Google, Netflix, Airbnb, Spotify, Uber, Shopify, Every FAANG. Every major bank. Every large telco.)
Industries mainstreaming K8s Banking:
    JPMorgan Chase, Goldman Sachs
    Retail: Target, Walmart
    Media: Disney+, HBO Max
    Healthcare: UnitedHealth, CVS

The reason
    Cloud + microservices + AI workloads all need the same primitive: run many containers reliably. 
    K8s is that primitive

1.2 What is Kubernetes?
Kubernetes (K8s) is a container orchestration tool. It automates the deployment, scaling and management of containerized applications across a group of machines (a cluster).
Kubernetes is open source. Cloud providers offer it as a managed service; on AWS this service is Amazon EKS (Elastic Kubernetes Service).

Pasted image
1.2.1 Origin and why it is open source
1.2.3 Kubernetes Architecture (Board)
A Kubernetes cluster is made of two parts:

The Control Plane: the “brain” of the cluster: it makes decisions and stores the cluster state.
The Worker Nodes: the machines that actually run the applications (Pods).
Managed K8s (EKS, GKE, AKS)
1.2.4 Control Plane Components
API Server
    Entry point of the cluster. Every request goes through it; it authenticates, authorizes and validates each request.

Scheduler
    Decides on which worker node each new Pod will run.

etcd
    Key-value database that stores the whole state and configuration of the cluster.

Controller Manager
    Runs control loops that compare the current state with the desired state and correct it (e.g. recreate a Pod that has died).

cloud-controller-manager
    Talks to AWS/GCP/Azure APIs to provision Load Balancers + attach EBS volumes + manage routes.

1.2.5 Worker Node components
Kubelet
    Agent on each node. It receives Pod specifications from the API Server and asks the container runtime to 
    pull images and start the containers.

Container runtime
    Software that actually runs the containers (e.g. containerd).

Kube-proxy
    Manages the network rules on the node so that traffic can reach the Pods (implements Services).

1.2.6 CRI + CNI + CSI : why K8s is pluggable
CRI : Container Runtime Interface
    How kubelet talks to whatever runs containers.
    Options : containerd (default), CRI-O (OpenShift), gVisor (sandboxed runtime).

CNI : Container Network Interface
    How Pods get IPs and how Pods talk to each other across nodes.
    Options : Calico (most common), Cilium (eBPF-based, modern), Flannel (simple), AWS VPC CNI (on EKS).

CSI : Container Storage Interface
    How volumes get provisioned + mounted into Pods.
    Options : AWS EBS CSI, AWS EFS CSI, Azure Disk CSI, GCP PD CSI, Rook (Ceph).

On managed K8s (EKS), the cloud picks for you : containerd + AWS VPC CNI + EBS CSI.
On self-managed K8s, you pick. Calico + containerd + Rook is a common stack.

1.2.7 Kubernetes Cluster Setup options
4 ways to run K8s on a developer laptop. Pick by what you need.

Docker Desktop Kubernetes (we use this in class)
    Single node. Persists across reboots. Zero install friction.

Killercoda kubernetes playgrounds
kind : Kubernetes IN Docker
    Multi-node clusters running as Docker containers.

minikube
    The old default. Still works. Slower to start. More moving parts (separate VM/driver).

k3d / k3s
    k3s : a lightweight K8s distro from Rancher. Single binary.
    k3d : k3s running inside Docker. Great for edge / IoT scenarios.

Prod cluster options : managed by the cloud, or you run it.

Managed K8s (the default for new builds)
    AWS EKS, Google GKE, Azure AKS. EKS is the AWS option. Standard for AWS-first shops.

Self-managed K8s (the long road)
    kubeadm : the official installer. You run it on your own EC2 instances or bare metal.

K8s distributions (wrappers)
    Rancher : multi-cluster management UI on top of vanilla K8s.
    Red Hat OpenShift : K8s + built-in CI/CD + dev console. Common in enterprise.

kubeSpray
    open-source Ansible-based tool that automates the deployment, configuration, 
    and management of production-ready Kubernetes clusters across multiple servers

In production
    Most teams use EKS/GKE/AKS. Self-managed is the senior-level specialty.

1.3 Setup and Practice with Killercoda and Docker Desktop
1.3.1 Practice repo: https://github.com/utrains/kubernetes-practice2.git
1.3.2 Kubectl installation
On Windows with choco
Plain text
Copy code
    choco install kubernetes-cli
On Mac with Homebrew
Bash / Shell
auto
Copy code
brew install kubectl
  or

Bash / Shell
auto
Copy code
brew install kubernetes-cli
Verify the version after installation
Bash / Shell
auto
Copy code
kubectl version --client
1.3.3 Local kluster on Docker Desktop
1- Open Docker Desktop → Settings → Kubernetes.
2- Check Enable Kubernetes, then click Apply & restart.
3- Wait until the Kubernetes status shows as running (green).

1.3.4 Killercoda
1- In your browser, go to the Killercoda official website at https://killercoda.com/.
2- Click on Playgrounds.
3- Select the latest Kubernetes playground

1.3.5 First kubectl commands
Bash / Shell
Copy code
kubectl get nodes
kubectl get ns
kubectl get pods
1.3.6 Alias for kubectl
Plain text
Copy code
alias k='kubectl'
Note: This alias only lasts for the current terminal session
Plain text
Copy code
k get nodes
k get pods -n dev
k get pods -n kube-system
1.3.7 The kubectl apply flow (What happen when you run kubectl apply -f deploy.yaml)
1- kubectl
    Reads your YAML, sends an HTTPS POST to the API server.
2-  API server
    Authenticates (kubeconfig cert) + authorizes (RBAC) + validates schema.
    Writes the desired state to etcd. Returns 'created'.
3- Scheduler
    Sees a new Pod with no node. Picks the best-fit node, patches etcd with 'spec.nodeName'.
4- kubelet (on that node)
    Watches API server, pulls the image, calls the container runtime via CRI to start it.
5- Network + storage + report
    CNI plugin assigns an IP. CSI mounts any volumes.
    kubelet reports 'Pod is Running' back to the API server. kubectl get pods now shows Running.

2. Kubernetes Objects
2.1 Namespaces
A namespace is a logical partition of the cluster

2.1.1 Namespace with kubectl command
Bash / Shell
auto
Copy code
kubectl create ns dev
kubectl create ns staging
2.1.2 Namespace from a manifest
Create a file named namespace.yaml

YAML
Copy code
apiVersion: v1
kind: Namespace
metadata:
  name: security
then run the command:

Bash / Shell
auto
Copy code
kubectl apply -f namespace.yaml
2.1.3 List namespaces
Bash / Shell
auto
Copy code
kubectl get namespace
kubectl get ns
2.1.4 describe a namespace
Bash / Shell
auto
Copy code
kubectl describe ns dev
2.1.5 Delete a namespace
Delete from command line
Bash / Shell
auto
Copy code
kubectl delete ns dev
Delete a from a manifest file
Bash / Shell
auto
Copy code
kubectl delete -f namespace.yaml 
2.2 Pods
A Pod is the smallest and simplest Kubernetes object (Smallest or atomic unit in K8s)
It can contain one or more containers. Each pod has its own IP address for communication.
Containers in the pod share the pod IP address and namespace.
2.1 Create a Pod from the command line
Bash / Shell
Copy code
kubectl run app2 --image=httpd
the pods is created in the default namespace
List and describe the pods
Bash / Shell
auto
Copy code
kubectl get pods 
kubectl get pods -A 
kubectl get pods -o wide
kubectl describe pod app2
2.2.1 Create a pods from a manifest file in the dev namespace
Note: The namespace must first be created.
Bash / Shell
auto
Copy code
kubectl create namespace dev
Create a file named app.yaml
YAML
auto
Copy code
apiVersion: v1
kind: Pod
metadata:
 name: utrains
 namespace: dev
 labels:
  app: utrains
spec:
 containers:
   - name: utrains-app
     image: httpd
     ports:
      - containerPort:80
        name: http
        protocol: TCP
Run the command to create the pod
Bash / Shell
auto
Copy code
  kubectl apply -f pod-namespace.yaml
List and describe pods
Bash / Shell
auto
Copy code
kubectl get pods 
kubectl get pods -n dev
kubectl describe pod utrains -n dev
Get logs of a pod
This is usefull for troubleshooting
Bash / Shell
auto
Copy code
kubectl logs utrains -n dev
Delete pods
Bash / Shell
auto
Copy code
kubectl delete pod utrains -n dev
kubectl delete pod app2
2.2.2 Pods Quality of Service (QoS)
Kubernetes gives each Pod a Quality of Service (QoS) class based on its resource (CPU, memory) requests and limits.

BestEffort: No requests and no limits. Example: app1.yaml
Burstable: At least one request or limit, not equal for all. Example: app2.yaml
Guaranteed: CPU and memory requests equal to limits for every container. Example: app3.yaml
  a. BestEffort: No resources requests or limits defined

Create a file named app1.yaml
Plain Text
Copy code
apiVersion: v1
kind: Pod
metadata:
  name: app1
  labels:
    app: app1
spec:
  containers:
    - name: nginx
      image: nginx
Deploy the pod and check
Bash / Shell
Copy code
kubectl apply -f app1.yaml
kubectl get pod
kubectl describe pod app1  # QoS Class: BestEffort
In the kubectl describe pod output, look at: QoS Class,
Node (where the Scheduler placed the Pod)
And the Events section (Scheduled → Pulling → Pulled → Created → Started).
  b. Burstable: resources requests < resources limits

Create a file named app2.yaml
YAML
auto
Copy code
apiVersion: v1
kind: Pod
metadata:
  name: app2
  labels:
   app: utrains
spec:
  containers:
    - name: nginx
      image: nginx
      resources:
        limits:
          memory: "250Mi"
        requests:
          memory: "150Mi"
Deploy the pod and check
Bash / Shell
Copy code
kubectl apply -f app2.yaml
kubectl get pod
kubectl describe pod app2  # QoS Class: Burstable
In the kubectl describe pod output, look at: QoS Class,
c. Guaranteed: resources requests = resources limits

Create a file named app3.yaml
YAML
auto
Copy code
apiVersion: v1
kind: Pod
metadata:
  name: app3
  labels:
   app: utrains
spec:
  containers:
    - name: nginx
      image: nginx
      resources:
        limits:
          memory: "250Mi"
        requests:
          memory: "150Mi"
Deploy the pod and check
Bash / Shell
Copy code
kubectl apply -f app3.yaml
kubectl get pod
kubectl describe pod app3  # QoS Class: Guaranteed
In the kubectl describe pod output, look at: QoS Class
Delete pods
Bash / Shell
auto
Copy code
kubectl delete -f app1.yaml
kubectl delete -f app2.yaml
kubectl delete -f app3.yaml
2.2.2 Pods probes (health checks)
A Probe is like a diagnostic that is periodically performed by the Kubelet on the containers of a Pod.
The result of these checks can be Success, Failure or Unknown.
LivenessProbe: indicates whether the container is running or not.
ReadinessProbe: indicates whether the container is ready to respond to requests or not.
StartupProbe: indicates when a container application has started. This is very useful for slow starting containers.




<img width="1028" height="692" alt="image" src="https://github.com/user-attachments/assets/3fba1c3c-0e3a-437b-87bd-deb63232f0be" />


