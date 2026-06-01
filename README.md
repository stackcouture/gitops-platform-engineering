# GitOps Platform Engineering

A personal portfolio project demonstrating a production-grade **GitOps platform** using Kubernetes, ArgoCD, Terraform, and Helm/Kustomize — built around a microservices voting application derived from [Docker's Example Voting App](https://github.com/dockersamples/example-voting-app).

> This project showcases end-to-end platform engineering practices: infrastructure provisioning, GitOps-driven continuous delivery, and multi-service application deployment on Kubernetes.

---

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Repository Structure](#repository-structure)
- [Submodules](#submodules)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [GitOps Workflow](#gitops-workflow)
- [Contributing](#contributing)
- [Acknowledgements](#acknowledgements)

---

## Architecture Overview

This platform follows a **pull-based GitOps model** where ArgoCD continuously reconciles the desired state (defined in Git) with the actual state of the Kubernetes cluster.

```
┌──────────────────────────────────────────────────────────────┐
│                        GitHub Repos                          │
│                                                              │
│   ┌─────────────────┐   ┌──────────────┐   ┌────────────┐    │ 
│   │  platform-infra │   │  voting-app  │   │gitops-micro│    │
│   │  (Terraform)    │   │ (App source) │   │-platform   │    │
│   └────────┬────────┘   └──────┬───────┘   └─────┬──────┘    │
└────────────┼───────────────────┼─────────────────┼───────────┘
             │                   │                 │
             ▼                   ▼                 ▼
     ┌───────────────┐   ┌──────────────┐   ┌──────────────┐
     │  Terraform    │   │  CI Pipeline │   │    ArgoCD    │
     │  provisions   │   │  builds &    │   │  watches &   │
     │  k8s cluster  │   │  pushes image│   │  syncs state │
     └───────┬───────┘   └──────┬───────┘   └──────┬───────┘
             │                   │                  │
             └───────────────────┴──────────────────┘
                                 │
                    ┌────────────▼─────────────┐
                    │    Kubernetes Cluster    │
                    │                          │
                    │  ┌────────┐ ┌─────────┐  │
                    │  │  vote  │ │ result  │  │
                    │  └────────┘ └─────────┘  │
                    │  ┌────────┐ ┌─────────┐  │
                    │  │ worker │ │  redis  │  │
                    │  └────────┘ └─────────┘  │
                    │        ┌──────┐          │
                    │        │  db  │          │
                    │        └──────┘          │
                    └──────────────────────────┘
```
**Key design principles:**
- Git is the single source of truth for both infrastructure and application state
- No manual `kubectl apply` — all changes flow through Git commits and ArgoCD sync
- Infrastructure is fully provisioned via Terraform before any workloads are deployed
- Kustomize manages application packaging and environment-specific configuration

---

## Acknowledgements

This project is based on Docker's Example Voting App:
https://github.com/dockersamples/example-voting-app

Original project licensed under Apache 2.0.