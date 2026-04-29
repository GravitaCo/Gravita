---
name: figma-variable-architecture
description: >
  Design system variable/token architecture in Figma. Use this skill whenever building, auditing, 
  or restructuring a design system's token hierarchy in Figma — for any project type (SaaS, websites, 
  apps, dashboards). Triggers on: "design tokens", "variable structure", "token architecture", 
  "design system variables", "Figma variables", "token hierarchy", "create a design system", 
  "set up design tokens", "variable collections", "multi-mode variables", or any request to 
  organize design values into a scalable system. Also trigger when someone asks how to structure 
  colors, spacing, typography, or theming in Figma for code consumption.
---

# Figma Variable Architecture

A methodology for structuring design system variables in Figma as a 4-tier alias chain 
that maps directly to CSS custom properties. This approach works for any component framework 
(Radix, shadcn, MUI, Chakra, custom) and any project type.

## The Core Problem

Most design systems fail at the handoff between design and code because:
- Designers use raw hex values and magic numbers — devs can't tell what's semantic
- Token names don't match between Figma and the codebase
- Responsive scaling requires manual overrides everywhere
- Theme switching (dark/light) is bolted on rather than built in
- Per-component control is lost when everything shares the same spacing scale

## The 4-Tier Variable System

Every design system should have exactly 4 variable collections in Figma, loaded in this order:

```
Tier 1: Primitives     → Raw values (the only place numbers and hex codes live)
Tier 2: Semantic Tokens → Purpose-based aliases to Primitives (responsive modes)
Tier 3: Theme           → Color role aliases to Primitives (dark/light modes)  
Tier 4: Component Tokens → Per-component aliases to Semantic (responsive modes)
```

### Why This Order Matters

Each tier can only reference the tier above it or equal. Never skip tiers, never alias downward.

A component's padding in Figma resolves like this:
```
Button/Spacing/Left-Right/L  (Component Token)
  → Padding/Padding-ML       (Semantic Token, Desktop mode)
    → Size/28                 (Primitive, raw value 28)

Switch to Mobile mode:
Button/Spacing/Left-Right/L  (same Component Token)
  → Padding/Padding-ML       (Semantic Token, Mobile mode — different alias)
    → Size/24                 (Primitive, raw value 24)
```

One variable in Figma, different resolved values per breakpoint. No overrides needed.

---

## Tier 1: Primitives

**Purpose:** Single source of truth for every raw value in the system.
**Modes:** Usually 1 (no reason for primitives to be modal).
**Scoping:** Only Semantic Tokens and Theme should reference these. Components never touch Primitives directly.

### What Goes Here

Read the reference file for detailed variable lists: `references/primitives-guide.md`

**Sizes** — A continuous scale of pixel values used for spacing, typography, and layout.
Don't use a linear scale (4, 8, 12, 16...) — use a scale that matches real UI needs.
A good starting set: 2, 4, 6, 8, 10, 12, 14, 16, 20, 24, 28, 32, 36, 40, 42, 48, 52, 56, 60, 80, 100, 120.

**Colors** — Complete ramps for each color family.
- Neutral scale: 15-18 steps from white to black (more granularity in the dark end for dark-first UIs)
- Accent colors: 8 steps per ramp (100-800)
- Notification: Success, Caution, Error (light + dark variants of each)
- Alpha channels: 10-90% opacity for both black and white overlays

**Typography** — Font families (as STRING type), font weights (as FLOAT: 300, 400, 600, 700, 800, 900).

### Figma Implementation

```
Collection: "Primitives"
Modes: 1 (e.g., "Default" or "Dark" — just one)
Variable types: FLOAT for sizes/weights, COLOR for colors, STRING for font families
Naming: Category/Subcategory/Value (e.g., "Color/Neutral/500", "Size/16", "Font/Family/Body")
```

**Critical:** The Primitives collection name must be spelled correctly. Typos here propagate through every alias.

---

## Tier 2: Semantic Tokens

**Purpose:** Give meaning to primitive values. "Padding-ML" means something; "Size/28" doesn't.
**Modes:** Desktop, Tablet, Mobile (responsive breakpoints).
**Scoping:** Referenced by Component Tokens. Components can also reference these directly for one-off needs.

### What Goes Here

**Typography scales** — Size, Line Height, Letter Spacing, Button-specific sizes. Each as a 9-step named scale (XXL through XXS) that aliases to different Size primitives per mode.

**Spacing** — Padding (9 steps + Null) and Gap/Flex (8 steps + Null). Each step aliases to a Size primitive and can change per breakpoint.

**Radius** — 7 steps (XL through XS + Null). Usually stays the same across breakpoints — physical corners don't scale with screen size.

**Max Width** — Container widths. Set to 0 on Tablet/Mobile for full-width behavior.

**Border** — Stroke widths (S/M/L).

### Responsive Scaling Strategy

The key insight: headings scale down aggressively on smaller screens, body text stays the same.

```
                    Desktop     Tablet      Mobile
Typography/XXL      Size/60     Size/60     Size/48      ← headings shrink
Typography/S        Size/16     Size/16     Size/16      ← body stays
Padding/XXL         Size/80     Size/60     Size/48      ← spacing tightens
Gap/ML              Size/20     Size/20     Size/16      ← gaps tighten
Radius/M            Size/16     Size/16     Size/16      ← corners stay
```

When adding a new mode (Mobile), don't guess values — follow these principles:
1. Headings: reduce by 1-2 size steps
2. Body text: keep or reduce by max 1 step (readability)
3. Padding: reduce proportionally (80→48 at the extreme end)
4. Gaps: tighten by 1 step at most
5. Radius: keep the same (corners don't change with viewport)
6. Max widths: set to 0 (full width)

---

## Tier 3: Theme

**Purpose:** Assign colors to roles. "Panel/Background" means something; "Neutral/1100" doesn't.
**Modes:** Dark, Light (and optionally High Contrast, Brand variants, etc.).
**Scoping:** Referenced by components for all color values.

### What Goes Here

Organize by surface, not by color. A theme variable answers: "What color is the background of a Panel?" — not "Where is Neutral/1100 used?"

**Global** — page-level colors: Background, Body text, Heading, Subheading, Meta, Icon, Link, Link-Hover, Border, Menu-Text, Menu-Text-Active.

**Per-surface groups** — each surface that has its own color identity gets a full group:
- Button/Primary/*, Button/Secondary/* (Background, Hover, Text, Icon, Border, Disabled states)
- Panel/* (Border, Body, Heading, Meta, Icon, Link, Level-0/1/2 Background + Divider)
- Card/* (Background, Border, Divider, Body, Heading, Meta, Link)
- Input/* (Background, Focused, Inactive, Border states, Body, Icon, Placeholder, Error, Checked)
- Menu/* (Container, Hover, Text, Divider)

### Dark/Light Inversion Logic

When building Light mode from an existing Dark mode:

```
Dark mode surface backgrounds    →  Light mode: invert to lightest neutrals
  Neutral/1100 (near black)      →  Neutral/0 (white) or Neutral/50
  Neutral/1200                   →  Neutral/50
  Neutral/1400                   →  Neutral/100

Dark mode text                   →  Light mode: invert to darkest neutrals
  Neutral/0 (white)              →  Neutral/1000 (near black)
  Neutral/300                    →  Neutral/700
  Neutral/500                    →  Neutral/600

Accent colors                    →  Usually stay the same (orange is orange)
Notification colors              →  Swap -Light ↔ -Dark variants
```

Every Theme variable must be an alias to a Primitive — never hardcode hex values in Theme.

---

## Tier 4: Component Tokens

**Purpose:** Per-component control over spacing, fonts, and radius. This is what makes the system more powerful than generic scales.
**Modes:** Desktop, Tablet, Mobile (same breakpoints as Semantic).
**Scoping:** These are what your actual components consume.

### What Goes Here

Each UI component gets its own token group:

```
Component/Font/Body         → STRING alias to Primitive font family
Component/Font/Heading      → STRING alias
Component/Font/Meta         → STRING alias
Component/Font/Size/Body    → FLOAT alias to Semantic typography scale
Component/Spacing/S         → FLOAT alias to Semantic padding
Component/Spacing/M         → FLOAT alias
Component/Spacing/L         → FLOAT alias
Component/Radius/S          → FLOAT alias to Semantic radius
Component/Radius/M          → FLOAT alias
Component/Radius/L          → FLOAT alias
Component/Gap/S             → FLOAT alias to Semantic gap
Component/Gap/M             → FLOAT alias
Component/Gap/L             → FLOAT alias
```

### Why Component Tokens Matter

Without this tier, a Button and a Panel both use `Padding/Padding-ML` → same padding. But in practice, you often want a Panel to have more padding than a Button at the same "size" designation. Component tokens let you:
- Give Button/Spacing/L → Padding-MS (16px)
- Give Panel/Spacing/L → Padding-ML (28px)
- Both called "L" in their component context, but different resolved values

This is the key architectural difference from systems like Radix Themes that use a single shared scale.

---

## Implementation Checklist

When setting up a new design system in Figma:

1. [ ] Create Primitives collection — all raw sizes, colors, fonts, weights
2. [ ] Create Semantic Tokens collection with Desktop mode — alias all to Primitives
3. [ ] Add Tablet mode to Semantic Tokens — adjust heading/spacing aliases
4. [ ] Add Mobile mode to Semantic Tokens — further reduce headings/spacing
5. [ ] Create Theme collection with Dark mode — assign color roles via Primitive aliases
6. [ ] Add Light mode to Theme — invert the Dark assignments
7. [ ] Create Component Tokens collection with Desktop mode — alias to Semantic
8. [ ] Add Tablet + Mobile modes to Component Tokens
9. [ ] Verify alias chains resolve correctly (no circular refs, no broken links)
10. [ ] Confirm Grid gap tokens are distinct from Flex gap tokens if both exist

## Common Mistakes

- **Skipping tiers:** Don't alias Component Tokens directly to Primitives. The Semantic layer is what makes responsive scaling work automatically.
- **Hardcoding in Theme:** Every Theme variable must alias to a Primitive. Hardcoded hex breaks the cascade.
- **Grid vs Flex confusion:** If you have both Grid and Flex gap scales, make sure components reference the correct one. Buttons use Flex; layouts may use Grid.
- **Missing Null tokens:** Padding-Null and Gap-Null (value: 0) are needed for components that need explicit zero spacing.
- **Mode count limits:** Figma free/pro plans limit modes. Plan your mode strategy around your plan's limits.

## Async API Requirement

When working with Figma variables through the Plugin API (via Desktop Bridge or plugin code), all variable operations must use async versions:
- `getLocalVariableCollectionsAsync()` not `getLocalVariableCollections()`
- `getVariableByIdAsync(id)` not `getVariableById(id)`
- Alias values require format: `{ type: 'VARIABLE_ALIAS', id: 'VariableID:x:y' }`
- Batch operations in groups of 8 or fewer to avoid timeouts
