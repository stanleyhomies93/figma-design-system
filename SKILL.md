---
name: figma-design-system
description: 'Design UI components in Figma via the Figma MCP server with strict design-system discipline — auto-layout everywhere, frames over shapes, design tokens bound to every color/typography value, 4/8pt spacing grid, and atomic structure (atoms → molecules → organisms → templates). Use when the user asks to "design in Figma", "create a Figma component", "build the Figma file", "mirror the repo to Figma", or any task producing Figma frames/components — especially when a frontend repo is the source of truth. Also use when the user mentions auto-layout, hug/fill, design tokens, Figma variables, or atomic design. Use even without explicit "design system" mention — any Figma component creation benefits from these rules. Do NOT use for: code generation from existing Figma files, Figma plugin development, or pure design critique without component creation.'
---

# Figma Design System

Build Figma components with strict design-system discipline. When a frontend repo is provided, it is the source of truth — Figma mirrors the repo's tokens, structure, and behavior. When no repo is provided, the same rules still apply: auto-layout everywhere, frames over shapes, every color and typography value bound to a variable or text style, spacing on a 4/8pt grid, and components built atomically.

## Operating principles

Read these before doing anything. They override defaults.

1. **The repo is truth.** Tokens, spacing, typography, and colors are extracted from the repo (Tailwind config, CSS variables, theme files, styled-components themes, etc.). Figma mirrors them. If the repo hardcodes a value, Figma uses that hardcoded value too — do not invent token names for things the repo itself didn't tokenize.

2. **Check Figma before creating.** Every variable, component, or style should be looked up in Figma first via the MCP server. Reuse what exists. Only create when missing. Never duplicate.

3. **Auto-layout by default.** Every frame that contains more than one child, or that needs to respond to content, uses auto-layout. Static absolute positioning is a last resort and only inside leaf-level decorative frames.

4. **No raw values for color or typography.** Every fill, stroke, and text style binds to a Figma variable or text style. If a binding doesn't exist yet, create the variable from the repo's token, then bind. Hardcoded color hex or font sizes on a node are a failure — except when the repo itself is hardcoding them (rule 1).

5. **Spacing follows a 4/8pt grid.** Padding and gap values must be multiples of 4 (preferably 8). If the repo's value isn't on the grid, surface this to the user before proceeding — it usually indicates a bug or a non-system value.

6. **Build atomically, in order.** Atoms → molecules → organisms → templates. Never start at the molecule level if the atoms don't exist yet. This is non-negotiable because skipping levels produces components that can't be reused.

7. **Frames, not shapes.** Use frames for almost everything — including things that look like simple rectangles or circles (avatars, badges, icon backgrounds, dividers, color swatches, container surfaces). Shapes (Rectangle, Ellipse, Line) cannot hold auto-layout, cannot have child nodes, and cannot expose component properties. Reserve shapes for vector primitives inside icon artwork or purely decorative paths. See `references/auto-layout-rules.md` for the full rule.

## Workflow

### Step 1: Read the repo's design tokens

Before touching Figma, locate and read the source-of-truth files. Look in this order:

1. `tailwind.config.{js,ts,mjs,cjs}` — `theme.extend` and `theme` for colors, spacing, fontSize, fontFamily, fontWeight, borderRadius, boxShadow.
2. CSS variables — `:root` blocks in `globals.css`, `tokens.css`, `theme.css`, or similar. Also check for `[data-theme]` blocks for multi-theme support.
3. A dedicated tokens file — `tokens.json`, `design-tokens.{js,ts}`, `theme.{js,ts}`.
4. Styled-components / Emotion theme objects.
5. CSS Modules with `@value` declarations.

Extract a flat list of tokens grouped by category:
- `color/*` (foreground, background, border, semantic — e.g. `color/bg/primary`, `color/text/muted`)
- `spacing/*` (in px or rem; convert rem to px assuming 16px base unless the repo overrides)
- `typography/*` (font family, size, weight, line-height, letter-spacing — bundle into text styles)
- `radius/*`
- `shadow/*`
- `border/*` (widths)

If the repo uses semantic naming (`primary`, `surface`, `muted`), preserve it. If it uses scale-based naming (`blue-500`, `gray-100`), preserve that too. **Do not rename tokens** — Figma variable names should match the repo's token names exactly, with `/` as the hierarchy separator (Figma's convention).

### Step 2: Read the component's source code

Find the component file the user wants to reverse-design. Read the JSX/template, the styles, and any child components it imports. Note:

- What atoms it depends on (Button, Input, Icon, etc.) — these must exist in Figma first.
- Layout structure: flex direction, gap, padding, alignment.
- States: default, hover, focus, disabled, loading, error. Each state typically becomes a component variant.
- Responsive behavior: which dimensions are fixed, which fill, which hug.
- Conditional rendering: optional slots, icon presence, etc. — these become boolean component properties.

### Step 3: Audit Figma for existing variables and components

Use the Figma MCP server to query the current file:

- List existing variables in each collection. Match them against the tokens extracted in Step 1.
- List existing components by name. Match against the atoms identified in Step 2.

Produce a delta: what tokens need to be created, what components already exist (and at what node IDs), what's missing.

Show this delta to the user before creating anything. They may want to redirect (e.g., "use the existing `color/primary` instead of creating `color/brand/500`").

### Step 4: Create missing variables

For each missing token, create the corresponding Figma variable in the appropriate collection. See `references/figma-variables.md` for collection structure, modes (light/dark), and naming conventions.

Group variables by category into collections:
- `Primitives` — raw values (`color/blue/500`, `spacing/4`)
- `Semantic` — references to primitives (`color/bg/primary` → `color/blue/500`)
- `Component` — component-specific overrides only if the repo has them

If the repo only has one layer (e.g., just `primary: '#3B82F6'` with no underlying scale), create only the Semantic collection. Don't invent a primitive layer.

### Step 5: Build the component, atomically

Work bottom-up. For each level, follow the rules in `references/auto-layout-rules.md`.

**Atoms** (Button, Input, Icon, Text, Badge, Avatar):
- Create as Figma components with variants for each state.
- Bind every fill, stroke, and text property to a variable or text style.
- Set auto-layout: padding, gap, alignment matching the repo.
- Set hug/fill/fixed on width and height — see decision tree in references.
- Add component properties for things the repo exposes as props: `variant`, `size`, `disabled`, `iconLeft`, `iconRight`, etc.

**Molecules** (FormField, SearchBar, Card header):
- Compose from atom instances. Never re-draw an atom inside a molecule.
- Auto-layout wraps the atom instances.
- Expose nested instance swaps for slots (e.g., the icon inside an input).

**Organisms** (Card, Navigation, DataTable row):
- Compose from molecules and atoms.
- This is usually where responsive behavior lives — set fill-container on the right children.

**Templates** (full page or section layout):
- Compose from organisms.
- Use frame-level auto-layout with appropriate fill/hug to match the repo's layout (CSS grid, flex container, etc.).

### Step 6: Verify

After building, do these checks before declaring done:

- [ ] No node has a hardcoded color, stroke, or font property unless the repo also hardcodes it.
- [ ] All padding and gap values are multiples of 4.
- [ ] Every multi-child frame uses auto-layout.
- [ ] Hug/fill settings match the repo's layout intent (see references).
- [ ] No raw shapes (Rectangle, Ellipse, Line) are used as containers, surfaces, dividers, or anywhere a frame should be. Shapes appear only inside icon vector artwork.
- [ ] Component variants match the states present in the repo's code.
- [ ] Component properties match the props exposed by the repo's component.
- [ ] No duplicate variables or components were created — the audit from Step 3 was respected.

Report any check that failed to the user with the specific node and reason.

## When to stop and ask

Pause and ask the user when:

- A repo value can't be cleanly mapped to a token (e.g., a one-off `padding: 13px` that's neither a token nor on the 4pt grid).
- The repo uses CSS features that don't translate to Figma (e.g., `clamp()`, container queries, `:has()` selectors, complex pseudo-elements). Describe what you see and propose the closest Figma equivalent.
- An atom dependency is missing and the user didn't ask for it to be built first. Don't silently build extra components.
- The repo and an existing Figma variable disagree (e.g., repo says `#3B82F6`, Figma's `color/primary` is `#2563EB`). Surface the conflict; let the user choose.

## Reference files

- `references/auto-layout-rules.md` — Hug/Fill/Fixed decision tree, padding patterns, alignment, wrap behavior. **Read this before creating any frame.**
- `references/figma-variables.md` — Variable collection structure, modes, naming conventions, binding patterns.
- `references/token-extraction.md` — Patterns for extracting tokens from Tailwind, CSS variables, and JS/TS theme files. Read when Step 1 is non-obvious.
- `references/atomic-checklist.md` — Per-level checklist (atom, molecule, organism, template) of what must be true before moving up a level.
