# StatefulSet with MySQL - Complete Notes

## 1. Key Differences Between Deployment and StatefulSet

| Feature            | Deployment                                  | StatefulSet                                      |
| ------------------ | ------------------------------------------- | ------------------------------------------------ |
| Pod Identity       | Random (no stable name or ID)               | Stable, predictable pod names (mysql-0, mysql-1) |
| Persistent Storage | No guarantee, shared storage optional       | Each Pod gets its own PersistentVolumeClaim      |
| Scaling Behavior   | All replicas are identical                  | Ordered scaling up/down                          |
| Use Cases          | Stateless apps (API, Frontend, Spring Boot) | Stateful apps (MySQL, MongoDB, Kafka, Zookeeper) |
| Headless Service   | Not required                                | Usually required                                 |
| Rolling Updates    | Faster                                      | Slower (ordered updates)                         |

---

# 2. Solution Architecture

### Components

* StatefulSet for MySQL
* ConfigMap for MySQL Username
* Secret for MySQL Password
* Headless Service for stable DNS
* PersistentVolumeClaim for MySQL Data

---

# 3. Complete YAML Setup

## Step 1: ConfigMap (MySQL Username)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
data:
  MYSQL_USER: "myuser"
```

---

## Step 2: Secret (MySQL Password)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  MYSQL_PASSWORD: bXlwYXNzd29yZA==
```

> Note: Secret values must be Base64 encoded.

---

## Step 3: Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
  - port: 3306
```

### Purpose

* Provides stable DNS names.
* Enables direct Pod-to-Pod communication.
* Required for StatefulSet.

---

## Step 4: StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql-headless
  replicas: 1

  selector:
    matchLabels:
      app: mysql

  template:
    metadata:
      labels:
        app: mysql

    spec:
      containers:
      - name: mysql
        image: mysql:8.0

        ports:
        - containerPort: 3306

        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_PASSWORD

        - name: MYSQL_USER
          valueFrom:
            configMapKeyRef:
              name: mysql-config
              key: MYSQL_USER

        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_PASSWORD

        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql

  volumeClaimTemplates:
  - metadata:
      name: mysql-persistent-storage

    spec:
      accessModes:
      - ReadWriteOnce

      resources:
        requests:
          storage: 1Gi
```

---

# 4. How It Works

| Component        | Purpose                                  |
| ---------------- | ---------------------------------------- |
| ConfigMap        | Stores MySQL username                    |
| Secret           | Stores MySQL password                    |
| Headless Service | Provides stable DNS                      |
| StatefulSet      | Creates MySQL Pod with stable hostname   |
| PVC              | Creates persistent storage automatically |

---

# 5. Important Details

## Stable Hostname

```text
mysql-0.mysql-headless.default.svc.cluster.local
```

---

## Auto-Created PVC

```text
mysql-persistent-storage-mysql-0
```

---

## Access Password Inside Pod

```bash
echo $MYSQL_PASSWORD
```

---

# 6. Why StatefulSet for MySQL?

MySQL requires:

* Stable Pod Identity
* Persistent Storage
* Ordered Startup and Shutdown

Deployment cannot guarantee these requirements.

Therefore StatefulSet is the preferred choice.

---

# 7. Why Headless Service?

A Headless Service:

* Does not get a ClusterIP.
* Provides direct Pod access.
* Creates stable DNS names.

Example:

```text
mysql-0.mysql-headless.default.svc.cluster.local
```

This allows MySQL Pods to communicate directly.

---

# 8. Troubleshooting Guide

## PVC Pending

Check:

```bash
kubectl get pvc
kubectl describe pvc <pvc-name>
kubectl get storageclass
```

Verify:

* StorageClass exists.
* Provisioner is working.
* PV is available.

---

## Pod CrashLoopBackOff

Check:

```bash
kubectl logs <pod-name>
kubectl describe pod <pod-name>
```

Verify:

* Username and password.
* ConfigMap values.
* Secret values.
* Volume mount path.

---

## MySQL Connectivity Issues

Check:

```bash
kubectl get svc
kubectl get endpoints
```

Test DNS:

```bash
nslookup mysql-headless
```

---

## Volume Mount Issues

Check:

```bash
kubectl describe pod <pod-name>
kubectl describe pvc <pvc-name>
kubectl describe pv <pv-name>
```

Verify:

* PVC is Bound.
* PV is Bound.
* Storage backend is reachable.

---

# 9. Why This Setup Works in Production

* Secrets prevent hardcoded passwords.
* Persistent storage preserves database data.
* StatefulSet provides stable Pod identities.
* Headless Service provides stable DNS.
* Ordered deployment and scaling improve reliability.
* MySQL 8.0 is production-ready.

---

# Interview Summary

### Why StatefulSet for MySQL?

> MySQL requires stable Pod identities, persistent storage, and ordered deployment. StatefulSet provides these capabilities, making it the preferred choice for running MySQL in Kubernetes.

### Why Headless Service?

> Headless Service provides stable DNS names and direct Pod access, which are required by StatefulSet-based applications such as MySQL, MongoDB, and Kafka.
