# Persistence Guidelines

> This project has no database. All persistence is files and process-level singletons.

---

## Overview

Data survives in three forms. Treat these files as the "schema": never parse them ad-hoc, always go through the owning module.

## Session Storage (append-only log)

`~/.pi/agent/sessions/<encoded-cwd>/<timestamp>_<uuid>.jsonl`. Entry types: `session`, `model_change`, `message`, `compaction`, `session_info` (full shape in root `AGENTS.md` "Pi Session File Format"). Access only through:

- `lib/session-reader.ts` — SDK `SessionManager` wrappers + path cache (`buildSessionContext` adapter).
- `lib/normalize.ts` — `normalizeToolCalls()` bridges file-format field names to our types.
- Session listing: `lib/session-list-scanner.ts` (uses `userHome()`, never literal `homedir()`).

Fork (new `.jsonl` file, `parentSession` header) and in-session branch (`navigate_tree`, shared `parentId`) are different operations — do not conflate them.

## Configuration Files (JSON)

| File | Owner | Rule |
|------|-------|------|
| `~/.pi/agent/settings.json` | pi SDK settings manager | Project overrides agent-dir settings; resource lists combine |
| `~/.pi/agent/models.json` | `app/api/models-config/**` + `lib/enabled-models*.ts` | Literal API keys are redacted in reads; only minimal edits write `enabledModels` |
| `~/.pi/agent/auth.json` | `lib/*credential*` helpers | Deletes go through `removeStoredCredentialIfType()` |
| `.pi/mcp.json` / `~/.pi/agent/mcp.json` | `lib/mcp-config-file.ts` | Every write has an undo (`lib/mcp-undo.ts`); reads never execute anything |

## Process Singletons

`globalThis.__piSessions` (session wrappers) and `globalThis.__piStartLocks` (start promises) in `lib/rpc-manager.ts`; scheduler state on `globalThis` (`docs/agents/scheduled-tasks.md`). They survive Next.js hot-reload — that is the entire reason they are there.

## Forbidden Patterns

- Reading/writing any of these files with bare `fs` calls outside the owning lib module.
- Running `next build` and assuming `.next/` state is disposable mid-dev (it breaks the dev server — see backend quality spec).
- Trusting session `.jsonl` files as a safe cross-session mutable store: entries are append-only; rewrites happen only through the SDK session rewrite path.
