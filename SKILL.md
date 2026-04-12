---
name: figma-pixel-perfect
version: 1.2.0
description: Use when the user wants to generate a pixel-perfect design system npm library or app-local design system from a Figma file. Extracts theme tokens, colors, typography, and spacing via Figma MCP, scaffolds React components on shadcn/ui + Tailwind CSS + Radix UI, ensures WCAG AAA accessibility, supports light and dark mode, and generates Storybook documentation. Triggers on phrases like "build design system from Figma", "extract Figma theme", "generate component library from Figma", "create pixel-perfect components", "Figma to React", "design system from Figma file".
---

# Figma to Design System Pipeline

Generate a pixel-perfect, accessible, dark-mode-ready React component library from Figma designs — publishable to npm with Storybook documentation.

## When to Use This Skill

- User provides a Figma file URL and wants a React component library
- User says "build design system from Figma", "extract Figma theme", "generate component library"
- User wants pixel-perfect component replication from Figma designs
- User needs an npm-publishable design system with Storybook

## Prerequisites Check

Before starting, verify ALL of the following:

### 1. Figma MCP Connected
A Figma MCP server must be running and connected in your IDE. The user provides an authorized Figma file URL — the MCP handles authentication and file access. Test the connection with any available Figma MCP tool:
- `get_design_context(fileKey="<key>", nodeId="0:1")` — returns node data
- `get_screenshot(fileKey="<key>", nodeId="0:1")` — returns an image
- `use_figma(fileKey="<key>", code="return figma.root.name")` — if plugin API is available

Not all Figma MCP servers expose the same tools. `get_design_context` and `get_screenshot` are the most widely available.

If all tools fail, check that the Figma MCP server is configured and running in your IDE settings.

### 2. Node.js >= 20
```bash
node --version  # Must be >= 20.9.0
```
If using nvm: `nvm use 22` (or latest LTS)

### 3. Required Global Tools
- `npm` or `pnpm`
- `npx` (for shadcn CLI and create-next-app)

---

## Figma MCP Tools Reference

These tools are provided by the Figma MCP server. They are available when the Figma MCP is connected in your IDE.

| Tool | Purpose | Key Parameters |
|------|---------|---------------|
| `get_design_context` | Primary extraction tool. Returns structured layout, typography, colors, tokens, spacing for a node. | `fileKey`, `nodeId` |
| `get_screenshot` | Returns a screenshot image of a Figma node for visual reference. | `fileKey`, `nodeId` |
| `get_metadata` | Returns the node tree structure (names, types, IDs, positions). Use when `get_design_context` response is too large. | `fileKey`, `nodeId` |
| `use_figma` | Executes JavaScript via the Figma Plugin API. Use for custom extraction when the above tools don't provide enough detail. | `fileKey`, `code`, `description` |
| `search_design_system` | Searches for existing design system components in the Figma file. | query string |
| `create_design_system_rules` | Generates project-specific AI agent rules for consistent Figma-to-code workflows. | `clientLanguages`, `clientFrameworks` |

**Note:** `preview_inspect`, `preview_screenshot`, `preview_eval`, and `preview_click` are browser preview tools — they are NOT part of the Figma MCP. They are available when a local dev server (e.g., Storybook) is running and a browser preview MCP is connected. If unavailable, use manual browser DevTools inspection instead.

---

## Pipeline Overview

### Phase 1: Extract (Figma Analysis)
See [workflow.md — Phase 1](workflow.md)

### Phase 2: Scaffold (Project Setup)
See [workflow.md — Phase 2](workflow.md)

### Phase 3: Generate Components
See [component-patterns.md](component-patterns.md)

### Phase 4: Verify Pixel Fidelity
See [workflow.md — Phase 4](workflow.md)

### Phase 5: Dark Mode & Accessibility
See [accessibility-rules.md](accessibility-rules.md)

### Phase 6: Storybook & Publish
See [storybook-conventions.md](storybook-conventions.md)

### Phase 7: Generate DESIGN.md
See [design-md-generation.md](design-md-generation.md) — generates a [DESIGN.md](https://stitch.withgoogle.com/docs/design-md/overview/) file from the extracted tokens. This is the emerging standard for encoding design systems in agent-readable markdown. Drop it in your project root and any AI editor instantly understands your visual identity.

---

## Critical Rules (Always Apply)

### 1. Never Trust Generated Output — Always Verify
After building any component, verify pixel fidelity by comparing computed CSS against Figma specs. **Default method:** Open Storybook in your browser, right-click → Inspect Element, and check computed styles (height, font-size, font-weight, color, border-radius, padding). **Enhanced method:** If a browser preview MCP is available (e.g., Claude Preview), use `preview_inspect` for automated CSS comparison. Screenshots alone are insufficient — JPEG compression obscures color differences.

### 2. Never Hardcode Hex Colors
Every color MUST use a CSS variable from `globals.css`. Use `bg-background`, `text-foreground`, `border-border` — never `bg-[#FFFFFF]`, `text-[#242424]`, `border-[#CCCCCC]`. Hardcoded hex values break dark mode.

### 3. Font Family Uses Arbitrary Property Syntax
In Tailwind CSS v4, `font-['Roboto',sans-serif]` conflicts with `font-medium`. Always use:
```
[font-family:'Roboto',sans-serif]
```
Never:
```
font-['Roboto',sans-serif]
```

### 4. Radix `asChild` Requires `forwardRef`
Any component used as a child of a Radix trigger with `asChild` MUST use `React.forwardRef` and spread `...props`. Otherwise event handlers and `data-state` attributes are silently lost and the component appears broken.

**Note:** This rule applies to Radix-based shadcn components. If your project uses Base UI (via `npx shadcn init --base base-ui`), Base UI uses a different composition model — check the Base UI docs for equivalent patterns.

### 5. Storybook Stories Must Be Stateful
Never use static `args` for controlled value props (checked, value, selected). Always use `render` functions with `React.useState`. Static args don't update on interaction — components appear broken.

### 6. Input Components Must Be Unified
All input-type components (input, select, textarea, autocomplete, date pickers, search, phone number, secure input, text mask) must share identical: height (48px default), font-size (16px), border-radius (8px), border-color, placeholder color, focus ring style, label style (14px), helper text style (12px), disabled style. Inconsistency is the #1 signal of amateur design systems.

### 7. SVG Icons Use `currentColor`
Never hardcode `stroke="#242424"` or `fill="#767676"` in SVG icons. Always use `stroke="currentColor"` / `fill="currentColor"` so icons adapt to dark mode and parent color context.

### 8. AAA Contrast Ratios
Placeholder text must achieve 7:1 contrast ratio (WCAG AAA). `#767676` on white is only 4.54:1 (AA). Use `#595959` for AAA compliance. Set this as the `--muted-foreground` CSS variable.

### 9. Canvas Drawing Must Read Theme
For canvas-based components (signature pads, charts), read the foreground color from CSS at draw time:
```javascript
ctx.strokeStyle = getComputedStyle(document.documentElement).getPropertyValue('--foreground').trim();
```

### 10. Exclude Non-Library Code from TypeScript
If legacy code or unrelated files cause build failures, exclude them in `tsconfig.json` rather than modifying them:
```json
"exclude": ["node_modules", "src/lib/_elements"]
```

### 11. Always Use `get_design_context` Before `use_figma`
`get_design_context` is the primary extraction tool — it returns structured layout, typography, colors, and tokens. Use `use_figma` plugin API scripts only when `get_design_context` doesn't provide enough detail (e.g., extracting fills from deeply nested nodes or reading component property definitions). For standard extraction of layout, typography, colors, and spacing, `get_design_context` is faster, more reliable, and returns pre-structured data.

### 12. Search for Existing Design System Components
Before creating any component from scratch, call `search_design_system` to check if the Figma file already has reusable components. Import and extend matches instead of duplicating them. If the Figma file has a design system library, use it as the source of truth for tokens, patterns, and component structure.

### 13. Wire Fonts Through `next/font` AND `@theme inline`
`next/font/google` injects a CSS variable at runtime (e.g., `--font-geist-sans`). But Tailwind v4's `@theme inline` resolves at **parse time**, not runtime — so `--font-sans: var(--font-geist-sans)` won't work. Instead, use literal font names in `@theme inline`:
```css
@theme inline {
  --font-sans: 'YourFont', ui-sans-serif, system-ui, sans-serif;
}
```
Move the `next/font` variable className to `<html>`, not `<body>`. And load fonts separately in Storybook via `.storybook/preview-head.html` since `next/font` doesn't run there.

---

## Common Mistakes to Avoid

See [lessons-learned.md](lessons-learned.md) before starting any new component. Key mistakes that cost significant time in real projects:

1. **Figma colors inverted** — Avatar/Badge colors were used as background instead of text (and vice versa)
2. **Wrong component structure** — Alert had left-border stripe when Figma used full-background tint
3. **Accordion as separator list** — Figma used bordered panels with rounded corners, not border-bottom separators
4. **`rounded-lg` resolving to wrong value** — Always use explicit `rounded-[8px]` instead of relying on theme aliases
5. **DatePicker not functional** — Controlled/uncontrolled mode mismatch caused clicks to not update the input
6. **Portal content not inheriting dark mode** — Radix portals append to `document.body` but `.dark` class on `<html>` propagates via `&:is(.dark *)`

---

## Quick Start

```bash
# 1. Install the skill
npx skills add codefunded/figma-pixel-perfect

# 2. Ensure Figma MCP is configured in your IDE

# 3. Paste your Figma URL into your AI editor and say:
# "Build a design system component library from this Figma file"
```

## Dependencies

- **Next.js** (latest via `create-next-app@latest`) (scaffold framework)
- **Tailwind CSS** >= 4 (styling)
- **shadcn/ui** (component primitives)
- **Radix UI or Base UI** (accessibility primitives — shadcn supports both)
- **class-variance-authority** (variant management)
- **Storybook** >= 10 (documentation)
- **Figma MCP** (design extraction)
