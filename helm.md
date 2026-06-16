# Helm in Kubernetes

## What is Helm?

Helm is a package manager for Kubernetes, similar to **yum**, **apt**, or **dnf** in Linux.

It helps package, deploy, upgrade, and manage Kubernetes applications using reusable packages called **Charts**.

---

# Key Concepts

## 1. Chart

A **Helm Chart** is a reusable package of Kubernetes YAML templates and configuration files used to deploy and manage applications in a Kubernetes cluster.

### Chart Structure

```text
javawebapp/
├── Chart.yaml          # Metadata about the chart
├── values.yaml         # Configuration values
├── charts/             # Dependency charts (subcharts)
└── templates/          # Kubernetes resource templates
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    └── configmap.yaml
```

---

## 2. Release

A **Release** is an instance of a Helm Chart deployed in a Kubernetes cluster.

Example:

```bash
helm install myapp ./javawebapp
```

* Chart → `javawebapp`
* Release → `myapp`

A release maintains:

* Deployment history
* Upgrade history
* Rollback capability

---

## 3. Repository

A repository stores Helm Charts.

Examples:

* Artifact Hub
* Internal Helm Repository

Commands:

```bash
helm repo add <repo-name> <repo-url>
helm repo list
helm repo update
```

---

# Helm Architecture

## Helm 2

Components:

* Helm Client
* Tiller Server

Issues:

* Security concerns
* Additional maintenance

---

## Helm 3

Components:

* Helm Client only

Benefits:

* No Tiller
* Better security
* Simpler architecture

---

# Helm Installation

```bash
curl -fsSL -o get_helm.sh \
https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

chmod 700 get_helm.sh

sh get_helm.sh
```

Verify:

```bash
helm version
```

---

# Uninstall Helm

```bash
sudo rm /usr/local/bin/helm

rm -rf ~/.config/helm

rm get_helm.sh
```

---

# Chart.yaml

`Chart.yaml` is the metadata file of a Helm Chart.

Example:

```yaml
apiVersion: v2
name: javawebapp
description: Java Web Application Helm Chart
version: 1.0.0
appVersion: "1.0.1"
```

### Important Fields

| Field       | Description            |
| ----------- | ---------------------- |
| apiVersion  | Helm Chart API Version |
| name        | Chart Name             |
| description | Chart Description      |
| version     | Chart Version          |
| appVersion  | Application Version    |

---

# values.yaml

`values.yaml` is a configuration file that stores configurable values used by Helm templates.

Example:

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: "1.25"

service:
  type: ClusterIP
  port: 80
```

### Environment Specific Values

**dev-values.yaml**

```yaml
replicaCount: 2
```

**prod-values.yaml**

```yaml
replicaCount: 5
```

Deployment:

```bash
helm upgrade --install app \
-f prod-values.yaml \
./chart
```

---

# Helm Templates

Helm Templates are Kubernetes YAML files that contain placeholders (variables).

Helm replaces these placeholders with values from `values.yaml` and generates final Kubernetes manifests.

Example:

```yaml
replicas: {{ .Values.replicaCount }}

image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

Rendered Output:

```yaml
replicas: 2

image: nginx:1.25
```

---

# Common Template Variables

```yaml
{{ .Values.replicaCount }}
{{ .Values.image.repository }}
{{ .Values.image.tag }}

{{ .Release.Name }}
{{ .Release.Namespace }}
```

---

# Deploy Third-Party Applications Using Helm

## Example: Metrics Server

### Verify Cluster

```bash
kubectl config current-context

kubectl top nodes

kubectl top pods
```

---

## Add Repository

```bash
helm repo add metrics-server \
https://kubernetes-sigs.github.io/metrics-server/

helm repo update
```

---

## Verify Repository

```bash
helm repo list
```

---

## Search Charts

```bash
helm search repo metrics-server
```

---

## View Default Values

```bash
helm show values metrics-server/metrics-server
```

---

## View Templates

```bash
helm template metrics-server/metrics-server
```

---

## Dry Run

```bash
helm upgrade --install metrics-server \
metrics-server/metrics-server \
-n test-ns \
--dry-run
```

---

## Install

```bash
helm upgrade --install metrics-server \
metrics-server/metrics-server \
-n test-ns
```

---

## Verify Installation

```bash
helm ls -A

kubectl top nodes

kubectl top pods
```

---

# Upgrade Release

```bash
helm upgrade --install metrics-server \
metrics-server/metrics-server \
--set replicas=3
```

---

# Rollback Release

```bash
helm history metrics-server

helm rollback metrics-server <revision>
```

---

# Uninstall Release

```bash
helm uninstall metrics-server
```

---

# Using Custom Values File

Create:

```yaml
# metricservervalues.yaml

replicas: 2

resources:
  requests:
    cpu: 300m
    memory: 512Mi
```

Deploy:

```bash
helm upgrade --install metrics-server \
-f metricservervalues.yaml \
metrics-server/metrics-server
```

Verify:

```bash
kubectl describe pod <pod-name>
```

---

# Create Your Own Helm Chart

## Create Chart

```bash
helm create javawebapp
```

---

## Validate Chart

```bash
helm lint javawebapp
```

---

## values.yaml

```yaml
replicaCount: 2

image:
  repository: kkeducation123456/java-web-app
  tag: "1.0.1"

service:
  type: NodePort
  port: 80
  targetPort: 8080
```

---

## deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: {{ .Release.Name }}

spec:
  replicas: {{ .Values.replicaCount }}

  selector:
    matchLabels:
      app: {{ .Release.Name }}

  template:
    metadata:
      labels:
        app: {{ .Release.Name }}

    spec:
      containers:
      - name: {{ .Release.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

---

## service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: {{ .Release.Name }}-service

spec:
  type: {{ .Values.service.type }}

  selector:
    app: {{ .Release.Name }}

  ports:
  - port: {{ .Values.service.port }}
    targetPort: {{ .Values.service.targetPort }}
```

---

# Package Chart

```bash
helm package javawebapp
```

Output:

```text
javawebapp-1.0.0.tgz
```

---

# Install Packaged Chart

```bash
helm upgrade --install javawebapp \
javawebapp-1.0.0.tgz \
-n prod
```

---

# Useful Helm Commands

```bash
helm create <chart>

helm lint <chart>

helm template <chart>

helm install <release> <chart>

helm upgrade <release> <chart>

helm upgrade --install <release> <chart>

helm history <release>

helm rollback <release> <revision>

helm status <release>

helm uninstall <release>

helm package <chart>

helm repo add <repo> <url>

helm repo list

helm search repo <chart>

helm show chart <chart>

helm show values <chart>
```

---

# Best Practices

* Store Helm Charts in Git repositories.
* Use separate values files for Dev, UAT, and Production.
* Avoid excessive use of `--set`.
* Use chart versioning.
* Validate charts using `helm lint`.
* Use `helm template` before deployment.
* Use `--dry-run` before production changes.
* Secure sensitive values using Kubernetes Secrets.
* Use rollback during failed deployments.
* Keep charts reusable and modular.

---

# Common Helm Charts

* NGINX
* Prometheus
* Grafana
* MySQL
* PostgreSQL
* Jenkins
* Metrics Server
* ArgoCD
* Elasticsearch
* Kafka
