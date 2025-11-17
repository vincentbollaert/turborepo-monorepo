# Turborepo Playground - Comprehensive Standards & Patterns

**Extraction Date:** 2025-11-13  
**Codebase Version:** Current state  
**Monorepo Tool:** Turborepo 2.4.2  
**Package Manager:** Bun 1.2.2  
**Confidence Level:** High - based on consistent patterns across codebase

---

## Table of Contents

1. [Package Architecture](#1-package-architecture)
2. [Code Conventions](#2-code-conventions)
3. [State Management](#3-state-management)
4. [Testing Standards](#4-testing-standards)
5. [Design System](#5-design-system)
6. [Accessibility](#6-accessibility)
7. [Build & Tooling](#7-build--tooling)
8. [CI/CD Pipelines](#8-cicd-pipelines)
9. [Environment Management](#9-environment-management)
10. [Architecture Decisions](#10-architecture-decisions)
11. [AI Agent Optimization](#11-ai-agent-optimization)
12. [Performance Standards](#12-performance-standards)
13. [Security Patterns](#13-security-patterns)
14. [Git Workflow](#14-git-workflow)
15. [Anti-Patterns Observed](#15-anti-patterns-observed)
16. [Quick Reference for AI](#16-quick-reference-for-ai)

---

## 1. Package Architecture

### 1.1 Workspace Structure

**Pattern:** Clear separation between applications and shared packages  
**Frequency:** Consistent across entire monorepo  
**Example:**

```
.
├── apps/                      # Application packages
│   ├── client-next/           # Next.js 15 application
│   ├── client-react/          # React 19 + Vite application
│   └── server/                # Express API server
├── packages/                  # Shared packages
│   ├── api/                   # API client and types (generated from OpenAPI)
│   ├── api-mocks/             # MSW handlers and mock data
│   ├── ui/                    # Shared UI component library
│   ├── eslint-config/         # Shared ESLint configurations
│   ├── typescript-config/     # Shared TypeScript configurations
│   ├── prettier-config/       # Shared Prettier configuration
│   └── vitest-config/         # Shared Vitest configuration
```

**Rationale:** Separates deployable applications from reusable packages, enabling code sharing while maintaining clear boundaries.

### 1.2 Package Naming Conventions

**Pattern:** Internal packages use `@repo/` prefix, apps use simple names  
**Frequency:** 100% consistent across all packages  
**Example:**

```json
// packages/ui/package.json:2
{
  "name": "@repo/ui"
}

// apps/client-next/package.json:2
{
  "name": "client-next"
}
```

**Rationale:** `@repo/` prefix clearly identifies internal packages and prevents naming conflicts with npm packages.

### 1.3 Dependency Management

**Pattern:** Use wildcard `*` for internal workspace dependencies  
**Frequency:** Consistent across all packages  
**Example:**

```json
// apps/client-next/package.json:14-16
{
  "dependencies": {
    "@repo/api": "*",
    "@repo/ui": "*"
  }
}
```

**Rationale:** Wildcard ensures packages always use the local workspace version, simplifying version management.

### 1.4 Package Exports Pattern

**Pattern:** Explicit named exports for granular imports  
**Frequency:** Used in @repo/ui and @repo/api packages  
**Example:**

```json
// packages/ui/package.json:11-24
{
  "exports": {
    "./global.scss": "./src/styles/global.scss",
    "./skeleton": "./src/primitives/skeleton/skeleton.tsx",
    "./info": "./src/components/info/info.tsx",
    "./button": "./src/components/button/button.tsx",
    "./switch": "./src/components/switch/switch.tsx",
    "./select": "./src/components/select/select.tsx",
    "./feature": "./src/patterns/feature/feature.tsx",
    "./navigation": "./src/patterns/navigation/navigation.tsx",
    "./mockSelector": "./src/patterns/mockSelector/mockSelector.tsx",
    "./socials": "./src/patterns/socials/socials.tsx",
    "./appSwitcher": "./src/patterns/appSwitcher/appSwitcher.tsx",
    "./frame": "./src/templates/frame/frame.tsx",
    "./hooks": "./src/hooks/index.ts"
  }
}
```

**Rationale:** Enables tree-shaking, provides clear API surface, and prevents importing internal implementation details.

### 1.5 UI Library Organization

**Pattern:** Four-tier component hierarchy  
**Frequency:** Consistently applied in @repo/ui package  
**Structure:**

1. **Primitives** (`src/primitives/`) - Low-level UI building blocks (skeleton)
2. **Components** (`src/components/`) - Reusable UI components (button, switch, select, info)
3. **Patterns** (`src/patterns/`) - Higher-level UI patterns (feature, navigation, socials, appSwitcher, mockSelector)
4. **Templates** (`src/templates/`) - Page templates and layouts (frame)

**Example:**

```
packages/ui/src/
├── primitives/skeleton/     # Basic loading skeleton
├── components/button/       # Reusable button component
├── patterns/feature/        # Feature list pattern
└── templates/frame/         # Page frame template
```

**Rationale:** Clear hierarchy from simple to complex, following atomic design principles adapted for this project.

---

## 2. Code Conventions

### 2.1 Component Architecture

**Pattern:** Functional components with TypeScript, using class-variance-authority for variants  
**Frequency:** Consistent across all UI components  
**Example:**

```typescript
// packages/ui/src/components/button/button.tsx:8-25
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
```

**Rationale:** class-variance-authority provides type-safe variant management, clsx handles conditional classes, SCSS modules provide scoped styling.

### 2.2 Props Pattern

**Pattern:** Extend native HTML element props with component-specific props
**Frequency:** Used in all UI components
**Example:**

```typescript
// packages/ui/src/components/button/button.tsx:27-30
export type ButtonProps = React.ComponentProps<"button"> &
  VariantProps<typeof buttonVariants> & {
    asChild?: boolean;
  };
```

**Rationale:** Inherits all standard HTML attributes, adds type-safe variants, enables polymorphic behavior with `asChild`.

### 2.3 File and Directory Naming

**Pattern:** kebab-case for directories and files, PascalCase for component files
**Frequency:** 100% consistent
**Examples:**

- Components: `button.tsx`, `button.module.scss`, `button.stories.tsx`
- Directories: `client-next/`, `api-mocks/`, `eslint-config/`
- Test files: `features.test.tsx`, `navigation.test.tsx`

**Rationale:** Follows React/TypeScript community standards, ensures consistency across platforms.

### 2.4 Import/Export Patterns

**Pattern:** Named exports preferred, no default exports in libraries
**Frequency:** Consistent across all packages
**Example:**

```typescript
// packages/ui/src/components/button/button.tsx:44
export { Button, buttonVariants };
```

**Rationale:** Named exports enable better tree-shaking, clearer imports, and easier refactoring.

### 2.5 Type Definitions

**Pattern:** Co-located type definitions with components
**Frequency:** Consistent across all components
**Example:**

```typescript
// packages/ui/src/patterns/feature/feature.tsx:9-14
export type FeatureProps = {
  id: string;
  title: string;
  status: string;
  description: string;
};
```

**Rationale:** Types live close to their usage, making them easier to find and maintain.

### 2.6 Component File Structure

**Pattern:** Each component in its own directory with related files
**Frequency:** 100% consistent in UI package
**Example:**

```
packages/ui/src/components/button/
├── button.tsx              # Component implementation
├── button.module.scss      # Component styles
└── button.stories.tsx      # Ladle stories
```

**Rationale:** Co-location makes components self-contained and easier to move or delete.

---

## 3. State Management

### 3.1 Server State vs Client State Separation

**Strategy:** Strict separation - React Query for ALL server data, local state for UI
**Frequency:** Consistently applied across both apps
**Pattern:**

- **Server State:** React Query (`@tanstack/react-query`) for API data
- **Client State:** React `useState` for component-local UI state
- **No global client state library** (Zustand/Redux) - not needed yet

**Rationale:** React Query handles server state caching, synchronization, and invalidation. Local state is sufficient for UI state.

### 3.2 React Query Configuration

**Pattern:** Centralized QueryClient with shared defaults
**Frequency:** Used in both client apps
**Example:**

```typescript
// apps/client-next/app/providers.tsx:9-15
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: false,
    },
  },
});
```

**Rationale:** `retry: false` prevents unnecessary retries during development with MSW mocks.

### 3.3 API Client Integration

**Pattern:** OpenAPI-generated clients with React Query hooks
**Frequency:** Used across all data fetching
**Example:**

```typescript
// apps/client-next/app/features.tsx:15
const { isSuccess, isPending, error, data } = useQuery(getFeaturesOptions());
```

**Generated from:**

```typescript
// packages/api/openapi-ts.config.ts:10
plugins: [...defaultPlugins, "@hey-api/client-fetch", "@tanstack/react-query"],
```

**Rationale:** Type-safe API calls generated from OpenAPI schema, automatic React Query integration.

### 3.4 Query Hook Pattern

**Pattern:** Use generated query options from API package
**Frequency:** Consistent across all data fetching
**Example:**

```typescript
// apps/client-next/app/features.tsx:3,15
import { getFeaturesOptions } from "@repo/api/reactQueries";
const { isSuccess, isPending, error, data } = useQuery(getFeaturesOptions());
```

**Rationale:** Generated hooks provide type safety, consistent error handling, and automatic cache management.

### 3.5 Loading and Error States

**Pattern:** Explicit handling of all query states
**Frequency:** Consistent pattern in all data-fetching components
**Example:**

```typescript
// apps/client-next/app/features.tsx:17-46
if (isPending) {
  return (
    <Shell>
      {Array(7).fill("").map((_, index) => (
        <div className={styles.row} key={index}>
          <Skeleton className={styles.skeletonCircle} />
          <Skeleton />
        </div>
      ))}
    </Shell>
  );
}

if (isSuccess && !data.features?.length) {
  return (
    <Shell>
      <Info variant="info" message="No features found" />
    </Shell>
  );
}

if (error) {
  return (
    <Shell>
      <Info variant="error" message={`An error has occurred: ${error}`} />
    </Shell>
  );
}
```

**Rationale:** Provides clear feedback for all states, improves UX, makes testing easier.

---

## 4. Testing Standards

### 4.1 Test Organization

**Structure:** Co-located tests in `__tests__` directories
**Naming:** `*.test.tsx` for all test files
**Location:** Adjacent to implementation
**Frequency:** 86 test files found

**Example:**

```
apps/client-react/src/home/
├── features.tsx
├── home.tsx
├── learnings.tsx
└── __tests__/
    ├── features.test.tsx
    └── navigation.test.tsx
```

**Rationale:** Co-location makes tests easy to find and maintain alongside implementation.

### 4.2 Testing Framework

**Framework:** Vitest with React Testing Library
**Configuration:** Shared base config in `@repo/vitest-config`
**Example:**

```typescript
// packages/vitest-config/vite.config.ts:5-16
export const baseViteConfig = defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    globals: true,
    setupFiles: [], // override this in consumer
    coverage: {
      provider: "v8",
      reporter: ["text", "json", "html"],
      include: ["src/**/*.{ts,tsx}"],
    },
  },
});
```

**Rationale:** Vitest is fast, Vite-native, and has excellent TypeScript support.

### 4.3 Integration Testing with MSW

**Pattern:** Test components with real API integration using MSW
**Frequency:** Primary testing strategy
**Example:**

```typescript
// apps/client-react/src/home/__tests__/features.test.tsx:1-7,19-24
import { getFeaturesHandlers } from "@repo/api-mocks/handlers";
import { defaultFeatures } from "@repo/api-mocks/mocks";
import { serverWorker } from "@repo/api-mocks/serverWorker";
import { screen, waitFor, within } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { describe, expect, it } from "vitest";
import { renderApp } from "../../testSetup/testUtils.local";

it("should render empty state", async () => {
  serverWorker.use(getFeaturesHandlers.emptyHandler());
  renderApp();

  await expect(screen.findByText("No features found")).resolves.toBeInTheDocument();
});
```

**Rationale:** Tests real component behavior with API integration, catches more bugs than unit tests alone.

### 4.4 Mock Data Management

**Pattern:** Centralized mock data in `@repo/api-mocks` package
**Structure:** Separate handlers and mocks
**Frequency:** Consistent across all API endpoints
**Example:**

```typescript
// packages/api-mocks/src/handlers/features/getFeatures.ts:14-27
const defaultHandler = () =>
  http.get(URL, async () => {
    switch (mockVariantsByEndpoint.features) {
      case "empty": {
        return emptyResponse();
      }
      case "error": {
        return errorResponse();
      }
      default: {
        return defaultResponse();
      }
    }
  });
```

**Rationale:** Centralized mocks enable testing different scenarios, shared between apps and tests.

### 4.5 Test Patterns

**What to test:**

- Component behavior with different data states (loading, empty, error, success)
- User interactions (clicks, form inputs)
- State changes and side effects
- Integration with API layer

**What NOT to test:**

- Implementation details (internal state, private methods)
- Third-party library behavior
- Styling (unless critical to functionality)

**Example:**

```typescript
// apps/client-react/src/home/__tests__/features.test.tsx:64-73
it("should toggle the feature", async () => {
  renderApp();

  const feature = await getFeature();
  const switchElement = within(feature).getByRole("switch");
  expect(switchElement).toBeChecked();

  userEvent.click(switchElement);
  await waitFor(() => expect(switchElement).not.toBeChecked());
});
```

**Rationale:** Focus on user-facing behavior, not implementation details.

### 4.6 Pre-push Testing

**Pattern:** Run tests before pushing to prevent broken builds
**Frequency:** Enforced via Husky hook
**Example:**

```bash
# .husky/pre-push:1
cd apps/client-react && bun run test --watch=false
```

**Rationale:** Catches test failures before they reach CI, saves time and resources.

---

## 5. Design System

### 5.1 Token Architecture

**System:** Two-tier token structure (Core → Semantic)
**Frequency:** Consistently applied across all styles
**Example:**

```scss
// packages/ui/src/styles/variables.scss:1-50
:root {
  // CORE (Tier 1: Base values)
  --core-space-unit: 0.2rem; // 2px
  --core-space-2: calc(var(--core-space-unit) * 2); // 4px
  --core-space-4: calc(var(--core-space-unit) * 4); // 8px
  --core-space-6: calc(var(--core-space-unit) * 6); // 12px
  --core-space-8: calc(var(--core-space-unit) * 8); // 16px
  --core-space-10: calc(var(--core-space-unit) * 10); // 20px
  --core-space-12: calc(var(--core-space-unit) * 12); // 24px
  --core-space-16: calc(var(--core-space-unit) * 16); // 32px

  --core-text-size-1: 1.6rem;
  --core-text-size-2: 1.8rem;
  --core-text-size-3: 2rem;

  // SEMANTIC (Tier 2: Purpose-driven)
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

  --space-sm: var(--core-space-2); // 4px
  --space-md: var(--core-space-4); // 8px
  --space-lg: var(--core-space-6); // 12px
  --space-xlg: var(--core-space-10); // 20px
  --space-xxlg: var(--core-space-12); // 24px
  --space-xxxlg: var(--core-space-16); // 32px

  --text-size-icon: var(--core-text-size-1); // 16px
  --text-size-body: var(--core-text-size-1); // 16px
  --text-size-body2: var(--core-text-size-2); // 18px
  --text-size-heading: var(--core-text-size-3); // 20px

  --radius-sm: 0.4rem;
  --radius-full: 9999px;
  --radius-circle: 50%;

  --shadow-md: var(--shadow-2);
  --shadow-lg: var(--shadow-3);
}
```

**Rationale:** Core tokens define raw values, semantic tokens provide meaning. Components use semantic tokens for maintainability.

### 5.2 Open Props Integration

**Pattern:** Use Open Props for base design tokens (colors, shadows, fonts)
**Frequency:** Imported in all apps
**Example:**

```json
// apps/client-next/package.json:20
{
  "dependencies": {
    "open-props": "^1.7.13"
  }
}
```

**Usage:**

```scss
// packages/ui/src/styles/variables.scss:17-30
--color-primary: var(--blue-2);  // From Open Props
--color-accent: var(--cyan-4);   // From Open Props
--shadow-md: var(--shadow-2);    // From Open Props
```

**Rationale:** Open Props provides battle-tested design tokens, reducing custom CSS and ensuring consistency.

### 5.3 Spacing System

**Scale:** 2px base unit with calculated multiples
**Pattern:** Use semantic spacing tokens in components
**Example:**

```scss
// packages/ui/src/components/button/button.module.scss:63
.btnSizeDefault {
  padding: var(--space-md);  // 8px
}

// packages/ui/src/components/button/button.module.scss:71
.btnSizeLarge {
  padding: var(--space-xlg) var(--space-xxlg);  // 20px 24px
}
```

**Rationale:** Consistent spacing creates visual rhythm, semantic names make intent clear.

### 5.4 Color System

**Pattern:** Semantic color tokens referencing Open Props colors
**Categories:**

- **Text colors:** `--color-text-default`, `--color-text-muted`, `--color-text-subtle`, `--color-text-inverted`
- **Surface colors:** `--color-surface-base`, `--color-surface-subtle`, `--color-surface-strong`, `--color-surface-stronger`, `--color-surface-strongest`
- **Accent colors:** `--color-primary`, `--color-accent`, `--color-accent-brighter`

**Example:**

```scss
// packages/ui/src/components/button/button.module.scss:33-34
.btnDefault {
  background-color: var(--color-surface-base);
  color: var(--color-text-default);
}
```

**Rationale:** Semantic naming makes color purpose clear, enables easy theme switching.

### 5.5 Typography

**Scale:** REM-based with semantic naming
**Example:**

```scss
// packages/ui/src/styles/variables.scss:39-42
--text-size-icon: var(--core-text-size-1); // 16px
--text-size-body: var(--core-text-size-1); // 16px
--text-size-body2: var(--core-text-size-2); // 18px
--text-size-heading: var(--core-text-size-3); // 20px
```

**Usage:**

```scss
// packages/ui/src/components/button/button.module.scss:11
.btn {
  font-size: var(--text-size-body);
}
```

**Rationale:** REM units respect user font size preferences, semantic names clarify usage.

### 5.6 SCSS Modules Pattern

**Pattern:** CSS Modules with SCSS for component styling
**Frequency:** 100% of components use this pattern
**Example:**

```typescript
// packages/ui/src/components/button/button.tsx:6
import styles from "./button.module.scss";

// Usage:
<Comp className={clsx(buttonVariants({ variant, size, className }))} />
```

**Rationale:** Scoped styles prevent conflicts, SCSS provides nesting and variables, modules enable tree-shaking.

### 5.7 Component-Specific Variables

**Pattern:** Create component variables only when they add value
**Guidance from:** `.claude/patterns/component-styling.md:15-28`

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

.sizeSm { width: var(--component-size-sm); }
.sizeMd { width: var(--component-size-md); }
.sizeLg { width: var(--component-size-lg); }
```

**Example of unnecessary variables:**

```scss
// ❌ Don't do this
.component {
  --component-border-width: 1px;  // Used only once
  border: var(--component-border-width) solid;
}

// ✅ Do this instead
.component {
  border: 1px solid var(--color-surface-subtle);
}
```

**Rationale:** Reduces unnecessary abstraction, keeps styles simple and maintainable.

---

## 6. Accessibility

### 6.1 Semantic HTML

**Pattern:** Use semantic HTML elements with proper ARIA attributes
**Frequency:** Consistent across components
**Example:**

```typescript
// packages/ui/src/patterns/feature/feature.tsx:20-24
<li
  className={clsx(styles.feature, computeDrawnBorderClass(), computeChaosShadowsClass())}
  onClick={() => setIsExpanded((expandedState) => !expandedState)}
  data-expanded={isExpanded}
  data-testid="feature"
>
```

**Rationale:** Semantic HTML provides better accessibility and SEO.

### 6.2 Interactive Elements

**Pattern:** Use proper roles and keyboard support
**Example:**

```typescript
// packages/ui/src/patterns/feature/feature.tsx:28-35
<Switch
  id={`${id}-switch`}
  className={styles.switch}
  checked={status === "done"}
  onClick={(event) => {
    event.stopPropagation();
  }}
/>
```

**Rationale:** Radix UI components provide built-in accessibility, proper ARIA attributes, and keyboard navigation.

### 6.3 Testing Accessibility

**Pattern:** Use Testing Library's accessibility queries
**Example:**

```typescript
// apps/client-react/src/home/__tests__/features.test.tsx:68
const switchElement = within(feature).getByRole("switch");
```

**Rationale:** Role-based queries ensure components are accessible and encourage proper semantic HTML.

### 6.4 Data Attributes for State

**Pattern:** Use `data-*` attributes for state-based styling
**Frequency:** Used across interactive components
**Example:**

```typescript
// packages/ui/src/patterns/feature/feature.tsx:23
data-expanded={isExpanded}

// packages/ui/src/components/button/button.module.scss:56
&[data-active="true"] {
  color: var(--color-text-muted);
  background: var(--color-surface-strong);
}
```

**Rationale:** Separates state from styling, makes state visible in DevTools, avoids class name conflicts.

---

## 7. Build & Tooling

### 7.1 Turborepo Configuration

**Pipeline:**

```json
// turbo.json:4-23
{
  "tasks": {
    "test": {
      "dependsOn": ["^build"],
      "inputs": ["$TURBO_DEFAULT$", "src/**/*.tsx", "src/**/*.ts", "test/**/*.ts", "test/**/*.tsx"]
    },
    "build": {
      "dependsOn": ["^build"],
      "env": ["DATABASE_URL", "NODE_ENV"],
      "outputs": ["dist/**", ".next/**", "!.next/cache/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "generate": {
      "dependsOn": ["^generate"],
      "cache": false
    },
    "lint": {}
  }
}
```

**Key patterns:**

- `dependsOn: ["^build"]` - Ensures dependencies are built first
- `cache: false` for dev and generate - These tasks shouldn't be cached
- `persistent: true` for dev - Keeps dev servers running
- Explicit `inputs` for test task - Only re-run when relevant files change
- `outputs` defined for build - Enables caching of build artifacts

**Rationale:** Proper task dependencies ensure correct build order, caching speeds up CI/CD.

### 7.2 Linting Configuration

**Tool:** ESLint 9 with flat config
**Pattern:** Shared base config extended by apps
**Example:**

```javascript
// packages/eslint-config/base.js:12-32
export const baseConfig = [
  js.configs.recommended,
  eslintConfigPrettier,
  ...tseslint.configs.recommended,
  {
    plugins: {
      turbo: turboPlugin,
    },
    rules: {
      "turbo/no-undeclared-env-vars": "warn",
    },
  },
  {
    plugins: {
      onlyWarn,
    },
  },
  {
    ignores: ["dist/**", "generated/"],
  },
];
```

**Key features:**

- ESLint 9 flat config format
- TypeScript ESLint integration
- Prettier integration (no conflicts)
- Turbo plugin for environment variable validation
- `only-warn` plugin - all errors become warnings (better DX)

**Rationale:** Shared config ensures consistency, warnings don't block development.

### 7.3 Prettier Configuration

**Pattern:** Shared config package
**Example:**

```javascript
// packages/prettier-config/prettier.config.mjs:7-17
const config = {
  printWidth: 100,
  useTabs: false,
  tabWidth: 2,
  trailingComma: "all",
  semi: true,
  singleQuote: false,
  quoteProps: "as-needed",
  arrowParens: "always",
  bracketSpacing: true,
};
```

**Usage:**

```json
// apps/client-next/package.json:13
{
  "prettier": "@repo/prettier-config"
}
```

**Rationale:** Centralized formatting rules ensure consistency across all packages.

### 7.4 Pre-commit Hooks

**Tool:** Husky + lint-staged
**Pattern:** Per-package lint-staged configs
**Example:**

```bash
# .husky/pre-commit:1
bunx lint-staged
```

```javascript
// apps/client-react/lint-staged.config.mjs:1-3
export default {
  "*.{ts,tsx,scss}": "eslint --fix",
};
```

**Rationale:** Catches issues before commit, auto-fixes formatting, prevents broken code from entering repo.

### 7.5 TypeScript Configuration

**Pattern:** Shared base configs extended by packages
**Structure:**

- `base.json` - Common TypeScript settings
- `nextjs.json` - Next.js-specific settings
- `react-library.json` - React library settings
- `node.json` - Node.js server settings

**Example:**

```json
// apps/client-next/tsconfig.json
{
  "extends": "@repo/typescript-config/nextjs.json"
}
```

**Rationale:** Shared configs ensure consistent TypeScript settings, reduce duplication.

### 7.6 Package Manager

**Tool:** Bun 1.2.2
**Rationale:** Fast installs, native TypeScript support, compatible with npm packages

**Example:**

```json
// package.json:14
{
  "packageManager": "bun@1.2.2"
}
```

### 7.7 Dependency Synchronization

**Tool:** Syncpack
**Purpose:** Ensure consistent versions across workspace
**Example:**

```json
// package.json:22-23
{
  "scripts": {
    "deps:check": "syncpack list-mismatches",
    "deps:fix": "syncpack fix-mismatches"
  }
}
```

**Rationale:** Prevents version conflicts, reduces bundle size, simplifies dependency management.

---

## 8. CI/CD Pipelines

### 8.1 Current State

**Status:** No CI/CD configuration found
**Missing:**

- `.github/workflows/` directory
- GitLab CI configuration
- CircleCI configuration

### 8.2 Recommended CI/CD Setup

**Based on Turborepo best practices:**

```yaml
# Recommended .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # For affected detection

      - uses: oven-sh/setup-bun@v1
        with:
          bun-version: 1.2.2

      - name: Install dependencies
        run: bun install

      - name: Lint
        run: bun run lint

      - name: Type check
        run: bun run check-types

      - name: Test
        run: bun run test

      - name: Build
        run: bun run build
```

**Rationale:** Validates code quality, runs tests, ensures builds succeed before merging.

---

## 9. Environment Management

### 9.1 File Hierarchy

**Pattern:** `.env` files per app, `.env.example` for documentation
**Found files:**

- `apps/client-next/.env`
- `apps/client-next/.env.production`
- `apps/client-react/.env`
- `apps/client-react/.env.production`
- `apps/server/.env`
- `apps/server/.env.example`
- `apps/server/.env.local.example`
- `packages/api/.env`
- `packages/api-mocks/.env`

**Example:**

```bash
# apps/server/.env.example:1-3
# base configuration
NODE_ENV=development
PORT=1337
```

**Rationale:** Per-app env files prevent conflicts, examples document required variables.

### 9.2 Naming Conventions

**Pattern:** Framework-specific prefixes for public variables
**Examples:**

```bash
# apps/client-next/.env:1
NEXT_PUBLIC_API_URL=http://localhost:3000/api/v1
```

**Rules:**

- Next.js: `NEXT_PUBLIC_*` for client-side variables
- Vite: `VITE_*` for client-side variables (if used)
- Server: No prefix needed

**Rationale:** Framework conventions ensure variables are properly exposed to client or kept server-side.

### 9.3 Environment Variable Validation

**Pattern:** Turborepo plugin validates env vars
**Example:**

```javascript
// packages/eslint-config/base.js:20-22
rules: {
  "turbo/no-undeclared-env-vars": "warn",
}
```

**Rationale:** Catches missing env vars early, prevents runtime errors.

### 9.4 API Client Configuration

**Pattern:** Configure API client with environment variables
**Example:**

```typescript
// apps/client-next/app/providers.tsx:17-19
client.setConfig({
  baseUrl: getClientEnv().NEXT_PUBLIC_API_URL,
});
```

**Rationale:** Centralizes API configuration, enables different URLs per environment.

---

## 10. Architecture Decisions

### 10.1 ADR Status

**Current state:** No formal ADRs found
**Location checked:** Root directory, `docs/` directory

### 10.2 Implicit Decisions Documented

Based on codebase analysis, these architectural decisions are evident:

#### Decision: Turborepo over Nx/Lerna

**Context:** Need monorepo tool for managing multiple apps and packages
**Decision:** Use Turborepo
**Rationale:**

- Simpler configuration than Nx
- Better caching than Lerna
- Good TypeScript support
- Active development by Vercel

#### Decision: OpenAPI-First API Development

**Context:** Need type-safe API integration between frontend and backend
**Decision:** Use OpenAPI schema with code generation
**Tools:** `@hey-api/openapi-ts` with `@tanstack/react-query` plugin

**Example:**

```typescript
// packages/api/openapi-ts.config.ts:3-11
export default defineConfig({
  input: "./openapi.yaml",
  output: {
    format: "prettier",
    lint: "eslint",
    path: "./src/apiClient",
  },
  plugins: [...defaultPlugins, "@hey-api/client-fetch", "@tanstack/react-query"],
});
```

**Rationale:**

- Single source of truth for API contracts
- Automatic type generation
- Automatic React Query hooks
- Reduces manual typing errors

#### Decision: MSW for API Mocking

**Context:** Need consistent API mocking for development and testing
**Decision:** Use Mock Service Worker (MSW)
**Rationale:**

- Works in both browser and Node.js
- Intercepts requests at network level
- Same mocks for development and testing
- Realistic API simulation

#### Decision: Component Library Structure

**Context:** Need organized component library
**Decision:** Four-tier hierarchy (Primitives → Components → Patterns → Templates)
**Rationale:**

- Clear progression from simple to complex
- Easy to find components
- Encourages composition over duplication

#### Decision: SCSS Modules over CSS-in-JS

**Context:** Need styling solution
**Decision:** SCSS Modules with Open Props
**Rationale:**

- No runtime cost (unlike CSS-in-JS)
- Scoped styles prevent conflicts
- SCSS provides familiar syntax
- Open Props provides design tokens
- Better performance than styled-components

#### Decision: Ladle over Storybook

**Context:** Need component documentation
**Decision:** Use Ladle
**Rationale:**

- Faster than Storybook
- Simpler configuration
- Vite-based (matches build tool)
- Sufficient for current needs

### 10.3 Recommended ADR Template

**Based on MADR (Markdown Any Decision Records):**

```markdown
# [Short title of decision]

**Status:** [Proposed | Accepted | Deprecated | Superseded]
**Date:** YYYY-MM-DD
**Deciders:** [List of people involved]

## Context and Problem Statement

[Describe the context and the question forcing the decision]

## Decision Drivers

- [Driver 1]
- [Driver 2]
- [Driver 3]

## Considered Options

- [Option 1]
- [Option 2]
- [Option 3]

## Decision Outcome

Chosen: [option 1]

### Consequences

**Good:**

- [Positive outcome 1]
- [Positive outcome 2]

**Bad:**

- [Negative consequence 1]
- [Negative consequence 2]

**Neutral:**

- [Neutral outcome 1]

## Confirmation

[How to verify this decision is working as expected]

## Links

- [Link to related ADRs]
- [Link to relevant documentation]
```

---

## 11. AI Agent Optimization

### 11.1 Existing AI Documentation

**Found:**

- `.claude/` directory with comprehensive agent guidelines
- `.claude/agents/pattern-scout.md` - This agent's configuration
- `.claude/patterns/` - Pattern documentation
  - `component-styling.md` - SCSS module guidelines
  - `storybook.md` - Ladle story guidelines
  - `index.md` - Core patterns overview
  - `insights.md`, `decisions.md`, `tailwind-conversion.md`

**Missing:**

- Root-level `AGENTS.md` for general AI agent guidance
- `CLAUDE.md` for Claude-specific context
- `llms.txt` for AI navigation

### 11.2 Recommended AGENTS.md Structure

**Location:** Root of repository
**Purpose:** Guide AI agents working on this codebase

```markdown
# Development Guidelines for AI Agents

## Tech Stack Summary

- **Monorepo:** Turborepo 2.4.2 with Bun 1.2.2
- **Frontend:** React 19, Next.js 15, Vite 6
- **State:** React Query (TanStack Query) for server state
- **Styling:** SCSS Modules + Open Props design tokens
- **UI:** Radix UI primitives + custom components
- **Testing:** Vitest + React Testing Library + MSW
- **API:** OpenAPI-driven with generated clients
- **Documentation:** Ladle for component stories

## Critical Do's

1. **Use workspace dependencies:** `"@repo/package": "*"` for internal packages
2. **Use React Query for ALL server data:** Never store API data in local state
3. **Use design tokens:** Reference `--color-*`, `--space-*`, `--text-size-*` from variables.scss
4. **Use SCSS Modules:** Every component gets `.module.scss` file
5. **Use MSW for mocking:** Add handlers to `@repo/api-mocks` package
6. **Follow component hierarchy:** Primitives → Components → Patterns → Templates
7. **Use explicit exports:** Add to package.json exports field
8. **Co-locate tests:** Put tests in `__tests__/` directory next to implementation
9. **Use Radix UI:** For interactive components (Switch, Select, etc.)
10. **Generate API clients:** Run `bun run generate` after OpenAPI changes

## Critical Don'ts

1. **Don't hardcode colors/spacing:** Always use design tokens
2. **Don't use default exports:** Use named exports for better tree-shaking
3. **Don't create global state:** Use React Query for server state, local state for UI
4. **Don't bypass package exports:** Import from package exports, not internal paths
5. **Don't skip Ladle stories:** Every component needs `.stories.tsx`
6. **Don't create component variables unnecessarily:** See component-styling.md guidelines
7. **Don't use CSS-in-JS:** Use SCSS Modules
8. **Don't commit .env files:** Use .env.example for documentation
9. **Don't skip tests:** Integration tests with MSW are required
10. **Don't modify generated files:** Regenerate from OpenAPI schema instead

## Commands (File-Scoped for Fast Feedback)

```bash
# Type check single file
bun tsc --noEmit path/to/file.ts

# Lint single file
bun eslint path/to/file.ts --fix

# Test single file
bun vitest run path/to/file.test.ts

# Run specific app
bun run dev --filter=client-react
bun run dev --filter=client-next
bun run dev --filter=server

# Generate API client
cd packages/api && bun run build

# Check dependency versions
bun run deps:check
```

## Safety and Permissions

**Safe commands (no confirmation needed):**

- Type checking
- Linting
- Running tests
- Building packages
- Generating API clients

**Require confirmation:**

- Installing dependencies
- Modifying package.json exports
- Changing Turborepo configuration
- Modifying shared configs (eslint, prettier, typescript)
- Deploying applications

## Component Creation Workflow

1. Use turbo generator: `cd packages/ui && bun run generate:component`
2. Create component file: `src/components/name/name.tsx`
3. Create styles: `src/components/name/name.module.scss`
4. Create stories: `src/components/name/name.stories.tsx`
5. Add to package exports: `"./name": "./src/components/name/name.tsx"`
6. Run sync script: `bun run sync-package-exports`

## Testing Approach

**Integration tests are primary strategy:**

```typescript
// Test component with API integration
import { serverWorker } from "@repo/api-mocks/serverWorker";
import { getDataHandlers } from "@repo/api-mocks/handlers";

it("should handle empty state", async () => {
  serverWorker.use(getDataHandlers.emptyHandler());
  renderApp();
  await expect(screen.findByText("No data")).resolves.toBeInTheDocument();
});
```

**Test all states:** loading, empty, error, success
**Use MSW handlers:** Centralized in `@repo/api-mocks`
**Focus on behavior:** Not implementation details

## Pattern Files to Reference

Before creating/modifying components, read:

1. `.claude/patterns/index.md` - Start here
2. `.claude/patterns/component-styling.md` - For SCSS modules
3. `.claude/patterns/storybook.md` - For Ladle stories

## Common Pitfalls

1. **Forgetting to add package exports:** Components won't be importable
2. **Using hardcoded values:** Always use design tokens
3. **Skipping loading/error states:** All data fetching needs full state handling
4. **Not using MSW handlers:** Tests should use centralized mocks
5. **Creating unnecessary component variables:** See component-styling.md criteria
```

### 11.3 Recommended CLAUDE.md

**Location:** Root of repository
**Purpose:** Project context for Claude Code memory

```markdown
# Turborepo Playground - Project Context

## Project Overview

A monorepo showcasing modern React/Next.js architecture with:

- Multiple frontend implementations (React + Vite, Next.js)
- Express backend with PostgreSQL
- Shared component library
- OpenAPI-driven API development
- Comprehensive testing with MSW

## Key Architectural Patterns

### Component Library Structure

Four-tier hierarchy in `packages/ui/`:

- **Primitives:** Basic building blocks (skeleton)
- **Components:** Reusable UI (button, switch, select, info)
- **Patterns:** Composed features (feature, navigation, socials)
- **Templates:** Page layouts (frame)

### API Integration

1. Define API in `packages/api/openapi.yaml`
2. Generate client: `cd packages/api && bun run build`
3. Use generated hooks: `import { getFeaturesOptions } from "@repo/api/reactQueries"`
4. Create MSW handlers in `packages/api-mocks/`

### Styling System

- SCSS Modules for component styles
- Open Props for base design tokens
- Two-tier token system (Core → Semantic)
- Design tokens in `packages/ui/src/styles/variables.scss`

### Testing Strategy

- Integration tests with MSW
- Test all states: loading, empty, error, success
- Co-located tests in `__tests__/` directories
- Shared test utilities in `@repo/vitest-config`

## Recent Changes

[Update this section when making significant changes]

## Known Issues

[Document any known issues or technical debt]
```

### 11.4 Recommended llms.txt

**Location:** Root of repository
**Purpose:** Help AI navigate the codebase

```
# Turborepo Playground

## Key Directories

/apps/client-next - Next.js 15 application
/apps/client-react - React 19 + Vite application
/apps/server - Express API server
/packages/ui - Shared component library
/packages/api - OpenAPI client (generated)
/packages/api-mocks - MSW handlers and mocks
/packages/*-config - Shared configurations

## Important Files

/turbo.json - Turborepo task configuration
/package.json - Root workspace configuration
/packages/ui/src/styles/variables.scss - Design tokens
/packages/api/openapi.yaml - API schema
/.claude/patterns/ - Development patterns

## Component Library

Hierarchy: Primitives → Components → Patterns → Templates
Location: packages/ui/src/
Exports: packages/ui/package.json

## Testing

Framework: Vitest + React Testing Library
Mocking: MSW (Mock Service Worker)
Location: __tests__/ directories
Config: packages/vitest-config/

## Styling

System: SCSS Modules + Open Props
Tokens: packages/ui/src/styles/variables.scss
Pattern: .module.scss files co-located with components

## API

Schema: packages/api/openapi.yaml
Generation: @hey-api/openapi-ts
Client: packages/api/src/apiClient/
Hooks: packages/api/src/apiClient/@tanstack/react-query.gen.ts
```

---

## 12. Performance Standards

### 12.1 Build Performance

**Current state:** No explicit performance budgets defined

**Recommended targets:**

- Full build: < 2 minutes
- Incremental build: < 30 seconds
- Cache hit ratio: > 80%

**Monitoring:**

```bash
# Check build times
bun run build --summarize

# Check cache effectiveness
turbo run build --dry-run
```

### 12.2 Bundle Size

**Current state:** No bundle size budgets defined

**Recommended approach:**

```json
// next.config.ts - Add bundle analyzer
{
  "webpack": (config) => {
    if (process.env.ANALYZE) {
      const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');
      config.plugins.push(new BundleAnalyzerPlugin());
    }
    return config;
  }
}
```

### 12.3 Runtime Performance

**Patterns observed:**

- React Query caching reduces API calls
- SCSS Modules enable CSS tree-shaking
- Named exports enable better tree-shaking
- Lazy loading not currently implemented

**Recommended improvements:**

1. Add Next.js dynamic imports for large components
2. Implement route-based code splitting
3. Add bundle size monitoring
4. Set performance budgets

---

## 13. Security Patterns

### 13.1 Environment Variables

**Pattern:** Never commit secrets
**Status:** `.env` files are gitignored
**Example:**

```bash
# .gitignore includes:
.env
.env.local
.env.*.local
```

**Best practice:** Use `.env.example` files to document required variables

### 13.2 Dependency Security

**Current state:** No automated security scanning found

**Recommended:**

```json
// package.json - Add security scripts
{
  "scripts": {
    "security:audit": "bun audit",
    "security:check": "bunx audit-ci --moderate"
  }
}
```

### 13.3 API Security

**Pattern:** Helmet.js for Express security headers
**Example:**

```json
// apps/server/package.json:23
{
  "dependencies": {
    "helmet": "^8.0.0"
  }
}
```

**Rationale:** Helmet sets secure HTTP headers automatically.

### 13.4 Input Validation

**Pattern:** Zod for runtime type validation
**Frequency:** Used in both frontend and backend
**Example:**

```json
// apps/client-next/package.json:26
// apps/server/package.json:26
{
  "dependencies": {
    "zod": "^3.24.2"
  }
}
```

**Rationale:** Zod provides runtime validation matching TypeScript types.

---

## 14. Git Workflow

### 14.1 Branch Strategy

**Current state:** Not explicitly documented

**Recommended:** GitHub Flow (simple, effective for small teams)

- `main` branch is always deployable
- Feature branches for new work
- Pull requests for code review
- Merge to main after approval

### 14.2 Commit Conventions

**Current state:** No enforced commit message format

**Recommended:** Conventional Commits

```
feat(ui): add Button component
fix(api): handle network errors
docs(readme): update installation steps
chore(deps): update dependencies
test(features): add empty state test
```

**Implementation:**

```bash
# Install commitlint
bun add -D @commitlint/cli @commitlint/config-conventional

# Add husky hook
echo "bunx commitlint --edit \$1" > .husky/commit-msg
```

### 14.3 Pre-commit Hooks

**Current implementation:**

```bash
# .husky/pre-commit:1
bunx lint-staged
```

**Per-package lint-staged:**

```javascript
// apps/client-react/lint-staged.config.mjs:1-3
export default {
  "*.{ts,tsx,scss}": "eslint --fix",
};
```

**Rationale:** Catches issues early, auto-fixes formatting, prevents broken code.

### 14.4 Pre-push Hooks

**Current implementation:**

```bash
# .husky/pre-push:1
cd apps/client-react && bun run test --watch=false
```

**Limitation:** Only tests client-react app

**Recommended improvement:**

```bash
# .husky/pre-push
bun run test
```

**Rationale:** Test all packages before pushing to catch more issues.

---

## 15. Anti-Patterns Observed

### 15.1 Inconsistent Test Coverage

**What:** Only client-react app has comprehensive tests
**Where:** client-next and server apps lack tests
**Why problematic:** Uneven test coverage creates blind spots
**Better approach:** Add tests to all apps, especially server API endpoints

### 15.2 No CI/CD Pipeline

**What:** No automated testing/deployment
**Where:** Missing `.github/workflows/`
**Why problematic:** Manual testing is error-prone, no automated quality gates
**Better approach:** Add GitHub Actions workflow for CI/CD

### 15.3 Duplicate QueryClient Configuration

**What:** QueryClient configured separately in each app
**Where:**

- `apps/client-next/app/providers.tsx:9-15`
- `packages/api/src/reactQueryClient.tsx:3-9`

**Why problematic:** Configuration drift, harder to maintain
**Better approach:** Export configured client from `@repo/api` package

### 15.4 Missing Environment Variable Documentation

**What:** Some apps have `.env` but no `.env.example`
**Where:** client-next, client-react, packages/api, packages/api-mocks
**Why problematic:** New developers don't know what variables are needed
**Better approach:** Create `.env.example` for all packages with env vars

### 15.5 No Bundle Size Monitoring

**What:** No tracking of bundle sizes
**Where:** Build configuration
**Why problematic:** Bundle size can grow unnoticed
**Better approach:** Add bundle analyzer and size budgets

### 15.6 Hardcoded API URLs in MSW Handlers

**What:** API URLs constructed with string concatenation
**Where:**

```typescript
// packages/api-mocks/src/handlers/features/getFeatures.ts:7-8
const URL =
  getEnv().NODE_ENV === "development" ? "api/v1/features" : "http://localhost:5173/api/v1/features";
```

**Why problematic:** Brittle, port conflicts, hard to configure
**Better approach:** Use environment variables for base URL

---

## 16. Quick Reference for AI Agents

### Essential Patterns

```typescript
// Import from package exports
import { Button } from "@repo/ui/button";
import { getFeaturesOptions } from "@repo/api/reactQueries";

// Use React Query for data fetching
const { data, isPending, error } = useQuery(getFeaturesOptions());

// Use design tokens in SCSS
.component {
  padding: var(--space-md);
  color: var(--color-text-default);
  background: var(--color-surface-base);
}

// Create MSW handler
export const getDataHandlers = {
  defaultHandler: () => http.get(URL, async () => HttpResponse.json(mockData)),
  emptyHandler: () => http.get(URL, async () => HttpResponse.json({ items: [] })),
  errorHandler: () => http.get(URL, async () => new HttpResponse("Error", { status: 500 })),
};

// Test with MSW
it("should handle empty state", async () => {
  serverWorker.use(getDataHandlers.emptyHandler());
  renderApp();
  await expect(screen.findByText("No data")).resolves.toBeInTheDocument();
});
```

### Critical Do's

1. ✅ Use `"@repo/package": "*"` for workspace dependencies
2. ✅ Use React Query for ALL server data
3. ✅ Use design tokens from `variables.scss`
4. ✅ Use SCSS Modules (`.module.scss`)
5. ✅ Add package exports to `package.json`
6. ✅ Create Ladle stories (`.stories.tsx`)
7. ✅ Write integration tests with MSW
8. ✅ Use Radix UI for interactive components
9. ✅ Follow component hierarchy (Primitives → Components → Patterns → Templates)
10. ✅ Use named exports (not default)

### Critical Don'ts

1. ❌ Don't hardcode colors/spacing
2. ❌ Don't use default exports
3. ❌ Don't store server data in local state
4. ❌ Don't import from internal paths
5. ❌ Don't skip component stories
6. ❌ Don't create unnecessary component variables
7. ❌ Don't use CSS-in-JS
8. ❌ Don't commit `.env` files
9. ❌ Don't skip tests
10. ❌ Don't modify generated files

### File-Scoped Commands (Fast Feedback)

```bash
# Type check
bun tsc --noEmit path/to/file.ts

# Lint
bun eslint path/to/file.ts --fix

# Test
bun vitest run path/to/file.test.ts

# Run specific app
bun run dev --filter=client-react

# Generate API client
cd packages/api && bun run build
```

### Component Creation Checklist

- [ ] Create component file: `src/components/name/name.tsx`
- [ ] Create styles: `src/components/name/name.module.scss`
- [ ] Create stories: `src/components/name/name.stories.tsx`
- [ ] Add to package exports in `package.json`
- [ ] Use design tokens (no hardcoded values)
- [ ] Add TypeScript types
- [ ] Handle all component states
- [ ] Add accessibility attributes
- [ ] Write integration tests

### Testing Checklist

- [ ] Test loading state
- [ ] Test empty state
- [ ] Test error state
- [ ] Test success state
- [ ] Test user interactions
- [ ] Use MSW handlers from `@repo/api-mocks`
- [ ] Use Testing Library queries (getByRole, findByText)
- [ ] Test behavior, not implementation

---

## Confidence & Coverage Notes

### High Confidence Patterns (5+ instances)

- Package architecture and naming
- Component structure and styling
- React Query usage
- SCSS Modules pattern
- MSW integration
- Testing patterns
- Design token usage
- Turborepo configuration

### Medium Confidence Patterns (3-4 instances)

- Ladle story structure
- Environment variable management
- Pre-commit hooks
- TypeScript configuration

### Low Confidence Patterns (1-2 instances)

- API security patterns (only server has helmet)
- Performance monitoring (not implemented)
- CI/CD (not implemented)

### Coverage Gaps

**Missing documentation:**

- [ ] No formal ADRs
- [ ] No root-level AGENTS.md
- [ ] No CLAUDE.md
- [ ] No llms.txt
- [ ] No CI/CD configuration
- [ ] No performance budgets
- [ ] No security scanning

**Inconsistent patterns:**

- [ ] Test coverage varies by app
- [ ] Environment variable documentation incomplete
- [ ] QueryClient configuration duplicated

**Recommended additions:**

- [ ] Add CI/CD pipeline
- [ ] Create AGENTS.md
- [ ] Add bundle size monitoring
- [ ] Standardize test coverage
- [ ] Document all environment variables
- [ ] Add security scanning
- [ ] Create formal ADRs for key decisions

---

## Implementation Priority

### Immediate (Week 1)

1. **Create AGENTS.md** - Guide AI agents working on codebase
2. **Add .env.example files** - Document required environment variables
3. **Standardize QueryClient** - Export from @repo/api package
4. **Add CI/CD pipeline** - GitHub Actions for automated testing

### High Priority (Week 2-3)

5. **Add tests to client-next** - Match client-react test coverage
6. **Add tests to server** - API endpoint testing
7. **Create ADRs** - Document key architectural decisions
8. **Add bundle size monitoring** - Prevent bundle bloat
9. **Fix MSW URL configuration** - Use environment variables

### Medium Priority (Month 1)

10. **Add security scanning** - Dependabot or Snyk
11. **Add performance budgets** - Monitor build and bundle sizes
12. **Create CLAUDE.md** - Project context for Claude Code
13. **Create llms.txt** - Help AI navigate codebase
14. **Add commit message linting** - Enforce conventional commits

### Ongoing

- Keep AGENTS.md updated with new patterns
- Record architectural decisions as ADRs
- Maintain test coverage above 80%
- Monitor and optimize bundle sizes
- Review and refactor anti-patterns
- Update documentation as codebase evolves

---

## Extraction Metadata

**Extraction completed:** 2025-11-13
**Files analyzed:** 100+
**Patterns documented:** 15 major categories
**Confidence level:** High (based on consistent patterns)
**Codebase state:** Active development, well-structured
**Recommended next steps:** Implement immediate priority items, create AGENTS.md

**This document should be updated as the codebase evolves.**

