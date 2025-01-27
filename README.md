# 🔒 External Secrets Operator Setup for Kubernetes

This repository demonstrates how to configure the External Secrets Operator (ESO) to manage secrets in Kubernetes using Google Secret Manager as the backend.

## 📋 Prerequisites
- 🖥️ Kubernetes cluster (v1.22 or later recommended)
- 🎛️ Helm installed on your local machine
- 🔑 Access to Google Secret Manager
- 🌐 Workload Identity Federation or Service Account Key for authentication

## 🚀 Setup Instructions

### 1️⃣ Install External Secrets Operator
Add the Helm repository and install ESO:
```bash
helm repo add external-secrets https://charts.external-secrets.io
helm repo update
helm install external-secrets external-secrets/external-secrets 
```

Verify the installation:

```bash
kubectl get pods 
kubectl get crds | grep external
```
---

### 2️⃣ Create a SecretStore Resource
Define a SecretStore to connect to Google Secret Manager:

```bash
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: gcp-secret-store
spec:
  provider:
    gcp:
      projectID: <your-gcp-project-id>
```

Apply the resource:

```bash
kubectl apply -f secretstore.yaml
```

---

### 3️⃣ Create an ExternalSecret Resource
Define an ExternalSecret to sync a secret from Google Secret Manager to Kubernetes:

```bash
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: my-secret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: gcp-secret-store
    kind: SecretStore
  target:
    name: my-k8s-secret
    creationPolicy: Owner
  data:
    - secretKey: my-key
      remoteRef:
        key: <google-secret-key-name>
```
Apply the resource:

```bash
kubectl apply -f externalsecret.yaml
```

---

### 4️⃣ ✅ Validate the Setup
Check if the Kubernetes Secret is created:

```bash
kubectl get secrets
kubectl describe secret my-k8s-secret
```

---

### 5️⃣ 🔗 Using the Secret in Applications
Mount the secret as an environment variable or a volume in your pods. Example deployment:

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app-container
        image: nginx
        env:
        - name: MY_SECRET_KEY
          valueFrom:
            secretKeyRef:
              name: my-k8s-secret
              key: my-key
```

Apply the deployment:

```bash
kubectl apply -f deployment.yaml
```

---

### 6️⃣ 🧹 Clean Up
Remove the resources:

```bash
kubectl delete -f deployment.yaml
kubectl delete -f externalsecret.yaml
kubectl delete -f secretstore.yaml
helm uninstall external-secrets
```
