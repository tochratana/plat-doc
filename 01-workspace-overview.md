# 01 - Workspace Overview

## Purpose

This workspace builds a multi-tenant application deployment platform (similar to Vercel/Render flow):

- connect repo
- trigger deploy
- build image
- push to registry
- update GitOps
- ArgoCD sync to Kubernetes
- stream build logs

## Repository roles

| Repository | Main role | Tech |
|---|---|---|
| `a8s-cli` | Terminal client for login/deploy/logs/rollback | Go |
| `monolotic-app-backend` | Auth + repo + deploy orchestration + Jenkins log stream | Spring Boot |
| `monolotic-app-frontend` | Dashboard UI + GitHub OAuth + deploy/log stream UX | Next.js |
| `platform-backend-fastapi` | Reference control-plane implementation | FastAPI |
| `platform-frontend-reference` | Reference UI component architecture | React/Tailwind |
| `plateform-infra` | Jenkins pipeline + scripts + Dockerfile templates + Helm chart | Jenkins/Bash/Helm |
| `plateform-gitops` | GitOps source for ArgoCD app discovery and tenant deployments | Kubernetes/Helm/ArgoCD |

## Main runtime flow

1. User logs in (CLI or Web).
2. Backend stores Git provider token securely and returns backend auth token.
3. User triggers deployment.
4. Backend creates project/deployment record and triggers Jenkins pipeline.
5. Jenkins detects framework, builds container, pushes image.
6. Jenkins updates `plateform-gitops` values (`image.tag`).
7. ArgoCD ApplicationSet detects Git change and syncs to Kubernetes.
8. UI/CLI stream and show logs/status.

## Data boundaries

- Stateful records: PostgreSQL (`projects`, `deployments`, logs, tokens/events)
- Build execution: Jenkins
- Desired state: GitOps repository
- Cluster reconciliation: ArgoCD
- Runtime serving: Kubernetes + Ingress
