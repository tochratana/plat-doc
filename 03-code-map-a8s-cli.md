# 03 - Code Map: `a8s-cli`

## What this module does

`a8s-cli` is a Go command-line client for:

- authentication (`login`, `logout`, `config`)
- deployment trigger (`deploy`)
- read models (`repos`, `projects`)
- operational actions (`logs`, `rollback`)

## Execution flow

1. `main.go` receives CLI args.
2. `internal/cli/run.go` routes command.
3. Command handler parses flags and validates inputs.
4. Handler reads local config/token from `internal/config/config.go`.
5. Handler calls backend API through `internal/api/client.go`.
6. Output is printed as table/plain/json.

## File-by-file responsibilities

| File | Responsibility |
|---|---|
| `main.go` | Program entrypoint; executes `cli.Run` and exits non-zero on error |
| `internal/cli/run.go` | Command router and usage text |
| `internal/cli/login.go` | Login flow (PAT or GitHub device flow), exchanges for backend JWT |
| `internal/cli/deploy.go` | Deploy command; auto-detects repo/branch/app name, triggers project creation |
| `internal/cli/deploy_context.go` | Git metadata detection and app-name normalization |
| `internal/cli/repos.go` | Lists repos from backend `/api/v1/repos` |
| `internal/cli/projects.go` | Lists projects from backend `/api/v1/projects` |
| `internal/cli/logs.go` | Reads deployment logs, optional polling/follow mode |
| `internal/cli/rollback.go` | Calls deployment rollback endpoint |
| `internal/cli/config.go` | Displays effective local config and token status |
| `internal/cli/logout.go` | Clears token or deletes config file |
| `internal/api/client.go` | Shared HTTP request helper, auth header, error shaping, pretty JSON |
| `internal/config/config.go` | Local config path resolution, env overrides, read/write JSON config |
| `internal/oauth/device.go` | GitHub device-code OAuth flow (start, poll, browser open helper) |
| `internal/types/types.go` | Request/response DTOs shared by CLI handlers |
| `pkg/output/table.go` | Tabular formatter for console output |
| `pkg/version/version.go` | CLI version constant |
| `config/config.example.yaml` | Onboarding reference config (runtime config is JSON in home dir) |
| `Makefile` | Build/run/install/test shortcuts |
| `push.sh` | Helper script to git add/commit/push |

## Command-to-endpoint mapping

| Command | Backend endpoint |
|---|---|
| `a8s login` | `POST /api/v1/auth/github` |
| `a8s repos` | `GET /api/v1/repos` |
| `a8s projects` | `GET /api/v1/projects` |
| `a8s deploy` | `POST /api/v1/projects` |
| `a8s logs` | `GET /api/v1/deployments/{deploymentId}/logs` |
| `a8s rollback` | `POST /api/v1/deployments/{deploymentId}/rollback` |

## Integration notes

- `deploy` auto-runs login if backend token is missing.
- Deploy metadata is auto-detected from the current git repository if not provided.
- CLI uses backend JWT, not GitHub token directly, after login exchange.
- `logs --follow` polls logs endpoint; it does not use websocket.
