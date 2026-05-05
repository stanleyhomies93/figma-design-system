# Atomic Design Checklist

Per-level checklist. Each level must pass before building anything at the level above.

## Atom

The smallest meaningful UI unit. Examples: Button, Input, Icon, Text, Badge, Avatar, Checkbox, Radio, Switch, Tag, Spinner, Divider.

Before declaring an atom done:

- [ ] It is a Figma component (not just a frame).
- [ ] It is built from frames, not shapes. Rectangles and Ellipses appear only inside icon vector artwork, never as containers, surfaces, badges, avatars, or dividers.
- [ ] It has variants for every state present in the repo's code: default, hover, focus, active, disabled, loading, error, etc.
- [ ] Every fill, stroke, and text property is bound to a variable or text style. No raw colors. No raw font sizes.
- [ ] Auto-layout is set, with padding and gap matching the repo.
- [ ] Width and height settings (hug/fill/fixed) match the repo's CSS intent.
- [ ] Component properties are exposed for every prop the repo's component accepts: `variant`, `size`, `disabled`, `iconLeft`, `iconRight`, `label`, etc.
- [ ] Boolean properties for optional slots (e.g., `hasIcon`).
- [ ] Instance swap properties for slot contents (e.g., the icon in a Button).
- [ ] Component description includes the source path of the repo file it mirrors.

## Molecule

A small group of atoms working together. Examples: FormField (Label + Input + HelperText), SearchBar (Input + IconButton), Card header (Avatar + Text stack + IconButton).

Before declaring a molecule done:

- [ ] All required atoms exist as components first.
- [ ] The molecule is a Figma component composed of atom **instances** — not redrawn copies.
- [ ] Auto-layout wraps the atom instances; padding and gap match the repo.
- [ ] No atom is detached or overridden in a way that should have been a new variant on the atom itself.
- [ ] Slot contents that vary in usage are exposed via instance swap properties.
- [ ] Variants exist for any molecule-level states (e.g., FormField error vs default — even though the Input atom has its own error state, the molecule wraps the whole error presentation including the helper text color).
- [ ] All atom instances within still bind to variables — no overrides that introduce raw values.

## Organism

A self-contained section of a UI. Examples: Card, NavigationBar, DataTable row, ModalDialog, Toast, FilterPanel.

Before declaring an organism done:

- [ ] All molecules and atoms it depends on exist.
- [ ] Composed of molecule and atom instances; nothing redrawn.
- [ ] Responsive behavior matches the repo: which children fill, which hug, which are fixed.
- [ ] If the organism has slots (e.g., Card's header / body / footer), each slot is an exposed property or an instance swap.
- [ ] Variants for organism-level state if applicable (e.g., Card collapsed vs expanded).
- [ ] Min/max width or height settings match the repo's CSS.
- [ ] If the repo's organism has internal scrolling, set the equivalent in Figma (clip content + the appropriate fill behavior).

## Template

A page or large section layout. Examples: DashboardPage, SettingsLayout, ProductDetailPage.

Before declaring a template done:

- [ ] All organisms it composes exist.
- [ ] The template is a Figma frame (or component if it's reused) with auto-layout.
- [ ] Layout direction, gap, padding, and alignment match the repo's outermost container.
- [ ] Children's fill/hug/fixed settings produce the right responsive behavior at the chosen frame width.
- [ ] If the repo uses CSS Grid for the template, document the grid structure in the frame description and approximate with nested auto-layout. Surface the approximation to the user.
- [ ] Any responsive variants (mobile, tablet, desktop) are separate frames or component variants, named with their breakpoint.

## Stop conditions

At any level, stop and ask the user if:

- A required lower-level component is missing and the user didn't request it.
- The repo's structure is ambiguous (e.g., a component that could be classified at two levels).
- A repo prop doesn't have a clean Figma equivalent (e.g., a render prop, a polymorphic `as` prop). Describe the situation and propose an approach.
