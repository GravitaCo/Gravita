---
name: design-token-pipeline
description: >
  Export Figma design system variables as CSS custom properties for React/web consumption. Use this 
  skill whenever generating CSS tokens from Figma variables, building a Figma-to-code sync pipeline, 
  creating theme files from design variables, or setting up a design token export workflow. Triggers 
  on: "export tokens", "CSS custom properties", "design tokens to code", "figma to CSS", "token 
  pipeline", "sync figma to react", "generate CSS from figma", "design system export", "CSS 
  variables from figma", "token build", "style dictionary", or any request to get design values 
  out of Figma and into a codebase. Also trigger when someone needs theme switching (dark/light) 
  or responsive tokens in CSS.
---

# Design Token Export Pipeline

A complete system for converting Figma design variables into CSS custom properties that drive 
web/React components. Handles the 4-tier variable architecture, responsive breakpoints, and 
theme switching with zero JavaScript runtime.

## Architecture Overview

```
Figma Variables                    CSS Custom Properties
───────────────                    ─────────────────────
Primitives (raw values)        →   primitives.css    (:root)
Semantic Tokens (responsive)   →   semantic.css      (:root + @media overrides)
Theme (dark/light colors)      →   theme.css         (:root + [data-theme] + @media)
Component Tokens (per-component) → components.css    (:root)
                                   index.css         (single import entry point)
                                   types.ts          (TypeScript autocomplete)
```

**Single import in your app:**
```tsx
import './tokens/index.css';
```

## CSS File Structure

### primitives.css — Raw Values

All numbers and hex codes live here. Nothing else references raw values directly.

**Naming:** `--{prefix}-{category}-{value}`

```css
:root {
  /* Sizes */
  --cx2-size-4: 4px;
  --cx2-size-8: 8px;
  --cx2-size-16: 16px;
  /* ... full scale ... */

  /* Colors */
  --cx2-color-neutral-0: #ffffff;
  --cx2-color-neutral-500: #9da0a9;
  --cx2-color-neutral-1000: #1d1e22;
  /* ... full ramps ... */

  /* Fonts */
  --cx2-font-headings: 'Your Heading Font', sans-serif;
  --cx2-font-body: 'Your Body Font', sans-serif;
  --cx2-font-meta: 'Your Mono Font', monospace;

  /* Weights */
  --cx2-weight-regular: 400;
  --cx2-weight-bold: 800;
}
```

### semantic.css — Responsive Scaling

References primitives via `var()`. Desktop is default, smaller breakpoints override.

**Key pattern:** Each semantic token maps to a primitive, and the mapping changes per breakpoint.

```css
:root {
  --cx2-typo-xxl: var(--cx2-size-60);
  --cx2-padding-ml: var(--cx2-size-28);
  --cx2-gap-m: var(--cx2-size-12);
  --cx2-radius-m: var(--cx2-size-16);
}

@media (max-width: 1024px) {  /* Tablet */
  :root {
    --cx2-typo-xs: var(--cx2-size-14);  /* Only override what changes */
    --cx2-padding-xxl: var(--cx2-size-60);
  }
}

@media (max-width: 768px) {  /* Mobile */
  :root {
    --cx2-typo-xxl: var(--cx2-size-48);  /* Headings shrink */
    --cx2-padding-ml: var(--cx2-size-24);  /* Spacing tightens */
    --cx2-gap-xl: var(--cx2-size-28);
  }
}
```

**Breakpoint strategy:**
- Desktop: default (no media query)
- Tablet: `@media (max-width: 1024px)` — override only what changes
- Mobile: `@media (max-width: 768px)` — override more aggressively

### theme.css — Color Mode Switching

Three mechanisms supported simultaneously. The CSS cascade handles priority:

```css
/* Dark mode — DEFAULT */
:root,
[data-theme="dark"] {
  --cx2-global-bg: var(--cx2-color-neutral-1500);
  --cx2-panel-level0-bg: var(--cx2-color-neutral-1100);
  --cx2-btn-primary-bg: var(--cx2-color-neutral-1000);
  /* ... all color role tokens ... */
}

/* Light mode — EXPLICIT */
[data-theme="light"] {
  --cx2-global-bg: var(--cx2-color-neutral-50);
  --cx2-panel-level0-bg: var(--cx2-color-neutral-0);
  --cx2-btn-primary-bg: var(--cx2-color-neutral-1000);
  /* ... inverted color role tokens ... */
}

/* Light mode — SYSTEM PREFERENCE */
@media (prefers-color-scheme: light) {
  :root:not([data-theme]) {
    /* Same as [data-theme="light"] above */
    /* Only applies when no explicit data-theme is set */
  }
}
```

**Theme switching in your app:**
```tsx
// Manual toggle
const toggle = () => {
  const html = document.documentElement;
  html.dataset.theme = html.dataset.theme === 'dark' ? 'light' : 'dark';
};

// Or let the system decide (remove data-theme entirely)
delete document.documentElement.dataset.theme;
```

### components.css — Per-Component Tokens

References semantic tokens. This is what your React components actually consume.

```css
:root {
  /* Button */
  --cx2-btn-font: var(--cx2-font-body);
  --cx2-btn-spacing-tb-s: var(--cx2-padding-xs);
  --cx2-btn-spacing-lr-l: var(--cx2-padding-ml);
  --cx2-btn-gap-m: var(--cx2-gap-m);
  --cx2-btn-radius-m: var(--cx2-radius-sm);

  /* Panel */
  --cx2-panel-font-heading: var(--cx2-font-headings);
  --cx2-panel-spacing-m: var(--cx2-padding-ms);
  --cx2-panel-radius-l: var(--cx2-radius-ml);

  /* ... more components ... */
}
```

### index.css — Entry Point

```css
@import './primitives.css';
@import './semantic.css';
@import './theme.css';
@import './components.css';
```

Import order matters: each file can reference the files above it.

## Usage in React Components

### With Radix Primitives (or any headless library)

```tsx
import * as Dialog from '@radix-ui/react-dialog';
import './tokens/index.css';

<Dialog.Content className="ds-dialog-content">
  <Dialog.Title className="ds-dialog-title">Title</Dialog.Title>
  <Dialog.Description className="ds-dialog-desc">Body text</Dialog.Description>
</Dialog.Content>
```

```css
.ds-dialog-content {
  background: var(--cx2-panel-level0-bg);
  border: 1px solid var(--cx2-panel-border);
  padding: var(--cx2-panel-spacing-m);
  border-radius: var(--cx2-panel-radius-l);
}

.ds-dialog-title {
  font-family: var(--cx2-panel-font-heading);
  font-size: var(--cx2-typo-ms);
  color: var(--cx2-panel-heading);
}

.ds-dialog-desc {
  font-family: var(--cx2-panel-font-body);
  font-size: var(--cx2-typo-xs);
  color: var(--cx2-panel-body);
}
```

### Button size pattern

```css
.ds-btn { font-family: var(--cx2-btn-font); display: inline-flex; align-items: center; }
.ds-btn--s { padding: var(--cx2-btn-spacing-tb-s) var(--cx2-btn-spacing-lr-s); font-size: var(--cx2-typo-btn-s); gap: var(--cx2-btn-gap-s); }
.ds-btn--m { padding: var(--cx2-btn-spacing-tb-m) var(--cx2-btn-spacing-lr-m); font-size: var(--cx2-typo-btn-m); gap: var(--cx2-btn-gap-m); }
.ds-btn--l { padding: var(--cx2-btn-spacing-tb-l) var(--cx2-btn-spacing-lr-l); font-size: var(--cx2-typo-btn-l); gap: var(--cx2-btn-gap-l); }

.ds-btn--primary { background: var(--cx2-btn-primary-bg); color: var(--cx2-btn-primary-text); border: 1px solid var(--cx2-btn-primary-border); }
.ds-btn--primary:hover { background: var(--cx2-btn-primary-bg-hover); border-color: var(--cx2-btn-primary-border-hover); }
.ds-btn--primary:disabled { background: var(--cx2-btn-primary-disabled-bg); color: var(--cx2-btn-primary-disabled-text); }
```

## TypeScript Token Helpers

Generate a `types.ts` that provides autocomplete and type safety:

```typescript
export type TypographyScale = 'xxl' | 'xl' | 'l' | 'ml' | 'm' | 'ms' | 's' | 'xs' | 'xxs';
export type SpacingScale = 'xxl' | 'xl' | 'l' | 'ml' | 'm' | 'ms' | 's' | 'xs' | 'xxs' | 'null';

export function token(name: string): string { return `var(--cx2-${name})`; }
export function typo(scale: TypographyScale): string { return `var(--cx2-typo-${scale})`; }
export function padding(scale: SpacingScale): string { return `var(--cx2-padding-${scale})`; }
```

## Figma Export Script

To extract fresh token values from Figma, run this in the Desktop Bridge or Plugin Console.
See `references/export-script.md` for the complete extraction script.

The script:
1. Walks all variable collections
2. Resolves alias chains to get both the alias path and final value
3. Converts Figma RGBA objects to hex/rgba strings
4. Outputs structured JSON that can drive CSS generation

## Naming Convention: Figma → CSS

```
Figma Path                      CSS Custom Property
──────────                      ───────────────────
Color/Neutral/500               --cx2-color-neutral-500
Button/Primary/Background       --cx2-btn-primary-bg
Panel/Level-0-Background        --cx2-panel-level0-bg
Typography/Size/XXL             --cx2-typo-xxl
Padding/Padding-ML              --cx2-padding-ml
Gap/Flex/Gap-M                  --cx2-gap-m
Radius/Radius-SM                --cx2-radius-sm
```

**Transform rules:**
- Lowercase everything
- Replace `/` with `-`
- Remove redundant words (Padding/Padding → padding, Radius/Radius → radius)
- Abbreviate where clear (Background → bg, Typography → typo)
- Remove spaces, replace with `-`

## Keeping in Sync

When your design team changes values in Figma:

1. Run the export script via Desktop Bridge → get `tokens.json`
2. Regenerate CSS files from the JSON (manually or via build script)
3. Commit to your repo — CSS changes flow to all components automatically

No code changes needed unless you're adding entirely new token categories.

**What triggers a code change:**
- New component token group → add to `components.css`
- New breakpoint → add `@media` block in `semantic.css`  
- New theme mode → add `[data-theme="..."]` block in `theme.css`
- New primitive → add to `primitives.css`

**What doesn't trigger a code change:**
- Changing a color value
- Adjusting spacing/typography values
- Tweaking responsive scaling
- Adjusting light mode assignments
