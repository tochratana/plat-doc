# 12 - Local Runbook

## Goal

Run the active stack locally:

- backend: `monolotic-app-backend` on `:8080`
- frontend: `monolotic-app-frontend` on `:3000`
- optional: CLI `a8s-cli`

## Prerequisites

- Java 21
- Node.js 20+
- Go 1.22+
- Docker (for local PostgreSQL)

## 1) Start PostgreSQL

```bash
docker run --name monolotic-app-db \
  -e POSTGRES_DB=monolotic-app-db \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=1234 \
  -p 5432:5432 -d postgres:16
```

## 2) Run backend

```bash
cd /Users/tochratana/Project/devops/monolotic-app-backend
./gradlew bootRun
```

Expected base URL: `http://localhost:8080`

## 3) Run frontend

```bash
cd /Users/tochratana/Project/devops/monolotic-app-frontend
npm install
npm run dev
```

Expected UI URL: `http://localhost:3000`

## 4) Use CLI (optional)

```bash
cd /Users/tochratana/Project/devops/a8s-cli
go build -o a8s .
./a8s login
./a8s repos
./a8s deploy
```

## 5) Smoke tests

- Login works and backend token is stored.
- Repositories load in UI.
- Deploy action returns queue item id.
- Live logs stream appears in UI.
- Project status appears in deployments table.

## Troubleshooting quick list

- `401 Unauthorized`: backend token missing/expired.
- WebSocket error: check backend URL/token and CORS/origin config.
- No deploy queue id: backend/Jenkins integration issue.
- No Argo sync: check GitOps push, ArgoCD app health, namespace.
