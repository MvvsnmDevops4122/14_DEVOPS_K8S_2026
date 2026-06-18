# EBS Dynamic Provisioning Not Working

=====================================

By default, EKS does not provision EBS volumes automatically unless the EBS CSI Driver is installed and properly configured.

---

# Step-by-Step Fix

==================

## Step 1: Verify EBS CSI Driver Installation

By default, EKS clusters may not have the EBS CSI Driver configured correctly.

Check:

```bash
kubectl get pods -n kube-system | grep ebs
```

If no EBS CSI pods are running, install the driver:

```bash
kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.35"
```

### What this does

* Installs EBS CSI Controller Deployment.
* Installs EBS CSI Node DaemonSet.
* Creates required Service Accounts.
* Creates RBAC Roles and RoleBindings.
* Registers the `ebs.csi.aws.com` CSI driver with the cluster.

Verify:

```bash
kubectl get pods -n kube-system | grep ebs
```

Expected:

```text
ebs-csi-controller-xxxxx   Running
ebs-csi-node-xxxxx         Running
```

---

## Step 2: Attach IAM Permissions

Even if the driver is installed, PVCs may remain in Pending state if worker nodes do not have EBS permissions.

### Verify PVC Status

```bash
kubectl get pvc
```

Example:

```text
NAME      STATUS    VOLUME
mypvc     Pending
```

### Fix

1. Go to AWS Console
2. Open IAM
3. Select Roles
4. Find the Worker Node IAM Role

Example:

```text
eksctl-my-cluster-nodegroup-NodeInstanceRole
```

5. Click **Add Permissions**
6. Click **Attach Policies**
7. Attach:

```text
AmazonEBSCSIDriverPolicy
```

### Why?

This policy allows Kubernetes to:

* Create EBS Volumes
* Attach EBS Volumes
* Detach EBS Volumes
* Delete EBS Volumes

---

## Step 3: Verify StorageClass

Check available StorageClasses:

```bash
kubectl get storageclass
```

Example:

```text
NAME            PROVISIONER
gp2 (default)   kubernetes.io/aws-ebs
```

Use the correct StorageClass in PVC:

```yaml
storageClassName: gp2
```

Without a valid StorageClass, dynamic provisioning will fail.

---

## Step 4: Verify PVC Events

Check:

```bash
kubectl describe pvc <pvc-name>
```

Look at:

```text
Events:
```

Common Errors:

### StorageClass Not Found

```text
storageclass.storage.k8s.io "gp2" not found
```

### EBS CSI Driver Missing

```text
waiting for a volume to be created
```

### Permission Issues

```text
could not create volume in EC2
UnauthorizedOperation
```

---

## Step 5: Verify Volume Creation

Check PVs:

```bash
kubectl get pv
```

Expected:

```text
NAME         STATUS   CAPACITY
pvc-xxxxx    Bound    1Gi
```

Check EBS Volume in AWS Console:

```text
EC2
 └── Volumes
      └── New EBS Volume Created
```

---

# Troubleshooting Guide

=======================

## PVC Stuck in Pending

Check:

```bash
kubectl get pvc
kubectl describe pvc <pvc-name>
kubectl get storageclass
```

Verify:

* StorageClass exists.
* EBS CSI Driver is installed.
* IAM permissions are attached.

---

## No PV Created

Check:

```bash
kubectl get pv
```

If no PV exists:

* Verify EBS CSI Controller Pods.
* Verify IAM Role permissions.

---

## PVC Bound but Pod Pending

Check:

```bash
kubectl describe pod <pod-name>
```

Possible causes:

* Node Affinity mismatch
* EBS attached to different Availability Zone
* Scheduling issues

---

## Verify EBS CSI Driver

```bash
kubectl get csidriver
```

Expected:

```text
ebs.csi.aws.com
```

---

# Interview Scenario

====================

### PVC is stuck in Pending state. How do you troubleshoot?

> First, I check the PVC status using `kubectl get pvc` and inspect events using `kubectl describe pvc`. Then I verify that the EBS CSI Driver is installed and running. Next, I check whether the worker node IAM role has the `AmazonEBSCSIDriverPolicy` attached. Finally, I verify that the correct StorageClass is configured. Once these issues are resolved, the PVC should dynamically provision an EBS volume and move to Bound state.

---

# Summary

=========

```text
PVC Pending
      ↓
Check StorageClass
      ↓
Check EBS CSI Driver
      ↓
Check IAM Permissions
      ↓
Verify PV Creation
      ↓
PVC Bound
```
