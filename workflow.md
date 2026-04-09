# Figma-to-Design-System Workflow

The complete end-to-end pipeline for generating a pixel-perfect, accessible React component library from a Figma file.

---

## Phase 1: Extract (Figma Analysis)

### 1.1 Get the Figma File Key

Extract the file key from any Figma URL:

```
https://www.figma.com/design/ABC123xyz/My-Design-System?node-id=...
                              ^^^^^^^^^
                              This is the file key: ABC123xyz
```

The file key is the segment between `/design/` (or `/file/`) and the next `/`.

### 1.2 List All Top-Level Frames (Component Inventory)

Run a `use_figma` call to enumerate every page and its top-level frames:

```javascript
use_figma({
  fileKey: "<key>",
  code: `
    const pages = figma.root.children;
    return pages.map(page => ({
      name: page.name,
      frames: page.children
        .filter(c => c.type === 'FRAME' || c.type === 'COMPONENT_SET' || c.type === 'COMPONENT')
        .map(f => ({ name: f.name, type: f.type, id: f.id, width: f.width, height: f.height }))
    }));
  `
})
```

This gives you the full component inventory. Record frame names and IDs for later extraction.

### 1.3 Extract Design Tokens

#### Colors
```javascript
use_figma({
  fileKey: "<key>",
  code: `
    // Find color styles or variables
    const styles = figma.getLocalPaintStyles();
    return styles.map(s => ({
      name: s.name,
      color: s.paints[0]?.type === 'SOLID'
        ? { r: Math.round(s.paints[0].color.r * 255),
            g: Math.round(s.paints[0].color.g * 255),
            b: Math.round(s.paints[0].color.b * 255),
            opacity: s.paints[0].opacity }
        : null
    }));
  `
})
```

#### Typography
```javascript
use_figma({
  fileKey: "<key>",
  code: `
    const styles = figma.getLocalTextStyles();
    return styles.map(s => ({
      name: s.name,
      fontFamily: s.fontName.family,
      fontWeight: s.fontName.style,
      fontSize: s.fontSize,
      lineHeight: s.lineHeight,
      letterSpacing: s.letterSpacing
    }));
  `
})
```

#### Spacing and Radii
Extract from specific components by inspecting their auto-layout and corner radius properties:
```javascript
use_figma({
  fileKey: "<key>",
  code: `
    const node = figma.getNodeById("<node-id>");
    return {
      padding: {
        top: node.paddingTop,
        right: node.paddingRight,
        bottom: node.paddingBottom,
        left: node.paddingLeft
      },
      gap: node.itemSpacing,
      cornerRadius: node.cornerRadius,
      topLeftRadius: node.topLeftRadius,
      topRightRadius: node.topRightRadius,
      bottomRightRadius: node.bottomRightRadius,
      bottomLeftRadius: node.bottomLeftRadius
    };
  `
})
```

### 1.4 Extract Component Variant Properties

For component sets (components with variants), extract the variant matrix:

```javascript
use_figma({
  fileKey: "<key>",
  code: `
    const node = figma.getNodeById("<component-set-id>");
    if (node.type === 'COMPONENT_SET') {
      return {
        name: node.name,
        variantProperties: node.componentPropertyDefinitions,
        variants: node.children.map(v => ({
          name: v.name,
          properties: v.variantProperties,
          width: v.width,
          height: v.height
        }))
      };
    }
  `
})
```

This reveals matrices like `Type=Primary, Size=Large, State=Default` which map directly to CVA variants.

### 1.5 Get Screenshots for Reference

Use `get_screenshot` to capture individual components:

```
get_screenshot({ fileKey: "<key>", nodeId: "<node-id>" })
```

Screenshots are useful for visual reference but NOT sufficient for pixel verification. JPEG compression alters colors. Always extract numerical values (fills, font-size, padding) programmatically.

### 1.6 Extract Fills, Strokes, and Text Styles from Any Node

```javascript
use_figma({
  fileKey: "<key>",
  code: `
    const node = figma.getNodeById("<node-id>");
    return {
      fills: node.fills,
      strokes: node.strokes,
      strokeWeight: node.strokeWeight,
      effects: node.effects,
      // For text nodes:
      ...(node.type === 'TEXT' ? {
        characters: node.characters,
        fontSize: node.fontSize,
        fontName: node.fontName,
        fontWeight: node.fontName?.style,
        textAlignHorizontal: node.textAlignHorizontal,
        textAlignVertical: node.textAlignVertical,
        textDecoration: node.textDecoration,
        lineHeight: node.lineHeight,
        letterSpacing: node.letterSpacing,
        fills: node.fills
      } : {}),
      // Layout
      width: node.width,
      height: node.height,
      cornerRadius: node.cornerRadius,
      paddingTop: node.paddingTop,
      paddingRight: node.paddingRight,
      paddingBottom: node.paddingBottom,
      paddingLeft: node.paddingLeft,
      itemSpacing: node.itemSpacing,
      layoutMode: node.layoutMode,
      primaryAxisAlignItems: node.primaryAxisAlignItems,
      counterAxisAlignItems: node.counterAxisAlignItems
    };
  `
})
```

---

## Phase 2: Scaffold (Project Setup)

### 2.1 Create Next.js Project

```bash
npx create-next-app@latest <project-name> \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*" \
  --turbopack
```

### 2.2 Initialize shadcn

```bash
cd <project-name>
npx shadcn@latest init -d
```

The `-d` flag runs non-interactively with defaults. This creates:
- `components.json` configuration
- `src/lib/utils.ts` with `cn()` utility
- `src/app/globals.css` with CSS variable theme

### 2.3 Replace Default Theme in globals.css

Replace the generated CSS variables with tokens extracted from Figma:

```css
@import "tailwindcss";

@plugin "tailwindcss-animate";

@custom-variant dark (&:is(.dark *));

:root {
  --background: oklch(1 0 0);           /* extracted from Figma */
  --foreground: oklch(0.145 0 0);       /* extracted from Figma */
  --card: oklch(1 0 0);
  --card-foreground: oklch(0.145 0 0);
  --primary: oklch(0.205 0 0);          /* mapped from Figma primary */
  --primary-foreground: oklch(0.985 0 0);
  --secondary: oklch(0.97 0 0);
  --secondary-foreground: oklch(0.205 0 0);
  --muted: oklch(0.97 0 0);
  --muted-foreground: oklch(0.384 0 0); /* #595959 for AAA contrast */
  --accent: oklch(0.97 0 0);
  --accent-foreground: oklch(0.205 0 0);
  --destructive: oklch(0.577 0.245 27.325);
  --border: oklch(0.922 0 0);
  --input: oklch(0.922 0 0);
  --ring: oklch(0.708 0 0);
  --radius: 0.625rem;                   /* default from Figma */

  /* Custom tokens from Figma */
  --success: oklch(0.55 0.15 145);
  --warning: oklch(0.75 0.15 75);
  --info: oklch(0.6 0.15 250);
}

.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
  /* ... all dark mode values ... */
}
```

### 2.4 Set Up @theme Inline Block

Map CSS variables to Tailwind utilities:

```css
@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-card: var(--card);
  --color-card-foreground: var(--card-foreground);
  --color-primary: var(--primary);
  --color-primary-foreground: var(--primary-foreground);
  --color-secondary: var(--secondary);
  --color-secondary-foreground: var(--secondary-foreground);
  --color-muted: var(--muted);
  --color-muted-foreground: var(--muted-foreground);
  --color-accent: var(--accent);
  --color-accent-foreground: var(--accent-foreground);
  --color-destructive: var(--destructive);
  --color-border: var(--border);
  --color-input: var(--input);
  --color-ring: var(--ring);
  --radius: var(--radius);

  /* Custom semantic colors */
  --color-success: var(--success);
  --color-warning: var(--warning);
  --color-info: var(--info);
}
```

### 2.5 Configure Storybook

Install Storybook:
```bash
npx storybook@latest init --skip-install
npm install
```

#### .storybook/preview.ts
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
          <div className="bg-background text-foreground p-8">
            <Story />
          </div>
        </div>
      );
    },
  ],
};

export default preview;
```

#### .storybook/preview-head.html
```html
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link
  href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500;600;700&display=swap"
  rel="stylesheet"
/>
```

Note: `next/font` does NOT work in Storybook. You MUST load fonts via `<link>` in `preview-head.html`.

### 2.6 Additional Project Files

#### .nvmrc
```
22
```

#### src/index.ts (barrel export)
```typescript
export { Button } from "./components/ui/button";
export { Card, CardHeader, CardContent, CardFooter } from "./components/ui/card";
// ... add each component as it's built
```

#### src/lib/tokens.ts
```typescript
export const tokens = {
  colors: {
    primary: "var(--primary)",
    secondary: "var(--secondary)",
    // ... all CSS variable references
  },
  radii: {
    sm: "6px",
    md: "8px",
    lg: "10px",
    xl: "12px",
    full: "9999px",
  },
  spacing: {
    xs: "4px",
    sm: "8px",
    md: "12px",
    lg: "16px",
    xl: "24px",
  },
} as const;
```

---

## Phase 3: Generate Components

### Dispatch Strategy

Build components in batches of 3 using parallel agents. Each agent receives:
1. Figma extraction data (fills, strokes, padding, typography, variant matrix)
2. Design tokens (the CSS variables from globals.css)
3. Target file paths (component.tsx + component.stories.tsx)
4. The story format from storybook-conventions.md
5. The component template from component-patterns.md

### Priority Order

1. **Foundation components** first: Button, Badge, Avatar, Card
2. **Form inputs** second: Input, Select, Checkbox, Radio, Toggle, Textarea
3. **Complex composites** third: Modal, Dropdown, Accordion, Tabs, Table
4. **Specialized** last: DatePicker, Calendar, Signature, FileUpload, RichTextEditor

### Per-Component Steps

1. Extract all variant data from Figma (sizes, types, states)
2. Extract measurements (padding, radius, font-size, height, gap)
3. Extract colors as CSS variable mappings
4. Generate component with CVA variants
5. Generate Storybook stories with stateful render functions
6. Verify in Storybook (Phase 4)

See [component-patterns.md](component-patterns.md) for detailed per-component patterns.

---

## Phase 4: Verify Pixel Fidelity

### 4.1 Start Storybook

```bash
npx storybook dev -p 6006
```

### 4.2 Navigate and Inspect

For each component story, navigate to its iframe URL:
```
http://localhost:6006/iframe.html?id=components-button--primary&viewMode=story
```

Use `preview_inspect` on specific elements to get computed CSS properties.

### 4.3 Property-by-Property Comparison

Compare these properties against Figma extraction data:

| Property | How to check |
|---|---|
| height | `preview_inspect` → computed height |
| font-size | `preview_inspect` → computed fontSize |
| font-weight | `preview_inspect` → computed fontWeight |
| font-family | `preview_inspect` → computed fontFamily |
| color | `preview_inspect` → computed color (convert to hex) |
| background-color | `preview_inspect` → computed backgroundColor |
| border-radius | `preview_inspect` → computed borderRadius |
| padding | `preview_inspect` → computed padding (top/right/bottom/left) |
| gap | `preview_inspect` → computed gap |
| border | `preview_inspect` → computed border |
| line-height | `preview_inspect` → computed lineHeight |

### 4.4 State Verification

Test each interactive state:
- **Hover**: Use `preview_eval` to dispatch mouseenter events or add `:hover` pseudo-class
- **Focus**: Tab to the element and inspect focus ring
- **Disabled**: Set `disabled` prop and verify opacity/cursor changes
- **Active/Pressed**: Click and hold, verify state change
- **Error**: Set error prop and verify red border/text

### 4.5 Interactivity Testing

- **Click**: `preview_click` on buttons, verify callback fires
- **Toggle**: Click checkboxes/toggles, verify state changes visually
- **Type**: `preview_fill` into inputs, verify value appears
- **Clear**: Click clear buttons, verify input empties
- **Open/Close**: Click dropdowns/modals, verify panel appears/disappears

### 4.6 Fix Discrepancies

When a property doesn't match:
1. Re-extract the specific value from Figma using `use_figma`
2. Update the component code
3. Re-verify in Storybook

Common discrepancies:
- `rounded-lg` resolving to a different value than expected (use explicit `rounded-[8px]`)
- Font weight names mapping differently (Figma "Medium" = 500, not CSS "medium")
- Padding values being off because Figma measures from different edges

---

## Phase 5: Dark Mode and Accessibility

### 5.1 Replace Hardcoded Colors

Search the entire codebase for hardcoded hex values:
```bash
grep -rn '#[0-9A-Fa-f]\{3,8\}' src/components/
```

Replace every instance with the appropriate CSS variable class:
- `#FFFFFF` / `#FAFAFA` → `bg-background`
- `#242424` / `#1A1A1A` → `text-foreground`
- `#767676` → `text-muted-foreground` (but verify it's `#595959` for AAA)
- `#E0E0E0` / `#CCCCCC` → `border-border`

### 5.2 Verify Dark Mode

Toggle the `.dark` class in Storybook using the theme switcher toolbar button. Check every component:
- Background should flip from light to dark
- Text should flip from dark to light
- Borders should remain visible
- Badges, tags, and status indicators should remain readable
- Canvas-based components should redraw with correct colors

### 5.3 AAA Contrast Compliance

- **Normal text**: 7:1 contrast ratio minimum (WCAG AAA)
- **Large text** (18px+ or 14px bold): 4.5:1 minimum
- **Placeholder text**: Use `#595959` on white (7:1 ratio). NOT `#767676` (only 4.54:1)
- **Muted foreground**: Set `--muted-foreground` to a value achieving 7:1 on both light and dark backgrounds

### 5.4 Reduced Motion

Add to globals.css:
```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

### 5.5 SVG Icons

Verify all SVG icons use `currentColor`:
```bash
grep -rn 'stroke="#' src/components/
grep -rn 'fill="#' src/components/
```

Replace all with `stroke="currentColor"` or `fill="currentColor"`.

### 5.6 Canvas Elements

For canvas-based components (signature pad, charts), read CSS variables at draw time:
```typescript
const style = getComputedStyle(document.documentElement);
const fg = style.getPropertyValue('--foreground').trim();
ctx.strokeStyle = fg;
```

---

## Phase 6: Finalize

### 6.1 Update Barrel Exports

Ensure `src/index.ts` exports every component:
```typescript
export { Button, type ButtonProps } from "./components/ui/button";
export { Card, CardHeader, CardContent, CardFooter } from "./components/ui/card";
export { Input, type InputProps } from "./components/ui/input";
// ... every component
```

### 6.2 Build Verification

```bash
# Next.js build
npx next build

# Storybook build
npx storybook build
```

Both must complete with zero errors. Warnings are acceptable but should be minimized.

### 6.3 Package Configuration

Ensure `package.json` has:
```json
{
  "name": "@yourorg/design-system",
  "version": "1.0.0",
  "main": "src/index.ts",
  "types": "src/index.ts",
  "exports": {
    ".": "./src/index.ts",
    "./globals.css": "./src/app/globals.css"
  },
  "files": [
    "src/components",
    "src/lib",
    "src/app/globals.css",
    "src/index.ts"
  ],
  "peerDependencies": {
    "react": ">=18",
    "react-dom": ">=18",
    "tailwindcss": ">=4"
  }
}
```

### 6.4 Final Checklist

- [ ] All components have stories
- [ ] All stories render without errors
- [ ] Dark mode works for every component
- [ ] No hardcoded hex values in component files
- [ ] All SVG icons use currentColor
- [ ] AAA contrast ratios verified
- [ ] Barrel exports complete
- [ ] Next.js build passes
- [ ] Storybook build passes
- [ ] package.json configured for npm publishing
