# Component Patterns

How to build each type of component in the design system.

---

## Check for Existing Design System Components First

Before creating ANY component from scratch, call `search_design_system` to check if the Figma file already has a design system with reusable components:

```
search_design_system({ fileKey: "<key>", query: "Button" })
```

If matches are found:
- Use the existing design system components as the **source of truth** for tokens, patterns, and structure
- Import reusable components via `importComponentByKeyAsync(key)` or `importComponentSetByKeyAsync(key)` in `use_figma` scripts
- Extract tokens (colors, typography, spacing) from these components rather than re-deriving them from individual frames
- Extend and customize the matched components rather than duplicating them

This avoids duplicating existing design system work and ensures consistency with the Figma file's established patterns.

---

## Standard Component Template

Every component follows this structure:

```tsx
"use client";

import * as React from "react";
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";

const componentVariants = cva(
  // Base classes (shared across all variants)
  "inline-flex items-center justify-center transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground",
        secondary: "bg-secondary text-secondary-foreground",
        outline: "border border-border bg-background text-foreground",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        destructive: "bg-destructive text-white",
      },
      size: {
        sm: "h-8 px-3 text-sm",
        md: "h-10 px-4 text-base",
        lg: "h-12 px-6 text-lg",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "md",
    },
  }
);

export interface ComponentProps
  extends React.HTMLAttributes<HTMLDivElement>,
    VariantProps<typeof componentVariants> {
  // Additional props
}

const Component = React.forwardRef<HTMLDivElement, ComponentProps>(
  ({ className, variant, size, ...props }, ref) => {
    return (
      <div
        ref={ref}
        className={cn(componentVariants({ variant, size }), className)}
        {...props}
      />
    );
  }
);
Component.displayName = "Component";

export { Component, componentVariants };
```

Key rules:
- Always use `React.forwardRef` — Radix `asChild` requires it
- Always spread `...props` LAST so consumers can override
- Always set `displayName` for React DevTools
- Always export the CVA variants for compound usage
- Always use `cn()` to merge className

---

## Mapping Figma Variant Matrix to CVA

Figma component sets have variant properties like:

```
Type: Primary | Secondary | Outline | Ghost | Destructive
Size: Small | Medium | Large
State: Default | Hover | Focus | Disabled
```

### Mapping Rules

1. **Type** → CVA `variant`
2. **Size** → CVA `size`
3. **State** → CSS pseudo-classes and `disabled` prop (NOT CVA variants)

States should NOT be CVA variants because they are interactive, not configurable:
- `Default` → no additional classes
- `Hover` → `hover:` prefix classes
- `Focus` → `focus-visible:` prefix classes
- `Disabled` → `disabled:opacity-50 disabled:pointer-events-none`

### Example: Button with Full Matrix

```tsx
const buttonVariants = cva(
  "inline-flex items-center justify-center gap-2 whitespace-nowrap transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring disabled:pointer-events-none disabled:opacity-50 [font-family:'Roboto',sans-serif]",
  {
    variants: {
      variant: {
        primary: "bg-primary text-primary-foreground hover:bg-primary/90",
        secondary: "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        outline: "border border-border bg-background text-foreground hover:bg-accent",
        ghost: "text-foreground hover:bg-accent",
        destructive: "bg-destructive text-white hover:bg-destructive/90",
        link: "text-primary underline-offset-4 hover:underline",
      },
      size: {
        sm: "h-8 rounded-[6px] px-3 text-[13px] font-medium",
        md: "h-10 rounded-[8px] px-4 text-[14px] font-medium",
        lg: "h-12 rounded-[10px] px-6 text-[16px] font-medium",
        icon: "h-10 w-10 rounded-[8px]",
      },
    },
    defaultVariants: {
      variant: "primary",
      size: "md",
    },
  }
);
```

---

## When to Use shadcn Base vs Custom Radix

### Use shadcn base when:
- The component exists in shadcn (Button, Input, Card, Badge, etc.)
- You need standard behavior with custom styling
- Run: `npx shadcn@latest add button` then customize

### Build custom with Radix when:
- The component doesn't exist in shadcn (ButtonGroup, AvatarGroup, CheckboxCard)
- The Figma design diverges significantly from shadcn's structure
- You need non-standard composition patterns

### Radix primitives to use:

Import from the unified `radix-ui` package: `import { Dialog as DialogPrimitive } from "radix-ui"`

Available primitives: Accordion, Dialog, DropdownMenu, Popover, Tabs, Toggle, ToggleGroup, Tooltip, Checkbox, RadioGroup, Select, Switch, Slider, Progress.

---

## Form Input Unified Standard

ALL input-type components MUST share these exact properties for visual consistency:

| Property | Value | Tailwind Class |
|---|---|---|
| Height | 48px (default/lg), 40px (md), 32px (sm) | `h-12`, `h-10`, `h-8` |
| Font size | 16px (lg), 14px (md), 13px (sm) | `text-[16px]`, `text-[14px]`, `text-[13px]` |
| Border radius | 8px | `rounded-[8px]` |
| Border color | var(--border) | `border-border` |
| Border width | 1px | `border` |
| Background | var(--background) | `bg-background` |
| Text color | var(--foreground) | `text-foreground` |
| Placeholder color | var(--muted-foreground) | `placeholder:text-muted-foreground` |
| Focus ring | 2px ring, ring-color | `focus-visible:ring-2 focus-visible:ring-ring` |
| Focus border | var(--ring) | `focus-visible:border-ring` |
| Disabled | 50% opacity, no pointer | `disabled:opacity-50 disabled:pointer-events-none` |
| Error border | var(--destructive) | `border-destructive` |
| Label | 14px, font-medium, foreground | `text-[14px] font-medium text-foreground` |
| Helper text | 12px, muted-foreground | `text-[12px] text-muted-foreground` |
| Error text | 12px, destructive | `text-[12px] text-destructive` |
| Padding horizontal | 12px | `px-3` |
| Font family | Design system font (replace `'Roboto'` with the actual Figma font) | `[font-family:'Roboto',sans-serif]` |

Components that must follow this standard:
- Input
- Textarea
- Select
- Autocomplete / Combobox
- DatePicker
- DateRangePicker
- DateSelect
- PhoneNumber
- SecureInput
- TextMask
- Search

---

## Button Pattern

```tsx
const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, children, ...props }, ref) => {
    return (
      <button
        className={cn(buttonVariants({ variant, size }), className)}
        ref={ref}
        {...props}
      >
        {children}
      </button>
    );
  }
);
```

For icon buttons, add `aria-label`:
```tsx
<Button variant="ghost" size="icon" aria-label="Close">
  <X className="h-4 w-4" />
</Button>
```

---

## Composable Sub-Component Pattern

For complex components like Card, Modal, Table, use the compound component pattern:

```tsx
// Card root
const Card = React.forwardRef<HTMLDivElement, CardProps>(
  ({ className, ...props }, ref) => (
    <div ref={ref} className={cn("rounded-[12px] border border-border bg-card", className)} {...props} />
  )
);

// Card header
const CardHeader = React.forwardRef<HTMLDivElement, React.HTMLAttributes<HTMLDivElement>>(
  ({ className, ...props }, ref) => (
    <div ref={ref} className={cn("flex flex-col gap-1.5 p-6", className)} {...props} />
  )
);

// Card content
const CardContent = React.forwardRef<HTMLDivElement, React.HTMLAttributes<HTMLDivElement>>(
  ({ className, ...props }, ref) => (
    <div ref={ref} className={cn("p-6 pt-0", className)} {...props} />
  )
);

// Card footer
const CardFooter = React.forwardRef<HTMLDivElement, React.HTMLAttributes<HTMLDivElement>>(
  ({ className, ...props }, ref) => (
    <div ref={ref} className={cn("flex items-center p-6 pt-0", className)} {...props} />
  )
);
```

Usage:
```tsx
<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
    <CardDescription>Description</CardDescription>
  </CardHeader>
  <CardContent>Content here</CardContent>
  <CardFooter>
    <Button>Action</Button>
  </CardFooter>
</Card>
```

---

## Radix Primitive Usage

**shadcn supports both Radix UI and Base UI** as primitive layers. This guide shows Radix patterns (the default). If you initialized with `--base base-ui`, adapt the import paths and composition patterns accordingly. The component structure (CVA variants, `cn()`, `forwardRef`) remains the same regardless of the primitive layer.

### Dialog/Modal Pattern

```tsx
import { Dialog as DialogPrimitive } from "radix-ui";

const Modal = ({ children, ...props }) => (
  <DialogPrimitive.Root {...props}>{children}</DialogPrimitive.Root>
);

const ModalTrigger = DialogPrimitive.Trigger;

const ModalContent = React.forwardRef<
  React.ElementRef<typeof DialogPrimitive.Content>,
  React.ComponentPropsWithoutRef<typeof DialogPrimitive.Content>
>(({ className, children, ...props }, ref) => (
  <DialogPrimitive.Portal>
    <DialogPrimitive.Overlay className="fixed inset-0 z-50 bg-black/50 data-[state=open]:animate-in data-[state=closed]:animate-out" />
    <DialogPrimitive.Content
      ref={ref}
      className={cn(
        "fixed left-1/2 top-1/2 z-50 -translate-x-1/2 -translate-y-1/2 rounded-[12px] bg-background p-6 shadow-lg",
        className
      )}
      {...props}
    >
      {children}
      <DialogPrimitive.Close className="absolute right-4 top-4">
        <X className="h-4 w-4" />
      </DialogPrimitive.Close>
    </DialogPrimitive.Content>
  </DialogPrimitive.Portal>
));
```

### Popover Pattern

```tsx
import { Popover as PopoverPrimitive } from "radix-ui";

const Popover = PopoverPrimitive.Root;
const PopoverTrigger = PopoverPrimitive.Trigger;

const PopoverContent = React.forwardRef<...>(
  ({ className, align = "center", sideOffset = 4, ...props }, ref) => (
    <PopoverPrimitive.Portal>
      <PopoverPrimitive.Content
        ref={ref}
        align={align}
        sideOffset={sideOffset}
        className={cn("z-50 rounded-[8px] border bg-popover p-4 shadow-md", className)}
        {...props}
      />
    </PopoverPrimitive.Portal>
  )
);
```

---

## Controlled/Uncontrolled Modes

Support both patterns for every interactive component:

```tsx
interface ToggleProps {
  checked?: boolean;           // Controlled
  defaultChecked?: boolean;    // Uncontrolled
  onCheckedChange?: (checked: boolean) => void;
}

const Toggle = React.forwardRef<HTMLButtonElement, ToggleProps>(
  ({ checked, defaultChecked, onCheckedChange, ...props }, ref) => {
    // If checked is provided, it's controlled
    // If only defaultChecked, it's uncontrolled
    // Radix handles this automatically when using their primitives

    return (
      <SwitchPrimitive.Root
        ref={ref}
        checked={checked}
        defaultChecked={defaultChecked}
        onCheckedChange={onCheckedChange}
        {...props}
      >
        <SwitchPrimitive.Thumb />
      </SwitchPrimitive.Root>
    );
  }
);
```

In Storybook stories, ALWAYS use controlled mode with `React.useState`:
```tsx
render: (args) => {
  const [checked, setChecked] = React.useState(false);
  return <Toggle {...args} checked={checked} onCheckedChange={setChecked} />;
}
```

---

## CSS Variable Usage

### Never hardcode hex values

```tsx
// WRONG
className="bg-[#F5F5F5] text-[#242424] border-[#E0E0E0]"

// CORRECT
className="bg-muted text-foreground border-border"
```

### For colors not in the default palette, add CSS variables

```css
:root {
  --success: #16A34A;
  --success-foreground: #FFFFFF;
}
```

Then use: `bg-success text-success-foreground`

---

## Font Family Arbitrary Property Syntax

**Replace `'Roboto'` with the actual font extracted from the Figma design.** Roboto is used here as an example. Always extract the real font from Figma before scaffolding.

In Tailwind CSS v4, the `font-` prefix is overloaded for both font-family and font-weight. Using `font-['Roboto',sans-serif]` conflicts with `font-medium`.

### Correct syntax:
```tsx
className="[font-family:'Roboto',sans-serif] font-medium"
```

### Wrong syntax:
```tsx
className="font-['Roboto',sans-serif] font-medium"
// This breaks — Tailwind interprets font- as font-weight
```

### Best practice:
Define the font family in `@theme inline` in globals.css:
```css
@theme inline {
  --font-sans: 'Roboto', sans-serif;
}
```
Then use `font-sans` which doesn't conflict with weight utilities.

---

## Icon Handling

### Lucide React (preferred)
```tsx
import { ChevronDown, X, Search, Calendar } from "lucide-react";

<ChevronDown className="h-4 w-4 text-muted-foreground" />
```

### Custom SVG Icons
Always use `currentColor`:
```tsx
const CustomIcon = ({ className, ...props }: React.SVGProps<SVGSVGElement>) => (
  <svg
    viewBox="0 0 24 24"
    fill="none"
    stroke="currentColor"
    strokeWidth={2}
    className={cn("h-4 w-4", className)}
    {...props}
  >
    <path d="..." />
  </svg>
);
```

Never:
```tsx
<svg stroke="#242424" fill="#767676">  // Breaks in dark mode
```
