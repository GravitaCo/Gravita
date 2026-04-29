# Radix Primitives — Component Parts Reference

Maps every Radix Primitive's sub-components to Figma layer names.
Only visual parts are listed — Root, Portal, Provider are code-only.

## Overlays & Floating

### Dialog
```
Dialog.Overlay     → Overlay (full-screen semi-transparent)
Dialog.Content     → [Component root frame]
Dialog.Title       → Title
Dialog.Description → Description
Dialog.Close       → Close
```

### AlertDialog
```
AlertDialog.Overlay     → Overlay
AlertDialog.Content     → [Component root frame]
AlertDialog.Title       → Title
AlertDialog.Description → Description
AlertDialog.Cancel      → Cancel
AlertDialog.Action      → Action
```

### Popover
```
Popover.Trigger → [Separate component or Button instance]
Popover.Content → [Component root frame]
Popover.Close   → Close
Popover.Arrow   → Arrow
```

### Tooltip
```
Tooltip.Trigger → [Separate — the element being hovered]
Tooltip.Content → [Component root frame]
Tooltip.Arrow   → Arrow
```

### Toast
```
Toast.Root        → [Component root frame]
Toast.Title       → Title
Toast.Description → Description
Toast.Action      → Action
Toast.Close       → Close
```

### HoverCard
```
HoverCard.Trigger → [Separate — the element being hovered]
HoverCard.Content → [Component root frame]
HoverCard.Arrow   → Arrow
```

## Menus & Selection

### DropdownMenu
```
DropdownMenu.Trigger       → [Separate component or Button]
DropdownMenu.Content       → [Component root frame]
DropdownMenu.Item          → Item
DropdownMenu.CheckboxItem  → CheckboxItem
DropdownMenu.RadioGroup    → RadioGroup
DropdownMenu.RadioItem     → RadioItem
DropdownMenu.ItemIndicator → ItemIndicator
DropdownMenu.Separator     → Separator
DropdownMenu.Sub           → [Nesting — SubTrigger + SubContent]
DropdownMenu.SubTrigger    → SubTrigger
DropdownMenu.SubContent    → [Nested menu component]
DropdownMenu.Label         → Label (group header)
```

### Select
```
Select.Trigger      → Select.Trigger [component]
Select.Value        → Value (text showing selection)
Select.Icon         → Icon (chevron)
Select.Content      → Select.Content [component]
Select.Viewport     → Viewport
Select.Group        → Group
Select.Label        → Label (group header)
Select.Item         → Item
Select.ItemText     → ItemText
Select.ItemIndicator → ItemIndicator (check mark)
Select.Separator    → Separator
```

### ContextMenu
Same structure as DropdownMenu. Trigger is implicit (right-click area).

### Menubar
```
Menubar.Root    → [Component root frame — horizontal bar]
Menubar.Menu    → [Group — contains trigger + content]
Menubar.Trigger → Trigger
Menubar.Content → [Uses DropdownMenu.Content structure]
```

## Form Controls

### Checkbox
```
Checkbox.Root      → [Component root frame]
Checkbox.Indicator → Indicator
[+ Label outside]  → Label
```

### Switch
```
Switch.Root  → [Component root frame]
Switch.Thumb → Thumb
[Track is implicit — the root frame IS the track]
```

### RadioGroup
```
RadioGroup.Root      → [Component root frame — groups items]
RadioGroup.Item      → Item
RadioGroup.Indicator → Indicator
```

### Slider
```
Slider.Root  → [Component root frame]
Slider.Track → Track
Slider.Range → Range
Slider.Thumb → Thumb
```

### Toggle
```
Toggle.Root → [Single element — the root IS the toggle]
```

### ToggleGroup
```
ToggleGroup.Root → [Component root frame]
ToggleGroup.Item → Item
```

### Label
```
Label.Root → [Single text element]
```

### Progress
```
Progress.Root      → [Component root frame — the track]
Progress.Indicator → Indicator (the fill bar)
```

## Navigation & Layout

### Accordion
```
Accordion.Root    → [Component root — groups items]
Accordion.Item    → [Item component]
Accordion.Header  → Header
Accordion.Trigger → Trigger (inside Header)
Accordion.Content → Content
```

### Tabs
```
Tabs.Root    → [Component root — groups list + content]
Tabs.List    → [Horizontal frame of triggers]
Tabs.Trigger → Tabs.Trigger [component]
Tabs.Content → [Content panel — separate per tab]
```

### Collapsible
```
Collapsible.Root    → [Component root]
Collapsible.Trigger → Trigger
Collapsible.Content → Content
```

### NavigationMenu
```
NavigationMenu.Root       → [Component root — horizontal bar]
NavigationMenu.List       → [Frame of items]
NavigationMenu.Item       → [Group — trigger + content]
NavigationMenu.Trigger    → Trigger
NavigationMenu.Content    → Content
NavigationMenu.Link       → Link
NavigationMenu.Indicator  → Indicator
NavigationMenu.Viewport   → Viewport
```

### ScrollArea
```
ScrollArea.Root      → [Component root — clips content]
ScrollArea.Viewport  → Viewport
ScrollArea.Scrollbar → Scrollbar
ScrollArea.Thumb     → Thumb
ScrollArea.Corner    → Corner (where scrollbars meet)
```

### Separator
```
Separator.Root → [Single element — line or frame]
```

### Toolbar
```
Toolbar.Root        → [Component root — horizontal bar]
Toolbar.Button      → Button
Toolbar.Separator   → Separator
Toolbar.ToggleGroup → ToggleGroup
Toolbar.ToggleItem  → ToggleItem
Toolbar.Link        → Link
```

## Display

### Avatar
```
Avatar.Root     → [Component root — circular clip]
Avatar.Image    → Image
Avatar.Fallback → Fallback (initials text)
```

### AspectRatio
```
AspectRatio.Root → [Single structural frame with aspect ratio constraint]
```
