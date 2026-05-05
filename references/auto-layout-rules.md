# Auto-Layout Rules

Read this before creating any frame in Figma. The hug/fill/fixed decision is where reverse-designed components most often go wrong.

## The decision tree

For every frame, set width and height independently using this tree.

### Width

1. **Does the repo's CSS pin this to a fixed pixel value?** (`width: 240px`, `w-60` in Tailwind)
   → **Fixed width**, value = the repo's value.

2. **Does the repo make it stretch to fill its parent?** (`width: 100%`, `flex: 1`, `flex-grow: 1`, `w-full`, child of a flex container with `flex-1`)
   → **Fill container**.

3. **Does the size depend on the content inside?** (no width set, or `width: max-content`, `width: fit-content`, `w-fit`, `w-max`)
   → **Hug contents**.

4. **Is there a max-width with otherwise fluid behavior?** (`max-width: 1200px; width: 100%`)
   → **Fill container** with a max-width applied via the frame's max-width property (Figma supports this on auto-layout frames).

### Height

Same logic, but the default for most components is **Hug contents** unless the repo sets an explicit height. Most flex column containers in modern web UI hug their content height.

Exception: full-page templates and sticky sidebars — these often `fill` the viewport. Match the repo.

## Padding

- Read padding directly from the repo. Tailwind `p-4` = 16px. Tailwind `px-6 py-4` = horizontal 24, vertical 16.
- Values must be multiples of 4. If the repo has `padding: 13px`, stop and ask the user.
- Asymmetric padding (different left/right or top/bottom) is fine and common — match the repo exactly.
- If the component has different padding per state (e.g., compact variant), encode this via component variants, not by overriding instances.

## Gap

- Gap = the space between children in a flex container. Maps to Tailwind `gap-*`, `space-x-*`, `space-y-*`, or CSS `gap`.
- Same 4/8pt rule as padding.
- If the repo uses margin between siblings instead of gap (older codebases do), translate to gap in Figma — Figma's auto-layout doesn't have a margin concept on children, and gap is the correct equivalent.

## Direction

- `flex-direction: row` → horizontal auto-layout.
- `flex-direction: column` (or default for `<div>` with `display: flex; flex-direction: column`, or `flex-col` in Tailwind) → vertical auto-layout.
- `flex-direction: row-reverse` / `column-reverse` → set Figma's "reverse" toggle, don't manually reorder children.

## Alignment

Map CSS to Figma's 9-point alignment grid:

- `justify-content: flex-start` + `align-items: flex-start` → top-left
- `justify-content: center` + `align-items: center` → center
- `justify-content: space-between` → set "Space between" packing in Figma
- `align-items: stretch` (default for flex) → set children to fill on the cross axis
- `align-items: baseline` → Figma doesn't support baseline alignment. Approximate with center and flag this to the user.

## Wrap

- `flex-wrap: wrap` → enable wrap on the auto-layout frame.
- When wrap is on, gap controls both row and column spacing. If the repo specifies different `row-gap` and `column-gap`, set both in Figma.

## When NOT to use auto-layout

Auto-layout doesn't fit when:

- The element is absolutely positioned over another (e.g., a badge overlapping an avatar's corner). Use a regular frame with the badge as a positioned child.
- The element is purely decorative (a background blob, a divider line). A non-auto-layout frame is fine.
- The repo uses CSS Grid with named areas that don't translate to flex. Document the structure in a frame description and approximate with nested auto-layout, or surface to the user.

## Frames vs shapes

Use a frame for anything that is or could become a container. Shapes (Rectangle, Ellipse, Line, Polygon, Star) are reserved for vector primitives inside icon artwork.

### Why this matters

Shapes cannot:
- Hold auto-layout
- Contain child nodes
- Expose component properties or instance swap slots
- Bind padding or gap to spacing variables
- Be promoted to a component cleanly

A shape used where a frame should be is a near-guaranteed rework when the design needs to evolve.

### Concrete mappings

| What it looks like | What to use |
|---|---|
| Avatar (circular image holder) | Frame, fully rounded corners, image fill or nested image |
| Badge / Pill | Frame with auto-layout, fully rounded corners |
| Icon background / chip | Frame with auto-layout, padding, rounded corners |
| Card / Surface / Panel | Frame with auto-layout |
| Color swatch in a palette | Frame |
| Divider line | Frame with fixed height (or width for vertical), bound stroke or fill |
| Skeleton loader block | Frame |
| Button background | Already a frame — the Button component's root |
| Tag, Status dot | Frame |
| Progress bar track and fill | Frames (parent + child), not rectangles |
| Modal scrim / overlay | Frame with fill |
| Empty state illustration container | Frame |

### When a shape is correct

- Inside an icon component, where the icon's vector geometry uses Rectangle, Ellipse, Path, etc. The icon component itself is a frame; the geometry inside it is shapes.
- Decorative vector artwork that has no structural role (a background blob, a wave, an abstract pattern). Even here, wrap it in a frame so it can be positioned and bound.
- A purely visual line that will never need to be a container — but in practice, dividers should be frames too because they often need spacing variables bound to their height/width.

### Quick test

Before placing a shape, ask: *Could this ever need a child, padding, a variant, or to be swapped as an instance?* If yes — even hypothetically — use a frame. The cost of a frame over a shape is zero. The cost of converting a shape to a frame later is high.

## Common mistakes to avoid

- **Using a Rectangle or Ellipse where a frame belongs** — avatars, badges, dividers, surfaces, swatches. See the Frames vs shapes section above. If it has any structural role, it's a frame.
- **Setting all children to "fill"** — only the children the repo flexes (`flex-1`, `flex-grow`) should fill. Others hug or are fixed.
- **Setting padding on the wrong frame** — padding belongs on the container, not the children. If you find yourself adding a margin-equivalent to a child, the parent's padding is probably wrong.
- **Forgetting min-width: 0** equivalent — when text is inside a fill-container frame and should truncate, the frame needs to allow shrinking. In Figma, this means setting the text node to "Truncate" and the parent to fill.
- **Nesting auto-layouts unnecessarily** — if a frame has only one child, you usually don't need auto-layout on the wrapper. Match the repo's actual DOM nesting, not more, not less.
