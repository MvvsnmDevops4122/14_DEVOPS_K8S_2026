# Taints and Tolerations in Kubernetes — Complete Guide

## Concept

Taints and Tolerations help control where Pods can run in Kubernetes.

### Taint

* Applied on a Node.
* Prevents Pods from being scheduled on that Node unless they have a matching toleration.

### Toleration

* Added to a Pod.
* Allows the Pod to be scheduled on a tainted Node.

---

# Simple Explanation

Taints act like a **"No Entry"** sign on a Node.

Only Pods with a matching **Toleration** (entry pass) can run on that Node.

```text
Node
 ↓
Taint Applied
 ↓
No Entry

Pod
 ↓
Toleration Present
 ↓
Allowed
```

---

# Why We Use Taints and Tolerations

* Reserve Nodes for specific workloads.
* Prevent unwanted Pods from running on specific Nodes.
* Improve resource utilization.
* Achieve workload isolation.
* Protect critical workloads.

### Examples

* Database Nodes
* Production Nodes
* GPU Nodes
* AI/ML Workloads

---

# Before Applying Taints

Kubernetes schedules Pods automatically on any available Node.

### Example Pod

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: normal-pod

spec:
  containers:
  - name: nginx
    image: nginx
```

This Pod can run on any Node.

---

# Applying Taint on a Node

## Syntax

```bash
kubectl taint nodes <node-name> key=value:effect
```

### Example

```bash
kubectl taint nodes node1 env=prod:NoSchedule
```

Verify:

```bash
kubectl describe node node1
```

Output:

```text
Taints:
env=prod:NoSchedule
```

---

# What Happens After Applying Taint?

Node:

```text
node1
 ↓
env=prod:NoSchedule
```

Pod:

```text
No Toleration
```

Result:

```text
Pod Status = Pending
```

Error:

```text
node(s) had untolerated taint(s)
```

Meaning:

The Node has a taint, but the Pod does not have a matching toleration.

---

# Pod with Matching Toleration

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: toleration-pod

spec:
  tolerations:
  - key: "env"
    operator: "Equal"
    value: "prod"
    effect: "NoSchedule"

  containers:
  - name: nginx
    image: nginx
```

Apply:

```bash
kubectl apply -f pod.yaml
```

Result:

```text
Pod Scheduled Successfully
```

---

# How Matching Works

## Node Taint

```text
env=prod:NoSchedule
```

## Pod Toleration

```yaml
tolerations:
- key: "env"
  operator: "Equal"
  value: "prod"
  effect: "NoSchedule"
```

Result:

```text
Match Found
↓
Pod Allowed
```

---

# Practical Use Case

## Dedicated Database Node

Apply Taint:

```bash
kubectl taint nodes db-node db=exclusive:NoSchedule
```

Database Pod:

```yaml
tolerations:
- key: "db"
  operator: "Equal"
  value: "exclusive"
  effect: "NoSchedule"
```

Result:

```text
Database Pods → Allowed

Normal Pods → Blocked
```

---

# Types of Taint Effects

## 1. NoSchedule

### Definition

Blocks Pods without matching tolerations from being scheduled.

Example:

```bash
kubectl taint nodes node1 env=prod:NoSchedule
```

### Result

```text
New Pods Without Toleration
↓
Not Scheduled
```

---

## 2. PreferNoSchedule

### Definition

Kubernetes tries to avoid scheduling Pods without tolerations.

Example:

```bash
kubectl taint nodes node1 env=prod:PreferNoSchedule
```

### Result

```text
Try to Avoid Scheduling
But Not Guaranteed
```

---

## 3. NoExecute

### Definition

* Blocks new Pods.
* Removes existing Pods without matching tolerations.

Example:

```bash
kubectl taint nodes node1 env=prod:NoExecute
```

### Result

```text
Existing Pods
↓
Evicted

New Pods
↓
Blocked
```

---

# Does Toleration Force a Pod onto a Node?

## Answer

No.

Toleration only allows a Pod to run on a tainted Node.

It does not guarantee scheduling on that Node.

### Example

```text
Toleration
     ↓
Permission
```

To force scheduling on a specific Node, use:

```yaml
nodeSelector
```

or

```yaml
nodeAffinity
```

along with tolerations.

---

# Difference Between NodeSelector and Taints/Tolerations

## NodeSelector

```text
Attract Pods to Nodes
```

Example:

```yaml
nodeSelector:
  env: prod
```

---

## Taints and Tolerations

```text
Repel Pods from Nodes
```

Example:

```bash
kubectl taint nodes node1 env=prod:NoSchedule
```

### Interview Answer

NodeSelector attracts Pods to specific Nodes, whereas Taints and Tolerations prevent Pods from being scheduled unless they explicitly tolerate the taint.

---

# Remove Taint

## Syntax

```bash
kubectl taint nodes <node-name> key=value:effect-
```

### Example

```bash
kubectl taint nodes node1 env=prod:NoSchedule-
```

Verify:

```bash
kubectl describe node node1
```

Output:

```text
Taints: <none>
```

---

# Learning Flow

### Step 1

Deploy a normal Pod.

```text
Pod
 ↓
Runs on Any Node
```

### Step 2

Apply Taint.

```text
Node
 ↓
Tainted
```

### Step 3

Deploy Pod Again.

```text
Pod
 ↓
Pending
```

### Step 4

Add Toleration.

```text
Pod
 ↓
Running
```

### Step 5

Remove Taint.

```text
Node
 ↓
Normal Scheduling
```

---

# Common Commands

| Task             | Command                                        |
| ---------------- | ---------------------------------------------- |
| Add Taint        | kubectl taint nodes node1 env=prod:NoSchedule  |
| Remove Taint     | kubectl taint nodes node1 env=prod:NoSchedule- |
| View Taints      | kubectl describe node node1                    |
| Deploy Pod       | kubectl apply -f pod.yaml                      |
| Check Pod Events | kubectl describe pod pod-name                  |
| View Nodes       | kubectl get nodes                              |

---

# Frequently Asked Interview Questions

### What are Taints?

Taints are applied on Nodes to restrict Pod scheduling.

---

### What are Tolerations?

Tolerations are applied on Pods to allow them to run on tainted Nodes.

---

### What is NoSchedule?

Prevents Pods from being scheduled unless they have matching tolerations.

---

### What is PreferNoSchedule?

A soft restriction that tries to avoid scheduling Pods.

---

### What is NoExecute?

Blocks new Pods and evicts existing Pods that do not tolerate the taint.

---

### Why do we use Taints and Tolerations?

To dedicate Nodes for specific workloads such as databases, production applications, or GPU workloads.

---

# One-Line Summary

Taints are applied on Nodes to restrict Pod scheduling, and Tolerations are applied on Pods to allow scheduling onto tainted Nodes.
