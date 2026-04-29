# Generic Component Parts — Framework-Agnostic Reference

These layer naming patterns work regardless of your component framework.
Use when you don't have a specific framework's part naming to follow,
or when building custom components outside a headless library.

## Universal Part Names

These names are understood across frameworks and should be your default:

```
Part Name        Role                           Figma Layer Type
─────────        ────                           ────────────────
Root             Outermost container            COMPONENT or FRAME
Trigger          Element that opens/activates   FRAME (clickable)
Content          Main content area              FRAME
Title            Primary heading text           TEXT
Description      Secondary/body text            TEXT
Header           Container for title row        FRAME (horizontal)
Body             Slot for user content          FRAME (transparent)
Actions          Row of buttons                 FRAME (horizontal)
Close            Close/dismiss button           FRAME or INSTANCE
Icon             Icon placeholder               FRAME (fixed size)
Label            Text label                     TEXT
Indicator        Visual state marker            FRAME or ELLIPSE
Track            Background rail (sliders/switches)  FRAME
Thumb            Draggable handle               FRAME
Range            Filled portion of track        FRAME
Separator        Visual divider                 FRAME (1px)
Arrow            Floating element pointer       RECTANGLE (rotated 45°)
Overlay          Semi-transparent backdrop      FRAME (full-screen)
Viewport         Scrollable content area        FRAME (clips content)
Scrollbar        Scroll indicator rail          FRAME (thin)
Fallback         Shown when primary fails       TEXT or FRAME
```

## Naming Patterns

### Dot notation for nested specificity
```
Trigger.Label    — text inside a trigger
Trigger.Icon     — icon inside a trigger
Item.Content     — wrapper frame inside a list item
Item.Label       — text inside a list item
Item.Icon        — icon inside a list item
```

### State-differentiated items
```
CheckboxItem     — item with a checkbox indicator
RadioItem        — item with a radio indicator
SubTrigger       — item that opens a sub-menu
SubContent       — nested content panel
SubIndicator     — arrow indicating sub-menu
ItemIndicator    — check/radio mark inside an item
```

### Level-differentiated surfaces
```
Level-0-Background   — deepest surface (page/panel)
Level-1-Background   — elevated surface (sidebar, drawer)
Level-2-Background   — highest surface (popover, modal)
Level-0-Divider      — divider on level 0
```

## Component Category Templates

### Overlay (Dialog, Modal, Sheet, Drawer)
```
[Root]
  Title
  Description
  Body           ← transparent slot for content
  Actions
    Cancel
    Action
  Close
```

### Menu (Dropdown, Context, Select list)
```
[Root]
  Item
    ItemText
  CheckboxItem
    ItemIndicator
    ItemText
  RadioGroup
    RadioItem
      ItemIndicator
      ItemText
  Separator
  SubTrigger
    ItemText
    SubIndicator
  Label          ← group header
```

### Form Field (Input, Textarea, Select trigger)
```
[Root]
  Value / Placeholder  ← text showing current value
  Icon                 ← optional trailing icon
```

### Toggle (Switch, Checkbox, Radio)
```
Switch:    [Root] → Track → Thumb
Checkbox:  [Root] → Indicator + Label
Radio:     [Root] → Item (Circle) → Indicator + Label
```

### Navigation (Tabs, Menubar, Breadcrumb)
```
Tabs:      [Root] → List → Trigger (×N) + Content (×N)
Menubar:   [Root] → Trigger (×N)
```

### Display (Card, Badge, Avatar, Toast)
```
Card:    [Root] → Header (Title + Subtitle) + Body + Footer
Avatar:  [Root] → Image + Fallback
Toast:   [Root] → TextContent (Title + Description) + Action + Close
Badge:   [Root] → Label + Icon (optional)
```

## Rules That Apply Everywhere

1. **No generic names ever** — Frame 1, Group 2, Rectangle 3 are never acceptable
2. **Auto-layout on everything** — Every frame that contains children should have auto-layout
3. **Transparent fills on slots** — Body, Content, Actions frames have no fill (transparent)
4. **Text layers get the role name** — "Title" not "Heading Text" or "Dialog Heading"
5. **Icons are frames, not instances** — Use a fixed-size empty frame as placeholder; designers swap the instance
6. **Separators are 1px frames** — Not lines (lines don't participate in auto-layout properly)
