# Lessons Learned

Hard-won knowledge from real Figma-to-design-system projects. Read this before generating any component — it will save hours of debugging.

These lessons are organized by category. Some are universal (apply to any Figma file), while others surface only with certain design patterns. Skim the whole document when starting a new project, then reference specific sections when building each component type.

---

## Category: Figma Extraction

### Lesson: Never Assume Component Structure — Always Extract First

**What goes wrong:** Building a component based on what you think it looks like (or what "standard" design systems do) rather than what the Figma file actually contains. For example, building an Alert with a left-border accent stripe when the Figma design uses a full-background tint with a large icon.

**Why it happens:** Design system conventions vary wildly. What's "standard" in Material UI is different from Ant Design, Atlassian, or a bespoke enterprise system. Agents trained on common patterns default to the most popular structure.

**Correct approach:** Before writing any component code, ALWAYS:
1. Get a screenshot of the Figma component (`get_screenshot`)
2. Extract the component set's variant properties (`componentPropertyDefinitions`)
3. Extract the inner node structure — fills, strokes, padding, corner radius, layout mode, text styles
4. Only then decide on the implementation structure

**Detection:** If you're writing component code without having first called `use_figma` or `get_screenshot` for that specific component, you're guessing.

---

### Lesson: Color Roles Can Be Inverted

**What goes wrong:** Figma extraction returns color values, but which one is the background vs. foreground is ambiguous. A common mistake: using a dark extracted color as the background with white text, when the design actually uses a light tint as background with the dark color as text.

**Why it happens:** Figma nodes have multiple fills. The extraction returns hex values without semantic labels. Avatars, badges, and status indicators are especially prone to this — they often use light pastel backgrounds with dark colored text, not dark backgrounds with white text.

**Correct approach:** Look at the Figma screenshot first. Then when extracting colors from a component:
- The outermost frame fill = container/card background
- The inner shape fill = the component's background (often a light tint)
- The text node fill = the text color (often a darker shade of the same hue)
- If in doubt, check the luminance — backgrounds are typically lighter than text

**Detection:** Visual comparison immediately reveals inverted colors. Dark circles with light text vs light circles with dark text are visually obvious.

---

### Lesson: Figma Components May Not Be Organized as a Design System

**What goes wrong:** The skill assumes the Figma file has neatly organized component sets with clear variant properties. In reality, many Figma files are just mockup pages with no reusable components.

**Why it happens:** Users send Figma files that are app mockups, not design system files. Components are embedded inline, duplicated with slight variations, or not componentized at all.

**Correct approach:** 
1. First, check if the file has a dedicated "Components" or "Design System" page — look at `figma.root.children` for page names
2. Search for `COMPONENT_SET` nodes — these are organized components with variants
3. If no component sets exist, scan the full mockup for repeated patterns: buttons, inputs, cards, headers, etc.
4. Extract the common patterns (shared colors, fonts, spacing) to build your own token system
5. Identify unique instances vs reusable patterns — if a button appears in 10 places with the same style, it's a component even if Figma doesn't mark it as one

**Detection:** `figma.root.children` returns only pages like "Home", "Dashboard", "Settings" — no "Components" or "Design System" page. Component set searches return empty results.

---

### Lesson: Used `use_figma` When `get_design_context` Would Have Been Faster

**What goes wrong:** Writing custom plugin API scripts to extract properties that `get_design_context` already returns in structured form.

**Why it happens:** `use_figma` feels more powerful and flexible, so agents default to it. But for standard extraction (layout, typography, colors, spacing), `get_design_context` is faster, more reliable, and returns pre-structured data.

**Correct approach:** Always try `get_design_context` first. If the response is truncated or missing specific details, then use `use_figma` for targeted extraction.

**Detection:** You're writing a multi-line `use_figma` script to extract fills, fonts, and padding — all of which `get_design_context` already returns.

---

### Lesson: Ignored Existing Design System Components in Figma

**What goes wrong:** Building components from scratch when the Figma file already has a design system with reusable components that could be imported.

**Why it happens:** The agent jumps straight into building without checking what's already available.

**Correct approach:** Call `search_design_system` before creating any component. If matches exist, use them as the source of truth for tokens, patterns, and structure.

**Detection:** The Figma file has a "Components" page or design system library, and you haven't called `search_design_system`.

---

### Lesson: Extract the Right Node — Not Its Parent

**What goes wrong:** Getting wrong dimensions/radius because the extraction targeted a wrapper frame instead of the actual component element. For example, extracting border-radius from an input's container frame (10px) when the input field itself has 8px.

**Why it happens:** Figma's node hierarchy often has wrapper frames around the actual component. The wrapper may have different padding, radius, or dimensions than the inner element.

**Correct approach:** When extracting properties, drill into the children until you reach the actual visual element. For inputs: the container frame → the input wrapper → the actual input rectangle. For buttons: the component frame → the inner auto-layout frame with the fill.

**Detection:** Computed CSS in Storybook doesn't match your Figma extraction. Re-extract targeting the inner node.

---

### Lesson: Figma MCP Can Fail Intermittently

**What goes wrong:** `get_screenshot` or `use_figma` calls fail with `net::ERR_FAILED`.

**Why it happens:** Network connectivity issues, Figma MCP server not running, invalid file key, or temporary API issues.

**Correct approach:** 
- Retry once after a short pause
- If screenshots fail, fall back to `use_figma` plugin API for data extraction (more reliable)
- If `use_figma` also fails, ask the user to verify the Figma MCP server is running and the file URL is valid
- Cache extracted data so you don't need to re-fetch on every iteration

**Detection:** Errors containing `net::ERR_FAILED`, `502`, or timeout messages.

---

## Category: Tailwind CSS v4

### Lesson: `font-[]` Arbitrary Value Conflicts with `font-medium`

Use `[font-family:'FontName',sans-serif]` (arbitrary property syntax), never `font-['FontName',sans-serif]` (arbitrary value syntax). See [SKILL.md Rule 3](SKILL.md) for details.

**Detection:** Inspect computed `font-family` or `font-weight` — one will be wrong. This is a SYSTEMIC issue affecting every file — if you find it in one component, grep for it across all.

---

### Lesson: `rounded-lg` May Not Be What You Think

**What goes wrong:** Using `rounded-lg` when the Figma design specifies 8px radius, but the Tailwind theme maps `rounded-lg` to 10px (or some other value).

**Why it happens:** `rounded-lg` resolves via the Tailwind theme's `--radius-lg` variable. If you've customized the theme, it may not match the Figma value. Or the default shadcn theme sets it differently.

**Correct approach:** Always use explicit pixel values: `rounded-[8px]`, `rounded-[10px]`, `rounded-[16px]`. Never rely on named radius utilities for design-system-critical values.

**Detection:** `preview_inspect` shows a different border-radius than expected.

---

### Lesson: Never Hardcode Hex Colors

Every color must use a CSS variable. Hardcoded hex values break dark mode. See [SKILL.md Rule 2](SKILL.md) for details.

**Detection:** Toggle dark mode — any element that stays the same color has a hardcoded value. Audit with:
```bash
grep -rn "bg-\[#\|text-\[#\|border-\[#" src/components/ui/
```

---

### Lesson: SVG Icons Must Use `currentColor`

Never hardcode `stroke` or `fill` hex values in SVGs — use `currentColor` so icons adapt to dark mode. See [SKILL.md Rule 7](SKILL.md) for details.

**Detection:** Dark mode toggle makes icons invisible. Audit with:
```bash
grep -rn 'stroke="#\|fill="#' src/components/ui/
```

---

## Category: Radix UI / shadcn Integration

### Lesson: `asChild` Requires `forwardRef` on the Child

**What goes wrong:** Radix trigger components (DropdownMenuTrigger, DialogTrigger, PopoverTrigger) with `asChild` don't work — clicking does nothing, no errors in console.

**Why it happens:** `asChild` merges the trigger's props (including `ref`, event handlers, `data-state`, `aria-*`) into its single child element. If the child doesn't accept a ref (no `forwardRef`) or doesn't spread `...props`, these are silently dropped.

**Correct approach:** ANY component used as a child of a Radix `asChild` trigger MUST:
1. Use `React.forwardRef`
2. Spread `...props` onto its root element
3. Pass `ref` to the root element
4. Have a `displayName`

```tsx
const TriggerButton = React.forwardRef<HTMLButtonElement, React.ButtonHTMLAttributes<HTMLButtonElement>>(
  ({ children, ...props }, ref) => (
    <button ref={ref} {...props}>{children}</button>
  )
);
TriggerButton.displayName = "TriggerButton";
```

**Detection:** Click a trigger — nothing happens. No console errors. Check if the child component uses `forwardRef`.

---

### Lesson: Radix Portals Inherit Dark Mode If Configured Correctly

**What goes wrong:** Dropdown menus, popovers, and dialogs rendered via Radix Portals appear in light mode even when the app is in dark mode.

**Why it happens:** Portals render at `document.body` level. If dark mode is set via a class on a specific container (not `<html>`), portal content won't inherit it.

**Correct approach:** Set the `.dark` class on `<html>` (not `<body>` or a wrapper div). Use the Tailwind dark mode selector `@custom-variant dark (&:is(.dark *))` which matches any element inside a `.dark` ancestor — including portal content appended to body.

**Detection:** Open a dropdown in dark mode — the popup is white while the background is dark.

---

### Lesson: Don't Override shadcn Defaults Without Checking Figma

**What goes wrong:** Using shadcn's default component structure (e.g., accordion with border-bottom separators) when the Figma design uses a completely different pattern (e.g., bordered panels with rounded corners).

**Why it happens:** shadcn `add` installs a component with sensible defaults. It's tempting to just customize colors/fonts. But the structural layout may be fundamentally different from the Figma design.

**Correct approach:** After `npx shadcn add <component>`, always compare the generated structure against the Figma design. If the structure differs (e.g., separator list vs card panels), rewrite the component from scratch using Radix primitives directly rather than modifying the shadcn output.

**Detection:** Visual comparison shows a fundamentally different layout, not just different colors.

---

## Category: Storybook

### Lesson: Controlled Components Need Stateful Stories

**What goes wrong:** Interactive components (checkboxes, inputs, toggles, date pickers) don't respond to clicks/input in Storybook.

**Why it happens:** Stories use `args: { checked: false, value: "text" }` which creates static props. Storybook args don't automatically create state management — clicking a checkbox calls `onCheckedChange` but no state updates, so the UI doesn't change.

**Correct approach:** For ANY component with controlled state (`value`, `checked`, `selected`, `open`), use a `render` function with `React.useState`:

```tsx
export const Default: Story = {
  render: function DefaultStory() {
    const [checked, setChecked] = React.useState(false);
    return <Checkbox checked={checked} onCheckedChange={setChecked} label="Accept terms" />;
  },
};
```

**Detection:** Click an interactive element — nothing visually changes. Check if the story uses `args` for value/state props.

---

### Lesson: Never Import from `@storybook/test`

**What goes wrong:** Build fails with `Cannot find module '@storybook/test'`.

**Why it happens:** Some generated stories import `{ fn } from "@storybook/test"` for action mocking. This package isn't always installed.

**Correct approach:** Use plain `() => {}` for callback stubs instead of `fn()`. Or install `@storybook/test` if action logging is needed.

**Detection:** Build error mentioning `@storybook/test`. Grep for it: `grep -rn "@storybook/test" src/`.

---

### Lesson: Storybook Doesn't Load `next/font`

**What goes wrong:** Components in Storybook render in the browser's default font instead of the design system font.

**Why it happens:** `next/font/google` only works within the Next.js runtime. Storybook runs independently and cannot execute Next.js font optimization.

**Correct approach:** Load fonts in Storybook via `.storybook/preview-head.html`:
```html
<link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap" rel="stylesheet" />
```
And set the font-family in `globals.css` via `@theme inline { --font-sans: 'Roboto', sans-serif; }`.

**Detection:** Text appears in Times New Roman or system font. Check if `.storybook/preview-head.html` exists and includes a font `<link>` tag.

---

## Category: Input Component Consistency

### Lesson: All Input Types Must Share the Same Visual DNA

**What goes wrong:** Input, Select, Autocomplete, DatePicker, PhoneNumber, TextMask, and Textarea all look slightly different — different heights, border radii, font sizes, placeholder colors, focus rings.

**Why it happens:** Each component was built independently, often by different agents or at different times, and each made slightly different styling decisions.

**Correct approach:** Define a unified input standard ONCE and apply it to ALL input-type components:
- Height: 48px (default/lg size)
- Font-size: 16px
- Border-radius: 8px (explicit `rounded-[8px]`)
- Border: `border-border` (CSS variable)
- Placeholder: `text-muted-foreground`
- Focus: `focus:border-primary focus:ring-2 focus:ring-primary/20`
- Error: `border-destructive focus:border-destructive focus:ring-destructive/20`
- Label: 14px, `text-label`
- Helper: 12px, `text-muted-foreground`
- Disabled: `opacity-50 pointer-events-none`

After building, audit ALL input components in a single Storybook view to catch inconsistencies.

**Detection:** Place all input types side by side in one story — inconsistent heights, radii, or focus styles are immediately visible.

---

## Category: Accessibility

### Lesson: Placeholder Contrast Must Be AAA

**What goes wrong:** Placeholder text uses `#767676` which passes WCAG AA (4.54:1) but fails AAA (needs 7:1).

**Why it happens:** `#767676` is widely cited as the "minimum AA gray" and used by default in many design systems.

**Correct approach:** Use `#595959` on white backgrounds (7.06:1 ratio) for AAA compliance. Set this as `--muted-foreground` in the light theme. For dark mode, ensure the equivalent passes on the dark background.

**Detection:** Run a contrast ratio checker. Any ratio between 4.5:1 and 7:1 passes AA but fails AAA.

---

### Lesson: Canvas Elements Don't Inherit CSS Colors

**What goes wrong:** Signature pads, drawing canvases, or chart components use hardcoded colors that don't adapt to dark mode.

**Why it happens:** The HTML Canvas API doesn't use CSS — colors are set imperatively via `ctx.strokeStyle = "#242424"`. CSS variables and Tailwind classes have no effect on canvas drawing.

**Correct approach:** Read CSS custom properties at draw time:
```typescript
const fg = getComputedStyle(document.documentElement).getPropertyValue('--foreground').trim();
ctx.strokeStyle = fg || '#242424';
```

**Detection:** Toggle dark mode — canvas content remains in light-mode colors.

---

### Lesson: Add `prefers-reduced-motion` Support

**What goes wrong:** Users with vestibular disorders experience discomfort from animations.

**Why it happens:** No one remembers to add reduced motion support until it's flagged in an accessibility audit.

**Correct approach:** Add this to `globals.css`:
```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

**Detection:** Enable "Reduce motion" in OS accessibility settings — animations should stop.

---

## Category: Build & Tooling

### Lesson: Check Node.js Version First

**What goes wrong:** Cryptic build failures, dependency installation errors, or runtime crashes.

**Why it happens:** The system has an old Node.js version (16 or 18) that doesn't support modern APIs required by Next.js 16+, Storybook 10+, and current npm packages.

**Correct approach:** Before any project setup:
```bash
node --version  # Must be >= 20.9.0
```
If not, use nvm: `nvm install 22 && nvm use 22`. Add `.nvmrc` with `22`.

**Detection:** Errors mentioning `ERR_UNSUPPORTED_DIR_IMPORT`, missing APIs, or `engines.node` requirements.

---

### Lesson: esbuild Architecture Mismatch

**What goes wrong:** Storybook fails to start with "installed for another platform" error.

**Why it happens:** `node_modules` was installed under a different CPU architecture (e.g., arm64 vs x64 via Rosetta on macOS).

**Correct approach:** Clean install:
```bash
rm -rf node_modules package-lock.json
npm install
```
If the user consistently uses Rosetta, install the alternate esbuild binary: `npm install @esbuild/darwin-x64 --save-optional`.

**Detection:** Error explicitly mentions esbuild platform mismatch.

---

### Lesson: Exclude Unrelated Code from TypeScript

**What goes wrong:** Build fails due to type errors in legacy code, test files, or unrelated directories.

**Why it happens:** `tsconfig.json` includes `**/*.ts` which catches everything — including legacy libraries, test fixtures, or code that depends on uninstalled packages.

**Correct approach:** Add problematic directories to `tsconfig.json` exclude:
```json
"exclude": ["node_modules", "src/lib/_legacy", "**/*.test.ts"]
```

**Detection:** Build error points to a file you didn't write or modify.

---

### Lesson: Barrel Export References Module Before It Exists
**What goes wrong:** `src/index.ts` imports from `./components/button` before the component file is created. TypeScript fails immediately.
**Why it happens:** The scaffold step creates the barrel export with all planned component paths, but components are built later in Phase 3. The build runs between phases and fails.
**Correct approach:** Create the barrel export with ONLY the components that exist. Add exports incrementally as each component is built. Never reference a module that doesn't exist yet.
**Detection:** `tsc` errors like `Cannot find module './components/button'` on a file you haven't created yet.

---

### Lesson: Tailwind CSS CLI Not on PATH
**What goes wrong:** `npm run build` fails with `tailwindcss: command not found` even though `tailwindcss` is installed.
**Why it happens:** The `tailwindcss` npm package is the library, not the CLI binary. The CLI is in `@tailwindcss/cli`. Without it, the `tailwindcss` command isn't available in npm scripts.
**Correct approach:** Install both: `npm install tailwindcss @tailwindcss/cli`. Or use `npx tailwindcss` in scripts instead of bare `tailwindcss`.
**Detection:** Build script fails with `command not found` for tailwindcss. Check if `@tailwindcss/cli` is in devDependencies.

---

### Lesson: package.json Exports Types Must Come First
**What goes wrong:** TypeScript consumers can't resolve types from the published package. Bundlers warn that the `"types"` condition is unreachable.
**Why it happens:** The `"types"` field is placed after `"import"` or `"require"` in the exports map. Bundlers evaluate conditions top-to-bottom and stop at the first match, so `"types"` never gets reached.
**Correct approach:** Always put `"types"` first in the exports condition order:
```json
"exports": {
  ".": {
    "types": "./dist/index.d.ts",
    "import": "./dist/index.mjs",
    "require": "./dist/index.js"
  }
}
```
**Detection:** TypeScript errors in consuming projects: `Could not find a declaration file for module`. Or bundler warnings about unreachable conditions.

---

### Lesson: Storybook Logs "use client" Warnings for Radix
**What goes wrong:** Storybook/Vite logs warnings about `"use client"` directives being ignored in Radix UI components.
**Why it happens:** `"use client"` is a React Server Components directive that only Next.js understands. Storybook uses Vite which doesn't process RSC directives. Radix components include `"use client"` at the top of every file.
**Correct approach:** Ignore these warnings — they are expected and harmless. Do not try to "fix" them by removing `"use client"` from your components or suppressing Vite warnings. Document this in your project README so teammates don't file bugs.
**Detection:** Console output shows `Module "..." has been externalized for browser compatibility...` or `"use client" was ignored`. These are safe to ignore.

---

### Lesson: Missing .gitignore in Scaffold
**What goes wrong:** `node_modules/`, `dist/`, and `storybook-static/` get committed to git.
**Why it happens:** The scaffold step creates project files but forgets the `.gitignore`. The agent doesn't think about git until the commit step.
**Correct approach:** Create `.gitignore` as one of the FIRST scaffold files, before `npm install`. Include: `node_modules/`, `dist/`, `.next/`, `storybook-static/`, `.DS_Store`, `*.tsbuildinfo`.
**Detection:** `git status` shows thousands of files in `node_modules/`.
