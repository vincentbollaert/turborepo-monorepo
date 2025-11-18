# Design System

> **Quick Guide:** Two-tier token system (Core → Semantic) with Open Props foundation. Semantic color tokens for text/surface/accent. 2px-based spacing scale. lucide-react for icons. SCSS Modules for all components. Create component variables only when they add value through reuse or variation.

---

## Token Architecture

**ACTUAL IMPLEMENTATION: Two-tier system (Core → Semantic)**

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

**ACTUAL IMPLEMENTATION: Open Props as base design token library**

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

**ACTUAL IMPLEMENTATION: Semantic color tokens referencing Open Props**

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

---

## Spacing System

**ACTUAL IMPLEMENTATION: 2px base unit with calculated multiples**

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

**ACTUAL IMPLEMENTATION: REM-based with semantic naming**

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

**ACTUAL IMPLEMENTATION: 100% of components use SCSS Modules**

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

**ACTUAL IMPLEMENTATION: Four-tier hierarchy with cva for variants**

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

## Component-Specific Variables

**GUIDANCE: Create component variables only when they add value**

**✅ CREATE variables for:**

- Values used multiple times within the component
- Values that change based on variants/states
- Complex calculated values that would be repeated
- Values that might need runtime modification via JavaScript

**❌ DON'T CREATE variables for:**

- Simple, single-use values like `1px`, `2px` for borders
- Standard font-weights like `600`, `500` used once
- Values that already exist as design tokens
- One-off calculations that aren't reused

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

---

## Iconography

**ACTUAL IMPLEMENTATION: lucide-react icon library**

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
