# State Management

> How state is managed in this project.

---

## Overview

There is no global state library (no Redux/Zustand/Jotai). State is split into four explicit kinds, each with a fixed home:

## State Categories

| Kind | Home | Example |
|------|------|---------|
| URL/tab state | `components/AppShell.tsx` (URL params + tab management) | open session, file tabs |
| Persisted client prefs | `lib/app-prefs.ts` (`APP_PREF_KEYS` + typed get/set over localStorage) | theme, sidebar widths, drafts (`pi-workspace-v1`) |
| Live server state | `hooks/useAgentSession.ts` + SSE (`/api/agent/[id]/events`) | messages, streaming, running state |
| Server-runtime shared state | `globalThis` keys in `lib/rpc-manager.ts` | `__piSessions` (one wrapper per session id), `__piStartLocks` |

## When to Use Global State

Almost never. Module-level singleton + `useSyncExternalStore` (see `hooks/useTheme.ts`, `hooks/useScheduledTasks.ts`) is the promotion path for cross-component client state. If two components need the same derived value, put the derivation in a hook or `lib/` helper instead of introducing a store.

## Server State

- `globalThis` survives Next.js hot-reload; a plain module-level Map does not — that is why wrapper registries live there (`AGENTS.md` "AgentSession lifecycle").
- Idle timeout 10 minutes; concurrent `startRpcSession()` calls share a single start Promise.
- Client SSE consumers never mutate session state locally; reconciliation flows through `useAgentSession`.

## Common Mistakes

- Reading `localStorage` directly instead of through `lib/app-prefs.ts` (SSR/private-mode safety lives there).
- Duplicating running-state logic: `isRunning()` ORs in `inner.isIdle === false`; don't reimplement it per component.
- Putting per-project UI toggles in `globalThis` — that is for server runtime singletons only.
