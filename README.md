# figma-pixel-perfect

An AI coding skill that generates a pixel-perfect, accessible, dark-mode-ready React component library from any Figma file. Works with **Cursor**, **Claude Code**, **VS Code + GitHub Copilot**, and any AI editor supporting the [Agent Skills](https://agentskills.io) standard.

Give it a Figma URL — it extracts design tokens, scaffolds a Next.js + Tailwind + shadcn/ui project, generates every component with Storybook docs, verifies pixel fidelity, and outputs an npm-publishable package with WCAG AAA accessibility.

Works for both **standalone npm packages** and **app-local design systems** — you choose the scope.

## Prerequisites

- **Figma MCP** — a Figma MCP server running and connected in your IDE.
- **Node.js 20+** — `nvm install 22` if needed.

## Install

```bash
npx skills add codefunded/figma-pixel-perfect
```

Target a specific agent:

```bash
npx skills add codefunded/figma-pixel-perfect -a cursor
npx skills add codefunded/figma-pixel-perfect -a claude-code
npx skills add codefunded/figma-pixel-perfect -a copilot
```

Install globally (available across all projects):

```bash
npx skills add codefunded/figma-pixel-perfect -g
```

## Usage

1. Grab your Figma file URL
2. Paste it into your AI editor and ask:

```
Build a design system from this Figma file:
https://www.figma.com/design/ABC123xyz/My-Design-System
```

The skill will:
1. Extract design tokens (colors, typography, spacing, radii)
2. Inventory components and their variant matrices
3. Scaffold a project with Tailwind CSS and shadcn/ui
4. Generate every component with pixel-perfect fidelity
5. Create Storybook stories with interactive controls
6. Verify against Figma using computed CSS inspection
7. Apply dark mode and WCAG AAA accessibility

## What Gets Generated

```
my-design-system/
├── .storybook/               # Storybook config + dark mode toggle
├── src/
│   ├── app/globals.css       # CSS variables (light + dark themes)
│   ├── components/ui/        # All components + stories (co-located)
│   ├── lib/utils.ts          # cn() utility
│   └── index.ts              # Barrel exports for npm
├── package.json              # npm-ready with exports + peerDependencies
└── tsconfig.json
```

## Customize

**Theme** — edit CSS variables in `src/app/globals.css` (`:root` for light, `.dark` for dark)

**Variants** — each component uses CVA. Edit the `variants` object in any component file.

**Fonts** — change in `globals.css` (`@theme inline { --font-sans }`) and `.storybook/preview-head.html`

## Publish to npm

The generated library ships TypeScript source files. Consumers need a compatible bundler (Next.js, Vite, etc.) to import them directly. For broader compatibility, add a build step:

```bash
# Option A: Ship TypeScript source (consumers must support .tsx imports)
npm publish --access public

# Option B: Add a build step for compiled output
npm install -D tsup
npx tsup src/index.ts --format esm,cjs --dts
# Then update package.json "exports" to point to dist/
```

Consumers install with:
```tsx
import { Button, Card, Input } from "@yourorg/design-system";
import "@yourorg/design-system/globals.css";
```

## Update

```bash
npx skills update
```

## Manage

```bash
npx skills list                        # see installed skills
npx skills check                       # check for updates
npx skills remove figma-pixel-perfect  # uninstall
```

## Reference Docs

| Document | What's inside |
|---|---|
| [design-system-patterns.md](design-system-patterns.md) | Industry-standard design system patterns and fallback defaults |
| [SKILL.md](SKILL.md) | Trigger conditions, prerequisites, 12 critical rules |
| [workflow.md](workflow.md) | 7-phase pipeline: extract → scaffold → generate → verify → a11y → finalize → DESIGN.md |
| [component-patterns.md](component-patterns.md) | Component templates, CVA mapping, Radix primitives, unified input standard |
| [accessibility-rules.md](accessibility-rules.md) | WCAG AAA, contrast ratios, ARIA, keyboard nav, reduced motion |
| [storybook-conventions.md](storybook-conventions.md) | Story format, stateful patterns, dark mode toggle |
| [design-md-generation.md](design-md-generation.md) | Generate DESIGN.md — the agent-readable design system standard |
| [lessons-learned.md](lessons-learned.md) | Common mistakes and how to avoid them |
| [CHANGELOG.md](CHANGELOG.md) | Version history |

## Contributing

1. Fork → branch → PR
2. New lessons follow this format in `lessons-learned.md`:

```markdown
## Lesson: [Short Name]
**What goes wrong:** ...
**Why it happens:** ...
**Correct approach:** ...
**Detection:** ...
```

## License

MIT
