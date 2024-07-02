# MERN App Deployment with ArgoCD and Kustomize.

## Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Repository Structure](#repository-structure)
4. [Kustomize Configuration](#kustomize-configuration)
5. [ArgoCD Setup](#argocd-setup)
6. [Deployment](#deployment)
7. [Managing Environments](#managing-environments)
8. [Accessing the Application](#accessing-the-application)
9. [Troubleshooting](#troubleshooting)
10. [Conclusion](#conclusion)

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

