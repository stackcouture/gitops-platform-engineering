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
## Repository Structure

This is a **mono-meta repository** that ties together three Git submodules:

```
gitops-platform-engineering/
├── .gitmodules                      # Submodule definitions
├── README.md
├── gitops-microservices-platform/   # ArgoCD + Helm/Kustomize configs (submodule)
├── platform-infra/                  # Terraform infrastructure code (submodule)
└── voting-app/                      # Application source code (submodule)
```

---
## Submodules

### `platform-infra`
Terraform code responsible for provisioning the underlying Kubernetes cluster and supporting cloud infrastructure (networking, IAM, Artifact Registry, GKE).

- Provisions a Kubernetes cluster (GKE)


### `voting-app`
The application source code — a polyglot microservices app based on Docker's example voting app. Consists of five services:

| Service    | Language   | Description                            |
|------------|------------|----------------------------------------|
| `vote`     | Python     | Web frontend to cast votes             |
| `result`   | Node.js    | Web frontend to display vote results   |
| `worker`   | .NET       | Processes votes from Redis to Postgres |
| `redis`    | Redis      | In-memory queue for incoming votes     |
| `db`       | PostgreSQL | Persistent storage for vote results    |

### `gitops-microservices-platform`
ArgoCD `ApplicationSets` manifests and Helm/Kustomize configurations for deploying the voting app services to Kubernetes. This is the GitOps "desired state" repository that ArgoCD watches.

- ArgoCD `ApplicationSet` and `AppProjects` CRDs per microservice
- Kustomize overlays for environment-specific patches

---
## Tech Stack

| Tool          | Role                                              |
|---------------|---------------------------------------------------|
| **Kubernetes**| Container orchestration platform                  |
| **ArgoCD**    | GitOps continuous delivery controller             |
| **Terraform** | Infrastructure as Code for cluster provisioning   |
| **Kustomize** | Environment-specific configuration management     |
| **Docker**    | Container image building                          |
| **GitHub**    | Source of truth for all config and code           |

---
## Prerequisites

Before getting started, ensure you have the following installed and configured:

- [Terraform](https://developer.hashicorp.com/terraform/install) `>= 1.3`
- [kubectl](https://kubernetes.io/docs/tasks/tools/) configured for your target cluster
- [Helm](https://helm.sh/docs/intro/install/) `>= 3.x`
- [ArgoCD CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/) (optional, for ArgoCD management)
- [Git](https://git-scm.com/) with submodule support
- Cloud provider CLI (`gcloud`) authenticated

---


## GitOps Workflow

Any change to application config or infrastructure follows this flow:

```
Developer → Git commit/PR → Merge to main
                                  │
                                  ▼
                           ArgoCD detects drift
                                  │
                                  ▼
                     ArgoCD syncs cluster to match Git
                                  │
                                  ▼
                        Deployment updated ✓
```
## Contributing

Contributions, suggestions, and improvements are welcome! To contribute:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature-name`
3. Commit your changes with a clear message: `git commit -m "feat: describe your change"`
4. Push to your fork: `git push origin feature/your-feature-name`
5. Open a Pull Request against `main`

Please keep PRs focused — one feature or fix per PR. For significant changes, open an issue first to discuss the approach.

---

## Acknowledgements

This project is based on Docker's Example Voting App:
[https://github.com/dockersamples/example-voting-app](https://github.com/dockersamples/example-voting-app)

Original project licensed under Apache 2.0.
