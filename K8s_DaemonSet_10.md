# Kubernetes DaemonSet

---

# 1. What Is a DaemonSet?

A DaemonSet is a Kubernetes object that ensures one copy of a Pod runs on every worker node in the cluster.

When a new node is added:
- DaemonSet automatically creates the Pod on that node.

When a node is removed:
- DaemonSet automatically removes the Pod on that node.

---

# 2. Main Purpose of DaemonSet

DaemonSet is mainly used for:
- monitoring
- logging
- networking
- node management tasks

where one Pod must run on every node.

---

# 3. Key Features of DaemonSet

## 1. One Pod Per Node

DaemonSet automatically runs one Pod on each node.

---

## 2. Automatic Pod Management

- When a new Node is added, the DaemonSet automatically creates a Pod on it.
- When a Node is removed, the Pod running on that Node is also terminated.

---

## 3. Runs Background Services

Used for:
- log collection
- monitoring agents
- networking plugins

---

## 4. Selective Node Deployment

A DaemonSet can be configured to run only on specific Nodes using:
- Node selectors
- Affinity rules
- Taints and tolerations

---

# 4. Common Use Cases of DaemonSet

## 1. Monitoring

Tools like Prometheus Node Exporter or Datadog Agent run on every Node to monitor:
- CPU
- Memory
- Disk
- Node health

DaemonSet ensures one monitoring Pod runs on every Node automatically.

---

## 2. Logging

Tools like Fluentd, Logstash, or Filebeat run on every Node to collect:
- container logs
- system logs

These logs are sent to a central logging server.

DaemonSet ensures one logging Pod runs on every Node.

---

## 3. Networking Services

Tools like kube-proxy, Calico, and Flannel run on every Node to manage:
- Pod networking
- network communication
- routing rules

DaemonSet ensures every Node has the required networking Pod.

---

# 5. How DaemonSet Works

1. User creates a DaemonSet YAML.

2. Kubernetes checks all worker nodes.

3. DaemonSet schedules one Pod on each node.

4. If a new node is added:
   - DaemonSet automatically creates a new Pod.

5. If a node is removed:
   - corresponding Pod is deleted automatically.

---

# 6. DaemonSet YAML Example (`ds.yaml`)

```yaml
apiVersion: apps/v1
kind: DaemonSet

metadata:
  name: nginx

spec:
  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

---

# Apply DaemonSet

```bash
kubectl apply -f ds.yaml
```

---

# Step 2: Check Cluster Nodes

```bash
kubectl get nodes
```

### Example Output

```text
Master   → ip-172-31-34-191
Worker1  → ip-172-31-5-194
Worker2  → ip-172-31-3-215
```

---

# Step 3: Verify DaemonSet Pods

```bash
kubectl get all -o wide
```

Shows:
- Pod IPs
- Node names
- Images
- Services

You will see:
- one nginx Pod on each Worker Node

No Pod runs on the Master Node by default.

---

# Step 4: Check Taints on Nodes

```bash
kubectl describe node ip-172-31-34-191
```

```bash
kubectl describe node ip-172-31-5-194
```

### Result

- Worker Nodes → No taints → Pods scheduled normally
- Master Node → Has `NoSchedule` taint → DaemonSet Pods are not scheduled there

---

# 7. Deploy DaemonSet on Master Node Also

## Step 1: Delete Old DaemonSet

```bash
kubectl delete -f ds.yaml
```

---

## Step 2: Add Tolerations in YAML

Update `ds.yaml`:

```yaml
apiVersion: apps/v1
kind: DaemonSet

metadata:
  name: nginx

spec:
  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      tolerations:
      - key: "node-role.kubernetes.io/control-plane"
        operator: "Exists"
        effect: "NoSchedule"

      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

---

## Step 3: Apply Updated DaemonSet

```bash
kubectl apply -f ds.yaml
```

---

## Step 4: Verify Pods

```bash
kubectl get pods -o wide
```

Now you should see:
- one nginx Pod on Master Node
- one nginx Pod on each Worker Node

Total:
```text
3 Pods
```

---

# 8. Key Takeaways

- DaemonSet = One Pod per Node
- Kubernetes automatically maintains this behavior
- By default, DaemonSet Pods run only on Worker Nodes
- To run Pods on Master Node, add tolerations
- DaemonSet does not use:
  ```yaml
  replicas:
  ```
- Pod count always equals Node count
- Commonly used for:
  - monitoring
  - logging
  - networking
````
