# Kubernetes Namespace

---

# 1. What is a Kubernetes Namespace?

- A Namespace in Kubernetes is a logical partition used to organize and isolate cluster resources like Pods, Services, Deployments, ConfigMaps, and Secrets.

- It acts like a virtual cluster inside the main Kubernetes cluster, helping manage multiple teams, projects, or environments such as Dev, Test, and Production.
  
<img width="995" height="504" alt="image" src="https://github.com/user-attachments/assets/7cf07ae3-0fed-46ae-98bf-4af5660138c9" />

---

## 2. What Namespaces Provide

# 2. What Namespaces Provide

## 1. Isolation

Namespaces keep resources separate inside the same Kubernetes cluster.

### Practical Example

Suppose:

* Dev team deploys application in `dev` namespace
* Production app runs in `prod` namespace

If Dev team deletes pods in `dev`, Production application in `prod` is not affected.

## 2. Access Control (RBAC)

Namespaces help control who can access resources.

### Practical Example

* Developers get access only to `dev`
* QA team gets access to `test`
* Admin team manages `prod`

This improves security.

## 3. Resource Quotas

Namespaces allow setting CPU and memory limits.

### Practical Example

Suppose Dev namespace gets:

* 2 CPU
* 4GB RAM

Even if developers deploy many pods, they cannot use more than assigned resources.

This protects Production workloads.  

---

# 3. Types of Namespaces in Kubernetes

## 1. `default` Namespace

* This is the default namespace in Kubernetes.
* If you do not mention any namespace while creating resources, Kubernetes automatically creates them here.

### Practical Example

```bash id="8tkzcv"
kubectl run nginx --image=nginx
```

The pod will be created in the `default` namespace.

## 2. `kube-system` Namespace

* Used by Kubernetes internal system components.
* Contains important cluster services and control plane components.
* Usually administrators manage this namespace.

### Examples

* CoreDNS
* kube-apiserver
* kube-proxy
* etcd

### Practical Example

```bash id="f8j5m7"
kubectl get pods -n kube-system
```

Shows all Kubernetes system pods.

## 3. `kube-public` Namespace

* Public namespace accessible to all users.
* Mainly used for public cluster-related information.

### Practical Example

Stores information that should be visible across the cluster.

## 4. `kube-node-lease` Namespace

* Used to store node lease objects.
* Helps Kubernetes monitor node health and availability.
* Reduces load on the Kubernetes control plane.

### Practical Example

Worker nodes regularly send heartbeat updates through node leases.

If a node stops responding, Kubernetes detects node failure quickly.

## 5. User-defined Namespace

* Custom namespaces created by users.
* Used to separate applications, teams, or environments.

### Examples

* `dev`
* `test`
* `prod`

### Practical Example

Create namespace:

```bash id="6gvf7z"
kubectl create namespace dev
```

Deploy application into namespace:

```bash id="ymnaxv"
kubectl apply -f app.yaml -n dev
```

Application runs only inside the `dev` namespace.

# Simple Understanding

| Namespace         | Purpose                        |
| ----------------- | ------------------------------ |
| `default`         | Default workspace              |
| `kube-system`     | Kubernetes internal components |
| `kube-public`     | Public cluster information     |
| `kube-node-lease` | Node health tracking           |
| User-defined      | Separate environments/projects |

# One-Line Interview Answer

“Kubernetes provides system namespaces for internal operations and user-defined namespaces for organizing and isolating applications.”

---

# 4. How to Work with Namespaces

## Creating a Namespace

### Using YAML

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
  labels:
    name: my-namespace
```

Apply the file:

```bash id="dj3m4w"
kubectl apply -f namespace.yaml
```

### Using Command Line

```bash id="j9s9ta"
kubectl create namespace test
```

---

## 1. Listing Namespaces

```bash id="k4n4uz"
kubectl get namespaces
```

OR

```bash id="j2vw2f"
kubectl get ns
```

## 2. Viewing Namespace Details

```bash id="5lw0pr"
kubectl describe ns my-namespace
```

## 3. Deleting a Namespace

⚠️ Warning: Deletes all resources inside the namespace.

```bash id="vjlwm3"
kubectl delete namespace my-namespace
```

---

# 5. Namespace Resource Quotas

* Limit resources (CPU, memory, pods) in a namespace.
* Example YAML to limit pods to 10:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
  namespace: my-namespace
spec:
  hard:
    pods: "10"
```

Apply the quota:

```bash id="0uq0o6"
kubectl apply -f resource-quota.yaml
```

Check quota:

```bash id="1jlwm5"
kubectl describe ns my-namespace
```

---

## Creating a Pod in the Namespace to Test Quota

```bash id="x6d5y8"
kubectl run nginx --image=nginx --port=80 -n my-namespace
```

Check quota again:

```bash id="mjlwm7"
kubectl describe ns my-namespace
```

Edit the quota:

```bash id="b4k1h2"
kubectl edit quota my-resource-quota -n my-namespace
```

### Example Output

```text id="q9t6r1"
Pods: 1/10 — 1 pod used, 9 left.
```

---

# 6. Switching Namespaces in kubectl Context

Set your current namespace context so commands default to it:

```bash id="r7u3c8"
kubectl config set-context --current --namespace=my-namespace
```

---
