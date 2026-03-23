# 06 - Code Map: `platform-backend-fastapi` (Reference)

## What this module does

This is a reference control-plane backend with similar responsibilities to the Spring service, but designed in FastAPI + SQLAlchemy.

Responsibilities:

- project CRUD + repository link
- deployment trigger + history
- rollback by GitOps image update
- webhook ingestion (GitHub/GitLab) with signature validation
- websocket deployment log streaming

## File-by-file responsibilities

### App bootstrap and routing

| File | Responsibility |
|---|---|
| `app/main.py` | FastAPI startup, router include, DB metadata create-all, health endpoint |
| `app/api/router.py` | Registers versioned route groups |
| `app/api/routes/projects.py` | Project create/list/repository-connect/deploy/deployment-list routes |
| `app/api/routes/deployments.py` | Rollback and deployment-log listing |
| `app/api/routes/webhooks.py` | GitHub/GitLab webhook processing and deploy trigger |
| `app/api/routes/logs.py` | WebSocket log stream endpoint |

### Core configuration and security

| File | Responsibility |
|---|---|
| `app/core/config.py` | Environment-driven settings model |
| `app/core/security.py` | Current user extraction from `X-User-Id` header (placeholder auth) |

### Database layer

| File | Responsibility |
|---|---|
| `app/db/base.py` | Declarative SQLAlchemy base |
| `app/db/session.py` | Engine/session factory and request DB dependency |
| `schema.sql` | SQL bootstrap schema for users/projects/deployments/logs/webhooks |

### Models

| File | Responsibility |
|---|---|
| `app/models/user.py` | User table model |
| `app/models/project.py` | Project metadata and repo linkage |
| `app/models/deployment.py` | Deployment execution history |
| `app/models/deployment_log.py` | Log lines per deployment |
| `app/models/webhook_event.py` | Raw webhook event audit trail |
| `app/models/__init__.py` | Model exports |

### Schemas

| File | Responsibility |
|---|---|
| `app/schemas/platform.py` | Pydantic request/response contracts |
| `app/schemas/__init__.py` | Schema exports |

### Services

| File | Responsibility |
|---|---|
| `app/services/jenkins.py` | Jenkins trigger integration (with optional crumb) |
| `app/services/gitops.py` | Calls external GitOps update script |
| `app/services/webhook.py` | Signature verification helpers |
| `app/services/__init__.py` | Service exports |

## Core call chains

### Manual deploy

`POST /api/v1/projects/{id}/deploy` -> create `Deployment(status=BUILDING)` -> trigger Jenkins -> store queue URL -> return accepted response.

### Webhook deploy

Webhook route -> project lookup by repo -> signature validation -> branch/event filter -> trigger Jenkins -> create deployment row -> mark webhook processed.

### Rollback

`POST /api/v1/deployments/{id}/rollback` -> validate target deployment -> create rollback deployment row -> run GitOps update script -> mark deployed.

## Integration notes

- Current auth is placeholder (`X-User-Id` header), not JWT/OIDC.
- Startup uses `Base.metadata.create_all`; production usually migrates with Alembic.
- GitOps updates are delegated to shell script configured by env var.
