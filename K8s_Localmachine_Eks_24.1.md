# Connecting Local Machine to Amazon EKS Cluster - Complete Notes

## 1. Introduction

After creating an EKS cluster, we need to connect our local machine to the cluster so that we can manage Kubernetes resources using `kubectl`.

Once connected, we can:

* Create Pods
* Create Deployments
* Create Services
* Create Namespaces
* Create ConfigMaps
* Create Secrets
* Troubleshoot Applications
* Manage Worker Nodes

---

# Architecture

```text
Local Machine (Windows/Linux/Mac)
        |
        | kubectl
        |
        | AWS CLI Authentication
        |
Amazon EKS Control Plane
        |
Worker Nodes
```

---

# Prerequisites

Before connecting your local machine to EKS, ensure:

1. AWS Account exists
2. EKS Cluster is running
3. IAM User has EKS permissions
4. AWS CLI installed
5. kubectl installed
6. Internet connectivity

---

# Step 1: Verify EKS Cluster

Login to AWS and verify cluster exists.

List available clusters:

```bash
aws eks list-clusters --region ap-south-1
```

Example Output:

```json
{
    "clusters": [
        "my-eks-cluster"
    ]
}
```

---

# Step 2: Install AWS CLI

AWS CLI is required to authenticate with AWS services.

Verify installation:

```bash
aws --version
```

Expected Output:

```bash
aws-cli/2.x.x
```

If not installed:

Download and install AWS CLI.

Verify again:

```bash
aws --version
```

---

# Step 3: Create IAM User

Create an IAM User if not already available.

Navigate:

```text
AWS Console
   ↓
IAM
   ↓
Users
   ↓
Create User
```

Example:

```text
Username:
eks-admin
```

---

# Step 4: Assign Permissions

Attach permissions:

```text
AdministratorAccess
```

or

```text
AmazonEKSClusterPolicy
AmazonEKSWorkerNodePolicy
AmazonEC2ContainerRegistryReadOnly
```

For learning purposes:

```text
AdministratorAccess
```

is acceptable.

---

# Step 5: Generate Access Key and Secret Key

Navigate:

```text
IAM
 ↓
Users
 ↓
Select User
 ↓
Security Credentials
 ↓
Create Access Key
```

AWS generates:

```text
Access Key ID
Secret Access Key
```

Store them safely.

Example:

```text
Access Key ID:
AKIAxxxxxxxxxxxx

Secret Access Key:
xxxxxxxxxxxxxxxxxxxx
```

---

# Why Access Key and Secret Key are Required?

AWS CLI cannot automatically identify who you are.

AWS uses:

```text
Access Key ID
Secret Access Key
```

to authenticate requests.

Without them:

```bash
aws eks list-clusters
```

fails.

---

# Step 6: Configure AWS CLI

Run:

```bash
aws configure
```

Enter:

```text
AWS Access Key ID:
AKIAxxxxxxxxxxxx

AWS Secret Access Key:
xxxxxxxxxxxxxxxx

Default region:
ap-south-1

Default output format:
json
```

---

# Verify AWS Authentication

Run:

```bash
aws sts get-caller-identity
```

Example Output:

```json
{
  "UserId":"ABCDEF",
  "Account":"123456789012",
  "Arn":"arn:aws:iam::123456789012:user/eks-admin"
}
```

If output appears successfully:

```text
AWS Authentication Successful
```

---

# Step 7: Install kubectl

kubectl is the Kubernetes command-line tool.

Verify:

```bash
kubectl version --client
```

Expected:

```bash
Client Version: v1.xx.x
```

---

# Why kubectl is Required?

kubectl communicates with Kubernetes API Server.

Examples:

```bash
kubectl get pods

kubectl get nodes

kubectl get svc
```

Without kubectl:

```text
Cannot manage Kubernetes resources.
```

---

# Step 8: Verify EKS Cluster Details

Check cluster information:

```bash
aws eks describe-cluster \
--name my-eks-cluster \
--region ap-south-1
```

This returns:

* Cluster Endpoint
* Certificate
* Cluster ARN
* VPC Details

---

# Step 9: Generate kubeconfig File

Most Important Step

Command:

```bash
aws eks update-kubeconfig \
--region ap-south-1 \
--name my-eks-cluster
```

Example:

```bash
aws eks update-kubeconfig \
--region ap-south-1 \
--name EKS-CLUSTER
```

Output:

```text
Added new context
arn:aws:eks:ap-south-1:123456789012:cluster/EKS-CLUSTER
```

---

# What is kubeconfig?

kubeconfig is a configuration file used by kubectl.

It stores:

```text
Cluster Information
Authentication Information
Context Information
```

Without kubeconfig:

```bash
kubectl get nodes
```

will fail.

---

# Why kubeconfig is Required?

kubectl needs to know:

```text
Which Cluster?
Which User?
Which API Server?
How to Authenticate?
```

kubeconfig provides all this information.

---

# kubeconfig Location

## Linux

```bash
~/.kube/config
```

## Mac

```bash
~/.kube/config
```

## Windows

```powershell
C:\Users\<username>\.kube\config
```

---

# View kubeconfig

```bash
kubectl config view
```

---

# Step 10: Verify Context

List contexts:

```bash
kubectl config get-contexts
```

Example:

```text
CURRENT   NAME

*         arn:aws:eks:ap-south-1:123456789012:cluster/EKS-CLUSTER
```

Check active context:

```bash
kubectl config current-context
```

---

# What is Context?

Context = Cluster + User + Namespace

Example:

```text
Development Cluster
Production Cluster
Testing Cluster
```

Switch context:

```bash
kubectl config use-context context-name
```

---

# Step 11: Verify Cluster Connectivity

Check Nodes:

```bash
kubectl get nodes
```

Expected:

```text
NAME                             STATUS
ip-10-0-1-100.ec2.internal       Ready
ip-10-0-2-101.ec2.internal       Ready
```

---

# Check Namespaces

```bash
kubectl get ns
```

---

# Check Pods

```bash
kubectl get pods -A
```

---

# Check Services

```bash
kubectl get svc -A
```

---

# Step 12: Test Deployment

Create Deployment:

```bash
kubectl create deployment nginx \
--image=nginx
```

Verify:

```bash
kubectl get pods
```

---

# Expose Deployment

```bash
kubectl expose deployment nginx \
--type=LoadBalancer \
--port=80
```

Verify:

```bash
kubectl get svc
```

---

# Step 13: Delete Test Deployment

```bash
kubectl delete deployment nginx
```

Delete Service:

```bash
kubectl delete svc nginx
```

---

# Common Error 1

```text
Unable to locate credentials
```

Reason:

```text
AWS CLI not configured.
```

Solution:

```bash
aws configure
```

---

# Common Error 2

```text
The connection to the server localhost:8080 was refused
```

Reason:

```text
kubeconfig not generated.
```

Solution:

```bash
aws eks update-kubeconfig \
--region ap-south-1 \
--name my-eks-cluster
```

---

# Common Error 3

```text
You must be logged in to the server (Unauthorized)
```

Reason:

IAM User not authorized in EKS.

Solution:

Add IAM User to EKS access.

---

# Common Error 4

```text
No resources found
```

Reason:

Namespace contains no resources.

---

# Interview Questions

## 1. Why do we connect Local Machine to EKS?

To manage Kubernetes resources remotely using kubectl.

---

## 2. Which tools are required?

* AWS CLI
* kubectl

---

## 3. Why is AWS CLI required?

AWS CLI authenticates with AWS and generates kubeconfig.

---

## 4. Why is kubectl required?

kubectl communicates with Kubernetes API Server.

---

## 5. What is kubeconfig?

A configuration file used by kubectl to connect and authenticate to a Kubernetes cluster.

---

## 6. Where is kubeconfig stored?

Linux/Mac:

```bash
~/.kube/config
```

Windows:

```powershell
C:\Users\<username>\.kube\config
```

---

## 7. How do you generate kubeconfig for EKS?

```bash
aws eks update-kubeconfig \
--region ap-south-1 \
--name my-eks-cluster
```

---

## 8. How do you verify EKS connectivity?

```bash
kubectl get nodes
```

---

## 9. How does kubectl authenticate with EKS?

Using AWS IAM credentials through AWS CLI generated tokens.

---

## 10. What happens internally when update-kubeconfig runs?

1. AWS CLI contacts EKS.
2. Retrieves API Server Endpoint.
3. Retrieves Certificate Authority Data.
4. Updates kubeconfig.
5. Configures IAM authentication.
6. Creates Kubernetes Context.

---

# Complete Flow

```text
Create EKS Cluster
        ↓
Create IAM User
        ↓
Generate Access Key
        ↓
Install AWS CLI
        ↓
aws configure
        ↓
Verify AWS Authentication
        ↓
Install kubectl
        ↓
aws eks update-kubeconfig
        ↓
Generate kubeconfig
        ↓
kubectl get nodes
        ↓
Connected to EKS Successfully
```
