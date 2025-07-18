# GitOps Configuration Management with ArgoCD

## Overview

This project demonstrates advanced GitOps configuration management using **ArgoCD**, integrating **Helm**, **Kustomize**, and **external secret managers** like **Vault** and **AWS Secrets Manager**. It showcases best practices for secure, declarative Kubernetes application deployment.

---


## Project Structure

```bash
.
├── helm-app/
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       └── ingress.yaml
├── kustomize-app/
│   ├── base/
│   │   ├── kustomization.yaml
│   │   └── deployment.yaml
│   └── overlays/
│       ├── dev/
│       │   ├── kustomization.yaml
│       │   └── patch.yaml
│       └── prod/
│           ├── kustomization.yaml
│           └── patch.yaml
└── README.md
```

----

## Objectives
- Integrate **Helm** charts with ArgoCD

- Manage environments with **Kustomize overlays**

- Secure secrets using **Kubernetes, Vault,** and **AWS Secrets Manager**

- Customize **resource management** and **sync policies** in ArgoCD


---


## Prerequisites

- **kubectl:** Kubernetes command-line tool.
- **Helm:** Package manager for Kubernetes.
- **ArgoCD CLI:** For managing ArgoCD.
- **HashiCorp Vault:** For secrets management.
- **AWS CLI:** For AWS Secrets Manager integration.
- **Git:** For version control.
- **A Kubernetes cluster:** Minikube, Kind, or a cloud-based cluster like EKS.
- **A Git repository:** Hosted on GitHub, GitLab, or Bitbucket.


---

## Verify Installed Tools

Ensure the following tools are installed:

```bash
helm version
kubectl version --client
kustomize version
vault -v
aws --version
git --version
argocd version --client
```
![](./img/1a.installed.tools.png)


---


## 1: Managing Configurations with Helm and Kustomize in ArgoCD

### 1.1: Set Up a Git Repository
- Create a Git repository to store your Helm charts and Kustomize configurations.

```bash
mkdir gitops-mini-project
cd gitops-mini-project
git init
touch .gitignore
echo "vault_token.txt" >> .gitignore
git add .gitignore
git commit -m "Initial commit"
```

---

## 1.2: Create a Helm Chart

- Create a Helm chart for a simple application (e.g., a basic Nginx deployment).

```bash
helm create my-app
```

### Edit `my-app/Chart.yaml` to set metadata:

```bash
apiVersion: v2
name: my-app
description: A simple Nginx application
version: 0.1.0
appVersion: "1.16.0"
```


### Edit `my-app/values.yaml` to define default values:
```bash
replicaCount: 1
image:
  repository: nginx
  tag: "latest"
  pullPolicy: IfNotPresent
service:
  type: ClusterIP
  port: 80
ingress:
  enabled: false
```


## 1.3: Deploy Helm Chart via ArgoCD


### Start Minikube and Check Cluster health

```bash
minikube start
minikube status
kubectl get nodes
```


### Set Up ArgoCD
- Install ArgoCD:
```bash
kubectl create namespace argocd
helm repo add argo https://argoproj.github.io/argo-helm
helm install argocd argo/argo-cd --namespace argocd --version 5.46.8
```
![](./img/1b.deployed.argocd.png)


### Get ArgoCD Admin Password:
```bash
INITIAL_PASSWORD=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)
echo "ArgoCD Admin Password: $INITIAL_PASSWORD"
```


### Access ArgoCD UI:
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

- Open `https://localhost:8080` in a browser, log in with username `admin` and the password from above.



### Create ArgoCD Application for Helm: 

- Create a file `helm-app.yaml`:
```bash
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-helm
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-username/gitops-mini-project.git
    targetRevision: main
    path: my-app
    helm:
      valueFiles:
      - values.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```      


### Apply the application:
```bash
kubectl apply -f helm-app.yaml
```
![](./img/1c.kube.apply.png)


### Push to GitHub
```
git add .
git commit -m "new update"
git branch -M main
git remote add origin https://github.com/your-username/gitops-project.git
git push -u origin main
```


### Test Helm Deployment:
```bash
kubectl get pods -n default
kubectl get svc -n default
argocd app get my-app-helm
```