# Front-end scaffold reference

Read this only at step 6 of the setup sequence. This is the house front-end
architecture: Vite + CUBE CSS + Every Layout + Utopia + OKLCH semantic tokens.
Lay down the structure, not a finished design.

## Directory shape

```
<slug>/
├── package.json
├── vite.config.js
├── src/
│   ├── js/
│   │   └── app.js              # entry; keep JS minimal
│   └── css/
│       ├── app.css             # imports the layers in order
│       ├── reset.css
│       ├── tokens/
│       │   ├── color.css       # OKLCH semantic tokens
│       │   ├── type.css        # Utopia fluid type scale
│       │   └── space.css       # Utopia fluid space scale
│       ├── composition/        # CUBE: layout primitives (Every Layout)
│       ├── utility/            # CUBE: single-purpose utilities
│       ├── blocks/             # CUBE: component-specific
│       └── exceptions/         # CUBE: data-attribute exceptions
└── templates/
    ├── _layouts/
    │   └── base.twig
    ├── _components/
    └── _primitives/            # Every Layout as Twig macros/includes
```

## Vite

Use the `vite` package with Craft's vite plugin (`nystudio107/craft-vite`) OR a
plain manifest setup, depending on the confirmed plugin set. Default to the
nystudio107 plugin if it's in the set.

`vite.config.js` essentials:
- `build.manifest = true`
- `build.outDir = 'web/dist'`
- `build.rollupOptions.input` → `src/js/app.js`
- dev server origin matching the DDEV URL so HMR works inside the container

Don't over-configure. Manifest + input + outDir is enough to start.

## CSS layer order (app.css)

Import in this order so the cascade behaves:

```css
@import "reset.css";
@import "tokens/color.css";
@import "tokens/type.css";
@import "tokens/space.css";
/* composition (layout primitives) */
/* utility */
/* blocks */
/* exceptions */
```

## OKLCH semantic tokens

Two-layer token system. Primitive ramp → semantic role. Roles are what templates
reference; never reference raw ramp values in components.

```css
:root {
  /* primitives — a ramp, not used directly in components */
  --_gray-100: oklch(98% 0.005 270);
  --_gray-900: oklch(20% 0.01 270);
  /* …fill the ramp per project palette… */

  /* semantic roles — reference THESE */
  --color-surface: var(--_gray-100);
  --color-text: var(--_gray-900);
  --color-accent: oklch(60% 0.15 250);  /* set per brand */
}
```

Verify text/surface pairs hit WCAG 2.2 AA (4.5:1 body, 3:1 large). If the build
needs dark mode, add a `:root[data-theme="dark"]` block remapping the semantic
roles, not the primitives.

## Utopia scales

Generate fluid type and space custom properties from utopia.fyi (or hand-roll the
clamp() values). Type scale and space scale each as `--step-*` / `--space-*`
custom properties in their token files. Don't hardcode px in components — reference
the scale.

## Every Layout primitives as Twig

Implement the primitives the project actually needs as Twig includes or macros in
`templates/_primitives/`. Don't scaffold all of them — start with `stack` and
`box`, add `cluster`, `sidebar`, `switcher`, `cover`, `grid`, `frame` as the
design calls for them. Each maps to a composition-layer CSS class.

Example `_primitives/stack.twig`:

```twig
{# {% include '_primitives/stack.twig' with { content: '…', space: 'var(--space-m)' } %} #}
<div class="stack" style="--stack-space: {{ space|default('var(--space-m)') }}">
  {{ content|raw }}
</div>
```

with the matching composition rule:

```css
.stack > * + * { margin-block-start: var(--stack-space, var(--space-m)); }
```

## Base layout

`_layouts/base.twig`: doctype, `<head>` with the Vite tags (via the plugin's
`{{ craft.vite.script('src/js/app.js') }}` or manifest helper), a `{% block %}`
for content, semantic landmarks. Keep it skeletal — it's a starting point, not a
designed page.

## Hard rule

No Tailwind, no utility-class framework, no CSS-in-JS. CUBE's utility layer covers
single-purpose utilities; that's the only "utility" layer in play.
