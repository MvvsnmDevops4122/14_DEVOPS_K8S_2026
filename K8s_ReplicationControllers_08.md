
## 1. What is a ReplicationController (RC)?

A ReplicationController (RC) is a Kubernetes object that ensures a fixed number of identical Pods are always running in the cluster.

It acts as a controller, continuously monitoring the cluster state.

It takes corrective actions to maintain the desired number of Pods.

---

## 2. Key Features

### 1. Self-healing
If a Pod crashes or is deleted, the ReplicationController creates a new Pod automatically.

### 2. No extra Pods allowed
If someone manually creates extra Pods with the same labels, the ReplicationController deletes the extra Pods to maintain the desired count.

### 3. Label-based tracking
The ReplicationController uses labels, not Pod names, to identify and manage Pods.

---

## 3. Desired State vs Observed State

### Desired state
The number of Pods you want Kubernetes to maintain (e.g., 3 Pods).

### Observed state
The actual number of Pods currently running with the correct labels.

### Reconciliation
The ReplicationController continuously compares the desired state with the observed state and takes corrective actions to make them match.

### Example

- If desired = 3, but only 2 Pods are running → RC creates 1 more Pod.
- If desired = 3, but 5 Pods are running (extra Pods created manually) → RC deletes 2 extra Pods.

---

## 4. How ReplicationController (RC) Works

1. User defines the desired number of Pod replicas in the RC YAML file.

2. The ReplicationController creates Pods based on the Pod template.

3. RC continuously monitors the running Pods using labels and selectors.

4. RC compares:
   - Desired state
   - Observed state

5. If the number of running Pods is less than the desired count, RC creates new Pods.

6. If the number of running Pods is greater than the desired count, RC deletes extra Pods.

7. RC continuously ensures that the desired number of Pods is always running in the cluster.

---

# 5. ReplicationController (RC) YAML Example

```yaml
apiVersion: v1
kind: ReplicationController

metadata:
  name: javawebapprc          # ReplicationController name
  namespace: test-ns

spec:
  replicas: 3                 # Desired number of Pods

  selector:
    app: javawebapp           # Matches Pods with this label

  template:                   # Pod template
    metadata:
      labels:
        app: javawebapp       # Pod label

    spec:
      containers:
      - name: javawebappcon
        image: satyamolleti4599/maven_web_app:1.0.0
        ports:
        - containerPort: 8080
```

---

# 6. Working with ReplicationController

## 1. Create RC

```bash
kubectl apply -f rc.yaml
```

---

## 2. Verify Resources

```bash
kubectl get all -n test-ns
kubectl get rc -n test-ns
```

---

## 3. Test Self-Healing

Delete one Pod manually:

```bash
kubectl delete pod <pod-name> -n test-ns
```

RC automatically creates a new Pod to maintain the desired count.

---

## 4. Check Pod Owner

```bash
kubectl describe pod <pod-name> -n test-ns
```

Check:

```text
Controlled By: ReplicationController/javawebapprc
```

---

# 7. Scaling ReplicationController

## Scale Up

```bash
kubectl scale rc javawebapprc --replicas=3 -n test-ns
```

RC creates additional Pods.

---

## Scale Down

```bash
kubectl scale rc javawebapprc --replicas=0 -n test-ns
```

All Pods are deleted, but the RC still exists.

---

## Scale Up Again

```bash
kubectl scale rc javawebapprc --replicas=3 -n test-ns
```

RC recreates the Pods.

---

# 8. Delete ReplicationController

## Delete RC with Pods

```bash
kubectl delete rc javawebapprc -n test-ns
```

Both:
- RC
- Managed Pods

will be deleted.
````
