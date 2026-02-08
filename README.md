# FastAPI + Helm + ArgoCD - End-to-End DevOps Workflow 🚀

A complete end-to-end DevOps portfolio project demonstrating GitOps principles, CI/CD automation, Kubernetes orchestration, comprehensive monitoring, and Slack notifications using industry-standard tools.


## 🎯 Project Overview

This project implements a **GitOps-based deployment pipeline** for a FastAPI application using:
- **CI/CD**: GitHub Actions for automated testing, building, and deployment
- **Containerization**: Docker for packaging the application
- **Orchestration**: Kubernetes for container orchestration
- **GitOps**: ArgoCD for declarative continuous deployment
- **Monitoring**: Prometheus & Grafana for metrics and observability
- **Notifications**: Slack integration for deployment alerts
- **IaC**: Helm charts for templated Kubernetes deployments

<details>
<summary><b>🏗️ Architecture</b></summary>

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
                    │          │
                    └────┬─────┘
                         ▼
                  ┌─────────────┐
                  │    Slack    │
                  │Notifications│
                  └─────────────┘
```

</details>

<details>
<summary><b>🛠️ Technologies Used</b></summary>

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
- **Notifications**: [Slack](https://slack.com/) - Real-time deployment notifications

### Additional Components
- **Ingress**: NGINX Ingress Controller - Load balancing and routing
- **Service Mesh**: Prometheus Operator (kube-prometheus-stack)
- **ArgoCD Notifications**: Integrated Slack webhook for deployment events

</details>

<details>
<summary><b>✅ Prerequisites</b></summary>

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
- **Slack Workspace** (Optional): For deployment notifications

</details>

<details>
<summary><b>🚀 Complete Setup Guide</b></summary>

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

### Step 7: Set Up Monitoring & Notifications (Optional)

```bash
# Apply ServiceMonitor for ArgoCD metrics
kubectl apply -f manifests/argocd-monitor.yaml

# Apply ArgoCD Slack notifications (requires Slack webhook - see Slack Notifications section)
# kubectl apply -f manifests/argo-notifications-setup.yaml
```

</details>

<details>
<summary><b>🌐 Accessing Services</b></summary>

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

</details>

<details>
<summary><b>🔄 CI/CD Pipeline</b></summary>

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
   └────────┬─────────┘
            │
   ┌────────▼─────────┐
   │ ArgoCD Detects   │
   │  & Syncs App     │
   └────────┬─────────┘
            │
   ┌────────▼─────────┐
   │  Slack Notify    │
   │ (if configured)  │
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
   - Slack notification sent on successful deployment (if configured)

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

</details>

<details>
<summary><b>📊 Monitoring & Observability</b></summary>

### ServiceMonitor for ArgoCD

The `manifests/argocd-monitor.yaml` creates a Prometheus ServiceMonitor to scrape ArgoCD metrics:

```yaml
- argocd_app_info: Application metadata
- argocd_app_sync_total: Sync counter
- argocd_app_health_status: Health status gauge
```

**Apply the ServiceMonitor**:
```bash
kubectl apply -f manifests/argocd-monitor.yaml
```

**Verify metrics are being scraped**:
1. Access Prometheus UI: http://localhost:9090
2. Go to **Status** → **Targets**
3. Look for `serviceMonitor/monitoring/argocd-metrics/0`
4. Status should be **UP**

### Key Metrics to Monitor

**ArgoCD Application Health**:
```promql
argocd_app_health_status{name="fastapi-app"}
```

**Sync Status**:
```promql
argocd_app_sync_total{name="fastapi-app"}
```

**Application Info**:
```promql
argocd_app_info{name="fastapi-app"}
```

</details>

<details>
<summary><b>📢 Slack Notifications</b></summary>

### Overview

This project integrates **ArgoCD Notifications** with Slack to provide real-time alerts about deployment status. You'll receive Slack messages when:
- ✅ Application sync succeeds
- ❌ Application sync fails
- 🔄 Application health changes

### Configuration Files

The Slack integration uses:
- **ConfigMap**: `manifests/argo-notifications-setup.yaml` - Defines notification templates and triggers
- **Secret**: Created manually with your Slack webhook URL

### Setup Instructions

#### Step 1: Create a Slack Incoming Webhook

1. Go to your Slack workspace
2. Navigate to: **Settings** → **Manage apps** → **Custom Integrations** → **Incoming Webhooks**
3. Click **Add to Slack**
4. Choose a channel (e.g., `#deployments`, `#devops-alerts`)
5. Copy the **Webhook URL** (format: `https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXX`)

#### Step 2: Create Kubernetes Secret

```bash
# Create the secret with your Slack webhook URL
kubectl create secret generic argocd-notifications-secret \
  -n argocd \
  --from-literal=slack-token=YOUR_SLACK_WEBHOOK_URL

# Verify secret was created
kubectl get secret argocd-notifications-secret -n argocd
```

#### Step 3: Apply Notification Configuration

```bash
# Apply the notification ConfigMap
kubectl apply -f manifests/argo-notifications-setup.yaml

# Verify ConfigMap was created
kubectl get configmap argocd-notifications-cm -n argocd
```

#### Step 4: Subscribe Application to Notifications

```bash
# Add notification annotation to the ArgoCD Application
kubectl patch application fastapi-app -n argocd --type merge -p '{
  "metadata": {
    "annotations": {
      "notifications.argoproj.io/subscribe.on-sync-succeeded.slack": ""
    }
  }
}'

# Verify the annotation was added
kubectl get application fastapi-app -n argocd -o yaml | grep notifications
```

### Notification Templates

The `argo-notifications-setup.yaml` includes the following template:

**Sync Success Notification**:
```
✅ Application {{.app.metadata.name}} is now running!
Sync Status: {{.app.status.sync.status}}
Health Status: {{.app.status.health.status}}
```

### Testing Notifications

Trigger a sync to test notifications:

```bash
# Make a code change
echo "# Test notification" >> app/app.py

# Commit and push
git add app/app.py
git commit -m "test: trigger Slack notification"
git push origin main

# Watch for the sync
kubectl get application fastapi-app -n argocd -w

# Check your Slack channel for the notification
```

### Advanced: Custom Notification Templates

You can customize the notification templates in `manifests/argo-notifications-setup.yaml`:

```yaml
data:
  template.app-degraded: |
    message: |
      ⚠️ Application {{.app.metadata.name}} is degraded!
      Health: {{.app.status.health.status}}
      Sync: {{.app.status.sync.status}}
  
  trigger.on-degraded: |
    - description: Application has degraded
      send: [app-degraded]
      when: app.status.health.status == 'Degraded'
```

**Apply the changes**:
```bash
kubectl apply -f manifests/argo-notifications-setup.yaml

# Subscribe to the new trigger
kubectl patch application fastapi-app -n argocd --type merge -p '{
  "metadata": {
    "annotations": {
      "notifications.argoproj.io/subscribe.on-degraded.slack": ""
    }
  }
}'
```

### Available Notification Triggers

- `on-sync-succeeded`: Sync operation succeeded
- `on-sync-failed`: Sync operation failed
- `on-sync-running`: Sync operation is running
- `on-health-degraded`: Application health is degraded
- `on-deployed`: New version deployed

### Troubleshooting Notifications

```bash
# Check notification controller logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-notifications-controller

# Verify secret exists and has correct data
kubectl get secret argocd-notifications-secret -n argocd -o yaml

# Check ConfigMap
kubectl get configmap argocd-notifications-cm -n argocd -o yaml

# Verify application annotations
kubectl get application fastapi-app -n argocd -o jsonpath='{.metadata.annotations}'
```

</details>

<details>
<summary><b>📁 Repository Structure</b></summary>

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
│   └── argo-notifications-setup.yaml  # ArgoCD Slack notifications ConfigMap
├── Dockerfile                    # Multi-stage Docker build
└── README.md                     # This file
```

</details>

<details>
<summary><b>🐛 Troubleshooting</b></summary>

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

### Slack Notifications Not Working

```bash
# Verify secret exists
kubectl get secret argocd-notifications-secret -n argocd

# Check notification controller logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-notifications-controller

# Verify application has subscription annotation
kubectl get application fastapi-app -n argocd -o yaml | grep notifications

# Test webhook manually
curl -X POST -H 'Content-type: application/json' \
  --data '{"text":"Test notification"}' \
  YOUR_SLACK_WEBHOOK_URL
```

</details>

## 🎓 Learning Outcomes

This project demonstrates:

✅ **GitOps Principles**: Declarative infrastructure and application deployment  
✅ **CI/CD Automation**: Automated testing, building, and deployment  
✅ **Container Orchestration**: Kubernetes deployment patterns  
✅ **Infrastructure as Code**: Helm charts for templated deployments  
✅ **Monitoring & Observability**: Prometheus metrics and Grafana dashboards  
✅ **Notification Integration**: Real-time Slack alerts for deployment events  
✅ **DevOps Best Practices**: Automated sync, self-healing, rollback capabilities  

## 👤 Author

**Omer Beit Halahmy**

- GitHub: [@omerbeithalahmy](https://github.com/omerbeithalahmy)
- Docker Hub: [omerbh7](https://hub.docker.com/u/omerbh7)


