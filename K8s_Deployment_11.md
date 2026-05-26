
# Kubernetes Deployment

---

# 1. What is a Deployment?

A Deployment is a Kubernetes object used to deploy, manage, and update applications in a Kubernetes cluster.

It is the most commonly used Kubernetes object in real-time projects because it automatically manages the complete lifecycle of Pods through ReplicaSets.

Deployments are considered the backbone of Kubernetes applications.

A Deployment provides features like:

- Scaling
- Self-healing
- Rolling updates
- Rollback

---

# 2. Main Purpose of Deployment

Deployment is mainly used for:

- application deployment
- application updates
- scaling applications
- rollback management
- maintaining application availability
- self-healing of failed Pods
- zero-downtime deployments

---

# 3. Key Features of Deployment

## 1. Self-Healing

If a Pod crashes or gets deleted, the Deployment automatically creates a new Pod to maintain the desired state with the help of a ReplicaSet.

---

## 2. Scaling

Deployment allows us to increase or decrease the number of Pod replicas based on application requirements.

---

## 3. Rolling Updates

Deployment updates the application version gradually without downtime by replacing old Pods with new Pods step-by-step.

---

## 4. Rollback

If the new application version fails, Deployment allows rollback to the previous stable version.

---

## 5. High Availability

Deployment ensures the required number of Pods are always running to keep the application highly available.

---

## 6. ReplicaSet Management

Deployment internally creates and manages ReplicaSets, which are responsible for managing Pods.

---

# 4. Deployment Architecture in Kubernetes

```text
                Deployment
                     │
                     │ manages
                     ▼
                ReplicaSet
                     │
                     │ manages
                     ▼
                    Pods
                     │
                     │ contains
                     ▼
                Containers
```

---

# 1. Deployment

A Deployment is a Kubernetes object used to deploy, manage, and update applications in a Kubernetes cluster.

It manages the complete lifecycle of applications.

Deployment provides features like:

- Scaling
- Self-healing
- Rolling updates
- Rollback

Deployment does not directly manage Pods. It manages Pods through ReplicaSets.

---

# 2. ReplicaSet

A ReplicaSet is responsible for maintaining the desired number of Pods.

Deployment internally creates and manages ReplicaSets automatically.

If a Pod crashes or gets deleted, the ReplicaSet automatically creates a new Pod to maintain the desired state.

ReplicaSet performs the actual Pod management and self-healing operations.

---

# 3. Pods

A Pod is the smallest deployable unit in Kubernetes.

Pods run one or more containers.

---

# 4. Containers

Containers run the actual application inside the Pod.

Examples:

- Nginx
- Java application
- Python application

---

# 5. How Deployment Works

1. User creates a Deployment YAML file using:

```bash
kubectl apply -f deployment.yaml
```

2. The Deployment automatically creates a ReplicaSet based on the Pod template defined in the YAML file.

3. The ReplicaSet creates and maintains the required number of Pods.

4. The Deployment continuously monitors ReplicaSets and Pods to maintain the desired state.

5. If a Pod crashes or gets deleted, the ReplicaSet automatically creates a new Pod to maintain the desired state.

6. Deployment updates the application version gradually without downtime by replacing old Pods with new Pods step-by-step.

7. If the new application version fails, Deployment allows rollback to the previous stable version.

---
# 6. Deployment Strategies in Kubernetes

Deployment strategies define how application updates are performed in Kubernetes.

They control:

- how old Pods are replaced
- how new Pods are created
- application availability during updates

---

# Types of Deployment Strategies

## 1. RollingUpdate Strategy (Default)

## 2. Recreate Strategy

---

# 1. Recreate Strategy

Recreate strategy deletes all old Pods first, then creates new Pods.

<img width="861" height="485" alt="image" src="https://github.com/user-attachments/assets/0ed261be-7ceb-4dfe-b4a7-dee8dc0cf439" />

---

# How Recreate Strategy Works

```text
Old Pods Running
        ↓
Delete All Old Pods
        ↓
Create New Pods
```

---

# Disadvantages of Recreate

- Application downtime occurs
- Users cannot access application during update

---

# When Recreate is Used

Recreate strategy is used when:

- old and new versions cannot run together
- database schema changes exist
- compatibility issues exist between versions

---

# YAML Example (Recreate Strategy)

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: javawebapp-deployment
  namespace: test

spec:
  replicas: 3

  strategy:
    type: Recreate

  selector:
    matchLabels:
      app: javawebapp

  template:
    metadata:
      labels:
        app: javawebapp

    spec:
      containers:
      - name: javawebapp-container
        image: kkeducation12345/spring-app:1.0.0
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

# Apply Deployment

```bash
kubectl apply -f deploy.yaml
```

---

# Verify Resources

```bash
kubectl get all -n test
```

---

# Check Rollout Status

```bash
kubectl rollout status deployment javawebapp-deployment -n test
```

---

# Update Application Version

Update image version in YAML:

```yaml
image: kkeducation12345/spring-app:1.0.1
```

Apply changes:

```bash
kubectl apply -f deploy.yaml
```

---

# What Happens Internally?

Because strategy is:

```yaml
type: Recreate
```

Kubernetes:

1. Deletes all old Pods
2. Creates new Pods with updated version

This causes temporary downtime.

---
# Checking Deployment Rollout History

* Each update creates a new ReplicaSet and a new revision number.
* You can confirm which image was deployed and what configuration was applied.

```bash
kubectl rollout history deployment javawebappdep -n test-ns --revision=2
```
<img width="889" height="461" alt="image" src="https://github.com/user-attachments/assets/7877d0e2-fb12-468d-bc4a-6c0d02ee4dbd" />

---

# Update Deployment to Version 1.0.2

Change YAML:

```yaml
image:  kkeducation12345/spring-app:1.0.2
```

Apply update:

```bash
kubectl apply -f deploy.yaml
kubectl get all -n test-ns
```

---

### Step 5: Update to Version 2.0.0

Change YAML:

```yaml
image:  kkeducation12345/spring-app:2.0.0
```

Apply and check rollout:

```bash
kubectl apply -f deploy.yaml
kubectl rollout status deployment javawebappdeploy -n test-ns
```

Rollout process shows:

* 0 of 3 available → downtime
* 1/3, 2/3 available → partial availability
* 3/3 available → rollout completed

---

## Kubernetes Deployment Rollout History and Rollback

1. **Check rollout history**

   ```bash
   kubectl rollout history deployment javawebappdeploy -n test-ns
   ```

2. **Verify state**

   ```bash
   kubectl get all -n test-ns
   ```

3. **Perform rollback**

   ```bash
   kubectl rollout undo deployment javawebappdeploy -n test-ns --to-revision=3
   ```

👉 Kubernetes scales down the current RS and scales up the older RS.

4. **After rollback**
   Check again → old ReplicaSet becomes active.

5. **Revision numbers keep increasing**
   Even rollbacks get a new revision number.
   Rollback to revision 3 → logged as revision 5.

   <img width="841" height="491" alt="image" src="https://github.com/user-attachments/assets/b33eebea-d221-46b1-96cc-bb79e7d95339" />


7. **Rolling back to Revision 1**

   ```bash
   kubectl rollout undo deployment javawebappdeploy -n test-ns --to-revision=1
   ```

---

## 2. RollingUpdate Strategy (Default in Kubernetes)

## What is RollingUpdate?

RollingUpdate is the default Deployment strategy in Kubernetes.

Pods are updated gradually without downtime.

Old Pods are replaced step-by-step with new Pods.

<img width="886" height="485" alt="image" src="https://github.com/user-attachments/assets/0736f16d-dd03-4dab-b089-af21553ce50b" />

---

### Example Flow

```
Step 1: [v1][v1][v1]
Step 2: [v2][v1][v1]
Step 3: [v2][v2][v1]
Step 4: [v2][v2][v2]
```

---

### YAML Example (RollingUpdate Strategy)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: javawebapp-deployment
  namespace: test
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
  selector:
    matchLabels:
      app: javawebapp
  template:
    metadata:
      labels:
        app: javawebapp
    spec:
      containers:
      - name: javawebapp-container
        image:  kkeducation12345/spring-app:1.0.0
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: javawebapp-service
  namespace: test-ns
spec:
  selector:
    app: javawebapp
  ports:
    - protocol: TCP
      port: 8080        # Service port
      targetPort: 8080  # Container port
      nodePort: 30008   # Must be between 30000–32767 if type=NodePort
  type: NodePort

```

---

### RollingUpdate Process

1. Apply Deployment:

   ```bash
   kubectl apply -f deploy.yaml
   ```

2. Check Pods:

   ```bash
   kubectl get po -n test-ns
   ```

   * New Pods → `ContainerCreating`
   * Old Pods → `Terminating`
   * New Pods eventually reach `Running` state.

3. Final State:

   * Old ReplicaSet → scaled down to 0.
   * New ReplicaSet → scaled up to desired count.

---

### Why RollingUpdate is Preferred?

**No downtime** – At least one Pod always running.
**Safer upgrades** – Stops rollout if new Pods fail.
**Easy rollback** – Switch back to older ReplicaSet.
**Production standard** – Default strategy for high availability clusters.
