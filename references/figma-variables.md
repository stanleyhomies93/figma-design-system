# Figma Variables

How to structure variable collections, modes, and bindings when reverse-designing from a repo.

## Collection structure

Default to up to three collections. Only create collections you actually need based on what's in the repo.

### `Primitives`

Raw values. Use only when the repo has a primitive scale layer.

Examples (only if the repo has them):
- `color/blue/50` through `color/blue/950`
- `color/gray/50` through `color/gray/950`
- `spacing/0`, `spacing/1` (4px), `spacing/2` (8px), … `spacing/16` (64px)
- `radius/none`, `radius/sm`, `radius/md`, `radius/lg`, `radius/full`

If the repo only declares `colors: { primary: '#3B82F6', surface: '#FFFFFF' }` with no scale, **skip the Primitives collection**. Don't invent a scale.

### `Semantic`

References to primitives, named by role. This is where bindings happen for almost all components.

Examples:
- `color/bg/primary` → `color/blue/600`
- `color/bg/surface` → `color/gray/50`
- `color/text/default` → `color/gray/900`
- `color/text/muted` → `color/gray/500`
- `color/border/default` → `color/gray/200`

If the repo's tokens are already semantic (no primitive layer), put them directly in this collection with their literal values.

### `Component`

Component-specific tokens, only if the repo defines them (e.g., `--button-height-sm: 32px`). Don't create these speculatively.

## Modes

Modes map to the repo's themes.

- One theme (just `:root`) → single `Default` mode.
- Light + dark (`:root` + `[data-theme="dark"]`, or `prefers-color-scheme: dark`) → `Light` and `Dark` modes on collections that have theme-dependent values (usually only Semantic).
- Brand themes (`[data-theme="brandA"]`, `[data-theme="brandB"]`) → one mode per brand.

Primitive collections rarely need modes — the raw scale doesn't change. Semantic collections are where modes do their work.

## Naming

- Use `/` as the path separator. Figma renders this as a nested group.
- Match the repo's casing and structure exactly. If the repo says `bgPrimary`, use `bg-primary` or `bg/primary` consistently — pick one and document it. Don't translate `bgPrimary` to `Background Primary` or similar.
- Lowercase, kebab-case for multi-word segments.
- Categories first, specifics last: `color/text/muted`, not `muted/text/color`.

## Binding rules

Every node fill, stroke, text fill, corner radius, padding, and gap that corresponds to a token must be bound to a variable.

Procedure for each property:
1. Identify the token name in the repo.
2. Look up the corresponding variable in Figma (use the MCP server's variable list).
3. If it exists, bind the property.
4. If it doesn't exist, create the variable in the appropriate collection, then bind.

Padding and gap bindings: Figma supports binding spacing values to number variables. Use them. A frame with `padding: var(--spacing-4)` in CSS should have its padding bound to `spacing/4` in Figma.

## Typography

Typography is handled via **text styles**, not raw variables — Figma's text styles bundle font family, weight, size, line-height, and letter-spacing into one applied object.

For each text scale entry in the repo (e.g., Tailwind's `text-sm`, `text-base`, `text-lg`, `text-xl`), create a text style:

- Name: `text/sm`, `text/base`, `text/lg`, `text/xl` (or match the repo's naming).
- Bundle: font-family, font-weight, font-size, line-height, letter-spacing.

When the repo has multiple weights of the same size (`text-sm font-medium` vs `text-sm font-bold`), create separate styles: `text/sm/medium`, `text/sm/bold`. Don't try to encode weight as a separate variable — text styles are atomic.

## Conflicts and updates

If a variable already exists in Figma but its value disagrees with the repo:

- Don't silently overwrite. Surface the conflict.
- Show: variable name, current Figma value, repo's value, where in the repo it's defined.
- Let the user decide: update Figma to match repo (usually correct), update repo to match Figma, or rename one of them.
