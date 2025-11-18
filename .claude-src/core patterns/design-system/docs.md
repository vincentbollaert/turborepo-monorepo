# Design System

> **Quick Guide:** Two-tier token system (Core → Semantic) with Open Props foundation. Semantic color tokens for text/surface/accent. 2px-based spacing scale. lucide-react for icons. SCSS Modules for all components (.module.scss). RGB color format with CSS color functions. Create component variables only when they add value through reuse or variation. Use semantic class names (purpose, not appearance). Data-attributes for state management. Modern CSS (:has(), proper nesting, max 3 levels).

---

## Token Architecture

**Two-tier system (Core → Semantic)**

**Location:** `packages/ui/src/styles/variables.scss`

- **Tier 1: Core tokens** - Base primitives defined locally
  - `--core-space-unit`, `--core-space-2`, `--core-space-4`, etc.
  - `--core-text-size-1`, `--core-text-size-2`, `--core-text-size-3`
- **Tier 2: Semantic tokens** - Purpose-driven tokens that reference Core or Open Props
  - `--color-primary`, `--color-accent`, `--color-text-default`
  - `--space-sm`, `--space-md`, `--space-lg`
  - `--text-size-body`, `--text-size-heading`
  - `--radius-sm`, `--radius-md`, `--shadow-md`

**Pattern:** Components use semantic tokens ONLY, never core tokens or Open Props variables directly

### Open Props Foundation

**Open Props as base design token library**

Open Props provides battle-tested design tokens. Semantic tokens reference Open Props. Components never use Open Props directly.

**Example:**

```scss
// packages/ui/src/styles/variables.scss
// ✅ CORRECT: Semantic tokens reference Open Props
--color-primary: var(--blue-2); // From Open Props
--color-accent: var(--cyan-4); // From Open Props
--shadow-md: var(--shadow-2); // From Open Props

// ✅ Component usage (always use semantic tokens)
.button {
  color: var(--color-primary); // NOT var(--blue-2)
  box-shadow: var(--shadow-md); // NOT var(--shadow-2)
}
```

**Commonly used:** Colors (`--gray-*`, `--blue-*`, `--cyan-4`), Shadows (`--shadow-1/2/3`), Spacing.

**RED FLAGS:**

- ❌ Using Open Props variables directly in components (bypasses semantic layer)
- ❌ Mixing custom color scales with Open Props (creates inconsistency)
- ❌ Not using semantic tokens (makes theme changes difficult)

---

## Color System

**Semantic color tokens referencing Open Props**

**Location:** `packages/ui/src/styles/variables.scss`

**Categories:**

- **Text colors:** `--color-text-default`, `--color-text-muted`, `--color-text-subtle`, `--color-text-inverted`
- **Surface colors:** `--color-surface-base`, `--color-surface-subtle`, `--color-surface-strong`, `--color-surface-stronger`, `--color-surface-strongest`
- **Accent colors:** `--color-primary`, `--color-accent`, `--color-accent-brighter`

**Pattern:** Semantic naming (purpose-based, not value-based)

```scss
// ✅ Use semantic tokens
.button {
  background-color: var(--color-surface-base);
  color: var(--color-text-default);
}

// ❌ Don't use Open Props directly
.button {
  background-color: var(--gray-0);
  color: var(--gray-7);
}
```

### Color Format Requirements

**RGB format with CSS color functions**

**Rules:**

- **Use RGB format for all colors:** `rgb(255 255 255)` instead of hex `#FFFFFF` or `rgba()`
- **Use CSS color functions for derived colors:**
  - Transparency: `rgb(0 0 0 / var(--opacity-medium))` or `rgb(0 0 0 / 0.5)`
  - Color mixing: `color-mix(in srgb, var(--color-primary), black 5%)`
  - Modern space-separated syntax: `rgb(255 255 255 / 0.5)` NOT `rgba(255, 255, 255, 0.5)`
- **Never use Sass color functions:** No `darken()`, `lighten()`, `transparentize()`, etc.
- **Avoid hard-coding color values directly** in component styles

**Why RGB format:**

- Better for design token systems and theming
- Native CSS color functions work with RGB
- Space-separated syntax enables CSS variable usage for opacity
- Eliminates need for Sass color manipulation

**Example:**

```scss
// ✅ CORRECT: RGB with CSS color functions
.button {
  background: rgb(255 255 255);
  color: rgb(0 0 0 / 0.8); // 80% opacity

  &:hover {
    background: color-mix(in srgb, var(--color-primary), white 10%);
  }
}

// ❌ WRONG: Hex colors and Sass functions
.button {
  background: #ffffff;
  color: rgba(0, 0, 0, 0.8);

  &:hover {
    background: darken($primary-color, 10%); // Don't use Sass functions
  }
}
```

**RED FLAGS:**

- ❌ Using hex colors (`#FFFFFF`, `#000000`)
- ❌ Using comma-separated rgba syntax (`rgba(0, 0, 0, 0.5)`)
- ❌ Using Sass color functions (`darken`, `lighten`, `transparentize`)
- ❌ Hard-coding color values instead of using design tokens

---

## Spacing System

**2px base unit with calculated multiples**

**Location:** `packages/ui/src/styles/variables.scss`

**Base unit:** `--core-space-unit: 0.2rem` (2px at default font size)

**Scale:**

- `--core-space-2`: 4px
- `--core-space-4`: 8px
- `--core-space-6`: 12px
- `--core-space-8`: 16px
- `--core-space-10`: 20px
- `--core-space-12`: 24px
- `--core-space-16`: 32px

**Semantic spacing tokens:**

- `--space-sm`: 4px
- `--space-md`: 8px
- `--space-lg`: 12px
- `--space-xlg`: 20px
- `--space-xxlg`: 24px
- `--space-xxxlg`: 32px

```scss
.button {
  padding: var(--space-md); // 8px
}

.container {
  gap: var(--space-lg); // 12px
}
```

---

## Typography

**REM-based with semantic naming**

**Location:** `packages/ui/src/styles/variables.scss`

**Core font sizes:**

- `--core-text-size-1`: 1.6rem (16px)
- `--core-text-size-2`: 1.8rem (18px)
- `--core-text-size-3`: 2rem (20px)

**Semantic typography tokens:**

- `--text-size-icon`: 16px
- `--text-size-body`: 16px
- `--text-size-body2`: 18px
- `--text-size-heading`: 20px

```scss
.button {
  font-size: var(--text-size-body);
}

h1 {
  font-size: var(--text-size-heading);
}
```

---

## Theme Implementation

**CURRENT STATUS: Not implemented**

- No light/dark mode switching
- No ThemeProvider in codebase
- Design tokens defined in `:root` only

**Future consideration:** If theme switching is needed, use data-attributes or CSS classes to swap token values

---

## SCSS Modules Pattern

**100% of components use SCSS Modules**

**Pattern:** CSS Modules with SCSS for component styling

**File structure:**

```
components/button/
├── button.tsx              # Component implementation
├── button.module.scss      # SCSS Module styles
└── button.stories.tsx      # Ladle stories
```

```typescript
import styles from "./button.module.scss";

<Comp className={clsx(buttonVariants({ variant, size, className }))} />
```

---

## Component Architecture

**Four-tier hierarchy with cva for variants**

**Hierarchy:**

1. **Primitives** (`src/primitives/`) - Low-level building blocks (skeleton)
2. **Components** (`src/components/`) - Reusable UI (button, switch, select, info)
3. **Patterns** (`src/patterns/`) - Higher-level patterns (feature, navigation, socials)
4. **Templates** (`src/templates/`) - Page templates and layouts (frame)

**Pattern:** Use `class-variance-authority` (cva) for type-safe variant management

**Key practices:**

- Functional components with TypeScript
- Use `cva` ONLY when component has multiple variants
- Ref forwarding with `forwardRef` for interactive components
- Expose `className` prop for customization
- Use `asChild` pattern for polymorphic components (design system components)

---

## Component Structure Standards

**Structure components with semantic classes and logical nesting**

**Key Principles:**

- Use design tokens directly from `variables.scss` in component styles
- Create component-specific variables ONLY when they add value (see criteria below)
- Define component variables at the top using `--component-[property]` naming format
- Structure components with logical nesting and clear separation of parts
- Use semantic class names that describe purpose, not appearance (e.g., `.submitButton` not `.blueButton`)
- Use data-attributes for state-based styling: `&[data-state="open"]`, `&[data-active="true"]`

**Example:**

```scss
.modal {
  // Component variables defined at top
  --modal-width: 600px;
  --modal-padding: var(--space-lg);

  // Base styles using design tokens
  width: var(--modal-width);
  padding: var(--modal-padding);
  background: var(--color-surface-base);
  border-radius: var(--radius-sm);

  // Nested elements with semantic names
  .modalHeader {
    padding-bottom: var(--space-md);
    border-bottom: 1px solid var(--color-surface-subtle);
  }

  .modalTitle {
    font-size: var(--text-size-heading);
    color: var(--color-text-default);
  }

  // State-based styling with data-attributes
  &[data-state="open"] {
    display: block;
  }

  &[data-state="closed"] {
    display: none;
  }
}
```

---

## Component-Specific Variables

**GUIDANCE: Create component variables only when they add value**

**✅ CREATE variables for:**

- Values used multiple times within the component
- Values that change based on variants/states (sizes, themes)
- Complex calculated values that would be repeated
- Values that might need runtime modification via JavaScript
- Component-specific sizing systems (e.g., avatar sizes: sm, md, lg, xl)

**❌ DON'T CREATE variables for:**

- Simple, single-use values like `1px`, `2px` for borders
- Standard font-weights like `600`, `500` used once
- Values that already exist as design tokens (avoid redeclaration)
- One-off calculations that aren't reused

**Naming Convention:** Use `--component-[property]` format (e.g., `--button-padding`, `--modal-width`)

**Example of good usage:**

```scss
// Component with variants - variables make sense
.component {
  --component-size-sm: 2rem;
  --component-size-md: 3rem;
  --component-size-lg: 4rem;
}

.sizeSm {
  width: var(--component-size-sm);
}
.sizeMd {
  width: var(--component-size-md);
}
.sizeLg {
  width: var(--component-size-lg);
}
```

**Example of unnecessary variables:**

```scss
// ❌ Don't do this
.component {
  --component-border-width: 1px; // Used only once
  border: var(--component-border-width) solid;
}

// ✅ Do this instead
.component {
  border: 1px solid var(--color-surface-subtle);
}
```

**RED FLAGS:**

- ❌ Redeclaring existing design tokens as component variables
- ❌ Creating variables for every single value in a component
- ❌ Using non-standard naming conventions (use `--component-[property]`)
- ❌ Defining variables inline instead of at the top of the component

---

## Advanced CSS Features

**Modern CSS with :has(), :global(), and data-attributes**

**Supported patterns:**

- **`:has()` for conditional styling** - Style parent based on child state
- **`:global()` for handling global classes** - Escape CSS Modules scoping when needed
- **Proper nesting with `&`** - SCSS nesting for modifiers and states
- **CSS classes for variants** - Use `cva` for type-safe variant classes
- **Data-attributes for state** - `&[data-state="open"]`, `&[data-active="true"]`

**Examples:**

```scss
// :has() for parent styling based on children
.container:has(.error) {
  border-color: var(--color-error);
}

// :global() for global class handling
.component {
  :global(.dark-mode) & {
    background: var(--color-surface-strong);
  }
}

// Proper nesting with &
.button {
  &:hover {
    background: var(--color-surface-subtle);
  }

  &[data-active="true"] {
    color: var(--color-accent);
  }
}

// Variants using CSS classes (used with cva)
.btnDefault {
  background: var(--color-surface-base);
}

.btnGhost {
  background: transparent;
}
```

**Best Practices:**

- Use data-attributes for boolean states: `data-active`, `data-state`, `data-variant`
- Prefer `:has()` over JavaScript for simple parent-child relationships
- Use `:global()` sparingly, only when necessary for third-party integration
- Keep nesting shallow (max 3 levels) for maintainability

**RED FLAGS:**

- ❌ Deep nesting (4+ levels) - harder to maintain
- ❌ Overusing `:global()` - defeats CSS Modules purpose
- ❌ Using inline styles in JavaScript instead of CSS classes
- ❌ Mixing state management approaches (pick data-attributes OR classes, not both)

---

## Iconography

**lucide-react icon library**

**Library:** `lucide-react` (installed in `packages/ui`)

**Key Principles:**

- **Consistent sizing** - Icons should use design tokens for sizing
- **Color inheritance** - Icons use `currentColor` to inherit text color from parent
- **Accessibility** - Icon-only buttons require descriptive labels
- **Visual consistency** - Use lucide-react's consistent icon set

**Sizing Pattern:**

```scss
.icon {
  width: var(--text-size-icon); // 16px
  height: var(--text-size-icon);
}
```

**Color Pattern:**

Icons automatically inherit color from their parent element's text color. Use semantic color tokens on parent elements:

```scss
.button {
  color: var(--color-text-default); // Icon inherits this color
}
```

**Accessibility:**

Icon-only buttons must have accessible labels:

```tsx
<Button size="icon" title="Expand details" aria-label="Expand details">
  <ChevronDown />
</Button>
```
