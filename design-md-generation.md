# DESIGN.md Generation

Generate a [DESIGN.md](https://stitch.withgoogle.com/docs/design-md/overview/) file from the extracted Figma design tokens. DESIGN.md is the emerging standard for encoding design systems in a format that AI coding agents can consume — it's plain markdown that any LLM reads natively.

## What is DESIGN.md?

DESIGN.md is a markdown file that captures a project's complete visual identity: colors, typography, component styles, spacing, elevation, and usage rules. Unlike JSON token files or Figma-specific formats, it's human-readable AND agent-readable. Drop it in your project root and any AI coding agent (Cursor, Claude Code, Copilot, Stitch) instantly understands how your UI should look.

## When to Generate

Generate DESIGN.md in **two passes**:
1. **Draft in Phase 2** — after extracting tokens, write a skeleton DESIGN.md with colors, typography, and spacing. This serves as the reference during component building.
2. **Finalize in Phase 7** — after all components are verified, update DESIGN.md with component stylings, elevation, do's/don'ts, and the agent prompt guide.

This two-pass approach prevents DESIGN.md from being skipped as 'optional' and keeps the design spec in sync throughout the build.

The file lives at the project root alongside CLAUDE.md or AGENTS.md.

## Required Sections

**Important:** DESIGN.md uses hex values for documentation purposes — this is correct and expected. [SKILL.md Rule 2](SKILL.md) ('never hardcode hex') applies to **component source code** (`.tsx` files), not to DESIGN.md. DESIGN.md is a specification document, not executable code. Use exact hex values in color tables and component spec sections.

DESIGN.md follows a 9-section structure. Each section must contain real, extracted values — never placeholders or generic descriptions.

### Section 1: Visual Theme & Atmosphere

Describe the overall design philosophy in 2-3 paragraphs. Cover:
- Primary impression (minimal, bold, corporate, playful)
- Font personality and how it's used (weight choices, tracking)
- Shadow/elevation philosophy
- Key visual characteristics as a bullet list

**Extract from Figma:** Look at the overall component library. Note the dominant font weights, border-radius patterns, shadow styles, and color temperature (warm vs cool neutrals).

### Section 2: Color Palette & Roles

Organize colors by function, not just hue. Every color needs a hex value and a role description.

**Subcategories:**
- **Primary** — brand color, CTA backgrounds, interactive highlights
- **Semantic** — success, warning, error, info with their specific hex values
- **Neutral scale** — heading, body, muted, disabled, placeholder text colors
- **Surface & Borders** — card backgrounds, border colors, divider colors
- **Interactive** — hover states, focus rings, selected states

**Extract from Figma:** Use `get_design_context` on the component page, or `use_figma` to scan for unique fill/stroke colors across all components. Map each color to its usage context.

### Section 3: Typography Rules

A complete type hierarchy table with exact values.

**Required columns:** Role, Font, Size, Weight, Line Height, Letter Spacing, Notes

**Extract from Figma:** Scan text nodes across components. Group by font-size to identify the hierarchy. Note font-family, font-weight (use name: Regular=400, Medium=500, Bold=700), and line-height (as ratio, e.g., 1.5).

```markdown
| Role | Font | Size | Weight | Line Height | Letter Spacing | Notes |
|------|------|------|--------|-------------|----------------|-------|
| Display | Roboto | 32px | 700 | 1.2 | -0.5px | Page titles |
| Heading 1 | Roboto | 24px | 500 | 1.3 | -0.3px | Section titles |
| Heading 2 | Roboto | 18px | 500 | 1.4 | normal | Card titles |
| Body | Roboto | 16px | 400 | 1.5 | normal | Default text |
| Body Small | Roboto | 14px | 400 | 1.5 | normal | Secondary text |
| Caption | Roboto | 12px | 400 | 1.4 | normal | Labels, helpers |
| Button LG | Roboto | 18px | 500 | 1.5 | normal | Large buttons |
| Button MD | Roboto | 16px | 500 | 1.5 | normal | Default buttons |
| Button SM | Roboto | 14px | 500 | 1.5 | normal | Small buttons |
```

### Section 4: Component Stylings

Document each component type with exact values. At minimum cover:

**Buttons** — each variant (primary, secondary, ghost, link) with: background, text color, padding, border-radius, font size/weight, hover state, disabled state.

**Cards** — background, border, border-radius, shadow, padding, title typography, body typography.

**Inputs** — height, border, border-radius, padding, font size, placeholder color, focus state, error state, label style, helper text style.

**Badges/Tags** — each variant with background, text color, padding, border-radius, font size.

**Alerts/Toasts** — each type (success, warning, error, info) with background tint, text color, accent color, border-radius, padding.

**Extract from Figma:** For each component, use `get_design_context` on a representative variant, then extract fills, strokes, padding, corner-radius, and text styles.

### Section 5: Layout Principles

**Spacing system** — list the spacing scale used (e.g., 4px base: 4, 8, 12, 16, 20, 24, 32, 48, 64).

**Grid** — max content width, column system, gutter sizes.

**Whitespace philosophy** — how spacing is used between sections, inside cards, between form elements.

**Border-radius scale** — list all distinct radii used (e.g., 4px for inputs, 8px for cards, 10px for buttons, 16px for modals, 9999px for pills).

### Section 6: Depth & Elevation

A table of shadow levels from flat to modal:

```markdown
| Level | Shadow | Use |
|-------|--------|-----|
| Flat | none | Backgrounds, inline elements |
| Subtle | 0 1px 2px rgba(0,0,0,0.05) | Hover hints |
| Card | 2px 2px 6px rgba(0,0,0,0.25) | Cards, panels |
| Elevated | 4px 4px 12px rgba(0,0,0,0.25) | Dropdowns, popovers |
| Modal | 8px 8px 24px rgba(0,0,0,0.25) | Modals, drawers |
| Focus | 0 0 0 2px var(--primary) | Keyboard focus ring |
```

### Section 7: Do's and Don'ts

Concrete rules extracted from the design patterns:

```markdown
### Do
- Use CSS variables for all colors — never hardcode hex values
- Use `currentColor` for SVG icons so they adapt to context
- Apply consistent border-radius: 8px for inputs, 10px for buttons/cards, 16px for modals
- Use font-weight 500 (Medium) for interactive text, 400 (Regular) for body

### Don't
- Don't use pure black (#000000) for text — use the foreground variable
- Don't mix border-radius values within the same component type
- Don't hardcode shadows — use the elevation scale
- Don't use font-weight 700 (Bold) for UI text — reserve for data emphasis
```

### Section 8: Responsive Behavior

Breakpoints, touch targets, and collapse strategies:

```markdown
| Breakpoint | Width | Key Changes |
|------------|-------|-------------|
| Mobile | <640px | Single column, reduced heading sizes |
| Tablet | 640-1024px | 2-column grids |
| Desktop | >1024px | Full layout |
```

- Minimum touch target: 44x44px
- Input heights maintained across breakpoints
- Typography scales down at mobile (e.g., 24px → 20px headings)

### Section 9: Agent Prompt Guide

Quick reference for AI agents generating components:

**Quick Color Reference** — primary, background, foreground, border, muted, success, warning, error hex values.

**Example Component Prompts** — 3-5 ready-to-use prompts showing how to build components using this design system's exact values.

## Extraction Script

Use this `use_figma` script pattern to bulk-extract the data needed for DESIGN.md:

```javascript
// Extract all unique colors, fonts, and radii from all components
const page = figma.root.children[0];
await figma.setCurrentPageAsync(page);

const colors = new Set();
const fonts = new Set();
const fontSizes = new Set();
const radii = new Set();
const shadows = [];

function rgbToHex(r, g, b) {
  const toHex = c => Math.round(c * 255).toString(16).padStart(2, '0');
  return `#${toHex(r)}${toHex(g)}${toHex(b)}`;
}

function traverse(node) {
  if (node.fills && Array.isArray(node.fills)) {
    for (const fill of node.fills) {
      if (fill.type === 'SOLID' && fill.visible !== false) {
        colors.add(rgbToHex(fill.color.r, fill.color.g, fill.color.b));
      }
    }
  }
  if (node.type === 'TEXT') {
    if (node.fontName !== figma.mixed) fonts.add(`${node.fontName.family} ${node.fontName.style}`);
    if (node.fontSize !== figma.mixed) fontSizes.add(node.fontSize);
  }
  if (node.cornerRadius !== undefined && node.cornerRadius !== figma.mixed && node.cornerRadius > 0) {
    radii.add(node.cornerRadius);
  }
  if (node.effects) {
    for (const e of node.effects) {
      if (e.type === 'DROP_SHADOW' && e.visible) shadows.push(e);
    }
  }
  if ('children' in node) node.children.forEach(traverse);
}

page.children.forEach(traverse);

return {
  colors: [...colors].sort(),
  fonts: [...fonts].sort(),
  fontSizes: [...fontSizes].sort((a, b) => a - b),
  radii: [...radii].sort((a, b) => a - b),
  shadowCount: shadows.length
};
```

## Output Location

Save as `DESIGN.md` in the project root. It sits alongside:
- `CLAUDE.md` / `AGENTS.md` — project-level AI instructions
- Any project-level AI agent configuration files
- `package.json` — project configuration

## Validation

After generating DESIGN.md, verify it by:
1. Checking every hex color appears in `globals.css` as a CSS variable
2. Confirming font names match what's loaded in `preview-head.html` and `layout.tsx`
3. Ensuring component styles match what's rendered in Storybook
4. Testing that an AI agent can use the file to generate a new component that matches the existing design system
