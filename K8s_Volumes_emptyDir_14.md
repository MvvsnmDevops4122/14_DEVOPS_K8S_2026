# Kubernetes Volumes

## 1. Why Do We Need Kubernetes Volumes?

By default, data stored inside a container running in a Pod is temporary (**ephemeral**).

This means:

* If a container restarts → data may be lost
* If a Pod is deleted or rescheduled to another node → data is lost

As a result:

* Logs disappear
* Uploaded files disappear
* Database data gets deleted

Because containers do not store data permanently, Kubernetes introduces **Volumes** to store data outside the container filesystem.

Volumes provide:

* Persistent storage
* Permanent data retention
* Shared storage between containers
* Data retention during container restart
* Stable storage for stateful applications

Volumes are attached to the Pod, and containers inside the Pod mount and use the Volume.

Pod created  → Volume provisioned → Mounted into container → Container reads/writes → Pod deleted → Volume released/deleted

---

# 2. What is a Volume in Kubernetes?

A Kubernetes Volume is a storage mechanism that allows containers inside a Pod to store and access data persistently.

## Features of Kubernetes Volumes

* Persistent storage
* Permanent data retention
* Shared storage between containers
* Data retention during container restart
* Stable storage for stateful applications

By default, container storage is:

* Temporary
* Deleted when the container crashes/restarts

Volumes solve this problem.

---

# 3. Types of Kubernetes Volumes

## 1️⃣ emptyDir

`emptyDir` is a temporary Volume created automatically when a Pod is assigned to a node.

It provides temporary storage shared between containers running inside the same Pod.

## Features

* Created when Pod starts
* Deleted when Pod is removed
* Suitable for temporary data
* Good for temporary storage shared between containers running inside the same Pod

## Common Use Cases

* Logs
* Cache
* Temporary files
* Shared data between containers in same Pod

## YAML Example

```yaml
volumes:
  - name: cache-data
    emptyDir: {}
```

---

# 4. Spring Application Deployment (Example)

## springapp.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springapp-pod
  namespace: test

spec:
  replicas: 2

  selector:
    matchLabels:
      app: springapp

  template:
    metadata:
      labels:
        app: springapp

    spec:
      containers:
      - name: springapp-container
        image: kkeducation12345/spring-app:1.0.0

        ports:
        - containerPort: 8080

        env:
        - name: MONGO_DB_HOSTNAME
          value: mongosvc

        - name: MONGO_DB_USERNAME
          value: devdb

        - name: MONGO_DB_PASSWORD
          value: devdb@123

---
apiVersion: v1
kind: Service

metadata:
  name: springapp-svc
  namespace: test

spec:
  type: NodePort

  selector:
    app: springapp

  ports:
  - port: 80
    targetPort: 8080
```

---

# 5. MongoDB ReplicaSet with emptyDir Volume

## mongo_db_with_emptyDir.yaml

```yaml
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: mongodb-emptydir-rs
  namespace: test

spec:
  replicas: 1

  selector:
    matchLabels:
      app: mongodb

  template:
    metadata:
      labels:
        app: mongodb

    spec:
      containers:
      - name: mongocon
        image: mongo:6.0

        ports:
        - containerPort: 27017

        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value: devdb

        - name: MONGO_INITDB_ROOT_PASSWORD
          value: devdb@123

        volumeMounts:
        - name: mongo-emptydir-storage
          mountPath: /data/db

      volumes:
      - name: mongo-emptydir-storage
        emptyDir: {}

---
apiVersion: v1
kind: Service

metadata:
  name: mongosvc
  namespace: test

spec:
  type: ClusterIP

  selector:
    app: mongodb

  ports:
  - port: 27017
    targetPort: 27017
```
---

# Commands to Deploy

```bash
kubectl apply -f mongo_db_with_emptyDir.yaml
kubectl apply -f springapp.yaml
kubectl get pods -n test
kubectl get rs -n test
kubectl get svc -n test
```

If pod errors:

```bash
kubectl describe pod <pod-name> -n test
```

---

#  How to Check MongoDB Data is Stored

<img width="1916" height="967" alt="image" src="https://github.com/user-attachments/assets/d06b2053-d15f-4238-9ca9-28bf285c9aff" />


## 1️⃣ Get Pod Name

```bash
kubectl get pods -n test
```

Find pod like:

```
mongodb-emptydir-rs-xxxxx
```

## 2️⃣ Enter into MongoDB Pod

```bash
kubectl exec -it <pod-name> -n test -- bash
```

## 3️⃣ Check MongoDB Data Folder

```bash
ls -l /data/db
```

Expected files:

* WiredTiger
* WiredTiger.wt
* journal
* *.wt files
* metadata file

This confirms MongoDB is writing data to the emptyDir volume.

---

#  Test: Delete Pod and Check Data

## Step 1 — Delete Pod

```bash
kubectl delete pod <pod-name> -n test
```

ReplicaSet will create a new pod automatically.

## Step 2 — Check New Pod

```bash
kubectl get pods -n test
```

## Step 3 — Enter the New Pod

```bash
kubectl exec -it <new-pod-name> -n test -- bash
```

## Step 4 — Check Folder

```bash
ls /data/db
```

###  RESULT:

* Folder `/data/db` exists
* **BUT old data is gone**

###  Why?

Because:

> **emptyDir data is deleted whenever the Pod is deleted.**
---

## How emptyDir Volume is Stored on the Node

`emptyDir` uses the local storage of the worker node where the Pod is running.

When a Pod is scheduled to a node:

* Kubernetes automatically creates the `emptyDir` volume on that node.
* The volume is attached to the Pod.
* Containers access the volume through the specified `mountPath`.

### Example

```yaml
volumes:
- name: mongo-data
  emptyDir: {}

volumeMounts:
- name: mongo-data
  mountPath: /data/db
```

In this example:

* Volume Name = `mongo-data`
* Mount Path = `/data/db`

MongoDB stores its data inside `/data/db`, which is backed by the `emptyDir` volume.

---

## How to Verify emptyDir Volume is Mounted

### Step 1: Check Pod YAML

```bash
kubectl get pod mongo-rs-nczzb -n test -o yaml
```

Verify:

```yaml
volumes:
- emptyDir: {}
  name: mongo-data
```

and

```yaml
volumeMounts:
- mountPath: /data/db
  name: mongo-data
```

---

### Step 2: Check Inside the Container

Login into the Pod:

```bash
kubectl exec -it mongo-rs-nczzb -n test -- sh
```

Check the mount:

```bash
mount | grep /data/db
```

Output:

```bash
/dev/root on /data/db type ext4
```

This confirms that storage is mounted at `/data/db`.

---

## How to Verify emptyDir on the Worker Node

### Step 1: Find Pod UID

```bash
kubectl get pod mongo-rs-nczzb -n test -o jsonpath='{.metadata.uid}'
```

Example:

```bash
9fc81a23-38dc-4fa7-8297-26a03f86971f
```

### Step 2: Login to the Worker Node

```bash
ssh ubuntu@<worker-node>
```

### Step 3: Navigate to Pod Storage Directory

```bash
cd /var/lib/kubelet/pods/<POD-UID>/volumes/
```

Example:

```bash
cd /var/lib/kubelet/pods/9fc81a23-38dc-4fa7-8297-26a03f86971f/volumes/
```

### Step 4: Verify emptyDir Volume

```bash
ls
```

Output:

```bash
kubernetes.io~empty-dir
kubernetes.io~projected
```

Check the volume:

```bash
ls kubernetes.io~empty-dir/
```

Output:

```bash
mongo-data
```

This confirms that the `emptyDir` volume exists on the worker node.

---
