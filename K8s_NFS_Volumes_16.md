#  **Kubernetes NFS Volume**
---

# NFS Server (Network File System)

## What is NFS?

**NFS (Network File System)** is a distributed file system protocol that allows multiple servers or clients to access and share files and directories over a network.

It enables a directory on one server (**NFS Server**) to be mounted and accessed by multiple remote systems (**NFS Clients**) as if it were a local filesystem.

NFS provides centralized, shared, and persistent storage that can be accessed simultaneously by multiple servers, applications, or Kubernetes Pods.

---

## Why Do We Need NFS?

We need NFS (Network File System) to provide a centralized shared storage location that can be accessed by multiple servers, clients, or worker nodes simultaneously.

Without NFS, data is stored locally on individual servers or worker nodes. If an application moves to another server or node, it may not be able to access the same data.

NFS solves this problem by providing shared storage that can be accessed by multiple servers or worker nodes, ensuring data availability, consistency, and persistence.

---

## Benefits of NFS

✔ Read and write shared files

✔ Access the same data from multiple nodes

✔ Store application or database data centrally

✔ Provide persistent storage for Kubernetes Pods

✔ Eliminate data inconsistency across nodes

✔ Support shared storage for Stateful Applications

✔ Enable centralized storage management

---

## Real-Time Example

Without NFS:

```text
Node-1  → Local Storage
Node-2  → Local Storage
Node-3  → Local Storage
```

Data may become inconsistent because each node stores data separately.

With NFS:

```text
Node-1 ----\
Node-2 ----- > NFS Server
Node-3 ----/
```

All nodes access the same shared storage, ensuring data consistency and persistence.

---

## One-Line Interview Answer

An NFS Server is a centralized storage server that shares directories over a network, allowing multiple clients, servers, or Kubernetes Pods to access the same persistent data simultaneously.

---

# 3️⃣ NFS Working Diagram (Very Simple)

```
         ┌──────────────────┐
         │   NFS SERVER     │
         │ /mnt/nfs_share   │
         └───────┬──────────┘
                 │
    ┌────────────┼────────────┐
    │            │             │
┌──────────┐┌──────────┐┌──────────┐
│ Node 1   ││ Node 2   ││ Node 3   │
│ (client) ││ (client) ││ (client) │
└──────────┘└──────────┘└──────────┘
```

All Kubernetes nodes mount:

```
172.31.11.218:/mnt/nfs_share → /data/db
```

So **any pod** can read/write the same data.

---

# 4️⃣ How to Setup an NFS Server (Simple)

### ✔ Step 1: Launch an Ubuntu EC2 machine

Open NFS port **2049** in Security Group.

### ✔ Step 2: Update packages

```bash
sudo apt update -y
```

### ✔ Step 3: Install NFS Server

```bash
sudo apt install nfs-kernel-server -y
```

### ✔ Step 4: Create shared folder

```bash
sudo mkdir -p /mnt/nfs_share
sudo chown nobody:nogroup -R /mnt/nfs_share
sudo chmod 777 -R /mnt/nfs_share
```

### ✔ Step 5: Configure /etc/exports

```bash
sudo vi /etc/exports
```

Add line:

```
/mnt/nfs_share *(rw,sync,no_subtree_check,no_root_squash)
```

### ✔ Step 6: Apply export settings

```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

### ✔ Step 7: Check NFS running

```bash
ps -ef | grep nfs
```

---

# 5️⃣ Setup NFS Client on Kubernetes Nodes

### Install NFS client package:

```bash
sudo apt update -y
sudo apt install nfs-common -y
```

---

# 🟦 Kubernetes Deployment Files

---

# 🟩 **Spring Boot Application (Front-end API Layer)**

### springapplication.yaml

✔ Namespace: `test`
✔ Uses MongoDB Service via ENV variables
✔ NodePort service for browser access

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springapp
  namespace: test-ns 
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
  namespace: test-ns
spec:
  type: NodePort
  selector:
    app: springapp
  ports:
  - port: 80
    targetPort: 8080
```
# Apply Spring Application

```yaml
kubectl apply -f springapplication.yaml
```
---

# 🟨 **MongoDB Deployment Using NFS

### mongo.yaml

✔ ReplicaSet
✔ NFS mounted to `/data/db`
✔ Stores data permanently on NFS Server
✔ Namespace should be the same as spring (`test-ns`)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mongodb
  namespace: test-ns
spec:
  selector:
    matchLabels:
      app: mongodb
  replicas: 1
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
        - name: mongonfsvol
          mountPath: /data/db
      volumes:
      - name: mongonfsvol
        nfs:
          server: 172.31.11.218
          path: /mnt/nfs_share
---
apiVersion: v1
kind: Service
metadata:
  name: mongosvc
  namespace: test-ns
spec:
  type: ClusterIP
  selector:
    app: mongodb
  ports:
    - port: 27017
      targetPort: 27017
```
# Apply MongoDB

```yaml
kubectl apply -f mongo.yaml
```
---

# 6️⃣ Why This Setup Works Perfectly

### ✔ MongoDB uses NFS → data persists

### ✔ Spring connects to MongoDB via service

### ✔ Both apps are in the same namespace → DNS resolves

### ✔ NFS is outside the cluster → no data loss


✅ NFS-based PV & PVC version
✅ StatefulSet version for MongoDB
✅ Architecture diagram (Visually beautiful)
Just tell me!
