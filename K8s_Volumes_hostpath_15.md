# 2️.hostPath Volume

---

## What is hostPath Volume?

A `hostPath` Volume is used to mount a file or directory from the worker node's filesystem(host node’s filesystem) into a Pod.

It allows containers running inside the Pod to access files and directories available on the host machine.

Unlike `emptyDir`, which is created and managed by Kubernetes, `hostPath` uses an existing path from the worker node.

When a Pod is deleted and recreated on the same node, it mounts the same host directory and can access previously stored data, providing basic data persistence.

Stores data on the Kubernetes node filesystem.

Good for testing, monitoring, and node-level integrations.

Not recommended for production because the data is tied to a specific node(node failure = data loss).

---

## Features

* Maps a host file or directory into a Pod.
* Uses the worker node's local filesystem.
* Data persists even after Pod deletion.
* Allows sharing files between the host and container.
* Useful for monitoring and debugging applications.

---

## Common Use Cases

* Accessing host log files (`/var/log`)
* Running monitoring tools such as cAdvisor
* Accessing Docker files (`/var/lib/docker`)
* Accessing Kubernetes files (`/var/lib/kubelet`)
* Sharing files between host and container

---

## Limitations of hostPath Volume

* Data is tied to a specific worker node.
* If the worker node fails, the data may be lost.
* If the Pod is rescheduled to another node, it cannot access the data stored on the previous node.
* Data may become inconsistent across different nodes.
* Not recommended for production workloads.

---


### emptyDir

```text
Pod Deleted
    ↓
Volume Deleted
    ↓
Data Lost
```

### hostPath

```text
Pod Deleted
    ↓
Host Directory Remains
    ↓
Data Preserved
```

**Note:** Data is preserved only when the Pod is recreated on the same worker node.

---

## Example: MongoDB and service without Volumes

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata: 
  name: mongodb
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
        image: mongo:8.0.9-noble
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value: devdb
        - name: MONGO_INITDB_ROOT_PASSWORD
          value: devdb@123
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
````

### ✔ Commands to Apply

```
kubectl apply -f mongodb.yaml
kubectl get pods -n test-ns
kubectl get rs -n test-ns
kubectl get svc -n test-ns
```

## Example: Spring Boot app and service without Volumes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springapp
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
      - name: springapp
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
  name: springappsvc
  namespace: test
spec:
  type: NodePort
  selector:
    app: springapp
  ports:
  - port: 80
    targetPort: 8080
```
<img width="1067" height="442" alt="image" src="https://github.com/user-attachments/assets/00e6f660-1834-4240-83fd-a5cbef3644a4" />

### ✔ Commands to Deploy Spring App

```
kubectl apply -f springapp.yaml
kubectl get pods -n test-ns
kubectl get svc -n test-ns
```
Accessing Application Through NodeIP:Port

 ✔ How to Access Application
👉 From Browser or Curl:
http://<Node-IP>:<NodePort>


Example:

http://54.201.122.90:30080

<img width="1914" height="968" alt="image" src="https://github.com/user-attachments/assets/36340f81-e0f9-48a9-a04d-f854e24cf44d" />


### ✔ Check MongoDB Data Directory

```
kubectl exec -it <pod-Name> -n <NS> -- ls /data/db
```

## After adding host path volumes

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata: 
  name: mongodb
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
        image: mongo:8.0.9-noble
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value: devdb
        - name: MONGO_INITDB_ROOT_PASSWORD
          value: devdb@123
        volumeMounts:
        - name: mongovol
          mountPath: /data/db
      volumes:
      - name: mongovol
        hostPath:
          path: /mongobkp
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
<img width="1061" height="310" alt="image" src="https://github.com/user-attachments/assets/1b0273a5-cc33-4fe2-8064-234771ea91d6" />

### ✔ Apply MongoDB With HostPath

```
kubectl apply -f mongodb-hostpath.yaml
kubectl get pods -n test-ns
kubectl describe pod <pod-name> -n test-ns
```

## ✔ Verify Data Is Stored on Node
<img width="1704" height="965" alt="image" src="https://github.com/user-attachments/assets/9c9cf729-c2bc-412f-abe3-e01187840779" />

### 1️⃣ Check inside Pod

```
kubectl exec -it <pod-name> -n test-ns -- ls /data/db
```

### 2️⃣ Check on Node

SSH into node:

```
ls -l /mongobkp
```

You will see MongoDB's files (WiredTiger, .wt files, journal).

### 3 — Delete the Pod

If using ReplicaSet or Deployment:
```
kubectl delete pod <mongodb-pod> -n test
```

Kubernetes will automatically create a new Pod.

### 4 — Check Pod Re-Created
```
kubectl get pods -n test

```
You will see a new pod created automatically.

### 5 — Verify Data Inside New Pod

Now check the data directory again inside the new pod:
```
kubectl exec -it <new-mongodb-pod> -n test -- ls /data/db
```

If hostPath is working correctly, you will see:

WiredTiger
WiredTiger.wt
*.wt files
journal/


✔ Data restored → Persistence working
If empty → hostPath path is wrong or node changed

### Step 6 — Verify Data Still Present on Node

```
ls -l /mongobkp
```

This ensures data never got deleted.

⚠️ Important: hostPath Works Only If Pod Comes Back on SAME Node

* If new Pod gets scheduled on another node, hostPath folder will not exist → data looks lost.

* To prevent this, check node of Pod:
```
kubectl get pod -n test -o wide
```

You will see:
```
NAME          NODE
mongodb-xxx   ip-172-31-21-134
```

If Pod restarts on same node → data is safe
If Pod moves to another node → data will be empty

---
