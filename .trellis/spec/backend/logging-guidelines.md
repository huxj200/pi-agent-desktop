# Logging Guidelines

> Server-side logging and diagnostics conventions.

---

## Overview

There is no structured logging library. Server-side diagnostics are `console` writes from Next.js route handlers and lib code. In the desktop shell they do NOT land in the repo's `.logs/` — the Rust side spawns the Node server with stdout/stderr redirected to `server.log` under Tauri's `app_log_dir()`, and its own diagnostics go to `desktop.log` (256 KB cap, rotated to `desktop.log.1`, panics captured by a custom hook — `src-tauri/src/logging.rs`, `src-tauri/src/lib.rs` `spawn_server`). The repo's `.logs/` is a local dev artifact (gitignored). Client-side state that must survive reloads goes through `lib/app-prefs.ts`, not logs.

## Where Output Goes

- Desktop packaged app: `server.log` (Node stdout+stderr) and `desktop.log` (Rust diagnostics + panics) in the OS app log dir; users open it via the `open_log_dir` Tauri command.
- Local dev: repo `.logs/` and terminal output; both gitignored/ephemeral.
- Unit-test failures print via `node --test`; e2e output lands in `test-results/` (gitignored).

## Hard Red Lines

- Never log or return credentials: OAuth tokens, API keys, `~/.pi/agent/auth.json` contents, `.env*` values. Model-config reads redact literal API keys (`docs/agents/models.md`).
- MCP test/sign-in output goes through the redaction rules in `docs/agents/mcp-test-sign-in.md` before it reaches any log surface.
- Throttle and auth state must not leak retry timing in a way an attacker can read from responses (`docs/agents/fork-files-security.md`).

## Patterns

- Prefer returning typed failure values (`withModelRuntimeError`-style wrappers) over printing; the UI surfaces errors, the log only aids local debugging.
- Windows-specific shell behavior (explorer.exe, junctions) is documented in `docs/agents/fork-desktop.md`, not discovered via logging in production paths.

## Common Mistakes

- Writing `JSON.stringify` of a whole SDK settings object — it can contain provider credentials. Log the field you need, redacted.
- Logging inside hot SSE event loops; it ends up in `.logs/` at stream rate.
