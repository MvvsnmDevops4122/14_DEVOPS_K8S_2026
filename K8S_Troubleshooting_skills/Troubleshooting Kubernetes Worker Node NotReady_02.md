# Troubleshooting Kubernetes Worker Node NotReady

---

## Problem

```bash
kubectl get nodes
```

Output:

```text
ip-172-31-0-14   NotReady
```

---

## Step 1: Check Node Details

```bash
kubectl describe node <node-name>
```

Look under:

```text
Conditions:
```

Example:

```text
Ready                Unknown
Reason: NodeStatusUnknown
Message: Kubelet stopped posting node status.
```

### Meaning

The control plane is not receiving health updates from the worker node.

---

## Step 2: Check Node Reachability

SSH into the worker node:

```bash
ssh -i <pem-file> ubuntu@<public-ip>
```

### Results

#### If SSH Fails

Possible causes:

* EC2 instance stopped
* Security Group issue
* Network issue
* Instance crash

#### If SSH Works

Node is reachable.

---

## Step 3: Check Kubelet Service

```bash
sudo systemctl status kubelet
```

Expected:

```text
Active: active (running)
```

If stopped:

```bash
sudo systemctl restart kubelet
```

---

## Step 4: Check Container Runtime

```bash
sudo systemctl status containerd
```

Expected:

```text
Active: active (running)
```

If stopped:

```bash
sudo systemctl restart containerd
sudo systemctl restart kubelet
```

---

## Step 5: Check Kubelet Logs

```bash
sudo journalctl -u kubelet -n 100 --no-pager
```

Common errors:

```text
certificate expired
failed to register node
unable to reach API server
container runtime not ready
```

---

## Step 6: Check Disk Usage

```bash
df -h
```

If disk usage reaches:

```text
95% - 100%
```

Kubelet may stop functioning correctly.

---

## Step 7: Check Memory Usage

```bash
free -h
```

```bash
top
```

Look for:

```text
MemoryPressure
OOM
```

---

## Step 8: Verify Connectivity to Control Plane

```bash
ping <control-plane-private-ip>
```

Example:

```bash
ping 172.31.17.111
```

---

## Step 9: Verify Node Status

From Control Plane:

```bash
kubectl get nodes
```

Expected:

```text
STATUS = Ready
```

---

# Important Node Conditions

| Condition          | Meaning            |
| ------------------ | ------------------ |
| Ready              | Node is healthy    |
| NotReady           | Node is unhealthy  |
| MemoryPressure     | Low memory         |
| DiskPressure       | Low disk space     |
| PIDPressure        | Too many processes |
| NetworkUnavailable | Network/CNI issue  |

---

# Common Causes of NotReady Nodes

### 1. Kubelet Stopped

```text
NodeStatusUnknown
Kubelet stopped posting node status
```

### 2. Container Runtime Failure

```text
containerd stopped
```

### 3. Disk Full

```bash
df -h
```

### 4. Memory Exhaustion

```bash
free -h
```

### 5. Network Issue

Worker cannot communicate with Control Plane.

### 6. Node Reboot or Crash

Unexpected restart.

---

# Interview Answer

**Q: A Kubernetes worker node is in NotReady state. How do you troubleshoot it?**

**Answer:**

1. Check node status using `kubectl get nodes`.
2. Describe the node using `kubectl describe node <node-name>`.
3. Review node conditions and events.
4. SSH into the worker node.
5. Verify kubelet service status.
6. Verify container runtime status.
7. Check kubelet logs.
8. Verify disk and memory utilization.
9. Check connectivity to the control plane.
10. Restart kubelet/container runtime if required and confirm the node returns to Ready state.

---

### Key Learning from Your Real Issue

You saw:

```text
Ready = Unknown
Reason = NodeStatusUnknown
Message = Kubelet stopped posting node status
```

This immediately points a DevOps engineer toward:

```bash
systemctl status kubelet
systemctl status containerd
journalctl -u kubelet
```

These are the first commands to check whenever a Kubernetes worker node becomes **NotReady**. 🚀
