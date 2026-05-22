
# Static Pods in Kubernetes
---

## 1. What Is a Static Pod in Kubernetes?

* A Static Pod is a special type of Pod managed directly by the `kubelet` on a specific node.

* Static Pods are not managed by the Kubernetes API Server or Control Plane.

* The `kubelet` automatically creates and manages Static Pods from manifest files stored on the node.

---

# 2. Key Characteristics of Static Pods

## 1. Node-Specific

* Static Pods are tied to a specific node and run only on that node.

* They cannot be scheduled or automatically moved to another node by Kubernetes.

---

## 2. Managed by Kubelet

* Static Pods are managed directly by the `kubelet`.

* The `kubelet` continuously watches the:

```text id="a1b2c3"
/etc/kubernetes/manifests/
```

directory and automatically creates and manages Static Pods from the manifest files.

---

## 3. Visible but Not Fully Controllable

* Static Pods are visible when you run:

```bash id="d4e5f6"
kubectl get pods
```

* But they cannot be updated, scaled, or deleted from the Master Node.

---

## 4. Self-Healing

* If a Static Pod crashes or stops, the `kubelet` automatically restarts it.

---

## 5. No Controllers

* Static Pods are not managed by higher-level controllers like:

  * Deployments
  * ReplicaSets
  * StatefulSets

* They are managed only by the `kubelet`.

---

# 3. How Static Pods Work

* Create a Pod manifest file (`YAML` or `JSON`) inside the kubelet watched directory:

```text id="g7h8i9"
/etc/kubernetes/manifests/
```

* The `kubelet` continuously monitors this directory and automatically creates the Static Pod on that node.

* If the manifest file is updated, the `kubelet` automatically restarts the Pod with the updated configuration.

* If the manifest file is deleted, the `kubelet` automatically stops and removes the Static Pod.


## 4. How Static Pods Work in Kubernetes Step by Step

* Static Pods are managed directly by the kubelet on a node and are not controlled by the Kubernetes API server.  
  Here’s how to create, observe, and delete a Static Pod:

### Step 1: SSH into the Node

```bash
ssh root@<node-ip>
````

### Step 2: Go to the Manifest Directory

```bash
cd /etc/kubernetes/manifests
```

### Step 3: Create a YAML File for the Pod

Example: **nginx-static.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-static
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

### Step 4: Verify from Master

```bash
kubectl get pods -o wide
```

👉 You will see the static pod running on the node.

---

## 5. Steps to Delete a Static Pod

### Step 1: SSH into the Node

```bash
ssh root@<node-ip>
```

### Step 2: Remove the Pod Manifest File

```bash
rm /etc/kubernetes/manifests/nginx-static.yaml
```

### Step 3: Confirm from Master

```bash
kubectl get pods
```

* The pod will no longer appear.
* Kubelet detects the file is gone → it will stop and delete the pod from that node.

---

## 6. Advantages of Static Pods

* Self-healing → If a static pod crashes or is deleted, the Kubelet will automatically restart it.
* Node-specific → A Static Pod belongs to one specific node.

---

## 7. Limitations of Static Pod

* Not scalable (you must copy files to each node manually).
* No features like Deployments/ReplicaSets.
* To update or delete → you must directly edit/remove the file on the node.
* No rolling updates.
* Harder to manage (requires SSH for edits/removals).
* Not suited for regular apps.

```
