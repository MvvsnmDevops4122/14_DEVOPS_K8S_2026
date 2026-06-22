# Kubernetes Node Maintenance

# Detailed Guide on Cordon, Drain, Uncordon

---

# Concept

In Kubernetes, Nodes sometimes need to be temporarily removed from scheduling for:

* OS Patching
* Kernel Updates
* Hardware Replacement
* Troubleshooting
* Node Reboots
* Kubernetes Upgrades

For such situations, Kubernetes provides three important commands:

1. Cordon
2. Drain
3. Uncordon

---

# 1. Cordon

## Purpose

Prevent new Pods from being scheduled on a Node.

Existing Pods continue running.

## Command

```bash
kubectl cordon <node-name>
```

## Example

```bash
kubectl cordon ip-192-168-59-88.ap-south-1.compute.internal
```

## What Happens?

* Node becomes Unschedulable.
* Existing Pods continue running.
* New Pods cannot be scheduled on the Node.

## Verify

```bash
kubectl get nodes
```

Example:

```text
NAME                                           STATUS
ip-192-168-59-88.ap-south-1.compute.internal   Ready,SchedulingDisabled
```

## Interview One-Liner

> Cordon marks a Node as unschedulable so that new Pods cannot be scheduled on it while existing Pods continue running.

---

# 2. Drain

## Purpose

Safely evict all Pods from a Node and prepare it for maintenance.

## Command

```bash
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

## Example

```bash
kubectl drain ip-192-168-59-88.ap-south-1.compute.internal \
--ignore-daemonsets \
--delete-emptydir-data
```

## Parameters

### --ignore-daemonsets

Ignores DaemonSet Pods such as:

* kube-proxy
* node-exporter
* monitoring agents

These Pods are managed automatically by Kubernetes.

### --delete-emptydir-data

Deletes Pods using emptyDir volumes.

Required for complete node draining.

## What Happens?

* All application Pods are evicted safely.
* Pod Disruption Budgets are respected.
* DaemonSet Pods remain.
* Node remains in SchedulingDisabled state.

## Verify

```bash
kubectl get pods -o wide
```

Pods should move to another available node.

## Common Error

```text
cannot delete Pod because it would violate PodDisruptionBudget
```

## Interview One-Liner

> Drain safely evicts all application Pods from a Node and marks the Node unschedulable for maintenance activities.

---

# 3. Uncordon

## Purpose

Make the Node available again for new Pods to be scheduled.

## Command

```bash
kubectl uncordon <node-name>
```

## Example

```bash
kubectl uncordon ip-192-168-59-88.ap-south-1.compute.internal
```

## Result

Node status changes back to:

```text
Ready
```

New Pods can now be scheduled on the Node.

## Verify

```bash
kubectl get nodes
```

## Interview One-Liner

> Uncordon makes a cordoned or drained Node schedulable again, allowing new Pods to be placed on it.

---

# Complete Workflow

## Step 1: Cordon Node

```bash
kubectl cordon <node-name>
```

Stop scheduling new Pods.

---

## Step 2: Drain Node

```bash
kubectl drain <node-name> \
--ignore-daemonsets \
--delete-emptydir-data
```

Move all application Pods to other Nodes.

---

## Step 3: Perform Maintenance

Examples:

* OS Patching
* Kubernetes Upgrade
* Disk Cleanup
* Hardware Maintenance
* Troubleshooting

---

## Step 4: Uncordon Node

```bash
kubectl uncordon <node-name>
```

Allow new Pods to be scheduled again.

---

# Real-Time Use Cases

| Scenario             | Solution                            |
| -------------------- | ----------------------------------- |
| Kubernetes Upgrade   | cordon → drain → upgrade → uncordon |
| OS Patching          | cordon → drain → patch → uncordon   |
| Node Reboot          | cordon → drain → reboot → uncordon  |
| Hardware Replacement | cordon → drain → replace → uncordon |
| Troubleshooting      | cordon → drain → debug → uncordon   |

---

# Command Cheat Sheet

| Task                | Command                                                            |
| ------------------- | ------------------------------------------------------------------ |
| Cordon Node         | kubectl cordon node-name                                           |
| Drain Node          | kubectl drain node-name --ignore-daemonsets --delete-emptydir-data |
| Uncordon Node       | kubectl uncordon node-name                                         |
| Check Nodes         | kubectl get nodes                                                  |
| Check Pod Placement | kubectl get pods -o wide                                           |
| Check PDB           | kubectl get poddisruptionbudgets                                   |
| Describe Node       | kubectl describe node node-name                                    |

---

# Important Notes

### Cordon

* Existing Pods continue running.
* New Pods cannot be scheduled.

### Drain

* Existing Pods are evicted.
* DaemonSet Pods remain.
* Node becomes unschedulable.

### Uncordon

* Node becomes schedulable again.
* New Pods can be placed on the Node.

---

# Troubleshooting

## Check Pod Disruption Budgets

```bash
kubectl get poddisruptionbudgets
```

Pods protected by PDB may block draining.

---

## Force Drain

```bash
kubectl drain <node-name> \
--force \
--ignore-daemonsets \
--delete-emptydir-data
```

Use carefully in Production.

---

## Check Events

```bash
kubectl describe node <node-name>
```

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

# Quick Revision

| Command  | Purpose              |
| -------- | -------------------- |
| Cordon   | Stop New Pods        |
| Drain    | Move Existing Pods   |
| Uncordon | Allow New Pods Again |

---

# Easy Memory Trick

```text
Cordon
↓
Stop New Pods

Drain
↓
Move Existing Pods

Uncordon
↓
Allow New Pods Again
```

---

# Interview Questions

## What is Cordon?

Cordon marks a Node as unschedulable so that new Pods cannot be scheduled on it.

---

## What is Drain?

Drain safely evicts all running application Pods from a Node and prepares it for maintenance.

---

## What is Uncordon?

Uncordon makes a cordoned or drained Node schedulable again.

---

## Difference Between Cordon and Drain

| Cordon                     | Drain                                 |
| -------------------------- | ------------------------------------- |
| Stops New Pods             | Stops New Pods + Evicts Existing Pods |
| Existing Pods Stay Running | Existing Pods Move to Other Nodes     |
| Used Before Maintenance    | Used During Maintenance               |

---

## Difference Between Drain and Uncordon

| Drain                    | Uncordon               |
| ------------------------ | ---------------------- |
| Makes Node Unschedulable | Makes Node Schedulable |
| Evicts Pods              | Allows New Pods        |
| Maintenance Mode         | Back to Normal State   |
