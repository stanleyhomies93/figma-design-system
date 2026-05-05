# Token Extraction

Patterns for pulling design tokens out of common repo configurations. Read this when Step 1 of the workflow is non-obvious — e.g., the repo uses a non-standard setup.

## Tailwind config

The most common case. Look in `tailwind.config.{js,ts,mjs,cjs}`.

```js
// tailwind.config.ts
export default {
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#EFF6FF',
          500: '#3B82F6',
          900: '#1E3A8A',
        },
        surface: 'var(--color-surface)', // CSS variable reference!
      },
      spacing: {
        '4.5': '18px', // non-default values to capture
      },
      fontSize: {
        'xs': ['12px', { lineHeight: '16px' }],
      },
      borderRadius: {
        'card': '12px',
      },
    },
  },
}
```

Extraction notes:
- `theme.extend.*` adds to defaults. `theme.*` (without extend) replaces defaults — be careful which one is in use.
- Default Tailwind spacing scale is implicit: `0, px, 0.5, 1, 1.5, 2, 2.5, 3, 3.5, 4, 5, 6, 7, 8, 9, 10, 11, 12, 14, 16, 20, 24, 28, 32, 36, 40, 44, 48, 52, 56, 60, 64, 72, 80, 96`. If the component uses `p-6`, it's 24px even if the config doesn't list it.
- When a value references a CSS variable (`'var(--color-surface)'`), the actual value is in CSS — chase it down.
- `fontSize` entries can be tuples: `[size, { lineHeight, letterSpacing }]`. Bundle these into text styles.

## CSS variables

Look for `:root` blocks in `globals.css`, `index.css`, or theme files. Also `[data-theme]` for multi-theme.

```css
:root {
  --color-bg-primary: #ffffff;
  --color-text-default: #111827;
  --spacing-card: 24px;
  --radius-md: 8px;
  --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
}

[data-theme="dark"] {
  --color-bg-primary: #0a0a0a;
  --color-text-default: #f5f5f5;
}
```

Extraction notes:
- Variable name → Figma variable name with `/` replacing `-` segments where it makes sense (`--color-bg-primary` → `color/bg/primary`).
- Each `[data-theme]` block is a Figma mode. Variables present in both blocks get values per mode. Variables only in `:root` are mode-independent.
- Watch for variables defined inside component selectors (`.button { --button-bg: ... }`) — these are component-scoped tokens, not global. Treat them as Component collection candidates only if they're truly reusable.

## CSS variables referenced by Tailwind

Modern Tailwind setups often use this hybrid:

```css
:root {
  --color-primary: 59 130 246; /* RGB triplet for opacity support */
}
```
```js
// tailwind.config.ts
colors: {
  primary: 'rgb(var(--color-primary) / <alpha-value>)',
}
```

The actual value lives in CSS. The Tailwind config just exposes it. Extract from CSS, not from the config.

## JS/TS theme objects

```ts
// theme.ts
export const theme = {
  colors: {
    bg: { primary: '#fff', surface: '#f5f5f5' },
    text: { default: '#111', muted: '#666' },
  },
  space: [0, 4, 8, 12, 16, 24, 32, 48, 64], // array — index is the token name
  radii: { sm: 4, md: 8, lg: 12 },
};
```

Extraction notes:
- Array-based scales (common in Theme UI / styled-system) use the array index as the token name. `space[3]` = 12px, named `space/3`.
- Nested objects flatten with `/`: `colors.bg.primary` → `color/bg/primary`.

## When the repo has no token system

Some repos just hardcode values everywhere. In that case:

- Scan the component being designed for the literal values it uses.
- Don't create a token system speculatively. Apply the hardcoded values directly to Figma nodes.
- Mention to the user that the repo has no tokens, and offer to help establish one as a separate task — but don't do it as part of the reverse-design.

## Edge cases

- **`clamp()` for fluid type**: `font-size: clamp(1rem, 2vw, 1.5rem)`. Figma can't replicate this. Use the middle value as the Figma text style size and note the clamp in the component description.
- **Color functions**: `color-mix()`, `oklch()`, `hsl()` with calculations. Resolve to a static hex/RGB at the time of extraction.
- **Conditional tokens**: `--color-bg: light-dark(#fff, #000)`. Treat as a two-mode variable.
- **Nested CSS variables**: `--btn-bg: var(--color-primary)`. Resolve the chain to the leaf value, then create the variable as a reference (alias) in Figma where supported.
