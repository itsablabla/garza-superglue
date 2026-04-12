# Changelog — Garza Glue (fork of superglue-ai/superglue)

All custom changes made on top of the upstream superglue codebase.

---

## PR #6 — Revert @serwist/next, restore hand-rolled PWA service worker
**Branch**: `devin/1776031795-revert-serwist`

- Reverted PR #5 — serwist migration broke mobile spacing and removed voice memo UI
- Restored `public/sw.js` (hand-rolled service worker with BUILD_ID cache versioning)
- Restored Dockerfile BUILD_ID sed injection
- Removed serwist/concurrently/esbuild devDependencies
- Restored original `dev` and `build` scripts

---

## PR #4 — Fix chat framing, add voice memos, improve PWA service worker
**Branch**: `devin/1744438820-chat-voice-pwa`

- Fixed chat content clipping after extended use (removed `visualViewport` transform hack)
- Added voice memo support to agent chat input
  - Web Speech API for real-time transcription (Chrome/Edge)
  - MediaRecorder API fallback for audio file recording (Safari/Firefox)
  - Mic button with red pulse animation when recording
- Added `interactive-widget=resizes-content` viewport meta for iOS keyboard handling
- Improved service worker with build-time cache versioning via BUILD_ID

### Files Modified
- `packages/web/src/app/layout.tsx` — viewport meta tags
- `packages/web/src/app/client-layout.tsx` — removed visualViewport hack
- `packages/web/src/components/agent/AgentInputArea.tsx` — voice recording
- `packages/web/public/sw.js` — cache versioning, stale-while-revalidate

---

## PR #3 — iOS safe area inset padding for PWA standalone mode
**Branch**: `devin/1744438820-ios-safe-area`

- Added `env(safe-area-inset-top/bottom)` padding to main layout containers
- Fixed "Past Chats" button overlapping iOS status bar/notch/Dynamic Island
- Set `html` background color to match app theme behind the notch

### Files Modified
- `packages/web/src/app/client-layout.tsx` — safe area padding on containers
- `packages/web/src/app/globals.css` — html background color

---

## PR #2 — Server-side chat persistence + mobile optimization + rebrand
**Branch**: `devin/1744438820-garza-glue`

### Server-Side Chat Persistence
- Created REST API endpoints for conversations (PUT/GET/DELETE)
- Added `conversations` table to Postgres datastore
- Conversations stored per-org with full message history
- Chat survives page refresh and browser restart

### Garza Glue Branding
- Replaced all user-facing "superglue" text with "Garza Glue"
- Updated logo SVG/PNG, page titles, chat placeholders, welcome page
- Agent prompts reference "Garza Glue" instead of "superglue"

### Mobile Optimization
- Floating sidebar: hamburger menu opens card overlay (like Devin's app)
- iOS Safari scroll lock: JS `touchmove` interception with `passive: false`
- `h-dvh` for dynamic viewport height (keyboard-safe)
- iOS Install Banner: detects Safari, shows "Add to Home Screen" instructions

### PWA Foundation
- Created `manifest.json` with Garza Glue branding
- Created `offline.html` branded fallback page
- Added PWA icons (192x192, 512x512)
- iOS meta tags: `apple-mobile-web-app-capable`, `black-translucent` status bar
- Manual service worker registration

### Restored EE Pages
- Landscape page (`/landscape`)
- Control Panel with sub-pages (`/control-panel`, `/control-panel/overview`, `/control-panel/schedules`, `/control-panel/api-keys`)
- Notifications page (`/notifications`)
- Runs page with detail view (`/runs`, `/runs/[id]`)

### Bug Fixes
- Set `orgId: "default"` for self-hosted deployments (was `""`, disabling all React Query hooks)
- Added `deno-runtime/` copy to Dockerfile production stage
- Set API body limit to 50MB (was unlimited)

### Files Added
- `packages/core/api/ee/conversations.ts`
- `packages/web/public/manifest.json`
- `packages/web/public/offline.html`
- `packages/web/public/icon-192x192.png`
- `packages/web/public/icon-512x512.png`
- `packages/web/src/app/landscape/page.tsx`
- `packages/web/src/app/control-panel/page.tsx`
- `packages/web/src/app/control-panel/overview/page.tsx`
- `packages/web/src/app/control-panel/schedules/page.tsx`
- `packages/web/src/app/control-panel/api-keys/page.tsx`
- `packages/web/src/app/notifications/page.tsx`
- `packages/web/src/app/runs/page.tsx`
- `packages/web/src/app/runs/[id]/page.tsx`
- `packages/web/src/components/pwa/IOSInstallBanner.tsx`
- `packages/web/src/hooks/use-ios-scroll-lock.ts`

### Files Modified
- `packages/core/api/ee/index.ts` — import conversations
- `packages/core/api/api-server.ts` — body limit
- `packages/core/datastore/postgres.ts` — conversation CRUD
- `packages/core/datastore/types.ts` — Conversation type
- `packages/shared/types.ts` — Conversation interface
- `packages/web/src/app/layout.tsx` — branding, viewport meta, orgId fix
- `packages/web/src/app/client-layout.tsx` — safe area, scroll lock, SW registration
- `packages/web/src/app/welcome/page.tsx` — Garza Glue branding
- `packages/web/src/app/page.tsx` — branding
- `packages/web/src/app/agent/page.tsx` — branding
- `packages/web/src/components/sidebar/LeftSidebar.tsx` — floating mobile sidebar, full nav
- `packages/web/src/components/agent/AgentInputArea.tsx` — voice recording
- `packages/web/src/components/agent/AgentInterface.tsx` — branding
- `packages/web/public/logo.svg` — Garza Glue logo
- `docker/Dockerfile` — Deno installation, deno-runtime copy

---

## PR #1 — Allow agents to set credentials directly
**Branch**: `devin/1744438820-agent-credentials`

- Agents can now set system credentials programmatically without requiring the user to fill out a confirmation form
- Streamlines the credential flow for automated system setup

### Files Modified
- `packages/core/api/systems.ts` — direct credential set endpoint
- `packages/web/src/components/systems/sections/AuthenticationSection.tsx` — agent credential flow
