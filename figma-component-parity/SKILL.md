---
name: figma-component-parity
description: >
  Structure Figma components so their layer hierarchy matches the code component tree. Use this skill 
  whenever building Figma components for a design system that needs to stay in sync with a codebase, 
  renaming layers to match framework component parts, auditing component structure for design-code 
  parity, or setting up components for token binding. Triggers on: "match figma to code", "layer 
  naming", "component structure", "design-code parity", "radix parts", "component anatomy", 
  "design system components", "rename layers", "figma to react", "component architecture", 
  "shadcn components", "headless UI components", or any request to make Figma components mirror 
  their code counterparts. Works with any component framework.
---

# Figma Component Structuring for Code Parity

A methodology for building Figma components whose layer trees mirror the code component tree,
enabling automated design-to-code syncing and eliminating drift between what's designed and built.

## The Core Principle

Your Figma layer panel should read like your JSX. When a developer inspects a component in Figma,
they should see the same names they'd write in code. This isn't about aesthetics — it's about 
making token binding, code generation, and parity checking automatable.

```
Figma Layer Panel          Code (JSX)
─────────────────          ──────────
Dialog                     <Dialog.Root>
  ├── Title                  <Dialog.Title>
  ├── Description            <Dialog.Description>
  ├── Body                   {children}
  └── Actions                <div className="actions">
       ├── Cancel              <Dialog.Close>Cancel</Dialog.Close>
       └── Action              <button>Confirm</button>
```

## Before You Build: Audit First

Before creating new components, always check the existing file for components that already exist 
under different names. Common patterns of "same thing, different name":

```
Existing Name        → Should Be
─────────────        ──────────
Tab                  → Tabs.Trigger
Accordian            → Accordion (typo)
Group 1              → Track (in a Switch)
Button (rectangle)   → Thumb (in a Switch)
Checked (frame)      → Indicator (in a Checkbox)
menu-option          → DropdownMenu.Item or Select.Item
inner-menu-option    → Item.Content
```

**Audit steps:**
1. List all component sets on the Elements/Components pages
2. Map each to its code framework equivalent
3. Walk the layer tree of each variant — flag generic names (Group 1, Frame 5, Rectangle 2)
4. Rename before creating new components to avoid duplication

## Layer Naming Convention

### Rule 1: Name layers after their code component part

The Figma layer name should match the component "part" name in code. For headless libraries 
(Radix, Headless UI, Ark UI, React Aria), these are the documented "parts" or sub-components.

```
Framework           Part Syntax             Figma Layer Name
─────────           ───────────             ────────────────
Radix               Dialog.Title            Title
Radix               Dialog.Description      Description
Radix               Switch.Thumb            Thumb
shadcn/ui           <SelectTrigger>         Trigger
Headless UI         <Menu.Button>           Trigger
React Aria          <ModalOverlay>          Overlay
Custom              <ButtonIcon>            Icon
```

### Rule 2: Use dot notation for sub-parts

When a part contains elements that aren't framework parts but need identification:

```
Trigger.Label    — the text inside a trigger
Trigger.Icon     — the icon inside a trigger
Item.Content     — wrapper inside a menu item
Item.Label       — text inside a menu item
Item.Icon        — icon inside a menu item
```

### Rule 3: Structural frames get semantic names

Never leave auto-generated names. Every frame should describe its role:

```
Bad                  Good
───                  ────
Frame 47             Actions
Group 1              Track
Rectangle 2          Indicator
Auto layout 3        Header
Frame 12             Content
```

### Rule 4: Variant property names match code props

Figma component properties should map to the component's code API:

```
Figma Property       Code Prop          Values
──────────────       ─────────          ──────
State                state/data-state   Default, Hover, Focused, Disabled
Size                 size               S, M, L
Type/Theme           variant            Primary, Secondary, Ghost
Pressed              pressed            On, Off (for toggles)
Active               active             Left, Center, Right (for toggle groups)
```

## Framework-Specific Part Maps

Read the appropriate reference file for your framework:
- `references/radix-parts.md` — Radix Primitives component parts
- `references/generic-parts.md` — Universal patterns that work with any framework

## Building Components: The Process

### Step 1: Document the code anatomy

Before touching Figma, write out the code tree:

```
<Popover.Root>           ← no Figma equivalent (just context)
  <Popover.Trigger />    ← separate component or just a Button
  <Popover.Portal>       ← no Figma equivalent (portal logic)
    <Popover.Content>    ← THE Figma component starts here
      <Popover.Close />  ← Close button inside
      <Popover.Arrow />  ← Optional arrow
    </Popover.Content>
  </Popover.Portal>
</Popover.Root>
```

**Skip non-visual parts:** Root, Portal, Provider — these exist in code for state/context 
management but have no visual representation in Figma.

### Step 2: Build the Figma frame tree

Create the component frame matching the visual tree. Use auto-layout throughout:

- **Vertical auto-layout** for stacking (Dialog content, Accordion items)
- **Horizontal auto-layout** for rows (Actions bar, Header with title + close)
- **Hug contents** for child size, **Fill container** for expanding text/content
- Set gaps using Semantic Token gap variables, not fixed values

### Step 3: Bind variables

Each layer gets variables bound according to its role:

```
Layer Role        Fill Variable         Spacing Variable        Text Variables
──────────        ─────────────         ────────────────        ──────────────
Container         Panel/Level-0-Bg      Panel/Spacing/M         —
Title text        Panel/Heading         —                       Typography/Size/MS, fontWeight
Description       Panel/Body            —                       Typography/Size/XS
Action button     Button/Secondary/Bg   Button/Spacing/LR/S     Typography/Button/S
Menu item         (transparent)         Menu/Spacing/S          Typography/Size/XS
Input field       Input/Background      Inputs/Spacing/S        Typography/Size/XS
Separator         Menu/Divider          —                       —
```

### Step 4: Create variant states

Clone the default component and modify variable bindings for each state:

```
State       What Changes
─────       ────────────
Default     Base variable bindings
Hover       Fill → *-Hover variant, Border → *-Hover variant
Focused     Border → accent color (e.g., Input/Border-Focused)
Disabled    Fill → *-Disabled, Text → *-Disabled, opacity 0.5-0.6
Active      Fill → accent background (for toggles, selected tabs)
Open/Closed Content visibility toggle (for collapsibles, accordions)
```

Then combine all states into a Component Set using `figma.combineAsVariants()`.

### Step 5: Verify the chain

For each component, verify:
- [ ] Every layer has a semantic name (no "Frame 1", "Group 2")
- [ ] Layer hierarchy matches the code component tree
- [ ] All fill colors are bound to Theme variables (not hardcoded)
- [ ] All spacing is bound to Semantic or Component Token variables
- [ ] All text has fontSize bound to Typography tokens
- [ ] All text has fontWeight bound
- [ ] Font family is set to the correct design system font (not Inter/system default)
- [ ] Variant states swap the correct variable bindings

## Component Categories and Their Token Mapping

Different component categories use different token groups. This mapping tells you which tokens 
to bind based on what kind of component you're building:

```
Category          Container Fill          Text Color              Spacing From
────────          ──────────────          ──────────              ────────────
Panel/Dialog      Panel/Level-*-Bg       Panel/Heading,Body      Panel/Spacing/*
Card              Card/Background        Card/Heading,Body       Card/Spacing/*
Menu/Select       Menu/Container         Menu/Text               Menu/Spacing/*
Input/Form        Input/Background       Input/Body              Inputs/Spacing/*
Button            Button/*/Background    Button/*/Text           Button/Spacing/*
Tab               (transparent)          Global/Menu-Text        Tab/Spacing/*
Toast             Panel/Level-0-Bg       Panel/Heading,Body      Panel/Spacing/*
Tooltip           (inverted)             (inverted)              Card/Spacing/*
```

## Font Assignment Rules

When building components, assign fonts based on the text's role, not its size:

```
Text Role         Font Family              Example Layers
─────────         ───────────              ──────────────
Headings/Titles   Heading font (display)   Title, Trigger.Label
Body/Content      Body font (readable)     Description, ItemText, Value
Meta/Labels       Meta font (compact)      Label (group headers), timestamps
```

Bind fontWeight as a variable (from Primitives) — don't rely on the font style name, as that 
creates implicit coupling to a specific font's style names.

## Common Component Architectures

### Overlay components (Dialog, AlertDialog, Popover, Toast, HoverCard, Tooltip)
```
[Root container with Panel tokens]
  Title (heading font + heading color)
  Description (body font + body color)
  Body/Content (slot frame, transparent)
  Actions (horizontal auto-layout with gap)
    Cancel (button tokens)
    Action (button tokens, accent variant)
  Close (small frame, top-right or inline)
  Arrow (rotated rectangle — for popovers/tooltips)
```

### Menu components (DropdownMenu, ContextMenu, Select, Menubar)
```
[Root container with Menu/Container + border]
  Item (horizontal auto-layout, transparent fill)
    ItemText (body font + Menu/Text)
  CheckboxItem (same as Item + ItemIndicator frame at start)
    ItemIndicator (small frame for check icon)
    ItemText
  RadioGroup (vertical frame grouping RadioItems)
    RadioItem (same structure as CheckboxItem)
  Separator (1px frame with Menu/Divider fill)
  SubTrigger (Item + SubIndicator arrow at end)
```

### Form components (Input, Checkbox, Switch, RadioGroup, Slider)
```
Switch: Root (auto-layout) > Track (frame) > Thumb (frame)
Checkbox: Root (auto-layout) > Indicator (frame) + Label (text)
Radio: Root (auto-layout) > Item (ellipse group) > Indicator (inner ellipse) + Label
Slider: Root (frame) > Track (frame) > Range (frame) + Thumb (frame)
Input: Root (auto-layout) > Placeholder/Value (text) [+ Icon frame]
```
