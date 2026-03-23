# 11 - Security and Hardening Notes

## Current observations from workspace

Sensitive values are currently present in tracked local files. Examples include:

- `monolotic-app-backend/src/main/resources/application-dev.yml`
- `monolotic-app-frontend/.env.local`

These files currently include real-looking secrets/tokens (GitHub/Jenkins/Argo/JWT/encryption keys).

## Immediate actions recommended

1. Rotate all exposed credentials immediately.
2. Remove sensitive values from tracked files.
3. Move secrets to secret manager or CI/CD secret store.
4. Add `.env.local` and local override files to `.gitignore` if not already excluded.
5. Use environment placeholders in committed config.

## Backend hardening

- Use strong random secrets for JWT and encryption keys.
- Store token-encryption key in secret manager, not source.
- Shorten backend JWT TTL or add refresh flow.
- Add endpoint-level authorization checks for tenant isolation.
- Add rate limiting and audit logging for auth/deploy endpoints.

## WebSocket hardening

- Prefer short-lived websocket token or one-time session token.
- Avoid long-lived JWT in query parameters where possible.
- Enforce origin checks and strict CORS values.

## CI/CD and supply chain hardening

- Restrict Jenkins credentials per job/folder.
- Sign container images and verify at deploy time.
- Pin base image digests for Dockerfile templates.
- Add SAST/secret scanning in CI before merge.

## GitOps hardening

- Keep registry credentials out of Git; use External Secrets/Sealed Secrets.
- Restrict ArgoCD project permissions by namespace/repo path.
- Require review/approval policy for manual GitOps edits.
