# figma-pixel-perfect

A Claude Code skill that generates a pixel-perfect, accessible, dark-mode-ready React component library from Figma designs. It extracts design tokens and component specifications via the Figma MCP, scaffolds a Next.js + Tailwind CSS + shadcn/ui project, generates every component with Storybook documentation, verifies pixel fidelity against the source design, and produces an npm-publishable package with WCAG AAA compliance.

## Prerequisites

- **Figma MCP** — The Figma MCP server must be connected to your IDE (Claude Code, VS Code with Claude, or Cursor). The Figma Desktop app must be running with the target file open.
- **Node.js 20+** — Required for Next.js 16+, Storybook 10+, and modern dependency resolution. Use `nvm install 22` if needed.
- **npm** — For package installation and publishing.

## Repository Structure

All skill files live at the **root** of this repository:

```
figma-pixel-perfect/          # Repository root = Skill root
├── SKILL.md                  # Entry point (Claude Code loads this)
├── README.md                 # This file
├── CHANGELOG.md              # Version history
├── workflow.md               # Pipeline reference
├── component-patterns.md     # Component templates
├── accessibility-rules.md    # WCAG AAA rules
├── storybook-conventions.md  # Storybook conventions
└── lessons-learned.md        # Common mistakes & corrections
```

## Installation

This skill follows the [Agent Skills](https://agentskills.io) open standard and can be installed via the `npx skills` CLI across all supported AI coding agents.

### Quick Install (Recommended)

```bash
# Install for all supported agents (Claude Code, Codex, OpenCode, etc.)
npx skills add codefunded/figma-pixel-perfect

# Install for a specific agent only
npx skills add codefunded/figma-pixel-perfect -a claude-code

# Install globally (available across all projects)
npx skills add codefunded/figma-pixel-perfect -g

# Preview what will be installed without installing
npx skills add codefunded/figma-pixel-perfect --list
```

### Managing Skills

```bash
# List installed skills
npx skills list

# Check for updates
npx skills check

# Update to latest version
npx skills update

# Remove the skill
npx skills remove figma-pixel-perfect
```

## Quick Start

1. Open your Figma file in Figma Desktop
2. Copy the file URL from the browser or Figma
3. In Claude Code, run:

```
Build a design system component library from this Figma file:
https://www.figma.com/design/ABC123xyz/My-Design-System
```

The skill will:
1. Extract all design tokens (colors, typography, spacing, radii)
2. Inventory all components and their variant matrices
3. Scaffold a Next.js project with Tailwind CSS and shadcn/ui
4. Generate every component with CVA variants matching Figma specs
5. Create Storybook stories for each component
6. Verify pixel fidelity using computed CSS inspection
7. Ensure dark mode and WCAG AAA accessibility

## What Gets Generated

```
my-design-system/
├── .storybook/
│   ├── main.ts              # Storybook configuration
│   ├── preview.ts            # Theme decorator, globals.css import
│   └── preview-head.html     # Google Fonts loading
├── src/
│   ├── app/
│   │   └── globals.css       # CSS variables (light + dark), @theme inline
│   ├── components/
│   │   └── ui/
│   │       ├── button.tsx           # Component
│   │       ├── button.stories.tsx   # Storybook stories
│   │       ├── card.tsx
│   │       ├── card.stories.tsx
│   │       ├── input.tsx
│   │       ├── input.stories.tsx
│   │       └── ...                  # All components from Figma
│   ├── lib/
│   │   ├── utils.ts          # cn() utility
│   │   └── tokens.ts         # Design token constants
│   └── index.ts              # Barrel exports
├── .nvmrc                    # Node.js version
├── components.json           # shadcn configuration
├── package.json              # npm package config with exports
├── tailwind.config.ts        # Tailwind configuration
└── tsconfig.json             # TypeScript configuration
```

## Customizing the Output

### Theme Colors

Edit `src/app/globals.css` to adjust CSS variables in both `:root` (light) and `.dark` (dark) sections. The `@theme inline` block maps CSS variables to Tailwind utility classes.

### Component Variants

Each component uses `class-variance-authority` (CVA) for variant management. Edit the `variants` object in any component file to add, remove, or modify variants.

### Adding New Components

1. Create `src/components/ui/my-component.tsx` following the template in `component-patterns.md`
2. Create `src/components/ui/my-component.stories.tsx` following `storybook-conventions.md`
3. Add the export to `src/index.ts`

### Fonts

Change the font in three places:
1. `.storybook/preview-head.html` — the Google Fonts `<link>` tag
2. `src/app/globals.css` — the `@theme inline { --font-sans: ... }` declaration
3. `src/app/layout.tsx` — the `next/font` import (for the Next.js app itself)

## Publishing to npm

1. Update `package.json` with your package name, version, and scope:
```json
{
  "name": "@yourorg/design-system",
  "version": "1.0.0"
}
```

2. Build and verify:
```bash
npx next build        # Verify compilation
npx storybook build   # Verify all stories
```

3. Publish:
```bash
npm login
npm publish --access public
```

4. In consuming projects:
```bash
npm install @yourorg/design-system
```

```tsx
import { Button, Card, Input } from "@yourorg/design-system";
import "@yourorg/design-system/globals.css";
```

## Updating the Skill

The skill uses **semantic versioning** (major.minor.patch). The current version is in the `version` field of [SKILL.md](SKILL.md) frontmatter and changes are documented in [CHANGELOG.md](CHANGELOG.md).

### Check Your Version

Look at the `version` field in your local `SKILL.md`:
```bash
head -5 SKILL.md  # or wherever you installed the skill
```

### Update

```bash
npx skills update
```

### Version Policy

| Change Type | Version Bump | Examples |
|-------------|-------------|---------|
| New lessons, improved docs, typo fixes | Patch (1.0.x) | Better error detection tips, new Figma gotchas |
| New component patterns, new workflow phases, new platform support | Minor (1.x.0) | Vue/Svelte support, Figma variables extraction, new accessibility checks |
| Breaking changes to skill structure or workflow | Major (x.0.0) | Restructured SKILL.md, changed pipeline phases, removed files |

## Reference Documentation

| Document | Purpose |
|---|---|
| [SKILL.md](SKILL.md) | Entry point — trigger conditions, prerequisites, critical rules, pipeline overview |
| [workflow.md](workflow.md) | End-to-end pipeline (extract, scaffold, generate, verify, finalize) |
| [component-patterns.md](component-patterns.md) | Component templates, CVA mapping, Radix usage, form input standard |
| [accessibility-rules.md](accessibility-rules.md) | WCAG AAA compliance, contrast ratios, ARIA patterns, keyboard nav |
| [storybook-conventions.md](storybook-conventions.md) | Story format, stateful patterns, dark mode setup, font loading |
| [lessons-learned.md](lessons-learned.md) | Common mistakes and corrections from real projects |
| [CHANGELOG.md](CHANGELOG.md) | Version history and release notes |

## Contributing

1. Fork the repo
2. Create a feature branch (`git checkout -b feature/new-lesson`)
3. Add your changes (new lessons, improved patterns, bug fixes)
4. Submit a PR with a clear description of what changed and why

New lessons for `lessons-learned.md` should follow this format:
```markdown
## Lesson: [Short Name]
**What goes wrong:** ...
**Why it happens:** ...
**Correct approach:** ...
**Detection:** ...
```

## License

MIT