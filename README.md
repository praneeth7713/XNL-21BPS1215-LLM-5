Intensive CI/CD Pipeline & Multi-Cloud Deployment for LLM-Powered SolutionsCI/CD Pipeline with FastAPI on Azure

1. Overview
This guide documents the entire setup, workflow, environment settings, and rollback simulations for deploying a FastAPI application on Azure with Azure Container Registry (ACR), Azure Container Instances (ACI), Prometheus & Grafana for monitoring, and GitHub Actions for CI/CD automation.

2. Prerequisites
> Azure Account with CLI installed (az command available)

> Docker installed and set up

> GitHub Repository with FastAPI project

> GitHub Actions set up for CI/CD

3. Setup and Configuration
3.1 Create Azure Resources

> Create a Resource Group:
az group create --name my-llm-app-rg --location eastus

> Create an Azure Container Registry (ACR):
az acr create --name praneeth7713 --resource-group my-llm-app-rg --sku Basic --admin-enabled true

> Login to ACR:
az acr login --name praneeth7713

5. CI/CD Workflow
4.1 Build & Push Docker Image to ACR

> Step 1: Build Docker Image
docker build -t praneeth7713.azurecr.io/llm-app:latest .

> Step 2: Push Image to ACR
docker push praneeth7713.azurecr.io/llm-app:latest

4.2 Deploy Application to Azure Container Instance (ACI)
az container create \
  --resource-group my-llm-app-rg \
  --name llm-app-container \
  --image praneeth7713.azurecr.io/llm-app:latest \
  --dns-name-label llm-app-demo \
  --ports 80 \
  --os-type Linux \
  --cpu 1 \
  --memory 1.5 \
  --registry-login-server praneeth7713.azurecr.io \
  --registry-username praneeth7713 \
  --registry-password "<your-password>"

4.3 Observability: Prometheus & Grafana Setup
Deploy Prometheus in ACI
az container create \
  --resource-group my-llm-app-rg \
  --name prometheus-container \
  --image praneeth7713.azurecr.io/prometheus \
  --ports 9090 \
  --os-type Linux \
  --cpu 1 \
  --memory 1.5 \
  --dns-name-label prometheus-monitoring \
  --restart-policy Always
Deploy Grafana in ACI
az container create \
  --resource-group my-llm-app-rg \
  --name grafana-container \
  --image grafana/grafana \
  --ports 3000 \
  --os-type Linux \
  --cpu 1 \
  --memory 1.5 \
  --dns-name-label grafana-dashboard \
  --restart-policy Always

4.4 Monitoring & Logs
Check Application Logs
az container logs --resource-group my-llm-app-rg --name llm-app-container
Check Container Status
az container show --resource-group my-llm-app-rg --name llm-app-container --query "{Status:instanceView.state,FQDN:ipAddress.fqdn}" -o table

