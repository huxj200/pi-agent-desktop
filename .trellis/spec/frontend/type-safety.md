# Type Safety

> TypeScript conventions in this project.

---

## Overview

Strict TypeScript (`tsconfig.json`: `"strict": true`, `noEmit`, bundler module resolution, `@/*` path alias to repo root). Validation of the whole repo: `node_modules/.bin/tsc --noEmit`.

## Type Organization

- Shared wire/UI types: `lib/types.ts`.
- Structural types for pi SDK objects: `lib/pi-types.ts` — local mirrors instead of importing SDK internals; keep them structural so SDK bumps don't break the app.
- Contract drift between the session file format and our types is handled by `lib/normalize.ts` (`normalizeToolCalls()` fixes field-name mismatches) — never widen types to `any` to make a mismatch compile.

## Validation

- No runtime validation library (Zod etc.) is in the dependency set; do not add one. Validate at the entry point with plain TS guards (see `isThemePreference`, `isDarkTheme` in `lib/theme.ts`).
- Cross-boundary payloads (SSE events, `.jsonl` entries) get typed through `lib/session-reader.ts` adapters rather than inline casts.

## Common Patterns

- `Omit<ComponentPropsWithoutRef<...>, ...>` intersection types for component props (`components/AppSelect.tsx`).
- `as const` for string-key maps (`APP_PREF_KEYS` in `lib/app-prefs.ts`).
- Named exports of narrow literal-union types (`ThemePreference`, `ResolvedTheme`) co-located with their guard.

## Forbidden Patterns

- `any` in new code; `as unknown as X` double casts to force compile.
- Comparing `enabledModels` patterns as strings, or assuming `provider/*` covers a provider — the minimal-edit engine in `lib/enabled-models.ts` is the only writer (`docs/agents/fork-models.md`).
- Rebuilding model lists from `getAvailable()` instead of minimal edits.
