<h1>Project-7</h1>

### **Project: GitOps with Kubernetes and GitHub**

The goal of this project is to implement GitOps using GitHub as the **source of truth** for managing Kubernetes deployments. In this setup, Kubernetes manifests or Helm charts are stored in a GitHub repository, and a GitOps tool like **ArgoCD** or **FluxCD** is used to automatically apply changes to the Kubernetes cluster when updates are made to the GitHub repository.

---

### **1. Project Directory Structure**

```bash
gitops-kubernetes/
│
├── .github/
│   └── workflows/
│       └── kubernetes-deployment.yml    # GitHub Actions workflow file
│
├── manifests/                           # Kubernetes YAML files
│   ├── deployment.yaml                  # Application deployment manifest
│   ├── service.yaml                     # Service manifest
│   └── ingress.yaml                     # Ingress manifest (optional)
│
├── charts/                              # Helm charts (optional if using Helm)
│   └── my-app/
│       ├── Chart.yaml                   # Helm chart metadata
│       ├── values.yaml                  # Default configuration values
│       └── templates/
│           ├── deployment.yaml          # Deployment template
│           ├── service.yaml             # Service template
│           └── ingress.yaml             # Ingress template
│
├── .gitignore                           # Git ignore file
└── README.md                            # Project documentation
```

### **2. Project Files Content**

#### **2.1. manifests/deployment.yaml**

A simple Kubernetes Deployment for an application:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
  labels:
    app: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app-container
          image: nginx:latest
          ports:
            - containerPort: 80
```

#### **2.2. manifests/service.yaml**

Service for exposing the application within the cluster:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

#### **2.3. manifests/ingress.yaml** (Optional)

Ingress to expose the application externally (if using Ingress Controller):

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app-service
                port:
                  number: 80
```

#### **2.4. charts/my-app/Chart.yaml** (Optional, for Helm)

A basic Helm chart metadata file for deploying your app:

```yaml
apiVersion: v2
name: my-app
description: A Helm chart for Kubernetes deployment of my-app

# The version of the application
version: 0.1.0
appVersion: "1.0"
```

#### **2.5. charts/my-app/values.yaml**

Helm values file for configurable parameters:

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
```

#### **2.6. .github/workflows/kubernetes-deployment.yml**

This GitHub Actions workflow deploys Kubernetes resources whenever there are updates to the manifests or Helm charts:

```yaml
name: Kubernetes Deployment

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Kubernetes
      uses: azure/k8s-set-context@v2
      with:
        method: kubeconfig
        kubeconfig: ${{ secrets.KUBECONFIG }}

    - name: Apply Kubernetes manifests
      run: |
        kubectl apply -f manifests/
      env:
        KUBECONFIG: ${{ secrets.KUBECONFIG }}

    - name: Apply Helm chart (Optional)
      run: |
        helm upgrade --install my-app ./charts/my-app
      env:
        KUBECONFIG: ${{ secrets.KUBECONFIG }}
```

This workflow:
- Is triggered by pushes to the `main` branch.
- Checks out the repository.
- Sets up Kubernetes using a `KUBECONFIG` stored in GitHub secrets.
- Deploys the Kubernetes manifests using `kubectl` or deploys Helm charts using `helm`.

> **Note:** Store the Kubernetes credentials (KUBECONFIG) in GitHub Secrets for security.

#### **2.7. .gitignore**

Ignore files like `kubeconfig` or sensitive files that shouldn’t be committed:

```bash
# Ignore Kubernetes config and sensitive files
kubeconfig
```

#### **2.8. README.md**

```markdown
# GitOps with Kubernetes and GitHub

## Overview

This project demonstrates a GitOps workflow to manage Kubernetes deployments using GitHub and ArgoCD/FluxCD. The repository contains Kubernetes manifests for deploying a sample NGINX application, which are automatically applied to a Kubernetes cluster using GitHub Actions.

## Setup

### Prerequisites

- Kubernetes cluster (Minikube, AWS EKS, GKE, etc.).
- GitHub repository with Kubernetes manifests or Helm charts.
- ArgoCD or FluxCD installed on your Kubernetes cluster.

### Local Deployment

To deploy the application manually on your Kubernetes cluster:

```bash
kubectl apply -f manifests/
```

### GitOps Automation with GitHub Actions

The CI/CD pipeline is configured in `.github/workflows/kubernetes-deployment.yml`. It automatically applies Kubernetes changes when code is pushed to the `main` branch. The workflow:
- Uses `kubectl` to apply manifests.
- Optionally uses Helm to manage applications.

### ArgoCD/FluxCD Setup

- **ArgoCD**: Install ArgoCD on your Kubernetes cluster and configure it to sync with this repository.
- **FluxCD**: Install FluxCD and set up the repository as a source of truth.

### GitHub Secrets

To securely manage Kubernetes credentials, add the following GitHub secrets:
- `KUBECONFIG`: Your Kubernetes config file content.

## Conclusion

This project showcases the power of GitOps for automating Kubernetes deployments using GitHub as the single source of truth. Changes in this repository are automatically reflected in the Kubernetes cluster.
```

---

### **3. Steps to Set Up the GitOps Project**

1. **Create a GitHub Repository**: Create a new GitHub repository for your Kubernetes manifests or Helm charts.
2. **Clone the Repository**:

   ```bash
   git clone https://github.com/your-username/gitops-kubernetes.git
   cd gitops-kubernetes
   ```

3. **Create Directory Structure**: Set up the directory structure and add the Kubernetes manifests or Helm charts.
4. **Commit and Push the Changes**:

   ```bash
   git add .
   git commit -m "Initial GitOps setup with Kubernetes"
   git push -u origin main
   ```

5. **Set Up ArgoCD/FluxCD**:
   - **ArgoCD**: Install ArgoCD on your Kubernetes cluster and configure the repository as an application source.
   - **FluxCD**: Set up FluxCD to automatically sync changes from your GitHub repository.

6. **Configure GitHub Secrets**:
   - Go to your GitHub repository settings.
   - Add the following secrets:
     - `KUBECONFIG`: Your Kubernetes cluster configuration.

7. **Test the CI/CD Pipeline**: Push changes to the `main` branch and observe GitHub Actions automatically deploying changes to your Kubernetes cluster.

---

### **4. How GitOps Workflow Functions**

- **GitHub as Source of Truth**: Kubernetes manifests or Helm charts are stored in GitHub.
- **GitOps Tool (ArgoCD/FluxCD)**: Watches the repository for changes and automatically applies those changes to the Kubernetes cluster.
- **CI/CD Pipeline**: GitHub Actions ensure that every code commit triggers Kubernetes deployment workflows (`kubectl apply` or Helm `install/upgrade`).

This setup allows for **continuous delivery of infrastructure** changes, empowering teams to manage Kubernetes applications using a GitOps model. All changes to your Kubernetes cluster are driven by Git operations, ensuring traceability and version control.

Let me know if you'd like further clarification or additional features!
