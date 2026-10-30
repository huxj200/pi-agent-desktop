# Directory Structure

> How server-side code is organized in this project.

---

## Overview

There is no standalone backend. The server is the Next.js Node runtime running inside the Tauri desktop shell (or a plain terminal). Server code spans three layers: `app/api/**` route handlers (HTTP layer), `lib/` (all real logic), and `src-tauri/` (Rust: window state, native menus, dialogs). Session work runs in-process through the pi SDK (`lib/rpc-manager.ts`), not as separate services.

## Directory Layout

```
app/api/
  sessions/**    Read-only browsing of ~/.pi/agent/sessions/*.jsonl (GET)
  agent/**       POST any command to an AgentSession; SSE event stream (GET .../events)
  auth/**        OAuth device-code flows + API-key storage for providers
  models*/**     Model catalogs, models.json read/write, enabledModels switches
  mcp/**         mcp.json read/add/test + sign-in (writes always have undo)
  scheduled-tasks/**  Server-side scheduler: tasks, runs, SSE
  desktop/**     Native dialogs — require the per-process desktop token
  files/**       File contents for the viewer, behind lib/file-access.ts allow-list
lib/             All business logic; route files import from here
src-tauri/src/   Rust shell: popup menus (show_popup_menu), window flags, tray
```

## Module Organization

- Route files stay thin: parse/validate input, call a `lib/` helper, return. Example: `app/api/models/route.ts` delegates loading to `lib/models-cache.ts` and gating to `lib/project-trust.ts`.
- One `AgentSessionWrapper` per session id, keyed in `globalThis.__piSessions` (`lib/rpc-manager.ts`). `globalThis` survives Next.js hot-reload; a plain module-level Map does not. Idle timeout 10 min; concurrent `startRpcSession()` calls share one start Promise (`globalThis.__piStartLocks`).
- Anything about sessions/SSE must respect the guards in `hooks/useAgentSession.ts` (monotonic request ids, `fetchWithRetry`, `seedStreamingSnapshot()`) — see `docs/agents/fork-sessions.md`.

## Naming Conventions

- Route dirs are kebab-case REST-ish areas; one `route.ts` each, `export const dynamic = "force-dynamic"` where fresh state matters.
- Lib modules kebab-case; scheduled-task internals under `lib/scheduled-tasks/`.

## Examples

- `app/api/agent/[id]/route.ts` — GET state | POST any command (the main RPC surface).
- `lib/session-reader.ts` — SDK `SessionManager` wrappers + path cache adapter.
- `docs/agents/scheduled-tasks.md` — why shared scheduler state lives on `globalThis`.
