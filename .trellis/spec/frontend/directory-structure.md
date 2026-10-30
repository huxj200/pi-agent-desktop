# Directory Structure

> How frontend code is organized in this project.

---

## Overview

The UI is a Next.js 16 App Router app (`app/`) wrapped in a Tauri 2 desktop shell. Frontend code lives in four top-level directories. The authoritative file map is in the root `AGENTS.md` — read its "File Map" section before adding any new file; this spec mirrors it at a glance.

Layout is pinned by `components/AppShell.right-panel-row.test.mjs`: the center column contains topbar and chat; the right panel is a full-height sibling of the outer row. Do not add `paddingLeft` to the topbar for the sidebar.

## Directory Layout

```
app/                 App Router pages + /api routes (thin HTTP layer, logic stays in lib/)
  api/               One route.ts per endpoint; routes import from lib/, never duplicate logic
components/          React components. Settings dialogs, chat, sidebar, artifact panel
hooks/               Client hooks (use*). Chat/session/SSE logic lives in useAgentSession.ts
lib/                 Shared client+server TypeScript: helpers, SDK adapters, types
public/preview-runtime/  Bundled React/Sucrase/Tailwind for sandboxed artifact previews
src-tauri/           Rust desktop shell (menus, window state, native dialogs)
docs/agents/         Topic notes — read the one matching the area you touch before coding
scripts/             Node tooling + *.test.mjs unit tests
```

## Module Organization

- New UI feature → component in `components/` (or a subfolder like `components/scheduled/`), hook in `hooks/`, server logic in `lib/`.
- New API endpoint → `app/api/<area>/route.ts` that imports from `lib/`. Keep the route file thin (see `app/api/models/route.ts`: validation + collation at the top, model loading delegated to `lib/models-cache.ts`).
- Shared types go in `lib/types.ts`; structural pi SDK types in `lib/pi-types.ts`.
- Topic docs in `docs/agents/` own the "why" for each subsystem — update the matching doc when behavior changes.

## Naming Conventions

- Components: `PascalCase.tsx` (`FileViewer.tsx`); hooks: `useXxx.ts`; lib modules: `kebab-case.ts` (`session-reader.ts`).
- Tests: same directory as the code, `*.test.mjs`, run with `node --test` via `npm test`.
- CSS variables come from `app/globals.css` tokens (`--bg`, `--bg-panel`, `--accent`, ...) re-tokened in `app/native-theme.css`.

## Examples

- `components/AppShell.tsx` — layout + URL state + tab management (top-level composition).
- `hooks/useScheduledTasks.ts` — shared store + SSE pattern.
- `docs/agents/settings-ui.md` — read before adding any settings section or add pane.
