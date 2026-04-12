# Universal Design System Patterns

Findings from analyzing 60 real brand design systems (Stripe, Linear, Vercel, Apple, Airbnb, Spotify, Notion, etc.). Use these patterns as the quality bar when extracting and building from Figma or Stitch.

## Color Architecture

Every quality design system follows this structure:

### Mandatory Color Roles
| Role | Description | Example |
|------|------------|---------|
| Primary text | Near-black, NOT pure #000000 — add warmth with navy/brown undertone | Stripe #061b31, Airbnb #222222, Cursor #26251e |
| Secondary text | Mid-gray for descriptions | #64748d - #8a8f98 range |
| Muted text | Light gray for captions, placeholders | #929090 - #a1a1aa range |
| Primary accent | ONE dominant chromatic color for CTAs and interactive elements | Brand blue, purple, green, etc. |
| Accent hover | Darker shade of primary accent | 10-15% darker |
| Background | Primary canvas | #ffffff or near-white |
| Surface | Elevated surface (cards, panels) | #f7f7f7 - #fafafa range |
| Border default | Subtle dividers | #e5edf5 - #e8e8e8 range |
| Border subtle | Very light borders | rgba(0,0,0,0.05-0.1) |
| Focus ring | Accessibility focus indicator | Usually matches primary accent |
| Success/Warning/Error | Semantic status colors | Green/Orange/Red |

### The "Not Black" Rule
11 out of 12 analyzed systems avoid pure #000000 for text. Use a slightly warm near-black with an intentional undertone:
- Navy undertone: #061b31 (Stripe), #0a0a0a (Vercel)
- Warm undertone: #222222 (Airbnb), #26251e (Cursor)
- Cool undertone: #191c1f (Revolut), #1d1d1f (Apple)

### Single Accent Discipline
10 out of 12 systems use ONE primary chromatic accent. Additional colors are for:
- Semantic status (success/warning/error) — NOT brand expression
- Decorative gradients — NOT interactive elements
- Tier differentiation (Airbnb's Purple for Luxe) — NOT general UI

## Typography Architecture

### Universal Hierarchy (12/12 systems)
| Tier | Size Range | Weight | Tracking | Present In |
|------|-----------|--------|----------|------------|
| Display/Hero | 48-136px | Light-Medium (300-500) | Negative (-0.5 to -2.8px) | 12/12 |
| Section Heading | 32-48px | Medium (400-500) | Negative (-0.3 to -1px) | 12/12 |
| Sub-heading | 20-26px | Medium (400-500) | Slight negative | 12/12 |
| Body Large | 18-20px | Regular (400) | Normal | 11/12 |
| Body | 16px | Regular (400) | Normal | 12/12 — universal baseline |
| Button/Nav/UI | 14-16px | Medium (500) | Normal | 12/12 |
| Caption | 12-14px | Regular (400) | Normal | 12/12 |
| Micro/Badge | 10-12px | Regular-Medium | Normal to slight positive | 11/12 |
| Code/Mono | 12-14px | Medium (500) | Normal | 11/12 |

### Key Insights
- **16px is the universal body text size** — never deviate
- **Negative tracking at display sizes is universal** — tighten headlines progressively
- **Weight restraint** — most systems use only 3-4 weight stops (300/400/500/600 or 400/500/600/700)
- **Monospace companion** — 11/12 systems have a dedicated monospace font for code/data
- **OpenType features matter** — 10/12 systems enable specific OT features (ss01, cv01, liga, kern) that define their typographic personality

## Border Radius Scale

### Common Pattern
| Use | Radius | Frequency |
|-----|--------|-----------|
| Inputs, small buttons | 4-8px | 12/12 |
| Cards, containers | 8-16px | 12/12 |
| Large featured cards | 16-24px | 8/12 |
| Pills, badges, tags | 9999px | 11/12 |
| Avatars, icon buttons | 50% (circle) | 10/12 |

**Key insight:** Extract the EXACT radius from the design — don't assume. Stripe uses 4px everywhere; Airbnb uses 20px for cards. These are brand-defining choices.

## Shadow/Elevation System

### Universal 4-Level Pattern
| Level | Description | Opacity Range | Frequency |
|-------|------------|---------------|-----------|
| Flat | No shadow | 0 | 12/12 |
| Subtle | Whisper border or ring | 0.02-0.08 | 10/12 |
| Standard | Card-level elevation | 0.08-0.15 | 10/12 |
| Elevated | Modal/dropdown | 0.15-0.25 | 9/12 |
| Focus | Accessibility ring | Varies | 12/12 |

**Quality signals in shadows:**
- Multi-layer stacks (2-3 shadow values per level) — Stripe, Vercel, Cal
- Brand-colored shadows (blue-tinted at rgba(50,50,93,0.25)) — Stripe
- Shadow-as-border technique (`0 0 0 1px rgba(...)`) — Vercel, Cal
- Zero shadows (depth through color contrast only) — Revolut, Supabase

## Component Checklist

### Tier 1: Must Have (every design system needs these)
- [ ] Button — primary (filled), secondary (ghost/outlined), tertiary (text/link)
- [ ] Card — standard, elevated, with image
- [ ] Navigation bar — desktop horizontal, mobile hamburger, sticky behavior
- [ ] Typography components — or at minimum, a consistent type scale
- [ ] Link — inline and standalone variants

### Tier 2: Expected (9-11/12 systems have these)
- [ ] Badge/Pill — status, tag, feature label
- [ ] Input/Form field — text, search, select with focus/error states
- [ ] Icon button — circular and square
- [ ] Divider — horizontal rule, section border
- [ ] Avatar — with image, initials, and online indicator

### Tier 3: Common (6-8/12 systems)
- [ ] Dropdown/Menu — product dropdown, context menu
- [ ] Tab navigation — pill tabs, underline tabs
- [ ] Modal/Dialog — center dialog, side drawer
- [ ] Metric/Stat card — large number + description
- [ ] Code block — inline and block
- [ ] Alert/Toast — success, warning, error, info
- [ ] Accordion — bordered panels or separator style
- [ ] Table — with header, striped rows, sort indicators

## Universal Anti-Patterns

Things that EVERY quality design system warns against:

1. **Never use pure #000000 for body text** — always add warmth
2. **Never introduce extra brand colors** — respect the accent budget
3. **Never use bold (700) for headlines where the system uses light/medium** — weight is a brand choice
4. **Never skip OpenType features** — they're identity-defining, not decorative
5. **Never use positive letter-spacing at display sizes** — headlines run tight
6. **Never use the brand accent for decoration** — accent is for interactive elements only
7. **Never use heavy shadows on light themes** — keep opacity below 0.15
8. **Never increase body text letter-spacing** — body should be tight or normal
9. **Never mix radius scales arbitrarily** — each component type has ONE radius
10. **Never use inline styles for tokens** — always CSS variables

## Quality Signals

What separates best-in-class design systems:

1. **Shadow sophistication** — multi-layer stacks, brand-colored, or conscious "zero shadow" philosophy
2. **OpenType precision** — specific features enabled per font context
3. **Progressive tracking** — negative letter-spacing scales proportionally with font size
4. **Custom font weight stops** — 510, 320, 480 instead of standard 400/500/600
5. **Conscious border philosophy** — shadow-as-border, semi-transparent, or oklab-space
6. **Semantic token architecture** — `--color-text-primary`, `--color-bg-surface`, not raw hex
7. **"Not black" text warmth** — intentional undertone in near-black colors
8. **Anti-pattern specificity** — Don'ts reference exact values, not generic advice
