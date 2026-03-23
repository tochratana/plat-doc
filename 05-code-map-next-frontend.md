# 05 - Code Map: `monolotic-app-frontend` (Next.js)

## What this module does

This is the active user dashboard.

Responsibilities:

- GitHub OAuth login via NextAuth
- backend token exchange + session augmentation
- repository listing and deploy action
- project listing/status view
- Jenkins live log stream over backend websocket

## Architecture

- App Router pages in `src/app`
- Auth routes in `src/app/api/auth`
- Redux state for repos/projects
- API client utilities in `src/lib`
- Session + store provider wrapper in `src/components/Providers.tsx`

## File-by-file responsibilities

### Core app shell

| File | Responsibility |
|---|---|
| `src/app/layout.tsx` | Root layout, fonts, providers, global metadata |
| `src/components/Providers.tsx` | Wraps app with `SessionProvider` and Redux `Provider` |
| `src/app/globals.css` | Global Tailwind/CSS styles |
| `src/app/page.tsx` | Landing/login page and redirect to dashboard |
| `src/app/dashboard/layout.tsx` | Dashboard shell, sidebar, user profile, sign-out |

### Dashboard pages

| File | Responsibility |
|---|---|
| `src/app/dashboard/page.tsx` | Repo list + deploy action + live queue/build log stream |
| `src/app/dashboard/projects/page.tsx` | Project table + manual Jenkins stream controls |

### API routes (server side)

| File | Responsibility |
|---|---|
| `src/app/api/auth/[...nextauth]/route.ts` | NextAuth config, GitHub provider, JWT/session callbacks |
| `src/app/api/sync-token/route.ts` | Explicit sync route to exchange GitHub token with backend |
| `src/app/api/jenkins/logs/route.ts` | Proxies SSE stream from backend with backend token |

### Client utilities

| File | Responsibility |
|---|---|
| `src/lib/api.ts` | Typed fetch wrapper and API functions (`repos`, `projects`, `deploy`) |
| `src/lib/utils.ts` | CSS utility merge helper |

### State management

| File | Responsibility |
|---|---|
| `src/store/index.ts` | Redux store setup |
| `src/store/repoSlice.ts` | Repo loading state/actions |
| `src/store/projectSlice.ts` | Project loading state/actions |

## Core call chains

### Login chain

GitHub OAuth -> NextAuth `jwt` callback -> backend `/api/v1/auth/github` -> backend token stored in session.

### Deploy chain

Dashboard deploy button -> `deployProject(...)` -> backend project deploy endpoint -> queue id + job name -> open websocket stream for logs.

### Project status chain

Projects page load -> `getUserProjects(...)` -> render status table and optional URL link for deployed apps.

## Integration notes

- Frontend relies on backend response fields: `queueItemId`, `jenkinsJobName`, `queueUrl`.
- WebSocket URL protocol switches automatically (`ws` or `wss`) based on API base URL.
- Browser websocket includes token in query string for backend handshake validation.
