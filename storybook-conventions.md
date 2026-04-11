# Storybook Conventions

Standards for writing Storybook stories in the design system.

---

## Import Pattern

Every story file uses the CSF3 (Component Story Format 3) pattern:

```tsx
import type { Meta, StoryObj } from "@storybook/react";
import React from "react";
import { ComponentName } from "./component-name";

const meta: Meta<typeof ComponentName> = {
  title: "Components/ComponentName",
  component: ComponentName,
  tags: ["autodocs"],
  parameters: {
    layout: "centered", // or "padded" for full-width components
  },
};

export default meta;
type Story = StoryObj<typeof ComponentName>;

export const Default: Story = {
  args: {
    // Only static, non-interactive props here
    children: "Label text",
    variant: "default",
    size: "md",
  },
};
```

---

## Never Import from @storybook/test

Do NOT import `fn`, `expect`, `within`, `userEvent`, or other test utilities from `@storybook/test`. This package may not be installed and causes build failures.

```tsx
// WRONG — do not use
import { fn } from "@storybook/test";
export const Default: Story = {
  args: {
    onClick: fn(),
  },
};

// CORRECT — use inline functions or omit
export const Default: Story = {
  args: {
    onClick: () => console.log("clicked"),
  },
};
```

---

## Stateful Stories with React.useState

For ANY component that has controlled state (checked, value, selected, open, activeIndex), you MUST use a `render` function with `React.useState`. Static `args` do NOT update on interaction — the component appears broken.

### Checkbox / Toggle / Switch

```tsx
export const Default: Story = {
  render: (args) => {
    const [checked, setChecked] = React.useState(false);
    return (
      <Checkbox
        {...args}
        checked={checked}
        onCheckedChange={setChecked}
      />
    );
  },
};
```

### Input / Search / Autocomplete

```tsx
export const Default: Story = {
  render: (args) => {
    const [value, setValue] = React.useState("");
    return (
      <Input
        {...args}
        value={value}
        onChange={(e) => setValue(e.target.value)}
      />
    );
  },
};
```

### With Clear Button

```tsx
export const WithClearButton: Story = {
  render: (args) => {
    const [value, setValue] = React.useState("Initial text");
    return (
      <Search
        {...args}
        value={value}
        onChange={(e) => setValue(e.target.value)}
        onClear={() => setValue("")}
      />
    );
  },
};
```

### Tabs

```tsx
export const Default: Story = {
  render: (args) => {
    const [activeTab, setActiveTab] = React.useState("tab1");
    return (
      <Tabs value={activeTab} onValueChange={setActiveTab}>
        <TabsList>
          <TabsTrigger value="tab1">Tab 1</TabsTrigger>
          <TabsTrigger value="tab2">Tab 2</TabsTrigger>
        </TabsList>
        <TabsContent value="tab1">Content 1</TabsContent>
        <TabsContent value="tab2">Content 2</TabsContent>
      </Tabs>
    );
  },
};
```

### CheckboxCard (Selectable Cards)

```tsx
export const Default: Story = {
  render: (args) => {
    const [selected, setSelected] = React.useState<string[]>([]);
    const toggle = (id: string) =>
      setSelected((prev) =>
        prev.includes(id) ? prev.filter((i) => i !== id) : [...prev, id]
      );
    return (
      <div className="flex gap-4">
        {["option1", "option2", "option3"].map((id) => (
          <CheckboxCard
            key={id}
            checked={selected.includes(id)}
            onCheckedChange={() => toggle(id)}
          >
            Option {id}
          </CheckboxCard>
        ))}
      </div>
    );
  },
};
```

### DatePicker

```tsx
export const Default: Story = {
  render: (args) => {
    const [date, setDate] = React.useState<Date | undefined>(undefined);
    return (
      <DatePicker
        {...args}
        value={date}
        onChange={setDate}
      />
    );
  },
};
```

---

## Radix Trigger Buttons MUST Use forwardRef

Any component used as a child of a Radix trigger with `asChild` MUST use `React.forwardRef`. Without it, Radix cannot attach event handlers and `data-state` attributes. The component will silently fail to open.

```tsx
// WRONG — Dropdown won't open
<DropdownMenu.Trigger asChild>
  <span onClick={...}>Open Menu</span>  {/* span doesn't forward ref */}
</DropdownMenu.Trigger>

// CORRECT
const TriggerButton = React.forwardRef<HTMLButtonElement, ButtonProps>(
  (props, ref) => <button ref={ref} {...props} />
);

<DropdownMenu.Trigger asChild>
  <TriggerButton>Open Menu</TriggerButton>
</DropdownMenu.Trigger>
```

This also applies to:
- `Dialog.Trigger`
- `Popover.Trigger`
- `Tooltip.Trigger`
- `AlertDialog.Trigger`
- `DropdownMenu.Trigger`
- `ContextMenu.Trigger`

---

## CSS Variable Classes in Stories

Never use hardcoded hex values in story files:

```tsx
// WRONG
<div style={{ backgroundColor: "#F5F5F5", color: "#242424" }}>

// CORRECT
<div className="bg-muted text-foreground">
```

When you need to demonstrate different states, use CSS variable classes:

```tsx
export const AllVariants: Story = {
  render: () => (
    <div className="flex flex-col gap-4 bg-background p-8">
      <Badge variant="default">Default</Badge>
      <Badge variant="secondary">Secondary</Badge>
      <Badge variant="destructive">Destructive</Badge>
      <Badge variant="outline">Outline</Badge>
    </div>
  ),
};
```

---

## SVG Icons in Stories

All SVG icons must use `currentColor`, never hardcoded colors:

```tsx
// WRONG
<svg stroke="#242424"><path d="..." /></svg>

// CORRECT — use Lucide icons
import { Search, X, ChevronDown } from "lucide-react";
<Search className="h-4 w-4 text-muted-foreground" />

// CORRECT — custom SVG
<svg stroke="currentColor" className="h-4 w-4 text-muted-foreground">
  <path d="..." />
</svg>
```

---

## Dark Mode Toggle Setup

### .storybook/preview.ts

```typescript
import type { Preview } from "@storybook/react";
import "../src/app/globals.css";

const preview: Preview = {
  globalTypes: {
    theme: {
      description: "Global theme for components",
      toolbar: {
        title: "Theme",
        icon: "circlehollow",
        items: [
          { value: "light", title: "Light", icon: "sun" },
          { value: "dark", title: "Dark", icon: "moon" },
        ],
        dynamicTitle: true,
      },
    },
  },
  initialGlobals: {
    theme: "light",
  },
  decorators: [
    (Story, context) => {
      const theme = context.globals.theme || "light";
      document.documentElement.classList.toggle("dark", theme === "dark");
      return (
        <div className={theme === "dark" ? "dark" : ""}>
          <div className="bg-background text-foreground min-h-screen p-8">
            <Story />
          </div>
        </div>
      );
    },
  ],
};

export default preview;
```

### Important Notes on Dark Mode in Storybook

1. The `.dark` class must be on `<html>` (document.documentElement) for CSS `&:is(.dark *)` to work
2. Radix portals (modals, popovers, dropdowns) append to `document.body` — they inherit dark mode via the `&:is(.dark *)` custom variant
3. The decorator wrapper div with `bg-background` ensures the Storybook canvas itself reflects the theme

---

## preview-head.html for Font Loading

**Replace `'Roboto'` with the actual font extracted from the Figma design.** Roboto is used here as an example. Always extract the real font from Figma before scaffolding.

`next/font` does NOT work in Storybook. You must load fonts via a `<link>` tag in `.storybook/preview-head.html`:

```html
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link
  href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500;600;700&display=swap"
  rel="stylesheet"
/>
```

Replace "Roboto" with the actual font from the Figma design. Include all weights used in the design.

---

## Title Naming Convention

Use a hierarchical naming convention:

```tsx
// Top-level components
title: "Components/Button"
title: "Components/Card"
title: "Components/Input"

// Sub-categorized components
title: "Components/Forms/Input"
title: "Components/Forms/Select"
title: "Components/Forms/Checkbox"

title: "Components/Data Display/Table"
title: "Components/Data Display/Badge"

title: "Components/Feedback/Alert"
title: "Components/Feedback/Toast"

title: "Components/Navigation/Tabs"
title: "Components/Navigation/Breadcrumb"

title: "Components/Overlay/Modal"
title: "Components/Overlay/Dropdown"
title: "Components/Overlay/Popover"
```

Keep it simple for smaller design systems — just use `Components/Name`. Add subcategories when you have 20+ components.

---

## Story Naming

Name stories after the variant or use case they demonstrate:

```tsx
export const Default: Story = { ... };
export const Primary: Story = { args: { variant: "primary" } };
export const Secondary: Story = { args: { variant: "secondary" } };
export const Small: Story = { args: { size: "sm" } };
export const Large: Story = { args: { size: "lg" } };
export const Disabled: Story = { args: { disabled: true } };
export const WithIcon: Story = { ... };
export const WithHelperText: Story = { ... };
export const ErrorState: Story = { ... };
export const AllVariants: Story = { ... };  // Grid showing every combination
```

---

## Grid Display for Variant Matrices

When a component has multiple variants, show them all in a grid:

```tsx
export const AllVariants: Story = {
  render: () => (
    <div className="flex flex-col gap-8">
      {(["primary", "secondary", "outline", "ghost", "destructive"] as const).map(
        (variant) => (
          <div key={variant} className="flex items-center gap-4">
            <span className="w-24 text-sm text-muted-foreground">{variant}</span>
            {(["sm", "md", "lg"] as const).map((size) => (
              <Button key={size} variant={variant} size={size}>
                {variant} {size}
              </Button>
            ))}
          </div>
        )
      )}
    </div>
  ),
};
```

---

## Layout Parameter

Use the appropriate layout for each component:

```tsx
// Centered — for small, self-contained components (buttons, badges, inputs)
parameters: {
  layout: "centered",
},

// Padded — for medium components that need some breathing room (cards, alerts)
parameters: {
  layout: "padded",
},

// Fullscreen — for full-width or page-level components (navbars, tables, modals)
parameters: {
  layout: "fullscreen",
},
```
