# CSS / SCSS — AGENTS Protocol

## CSS & Preprocessors

### Minimum Required Linters

- **Stylelint** — the only CSS linter that matters
- **Prettier** — formatting (same as JS/TS — one formatter for all frontend)

Install if missing:
```bash
npm install -D stylelint stylelint-config-standard prettier
```

Config (`.stylelintrc.json`):
```json
{
    "extends": ["stylelint-config-standard"],
    "rules": {
        "declaration-block-no-duplicate-properties": true,
        "no-descending-specificity": true,
        "color-no-invalid-hex": true,
        "unit-no-unknown": true
    }
}
```

Run:
```bash
npx stylelint "src/**/*.css" && npx prettier --check "src/**/*.css"
```

### No Inline Styles in Application Code

```jsx
// BAD: Inline styles — no reuse, no responsive, no hover states
<div style={{color: 'red', fontSize: '14px', marginTop: '10px'}}>Error</div>

// GOOD: CSS class — reusable, responsive, pseudo-classes work
<div className="error-message">Error</div>
```

```css
/* The class approach gives you everything inline styles can't */
.error-message {
    color: var(--color-error);
    font-size: 0.875rem;
    margin-top: var(--spacing-sm);
}

.error-message:hover {
    /* try doing this with inline styles */
}

@media (max-width: 768px) {
    .error-message {
        font-size: 0.75rem;
    }
}
```

### Use CSS Custom Properties, Not Magic Numbers

```css
/* GOOD: Design tokens as custom properties */
:root {
    --color-primary: #2563eb;
    --color-error: #dc2626;
    --spacing-xs: 0.25rem;
    --spacing-sm: 0.5rem;
    --spacing-md: 1rem;
    --spacing-lg: 2rem;
    --radius-sm: 0.25rem;
    --radius-md: 0.5rem;
    --font-size-sm: 0.875rem;
    --font-size-base: 1rem;
}

.card {
    padding: var(--spacing-md);
    border-radius: var(--radius-md);
    color: var(--color-primary);
}

/* BAD: Magic numbers — what does 13px mean? Why 7px margin? */
.card {
    padding: 16px;
    border-radius: 7px;
    color: #2563eb;
    margin: 13px;
}
```

### Responsive Design: Mobile-First

```css
/* GOOD: Mobile-first — base styles are mobile, then add complexity */
.grid {
    display: grid;
    grid-template-columns: 1fr;
    gap: var(--spacing-md);
}

@media (min-width: 768px) {
    .grid {
        grid-template-columns: repeat(2, 1fr);
    }
}

@media (min-width: 1024px) {
    .grid {
        grid-template-columns: repeat(3, 1fr);
    }
}

/* BAD: Desktop-first — complex base, then undo for mobile */
.grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
}

@media (max-width: 1024px) {
    .grid { grid-template-columns: repeat(2, 1fr); }
}

@media (max-width: 768px) {
    .grid { grid-template-columns: 1fr; }
}
```

### Specificity: Keep It Low

```css
/* GOOD: Low specificity — easy to override, predictable */
.button { }
.button--primary { }
.button--disabled { }

/* BAD: Specificity war — impossible to override without !important */
div#app .container > .row .col-md-6 .card .button.primary {
    /* good luck overriding this */
}

/* FORBIDDEN: */
.button {
    color: red !important;  /* never */
}
```

### No Unused CSS in Production

Dead CSS is dead weight. It increases load time, confuses developers, and
creates false positives in search. Use PurgeCSS or equivalent in the build
pipeline.

### Common Agent CSS Mistakes

1. **Inline styles** — no reuse, no responsive, no pseudo-classes.
2. **Magic numbers** — use custom properties and design tokens.
3. **`!important`** — a sign that specificity is out of control.
4. **Desktop-first media queries** — leads to undoing styles for mobile.
5. **Deep nesting in SCSS** (>3 levels) — generates bloated selectors.
6. **No fallbacks for custom properties** — `color: var(--c-primary, #2563eb)`.
7. **`px` for font sizes** — use `rem` for accessibility (user zoom).

---

> **This file is part of the AGENTS protocol.** For general rules, workflow, and the full table of contents, see [AGENTS.md](./AGENTS.md).
