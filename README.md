# Project Documentation Index

This folder contains full technical documentation for the workspace in:

`/Users/tochratana/Project/devops`

It explains:

- what each repository does
- how each important code file works
- how services communicate with each other
- deployment flow from source code to Kubernetes

## Document map

1. `01-workspace-overview.md`
2. `02-end-to-end-integration.md`
3. `03-code-map-a8s-cli.md`
4. `04-code-map-spring-backend.md`
5. `05-code-map-next-frontend.md`
6. `06-code-map-fastapi-backend-reference.md`
7. `07-code-map-frontend-reference.md`
8. `08-code-map-platform-infra.md`
9. `09-code-map-platform-gitops.md`
10. `10-api-and-events-contract.md`
11. `11-security-and-hardening-notes.md`
12. `12-local-runbook.md`

## Workspace repositories covered

- `a8s-cli`
- `monolotic-app-backend`
- `monolotic-app-frontend`
- `platform-backend-fastapi`
- `platform-frontend-reference`
- `plateform-infra`
- `plateform-gitops`

## Important note about architecture

This workspace currently contains **two control-plane stacks**:

- Active Spring + Next.js stack (`monolotic-app-backend` + `monolotic-app-frontend` + `a8s-cli`)
- Reference FastAPI + React structure (`platform-backend-fastapi` + `platform-frontend-reference`)

Both are documented because they share the same platform direction (Jenkins + GitOps + ArgoCD).
