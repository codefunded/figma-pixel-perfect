---
name: figma-pixel-perfect
version: 1.0.0
description: Use when the user wants to generate a pixel-perfect design system npm library from a Figma file. Extracts theme tokens, colors, typography, and spacing via Figma MCP, scaffolds React components on shadcn/ui + Tailwind CSS + Radix UI, ensures WCAG AAA accessibility, supports light and dark mode, and generates Storybook documentation. Triggers on phrases like "build design system from Figma", "extract Figma theme", "generate component library from Figma", "create pixel-perfect components", "Figma to React", "design system from Figma file".
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
Run a test call to confirm the Figma MCP is connected:
```
use_figma({ fileKey: "<key>", code: "return figma.root.children.map(p => p.name)" })
```
If this fails, instruct the user to:
- Open Figma Desktop app
- Ensure the file is open
- Check MCP server connection in their IDE settings

### 2. Node.js >= 20
```bash
node --version  # Must be >= 20.9.0
```
If using nvm: `nvm use 22` (or latest LTS)

### 3. Required Global Tools
- `npm` or `pnpm`
- `npx` (for shadcn CLI and create-next-app)

---

## Pipeline Overview

### Phase 1: Extract (Figma Analysis)
See [workflow.md §1 — Extract](#phase-1-extract)

### Phase 2: Scaffold (Project Setup)
See [workflow.md §2 — Scaffold](#phase-2-scaffold)

### Phase 3: Generate Components
See [component-patterns.md](component-patterns.md)

### Phase 4: Verify Pixel Fidelity
See [workflow.md §4 — Verify](#phase-4-verify-pixel-fidelity)

### Phase 5: Dark Mode & Accessibility
See [accessibility-rules.md](accessibility-rules.md)

### Phase 6: Storybook & Publish
See [storybook-conventions.md](storybook-conventions.md)

---

## Critical Rules (Always Apply)

### 1. Never Trust Generated Output — Always Verify
After building any component, inspect it in Storybook using `preview_inspect` to compare computed CSS (height, font-size, font-weight, color, border-radius, padding) against the Figma extraction data. Screenshots alone are insufficient — JPEG compression obscures color differences.

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

# 3. Run with a Figma file URL
# In Claude Code, paste your Figma URL and say:
# "Build a design system component library from this Figma file"
```

## Dependencies

- **Next.js** >= 16 (scaffold framework)
- **Tailwind CSS** >= 4 (styling)
- **shadcn/ui** (component primitives)
- **Radix UI** (accessibility primitives, unified `radix-ui` package)
- **class-variance-authority** (variant management)
- **Storybook** >= 10 (documentation)
- **Figma MCP** (design extraction)
