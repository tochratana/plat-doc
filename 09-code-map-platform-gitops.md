# 09 - Code Map: `plateform-gitops`

## What this module does

This repository is the desired-state source consumed by ArgoCD.

Responsibilities:

- bootstrap ArgoCD app-of-apps
- discover tenant apps using ApplicationSet
- store per-tenant/per-project Helm releases
- receive image-tag updates from Jenkins

## File-by-file responsibilities

### Bootstrap

| File | Responsibility |
|---|---|
| `bootstrap/namespace.yaml` | Creates `argocd` namespace |
| `bootstrap/argocd-app-of-apps.yaml` | Root ArgoCD Application that syncs bootstrap dir |
| `bootstrap/applicationset-user-projects.yaml` | Generates ArgoCD apps from `apps/*/*` directories |
| `bootstrap/apps-namespace.yaml` | Deprecated note (shared namespace not used) |
| `bootstrap/registery-secret.yaml` | Notes for per-tenant registry secret creation |
| `bootstrap/user-app-template.yaml` | Deprecated note (ApplicationSet now used) |

### Tenant app tree

| Path | Responsibility |
|---|---|
| `apps/<userId>/namespace.yaml` | Per-tenant namespace manifest |
| `apps/<userId>/<project>/Chart.yaml` | Chart metadata |
| `apps/<userId>/<project>/values.yaml` | Deploy contract values (`image.repository`, `image.tag`, host, ports) |
| `apps/<userId>/<project>/templates/*` | Deployment/Service/Ingress/HPA templates |

### Additional sample path

| Path | Responsibility |
|---|---|
| `apps/user-demo/demo-app/*` | Example tenant deployment structure |
| `app/my-next-app/*` | Legacy/incomplete sample manifests |

## Reconciliation flow

1. Jenkins updates tenant `values.yaml` image tag.
2. Git commit is pushed to `plateform-gitops`.
3. ArgoCD ApplicationSet detects updated directory.
4. ArgoCD syncs Helm chart to tenant namespace.
5. Kubernetes rolls out new ReplicaSet.

## Naming and tenancy contract

- App path: `apps/<safe-user-id>/<safe-project-name>`
- Namespace: `user-<safe-user-id>`
- Hostname: `<safe-project-name>.<platform-domain>`
