# 04 - Code Map: `monolotic-app-backend` (Spring)

## What this module does

This backend is the main control plane for the active stack.

Responsibilities:

- exchanges GitHub token for backend JWT
- stores encrypted GitHub token
- lists user repositories through GitHub API
- creates projects and triggers Jenkins deployment
- exposes Jenkins log stream (SSE + WebSocket)
- secures API with JWT resource server

## Backend layers

- Controller layer: request validation and HTTP response mapping
- Service layer: integration with GitHub/Jenkins/ArgoCD and crypto
- Repository layer: JPA persistence
- Config layer: security and websocket wiring
- Model/DTO layer: data and response contracts

## File-by-file responsibilities

### Application and config

| File | Responsibility |
|---|---|
| `src/main/java/.../MonoloticAppBackendApplication.java` | Spring Boot startup |
| `src/main/resources/application.yaml` | Base app profile activation (`dev`) |
| `src/main/resources/application-dev.yml` | Local env config (DB, JWT secret, Jenkins/Argo/GitOps settings) |
| `src/main/java/.../config/SecurityConfig.java` | JWT auth, CORS, route access rules |
| `src/main/java/.../config/WebSocketConfig.java` | Registers Jenkins log websocket endpoint |
| `src/main/java/.../config/JwtWebSocketAuthInterceptor.java` | Validates websocket token from query parameter |

### Controllers

| File | Responsibility |
|---|---|
| `controller/AuthController.java` | `POST /api/v1/auth/github`, token verify/store/exchange |
| `controller/RepoController.java` | `GET /api/v1/repos`, returns repo list per authenticated user |
| `controller/ProjectController.java` | `POST /api/v1/projects`, `GET /api/v1/projects` |
| `controller/JenkinsController.java` | `GET /api/v1/jenkins/logs/stream` SSE endpoint |

### Services

| File | Responsibility |
|---|---|
| `service/JenkinsService.java` | Jenkins service contract |
| `service/impl/JenkinsServiceImpl.java` | Trigger pipeline, queue polling, progressive log streaming |
| `service/JenkinsLogStreamListener.java` | Callbacks for open/log/queued/heartbeat/done/error |
| `service/GitHubService.java` | GitHub repo listing contract |
| `service/impl/GitHubServiceImpl.java` | Decrypt token and fetch paginated `/user/repos` |
| `service/ArgoCdService.java` | ArgoCD app creation contract |
| `service/impl/ArgoCdServiceImpl.java` | Creates ArgoCD Application if missing |
| `service/TokenEncryptionService.java` | Encrypt/decrypt contract |
| `service/impl/TokenEncryptionServiceImpl.java` | AES-GCM implementation for token-at-rest encryption |

### WebSocket

| File | Responsibility |
|---|---|
| `websocket/JenkinsLogWebSocketHandler.java` | Parses query params, streams queue/build logs over WS |

### Persistence

| File | Responsibility |
|---|---|
| `model/Project.java` | Project entity |
| `model/GitHubToken.java` | Encrypted GitHub token entity |
| `repository/ProjectRepository.java` | Project queries by user/app |
| `repository/GitHubTokenRepository.java` | Token query by user |

### DTOs

| File | Responsibility |
|---|---|
| `dto/ProjectRequest.java` | Deploy request payload validation |
| `dto/DeployProjectResponse.java` | Response with project + Jenkins queue metadata |
| `dto/JenkinsBuildTriggerResult.java` | Jenkins trigger result contract |
| `dto/RepoDto.java` | Repository response projection |

## Core call chains

### Deploy chain

`ProjectController.createProject` -> `JenkinsServiceImpl.triggerBuild` -> save `Project(status=BUILDING)` -> return queue info to client.

### Repo list chain

`RepoController.getRepos` -> `GitHubServiceImpl.getReposForUser` -> decrypt token -> call GitHub API.

### Log stream chain

WebSocket open -> JWT interceptor -> `JenkinsLogWebSocketHandler` -> `JenkinsServiceImpl.streamBuildLogsFromQueueAsync` -> progressive Jenkins log fetch -> WS event messages.

## Integration notes

- Queue item id is returned so frontend can stream logs before build number exists.
- API security is stateless JWT.
- WebSocket auth uses query `token` due browser header limitations.
- ArgoCD service exists but deployment flow currently centers on Jenkins + GitOps update.
