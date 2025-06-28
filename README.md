# 🚀 Azure DevOps CI/CD Pipeline for Dockerized App Deployment

This repository contains an Azure DevOps YAML pipeline that automates the build and deployment of a Dockerized application. The pipeline is configured to trigger on code changes to specific branches and deploy the container to Azure App Service.

---

## 📌 Features

* ✅ Automatically triggers on changes to selected branches (`dev`, `main`, `uat`, `prod`)
* 🐳 Builds a Docker image using the project's `Dockerfile`
* 📦 Pushes the image to Azure Container Registry (ACR)
* 🚀 Deploys the containerized app to Azure App Service

---

## 🛠️ Pipeline YAML Overview

```yaml
# ============================
# Azure DevOps Pipeline
# This pipeline runs automatically on changes to the specified branches.
# It builds the Docker image, pushes it to ACR, and deploys it to App Service.
# ============================

# Trigger pipeline only on selected branches
trigger:
  branches:
    include:
      - main
      - dev
      - prod
      - uat

# --------------------------
# Pipeline Variables
# --------------------------
variables:
  dockerRegistryServiceConnection: ''      # Azure DevOps service connection to ACR
  azureSubscription: ''                    # Azure subscription name or ID
  containerRegistry: ''                    # ACR login server (e.g., myregistry.azurecr.io)
  imageRepository: ''                      # Docker image repository in ACR
  dockerfilePath: 'Dockerfile'             # Path to Dockerfile
  tag: $(Build.SourceVersion)              # Git commit hash used as image tag
  appName: ''                              # Name of Azure App Service
  vmImageName: 'ubuntu-latest'             # Agent VM image for the pipeline

# --------------------------
# STAGE 1: Build Docker image and push to ACR
# --------------------------
stages:
  - stage: Build
    displayName: Build and Push to ACR
    jobs:
      - job: Docker
        pool:
          vmImage: $(vmImageName)
        steps:
          - task: Docker@2
            displayName: Build and Push image
            inputs:
              command: buildAndPush
              containerRegistry: $(dockerRegistryServiceConnection)
              repository: $(imageRepository)
              dockerfile: $(dockerfilePath)
              tags: |
                $(tag)

# --------------------------
# STAGE 2: Deploy Docker image to Azure App Service
# --------------------------
  - stage: Deploy
    displayName: Deploy to Azure App Service
    dependsOn: Build
    jobs:
      - job: DeployWebApp
        pool:
          vmImage: $(vmImageName)
        steps:
          - task: AzureWebAppContainer@1
            displayName: Deploy Docker Image to App Service
            inputs:
              azureSubscription: $(azureSubscription)
              appName: $(appName)
              containers: '$(containerRegistry)/$(imageRepository):$(tag)'
```

---

## 🧪 How to Use

1. **Create service connections** in Azure DevOps:

   * Azure Resource Manager for App Service (`azureSubscription`)
   * Docker Registry for ACR (`dockerRegistryServiceConnection`)

2. **Update variable values** in the YAML file with your actual Azure details.

3. **Commit to one of the trigger branches** to start the CI/CD workflow.

---

## 📄 Requirements

* Azure DevOps Project
* Azure Container Registry (ACR)
* Azure App Service configured for container deployment
* Dockerfile present in the repository

---

## 🔐 Security Note

Store all secrets (such as credentials, tokens) using Azure DevOps secure variable groups or link them from Azure Key Vault.

---

## 🧩 Supported Branches

* `dev`
* `main`
* `uat`
* `prod`

Modify the `trigger` section to suit your branching strategy.
