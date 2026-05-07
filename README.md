# EKS Application Deployment Repository Guide

## Project Overview

This repository deploys applications to AWS EKS using:

* Docker
* Amazon ECR
* Kubernetes
* GitHub Actions CI/CD

Application deployment flow:

```text
GitHub Actions
      ↓
Docker Build
      ↓
Amazon ECR
      ↓
Amazon EKS
      ↓
LoadBalancer Service
      ↓
Live Application
```

---

# Repository Name

```text
eks-app-deploy
```

---

# Project Structure

```text
eks-app-deploy/
│
├── app/
│   └── index.html
│
├── Dockerfile
│
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
│
├── .github/
│   └── workflows/
│       └── deploy.yml
│
└── .gitignore
```

---

# Step 1 — Create GitHub Repository

Create repository:

```text
eks-app-deploy
```

---

# Step 2 — Create Application File

File:

```text
app/index.html
```

Code:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Vinod DevOps Project</title>
</head>
<body>
  <h1>Application Deployed to AWS EKS</h1>
</body>
</html>
```

---

# Step 3 — Create Dockerfile

File:

```text
Dockerfile
```

Code:

```dockerfile
FROM nginx:latest

COPY app/index.html /usr/share/nginx/html/index.html
```

---

# Step 4 — Create Amazon ECR Repository

Run:

```bash
aws ecr create-repository \
--repository-name eks-demo-app \
--region us-east-1
```

Expected repository URI:

```text
578339677090.dkr.ecr.us-east-1.amazonaws.com/eks-demo-app
```

---

# Step 5 — Create Kubernetes Deployment

File:

```text
k8s/deployment.yaml
```

Code:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: eks-demo-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: eks-demo-app

  template:
    metadata:
      labels:
        app: eks-demo-app

    spec:
      containers:
      - name: eks-demo-app

        image: 578339677090.dkr.ecr.us-east-1.amazonaws.com/eks-demo-app:latest

        ports:
        - containerPort: 80
```

---

# Step 6 — Create Kubernetes Service

File:

```text
k8s/service.yaml
```

Code:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: eks-demo-service

spec:
  type: LoadBalancer

  selector:
    app: eks-demo-app

  ports:
  - port: 80
    targetPort: 80
```

---

# Step 7 — Create GitHub Actions Workflow

File:

```text
.github/workflows/deploy.yml
```

Code:

```yaml
name: Deploy-App-to-EKS

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:

    - name: Checkout Code
      uses: actions/checkout@v4

    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1

    - name: Login to ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v2

    - name: Build Docker Image
      run: |
        docker build -t eks-demo-app .

    - name: Tag Docker Image
      run: |
        docker tag eks-demo-app:latest 578339677090.dkr.ecr.us-east-1.amazonaws.com/eks-demo-app:latest

    - name: Push Docker Image
      run: |
        docker push 578339677090.dkr.ecr.us-east-1.amazonaws.com/eks-demo-app:latest

    - name: Update kubeconfig
      run: |
        aws eks update-kubeconfig \
        --region us-east-1 \
        --name vinod-eks-cluster

    - name: Deploy to Kubernetes
      run: |
        kubectl apply -f k8s/
```

---

# Step 8 — Add GitHub Secrets

Go to:

```text
GitHub Repo
→ Settings
→ Secrets and variables
→ Actions
```

Add:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

---

# Step 9 — Initialize Git Repository

Run:

```bash
git init
git branch -M main
```

---

# Step 10 — Connect GitHub Repository

Run:

```bash
git remote add origin https://github.com/vDevopsvinod/eks-app-deploy.git
```

---

# Step 11 — Push Code to GitHub

Run:

```bash
git add .
git commit -m "initial app deployment"
git push -u origin main
```

---

# Step 12 — GitHub Actions CI/CD

After push:

* GitHub Actions starts automatically
* Docker image builds
* Image pushes to ECR
* Kubernetes manifests deploy to EKS

---

# Step 13 — Verify Kubernetes Pods

Run locally:

```bash
kubectl get pods
```

Expected:

```text
Running
Running
```

---

# Step 14 — Verify Kubernetes Services

Run:

```bash
kubectl get svc
```

Expected:

```text
EXTERNAL-IP
```

Example:

```text
a123456789.us-east-1.elb.amazonaws.com
```

---

# Step 15 — Open Application

Open LoadBalancer URL in browser.

Expected page:

```text
Application Deployed to AWS EKS
```

---

# Step 16 — Verify ECR Image

Run:

```bash
aws ecr list-images \
--repository-name eks-demo-app \
--region us-east-1
```

---

# Step 17 — Update Application

Modify:

```text
app/index.html
```

Then push:

```bash
git add .
git commit -m "updated application"
git push origin main
```

GitHub Actions automatically redeploys application.

---

# Step 18 — Scale Deployment

Run:

```bash
kubectl scale deployment eks-demo-app --replicas=4
```

Verify:

```bash
kubectl get pods
```

---

# Step 19 — Delete Application

Run:

```bash
kubectl delete -f k8s/
```

---

# Step 20 — Best Practices

* Keep infrastructure repo separate
* Use ECR for images
* Use GitHub Actions for CI/CD
* Use Kubernetes manifests for deployments
* Never hardcode secrets

---

# Final Architecture

```text
GitHub Actions
      ↓
Docker Build
      ↓
Amazon ECR
      ↓
Amazon EKS
      ↓
Kubernetes Deployment
      ↓
AWS LoadBalancer
      ↓
Live Application
```

---

# Technologies Used

* Terraform
* Docker
* Kubernetes
* Amazon EKS
* Amazon ECR
* GitHub Actions
* AWS LoadBalancer

---

# Resume Project Title

```text
CI/CD Application Deployment to AWS EKS using Docker, Kubernetes and GitHub Actions
```
VINOD KUMAR
