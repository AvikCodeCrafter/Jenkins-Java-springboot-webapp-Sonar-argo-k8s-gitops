# 🚀 Argo CD on Amazon EKS -- Complete GitOps Setup Guide

This document provides a complete production-grade Argo CD setup on
Amazon EKS including installation, exposure, authentication, application
deployment, CI/CD integration, and GitOps workflow.

------------------------------------------------------------------------

# 🎨 Architecture Diagram

``` mermaid
flowchart LR
    Dev[Developer] -->|Push Code| GitHub
    GitHub -->|Trigger CI| Jenkins
    Jenkins -->|Build Docker Image| DockerHub
    Jenkins -->|Update Image Tag| GitOpsRepo[GitOps Repository]
    GitOpsRepo -->|Detected Change| ArgoCD
    ArgoCD -->|Sync| EKS[Amazon EKS Cluster]
    EKS -->|Deploy| App[Spring Boot Application]
```

------------------------------------------------------------------------

# 📌 Architecture Flow

Developer → GitHub → Jenkins CI\
Jenkins → Docker Build → DockerHub\
Jenkins → Update GitOps Manifests\
Argo CD → Sync → Amazon EKS

Developer → GitHub (App Repo)
            ↓
        Jenkins CI
            ↓
     Docker Build & Push
            ↓
     Update GitOps Repo
            ↓
        Argo CD Sync
            ↓
        Amazon EKS


------------------------------------------------------------------------

# 🏗 Prerequisites

-   Amazon EKS Cluster
-   kubectl configured
-   AWS CLI configured
-   IAM permissions for LoadBalancer
-   GitHub repository with Kubernetes manifests

Verify cluster:

``` bash
kubectl get nodes
```

------------------------------------------------------------------------

# 🛠 Step 1: Install Argo CD (Official Method)

## 1️⃣ Create Namespace

``` bash
kubectl create namespace argocd
```

## 2️⃣ Install Argo CD

``` bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## 3️⃣ Verify Installation

``` bash
kubectl get pods -n argocd
```

------------------------------------------------------------------------

# 🌐 Step 2: Expose Argo CD

## Option A --- NodePort

``` bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
kubectl get svc -n argocd
```

Access: https://`<NodeIP>`{=html}:`<NodePort>`{=html}

------------------------------------------------------------------------

## Option B --- LoadBalancer (Recommended)

``` bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
kubectl get svc -n argocd
```

Access: https://`<LoadBalancer-DNS>`{=html}

------------------------------------------------------------------------

# 🔐 Step 3: Get Admin Password

``` bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d && echo
```

Username: admin\
Password: `<command output>`{=html}

------------------------------------------------------------------------

# 🔁 Step 4: Reset Admin Password (Optional)

Generate bcrypt hash:

``` bash
htpasswd -nbBC 10 "" newpassword | tr -d ':
' | sed 's/$2y/$2a/'
```

Patch secret:

``` bash
kubectl patch secret argocd-secret -n argocd -p '{"stringData": {"admin.password": "<bcrypt-hash>", "admin.passwordMtime": "'$(date +%FT%T%Z)'"}}'
```

Restart:

``` bash
kubectl rollout restart deployment argocd-server -n argocd
```

------------------------------------------------------------------------

# 📂 Step 5: GitOps Repository Structure

. ├── GitOps/ │ ├── deployment.yml │ ├── service.yml │ └── ingress.yml

Important: Path in Argo CD must be: GitOps

------------------------------------------------------------------------

# 📦 Step 6: Create Application in Argo CD

Application Name: java-web-app\
Project: default\
Repo URL: your GitHub repo\
Revision: HEAD\
Path: GitOps\
Cluster: https://kubernetes.default.svc\
Namespace: default

Click Create → Sync

------------------------------------------------------------------------

# 🔄 Step 7: Enable Auto Sync

``` bash
argocd app set java-web-app --sync-policy automated
```

------------------------------------------------------------------------

# 🔁 Complete GitOps Flow

1.  Developer pushes code\
2.  Jenkins builds Docker image\
3.  Docker image pushed to DockerHub\
4.  Jenkins updates GitOps deployment file\
5.  Commit pushed to GitHub\
6.  Argo CD detects change\
7.  Argo CD syncs to EKS\
8.  Application deployed automatically

------------------------------------------------------------------------

# 🛡 Production Best Practices

-   Use AWS ALB Ingress Controller
-   Enable HTTPS via ACM
-   Configure RBAC
-   Integrate OIDC/SSO
-   Enable Resource Health Checks
-   Use Helm/Kustomize for multi-env
-   Monitor with Prometheus & Grafana

------------------------------------------------------------------------

# 🎯 Outcome

Fully automated CI/CD\
GitOps-driven deployment\
Production-ready EKS integration\
Secure admin management

------------------------------------------------------------------------

🔥 Argo CD GitOps deployment on Amazon EKS is now production-ready.
