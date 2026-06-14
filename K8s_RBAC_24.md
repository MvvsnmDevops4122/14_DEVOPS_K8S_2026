
# Kubernetes RBAC (Role-Based Access Control)

## What is RBAC?

RBAC (Role-Based Access Control) in Kubernetes is used to control access to Kubernetes resources based on roles assigned to users, groups, or service accounts.

---

# RBAC Components

## 1. Role

A Role defines a set of permissions within a specific namespace.

Example resources:

* Pods
* Deployments
* Services
* ConfigMaps

A Role can only grant access within its namespace.

---

## 2. RoleBinding

A RoleBinding grants the permissions defined in a Role to a user, group, or service account within a namespace.

```text
Role + User = RoleBinding
```

---

## 3. ClusterRole

A ClusterRole defines permissions at the cluster level.

Used for:

* Cluster-wide access
* Non-namespaced resources
* Administrative permissions

Examples:

* Nodes
* Namespaces
* PersistentVolumes

---

## 4. ClusterRoleBinding

A ClusterRoleBinding grants the permissions defined in a ClusterRole to a user, group, or service account across the entire cluster.

```text
ClusterRole + User = ClusterRoleBinding
```

---

# Practical Implementation

## Step 1: Install kubectl (Windows)

Official Documentation:

[https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)

Download:

```bash
curl.exe -LO "https://dl.k8s.io/release/v1.34.0/bin/windows/amd64/kubectl.exe"
```

Add the kubectl executable path to the Windows Environment Variables.

Example:

```text
C:\Users\<username>\Desktop\rbac-practise
```

Verify:

```bash
kubectl version --client
```

---

## Step 2: Install AWS CLI

Official Documentation:

[https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

Download:

[https://awscli.amazonaws.com/AWSCLIV2.msi](https://awscli.amazonaws.com/AWSCLIV2.msi)

Verify:

```bash
aws --version
```

---

## Step 3: Create IAM User and Generate Access Keys

# Create IAM User

Open AWS Console.
Navigate to IAM → Users.
Click Create User.
Enter Username.
Attach required permissions.

Example:

eks-readonly-user

# Create Access Key

Open IAM User.
Go to Security Credentials.
Click Create Access Key.
Choose CLI Access.
Copy:
Access Key ID
Secret Access Key

⚠️ Never commit credentials to GitHub.

## Step 3: Configure AWS CLI

```bash
aws configure
```

Provide:

```text
AWS Access Key ID
AWS Secret Access Key
Region: ap-south-1
Output Format: table
```

Verify:

```bash
aws sts get-caller-identity
```

---

## Step 4: Connect to EKS Cluster

List EKS Clusters:

```bash
aws eks list-clusters
```

Update kubeconfig:

```bash
aws eks update-kubeconfig \
--name my-demo-cluster \
--region ap-south-1
```

Verify connection:

```bash
kubectl get nodes
```

---

# EKS Authentication

In EKS, IAM authentication is configured using the aws-auth ConfigMap.

Check:

```bash
kubectl get cm aws-auth -n kube-system
```

Edit:

```bash
kubectl edit cm aws-auth -n kube-system
```

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system

data:
  mapRoles: |
    - groups:
      - system:bootstrappers
      - system:nodes
      rolearn: arn:aws:iam::<ACCOUNT_ID>:role/EKS-worker-role
      username: system:node:{{EC2PrivateDNSName}}

  mapUsers: |
    - userarn: arn:aws:iam::<ACCOUNT_ID>:user/devuser
      username: devuser
```

Note:

Adding a user to aws-auth only authenticates the user.

RBAC permissions still need to be assigned separately.

---

# Create Role and RoleBinding

## Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: prod
  name: pod-reader

rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

## RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: prod

subjects:
- kind: User
  name: kkfunda1
  apiGroup: rbac.authorization.k8s.io

roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Apply:

```bash
kubectl apply -f role-rolebinding.yaml
```

Verify:

```bash
kubectl get role -n prod
kubectl get rolebinding -n prod
```

---

# Create ClusterRole and ClusterRoleBinding

## ClusterRole

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: fullaccesscr

rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
```

## ClusterRoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: fullaccess

subjects:
- kind: User
  name: kkfunda1
  apiGroup: rbac.authorization.k8s.io

roleRef:
  kind: ClusterRole
  name: fullaccesscr
  apiGroup: rbac.authorization.k8s.io
```

Apply:

```bash
kubectl apply -f clusterrole-clusterrolebinding.yaml
```

Verify:

```bash
kubectl get clusterrole
kubectl get clusterrolebinding
```

---

# Common RBAC Commands

Check Roles:

```bash
kubectl get roles -A
```

Check RoleBindings:

```bash
kubectl get rolebindings -A
```

Check ClusterRoles:

```bash
kubectl get clusterroles
```

Check ClusterRoleBindings:

```bash
kubectl get clusterrolebindings
```

Describe Role:

```bash
kubectl describe role pod-reader -n prod
```

Describe ClusterRole:

```bash
kubectl describe clusterrole fullaccesscr
```
