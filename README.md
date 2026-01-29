# FastAPI + Helm + ArgoCD - End-to-End DevOps Workflow 🚀

A complete end-to-end DevOps portfolio project demonstrating GitOps principles, CI/CD automation, Kubernetes orchestration, and comprehensive monitoring using industry-standard tools.

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Technologies Used](#technologies-used)
- [Prerequisites](#prerequisites)
- [Complete Setup Guide](#complete-setup-guide)
- [Accessing Services](#accessing-services)
- [CI/CD Pipeline](#cicd-pipeline)
- [Monitoring & Observability](#monitoring--observability)
- [Repository Structure](#repository-structure)
- [Troubleshooting](#troubleshooting)

## 🎯 Project Overview

This project implements a **GitOps-based deployment pipeline** for a FastAPI application using:
- **CI/CD**: GitHub Actions for automated testing, building, and deployment
- **Containerization**: Docker for packaging the application
- **Orchestration**: Kubernetes for container orchestration
- **GitOps**: ArgoCD for declarative continuous deployment
- **Monitoring**: Prometheus & Grafana for metrics and observability
- **IaC**: Helm charts for templated Kubernetes deployments

## 🏗️ Architecture

```
┌─────────────────┐
│  Developer      │
│  Push Code      │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│     GitHub (Source Code Repository)      │
└────────┬───────────────────┬────────────┘
         │                   │
         ▼                   ▼
┌────────────────┐   ┌──────────────────┐
│ GitHub Actions │   │   ArgoCD Sync    │
│   CI Pipeline  │   │   (GitOps)       │
└────────┬───────┘   └─────────┬────────┘
         │                     │
         ▼                     ▼
┌────────────────┐   ┌──────────────────┐
│  Docker Hub    │   │   Kubernetes     │
│  Image Repo    │   │   Cluster        │
└────────────────┘   └─────────┬────────┘
                               │
                    ┌──────────┼──────────┐
                    ▼          ▼          ▼
              ┌─────────┬─────────┬──────────┐
              │FastAPI  │ArgoCD   │Prometheus│
              │   App   │  UI     │ Grafana  │
              └─────────┴─────────┴──────────┘
```

## 🛠️ Technologies Used

### Core Stack
- **Application**: [FastAPI](https://fastapi.tiangolo.com/) - Modern, high-performance Python web framework
- **Containerization**: [Docker](https://www.docker.com/) - Container platform
- **Orchestration**: [Kubernetes](https://kubernetes.io/) - Container orchestration (Minikube for local)
- **Package Manager**: [Helm](https://helm.sh/) - Kubernetes package manager

### DevOps Tools
- **GitOps**: [ArgoCD](https://argo-cd.readthedocs.io/) - Declarative GitOps continuous delivery
- **CI/CD**: [GitHub Actions](https://github.com/features/actions) - Automated workflows
- **Monitoring**: [Prometheus](https://prometheus.io/) - Metrics collection & alerting
- **Visualization**: [Grafana](https://grafana.com/) - Metrics dashboards
- **Testing**: [Pytest](https://pytest.org/) - Python testing framework

### Additional Components
- **Ingress**: NGINX Ingress Controller - Load balancing and routing
- **Service Mesh**: Prometheus Operator (kube-prometheus-stack)

## ✅ Prerequisites

Before starting, ensure you have the following installed:

### Required Tools
```bash
# Check if tools are installed
docker --version          # Docker 20.10+
kubectl version --client  # Kubernetes CLI 1.25+
minikube version         # Minikube 1.30+
helm version             # Helm 3.10+
git --version            # Git 2.30+
```

### Installation Commands

#### macOS (using Homebrew)
```bash
# Install Homebrew if not installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install required tools
brew install kubectl
brew install minikube
brew install helm
brew install docker
```

#### Linux (Ubuntu/Debian)
```bash
# Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### Accounts Required
- **Docker Hub Account**: [Sign up here](https://hub.docker.com/signup)
- **GitHub Account**: For forking and CI/CD

## 🚀 Complete Setup Guide

### Step 1: Clone the Repository

```bash
# Clone the repository
git clone https://github.com/omerbeithalahmy/fastapi-helm-argocd.git
cd fastapi-helm-argocd

# Fork the repository on GitHub first, then clone your fork
git clone https://github.com/YOUR_USERNAME/fastapi-helm-argocd.git
cd fastapi-helm-argocd
```

### Step 2: Start Minikube Cluster

```bash
# Start Minikube with recommended resources
minikube start --cpus=4 --memory=8192 --driver=docker

# Enable required addons
minikube addons enable ingress
minikube addons enable metrics-server

# Verify cluster is running
kubectl cluster-info
kubectl get nodes
```

### Step 3: Install ArgoCD

```bash
# Create ArgoCD namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for ArgoCD to be ready
kubectl wait --for=condition=available --timeout=300s deployment/argocd-server -n argocd

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
echo ""  # Print newline

# Port-forward ArgoCD UI (keep this terminal open)
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

**ArgoCD UI**: https://localhost:8080
- **Username**: `admin`
- **Password**: (from command above)

### Step 4: Install Prometheus & Grafana

```bash
# Add Prometheus Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Create monitoring namespace
kubectl create namespace monitoring

# Install kube-prometheus-stack (includes Prometheus + Grafana)
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false

# Wait for Prometheus stack to be ready
kubectl wait --for=condition=available --timeout=300s deployment/kube-prometheus-stack-operator -n monitoring

# Verify installation
kubectl get pods -n monitoring
```

### Step 5: Configure GitHub Secrets

For CI/CD pipeline to work, add the following secrets to your GitHub repository:

1. Go to: **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

2. Add these secrets:
   - `DOCKER_USERNAME`: Your Docker Hub username
   - `DOCKER_PASSWORD`: Your Docker Hub password or access token

### Step 6: Deploy Application with ArgoCD

```bash
# Apply ArgoCD Application manifest
kubectl apply -f manifests/application.yaml

# Verify application is syncing
kubectl get application -n argocd

# Watch deployment status
kubectl get pods -w
```

### Step 7: Set Up Monitoring (Optional)

```bash
# Apply ServiceMonitor for ArgoCD metrics
kubectl apply -f manifests/argocd-monitor.yaml

# Apply ArgoCD notifications (optional - requires Slack webhook)
# kubectl apply -f manifests/argo-notifications-setup.yaml
```

## 🌐 Accessing Services

### FastAPI Application

```bash
# Get Minikube IP
minikube ip

# Add to /etc/hosts (replace <MINIKUBE_IP> with output from above)
echo "$(minikube ip) fastapi.local" | sudo tee -a /etc/hosts

# Access application
curl http://fastapi.local
# OR open in browser: http://fastapi.local

# Alternative: Port-forward method
kubectl port-forward svc/fastapi-app 8000:8000
# Access: http://localhost:8000
```

**Expected Response**:
```json
{"message": "Hello from FastAPI Helm ArgoCD demo!"}
```

### ArgoCD Dashboard

```bash
# Port-forward ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Access: https://localhost:8080
# Username: admin
# Password: (get with command below)
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

**ArgoCD Features**:
- ✅ View application sync status
- ✅ Manual sync/refresh applications
- ✅ View deployment history and rollbacks
- ✅ Monitor resource health
- ✅ View live Kubernetes manifests

### Prometheus Dashboard

```bash
# Port-forward Prometheus
kubectl port-forward svc/kube-prometheus-stack-prometheus -n monitoring 9090:9090

# Access: http://localhost:9090
```

**Useful Prometheus Queries**:
```promql
# CPU usage
container_cpu_usage_seconds_total

# Memory usage
container_memory_usage_bytes

# ArgoCD app sync status
argocd_app_info

# Pod restart count
kube_pod_container_status_restarts_total
```

### Grafana Dashboard

```bash
# Get Grafana admin password
kubectl get secret kube-prometheus-stack-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 -d && echo

# Port-forward Grafana
kubectl port-forward svc/kube-prometheus-stack-grafana -n monitoring 3000:80

# Access: http://localhost:3000
# Username: admin
# Password: (from command above)
```

**Pre-configured Dashboards**:
- Kubernetes / Compute Resources / Cluster
- Kubernetes / Compute Resources / Namespace (Pods)
- ArgoCD Metrics (if ServiceMonitor applied)

## 🔄 CI/CD Pipeline

### Pipeline Workflow

The GitHub Actions pipeline (`.github/workflows/ci-build.yaml`) automatically:

**Triggers on**:
   - Push to `main` branch
   - Pull requests to `main`
   - Ignores changes to `helm/` and `manifests/` directories

**Pipeline Steps**:
   ```
   ┌──────────────────┐
   │  Code Checkout   │
   └────────┬─────────┘
            │
   ┌────────▼─────────┐
   │  Setup Python    │
   └────────┬─────────┘
            │
   ┌────────▼─────────┐
   │  Run Pytest      │
   └────────┬─────────┘
            │
   ┌────────▼─────────┐
   │  Docker Build    │
   └────────┬─────────┘
            │
   ┌────────▼─────────┐
   │  Push to Hub     │
   │  (SHA + latest)  │
   └────────┬─────────┘
            │
   ┌────────▼─────────┐
   │  Update Helm     │
   │  values.yaml     │
   └────────┬─────────┘
            │
   ┌────────▼─────────┐
   │  Git Commit      │
   │  & Push          │
   └──────────────────┘
   ```

**Docker Image Tags**:
   - `omerbh7/fastapi-helm-argocd:<git-sha-short>`
   - `omerbh7/fastapi-helm-argocd:latest`

**GitOps Flow**:
   - Pipeline updates `helm/fastapi-chart/values.yaml` with new image tag
   - Commits and pushes changes back to repository
   - ArgoCD detects Git changes and syncs automatically
   - New pods deployed with updated image

### Testing the Pipeline

```bash
# Make a change to the application
echo "# Test change" >> app/app.py

# Commit and push
git add app/app.py
git commit -m "test: trigger CI/CD pipeline"
git push origin main

# Watch GitHub Actions
# https://github.com/YOUR_USERNAME/fastapi-helm-argocd/actions

# Watch ArgoCD sync in UI or CLI
kubectl get application -n argocd -w
```

## 📊 Monitoring & Observability

### ServiceMonitor for ArgoCD

The `manifests/argocd-monitor.yaml` creates a Prometheus ServiceMonitor to scrape ArgoCD metrics:

```yaml
- argocd_app_info: Application metadata
- argocd_app_sync_total: Sync counter
- argocd_app_health_status: Health status gauge
```

### Setting Up Slack Notifications (Optional)

1. Create a Slack webhook URL
2. Create a Kubernetes secret:
```bash
kubectl create secret generic argocd-notifications-secret \
  -n argocd \
  --from-literal=slack-token=YOUR_SLACK_WEBHOOK_URL
```

3. Apply notification config:
```bash
kubectl apply -f manifests/argo-notifications-setup.yaml
```

## 📁 Repository Structure

```
fastapi-helm-argocd/
├── .github/
│   └── workflows/
│       └── ci-build.yaml          # GitHub Actions CI/CD pipeline
├── app/
│   ├── app.py                     # FastAPI application code
│   ├── requirements.txt           # Python dependencies
│   └── test_main.py              # Pytest tests
├── helm/
│   └── fastapi-chart/            # Helm chart for FastAPI app
│       ├── Chart.yaml            # Chart metadata
│       ├── values.yaml           # Default values (image tag updated by CI)
│       └── templates/
│           ├── deployment.yaml   # Kubernetes Deployment
│           ├── service.yaml      # Kubernetes Service
│           └── ingress.yaml      # Kubernetes Ingress
├── manifests/
│   ├── application.yaml          # ArgoCD Application definition
│   ├── argocd-monitor.yaml       # Prometheus ServiceMonitor for ArgoCD
│   └── argo-notifications-setup.yaml  # ArgoCD Slack notifications
├── Dockerfile                    # Multi-stage Docker build
└── README.md                     # This file
```

## 🐛 Troubleshooting

### ArgoCD Application Not Syncing

```bash
# Check application status
kubectl describe application fastapi-app -n argocd

# Force sync
kubectl patch application fastapi-app -n argocd --type merge -p '{"operation":{"initiatedBy":{"username":"admin"},"sync":{"revision":"main"}}}'

# Check ArgoCD logs
kubectl logs -n argocd deployment/argocd-application-controller
```

### Pods Not Starting

```bash
# Check pod status
kubectl get pods -o wide

# Check pod logs
kubectl logs <pod-name>

# Describe pod for events
kubectl describe pod <pod-name>

# Check deployment
kubectl describe deployment fastapi-app
```

### Ingress Not Working

```bash
# Verify ingress is created
kubectl get ingress

# Check ingress controller
kubectl get pods -n ingress-nginx

# Restart Minikube tunnel (if using minikube tunnel)
minikube tunnel

# Verify /etc/hosts entry
cat /etc/hosts | grep fastapi.local
```

### Docker Build Fails in CI

1. Check GitHub Actions logs
2. Verify Docker Hub credentials in GitHub Secrets
3. Test build locally:
```bash
docker build -t test-image .
docker run -p 8000:8000 test-image
```

### Prometheus Not Scraping Metrics

```bash
# Check ServiceMonitor
kubectl get servicemonitor -n monitoring

# Check Prometheus targets
# Access Prometheus UI → Status → Targets

# Verify labels match
kubectl get svc -n argocd --show-labels
```

## 🎓 Learning Outcomes

This project demonstrates:

✅ **GitOps Principles**: Declarative infrastructure and application deployment  
✅ **CI/CD Automation**: Automated testing, building, and deployment  
✅ **Container Orchestration**: Kubernetes deployment patterns  
✅ **Infrastructure as Code**: Helm charts for templated deployments  
✅ **Monitoring & Observability**: Prometheus metrics and Grafana dashboards  
✅ **DevOps Best Practices**: Automated sync, self-healing, rollback capabilities  


## 👤 Author

**Omer Beit Halahmy**

- GitHub: [@omerbeithalahmy](https://github.com/omerbeithalahmy)
- Docker Hub: [omerbh7](https://hub.docker.com/u/omerbh7)


