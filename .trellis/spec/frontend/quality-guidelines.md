# Quality Guidelines

> Code quality standards for frontend development.

---

## Overview

Validation is command-based and must be run after any change: `node_modules/.bin/tsc --noEmit` (types), `npm test` (unit tests), `npm run lint` (eslint + branding check). UI menu work additionally runs `node --test scripts/ui-menu-consistency.test.mjs` and, for shared choices, `node e2e/option-controls.mjs`.

## Forbidden Patterns

- **Raw `<select>` or OS-native option menus.** Form choices use `components/AppSelect.tsx`; every popup uses the shared `.app-menu` surface and menu tokens in `app/native-theme.css`. (`AGENTS.md` "Styling" section; `docs/ui-menu-audit.md`.)
- **Editing `app/globals.css` or `app/settings.css`.** They stay byte-identical to upstream — `scripts/upstream-css-baseline.test.mjs` fails on any edit. Every fork rule goes in `app/native-theme.css`.
- **New npm dependencies.** Look for an existing helper in `lib/`/`components/` or the pi SDK first.
- **Assigning `agent.state.systemPrompt`** (it is a getter since pi 0.86). Exact prompts go only through `lib/exact-system-prompt.ts`.
- **Re-adding declined upstream UI** (sidebar explorer resize handles, `DirectoryPicker`, `SessionSearch`, `ChatMinimap`, topbar quick-buttons, `ExtensionStatusBar`). See `docs/agents/fork-ui.md` and `AGENTS.md` "Upstream merge decisions".
- **Deleting assertions in `components/fork-extractions.test.mjs`** when they fail. A failure means restore the behavior, not drop the marker (`docs/ownership-boundaries.md`).

## Required Patterns

- Fork CSS rules live only in `app/native-theme.css`, overrides that must beat other theme rules go in its "fork base layer" at the top.
- Non-trivial new logic (branch, loop, parser, money/security) ships with a same-directory `*.test.mjs` runnable by `node --test`.
- i18n: user-facing strings go through the desktop i18n layer (`docs/i18n.md`), not hardcoded English in new surfaces.

## Testing Requirements

- Unit tests are `*.test.mjs` colocated with the code (`components/*.test.mjs`, `scripts/*.test.mjs`, ...), run via `npm test` (which first runs `npm run check:deps`).
- Windows fixtures use `scripts/test-platform.mjs`: junctions for directory links; POSIX mode checks stay active on POSIX hosts (details in `AGENTS.md`).
- Layout is pinned by a test: `components/AppShell.right-panel-row.test.mjs`. Keep it passing.

## Code Review Checklist

- Did you read the matching `docs/agents/<topic>.md` (and `fork-*.md` if one exists) before touching the area?
- Did the change keep every applicable "Hard rules" entry in `AGENTS.md` intact?
- E2E: never run `next build` during dev. If a production build is needed, use `PI_WEB_DESKTOP_BUILD=1` (builds into gitignored `.next-desktop`).
