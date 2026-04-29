---
name: gravita-webflow-methodology
description: >
  Gravita's methodology for building production sites in Webflow on top of the v2 design system —
  how classes bind to tokens, how HTML is structured, how custom CSS/JS split into component
  embeds, and how components expose properties. Use whenever building, auditing, or scoping
  Webflow work on the Gravita v2 token system, guiding class-binding/class-stacking decisions,
  structuring sections/grids/flex layouts, deciding between component variants vs property
  switches, or answering "how does Gravita build in Webflow." Triggers on: "Webflow build",
  "class binding", "class stacking", "v2 primitives in Webflow", "Webflow components",
  "Webflow structure", "padding classes", "flex classes", "layout classes", "Webflow custom
  code", "Webflow embeds", "Gravita Webflow", "section pattern", "grid offsets", "Webflow
  properties panel", "component props", "Webflow theme binding". Living document — update as
  decisions land; mark provisional items "DRAFT —".
---

# Gravita Webflow Build Methodology

How Gravita binds the v2 Figma design system to live Webflow markup so that designers and
developers work in lockstep, drift is minimised, and end-users get a small surface area of
predictable building blocks they can stack.

This skill describes **patterns and rules** — not a step-by-step build guide. When applying
these patterns to a real project, always confirm with Benn whether something is being treated as
canonical or as an open question.

---

## Mental Model

The v2 system is consumed in Webflow through **class stacking**, not through a single monolithic
class per element. A user picks one base class (e.g. a flex direction) and extends it with a
small number of additional classes (e.g. a gap size, an alignment override). Three classes is the
target ceiling. Custom CSS is reserved for things Webflow's UI cannot express.

Every visual decision should resolve to a **v2 token** — colour, type, spacing, radius. Hard
values in the canvas are a smell.

---

## Class Binding to v2 Tokens

### Naming convention

Class names follow `{category}-{value}`, where `value` matches the variable name in v2-Primitives
exactly. The category prefix is required because the same numeric value appears across ramps.

```
Old (deprecate)        New
──────────────         ──────────────
primary-dark           primary-950
darkest                greyscale-950
mono-dark              mono-black
notification-error     error-500   (or error-{step})
border-darkest         (TBD — see "Borders" below)
```

When auditing existing Webflow classes against v2:
1. List every class on the design system page.
2. For each, identify which v2 variable it should bind to.
3. Rename the class to `{category}-{step}` to match.
4. Re-bind the colour/value to the v2 variable in the Style panel (Webflow → variable picker).

### Binding mechanics for colour classes

A colour class like `primary-950` is created on a `div`. In the Style panel, the **background
colour** is bound to the `primary/950` variable in v2-Primitives. This div becomes the swatch.

When the same class is later applied to a heading or text, the background-colour binding is
**not** what we want — we want it to control text colour. The pattern:

1. On the colour class itself, set background to **transparent** (use the `transparent`
   primitive).
2. Apply the colour as a text-colour override on the relevant tag/class (e.g. `H1 → primary-950`
   binds text colour to the same v2 variable).

This way `primary-950` is reusable as both a swatch (when applied to a div with explicit bg) and
a text-colour modifier (when stacked on a heading).

### Borders — open question

Per-direction border classes (`darkest-border`, `lightest-border-top`, etc.) blow out the class
count and don't tell us which sides are bordered. **DRAFT —** prefer a smaller set of border
classes that bind only colour, and let direction be set on the element itself or via attribute
selectors. Confirm with Benn before deciding.

---

## Typography

### Per-tag binding

For each HTML tag that ships globally (H1–H6, body, paragraph, label, meta), select **All H{n}
Headings** in the Style selector and bind each axis to a v2 variable:

| Axis        | Bound to                                                  |
| ----------- | --------------------------------------------------------- |
| Font family | `v2-Primitives → font-family → heading` (or `body`, etc.) |
| Font weight | `v2-Primitives → font-weight → {weight}`                  |
| Font size   | `v2-Primitives → typography-size → {step}`                |
| Line height | `v2-Primitives → line-height → {step}`                    |

Sizes use `clamp()` in the published CSS for fluid typography. Verify the published output
contains `clamp(var(...), Xvw, var(...))` for the responsive type ramp.

### Override classes for header semantics

`H1` usually means "primary header" but designers need flexibility — sometimes an H2 needs to
look like a primary header, sometimes an H1 needs to look like a subheading. Solve this with two
override classes:

- `heading` — binds to the heading typeface stack
- `subheading` — binds to the subheading typeface stack

Stack with size overrides (`xl`, `l`, `ml`, `m`, `s`, `xs`) and colour modifiers (`primary-950`,
etc.) to compose any visual style without violating semantic HTML.

> **Naming gotcha (from the transcript):** Webflow may already reserve `heading` as a class
> name. If so, find the existing usage (Style Selectors → search `heading`), reassign or rename,
> then claim the name for the override.

### Typography theme binding

Headings should ultimately tie to the **theme** (so changing the section theme cascades to
heading colour automatically), but stacked classes override the theme. So the cascade is:

```
theme.heading.color  →  H1 default colour
  └─ overridden by  →  primary-950 (or any colour modifier class)
```

---

## Properties: Padding, Margins, Sizing

### Why directional padding classes exist

Standard Webflow practice is to set padding via the spacing widget. Gravita instead exposes
**directional padding classes** (`padding-left-right-top`, `padding-left-right-bottom`,
`padding-top`, `padding-bottom`, `padding-full`, etc.) sized by t-shirt suffix
(`-xs`/`-s`/`-m`/`-l`/`-xl`).

Reason: when two sections sit next to each other, identical full-padding causes a **double-stack
gutter**. The user needs to be able to say "this section pads top + sides only" so the next
section's top padding owns the seam. We can't use flex `gap` between sections because sections
often have full-bleed background colours — `gap` would leave white slivers.

### Composition

```
.padding-left-right-top.s    → padding: 0  S  0  S  (sides equal, top S, bottom 0)
.padding-full.l              → padding: L on all sides
.padding-bottom.xs           → padding-bottom: XS only
```

The `.s`, `.l`, etc. modifier maps to the matching `v2-Semantic Tokens → padding-{step}` value
across responsive modes.

### Margins, viewports, percentages

Same composition pattern applies. `margin-top.l`, `viewport-height.full`, etc. Custom values
should be exceptional and live in the per-page custom CSS embed, not in inline styles.

---

## Layout (Flex / Grid)

### Flex naming

Current convention:

```
flex-h                  → row, default alignment
flex-h-wrap             → row, wraps
flex-h-center-left      → row, vertically centred, horizontally start
flex-v                  → column
flex-v-stretch          → column, items stretch
flex-v-top-left         → column, top-left aligned
```

Gap is always available as a follow-on class (`flex-v-stretch.gap-m`, etc.).

**DRAFT —** consider renaming the family to `layout-h-*` / `layout-v-*`. "Flex" is technically
correct but assumes the user understands CSS flexbox; "layout" reads better to non-engineers.
Decision still open — don't rename in production builds without Benn's sign-off.

### Grid

- Use grid for **page-level columns**, not inside components.
- Pre-built classes: `2-column-grid-auto`, `3-column-grid-auto`, etc., extensible with
  `gap-{size}`.
- **Webflow's Grid UI cannot bind variables to track sizes** (`grid-template-columns` and
  similar). Variables are accepted on `gap`, `padding`, etc., but `1fr 0.75fr`-style track
  definitions must be expressed as raw values or as `var()` strings via custom CSS.

Inside components, prefer flex.

### Grid offset values

Per the v3 architecture decision, grid offset amounts (the unitless `fr` ratios used in
asymmetric splits like `1fr 0.75fr`) are stored as **direct numeric values in v2-Component
Tokens** — they are *not* aliased to a v2-Primitives ramp. Reasoning: a unitless fr-ratio is a
one-purpose token type that doesn't fit the spatial / sizing / typography ramps in v2-Primitives,
so promoting it to Primitives would create an orphan ramp.

Apply offsets through a custom CSS embed that references the tokens by name:

```css
/* In a custom CSS embed */
.grid-offset-left.tight     { grid-template-columns: 1fr var(--_v2-component-tokens---grid--tight--offset-amount); }
.grid-offset-left.cozy      { grid-template-columns: 1fr var(--_v2-component-tokens---grid--cozy--offset-amount); }
.grid-offset-left.spacious  { grid-template-columns: 1fr var(--_v2-component-tokens---grid--spacious--offset-amount); }
/* mirror with the offset-amount on the left track for offset-right variants */
```

Direction (left vs right — which track is the smaller fraction) is a class variant, not a
token. Keep the offset modifier classes co-located in one custom CSS embed so they ship as a
single unit.

---

## Build Structure (HTML5)

The body is organised in this order. Items marked **(component)** are Webflow components used
purely as containers for code embeds, with a Visibility prop so the user can disable them
per-page.

```
body
├── JS Libraries           (component)  — embeds for third-party JS that must load early
├── Custom CSS             (component)  — embeds split by subject (see "Custom CSS" below)
├── Notification Banner                 — landmark: nav (or banner)
├── Menu                                — landmark: nav
├── main                                — <main> tag
│   └── section …                       — <section>, rarely styled, holds anchor IDs
│       └── div                         — column / max-width / padding / flex live here
├── footer                              — outside <main>, its own landmark
├── modal-overlay                       — outside <main>, must contain its own landmark + ARIA
└── Custom JS              (component)  — embeds split per behaviour (accordion, modal, etc.)
```

### Sections

- **Rarely styled.** A section's job is to mark subject matter for the page outline.
- IDs for in-page anchors live on the section. When designing anchors, account for offsets
  caused by sticky/fixed/absolute navigation.
- Background colours and theming go on a child div, not on the section itself, unless the design
  truly demands a full-bleed coloured section.

### Max-width container

When a section needs a max-width:

- The **section** gets the padding (left/right).
- The **max-width container** is a child of the section.
- Do **not** put padding on the max-width container itself or on its children's outer wrappers
  — let the section's padding define the inset.

If the design has no max-width, skip this pattern entirely and go straight to a grid layout
inside the section.

### Footer and modals outside `main`

`<footer>` and modal overlays are their own landmarks. They live as siblings of `<main>`, not
inside it. Modals must include the correct ARIA roles and labelling.

---

## Custom CSS

### Where it lives

A single Webflow **component** named `Custom CSS` (or similar) contains multiple **separate
embeds**, one per subject. Splitting by subject keeps each embed small and lets the user toggle
visibility per page if a behaviour isn't needed.

Typical embed split:

```
Custom CSS (component)
├── global.css        — resets, root vars, body defaults
├── utilities.css     — helper attribute selectors, fluid clamps, fallbacks
├── sidebar.css       — sidebar-specific overrides
├── accordion.css     — accordion behaviour styling
├── marquees.css      — marquee animations
└── components.css    — per-component overrides (icon size, image aspect ratios, tabs layout)
```

### What goes in custom CSS vs the Webflow UI

Custom CSS is for things the Webflow UI **cannot** express:

- **Pseudo-classes Webflow doesn't expose:** `:disabled`, `:has()`, `:not()`, `:focus-visible`,
  `::before`, `::after`. (`:hover` is in the UI; most others aren't.)
- **Attribute-selector logic** driven by component props (see "Components" below).
- **Modern CSS features** Webflow doesn't surface: container queries, `clamp()`, conditional
  selectors, custom property fallbacks, `:has()`-based parent styling.

Always reference v2 tokens via `var(--_component-tokens---icon---size--size-s)` etc. — never
hard-code values in custom CSS.

### Why CSS lives inside a component

Webflow's Visibility prop controls whether a component's children are **published to the DOM**.
Wrapping CSS embeds in a component means a per-page disable of "Sidebar CSS" actually removes it
from the published page, not just hides it.

> **Critical scoping rule:** Visibility removes from the DOM **only** for elements **inside**
> components. Hiding an element outside a component still ships it to the DOM. This is why all
> CSS/JS embeds live inside components.

---

## Custom JS

Same pattern as Custom CSS:

- A `Custom JS` component contains a sub-component `Activate` with one embed per behaviour
  (Modal Controls, Accordion JS, Slider, Copy Code, Marquee, Tabs, Video Player, Video Element,
  Site Search Results, …).
- Each embed has its own Visibility prop. Disabling `Accordion JS` on a page removes its code
  from the published HTML.
- Behaviours are wired up via `data-*` attributes on elements — never inline `onclick` etc.

---

## Components

### Build philosophy

> **More components > one component with many props.**

Users navigate a long props list more slowly than they navigate a list of clearly-named
component variants. Lean toward separate components (`Button Primary`, `Button Secondary`,
`Button Icon`) rather than a single `Button` with a 6-deep variant tree.

That said, when variants are genuinely combinatorial (size × tone × icon position), nested
components are unavoidable until Webflow improves single-level prop expressiveness.

### Two ways to expose options

**1. Component variants (modes)**
Used when the choice is qualitative: alignment (left/centre/right), tone (base/light/dark),
purpose (primary/secondary). The user picks from a dropdown.

Pattern: a parent component contains nested components, each of which has a Mode/Variant
property. Deeply nested (e.g. `Heading → Alignment Variant → Tone Variant`) so the top-level
props panel stays simple.

**2. Properties panel (number / switch)**
Used when the choice is quantitative or binary. The Properties panel (left side of the component
editor) supports:

- **Number** — for things like icon size, font weight, padding step where a numeric scale is
  intuitive. Wired by attaching the prop value as a `data-*` attribute on the target element,
  then writing CSS like:

  ```css
  [data-icon-size="0"] { font-size: var(--_component-tokens---icon---size--size-s); }
  [data-icon-size="1"] { font-size: var(--_component-tokens---icon---size--size-m); }
  [data-icon-size="2"] { font-size: var(--_component-tokens---icon---size--size-l); }
  ```

- **Switch** — for binary props (icon visible/hidden, full-width on/off).

- **Text** — **avoid.** The user has no way to know what string is valid. Use variants instead.

### Theme binding inside components

Components should bind colour/type/spacing to the theme by default so they react to a parent
section's theme switch. Stacked override classes still take precedence — that's the whole point
of class stacking.

### Build sanity checks

Before publishing a component:
- Does it inherit the section theme?
- Can it be overridden by stacking a colour/size class on its wrapper?
- Are all values bound to v2 tokens (no hard-coded hex/px)?
- Are pseudo-class behaviours wired through the per-component custom CSS embed?
- If it has props, are they Number/Switch (not Text)?

---

## Webflow API Binding Gotchas

When binding via the Webflow MCP API (`style_tool`, `variable_tool`, `whtml_builder`) — as
opposed to manually in the Designer — there are several behaviours that aren't documented and
have to be learned by hitting them. The patterns below were captured through trial and error.

### `gap` shorthand rejects Size variables

`style_tool.update_style` rejects Size variables on the `gap` shorthand with:

> Property gap does not support setting a variable of type length

The fix is to bind the more specific properties instead:

| ❌ Won't pill          | ✅ Will pill                         |
| --------------------- | ------------------------------------ |
| `gap`                 | `grid-column-gap`, `grid-row-gap`    |

Always bind both `grid-column-gap` and `grid-row-gap` to the same gap variable. Webflow displays
them as a single gap pill in the Designer UI even though they're stored separately.

### Style→variable binding via API works (don't conflate with cross-collection alias creation)

The known constraint about cross-collection variable references requiring `custom_value` strings
applies **only to variable CREATION** (creating a variable in one collection that aliases a
variable in another). It does **not** apply to **style binding**, where `update_style` with
`variable_as_value` accepts variable IDs directly and produces proper pill bindings — including
when those variables ultimately resolve through cross-collection aliases.

In practice: when adding a new component class, bind its CSS properties to v2 tokens via the API
rather than queuing manual binding work in the Designer. Saves a context switch and keeps the
audit trail in tool calls.

### `update_style` partial-success behaviour

When `update_style` receives multiple properties and one errors, the other properties may still
have applied. The error response says `data: null` which suggests atomic rejection, but in
practice the valid properties land in the style anyway. **Always re-query the style after an
error** rather than trusting the response payload — you may need to clean up partials or skip
already-applied properties on retry.

### HTML import via `whtml_builder` auto-creates new classes

`whtml_builder` with HTML referencing class names that don't yet exist will create those classes
as empty styles, ready to receive bindings. This is a useful scaffolding pattern: import HTML
with new class chains, then bind via `update_style` in a follow-up call.

Side effect: classes imported alongside other classes get marked `isComboClass: true`
automatically by Webflow, even when the class is also usable standalone.

### `border-radius` shorthand auto-expands

When binding `border-radius` via `update_style`, Webflow stores it as both the shorthand AND
all four corner properties (`border-top-left-radius`, etc.). All five entries point to the same
variable. This is harmless but accounts for "duplicate"-looking radius entries in audit output.

The same auto-expansion does *not* uniformly apply to other shorthands — `padding` shorthand is
not auto-expanded the same way, and `gap` shorthand is rejected outright (see above). Bind the
shorthand when it works and accept the duplication; reach for the explicit longhand only when
the API forces it.

---

## Anti-Patterns

Things that look reasonable but break the system:

- ❌ **Padding on the max-width container.** Padding belongs on the section.
- ❌ **Flex `gap` between full-bleed sections.** Use directional padding classes instead.
- ❌ **Hard-coded hex/px in the Style panel.** Bind to a v2 token; if the token doesn't exist,
  add it to v2 first.
- ❌ **One mega-component with 12 props.** Split into separate components.
- ❌ **Text props on components.** Switch to variants or number props.
- ❌ **CSS/JS embeds outside a component.** They can't be visibility-toggled per page.
- ❌ **Class names that don't include the category prefix** (`950` instead of `primary-950`).
- ❌ **Per-direction colour border classes.** Too combinatorial — see "Borders" open question.
- ❌ **Density propagation from layout to components.** Density is per-component and explicit
  (ADR-002). A section's own density class never affects the density of nested components.
  Dropping a Button inside an airy section does not make the Button airy.
- ❌ **`.density-{step}` scope-swap classes** that redefine child component tokens. Not how
  the system works.

## Density Rules (ADR-002)

Density is always set explicitly per component. There is **no inherited density**.

- Every component that has density variants exposes a `density` prop. User picks it per
  instance.
- Component Tokens are structured per-component-per-density:
  `{Component}/{Density}/{Property}`.
- Layout classes like `section-padding-airy` control only the element they're on. They do
  not cascade into nested components.
- Viewport responsiveness is a separate axis — it cascades automatically through Semantic
  Tokens (ADR-001). Density does not.

---

## Status & Open Questions

This skill is a v0 capture from a surface-level walkthrough. Things explicitly left open:

1. Border class strategy — directional vs colour-only.
2. Whether to rename `flex-*` → `layout-*` for user friendliness.
3. Final naming for size modifiers — t-shirt sizes are agreed in principle, but the exact step
   list (`xs`/`s`/`m`/`l`/`xl` vs adding `xxs`/`xxl`) needs confirmation.
4. Grid offset patterns — currently requires custom CSS; revisit if Webflow exposes `minmax()`
   binding.
5. How to handle component-level theme overrides without breaking class-stack precedence.

Update this section as decisions land.

---

## When this skill applies

Read this skill before:
- Naming or renaming Webflow classes that consume v2 tokens.
- Building a Webflow component for the v2 system.
- Structuring a new page or section in a Gravita Webflow project.
- Writing custom CSS or JS for a Gravita Webflow site.
- Answering questions about how Gravita builds in Webflow.
- Auditing an existing Gravita Webflow site for v2 conformance.

Pair with:
- `figma-variable-architecture` — the upstream token system this consumes.
- `figma-component-parity` — how the Figma side mirrors the Webflow component tree.
- `design-token-pipeline` — how variables flow from Figma to CSS custom properties.
- `gravita-company` — broader context on Gravita's process and voice.
