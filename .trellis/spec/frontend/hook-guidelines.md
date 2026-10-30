# Hook Guidelines

> How hooks are used in this project.

---

## Overview

Hooks live in `hooks/`, `use*` naming, one concern per hook. The heaviest hook, `useAgentSession.ts`, owns messages/streaming/SSE/fork-navigate/reconciliation — read it before touching anything session-related.

## Custom Hook Patterns

Module-level store + `useSyncExternalStore` subscription for cross-component client state. Real example, `hooks/useTheme.ts`:

```ts
// Module-level singleton state, listeners, SSR-safe snapshot
const SERVER_SNAPSHOT: ThemeState = { preference: "auto", theme: "light" };
const listeners = new Set<() => void>();
let state: ThemeState | null = null;

function emit(): void { listeners.forEach((cb) => cb()); }
```

Components then subscribe via `useSyncExternalStore`. Shared stores (scheduled tasks: `hooks/useScheduledTasks.ts`) follow the same shape plus an SSE feed.

## Data Fetching

- No React Query/SWR. Server state reaches the client through typed `fetch` helpers (`lib/agent-client.ts`) and SSE streams (`app/api/agent/[id]/events`).
- `useAgentSession` load path must keep: monotonic request ids, `fetchWithRetry`, `seedStreamingSnapshot()`, the render-phase scroll save, and the `sessionHookMountedRef` re-assert before `maintainEventsConnected()`. These are regression-tested facts, not style (`docs/agents/fork-sessions.md`).

## Naming Conventions

- `useXxx.ts`, file name = hook name. `useIsMobile`, `useAudio`, `useDragDrop` are the reference sizes for a new hook.
- Private helpers inside a hook file stay unexported; anything shared moves to `lib/`.

## Common Mistakes

- Sending a second raw send from the composer — the composer sends exactly one `onSend` per submit (`AGENTS.md` Hard rules).
- Bypassing `hooks/useAgentSession.ts` to reach the session API with a fresh `fetch` — you lose the request-id guards and reintroduce stale-snapshot bugs.
- Storing preferences directly in `localStorage`; go through `lib/app-prefs.ts` (`APP_PREF_KEYS`).
