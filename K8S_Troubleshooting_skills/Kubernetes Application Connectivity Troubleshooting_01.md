# Kubernetes Application Connectivity Troubleshooting Guide

## Real-Time Scenario: Spring Boot Unable to Connect to MongoDB

---

# Problem Statement

Spring Boot application was opening successfully, but data was not getting inserted into MongoDB.

Application Error:

```text
Whitelabel Error Page

Timed out after 30000 ms while waiting to connect.

MongoTimeoutException

Connection refused

mongosvc:27017
```

---

# Step 1: Verify Pod Status

First, check whether application and database pods are running.

```bash
kubectl get pods -n test -o wide
```

Example:

```text
NAME                                    READY   STATUS    NODE
mongodb-deployment-7945568ff7-rzb9r     1/1     Running   ip-172-31-10-219
springapp-deployment-5bff7849b8-2vbml   1/1     Running   ip-172-31-0-14
```

### Observation

✔ MongoDB Pod Running

✔ Spring Boot Pod Running

---

# Step 2: Check Application Logs

Check Spring Boot logs.

```bash
kubectl logs <spring-pod-name> -n test
```

Example:

```text
MongoTimeoutException
Connection refused
mongosvc:27017
```

### Observation

Application is unable to connect to MongoDB Service.

---

# Step 3: Verify MongoDB Service

Check whether Service exists.

```bash
kubectl get svc mongosvc -n test
```

Example:

```text
NAME       TYPE        CLUSTER-IP      PORT(S)
mongosvc   ClusterIP   10.104.11.19    27017/TCP
```

### Observation

✔ Service exists.

---

# Step 4: Verify Service Endpoints (Most Important)

Check endpoints associated with Service.

```bash
kubectl get endpoints mongosvc -n test
```

Output:

```text
NAME       ENDPOINTS
mongosvc   <none>
```

### Observation

❌ Service has no endpoints.

This means Service is not forwarding traffic to any Pod.

---

# Step 5: Verify Service Selector

Check Service selector.

```bash
kubectl describe svc mongosvc -n test
```

Example:

```text
Selector:
app=mongodb
```

---

# Step 6: Verify Pod Labels

Check labels assigned to MongoDB Pod.

```bash
kubectl get pods -n test --show-labels
```

Example:

```text
mongodb-deployment-xxx   app=mongo
```

---

# Step 7: Compare Selector and Labels

Service:

```yaml
selector:
  app: mongodb
```

Pod:

```yaml
labels:
  app: mongo
```

### Observation

❌ Selector and Labels do not match.

Because of this:

```text
Service
   |
   X
   |
MongoDB Pod
```

No endpoint gets created.

---

# Root Cause

```text
Service Selector ≠ Pod Labels
```

Therefore:

```text
Endpoints = <none>

↓

Spring Boot cannot reach MongoDB

↓

Connection Refused

↓

MongoTimeoutException
```

---

# Step 8: Fix the Issue

Update Service selector:

```yaml
spec:
  selector:
    app: mongo
```

OR

Update Pod label:

```yaml
labels:
  app: mongodb
```

### Rule

Both values must be identical.

---

# Step 9: Apply Changes

```bash
kubectl apply -f mongodb.yaml
```

---

# Step 10: Verify Endpoints Again

```bash
kubectl get endpoints mongosvc -n test
```

Expected Output:

```text
NAME       ENDPOINTS
mongosvc   192.168.186.134:27017
```

### Observation

✔ Endpoint created successfully.

---

# Step 11: Test Application

Insert data from Spring Boot application.

Result:

```text
Data Inserted Successfully
```

---

# Kubernetes Service Troubleshooting Flow

```text
Application Not Working
          │
          ▼
Check Pod Status
          │
          ▼
Check Application Logs
          │
          ▼
Check Service
          │
          ▼
Check Endpoints
          │
          ▼
Endpoints = <none> ?
          │
          ▼
Check Service Selector
          │
          ▼
Check Pod Labels
          │
          ▼
Fix Mismatch
          │
          ▼
Verify Endpoints
          │
          ▼
Application Working
```

---

# Important Troubleshooting Commands

### Check Pods

```bash
kubectl get pods -n <namespace> -o wide
```

### Check Services

```bash
kubectl get svc -n <namespace>
```

### Check Endpoints

```bash
kubectl get endpoints -n <namespace>
```

### Describe Service

```bash
kubectl describe svc <service-name> -n <namespace>
```

### Show Pod Labels

```bash
kubectl get pods --show-labels -n <namespace>
```

### Check Application Logs

```bash
kubectl logs <pod-name> -n <namespace>
```

### Login to Pod

```bash
kubectl exec -it <pod-name> -n <namespace> -- sh
```

### Verify Environment Variables

```bash
kubectl exec -it <pod-name> -n <namespace> -- env
```

### Verify DNS Resolution

```bash
nslookup <service-name>
```

### Verify Connectivity

```bash
nc -zv <service-name> <port>
```

---

# Interview Question

### Application Pod is Running and Database Pod is Running, but Application Cannot Connect to Database. How Will You Troubleshoot?

### Answer:

1. Check Pod status.
2. Check application logs.
3. Check Service status.
4. Check Service endpoints.
5. Verify Service selector.
6. Verify Pod labels.
7. Verify DNS resolution.
8. Verify network connectivity.
9. Check environment variables.
10. Fix selector/label mismatch if endpoints are missing.

---

# Key Learning

 A Running Pod and Running Service do not guarantee application connectivity.

Always verify:

```bash
kubectl get endpoints <service-name>
```

If:

```text
ENDPOINTS = <none>
```

the first thing to check is:

```text
Service Selector and Pod Labels
```

This is one of the most common real-time Kubernetes issues faced by DevOps Engineers. 

---
