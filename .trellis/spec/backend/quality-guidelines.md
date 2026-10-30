# Quality Guidelines

> Code quality standards for server-side development.

---

## Overview

Validation: `node_modules/.bin/tsc --noEmit`, `npm test`, `npm run lint`. All three must pass before any change is handed over. `npm test` first runs `npm run check:deps` and stops on lockfile/node_modules drift — fix with `npm ci`, not by editing the check.

## Forbidden Patterns

- **`next build` during dev.** It pollutes `.next/` and breaks `npm run dev`. Production builds go to the isolated dir: `PI_WEB_DESKTOP_BUILD=1 node_modules/next/dist/bin/next build --webpack`.
- **Building from inside the Pi desktop app's shell.** It exports `__NEXT_PRIVATE_ORIGIN` / `__NEXT_PRIVATE_STANDALONE_CONFIG` and the build dies with `TypeError: generate is not a function`. Run from a normal terminal (or delete both vars first, like `scripts/prepare-desktop.mjs` does).
- **Literal `homedir()`** in session-scanner fs calls. Use `userHome()` routing (`AGENTS.md` Hard rules).
- **A second path for exact system prompts.** Only `lib/exact-system-prompt.ts` (`before_agent_start`).
- **Second MCP/Code-mode wiring.** They run on ADR 0006's `builtin:*` extensions; `resolveActiveToolNames()` replaced `withExtensionTools()`.
- **Editing `package-lock.json` tauri plugin versions by hand** — they stay pinned to `src-tauri/Cargo.lock`; restore HEAD's lock before `npm install --package-lock-only`.
- **Hard-coding a nested staging destination** in desktop packaging; `npm run desktop:prepare && npm run desktop:verify` after any SDK bump.

## Required Patterns

- `tauri-plugin-*` crates and `@tauri-apps/plugin-*` npm packages stay on the same minor; fix drift in `Cargo.lock`, never by pinning npm down.
- Window state flags: `SIZE | POSITION | MAXIMIZED` — never `VISIBLE` or `DECORATIONS`.
- Open local paths with `explorer.exe` on Windows, never `cmd.exe /C start`.
- Native popup menus are built in Rust (`show_popup_menu`).
- After an upstream merge: `node scripts/upstream-css-baseline.mjs update <ref>` and `npm run drift`.

## Testing Requirements

- New non-trivial logic gets a same-directory `*.test.mjs` (node --test). Release tooling tests: `npm run test:release-tools`.
- `npm run test:e2e` refuses to start while a dev server holds `.next/dev/lock`; see `AGENTS.md` "Local environment traps" for the isolated-build workaround.

## Code Review Checklist

- Hard rules in root `AGENTS.md` all intact?
- Topic doc (`docs/agents/<area>.md`, `fork-*.md` wins) updated if behavior changed?
- ADR list (`docs/adr/`) still accurate for the subsystem touched?
