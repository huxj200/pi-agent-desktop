# Component Guidelines

> How components are built in this project.

---

## Overview

React 19 function components, `"use client"` where interactivity is needed. Props are explicit TypeScript interfaces. Styling is global CSS with design tokens (`--bg`, `--bg-panel`, `--accent`, ...) from `app/globals.css`, re-tokened in `app/native-theme.css`. There is no CSS-in-JS, no component-level CSS files.

## Component Structure

```tsx
// components/AppSelect.tsx (real pattern)
"use client";
import { useId, type ComponentPropsWithoutRef } from "react";
import * as Select from "@radix-ui/react-select";

export interface AppSelectOption { value: string; label: string; disabled?: boolean; }

type AppSelectProps = Omit<ComponentPropsWithoutRef<typeof Select.Trigger>, "children" | "value" | "onChange"> & {
  value: string;
  onValueChange: (value: string) => void;
  options: readonly AppSelectOption[];
  "aria-label": string;
};

export function AppSelect({ value, onValueChange, options, ...props }: AppSelectProps) { /* ... */ }
```

Order: imports → exported prop interfaces → component. **The primary component matches the filename** (verified across all of `components/*.tsx`), but colocated helper exports are common and fine — real examples: `ChatInput.tsx` exports 20 symbols, `SettingsUi.tsx` 34, `BranchNavigator.tsx` 5 (helpers like `buildActivePath`, `compressChain` beside the main component). Do not invent a `helpers.ts` split for exported symbols that already live beside their component.

## Props Conventions

- Explicit typed interfaces; use `Omit<ComponentPropsWithoutRef<...>, ...>` to narrow third-party props instead of `any` spreads.
- Callbacks are `onXxx` handlers; controlled state is passed in, not hidden inside.
- Shared types come from `lib/types.ts`, never redefined locally.

## Styling Patterns

- Class names on global tokens (`.app-menu`, `.app-select-trigger`, ...). Every dropdown/popup/autocomplete must use the shared `.app-menu` surface — that is a hard project rule (`docs/ui-menu-audit.md`).
- Form choices use `components/AppSelect.tsx`. Never `<select>`, never OS-native option menus.
- Fork CSS lives only in `app/native-theme.css`. `app/globals.css` and `app/settings.css` stay byte-identical to upstream.

## Accessibility

- `aria-label` is required on triggers (see `AppSelectProps` above); decorative SVGs get `aria-hidden="true"`.
- Follow the existing focus treatment in both themes when adding interactive elements — test at desktop and narrow widths.

## Common Mistakes

- Inlining a popup with one-off styles instead of the `.app-menu` surface — `scripts/ui-menu-consistency.test.mjs` will call it out.
- Letting a merged-in upstream component reappear after a merge (upstream owns UI changes; declined upstream UI stays declined).
