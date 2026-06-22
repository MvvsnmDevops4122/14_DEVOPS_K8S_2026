# Kubernetes Pod Scheduling Techniques

# Node Selector, Node Affinity, Pod Affinity & Anti-Affinity

---

# 1. Node Selector (Basic Node Selection)

## Concept

Node Selector is the simplest way to schedule Pods on specific Nodes using Node Labels.

## Real-Time Use Case

Schedule Database Pods only on Nodes with SSD disks.

## Commands

### Label the Node

```bash
kubectl label nodes node1 disk=ssd
```

### Check Node Labels

```bash
kubectl get nodes --show-labels
```

### Pod YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: db-pod
spec:
  nodeSelector:
    disk: ssd
  containers:
  - name: mysql
    image: mysql
```

### Apply Pod

```bash
kubectl apply -f db-pod.yaml
```

## Troubleshooting

```bash
kubectl describe pod db-pod
```

Check:

* Node Labels
* Spelling Errors
* Label Mismatches

### Remove Label

```bash
kubectl label nodes node1 disk-
```

---

# 2. Node Affinity (Advanced Node Selection)

## Concept

Node Affinity is an advanced Kubernetes scheduling feature that allows Pods to be scheduled on specific Nodes based on Node Labels and matching rules.

## Types

### requiredDuringSchedulingIgnoredDuringExecution

* Hard Rule
* Must Match
* No Match → Pending

### preferredDuringSchedulingIgnoredDuringExecution

* Soft Rule
* Tries to Match
* No Match → Scheduled Elsewhere

## Real-Time Use Case

Schedule Pods only on Nodes in a specific Region and Zone.

## Label Nodes

```bash
kubectl label nodes ip-192-168-17-235.ap-south-1.compute.internal region=ap-south-1 zone=ap-south-1c

kubectl label nodes ip-192-168-59-88.ap-south-1.compute.internal region=ap-south-1 zone=ap-south-1a
```

### Verify Labels

```bash
kubectl get nodes --show-labels
```

## Required Node Affinity Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: region
            operator: In
            values:
            - ap-south-1
          - key: zone
            operator: In
            values:
            - ap-south-1c
  containers:
  - name: nginx
    image: nginx
```

### Apply

```bash
kubectl apply -f web-app-pod.yaml
```

## Preferred Node Affinity Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app-pod
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: region
            operator: In
            values:
            - ap-south-1
          - key: zone
            operator: In
            values:
            - ap-south-1b
  containers:
  - name: nginx
    image: nginx
```

## Troubleshooting

```bash
kubectl describe pod web-app-pod
```

Check:

* Node Labels
* Operators
* Typing Mistakes

## Operators

| Operator     | Meaning                      |
| ------------ | ---------------------------- |
| In           | Match Labels in Given List   |
| NotIn        | Exclude Labels in Given List |
| Exists       | Key Must Exist               |
| DoesNotExist | Key Must Not Exist           |
| Gt           | Greater Than                 |
| Lt           | Less Than                    |

---

# 3. Pod Affinity & Pod Anti-Affinity

## Concept

Schedule Pods based on the location of other Pods.

| Feature           | Purpose                |
| ----------------- | ---------------------- |
| Pod Affinity      | Schedule Pods Together |
| Pod Anti-Affinity | Schedule Pods Apart    |

## What is topologyKey?

topologyKey defines the boundary or scope where the Affinity or Anti-Affinity rule should be applied.

### Example

```yaml
topologyKey: kubernetes.io/hostname
```

Meaning:

* Apply the rule at Node Level.

---

# Pod Affinity

## Real-Time Use Case

Web Pod and Redis Pod should run on the same Node for fast communication.

### Frontend Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend-pod
  labels:
    app: frontend
spec:
  containers:
  - name: nginx
    image: nginx
```

### Apply

```bash
kubectl apply -f frontend-pod.yaml
```

### Backend Pod with Pod Affinity

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - frontend
        topologyKey: kubernetes.io/hostname
  containers:
  - name: nginx
    image: nginx
```

### Apply

```bash
kubectl apply -f backend-pod-affinity.yaml
```

### Result

```text
frontend-pod  ---> worker1
backend-pod   ---> worker1
```

---

# Pod Anti-Affinity

## Real-Time Use Case

Distribute application replicas across different Nodes for High Availability.

### Pod Anti-Affinity Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend-pod-anti-affinity
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - frontend
        topologyKey: kubernetes.io/hostname
  containers:
  - name: nginx
    image: nginx
```

### Apply

```bash
kubectl apply -f backend-pod-anti-affinity.yaml
```

### Result

```text
frontend-pod ---> worker1
backend-pod  ---> worker2
```

---

## Pod Affinity Operators

| Operator     | Meaning                      |
| ------------ | ---------------------------- |
| In           | Match Specific Pods          |
| NotIn        | Exclude Specific Pods        |
| Exists       | Match Any Pod with Label Key |
| DoesNotExist | No Pod with Label Key        |

---

## Troubleshooting

```bash
kubectl describe pod <pod-name>
```

Verify:

* Required Pods are Running
* Correct Labels
* Correct topologyKey

---

# Quick Revision Table

| Feature            | Purpose                       | YAML Field                                      |
| ------------------ | ----------------------------- | ----------------------------------------------- |
| Node Selector      | Match Node Labels Exactly     | nodeSelector                                    |
| Node Affinity Hard | Must Match Node Affinity      | requiredDuringSchedulingIgnoredDuringExecution  |
| Node Affinity Soft | Prefer Matching Node Affinity | preferredDuringSchedulingIgnoredDuringExecution |
| Pod Affinity       | Schedule Near Specific Pods   | affinity → podAffinity                          |
| Pod Anti-Affinity  | Avoid Pods on Same Node       | affinity → podAntiAffinity                      |

---

# Learning Flow

1. Node Selector (Simple Scheduling)
2. Node Affinity (Advanced Scheduling)
3. Pod Affinity (Pods Together)
4. Pod Anti-Affinity (Pods Apart)
5. Troubleshooting with kubectl describe

---

# Command Cheat Sheet

| Task              | Command                                 |
| ----------------- | --------------------------------------- |
| Label Node        | kubectl label nodes node-name key=value |
| Check Node Labels | kubectl get nodes --show-labels         |
| Apply YAML        | kubectl apply -f pod.yaml               |
| Describe Pod      | kubectl describe pod pod-name           |
| Remove Label      | kubectl label nodes node-name key-      |

---

# Interview One-Liners

### Node Selector

Node Selector schedules Pods on specific Nodes using exact label matching.

### Node Affinity

Node Affinity is an advanced version of Node Selector that supports matching rules and operators.

### Pod Affinity

Pod Affinity schedules Pods close to other Pods based on labels.

### Pod Anti-Affinity

Pod Anti-Affinity prevents Pods from running on the same Node, improving High Availability.

### topologyKey

topologyKey defines the scope where Affinity or Anti-Affinity rules are applied.
