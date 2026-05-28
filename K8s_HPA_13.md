
# Kubernetes Horizontal Pod Autoscaler (HPA)

---

# 1. What is HPA?

Horizontal Pod Autoscaler (HPA) in Kubernetes automatically increases or decreases the number of Pods in a Deployment, ReplicaSet, or StatefulSet based on resource usage like:

* CPU utilization
* Memory utilization
* Custom metrics
* External metrics

---

# 2. Why HPA is Used?

## Without HPA

* Traffic increases
* Existing Pods become overloaded
* Application becomes slow or crashes

Without HPA, Pods must be scaled manually:

```bash
kubectl scale deployment <DEPLOYMENT_NAME> --replicas=<NUMBER>
```

---

## With HPA

* Kubernetes automatically adds more Pods during high load
* Removes extra Pods during low load
* Saves resources and improves performance

---

# 3. Real-Time Example

Suppose:

* Java application normally runs with 2 Pods
* CPU usage suddenly increases to 80%

HPA automatically scales:

```text
2 → 4 → 6 Pods
```

When traffic decreases:

* HPA automatically scales down Pods

---

# 4. Types of Scaling in Kubernetes

## 4.1 Horizontal Scaling (HPA)

Horizontal Pod Autoscaler (HPA) automatically increases or decreases the number of Pods based on:

* CPU utilization
* Memory utilization
* Custom metrics
* External metrics

### Example

```text
2 Pods → 4 Pods when CPU > target
```

### Important Point

* Requires Metrics Server

### Best Use Case

* Best for stateless applications with variable traffic

---

## 4.2 Vertical Scaling (VPA)

Vertical Pod Autoscaler (VPA) increases or decreases Pod resources like:

* CPU
* Memory

### Example

```text
CPU: 200m → 500m
Memory: 512Mi → 2Gi
```

### Important Point

* Does not increase Pod count

### Best Use Case

* Best for stateful applications needing more CPU or memory

---

# 5. Kubernetes HPA Demo (Step-by-Step)

# 5.1 Check Metrics Availability

```bash
kubectl top nodes
kubectl top po -A
```

If you see:

```text
error: Metrics API not available
```

It means:

```text
Metrics Server is not installed
```

---

# 5.2 Install Metrics Server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Verify installation:

```bash
kubectl get all -n kube-system
```

If Metrics Server Pod shows:

```text
0/1 Running
```

Most likely:

```text
TLS issue between Metrics Server and Kubelet
```

---

# 5.3 Fix Metrics Server TLS Issue

Edit Metrics Server Deployment:

```bash
kubectl edit deploy metrics-server -n kube-system
```

Under:

```yaml
spec:
  containers:
  - args:
```

Add:

```yaml
- --cert-dir=/tmp
- --secure-port=10250
- --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
- --kubelet-use-node-status-port
- --metric-resolution=15s
- --kubelet-insecure-tls
```

Save and exit.

Verify again:

```bash
kubectl get pods -n kube-system
```

---

# 5.4 Verify Metrics

```bash
kubectl top nodes
kubectl top po -A
```

Now CPU and Memory metrics should appear.

---

# 6. Deploy HPA Demo Application

Create file:

```text
hpa-demo.yaml
```

This file contains:

* Deployment
* HPA
* Service

---

# 6.1 Deployment

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: my-deployment
  namespace: test

spec:
  replicas: 1

  selector:
    matchLabels:
      app: my-app

  template:
    metadata:
      labels:
        app: my-app

    spec:
      containers:
      - name: my-container
        image: satyamolleti4599/maven-web-app:1.0.0

        ports:
        - containerPort: 8080

        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"

          limits:
            memory: "128Mi"
            cpu: "100m"
```

---

## Important Points

### Initially creates:

```text
1 Pod
```

### Resource requests are mandatory for HPA

HPA calculates scaling using:

```text
Current Usage / Requested Resources
```

---

# 6.2 HPA Configuration

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler

metadata:
  name: my-hpa
  namespace: test

spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-deployment

  minReplicas: 2
  maxReplicas: 5

  metrics:
  - type: Resource

    resource:
      name: cpu

      target:
        type: Utilization
        averageUtilization: 30

  - type: Resource

    resource:
      name: memory

      target:
        type: Utilization
        averageUtilization: 80
```

---

# Important Points

## minReplicas: 2

```text
HPA maintains minimum 2 Pods
```

---

## maxReplicas: 5

```text
HPA cannot scale beyond 5 Pods
```

---

## Scaling Conditions

* CPU > 30%
* Memory > 80%

---

# 6.3 Service

```yaml
apiVersion: v1
kind: Service

metadata:
  name: hpaclusterservice

spec:
  selector:
    app: my-app

  ports:
  - port: 80
    targetPort: 8080

  type: ClusterIP
```

---

# Important Correction

Deployment label:

```yaml
app: my-app
```

So Service selector must also be:

```yaml
app: my-app
```

Otherwise Service cannot route traffic to Pods.

---

# Apply the YAML

```bash
kubectl apply -f hpa-demo.yaml
```

Verify resources:

```bash
kubectl get all -n test
```

Check HPA:

```bash
kubectl get hpa -n test
```

---

# 7. When Will Scaling Happen?

Scaling occurs when:

| Metric | Condition        |
| ------ | ---------------- |
| CPU    | Greater than 30% |
| Memory | Greater than 80% |

Example:

```text
2 Pods → 3 Pods → 4 Pods → 5 Pods
```

(Maximum = 5 Pods)

---

# 8. Watch Autoscaling Live

Open two terminals.

## Terminal 1

```bash
watch kubectl get po -n test
```

---

## Terminal 2

```bash
watch kubectl get hpa -n test
```

Initially:

* 2 Pods running
* CPU/Memory usage low
* No scaling occurs

---

# 9. Generate Load

Create BusyBox load generator:

```bash
kubectl run -it load-generator --rm --image=busybox -- /bin/sh
```

Inside BusyBox container:

```bash
while true; do wget -q -O- http://hpaclusterservice.test.svc.cluster.local; done
```

---

# What Happens Internally?

* BusyBox continuously sends requests
* Application CPU usage increases
* Metrics Server collects updated metrics
* HPA detects high utilization
* HPA increases Pod replicas

Example:

```text
2 Pods → 3 Pods → 4 Pods → 5 Pods
```

---

# 10. After Load Reduces

Exit BusyBox:

```bash
exit
```

What happens:

* Load generator Pod gets deleted
* Traffic stops
* CPU/Memory usage decreases
* HPA gradually scales down Pods

Example:

```text
5 Pods → 4 Pods → 3 Pods → 2 Pods
```

Kubernetes scales down slowly to:

* Avoid sudden outages
* Prevent application instability
* Maintain availability

---

# Real-Time Production Understanding

## Scale Up

```text
High Traffic → More Pods
```

---

## Scale Down

```text
Low Traffic → Fewer Pods
```

---

# Final Interview Summary

## HPA Workflow

```text
Traffic Increase
       ↓
CPU/Memory Usage Increases
       ↓
Metrics Server Collects Metrics
       ↓
HPA Checks Target Threshold
       ↓
Deployment Replicas Increase
       ↓
More Pods Created
```
