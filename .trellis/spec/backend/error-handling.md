# Error Handling

> Server-side error handling conventions.

---

## Overview

Errors are handled at the boundary that owns the concern: HTTP routes return typed failure shapes, SSE consumers reconnect with backoff, and agent-level retries are delegated to the pi SDK. Do not scatter validation across layers — validate once at the entry point (see `docs/agents/fork-sessions.md`).

## HTTP Routes

- Routes wrap failures in the typed helpers their area already provides instead of ad-hoc `500` strings. Example: `app/api/models/route.ts` uses `withModelRuntimeError` / `withSafeModelLoadFailure` from `lib/models-cache.ts`.
- MCP settings reads never execute anything; every `mcp.json` write has an undo (`lib/mcp-undo.ts`). See `docs/agents/mcp-settings.md`.

## Retry and Streaming

- Session loads go through `fetchWithRetry` in `hooks/useAgentSession.ts` — do not add a parallel hand-rolled retry.
- Do not re-add a manual DeepSeek xhigh clamp or legacy `auto_compaction_*` event cases; the SDK owns both (`AGENTS.md` Hard rules).
- `AgentSessionWrapper.isRunning()` ORs in `inner.isIdle === false`. Dropping that evicts wrappers mid-retry.

## Security Failure Modes

- `/api/files` is only as safe as `lib/file-access.ts` (`docs/agents/files-and-access.md`). Any new file-serving path must go through the allow-list.
- Native-dialog routes (`/api/desktop/*`, `?type=import`) require the per-process desktop token — never a static token or a User-Agent check.
- Never give one iframe both `allow-scripts` and `allow-same-origin` (artifact previews; `docs/agents/fork-files-security.md`).
- The web-auth throttle stays global and its reset window stays longer than the max delay.
- Manual-code OAuth handshake tokens are `crypto.randomUUID()`.

## Common Mistakes

- Catching an SSE error and silently swapping in a stale snapshot — the `useAgentSession` guards (monotonic request ids) exist to prevent exactly this.
- Assuming `node` on `PATH` is the runtime the dev server uses. It isn't (the desktop app bundles its own); check before spawning.
