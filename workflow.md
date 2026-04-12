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

### 1.2 Extract Design Data with `get_design_context` (Primary Tool)

`get_design_context` is the **primary extraction tool** — always use it before `use_figma`. It returns structured data including:

- **Layout properties**: Auto Layout direction, constraints, sizing modes (hug/fixed/fill)
- **Typography specs**: font family, size, weight, line height, letter spacing
- **Color values and design tokens**: fills, strokes, effects with resolved values
- **Component structure and variants**: component properties, variant definitions
- **Spacing and padding**: padding (top/right/bottom/left), item spacing, gaps

```
get_design_context({ fileKey: "<key>", nodeId: "<node-id>" })
```

Call it on the root or a top-level frame to get a comprehensive overview of the design. If the response is too large (truncated), use this fallback strategy:

1. Call `get_metadata` on the file to get the full node map with IDs
2. Then call `get_design_context` on individual child nodes one at a time
3. Aggregate the results

**When to fall back to `use_figma`:** Use `use_figma` plugin API scripts only when `get_design_context` doesn't provide enough detail — for example, extracting fills from deeply nested nodes, reading `componentPropertyDefinitions` on component sets, iterating over local paint/text styles, or accessing Figma variables. For standard extraction of layout, typography, colors, and spacing, `get_design_context` is faster, more reliable, and returns pre-structured data.

**Variable API fragility:** Figma variable/token APIs (`get_variable_defs`, `getLocalVariableCollectionsAsync`) may require specific node context or fail silently. If variable extraction fails, fall back to `get_design_context` which returns resolved token values inline with each node. Extract the color/spacing/radius values directly from the design context output — this is often more reliable than querying variable definitions separately.

### 1.3 List All Top-Level Frames (Component Inventory)

If `get_design_context` on the root is too large, or you need the full page/frame inventory, run a `use_figma` call to enumerate every page and its top-level frames:

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

### 1.4 Extract Design Tokens

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

### Quality Gate: Token Validation

Before proceeding to scaffold, validate extracted tokens against known design system patterns:

**Color check:**
- Use exactly what the source design specifies. If the source uses pure #000000 for text, replicate it faithfully.
- If the source doesn't define secondary/muted text colors, derive them: secondary at 60% opacity of primary, muted at 40%.
- If the source has multiple accent colors for interactive elements, extract all of them — some designs intentionally use more than one.

**Typography check:**
- Extract the exact font sizes from the source. 16px body text is the industry standard — if the source uses a different baseline, follow the source.
- If letter-spacing isn't specified in the source, use 0 (normal) for body and slight negative (-0.02em) for headlines 24px+.
- If fewer than 6 size levels are found, the source may be a mockup, not a design system — extract what exists and add standard intermediate sizes only if needed.

**Radius check:**
- Extract the EXACT value from each component — radius varies widely between design systems and is brand-defining.
- If only one radius value is found, apply it consistently. If none is found, use 8px as the industry default for inputs/cards.

**Shadow check:**
- If the source has shadows, extract the exact values including multi-layer stacks.
- If the source has NO shadows, that's a valid design choice — don't add them.
- If shadow values aren't extractable, use a standard 3-level scale: subtle (0 1px 3px rgba(0,0,0,0.08)), card (0 2px 8px rgba(0,0,0,0.12)), elevated (0 8px 24px rgba(0,0,0,0.16)).

### 1.5 Extract Component Variant Properties

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

### 1.6 Get Screenshots for Reference

Use `get_screenshot` to capture individual components:

```
get_screenshot({ fileKey: "<key>", nodeId: "<node-id>" })
```

Screenshots are useful for visual reference but NOT sufficient for pixel verification. JPEG compression alters colors. Always extract numerical values (fills, font-size, padding) programmatically.

### 1.7 Extract Fills, Strokes, and Text Styles from Any Node (Fallback)

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

### 1.8 Resolving Figma Spec Ambiguities

**Resolving Figma spec ambiguities:** When `get_design_context` returns values that conflict with Figma variables (e.g., a node shows `cornerRadius: 8` but the design system variable `border_radius/large` resolves to `10`), prefer the variable/token value. Variables represent the design system's intent; raw node values may be overridden instances. Always check if a value is bound to a variable before using the raw number.

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

### 2.2 Create .gitignore

Create `.gitignore` BEFORE running `npm install`:
```gitignore
node_modules/
dist/
.next/
out/
storybook-static/
*.tsbuildinfo
.DS_Store
```

### 2.3 Initialize shadcn

```bash
cd <project-name>
npx shadcn@latest init -d
```

The `-d` flag runs non-interactively with defaults. This creates:
- `components.json` configuration
- `src/lib/utils.ts` with `cn()` utility
- `src/app/globals.css` with CSS variable theme

### 2.4 Install Tailwind CLI

Ensure `@tailwindcss/cli` is installed alongside `tailwindcss`:
```bash
npm install -D tailwindcss @tailwindcss/cli
```

### 2.5 Replace Default Theme in globals.css

Replace the generated CSS variables with tokens extracted from Figma:

```css
@import "tailwindcss";

@plugin "tailwindcss-animate";

@custom-variant dark (&:is(.dark *));

:root {
  --background: #FFFFFF;                /* extracted from Figma */
  --foreground: #242424;                /* extracted from Figma */
  --card: #FFFFFF;
  --card-foreground: #242424;
  --primary: #333333;                   /* mapped from Figma primary */
  --primary-foreground: #FAFAFA;
  --secondary: #F5F5F5;
  --secondary-foreground: #333333;
  --muted: #F5F5F5;
  --muted-foreground: #595959;          /* #595959 for AAA contrast */
  --accent: #F5F5F5;
  --accent-foreground: #333333;
  --destructive: #DC2626;
  --border: #E5E5E5;
  --input: #E5E5E5;
  --ring: #A3A3A3;
  --radius: 0.625rem;                   /* default from Figma */

  /* Custom tokens from Figma */
  --success: #16A34A;
  --warning: #CA8A04;
  --info: #2563EB;
}

.dark {
  --background: #242424;
  --foreground: #FAFAFA;
  /* ... all dark mode values ... */
}
```

### 2.6 Set Up @theme Inline Block

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

### 2.7 Configure Storybook

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

### 2.8 Additional Project Files

#### .nvmrc
```
22
```

#### src/index.ts (barrel export — starts empty)
```typescript
export { cn } from './lib/utils';
// Add component exports incrementally as each component is built in Phase 3.
// Do NOT add exports for components that don't exist yet — TypeScript will fail.
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

Build components in batches of 3. If your IDE supports parallel agent execution (e.g., Claude Code subagents, Cursor background tasks), dispatch each batch concurrently. Otherwise, build them sequentially. Each batch receives:
1. Figma extraction data (fills, strokes, padding, typography, variant matrix)
2. Design tokens (the CSS variables from globals.css)
3. Target file paths (component.tsx + component.stories.tsx)
4. The story format from storybook-conventions.md
5. The component template from component-patterns.md

### Asset Handling from Figma

The Figma MCP server serves images, icons, and SVGs via localhost URLs. When `get_design_context` or `get_screenshot` returns asset URLs:

1. **Download and use assets directly** — the MCP server provides localhost URLs for images, icons, and SVG exports. Download them and store in the project.
2. **DO NOT import new icon packages** (e.g., `react-icons`, `heroicons`) if the Figma payload provides the actual asset files. Use what the design provides.
3. **DO NOT use placeholder images or icons** if a localhost source URL is available from the MCP server. Always download and embed the real asset.
4. **Storage locations:**
   - Raster images (PNG, JPG, WebP) → `public/images/`
   - SVG icons → `src/components/icons/` as React components, or `public/icons/` as static files
   - Brand assets (logos, favicons) → `public/`
5. **SVG optimization:** When downloading SVG assets, ensure they use `currentColor` for fills and strokes so they adapt to dark mode. If the downloaded SVG has hardcoded colors, replace them with `currentColor`.

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
6. **Verify immediately** — run the Phase 4 visual comparison loop for this component NOW, before starting the next one. Fetch the source screenshot, open Storybook, compare side-by-side, and fix any discrepancies.

See [component-patterns.md](component-patterns.md) for detailed per-component patterns.

---

## Phase 4: Verify Pixel Fidelity

**IMPORTANT: Automated builds (`next build`, `storybook build`) verify that code compiles — they do NOT verify pixel fidelity. You MUST open Storybook in a browser (or use `preview_inspect` if available) and visually compare each component against the Figma screenshot. Build passing ≠ design matching.**

### 4.1 Start Storybook and Fetch Source Screenshots

**This is not optional — visual comparison is the core verification step.**

1. Start Storybook:
```bash
npx storybook dev -p 6006
```

2. For each component, get the source reference:
```
get_screenshot(fileKey="<key>", nodeId="<component-node-id>")
```
Keep this screenshot visible — it's the ground truth.

### 4.2 Side-by-Side Visual Comparison Loop

For EVERY component, run this loop:

1. **Open the component in Storybook** — navigate to the iframe URL:
   ```
   http://localhost:6006/iframe.html?id=components-button--primary&viewMode=story
   ```

2. **Take a Storybook screenshot** — use `preview_screenshot` if available, or view it manually in the browser.

3. **Compare against the Figma screenshot** — look at both images and check:
   - Does the overall shape and layout match?
   - Are the proportions correct (height, width, padding)?
   - Do the colors look the same (not just close — the same)?
   - Is the typography visually identical (weight, size, spacing)?

4. **Inspect computed CSS** — right-click → Inspect Element (or use `preview_inspect`) to verify exact values:
   - height, padding, border-radius, font-size, font-weight, color, background-color

5. **If anything doesn't match** → fix the component, reload Storybook, repeat from step 2.

6. **Move to the next component** only when the current one is pixel-perfect.

Do NOT batch-verify after building all components. Verify each component IMMEDIATELY after building it, before starting the next one.

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

### 4.6 Pixel Fidelity Checklist

Run through this checklist for EVERY component before considering it complete:

- [ ] **Layout matches** — spacing, alignment, and sizing match the Figma frame (Auto Layout direction, gap, padding)
- [ ] **Typography matches** — font family, font size, font weight, and line height are identical to Figma
- [ ] **Colors match exactly** — background, text, border, and accent colors match the Figma fills (compare hex values, not visual approximation)
- [ ] **Interactive states work as designed** — hover, active, focus, and disabled states match their Figma variant counterparts
- [ ] **Responsive behavior follows Figma constraints** — fill/hug/fixed sizing modes, min/max widths, and wrapping behavior
- [ ] **Assets render correctly** — icons, images, and SVGs from the Figma file display at the right size and color
- [ ] **Accessibility standards met** — ARIA labels, keyboard navigation, focus rings, and AAA contrast ratios

#### Design System Quality Checks
- [ ] Text color matches the source design exactly
- [ ] Accent/interactive colors match the source — extract all that are defined
- [ ] Typography sizes match the source; if the source doesn't specify a full hierarchy, fill gaps with standard intermediate sizes
- [ ] If the source doesn't specify letter-spacing, body uses 0 and headlines 24px+ use -0.02em
- [ ] Border-radius matches extracted values exactly (not Tailwind defaults)
- [ ] Shadow levels are consistent (not ad-hoc per component); if none in source, don't add them
- [ ] Font weights match the source — don't default to bold (700) for headlines if the system uses medium (500)

### 4.7 Fix Discrepancies

When a property doesn't match:
1. Re-extract the specific value from Figma — try `get_design_context` first, fall back to `use_figma` for targeted extraction
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
    ".": {
      "types": "./src/index.ts",
      "import": "./src/index.ts"
    },
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

### 6.4 Generate Design System Rules

Use the `create_design_system_rules` tool to generate project-specific rules that future Figma-to-code tasks will follow automatically. This tool is provided by the Figma MCP server (see [SKILL.md — Tool Reference](SKILL.md)).

```
create_design_system_rules({
  clientLanguages: ["TypeScript"],
  clientFrameworks: ["React", "Next.js", "Tailwind CSS"]
})
```

This generates reusable content for `CLAUDE.md`, `AGENTS.md`, or `.cursorrules` files that encode:
- The project's design token conventions (CSS variable naming, color format)
- Component structure patterns (CVA variants, forwardRef, compound components)
- Styling rules (Tailwind v4 syntax, arbitrary property syntax)
- Accessibility requirements (AAA contrast, reduced motion, currentColor icons)

Run this after the design system is complete so future development sessions maintain consistency.

### 6.5 Final Checklist

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
