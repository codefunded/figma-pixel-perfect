# Accessibility Rules — WCAG AAA Compliance

This document defines the accessibility standards for the design system. All components MUST meet WCAG 2.1 Level AAA.

---

## Contrast Ratio Requirements

### AAA Minimums

| Text Type | Minimum Ratio | Standard |
|---|---|---|
| Normal text (< 18px) | **7:1** | WCAG AAA |
| Large text (>= 18px or >= 14px bold) | **4.5:1** | WCAG AAA |
| UI components and graphical objects | **3:1** | WCAG 2.1 |
| Decorative elements | No requirement | — |

### Specific Color Values That Pass AAA

#### On White (#FFFFFF) Background
| Use Case | Color | Hex | Ratio |
|---|---|---|---|
| Body text | Foreground | `#242424` | 13.3:1 |
| Placeholder text | Muted foreground | `#595959` | 7.06:1 |
| Helper/caption text | Muted foreground | `#595959` | 7.06:1 |
| Disabled text | N/A | `#767676` | 4.54:1 (AA only — acceptable for disabled) |
| Border | Border | `#CCCCCC` | 1.61:1 (decorative, no requirement) |

#### On Dark (#1A1A1A) Background
| Use Case | Color | Hex | Ratio |
|---|---|---|---|
| Body text | Foreground | `#F5F5F5` | 14.5:1 |
| Placeholder text | Muted foreground | `#A3A3A3` | 7.12:1 |
| Helper/caption text | Muted foreground | `#A3A3A3` | 7.12:1 |

### Colors That FAIL AAA (Do Not Use for Normal Text)

| Color | Hex | Ratio on White | Issue |
|---|---|---|---|
| Default placeholder gray | `#767676` | 4.54:1 | Only passes AA, fails AAA |
| Light gray | `#999999` | 2.85:1 | Fails both AA and AAA |
| Medium gray | `#AAAAAA` | 2.32:1 | Fails both AA and AAA |

---

## Icon-Only Buttons

Every button that contains only an icon (no visible text label) MUST have `aria-label`:

```tsx
// CORRECT
<Button variant="ghost" size="icon" aria-label="Close dialog">
  <X className="h-4 w-4" />
</Button>

// WRONG — screen reader announces "button" with no context
<Button variant="ghost" size="icon">
  <X className="h-4 w-4" />
</Button>
```

Guidelines for `aria-label` text:
- Use the action the button performs, not the icon name
- "Close dialog" not "X icon"
- "Open calendar" not "Calendar"
- "Remove item" not "Trash"
- "Search" not "Magnifying glass"

---

## Live Regions for Dynamic Content

### Toast Notifications

```tsx
<div role="alert" aria-live="assertive" aria-atomic="true">
  {/* Toast content */}
</div>
```

- Use `aria-live="assertive"` for errors and urgent notifications
- Use `aria-live="polite"` for success messages and informational toasts
- Use `aria-atomic="true"` so the entire toast is read, not just changed parts

### Alert Components

```tsx
<div role="alert">
  <AlertTitle>Error</AlertTitle>
  <AlertDescription>Something went wrong.</AlertDescription>
</div>
```

The `role="alert"` implicitly sets `aria-live="assertive"`.

### Status Updates

```tsx
<div role="status" aria-live="polite">
  Loading... 3 of 10 items
</div>
```

---

## Focus Management

### Modals / Dialogs

Radix Dialog handles focus management automatically:
- Focus traps inside the dialog when open
- Returns focus to trigger element on close
- Escape key closes the dialog

Do NOT implement custom focus traps — Radix handles this correctly.

### Dropdown Menus

Radix DropdownMenu provides:
- Arrow key navigation between items
- Home/End to jump to first/last item
- Type-ahead to jump to matching items
- Escape to close and return focus to trigger

### Custom Focus Management

When building custom interactive components, ensure:
1. Focus is visible (never `outline: none` without replacement)
2. Focus order matches visual order
3. Focus doesn't get lost when elements are removed from DOM

```css
/* Default focus ring for all interactive elements */
focus-visible:outline-none
focus-visible:ring-2
focus-visible:ring-ring
focus-visible:ring-offset-2
focus-visible:ring-offset-background
```

---

## Keyboard Navigation Requirements

### Universal

| Key | Action |
|---|---|
| Tab | Move focus to next interactive element |
| Shift+Tab | Move focus to previous interactive element |
| Enter | Activate button/link |
| Space | Activate button, toggle checkbox |
| Escape | Close modal/dropdown/popover |

### Component-Specific

| Component | Key | Action |
|---|---|---|
| Accordion | Enter/Space | Toggle section |
| Accordion | Arrow Up/Down | Move between headers |
| Tabs | Arrow Left/Right | Switch tabs |
| Tabs | Home/End | First/last tab |
| Select | Arrow Up/Down | Navigate options |
| Select | Enter | Select option |
| Slider | Arrow Left/Right | Decrease/increase value |
| Slider | Home/End | Min/max value |
| Calendar | Arrow keys | Navigate dates |
| Calendar | Page Up/Down | Previous/next month |
| Table | Arrow keys | Navigate cells (if interactive) |

---

## Touch Target Minimum Size

All interactive elements MUST have a minimum touch target of **44x44px** (WCAG 2.5.5 AAA).

```tsx
// For small visual elements, use padding to reach 44px
<button className="min-h-[44px] min-w-[44px] p-2">
  <Icon className="h-4 w-4" />
</button>

// For inline links, ensure line-height provides adequate height
<a className="py-2 inline-block">Link text</a>
```

Exceptions:
- Inline links within a paragraph (use sufficient line-height instead)
- Elements where size is essential to the information being conveyed

---

## Reduced Motion Support

Add to `globals.css`:

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

For JavaScript animations (e.g., canvas drawing, counters):

```typescript
const prefersReducedMotion = window.matchMedia(
  "(prefers-reduced-motion: reduce)"
).matches;

if (prefersReducedMotion) {
  // Skip animation, show final state immediately
} else {
  // Run animation
}
```

---

## Form Accessibility

### Required Fields

```tsx
<label htmlFor="email">
  Email <span aria-hidden="true">*</span>
</label>
<input
  id="email"
  aria-required="true"
  aria-invalid={hasError}
  aria-describedby={hasError ? "email-error" : "email-help"}
/>
{hasError ? (
  <p id="email-error" role="alert" className="text-[12px] text-destructive">
    Please enter a valid email address.
  </p>
) : (
  <p id="email-help" className="text-[12px] text-muted-foreground">
    We'll never share your email.
  </p>
)}
```

### Key Attributes

| Attribute | When to Use |
|---|---|
| `aria-required="true"` | Required form fields |
| `aria-invalid="true"` | Fields with validation errors |
| `aria-describedby` | Link to helper text or error message |
| `aria-errormessage` | Link to error message specifically |
| `aria-disabled="true"` | Disabled fields (in addition to HTML `disabled`) |
| `htmlFor` / `id` | Always pair labels with inputs |

### Fieldset and Legend

For groups of related fields (radio groups, checkbox groups, address fields):

```tsx
<fieldset>
  <legend>Shipping Address</legend>
  {/* Fields */}
</fieldset>
```

---

## Skip Navigation

Add a skip link as the first focusable element in the page:

```tsx
<a
  href="#main-content"
  className="sr-only focus:not-sr-only focus:fixed focus:top-4 focus:left-4 focus:z-50 focus:px-4 focus:py-2 focus:bg-background focus:text-foreground focus:border focus:border-border focus:rounded-[8px]"
>
  Skip to main content
</a>

{/* Later in the page */}
<main id="main-content" tabIndex={-1}>
  {/* Page content */}
</main>
```

---

## Screen Reader Patterns

### Visually Hidden Text

Use `sr-only` for text that should be read by screen readers but not displayed:

```tsx
<button>
  <X className="h-4 w-4" />
  <span className="sr-only">Close</span>
</button>
```

### Loading States

```tsx
<button disabled aria-busy="true">
  <Spinner className="animate-spin" aria-hidden="true" />
  <span className="sr-only">Loading, please wait</span>
  Loading...
</button>
```

### Tables

```tsx
<table>
  <caption className="sr-only">User accounts and their status</caption>
  <thead>
    <tr>
      <th scope="col">Name</th>
      <th scope="col">Email</th>
      <th scope="col">Status</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>John Doe</td>
      <td>john@example.com</td>
      <td>
        <Badge variant="success">Active</Badge>
        {/* Screen reader gets "Active" from badge text */}
      </td>
    </tr>
  </tbody>
</table>
```

### Progress Indicators

```tsx
<div
  role="progressbar"
  aria-valuenow={75}
  aria-valuemin={0}
  aria-valuemax={100}
  aria-label="Upload progress"
>
  <div className="h-2 bg-primary" style={{ width: "75%" }} />
</div>
```

---

## Color-Only Information

Never convey information through color alone. Always pair with:
- Text labels
- Icons or patterns
- Shape differences

```tsx
// WRONG — status conveyed only by color
<div className="h-3 w-3 rounded-full bg-green-500" />

// CORRECT — color + text
<div className="flex items-center gap-2">
  <div className="h-3 w-3 rounded-full bg-success" aria-hidden="true" />
  <span className="text-sm">Active</span>
</div>
```

---

## Image Accessibility

```tsx
// Informative images — describe the content
<img src="chart.png" alt="Sales increased 25% from Q1 to Q2 2024" />

// Decorative images — empty alt
<img src="pattern.svg" alt="" aria-hidden="true" />

// Complex images — link to longer description
<figure>
  <img src="diagram.png" alt="System architecture diagram" aria-describedby="diagram-desc" />
  <figcaption id="diagram-desc">
    The system consists of three layers: presentation, business logic, and data access...
  </figcaption>
</figure>
```
