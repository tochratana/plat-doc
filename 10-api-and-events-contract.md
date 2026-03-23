# 10 - API and Events Contract

## Active Spring stack API contract

### Auth

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/v1/auth/github` | Exchange GitHub token for backend JWT and store encrypted token |

### Repository and project operations

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/repos` | List repositories for authenticated user |
| POST | `/api/v1/projects` | Create project and trigger Jenkins deployment |
| GET | `/api/v1/projects` | List user projects |

### Deployment operations

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/deployments/{deploymentId}/logs` | Get deployment logs |
| POST | `/api/v1/deployments/{deploymentId}/rollback` | Roll back deployment |

### Jenkins logs

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/jenkins/logs/stream?job=&build=` | SSE stream from Jenkins build logs |
| WS | `/ws/jenkins/logs?job=&build=&token=` | WebSocket direct build logs |
| WS | `/ws/jenkins/logs?job=&queueItem=&token=` | WebSocket queue-to-build logs |

## Reference FastAPI stack API contract

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/v1/projects` | Create project |
| GET | `/api/v1/projects` | List user projects |
| PUT | `/api/v1/projects/{projectId}/repository` | Update repo linkage/webhook secret |
| POST | `/api/v1/projects/{projectId}/deploy` | Trigger deployment |
| GET | `/api/v1/projects/{projectId}/deployments` | Deployment history |
| POST | `/api/v1/deployments/{deploymentId}/rollback` | Rollback deploy |
| GET | `/api/v1/deployments/{deploymentId}/logs` | Deployment logs |
| POST | `/api/v1/webhooks/github` | GitHub webhook |
| POST | `/api/v1/webhooks/gitlab` | GitLab webhook |
| WS | `/ws/projects/{projectId}/deployments/{deploymentId}/logs` | Deployment log streaming |

## WebSocket message contract (Spring log stream)

### Server -> client event types

- `queued`: still waiting in Jenkins queue
- `open`: build number resolved, stream started
- `log`: chunk of Jenkins log output
- `heartbeat`: keepalive timestamp
- `done`: stream completed
- `error`: terminal failure

### Message examples

```json
{"type":"queued","queueItemId":42,"message":"Waiting for executor"}
```

```json
{"type":"open","job":"deploy-pipeline","build":128}
```

```json
{"type":"log","chunk":"Step 1/10...\n","offset":512,"build":128}
```

```json
{"type":"done","message":"Log stream completed","build":128}
```

## Deploy response contract used by frontend

Expected fields from deploy endpoint:

- `project` (object)
- `jenkinsJobName` (string)
- `queueUrl` (string)
- `queueItemId` (number)

The frontend depends on `queueItemId` to start websocket streaming immediately after deploy click.
