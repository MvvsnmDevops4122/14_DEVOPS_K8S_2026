# Kubernetes Init Containers & Sidecar Containers

## 1. Init Containers Example (Pre-Start Tasks)

### Purpose

Init Containers run before the main application container starts.

They are useful for:

* Waiting for services
* Setting up configurations
* Pre-start checks
* Database migrations
* Dependency validation

---

## YAML: Init Container Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-container-demo
  namespace: test-ns

spec:
  containers:
  - name: main-app
    image: busybox
    command:
    - sh
    - -c
    - echo Main App Running; sleep 3600

  initContainers:
  - name: init-wait
    image: busybox
    command:
    - sh
    - -c
    - echo Init Task Starting; sleep 10; echo Init Task Done
```

---

## Commands to Apply & Verify

### Deploy Pod

```bash
kubectl apply -f init-container.yaml
```

### Monitor Pod Status

```bash
kubectl get pods -n test-ns -w
```

### Check Init Container Logs

```bash
kubectl logs init-container-demo -n test-ns -c init-wait
```

### Check Main Container Logs

```bash
kubectl logs init-container-demo -n test-ns -c main-app
```

---

## Explanation

```text
Pod Created
     ↓
Init Container Starts
     ↓
Waits 10 Seconds
     ↓
Completes Successfully
     ↓
Main Application Starts
```

* Init Container runs first.
* Main Container waits until Init Container completes.
* If Init Container fails, Main Container will not start.

---

# 2. Sidecar Containers Example (Run Together)

## Purpose

Sidecar Containers run alongside the main application container.

They are useful for:

* Logging
* Monitoring
* Proxy Services
* Configuration Sync
* Service Mesh

---

## YAML: Sidecar Container Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-container-demo
  namespace: test-ns

spec:
  volumes:
  - name: shared-logs
    emptyDir: {}

  containers:

  - name: main-app
    image: busybox
    command:
    - sh
    - -c
    - while true; do echo $(date) Writing Logs >> /logs/app.log; sleep 5; done

    volumeMounts:
    - name: shared-logs
      mountPath: /logs

  - name: sidecar-log-watcher
    image: busybox
    command:
    - sh
    - -c
    - tail -f /logs/app.log

    volumeMounts:
    - name: shared-logs
      mountPath: /logs
```

---

## Commands to Apply & Verify

### Deploy Pod

```bash
kubectl apply -f sidecar-container.yaml
```

### Monitor Pod Status

```bash
kubectl get pods -n test-ns -w
```

### Check Sidecar Logs

```bash
kubectl logs sidecar-container-demo -n test-ns -c sidecar-log-watcher
```

### Check Main Container Logs

```bash
kubectl logs sidecar-container-demo -n test-ns -c main-app
```

### Verify Log File Inside Container

```bash
kubectl exec -it sidecar-container-demo -n test-ns -c main-app -- sh

cat /logs/app.log
```

---

## Explanation

### Shared Volume

```yaml
volumes:
- name: shared-logs
  emptyDir: {}
```

Creates a temporary shared volume.

---

### Main Container

```text
Writes Logs
      ↓
/logs/app.log
      ↓
Every 5 Seconds
```

Example:

```text
Wed Jun 18 10:00:00 Writing Logs
Wed Jun 18 10:00:05 Writing Logs
Wed Jun 18 10:00:10 Writing Logs
```

---

### Sidecar Container

```text
Reads /logs/app.log
      ↓
Displays Logs Continuously
```

Command:

```bash
tail -f /logs/app.log
```

---

## Sidecar Flow

```text
Main Container
      ↓
Writes Logs
      ↓
Shared Volume
      ↓
Sidecar Container
      ↓
Reads Logs
      ↓
Displays Logs
```

---

# Key Differences Between Init & Sidecar Containers

| Feature                           | Init Container               | Sidecar Container          |
| --------------------------------- | ---------------------------- | -------------------------- |
| When Runs                         | Before Main Container Starts | Alongside Main Container   |
| Purpose                           | Setup Tasks                  | Support Tasks              |
| Runs Together with Main Container | No                           | Yes                        |
| Lifecycle                         | Runs Once and Exits          | Runs Continuously          |
| Common Uses                       | DB Migration, Pre-checks     | Logging, Monitoring, Proxy |
| Must Complete Before App Starts   | Yes                          | No                         |

---

# Visual Flow

```text
+-------------------+
| Init Container(s) |
+-------------------+
          |
          v
+---------------------------+
| Main Application Starts   |
+---------------------------+
          |
          v
+---------------------------+
| Sidecar Container Running |
+---------------------------+
```

---

# Real-Time Examples

## Init Container

* Wait for Database
* Download Configuration Files
* Perform Database Migration
* Validate Dependencies

Example:

```text
Init Container
      ↓
Check Database Availability
      ↓
Database Ready
      ↓
Start Application
```

---

## Sidecar Container

* Fluentd Log Collector
* Istio Proxy
* Monitoring Agent
* Security Scanner

Example:

```text
Application
      ↓
Writes Logs
      ↓
Fluentd Sidecar
      ↓
ELK Stack
```

---

# Useful Commands Cheat Sheet

### Monitor Pods

```bash
kubectl get pods -n test-ns -w
```

### View Container Logs

```bash
kubectl logs <pod-name> -n test-ns -c <container-name>
```

### Access Container

```bash
kubectl exec -it <pod-name> -n test-ns -c <container-name> -- sh
```

### Describe Pod

```bash
kubectl describe pod <pod-name> -n test-ns
```

---

# Interview Answers

## What is an Init Container?

> An Init Container is a special container that runs before the main application container starts. It is used for initialization tasks such as waiting for dependencies, downloading configurations, or performing pre-start checks. The main container starts only after all Init Containers complete successfully.

---

## What is a Sidecar Container?

> A Sidecar Container is a helper container that runs alongside the main application container in the same Pod. It provides supporting services such as logging, monitoring, configuration synchronization, and service proxy functionality.

---

# Easy Memory Trick

```text
Init Container
--------------
Runs First
Completes
Exits

Sidecar Container
-----------------
Runs With Main App
Keeps Running
Provides Support
```
