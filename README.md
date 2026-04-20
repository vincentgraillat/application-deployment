# Application Deployment Repo

GitOps repository that drives the Kubernetes deployment of the R-FPOP changepoint detection app on SSPCloud. This repo contains no application code, only the declarative state of what should be running in the cluster. ArgoCD watches it and reconciles the cluster automatically.

## Repository Layout

```
application-deployment/
├── application.yaml          
└── deployment/
    ├── kustomization.yaml   
    ├── deployment.yaml       
    ├── service.yaml         
    └── ingress.yaml       
```

## How It Works

ArgoCD runs inside the SSPCloud cluster and continuously reconciles the cluster state against this repository.

1. `application.yaml` registers the app with ArgoCD, pointing it at the `deployment/` path on the `main` branch of this repo.
2. Whenever a commit lands on `main`, ArgoCD detects the drift and applies the updated manifests to namespace `user-vgraillat`.
3. `syncPolicy.automated` is enabled with `selfHeal: true` (reverts manual `kubectl` edits) and `prune: true` (removes resources deleted from this repo).

The Docker image is not built here. The application repo ([R-FPOP-Change-Points-Detection](https://github.com/audricms/R-FPOP-Change-Points-Detection)) handles image builds via GitHub Actions. Pushing a `v*.*.*` git tag triggers the workflow and pushes a versioned image to Docker Hub. Updating the image tag in `deployment/deployment.yaml` here is what closes the loop with ArgoCD.

## Full GitOps Flow

```
Create a git tag vX.Y.Z on MLOps repo
        │
        ▼
GitHub Actions builds & pushes audricms/r-fpop-change-points-detection:vX.Y.Z to Docker Hub
        │
Update image tag in this repo deployment/deployment.yaml
        │
        ▼
ArgoCD detects manifest drift → auto-syncs cluster → new image is live
```

## Streamlit app 
The application is reachable at: `https://rfpop-vgraillat.user.lab.sspcloud.fr`
