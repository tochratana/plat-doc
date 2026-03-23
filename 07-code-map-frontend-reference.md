# 07 - Code Map: `platform-frontend-reference`

## What this module does

This folder provides lightweight UI architecture examples for:

- project cards
- deployment timeline
- live log viewer via WebSocket

It is not a full production app; it is a reference scaffold.

## File-by-file responsibilities

| File | Responsibility |
|---|---|
| `README.md` | Purpose and scope of reference module |
| `src/lib/api.ts` | Generic authenticated request helper |
| `src/lib/ws.ts` | Builds websocket URL for deployment logs |
| `src/store/deploymentSlice.ts` | Minimal deployment UI state shape |
| `src/components/projects/ProjectCard.tsx` | Project summary card with domain link |
| `src/components/deployments/DeploymentTimeline.tsx` | Deployment history visualization |
| `src/components/logs/LogsViewer.tsx` | WebSocket log consumer and renderer |
| `src/app/dashboard/page.tsx` | Demo dashboard page using timeline |
| `src/app/dashboard/projects/page.tsx` | Demo projects page using cards |
| `src/app/dashboard/logs/[deploymentId]/page.tsx` | Demo logs page wiring deployment id to websocket viewer |

## Integration notes

- WebSocket URL points to FastAPI pattern: `/ws/projects/{projectId}/deployments/{deploymentId}/logs`.
- Components are intentionally reusable and minimal.
- Mock data is used in some pages to demonstrate structure.
