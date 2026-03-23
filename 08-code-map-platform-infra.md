# 08 - Code Map: `plateform-infra`

## What this module does

This repository provides the reusable delivery engine:

- generic Jenkins pipeline
- framework detection
- Dockerfile generation from templates
- image build/push
- GitOps repo update
- Helm chart templates for runtime objects

## File-by-file responsibilities

### Pipeline and scripts

| File | Responsibility |
|---|---|
| `jenkins/Jenkinsfile` | End-to-end CI/CD pipeline from source checkout to GitOps commit |
| `jenkins/scripts/detect-framework.sh` | Detects framework by project files/dependencies |
| `jenkins/scripts/generate-dockerfile.sh` | Selects Dockerfile template if app has no Dockerfile |
| `jenkins/scripts/build-app.sh` | Optional framework-specific build helper |
| `jenkins/scripts/update-gitops.sh` | Clones GitOps repo, updates values, commits/pushes with retry |

### Dockerfile templates

| File | Responsibility |
|---|---|
| `docker/dockerfiles/Dockerfile.nextjs` | Next.js multi-stage build/runtime |
| `docker/dockerfiles/Dockerfile.react` | React build -> Nginx static serve |
| `docker/dockerfiles/Dockerfile.nodejs` | Generic Node runtime |
| `docker/dockerfiles/Dockerfile.springboot` | Maven Spring Boot build + JRE runtime |
| `docker/dockerfiles/Dockerfile.gradle` | Gradle Java build + JRE runtime |
| `docker/dockerfiles/Dockerfile.fastapi` | FastAPI app runtime with uvicorn |
| `docker/dockerfiles/Dockerfile.flask` | Flask app runtime with gunicorn |
| `docker/dockerfiles/Dockerfile.python` | Generic Python web runtime |
| `docker/dockerfiles/Dockerfile.laravel` | Laravel runtime on PHP Apache |
| `docker/dockerfiles/Dockerfile.php` | Generic PHP Apache runtime |
| `docker/dockerfiles/Dockerfile.static` | Static content on Nginx |
| `docker/dockerfiles/Dockerfile.go` | Go multi-stage binary build |

### Helm app template

| File | Responsibility |
|---|---|
| `helm/app-template/Chart.yaml` | Helm chart metadata |
| `helm/app-template/values.yaml` | Tenant app values schema and defaults |
| `helm/app-template/templates/deployment.yaml` | Kubernetes Deployment template |
| `helm/app-template/templates/service.yaml` | Kubernetes Service template |
| `helm/app-template/templates/ingress.yaml` | Kubernetes Ingress template |
| `helm/app-template/templates/hpa.yaml` | HorizontalPodAutoscaler template |
| `helm/app-template/templates/_helpers.tpl` | Shared Helm helper labels/naming |

### Platform manifests/docs

| File | Responsibility |
|---|---|
| `kubernetes/namespace.yaml` | Base namespace bootstrap |
| `kubernetes/registery-secret.yaml` | Registry secret guidance |
| `argocd/install.yaml` | ArgoCD installation quick notes |
| `readme.md` | Infra repository usage and inputs |
| `docs/*.md` | Architecture, flow, API and structure documents |

## Pipeline stage summary

1. Validate input parameters.
2. Checkout infra repo and user app repo.
3. Normalize identifiers and compute immutable image tag.
4. Detect framework.
5. Generate Dockerfile if app does not provide one.
6. Build and push image.
7. Update GitOps values under tenant path.

## Integration contracts

- image tag format is immutable and build-specific.
- GitOps updater expects managed marker in values file.
- chart source path is passed into updater for initial project scaffold.
