# Kubernetes Requests and Limits

## Introduction

In Kubernetes, resource management is used to control how much CPU and memory a container can use.

The two main resources commonly managed are:

* CPU
* Memory (RAM)

---

# What Are Resource Requests?

Resource requests define the minimum guaranteed CPU and memory resources required for a container to run properly inside a pod.

Kubernetes scheduler uses resource request values to place the Pod on a suitable node.

## Example

```yaml id="t4m8x1"
resources:
  requests:
    cpu: "500m"
    memory: "256Mi"
```

## Meaning

| Resource      | Meaning                       |
| ------------- | ----------------------------- |
| cpu: 500m     | Minimum 0.5 CPU required      |
| memory: 256Mi | Minimum 256Mi memory required |

---

# What Are Resource Limits?

Resource limits define the maximum amount of CPU and memory a container is allowed to use inside a pod.

Kubernetes prevents the container from using more than the defined limit.

## Example

```yaml id="m7x2v5"
resources:
  limits:
    cpu: "1"
    memory: "512Mi"
```

## Meaning

| Resource      | Meaning                      |
| ------------- | ---------------------------- |
| cpu: 1        | Maximum 1 CPU allowed        |
| memory: 512Mi | Maximum 512Mi memory allowed |

---

# Deploying with Requests Only (No Limits)

If only requests are defined and limits are not specified:

* Resource requests define the minimum guaranteed CPU and memory resources required for a container to run properly inside a pod.
* Container can use more resources if available on the node.

## Example YAML: reqonly.yaml

```yaml id="x5m1v8"
apiVersion: apps/v1
kind: Deployment

metadata:
  name: javawebappdeploy
  namespace: test

spec:
  replicas: 2

  selector:
    matchLabels:
      app: javawebapp

  template:
    metadata:
      labels:
        app: javawebapp

    spec:
      containers:
      - name: javawebappcon
        image: satyamolleti4599/maven_web_app:1.0.0

        resources:
          requests:
            memory: "256Mi"
            cpu: "600m"

        ports:
        - containerPort: 8080

---

apiVersion: v1
kind: Service

metadata:
  name: javawebapp-svc
  namespace: test

spec:
  type: NodePort

  selector:
    app: javawebapp

  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080
```

---

# Commands

## Deploy Application

```bash id="n2x7m4"
kubectl apply -f reqonly.yaml
```

## Check Pods

```bash id="v8m1x5"
kubectl get pods -n test-ns
```

## Check Node Resource Allocation

```bash id="p4m7x2"
kubectl describe node <node-name>
```

---

# Observation

Under node allocation:

* Requests values are visible
* Limits show as 0 because limits are not configured

---

# Practical Understanding

Your Deployment requests:

| Resource | Per Pod |
| -------- | ------- |
| CPU      | 600m    |
| Memory   | 256Mi   |

Total requested for 2 Pods:

| Resource | Total           |
| -------- | --------------- |
| CPU      | 1200m (1.2 CPU) |
| Memory   | 512Mi           |

Your node capacity:

| Resource | Capacity      |
| -------- | ------------- |
| CPU      | 2 CPU (2000m) |
| Memory   | ~900Mi        |

So Kubernetes can still schedule Pods successfully.

---

# Creating Pending State Practically

Increase requests beyond node capacity.

## Use This YAML

```yaml id="q1m8x4"
resources:
  requests:
    memory: "900Mi"
    cpu: "1500m"
```

With:

```yaml id="r7v2m5"
replicas: 2
```

---

# Resource Calculation

Each Pod requests:

* 1500m CPU
* 900Mi memory

For 2 Pods:

| Resource | Total  |
| -------- | ------ |
| CPU      | 3000m  |
| Memory   | 1800Mi |

But node has only:

| Resource | Available |
| -------- | --------- |
| CPU      | 2000m     |
| Memory   | ~900Mi    |

Now scheduler cannot place Pods.

---

# Expected Result

```bash id="k4m7x1"
kubectl get po -n test
```

Output:

```text id="x8v2m5"
Pending
```

---

# Verify Reason

```bash id="w1m7x4"
kubectl describe po <pod-name> -n test
```

Events section:

```text id="n5v2x8"
Insufficient cpu
Insufficient memory
```

---

# Best Practical Understanding

```text id="m2x7v4"
Requested Resources > Available Node Resources = Pod Pending
```

Kubernetes scheduler uses resource request values during Pod scheduling. If sufficient CPU or memory is not available on any node, the Pod remains in Pending state.

---

# Deploying with Requests and Limits

## Example YAML: reqlimits.yaml

```yaml id="t8m1x5"
resources:
  requests:
    memory: "256Mi"
    cpu: "600m"

  limits:
    memory: "256Mi"
    cpu: "900m"
```

---

# Meaning

## Requests

Minimum guaranteed resources:

* 600m CPU
* 256Mi memory

## Limits

Maximum allowed resources:

* 900m CPU
* 256Mi memory

---

# Commands

## Deploy Application

```bash id="y4m8x2"
kubectl apply -f limits.yaml
```

## Check Pods

```bash id="h7x1m4"
kubectl get pods -n test
```

## Check Node Allocation

```bash id="p2m8x5"
kubectl describe node <node-name>
```

---

# Observation

Under:

```text id="v1x7m4"
Allocated resources
```

You can see:

* Requests values
* Limits values

---

# Real-Time Practical for Resource Limit Exceed

## Goal

Create a real-time practical situation where:

* Container exceeds memory limit
* Kubernetes kills the container
* Pod status becomes OOMKilled
* Container continuously restarts
* Pod enters CrashLoopBackOff

---

# Step 1 — Create YAML File

```bash id="m8v2x4"
vi limits.yaml
```

Paste this YAML:

```yaml id="q5m7x1"
apiVersion: apps/v1
kind: Deployment

metadata:
  name: stress-deploy
  namespace: test

spec:
  replicas: 1

  selector:
    matchLabels:
      app: stress-app

  template:
    metadata:
      labels:
        app: stress-app

    spec:
      containers:
      - name: stress-container
        image: polinux/stress

        resources:
          requests:
            memory: "128Mi"
            cpu: "200m"

          limits:
            memory: "256Mi"
            cpu: "500m"

        command:
        - stress

        args:
        - "--vm"
        - "1"
        - "--vm-bytes"
        - "300M"
        - "--cpu"
        - "2"
```

---

# What This Container Does

This stress container intentionally:

* Uses 300Mi memory
* Uses high CPU with 2 CPU workers

---

# Configured Limits

| Resource | Limit |
| -------- | ----- |
| Memory   | 256Mi |
| CPU      | 500m  |

---

# Expected Result

## Memory Behavior

Container tries to use:

```text id="x4m7v1"
300Mi memory
```

But limit is:

```text id="j8m2x5"
256Mi
```

Result:

```text id="w2x7m4"
OOMKilled
```

---

## CPU Behavior

Container tries to use more than:

```text id="v5m1x8"
500m CPU
```

Result:

* Kubernetes throttles CPU usage
* Container continues running

---

# Step 2 — Deploy Application

```bash id="q7m2x4"
kubectl apply -f limits.yaml
```

---

# Step 3 — Verify Pod Status

```bash id="m1v8x5"
kubectl get po -n test -w
```

Expected:

```text id="p8m4x1"
Running
CrashLoopBackOff
```

OR

```text id="r2x7m5"
OOMKilled
```

---

# Step 4 — Verify OOMKilled

```bash id="x7m1v4"
kubectl describe po <pod-name> -n test
```

Under:

```text id="v4m8x2"
Last State
```

You can see:

```text id="k5x2m7"
Reason: OOMKilled
```

---

# Step 5 — Check CPU Throttling

```bash id="t1m7x4"
kubectl top po -n test
```

You will notice:

* CPU usage restricted near limit
* Container not killed due to CPU

---

# Real-Time Understanding

## Memory Limit Exceed

```text id="n7m2x5"
Memory limit exceeded
        ↓
Kubernetes kills container
        ↓
OOMKilled
        ↓
CrashLoopBackOff
```

---

## CPU Limit Exceed

```text id="x2m8v4"
CPU limit exceeded
        ↓
CPU throttling
        ↓
Container continues running
```

---

# Important Difference

| Resource              | Behavior       |
| --------------------- | -------------- |
| CPU limit exceeded    | CPU throttling |
| Memory limit exceeded | OOMKilled      |

---

# CrashLoopBackOff

CrashLoopBackOff can occur when a container continuously crashes due to memory limit exceed (OOMKilled), and Kubernetes repeatedly restarts it.

---

# Interview Questions and Answers

## What happens if requests exceed node capacity?

If requested CPU or memory is greater than available node resources, Kubernetes scheduler cannot place the Pod, and the Pod remains in Pending state.

---

## Why does a Pod go into Pending state?

A Pod goes into Pending state when Kubernetes cannot find a suitable node with sufficient available resources like CPU or memory.

---

## What happens if memory limit exceeds?

If a container exceeds its memory limit, Kubernetes kills the container and the Pod may show OOMKilled status.

---

## What happens if CPU limit exceeds?

If a container exceeds its CPU limit, Kubernetes throttles CPU usage instead of killing the container.

---

## What is CPU throttling?

CPU throttling means Kubernetes restricts CPU usage when a container tries to use more CPU than the configured limit.

---

## What is OOMKilled?

OOMKilled means Kubernetes killed the container because it exceeded the configured memory limit.
