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

#  Kubernetes Architecture Overview

<img width="1402" height="882" alt="image" src="https://github.com/user-attachments/assets/4b67947d-7fa8-48f0-83a3-1b891e0bcbb1" />

* Kubernetes has two main components:

  1. Control Plane (Master)

  2. Nodes (Workers/Minions/Slaves)

A group of nodes (Control Plane + Workers) together form a Kubernetes Cluster.

##  1. Control Plane (Master)

* This is the brain of the cluster, handling decisions and maintaining the overall cluster state.

# API Server: 

* The API Server is the front-end of the Kubernetes control plane and the heart of the cluster.

* It acts as the central hub for communication.

* It accepts REST API requests (from kubectl, UIs, or other tools), validates them.

* It retrieves or stores data in etcd, and manages authentication and authorization to control access.

# etcd:

*  etcd is a Key-value store.

* etcd holds the current state of all Kubernetes resources, including Pods, Secrets (for sensitive information), 

  ConfigMaps (for non-sensitive configuration data), Deployments, Services, and more.

* When a request is made through the API Server, etcd provides the necessary information, 

  such as the list of Pods and their statuses.

# Scheduler:

* The scheduler identifies the unscheduled pod and assigns it to a node.  

* It selects the best node for each pod based on available CPU, memory, and other criteria.

# kube-controller-manager

* The Controller Manager monitors the state of all nodes and resources in the cluster.

* It ensure everything is working properly and matches the desired state.
 
* If something goes wrong, like a Pod crash, it automatically takes action to fix it by restarting or rescheduling the Pod.

# cloud-controller-manager:

* Integrates with cloud provider APIs (AWS, Azure, GCP) to manage cloud-specific resources (load balancers, storage, etc.).

## 2. Node Components (Workers of the cluster)

# kubelet: 

* The Kubelet is an agent on each worker node that makes sure Pods are running properly.

* The Kubelet running on the node detects the scheduled pod and starts the container runtime.

* If a Pod on the node crashes or has a problem, the Kubelet informs the API Server.

* The API Server passes this info to the Controller Manager, which then fixes it (like restarting or rescheduling the Pod).

👉 In short: Kubelet → API Server → Controller Manager → Fix the Pod ✅

# kube-proxy: 
 
* Maintains network rules and enables service-to-pod communication inside and outside the cluster.

# Pods: 

* Smallest deployable units in Kubernetes; contain one or more containers
 
# CRI (Container Runtime Interface): 

* Interface between kubelet and container runtime (e.g., containerd, Docker)

* The Container Runtime is the software on each worker node that actually runs the containers.

---
