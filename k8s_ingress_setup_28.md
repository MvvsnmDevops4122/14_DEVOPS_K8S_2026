# Complete Kubernetes Ingress Setup on AWS EKS (NGINX Ingress Controller)

## Topics Covered

* Deployment & Service
* NGINX Ingress Controller Installation (Helm)
* Ingress Resource Setup
* DNS Mapping (/etc/hosts)
* Browser Access
* NodePort vs LoadBalancer
* Troubleshooting
* Interview Questions

---

# Architecture

```text
Browser
   ↓
AWS LoadBalancer (ELB)
   ↓
NGINX Ingress Controller
   ↓
Ingress Rules
   ↓
Service
   ↓
Pods
```

---

# 1. Deploy Java Web Application

## Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: javawebappdep
  namespace: test-ns

spec:
  replicas: 2

  selector:
    matchLabels:
      app: javawebapp

  template:
    metadata:
      labels:
        app: javawebapp

    spec:
      containers:
      - name: javawebapp
        image: kkeducation12345/spring-app:1.0.5

        ports:
        - containerPort: 8080
```

Apply:

```bash
kubectl apply -f deployment.yaml
```

Verify:

```bash
kubectl get deploy -n test-ns
kubectl get pods -n test-ns -o wide
```

---

## Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: javawebappsvc
  namespace: test-ns

spec:
  type: NodePort

  selector:
    app: javawebapp

  ports:
  - port: 80
    targetPort: 8080
```

Apply:

```bash
kubectl apply -f service.yaml
```

Verify:

```bash
kubectl get svc -n test-ns
```

---

# 2. Install NGINX Ingress Controller Using Helm

## Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

Verify:

```bash
helm version
```

---

## Add Helm Repository

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

helm repo update
```

---

## Install Ingress Controller

```bash
helm install ingress-nginx ingress-nginx/ingress-nginx \
--namespace ingress-nginx \
--create-namespace
```

Verify:

```bash
kubectl get pods -n ingress-nginx
```

Expected:

```text
ingress-nginx-controller Running
```

---

# 3. Check Ingress Controller LoadBalancer

```bash
kubectl get svc -n ingress-nginx
```

Example:

```text
NAME                         TYPE           EXTERNAL-IP
ingress-nginx-controller     LoadBalancer   a8c9a4e82fb4d4726bb07851f26acae8.ap-south-1.elb.amazonaws.com
```

---

# 4. Create Ingress Resource

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: javawebapp-ingress
  namespace: test-ns

  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /

spec:
  ingressClassName: nginx

  rules:
  - host: javawebapp.example.com

    http:
      paths:
      - path: /
        pathType: Prefix

        backend:
          service:
            name: javawebappsvc

            port:
              number: 80
```

Apply:

```bash
kubectl apply -f javawebapp-ingress.yaml
```

Verify:

```bash
kubectl get ingress -n test-ns
kubectl describe ingress javawebapp-ingress -n test-ns
```

---

# 5. Configure DNS Mapping

## Find ELB IP

```bash
nslookup a8c9a4e82fb4d4726bb07851f26acae8.ap-south-1.elb.amazonaws.com
```

Example:

```text
15.206.18.139
3.6.27.227
```

---

## Linux

```bash
sudo vi /etc/hosts
```

Add:

```text
15.206.18.139 javawebapp.example.com
3.6.27.227 javawebapp.example.com
```

---

## Windows

Open Notepad as Administrator.

Open:

```text
C:\Windows\System32\drivers\etc\hosts
```

Add:

```text
15.206.18.139 javawebapp.example.com
3.6.27.227 javawebapp.example.com
```

Save.

Flush DNS:

```cmd
ipconfig /flushdns
```

Verify:

```cmd
ping javawebapp.example.com
```

---

# 6. Access Application

Open Browser:

```text
http://javawebapp.example.com
```

Expected:

```text
Spring Boot Application Home Page
```

---

# 7. Alternative Testing Without DNS

```bash
curl -H "Host: javawebapp.example.com" \
http://a8c9a4e82fb4d4726bb07851f26acae8.ap-south-1.elb.amazonaws.com
```

Useful during troubleshooting.

---

# 8. NodePort vs LoadBalancer

| Feature          | NodePort        | LoadBalancer |
| ---------------- | --------------- | ------------ |
| Access Method    | NodeIP:Port     | DNS/IP       |
| Easy to Use      | No              | Yes          |
| Production Ready | Limited         | Yes          |
| AWS Managed      | No              | Yes          |
| Ingress Support  | Backend Service | Preferred    |

### Interview Answer

NodePort is mainly used for testing and internal access, whereas LoadBalancer is preferred in production because it provides a stable external endpoint.

---

# 9. Common Troubleshooting

## Check Ingress

```bash
kubectl get ingress -A

kubectl describe ingress -n test-ns
```

---

## Check Service

```bash
kubectl get svc -n test-ns
```

---

## Check Endpoints

```bash
kubectl get endpoints -n test-ns
```

---

## Check Pods

```bash
kubectl get pods -n test-ns -o wide
```

---

## Check Ingress Controller

```bash
kubectl get pods -n ingress-nginx
```

---

## Check Logs

```bash
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller
```

---

# Common Issues

## Could Not Resolve Host

Cause:

* Missing DNS record
* Missing hosts file entry

Solution:

```bash
nslookup javawebapp.example.com
```

---

## 404 Not Found

Cause:

* Wrong host
* Wrong path

Check:

```bash
kubectl describe ingress
```

---

## 502 Bad Gateway

Cause:

* Service unavailable
* Pods not running

Check:

```bash
kubectl get svc
kubectl get pods
```

---

## LoadBalancer Pending

Cause:

* AWS provisioning delay

Check:

```bash
kubectl get svc -n ingress-nginx
```

---

# Interview Questions

## What is Ingress?

Ingress is a Kubernetes resource used to manage external HTTP/HTTPS access to applications.

---

## Why do we need Ingress?

To expose multiple applications through a single entry point.

---

## What is an Ingress Controller?

An Ingress Controller implements Ingress rules and routes traffic to backend Services.

---

## Difference Between Service and Ingress?

Service exposes Pods, whereas Ingress routes external traffic to Services.

---

## How Does Ingress Work?

```text
User
 ↓
LoadBalancer
 ↓
Ingress Controller
 ↓
Service
 ↓
Pods
```

---

## Which Ingress Controller Have You Used?

I have worked with NGINX Ingress Controller in AWS EKS environments.

---

## Real-Time Project Answer

In our EKS environment, we used the NGINX Ingress Controller to expose microservices externally. We implemented host-based routing, path-based routing, and centralized traffic management through a single AWS LoadBalancer instead of creating multiple LoadBalancer Services.

---

# One-Line Summary

Ingress is a Layer 7 Kubernetes resource that provides a single entry point for external HTTP/HTTPS traffic and routes requests to backend Services based on hostnames or URL paths.
