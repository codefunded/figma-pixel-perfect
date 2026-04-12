# Changelog

All notable changes to the **figma-pixel-perfect** skill will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.0] - 2026-04-12

### Added
- Design system patterns reference (design-system-patterns.md) — industry-standard fallback defaults
- Quality gates woven into token extraction (Phase 1), component building (Phase 3), and verification (Phase 4)
- Base UI support notes (shadcn supports both Radix and Base UI)
- Multi-tool Figma MCP prerequisite check (get_design_context, get_screenshot, use_figma)
- Real-world lessons: barrel export ordering, Tailwind CLI, package.json exports, Storybook "use client" warnings, .gitignore scaffold

## [1.1.0] - 2026-04-11

### Added
- DESIGN.md generation (Phase 7) — produces an agent-readable design system file following the Google Stitch standard
- Figma MCP Tools Reference table in SKILL.md — documents all 6 MCP tools and clarifies browser preview tools
- `get_design_context` as primary extraction tool (Rule 11)
- `search_design_system` for reusing existing Figma components (Rule 12)
- Asset handling guidance for Figma MCP localhost URLs
- Pixel fidelity validation checklist (7-point)
- `create_design_system_rules` tool reference for generating project-level AI agent rules
- Two new lessons: "Used use_figma when get_design_context would have been faster" and "Ignored existing design system components"


## [1.0.0] - 2026-04-09

### Added
- Initial release