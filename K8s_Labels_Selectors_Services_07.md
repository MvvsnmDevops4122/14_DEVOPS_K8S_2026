# Labels, Selectors and Services in Kubernetes

## 1. Labels

* Labels are key-value pairs attached to Kubernetes objects like Pods, Services, Deployments, ReplicaSets, StatefulSets, DaemonSets, and Nodes.
  
* Labels are used to organize, identify, and group Kubernetes resources.

---

## Example

```yaml id="s0n1p2"
labels:
  app: javawebapp
  environment: production
```

👉 Think of labels like tags in Kubernetes.

---

# 2. Selectors

* Selectors are used to identify or match Kubernetes objects using labels.

* Services, ReplicaSets, and Deployments use selectors to find the correct Pods.

---

## Example

```yaml id="q3r4s5"
selector:
  app: javawebapp
```

This selector matches Pods having:

```yaml id="t6u7v8"
labels:
  app: javawebapp
```

---

# 3. Why Are They Important?

* Labels and Selectors are crucial for creating Kubernetes Services.

* Selectors identify Pods using matching labels and route traffic to the correct Pods.

* Without labels, Services cannot find Pods, so application traffic will fail.

* Labels also help organize and manage environments like:

  * dev
  * test
  * prod

---

# Simple Workflow

```text id="w9x0y1"
Add Labels to Pods
        ↓
Create Service with Selector
        ↓
Selector matches Pod labels
        ↓
Service sends traffic to matching Pods
```

---

# 4. What Happens If Labels Are Missing?

* Services cannot identify which Pods to connect.

* Traffic will not reach the correct Pods.

* Difficult to organize and manage applications and environments.

---

# 5. Creating Pods with Labels

## Java Web Application Pod with Label

### javawebapp.yaml

```yaml id="z2a3b4"
apiVersion: v1
kind: Pod

metadata:
  name: javawebapp

  labels:
    app: javawebapp

  namespace: test

spec:
  containers:
    - name: javawebappcontainer
      image: satyamolleti4599/maven_web_app:1.0.0
      ports:
        - containerPort: 8080
```

---

# Run Commands

```bash id="c5d6e7"
kubectl apply -f javawebapp.yaml --dry-run=client
```

Checks YAML before creating Pod

---

```bash id="f8g9h0"
kubectl apply -f javawebapp.yaml
```

Creates the Pod

---

```bash id="i1j2k3"
kubectl get pods -n test
```

Verify Pod

---

# 6. Create Another Pod with Labels

## MongoDB Pod with Label

### mongo.yaml

```yaml id="l4m5n6"
apiVersion: v1
kind: Pod

metadata:
  name: mongodbpod

  namespace: test

  labels:
    app: mongo

spec:
  containers:
  - name: mongodbcontainer
    image: mongo
    ports:
    - containerPort: 27017
```

---

# Run Command

```bash id="o7p8q9"
kubectl apply -f mongo.yaml
```

Creates MongoDB Pod

---

```bash id="r0s1t2"
kubectl get pods -n test
```

Verify Pod

---

# 7. Check Labels of Pods

## Detailed Pod Information

```bash id="u3v4w5"
kubectl describe pod javawebapp -n test
```

---

```bash id="x6y7z8"
kubectl describe pod mongodbpod -n test
```

---

# View Only Labels

```bash id="a9b0c1"
kubectl get po -n test --show-labels
```

---

# 8. Check Pod IPs

```bash id="d2e3f4"
kubectl get pods -n test -o wide
```

Shows:

* Pod IP
* Node
* Status

Pod IPs are internal and used for Pod-to-Pod communication.

---

# 9. Verify Pod-to-Pod Communication

## Step 1: Enter Inside javawebapp Pod

```bash id="g5h6i7"
kubectl exec -it javawebapp -n test -- sh
```

This opens Pod shell.

---

## Step 2: Install Ping Utility

```bash id="j8k9l0"
apt-get update && apt-get install -y iputils-ping
```

Installs ping command inside Pod.

---

## Step 3: Ping MongoDB Pod IP

```bash id="m1n2o3"
ping <MongoPod-IP>
```

Example:

```bash id="p4q5r6"
ping 192.168.186.132
```

---

# What Happens?

```text id="s7t8u9"
javawebapp Pod
       ↓
Uses Mongo Pod IP
       ↓
Sends Network Request
       ↓
Mongo Pod Responds
```

This confirms:

```text id="v0w1x2"
Pod-to-Pod Communication Working
```
* But Pod IPs are temporary → in real projects, use Services instead of Pod IPs.

* “Kubernetes networking is provided automatically by CNI plugins like Calico or Flannel, which assign Pod IPs and enable Pod-to-Pod communication.”
---

# 10. Why Service is Needed in Kubernetes?

* Pods are ephemeral, meaning Pod IP addresses can change when Pods restart or recreate.

* A Service provides:

  * Stable DNS Name
  * Stable Virtual IP (ClusterIP)
  * Stable communication between Pods inside the cluster

* Services can distribute traffic across multiple Pods having the same labels.

* Kubernetes automatically creates an:

```text id="b6c7d8"
Endpoint Object
```

for every Service.

---

# 11. Endpoints

* Endpoints contain:

  * Pod IPs
  * Ports

* They ensure traffic from the Service is routed to the correct Pods.

* If Pods change, Endpoints update automatically.

---

# 12. Types of Kubernetes Services

```text id="e9f0g1"
ClusterIP | NodePort | LoadBalancer | ExternalName
```

---

# 1. ClusterIP Service (Default)

* ClusterIP is the default Service type in Kubernetes.

* Kubernetes automatically assigns an internal virtual IP called:

```text id="h2i3j4"
ClusterIP
```

to the Service.

* Service is accessible only inside the Kubernetes cluster.

* External users cannot access it directly unless exposed using:

  * NodePort
  * LoadBalancer
  * Ingress

* ClusterIP provides stable internal communication between Pods and Services inside the cluster.

---

# Java Web App Pod + ClusterIP Service

## javawebapp-service.yaml

```yaml id="k5l6m7"
apiVersion: v1
kind: Pod

metadata:
  name: javawebapp
  namespace: test

  labels:
    app: javawebapp

spec:
  containers:
  - name: javawebappcontainer
    image: satyamolleti4599/maven_web_app:1.0.0
    ports:
    - containerPort: 8080

---
apiVersion: v1
kind: Service

metadata:
  name: javawebapp-service
  namespace: test

spec:
  selector:
    app: javawebapp

  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080

  type: ClusterIP
```

---

# Apply YAML

```bash id="n8o9p0"
kubectl apply -f javawebapp-service.yaml
```

---

# Verify Resources

```bash id="q1r2s3"
kubectl get all -n test
```

---

# Verify Service

```bash id="t4u5v6"
kubectl get svc -n test
```

---

# Check Service Details

```bash id="w7x8y9"
kubectl describe svc javawebapp-service -n test
```

Shows:

* ClusterIP
* Endpoints
* Ports
* Selectors

---

# Verify Pod IP

```bash id="z0a1b2"
kubectl get pods -n test -o wide
```

Pod IP shown here should match Service Endpoints.

---

# Check Endpoints Directly

```bash id="c3d4e5"
kubectl get ep javawebapp-service -n test
```

OR

```bash id="f6g7h8"
kubectl get ep -n test
```

---

# Important Point

If:

```text id="i9j0k1"
ENDPOINTS = <none>
```

means:

* Service exists
* But no matching Pods found

Possible reasons:

* Wrong labels
* Pod not running

---

* “ClusterIP provides internal communication inside the Kubernetes cluster using a stable virtual IP.”

---

# 2. NodePort Service

* NodePort exposes Kubernetes applications externally using a port on each Node IP.

* NodePort range:

```text id="l2m3n4"
30000 - 32767
```

* External users can access application using:

```text id="o5p6q7"
<NodeIP>:<NodePort>
```

* NodePort is an extension of:

```text id="r8s9t0"
ClusterIP
```

* Internally, NodePort automatically creates ClusterIP.

---

# Java Web App + NodePort Service

```yaml id="u1v2w3"
apiVersion: v1
kind: Pod

metadata:
  name: javawebapp
  namespace: test

  labels:
    app: java

spec:
  containers:
  - name: javawebappcontainer
    image: satyamolleti4599/maven_web_app:1.0.0
    ports:
    - containerPort: 8080

---
apiVersion: v1
kind: Service

metadata:
  name: javawebapp-nodeport-service
  namespace: test

spec:
  selector:
    app: java

  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30080

  type: NodePort
```

---

# Apply YAML

```bash id="x4y5z6"
kubectl apply -f javawebapp-nodeport.yaml
```

---

# Verify Service

```bash id="a7b8c9"
kubectl get svc -n test
```

Example:

```text id="d0e1f2"
NAME                           TYPE       CLUSTER-IP      PORT(S)
javawebapp-nodeport-service    NodePort   10.96.10.15    8080:30080/TCP
```

---

# Important Port Understanding

| Port            | Purpose                    |
| --------------- | -------------------------- |
| 8080            | ClusterIP internal port    |
| 30080           | External NodePort          |
| targetPort 8080 | Container application port |

---

# Access Application

```text id="g3h4i5"
http://<NodeIP>:30080
```

Example:

```text id="j6k7l8"
http://13.220.144.11:30080/maven-web-application
```

---

# How Traffic Is Routed in NodePort Service?

```text id="m9n0o1"
targetPort → Container Port
port       → Service Port
nodePort   → External Access Port
```

---

# Step-by-Step Traffic Flow

## Step 1: User Accesses Application

```text id="p2q3r4"
http://<NodeIP>:30080
```

Example:

```text id="s5t6u7"
http://54.210.10.20:30080
```

---

## Step 2: Request Reaches NodePort

Kubernetes Node listens on:

```text id="v8w9x0"
30080
```

because:

```yaml id="y1z2a3"
nodePort: 30080
```

---

## Step 3: NodePort Forwards to Service Port

NodePort forwards request internally to:

```yaml id="b4c5d6"
port: 8080
```

This is the Kubernetes Service port.

---

## Step 4: Service Uses Selector

Service checks selector:

```yaml id="e7f8g9"
selector:
  app: java
```

It finds matching Pods.

---

## Step 5: Service Checks Endpoints

Endpoints contain:

```text id="h0i1j2"
Pod IP + Container Port
```

Example:

```text id="k3l4m5"
10.244.1.5:8080
```

---

## Step 6: Traffic Sent to Container

Finally request reaches:

```yaml id="n6o7p8"
targetPort: 8080
```

which is the actual application container port.

---

# Complete Traffic Flow

```text id="q9r0s1"
User
 ↓
<NodeIP>:30080
 ↓
NodePort
 ↓
Service Port (8080)
 ↓
Selector finds matching Pods
 ↓
Endpoints identify Pod IP
 ↓
targetPort:8080
 ↓
Application Container
```

---

# One-Line Interview Answer

“Traffic enters through NodePort, reaches the Service port, and Kubernetes forwards it to the container targetPort of matching Pods.”

Source File: 
