# Pods in Kubernetes

## 1. What is a Pod?

* A Pod is the smallest deployable unit (or building block) in Kubernetes.

* A Pod can contain one or more containers that share:

  * Network
  * Storage
  * Configuration

* Pods are always created inside a namespace for logical separation.

* Pods are the actual running instances of your application.

* Each Pod has a unique IP address within the cluster.

* Application Pods usually run on Worker Nodes, while Kubernetes system Pods run on the Master Node to manage the cluster.

* Some system Pods like `coredns`, `etcd`, `kube-apiserver`, `kube-scheduler`, `kube-proxy`, and `calico` run on the Master Node because Kubernetes needs them to manage cluster operations and functionality.

* These internal management Pods are part of the Control Plane and are stored inside the `kube-system` namespace.

---

# 2. Pod Characteristics

1. **Smallest Deployable Unit** : A Pod is the smallest deployable object (building block) in Kubernetes.

2. **Contains Containers** : A Pod can contain one or more containers that share the same network, storage, and configuration.

3. **Unique IP Address** : Each Pod gets its own unique IP address inside the Kubernetes cluster.

4. **Shared Network** : Containers inside the same Pod communicate using `localhost`.

5. **Shared Storage** : Containers inside a Pod can share the same storage volumes.

6. **Ephemeral (Temporary)** : Pods are temporary objects. If a Pod fails, Kubernetes creates a new Pod with a new IP address.

7. **Namespace Scoped** : Pods are always created inside a namespace for logical separation and resource management.

---

# 3. Types of Pods

## 1. Single-Container Pod

* A Pod contains only one container.

* This is the most commonly used Pod type in Kubernetes.

### Practical Example

```text
Nginx Pod → Nginx Container
```

### Use Cases

* Running web applications
* APIs
* Databases

---

## Pod YAML Example

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: my-first-pod

spec:
  containers:
  - name: my-first-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

---

## Commands

```bash
kubectl apply -f pod.yaml
```

Apply YAML file

---

```bash
kubectl get pods
```

Verify Pods

---

```bash
kubectl get pods -n <namespace>
```

View Pods in namespace

---

# 2. Multi-Container Pod

* A Pod contains two or more containers running together.

All containers share:

* Network
* Storage
* Lifecycle

---

# Sidecar Pattern

* Sidecar Pattern means running a helper container alongside the main application container inside the same Pod for:

  * Logging
  * Monitoring
  * Proxy
  * Security tasks

---

# Multi-Container Pod YAML Example

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: multi-container-pod

spec:
  containers:

  - name: multi-container-container-1
    image: nginx:latest
    ports:
    - containerPort: 80

  - name: log-collector
    image: fluentd:latest
```

---

## Commands

```bash
kubectl apply -f multi-container-pod.yaml
```

Apply YAML file

---

```bash
kubectl get pods
```

Verify Pods

---

```bash
kubectl get pods -n <namespace>
```

View Pods in namespace

---

# Check Both Containers

```bash
kubectl describe pod multi-container-pod
```

Under:

```text
Containers:
```

You will see:

```text
multi-container-container-1
log-collector
```

---

# Check Container Logs

## Nginx Container

```bash
kubectl logs multi-container-pod -c multi-container-container-1
```

---

## Log Collector Container

```bash
kubectl logs multi-container-pod -c log-collector
```

---

# 4. Pod-to-Pod Communication

## Same Node Communication

* Pods running on the same Worker Node can communicate directly using their Pod IP addresses.

* No additional configuration is required.

---

## Different Node Communication

* Pods running on different Worker Nodes communicate through the Kubernetes Cluster Network.

### Common Networking Plugins

* Calico
* Flannel
* Cilium

These plugins allow Pods to communicate across different nodes inside the cluster.

---

# 5. Pod Storage

By default:

```text
Pod storage is temporary (ephemeral)
```

If a Pod restarts or gets deleted:

```text
Data will be lost
```

---

# Solution: Volumes

Volumes are used to persist data inside Kubernetes Pods.

---

# 6. Pod Lifecycle

* Make a Pod request to the API server using a local Pod definition file.

* The API server saves the Pod information in ETCD.

* The scheduler identifies the unscheduled Pod and assigns it to a node.

* The Kubelet running on the node detects the scheduled Pod and starts the container runtime.

* The entire lifecycle state of the Pod is stored in ETCD.

---

# Kubernetes Pod Commands

```bash
kubectl api-resources
```

List all Kubernetes objects

---

```bash
kubectl api-resources --namespaced=true
```

List namespace-level objects

---

```bash
kubectl api-resources --namespaced=false
```

List cluster-level objects

---

```bash
kubectl get po -n <namespace>
```

Get Pods in namespace

---

```bash
kubectl get po -o wide -n <namespace>
```

Get Pod details (IP, Node)

(Check the worker node IP Pod assigned to which node)

---

```bash
watch kubectl get po -n <namespace>
```

Watch Pods continuously

---

```bash
kubectl get po -n <namespace> --show-labels
```

Get Pods with labels

---

```bash
kubectl describe po <podName> -n <namespace>
```

Describe Pod

---

```bash
kubectl logs <pod-name> -n <namespace>
```

Check Pod logs

---

```bash
kubectl exec -it <pod-name> -n <namespace> -- /bin/sh
```

Access Pod shell

---

```bash
kubectl get events -n <namespace>
```

Check namespace events

---

# Pod Examples

# Example 1: Nginx Pod

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginxpod
  namespace: test

spec:
  containers:
  - name: nginx-cont
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

```bash
kubectl apply -f nginx.yaml --dry-run
```

---

```bash
kubectl apply -f nginx.yaml --dry-run=client
```

---

```bash
kubectl apply -f nginx.yaml --dry-run=server
```

---

```bash
kubectl apply -f nginx.yaml
```

---

# Example 2: Nginx Pod with ImagePullBackOff

“ImagePullBackOff occurs when Kubernetes fails to pull the container image from the registry due to reasons like wrong image name, invalid tag, or authentication issues.”

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginxpod1
  namespace: test

spec:
  containers:
  - name: nginx-cont
    image: kkdevopsnginx:1.14.2
    ports:
    - containerPort: 80
```

```bash
kubectl apply -f nginx1.yaml
```

---

```bash
kubectl describe pod nginxpod1 -n test
```

---

# Example 3: Java WebApp Pod

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: javawebapppod
  labels:
    app: javawebapp
  namespace: test

spec:
  containers:
  - name: javawebappcon
    image: satyamolleti4599/maven-web-app:1.0.0
    ports:
    - containerPort: 8080
```

```bash
kubectl describe po javawebapp -n test
```

---

# Example 4: Mongo Pod

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: mongo-pod
  namespace: db-ns

  labels:
    app: mongo
spec:
  containers:
  - name: mongo-container
    image: mongo:latest
    ports:
    - containerPort: 27017
```

---

# Namespace Creation

```bash
kubectl create ns my-namespace
```

---

```bash
kubectl apply -f dbns.yaml
```

---

```yaml
apiVersion: v1
kind: Namespace

metadata:
  name: db-ns

  labels:
    purpose: database-resources
```

---

# Example 5: Tomcat Pod

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: tomcatpod
  namespace: test

spec:
  containers:
  - name: tomcatcontainer
    image: tomcat:9.0
    ports:
    - containerPort: 8080
```

```bash
kubectl apply -f tomcat-pod.yaml -n test
```

Source: 



