# MERN App Deployment with ArgoCD and Kustomize.

## Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Repository Structure](#repository-structure)
4. [Kustomize Configuration](#kustomize-configuration)
5. [ArgoCD Setup](#argocd-setup)
6. [Deployment](#deployment)
7. [Managing Environments](#managing-environments)
8. [Troubleshooting](#troubleshooting)


## Introduction
This document provides a step-by-step guide for deploying a MERN (MongoDB, Express, React, Node.js) application to a Kubernetes cluster using ArgoCD and Kustomize following the `GitOps` principles. This setup supports multiple environments such as `staging` and `production`.

## Prerequisites
- A `Kubernetes cluster`.
- `kubectl` installed locally.
- `ArgoCD` installed and configured in the cluster.
- `Kustomize` installed locally.
- `Docker` installed for building images.
- A `Docker Hub` account (or other container registry) for storing Docker images.

## Repository Structure
Here is the structure of the repository used for deploying the MERN app:
![alt text](pics/arch.png)



## Kustomize Configuration

### Base
The base directory holds configuration which is common to all environments. It is not expected to change often. To better organize, subdirectories for `backend, frontend, and database,` are created each with their respective manifests.

#### Backend
- **deployment.yaml**: Deployment configuration for the backend service.
- **service.yaml**: Service configuration for the backend service.
- **kustomization.yaml**: Kustomize configuration file for the backend resources.

#### Frontend
- **deployment.yaml**: Deployment configuration for the frontend service.
- **service.yaml**: Service configuration for the frontend service.
- **kustomization.yaml**: Kustomize configuration file for the frontend resources.

#### Database
- **pv.yaml**: Persistent Volume configuration for MongoDB.
- **pvc.yaml**: Persistent Volume Claim configuration for MongoDB.
- **secret.yaml**: Secret to prevent hardcoding of credentials for MongoDB.
- **deployment.yaml**: Deployment configuration for the MongoDB service.
- **service.yaml**: Service to enable internal communication.
- **kustomization.yaml**: Kustomize configuration file for the database resources.

### Environments
The `envs` directory contains environment-specific overlays. We can override the base configurations by patching the custom configurations like replica, image,etc.

#### staging
- **kustomization.yaml**: References the base and includes environment-specific patches.
- **backend-patch.yaml**: Specifies the backend Docker image and replica to be created for the staging environment.
- **frontend-patch.yaml**: Specifies the frontend Docker image and replica to be created for the staging environment.
- **database-patch.yaml**: Specifies the database Docker image and replica to be created for the staging environment.

#### prod
- **kustomization.yaml**: References the base and includes environment-specific patches.
- **backend-patch.yaml**: Specifies the backend Docker image and replica to be created for the staging environment.
- **frontend-patch.yaml**: Specifies the frontend Docker image and replica to be created for the staging environment.
- **database-patch.yaml**: Specifies the database Docker image and replica to be created for the staging environment.

## ArgoCD Setup

### ArgoCD Application Manifest
Since I'm deploying the applicatin in `staging` and `prod` envs, i also created the respective application definitions located at `argocd-apps/` directort with name `staging-app.yaml` and `prod-app.yaml`.

**staging-app.yaml:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: mern-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/asa-96/mern-app-gitops.git'
    targetRevision: HEAD
    path: envs/staging
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: staging
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
  syncOptions:    
    - CreateNamespace=true
```

**prod-app.yaml:**

```yaml
# prod-app.yaml`
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: mern-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/asa-96/mern-app-gitops.git'
    targetRevision: HEAD
    path: envs/prod
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: prod
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
  syncOptions:    
    - CreateNamespace=true
```

Apply the applocation manifests.
```
kubectl apply -f argocd/staging-app.yaml
kubectl apply -f argocd/prod-app.yaml
```

Verify the application.
```
kubectl get applications -n staging
kubectl get applications -n prod
```

## Deployment
Once the ArgoCD application definition is applied, ArgoCD will automatically monitor the specified `Git repository` for changes. Any updates to the Kustomize configurations or Docker image versions in the repository will be automatically synced to the Kubernetes cluster.

You can also apply the manifests in specific env by running the following command
```
kubectl apply -k kustomize/envs/staging
kubectl apply -k kustomize/envs/prod
```

## Managing Environments
To manage different environments, modify the `kustomization.yaml` and other respective configuration files under the respective environment directories.


## Troubleshooting
If you encounter issues, check the following:

- ArgoCD application status from UI or using ArgoCD CLI.
- Kubernetes pod logs.
- Kustomize build output.