# Kubernetes ReplicaSet (RS)

---

# 1. What is a ReplicaSet?

A ReplicaSet (RS) is a Kubernetes object that ensures a specified number of identical Pod replicas are always running in the cluster.

It continuously monitors Pods and maintains the desired number of running Pods.

ReplicaSet is the advanced version of ReplicationController (RC).

---

# 2. Key Features of ReplicaSet

## 1. Self-Healing

If a Pod crashes or gets deleted, ReplicaSet automatically creates a new Pod.

---

## 2. Label-Based Management

ReplicaSet uses labels and selectors to identify and manage Pods.

---

## 3. Scaling

ReplicaSet allows increasing or decreasing the number of Pod replicas.

---

## 4. Advanced Selectors

ReplicaSet supports:
- Equality-based selectors
- Set-based selectors

---

## 5. High Availability

Ensures application Pods are always available.

---

# 3. Desired State vs Observed State

## Desired State

The number of Pods you want Kubernetes to maintain (e.g., 3 Pods).

---

## Observed State

The actual number of Pods currently running with the correct labels.

---

## Reconciliation

ReplicaSet continuously compares:
- Desired state
- Current state

and takes corrective actions to make them match.

### Example

- If desired = 3, but only 2 Pods are running → RS creates 1 more Pod.
- If desired = 3, but 5 Pods are running (extra Pods created manually) → RS deletes 2 extra Pods.

---

# 4. How ReplicaSet (RS) Works

1. User defines the desired number of Pod replicas in the RS YAML file.

2. The ReplicaSet creates Pods based on the Pod template.

3. RS continuously monitors the running Pods using labels and selectors.

4. RS compares:
   - Desired state
   - Observed state

5. If the number of running Pods is less than the desired count, RS creates new Pods.

6. If the number of running Pods is greater than the desired count, RS deletes extra Pods.

7. RS continuously ensures that the desired number of Pods is always running in the cluster.

---

# 5. Types of Selectors in ReplicaSet

ReplicaSet uses selectors to identify and manage Pods.

---

# 1. `matchLabels` (Equality-Based Selector)

Simple key-value matching.

```yaml
selector:
  matchLabels:
    app: myapp
```

- This ReplicaSet will only control Pods with the label `app=myapp`.
- If a Pod doesn’t have this label, RS ignores it.

---

# Example of Equality-Based Selector

```yaml
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: javawebapp-rs
  namespace: test

spec:
  replicas: 3

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
        image: satyamolleti4599/maven_web_app:1.0.0
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

# 2. `matchExpressions` (Set-Based Selector)

Advanced selector used for multiple labels.

Example Pod labels:

```yaml
labels:
  env: prod
  tier: frontend
  app: myapp
```

---

## Example

```yaml
selector:
  matchExpressions:
  - key: env
    operator: In
    values:
    - prod
```

### Meaning

ReplicaSet manages Pods where:

```yaml
env=prod
```

---

# Common Operators

| Operator | Meaning |
|---|---|
| `In` | Label value must match |
| `NotIn` | Label value must not match |
| `Exists` | Label key must exist |
| `DoesNotExist` | Label key must not exist |

---

# Example with Multiple Values

```yaml
selector:
  matchExpressions:
  - key: env
    operator: In
    values:
    - prod
    - dev
```

### Meaning

RS manages Pods with:

```text
env=prod
OR
env=dev
```

---

# Example of Set-Based Selector

```yaml
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: javawebapp-rs
  namespace: test

spec:
  replicas: 3

  selector:
    matchExpressions:
    - key: app
      operator: In
      values:
      - javawebapp1
      - javawebapp2
      - javawebapp

  template:
    metadata:
      labels:
        app: javawebapp

    spec:
      containers:
      - name: javawebapp
        image: satyamolleti4599/maven_web_app:1.0.0
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

# 6. Operators in `matchExpressions`

## 1. `In`

```yaml
- key: app
  operator: In
  values:
  - nginx
  - apache
```

- Pod must have the label `app`.
- Its value must be either:
  - `nginx`
  - `apache`

---

## 2. `NotIn`

```yaml
- key: app
  operator: NotIn
  values:
  - mysql
```

- Pod must have the label `app`.
- Its value must NOT be `mysql`.

---

## 3. `Exists`

```yaml
- key: release
  operator: Exists
```

- Pod must contain the label key `release`.
- Label value does not matter.

Example:

```yaml
labels:
  release: v1
```

✅ Valid

---

## 4. `DoesNotExist`

```yaml
- key: debug
  operator: DoesNotExist
```

- Pod must NOT contain the label key `debug`.

---

# 7. Difference Between RC and RS Selector

## ReplicationController (RC)

- Selector is optional.
- If the selector is not defined, RC automatically uses the labels from the Pod template.
- Supports only equality-based selectors.

---

## ReplicaSet (RS)

- Selector is mandatory.
- RS requires an explicitly defined selector to identify and manage Pods.
- Pod template labels must match the selector labels.
- Supports both:
  - equality-based selectors
  - set-based selectors (`matchExpressions`)

---

# Important Point

If Pod template labels do not match the ReplicaSet selector:
- RS may create Pods,
- but it will not properly manage or track them.

---

# 8. Disadvantages of ReplicaSet

## 1. No Rolling Updates

ReplicaSet does not support rolling updates automatically.

If image version changes:
- existing Pods are not updated automatically.

---

## 2. No Rollback Support

ReplicaSet cannot rollback to previous application versions.

---

## 3. Manual Update Required

To update application versions:
- Pods must be deleted manually,
- or ReplicaSet must be recreated.

---

## 4. No Deployment Management

ReplicaSet only manages:
- Pod replicas
- Pod availability

It does not manage:
- application releases
- version control
- deployment strategies

---

## 5. Mostly Used Internally by Deployments

In real-time projects:
- Deployments are used instead of ReplicaSets directly.

Deployment provides:
- rolling updates
- rollback
- version management

---

# Important Point

ReplicaSet is mainly used for:
- maintaining desired Pod count,
- self-healing,
- and scaling Pods.
````

Based on your uploaded ReplicaSet notes file. 
