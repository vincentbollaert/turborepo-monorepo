# UI Components & Design System

**Auto-detection:** Design system, design tokens, SCSS modules, cva (class-variance-authority), component variants, Open Props, lucide-react icons

**When to use:**

- Building UI components with design system patterns
- Using design tokens (must use semantic tokens, not core or Open Props directly)
- Creating component variants with cva
- Using lucide-react icons with proper sizing and accessibility

**Key patterns covered:**

- Two-tier token system (Core → Semantic, never use core/Open Props directly in components)
- SCSS Modules for all component styling (100% of components)
- cva for type-safe variant management (only when component has variants)
- lucide-react icons with semantic sizing/coloring (currentColor, design tokens)

---

## Documentation

> **Quick Guide:** Two-tier token system (Core → Semantic) with Open Props foundation. Semantic color tokens for text/surface/accent. 2px-based spacing scale. lucide-react for icons. SCSS Modules for all components. Create component variables only when they add value through reuse or variation.

## 6.1 Token Architecture

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

### 6.1.1 Open Props Foundation

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

## 6.2 Color System

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

## 6.3 Spacing System

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

## 6.4 Typography

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

## 6.5 Theme Implementation

**CURRENT STATUS: Not implemented**

- No light/dark mode switching
- No ThemeProvider in codebase
- Design tokens defined in `:root` only

**Future consideration:** If theme switching is needed, use data-attributes or CSS classes to swap token values

## 6.6 SCSS Modules Pattern

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

## 6.7 Component Architecture

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

> See [Code Conventions - Component Architecture](../03-code-conventions/docs.md#31-component-architecture) for detailed component patterns

## 6.8 Component-Specific Variables

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

## 6.9 Iconography

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

> See [Code Conventions - Icon Library](../03-code-conventions/docs.md#313-icon-library) for implementation details and usage patterns

---

## Examples

## 6.1 Token Architecture

### Example: ACTUAL Two-Tier Token System (From `packages/ui/src/styles/variables.scss`)

```scss
// packages/ui/src/styles/variables.scss
:root {
  // ============================================
  // TIER 1: CORE TOKENS (Base primitives)
  // ============================================

  // Spacing - Base unit system
  --core-space-unit: 0.2rem; // 2px
  --core-space-2: calc(var(--core-space-unit) * 2); // 4px
  --core-space-4: calc(var(--core-space-unit) * 4); // 8px
  --core-space-6: calc(var(--core-space-unit) * 6); // 12px
  --core-space-8: calc(var(--core-space-unit) * 8); // 16px
  --core-space-10: calc(var(--core-space-unit) * 10); // 20px
  --core-space-12: calc(var(--core-space-unit) * 12); // 24px
  --core-space-16: calc(var(--core-space-unit) * 16); // 32px

  // Typography - Core sizes
  --core-text-size-1: 1.6rem; // 16px
  --core-text-size-2: 1.8rem; // 18px
  --core-text-size-3: 2rem; // 20px

  // ============================================
  // TIER 2: SEMANTIC TOKENS (Purpose-driven)
  // ============================================

  // Colors - Reference Open Props
  --color-primary: var(--blue-2);
  --color-accent: var(--cyan-4);
  --color-accent-brighter: var(--cyan-5);

  --color-text-default: var(--gray-7);
  --color-text-muted: var(--stone-2);
  --color-text-subtle: var(--gray-6);
  --color-text-inverted: var(--gray-0);

  --color-surface-base: var(--gray-0);
  --color-surface-subtle: var(--stone-3);
  --color-surface-strong: var(--gray-7);
  --color-surface-stronger: var(--gray-8);
  --color-surface-strongest: var(--gray-12);

  // Spacing - Semantic names
  --space-sm: var(--core-space-2); // 4px
  --space-md: var(--core-space-4); // 8px
  --space-lg: var(--core-space-6); // 12px
  --space-xlg: var(--core-space-10); // 20px
  --space-xxlg: var(--core-space-12); // 24px
  --space-xxxlg: var(--core-space-16); // 32px

  // Typography - Semantic names
  --text-size-icon: var(--core-text-size-1); // 16px
  --text-size-body: var(--core-text-size-1); // 16px
  --text-size-body2: var(--core-text-size-2); // 18px
  --text-size-heading: var(--core-text-size-3); // 20px

  // Border radius - Reference Open Props or define locally
  --radius-sm: 0.4rem;
  --radius-full: 9999px;
  --radius-circle: 50%;

  // Shadows - Reference Open Props
  --shadow-md: var(--shadow-2);
  --shadow-lg: var(--shadow-3);
}
```

### Example: Using Tokens in Component SCSS Module

```scss
// packages/ui/src/components/button/button.module.scss

.btn {
  // ✅ Use semantic tokens
  font-size: var(--text-size-body);
  padding: var(--space-md);
  border-radius: var(--radius-sm);

  // ❌ Never use core tokens directly
  // padding: var(--core-space-4);  // WRONG!

  // ❌ Never use Open Props directly
  // color: var(--gray-7);  // WRONG!
}

.btnDefault {
  background-color: var(--color-surface-base);
  color: var(--color-text-default);
}

.btnSizeDefault {
  padding: var(--space-md);
}

.btnSizeLarge {
  padding: var(--space-xlg) var(--space-xxlg);
}
```

**Why:** Two-tier system keeps tokens simple. Open Props provides base values. Semantic tokens make purpose clear.

**Key Rules:**

- Components use ONLY semantic tokens (Tier 2)
- Never use core tokens or Open Props directly in components
- Core tokens are building blocks for semantic tokens

## 6.2 Color System

### Example: Semantic Color Usage

```scss
// Text colors
.heading {
  color: var(--color-text-default); // Primary text
}

.description {
  color: var(--color-text-muted); // Secondary text
}

.label {
  color: var(--color-text-subtle); // Tertiary text
}

// Surface colors
.card {
  background: var(--color-surface-base); // Default background
}

.card-hover {
  background: var(--color-surface-subtle); // Subtle variation
}

.card-active {
  background: var(--color-surface-strong); // Strong emphasis
}

// Accent colors
.button-primary {
  background: var(--color-primary); // Primary brand color
}

.link {
  color: var(--color-accent); // Accent for links
}
```

**Why:** Semantic naming makes color purpose clear, not just the value.

## 6.3 Spacing System

### Example: Consistent Spacing with Semantic Tokens

```scss
// Small spacing
.compact-list {
  gap: var(--space-sm); // 4px
}

// Medium spacing (most common)
.button {
  padding: var(--space-md); // 8px
}

// Large spacing
.section {
  margin-bottom: var(--space-xlg); // 20px
}

// Mixed spacing for visual hierarchy
.card {
  padding: var(--space-lg); // 12px all sides
  margin-bottom: var(--space-xxlg); // 24px bottom
}
```

**Why:** Consistent spacing scale creates visual rhythm across the entire UI.

## 6.4 Typography

### Example: Typography Scale Usage

```scss
// Body text
.text {
  font-size: var(--text-size-body); // 16px
}

// Larger body text
.intro {
  font-size: var(--text-size-body2); // 18px
}

// Headings
h1,
h2,
h3 {
  font-size: var(--text-size-heading); // 20px
}

// Icons
.icon {
  font-size: var(--text-size-icon); // 16px
  width: var(--text-size-icon);
  height: var(--text-size-icon);
}
```

**Why:** REM-based typography respects user preferences, semantic names clarify usage.

## 6.5 Theme Implementation

### Example: NOT IMPLEMENTED (Future Reference)

**Current Status:** This codebase does NOT implement theme switching (light/dark mode).

**If you need to add theme switching in the future:**

```scss
// Define theme-specific values
:root {
  --color-text-default: var(--gray-7);
  --color-surface-base: var(--gray-0);
}

[data-theme="dark"] {
  --color-text-default: var(--gray-0);
  --color-surface-base: var(--gray-7);
}
```

**Note:** The ThemeProvider example has been removed as it doesn't exist in this codebase. If needed, implement theme switching with data-attributes as shown above.

## 6.6 SCSS Modules Pattern

### Example: Complete Component with SCSS Module

```typescript
// packages/ui/src/components/button/button.tsx
import { forwardRef } from "react";
import { Slot } from "@radix-ui/react-slot";
import { cva, type VariantProps } from "class-variance-authority";
import clsx from "clsx";
import styles from "./button.module.scss";

const buttonVariants = cva("btn", {
  variants: {
    variant: {
      default: clsx(styles.btn, styles.btnDefault),
      ghost: clsx(styles.btn, styles.btnGhost),
      link: clsx(styles.btn, styles.btnLink),
    },
    size: {
      default: clsx(styles.btn, styles.btnSizeDefault),
      large: clsx(styles.btn, styles.btnSizeLarge),
      icon: clsx(styles.btn, styles.btnSizeIcon),
    },
  },
  defaultVariants: {
    variant: "default",
    size: "default",
  },
});

export type ButtonProps = React.ComponentProps<"button"> &
  VariantProps<typeof buttonVariants> & {
    asChild?: boolean;
  };

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ variant, size, className, asChild = false, ...props }, ref) => {
    const Comp = asChild ? Slot : "button";
    return (
      <Comp
        className={clsx(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    );
  }
);

Button.displayName = "Button";
```

```scss
// packages/ui/src/components/button/button.module.scss
.btn {
  display: flex;
  align-items: center;
  justify-content: center;

  font-size: var(--text-size-body);
  font-weight: 600;

  border-radius: var(--radius-sm);
  border: 1px solid transparent;

  cursor: pointer;
  transition: all 0.2s ease;

  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
}

.btnDefault {
  background-color: var(--color-surface-base);
  color: var(--color-text-default);
  border-color: var(--color-surface-subtle);

  &:hover:not(:disabled) {
    background-color: var(--color-surface-subtle);
  }

  &[data-active="true"] {
    color: var(--color-text-muted);
    background: var(--color-surface-strong);
  }
}

.btnGhost {
  background-color: transparent;

  &:hover:not(:disabled) {
    background-color: var(--color-surface-subtle);
  }
}

.btnSizeDefault {
  padding: var(--space-md);
}

.btnSizeLarge {
  padding: var(--space-xlg) var(--space-xxlg);
}

.btnSizeIcon {
  padding: var(--space-md);
  aspect-ratio: 1;
}
```

**Why:** SCSS Modules provide scoped styles. cva provides type-safe variants. Design tokens ensure consistency.

## 6.7 Component Architecture

## 6.8 Component-Specific Variables

### Example: When to Create Component Variables

```scss
// ✅ GOOD: Component with multiple size variants - variables add value
.modal {
  // Component-specific variables that change between variants
  --modal-width-sm: 400px;
  --modal-width-md: 600px;
  --modal-width-lg: 900px;

  width: var(--modal-width-md); // Default
  padding: var(--space-lg);
  background: var(--color-surface-base);
}

.modal[data-size="sm"] {
  width: var(--modal-width-sm);
}

.modal[data-size="lg"] {
  width: var(--modal-width-lg);
}
```

### Example: When NOT to Create Component Variables

```scss
// ❌ BAD: Single-use values don't need variables
.card {
  --card-border-width: 1px; // Used only once - unnecessary!
  --card-border-radius: 0.5rem; // Already have --radius-sm!

  border: var(--card-border-width) solid var(--color-surface-subtle);
  border-radius: var(--card-border-radius);
}

// ✅ GOOD: Use design tokens directly
.card {
  border: 1px solid var(--color-surface-subtle);
  border-radius: var(--radius-sm);
}
```

### Example: Complex Calculated Values

```scss
// ✅ GOOD: Complex calculation used multiple times
.sidebar {
  --sidebar-width: 280px;
  --content-width: calc(100% - var(--sidebar-width));

  width: var(--sidebar-width);
}

.main-content {
  width: var(--content-width); // Reuse the calculation
  margin-left: var(--sidebar-width);
}
```

**Key Principle:** Only create component variables when they provide real value through reuse, variation, or runtime modification.

## 6.9 Iconography

### Example: ACTUAL Icon Usage with lucide-react

```typescript
// packages/ui/src/patterns/feature/feature.tsx
import { ChevronDown, ChevronUp } from "lucide-react";
import { Button } from "../../components/button/button";
import styles from "./feature.module.scss";

export const Feature = ({ id, title, description, status }: FeatureProps) => {
  const [isExpanded, setIsExpanded] = useState(false);

  return (
    <li onClick={() => setIsExpanded(!isExpanded)}>
      <h2>{title}</h2>
      <Button
        variant="ghost"
        size="icon"
        className={styles.expandButton}
        aria-label={isExpanded ? "Collapse details" : "Expand details"}
      >
        {isExpanded ? (
          <ChevronUp className={styles.icon} />
        ) : (
          <ChevronDown className={styles.icon} />
        )}
      </Button>
      {isExpanded && <p>{description}</p>}
    </li>
  );
};
```

```scss
// packages/ui/src/patterns/feature/feature.module.scss
.expandButton {
  // Button already has proper sizing
  // Icon inherits color from button
}

.icon {
  // Use design token for consistent sizing
  width: var(--text-size-icon); // 16px
  height: var(--text-size-icon);
}
```

**Why:** lucide-react provides consistent, tree-shakeable icons. Icons inherit color automatically.

### Example: Icon-Only Buttons with Accessibility

```typescript
// packages/ui/src/patterns/socials/socials.tsx
import { CircleUserRound, CodeXml } from "lucide-react";
import { Button } from "../../components/button/button";

export const Socials = () => {
  return (
    <ul>
      <li>
        <Button
          size="icon"
          title="View GitHub profile"
          aria-label="View GitHub profile"
          onClick={() => window.open("https://github.com/username", "_blank")}
        >
          <CodeXml />
        </Button>
      </li>
      <li>
        <Button
          size="icon"
          title="Visit blog"
          aria-label="Visit blog"
          onClick={() => window.open("https://blog.example.com", "_blank")}
        >
          <CircleUserRound />
        </Button>
      </li>
    </ul>
  );
};
```

**Why:** Icon-only buttons need both `title` (for tooltip) and `aria-label` (for screen readers).

### Example: Icon Color Inheritance

```scss
// Icons automatically inherit currentColor
.successButton {
  color: var(--color-text-default); // Icon inherits this

  &:hover {
    color: var(--color-accent); // Icon color changes on hover
  }
}

.errorButton {
  color: var(--color-text-muted); // Different icon color
}
```

```tsx
<Button className={styles.successButton}>
  <CheckCircle />  {/* Icon inherits green color */}
  Save
</Button>

<Button className={styles.errorButton}>
  <XCircle />  {/* Icon inherits red color */}
  Delete
</Button>
```

**Why:** Using `currentColor` keeps icon colors in sync with text, reducing duplication.
