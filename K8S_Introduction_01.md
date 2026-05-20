#  What is Kubernetes?

* Kubernetes, also called K8s, is an open-source container orchestration platform used to automate the deployment, scaling, load balancing, and management of containerized applications.

* It helps manage containers efficiently across multiple servers and ensures high availability and self-healing of applications.

---

## What is Container Orchestration?

* Container orchestration means managing containers automatically, such as:

- Starting containers
- Restarting failed containers
- Scaling containers based on traffic
- Load balancing application traffic

* Kubernetes continuously monitors the cluster and maintains the desired state of the application.

---

### Kubernetes is not a replacement for Docker

Docker and Kubernetes work together:

- Docker is used to create containers.
- Kubernetes is used to manage containers across multiple servers.

Kubernetes does not replace Docker.
It mainly replaces Docker Swarm, which is another container orchestration tool.

---

## History

- Developed by Google
- Written in Go/Golang
- Donated to CNCF (Cloud Native Computing Foundation) in 2014
- Kubernetes v1.0 was released on July 21, 2015

---

 # Kubernetes Key Features / Key Responsibilities

---

## 1. Auto-Scheduling

- Kubernetes provides an advanced scheduler to launch containers on cluster nodes.

- The Kubernetes Scheduler automatically assigns Pods to available worker nodes.

- It checks:
  - CPU and memory resources
  - Node conditions
  - Policies and constraints

---

## 2. Self-Healing Capabilities

- Kubernetes automatically:
  - Restarts failed containers
  - Replaces unhealthy Pods
  - Reschedules Pods to healthy nodes

- This helps maintain application availability.

---
## 3. Automated Rollouts and Rollbacks

- Kubernetes gradually updates applications without downtime.

- If the new version fails, Kubernetes can revert the application to the previous stable version.

---

## 4. Horizontal Scaling

- Kubernetes can automatically scale applications up or down based on:
  - CPU usage
  - Memory usage
  - Custom metrics

- This is called Horizontal Pod Autoscaling (HPA).

---

### 5. Load Balancing

- Kubernetes distributes incoming traffic equally across Pods/containers so that no single Pod is overloaded.

---

### 7. Service Discovery & Networking

- Kubernetes enables communication between containers using Services and internal DNS.

---

## 8. Storage Orchestration

Kubernetes can automatically manage storage for containers using Persistent Volumes (PV) and Persistent Volume Claims (PVC).

---

# Kubernetes Architecture Overview

---

Kubernetes architecture mainly consists of two components:

1. Control Plane (Master Node)
2. Worker Nodes

- A group of Control Plane and Worker Nodes together forms a Kubernetes Cluster.

---

# 1. Control Plane (Master Node)

- The Control Plane is the brain of the Kubernetes cluster.

- It manages the entire cluster and maintains the desired state.

---

## Main Components

---

# API Server

- The API Server is the front-end and central communication hub of the Kubernetes Control Plane.

- It receives requests from kubectl, UI dashboards, and external tools.

- It validates requests and communicates with other Kubernetes components.

- It stores and retrieves cluster information from etcd.

- It also handles authentication and authorization.

---

# etcd

- etcd is a distributed key-value store used by Kubernetes.

- It stores the current state and configuration of the Kubernetes cluster.

- It stores information about:
  - Pods
  - Deployments
  - Services
  - Secrets
  - ConfigMaps
  - Nodes

- The API Server stores and retrieves cluster information from etcd.

---

# Scheduler

- The Scheduler identifies Pods that are not assigned to any Worker Node.

- It selects the best Worker Node for each Pod based on:
  - CPU availability
  - Memory availability
  - Resource requirements
  - Node conditions

- After selecting the suitable node, it assigns the Pod to that node.

---

# kube-controller-manager

- The Controller Manager continuously monitors the state of the cluster and its resources.

- It ensures the actual state of the cluster matches the desired state.

- If any issue occurs, such as a Pod failure or node failure, it automatically takes corrective actions like restarting, recreating, or rescheduling Pods.

---

# cloud-controller-manager

- The cloud-controller-manager integrates Kubernetes with cloud platforms such as AWS, Azure, and GCP.

- It manages cloud resources like Load Balancers, storage, and networking services.

---

# 2. Worker Nodes

---

Worker Nodes run the actual application containers.

---

## Main Components

---

### kubelet

- kubelet is an agent that runs on each Worker Node.

- It communicates with the API Server and ensures Pods are running properly on the node.

- It starts and manages containers using the container runtime.

- If any Pod fails or becomes unhealthy, kubelet reports the issue to the API Server.

- The API Server passes the information to the Controller Manager, which takes corrective actions such as restarting or rescheduling the Pod.

👉 Flow:

Kubelet → API Server → Controller Manager → Fix the Pod

---

# kube-proxy

- kube-proxy manages network rules on each Worker Node.

- It enables communication between Services and Pods inside and outside the cluster.

- It also handles load balancing of network traffic.

---

# Pods

- Pods are the smallest deployable units in Kubernetes.

- A Pod contains one or more containers.

---

# CRI (Container Runtime Interface)

- CRI is the interface between kubelet and the container runtime.

- The container runtime is responsible for running containers on Worker Nodes.

Examples:
- containerd
- CRI-O
- Docker

---
