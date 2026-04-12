# Garza Glue

**AI-powered integration platform for building production-grade tools in natural language.**

Garza Glue is a customized enterprise deployment of [superglue](https://github.com/superglue-ai/superglue) with Garza-specific branding, mobile-first PWA support, server-side chat persistence, and additional enterprise pages.

---

## What's Different from Upstream Superglue

This repo contains all upstream superglue functionality **plus** the following custom features built for Garza:

### Custom Features

| Feature | Description | Key Files |
|---------|-------------|-----------|
| **Garza Glue Branding** | Full rebrand — logo, page titles, chat placeholders, welcome page, agent prompts | `layout.tsx`, `client-layout.tsx`, `welcome/page.tsx`, `logo.svg`, `logo.png` |
| **Server-Side Chat Persistence** | Conversations stored in Postgres (PUT/GET/DELETE), survive page refreshes | `packages/core/api/ee/conversations.ts`, `packages/core/datastore/postgres.ts` |
| **PWA Support** | Hand-rolled service worker with BUILD_ID cache versioning, offline fallback, push notifications | `public/sw.js`, `manifest.json`, `offline.html` |
| **iOS Safari Scroll Lock** | JS `touchmove` interception prevents rubber-banding in PWA standalone mode | `src/hooks/use-ios-scroll-lock.ts` |
| **iOS Install Banner** | Detects iOS Safari, shows "Add to Home Screen" instructions | `src/components/pwa/IOSInstallBanner.tsx` |
| **Voice Memos** | Mic button in chat — Web Speech API (Chrome/Edge) + MediaRecorder fallback (Safari/Firefox) | `src/components/agent/AgentInputArea.tsx` |
| **Mobile Floating Sidebar** | Hamburger menu opens a floating card overlay (not full-height drawer) | `src/components/sidebar/LeftSidebar.tsx` |
| **iOS Safe Area Insets** | Proper `env(safe-area-inset-*)` padding for notch/Dynamic Island in PWA | `client-layout.tsx`, `layout.tsx` |
| **Agent Direct Credentials** | Agents can set system credentials directly without user confirmation form | `packages/core/api/systems.ts`, `AuthenticationSection.tsx` |
| **Runs Page** | Full runs list with search, status filter, pagination + run detail view | `src/app/runs/page.tsx`, `src/app/runs/[id]/page.tsx` |
| **EE Pages Restored** | Landscape, Control Panel (Overview/Schedules/API Keys), Notifications | `landscape/`, `control-panel/`, `notifications/` |
| **Deno Runtime in Docker** | Dockerfile includes Deno installation for sandboxed tool execution | `docker/Dockerfile` |
| **Body Limit Fix** | API body limit set to 50MB (was unlimited, causing memory issues) | `packages/core/api/api-server.ts` |
| **Default orgId Fix** | Sets `orgId: "default"` for self-hosted deployments (was `""`, disabling all queries) | `layout.tsx` |

### Architecture

```
garza-superglue/
├── packages/
│   ├── core/               # Backend (Fastify + GraphQL + REST API)
│   │   ├── api/            # REST endpoints
│   │   │   └── ee/         # Enterprise endpoints (conversations, metrics, etc.)
│   │   ├── datastore/      # Postgres data layer (with conversation CRUD)
│   │   ├── scheduler/      # Cron scheduling (EE)
│   │   ├── notifications/  # Slack/email alerts (EE)
│   │   └── deno-runtime/   # Sandboxed JS execution
│   ├── web/                # Next.js 16 frontend (Turbopack)
│   │   ├── src/app/
│   │   │   ├── agent/         # AI agent chat (EE)
│   │   │   ├── runs/          # Runs list + detail pages
│   │   │   ├── landscape/     # System landscape view
│   │   │   ├── control-panel/ # Overview, Schedules, API Keys
│   │   │   └── notifications/ # Notification center
│   │   ├── public/
│   │   │   ├── sw.js          # PWA service worker
│   │   │   ├── manifest.json  # PWA manifest
│   │   │   ├── offline.html   # Offline fallback page
│   │   │   └── icon-*.png     # PWA icons
│   │   └── src/hooks/
│   │       └── use-ios-scroll-lock.ts  # iOS Safari fix
│   └── shared/             # Shared types and utilities
├── docker/
│   └── Dockerfile          # Production image (Node 22 + Deno)
└── docker-compose.yml      # Local development stack
```

---

## Quick Start

### Development

```bash
npm install
npm run dev          # Starts all services (turbo)
```

Ports: **3000** (GraphQL), **3001** (Web), **3002** (REST API)

### Docker (Production)

```bash
docker build -f docker/Dockerfile -t garza-glue:latest .
docker run -p 3000:3000 -p 3001:3001 \
  -e DATABASE_URL=postgresql://... \
  -e MASTER_ENCRYPTION_KEY=... \
  -e ENCRYPTION_KEY=... \
  -e AUTH_TOKEN=... \
  garza-glue:latest
```

### Docker Compose

```bash
docker-compose up -d
```

---

## Deployment

Currently deployed to:
- **s1.garzaglue.com** — Production
- **s2.garzaglue.com** — Staging

Both run Docker containers built from this repo with Traefik reverse proxy for HTTPS.

### Environment Variables

| Variable | Description |
|----------|-------------|
| `DATABASE_URL` | Postgres connection string |
| `MASTER_ENCRYPTION_KEY` | Encrypts system credentials at rest |
| `ENCRYPTION_KEY` | Secondary encryption key |
| `AUTH_TOKEN` | API authentication token |
| `GRAPHQL_PORT` | GraphQL server port (default: 3000) |
| `WEB_PORT` | Web UI port (default: 3001) |
| `REST_API_PORT` | REST API port (default: 3002) |

---

## Development Workflow

```bash
npm run dev          # Start all services
npm run test         # Run Vitest tests
npm run lint:fix     # Prettier formatting
npm run type-check   # TypeScript check
npm run build        # Production build
```

### Conventions

- **Imports at top** — no inline imports
- **Local imports use `.js` extension** — `import { x } from "./file.js"`
- **Types in shared** — `packages/shared/types.ts`
- **REST over GraphQL** — new endpoints go in `packages/core/api/`
- **Error handling** — `sendError(reply, 404, "message")`
- **Functional React** with TypeScript annotations
- **shadcn/Radix UI** components in `packages/web/src/components/ui/`
- **Tailwind** for styling, `cn()` for conditional classnames

---

## PWA Details

The service worker (`public/sw.js`) uses:
- **BUILD_ID cache versioning** — injected at Docker build time, auto-purges old caches on deploy
- **CacheFirst**: Static assets (images, fonts, icons)
- **StaleWhileRevalidate**: CSS/JS files
- **NetworkFirst**: API calls with timeout, page navigations
- **Offline fallback**: `/offline.html` for failed navigation requests
- **Push notifications**: Support for agent task completion alerts
- **Background sync**: Queue for offline chat messages

---

## Relationship to Upstream

This repo tracks `superglue-ai/superglue` as upstream. To pull upstream changes:

```bash
git remote add upstream https://github.com/superglue-ai/superglue.git
git fetch upstream main
git merge upstream/main
```

Custom changes are isolated to:
- `packages/core/api/ee/conversations.ts` (new file)
- `packages/core/datastore/postgres.ts` (conversation CRUD methods)
- `packages/core/datastore/types.ts` (Conversation type)
- `packages/shared/types.ts` (Conversation interface)
- `packages/web/` (branding, PWA, mobile fixes, new pages)
- `docker/Dockerfile` (Deno installation, BUILD_ID injection)

---

## License

FSL licensed (same as upstream superglue). Client SDKs are MIT licensed.
