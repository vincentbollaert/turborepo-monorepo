# 08. Build & Tooling

> **Quick Guide:** Turborepo 2.4.2 with Bun for monorepo orchestration. ESLint 9 flat config with `only-warn` plugin. Prettier shared config. Husky + lint-staged for pre-commit hooks. Shared TypeScript configs. Bun 1.2.2 package manager (3-10x faster installs).

## 8.1 Turborepo Configuration

**ACTUAL IMPLEMENTATION: Turborepo 2.4.2 with Bun 1.2.2**

**Location:** `turbo.json` in repository root

### 8.1.1 Task Pipeline

**Pattern:** Define task dependencies and caching behavior

**Key concepts:**
- `dependsOn: ["^build"]` - Run dependency tasks first (topological order)
- `outputs` - Define what files to cache
- `inputs` - Specify which files trigger cache invalidation
- `cache: false` - Disable caching for tasks with side effects
- `persistent: true` - Keep dev servers running

**ACTUAL tasks in this codebase:**

```json
{
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "env": ["DATABASE_URL", "NODE_ENV"],
      "outputs": ["dist/**", ".next/**", "!.next/cache/**"]
    },
    "test": {
      "dependsOn": ["^build"],
      "inputs": ["$TURBO_DEFAULT$", "src/**/*.tsx", "src/**/*.ts", "test/**/*.ts", "test/**/*.tsx"]
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

### 8.1.2 Caching Strategies

**What gets cached:**
- Build outputs (`dist/`, `.next/`)
- Test results (when `cache: true`)
- Lint results

**What doesn't get cached:**
- Dev servers (`cache: false`)
- Code generation (`cache: false` - generates files)
- Tasks with side effects

**Cache invalidation triggers:**
- Source file changes
- Dependency changes
- Environment variable changes (when in `env` array)
- Global dependencies changes (`.env`, `tsconfig.json`)

### 8.1.3 Remote Caching

**Optional but recommended for teams:**

```json
{
  "remoteCache": {
    "signature": true
  }
}
```

**Setup:** Link Vercel account, set `TURBO_TOKEN` and `TURBO_TEAM` env vars. Shares cache across team and CI.

### 8.1.4 Environment Variables

**Pattern:** Declare environment variables in `env` array

```json
{
  "tasks": {
    "build": {
      "env": ["DATABASE_URL", "NODE_ENV", "NEXT_PUBLIC_API_URL"]
    }
  }
}
```

**Why:** Turbo includes env vars in cache key - different values trigger rebuilds

> See [examples.md - Turborepo Configuration](./examples.md#81-turboreponxvite-configuration) for complete examples

## 8.2 Linting Configuration

**ACTUAL IMPLEMENTATION: ESLint 9 with flat config**

**Location:** `packages/eslint-config/`

### 8.2.1 Shared Config Pattern

**Pattern:** Shared base config extended by apps and packages

**Structure:**
```
packages/eslint-config/
├── base.js           # Base config for all packages
├── react.js          # React-specific rules
├── next.js           # Next.js-specific rules
└── package.json
```

**Base config includes:** Recommended JS/TS rules, Prettier conflict resolution, Turbo env var validation, `only-warn` plugin (converts errors to warnings), and ignore patterns for build outputs.

### 8.2.2 Key Features

**ESLint 9 flat config:**
- New configuration format (replaces `.eslintrc`)
- Simpler, more flexible
- Better TypeScript support

**Plugins used:**
- `@typescript-eslint` - TypeScript linting
- `eslint-config-prettier` - Disable conflicting Prettier rules
- `eslint-plugin-turbo` - Validate environment variables
- `eslint-plugin-only-warn` - Convert errors to warnings (better DX)

**Why `only-warn`:**
- Developers aren't blocked by errors
- Issues are visible but not disruptive
- CI can still fail on warnings if needed

### 8.2.3 ESLint vs Biome (2025)

**Current choice: ESLint 9**
- Mature ecosystem
- Extensive plugin support
- Good TypeScript integration

**Biome (emerging alternative):**
- ✅ Extremely fast (Rust-based)
- ✅ Combined linter + formatter
- ✅ Drop-in Prettier replacement
- ❌ Smaller ecosystem
- ❌ Fewer plugins

**Recommendation:**
- Stick with ESLint for now (mature, stable)
- Monitor Biome adoption (may switch in 2025-2026)
- Consider Biome for greenfield projects

> See [examples.md - Linting Configuration](./examples.md#82-linting-configuration) for configuration examples

## 8.3 Formatting

**ACTUAL IMPLEMENTATION: Prettier with shared config**

**Location:** `packages/prettier-config/`

### 8.3.1 Shared Configuration

**Pattern:** Centralized Prettier config shared across all packages

**Key settings:** 100 char line width, 2-space indentation, double quotes, required semicolons, trailing commas everywhere, always use arrow parens.

**Usage in packages:**

```json
{
  "prettier": "@repo/prettier-config"
}
```

### 8.3.2 Format Settings Explained

**Line width:** `100` characters
- Balances readability and screen width
- Works well with side-by-side diffs
- Alternative: `120` for wider screens

**Quotes:** Double quotes (`"`)
- Consistent with JSON
- Fewer escapes in JSX
- Alternative: Single quotes (`'`) for smaller bundles

**Semicolons:** Required
- Explicit statement termination
- Avoids ASI issues
- Alternative: No semicolons (modern style)

**Trailing commas:** All
- Cleaner git diffs
- Easier to reorder items
- Required for multi-line

**Arrow parens:** Always
- Consistent with multi-param functions
- Easier to add params later

### 8.3.3 Editor Integration

**VS Code:** Install Prettier extension

```json
// .vscode/settings.json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

**EditorConfig:** Basic formatting rules

```ini
# .editorconfig
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.{js,jsx,ts,tsx,json}]
indent_style = space
indent_size = 2
```

### 8.3.4 Prettier vs Biome

**Current choice: Prettier**
- Industry standard
- Extensive language support
- Battle-tested

**Biome alternative:**
- ~20x faster than Prettier
- Combined linter + formatter
- Compatible with Prettier config
- Growing adoption

**When to switch:**
- Large monorepos (speed matters)
- Greenfield projects
- When Biome plugin ecosystem matures

> See [examples.md - Formatting](./examples.md#83-formatting) for configuration examples

## 8.4 Pre-commit Hooks

**ACTUAL IMPLEMENTATION: Husky + lint-staged**

**Location:** `.husky/` directory, per-package `lint-staged.config.mjs`

### 8.4.1 Husky Configuration

**Pre-commit hook:**

```bash
# .husky/pre-commit
bunx lint-staged
```

**Pre-push hook:**

```bash
# .husky/pre-push
cd apps/client-react && bun run test --watch=false
```

### 8.4.2 lint-staged Setup

**Pattern:** Per-package configuration runs ESLint with auto-fix on staged TypeScript and SCSS files.

**What runs on commit:**
- ESLint with auto-fix
- Prettier formatting (via ESLint integration)
- Only on staged files (fast!)

### 8.4.3 What to Run Pre-commit

**Recommended (fast):**
- ✅ Linting with auto-fix
- ✅ Formatting
- ✅ Type checking (--noEmit, fast)

**NOT recommended (slow):**
- ❌ Full test suite (run in pre-push or CI)
- ❌ Full build (too slow)
- ❌ E2E tests (run in CI)

### 8.4.4 Advanced Patterns

**Run affected tests only:**

```javascript
// lint-staged.config.mjs
export default {
  "*.{ts,tsx}": (filenames) => {
    const testFiles = filenames
      .filter(f => f.includes('.test.'))
      .map(f => `bun test ${f}`)
    return [
      "eslint --fix",
      ...testFiles,
    ];
  },
};
```

**Type check only changed files:**

```javascript
export default {
  "*.{ts,tsx}": [
    "eslint --fix",
    () => "tsc --noEmit --incremental",
  ],
};
```

> See [examples.md - Pre-commit Hooks](./examples.md#84-pre-commit-hooks) for examples

## 8.5 TypeScript Configuration

**ACTUAL IMPLEMENTATION: Shared configs in `@repo/typescript-config`**

**Location:** `packages/typescript-config/`

### 8.5.1 Shared Config Pattern

**Structure:**
```
packages/typescript-config/
├── base.json              # Common settings for all packages
├── nextjs.json            # Next.js-specific settings
├── react-library.json     # React library settings
├── node.json              # Node.js server settings
└── package.json
```

**Usage:**

```json
// apps/client-next/tsconfig.json
{
  "extends": "@repo/typescript-config/nextjs.json",
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

### 8.5.2 Base Configuration

**Recommended settings:**

```json
{
  "compilerOptions": {
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "preserve",
    "incremental": true,
    "moduleResolution": "bundler"
  }
}
```

### 8.5.3 Path Aliases

**Pattern:** Configure in both tsconfig and build tool

```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"],
      "@lib/*": ["./src/lib/*"]
    }
  }
}
```

**Benefits:**
- Clean imports: `import { Button } from "@components/button"`
- No relative path hell: No `../../../components`
- Easy refactoring

> See [Code Conventions - Import/Export Patterns](../03-code-conventions/docs.md#33-importexport-patterns) for import patterns

## 8.6 Package Manager

**ACTUAL IMPLEMENTATION: Bun 1.2.2**

**Location:** `package.json` (root)

```json
{
  "packageManager": "bun@1.2.2"
}
```

### 8.6.1 Why Bun?

**Benefits:**
- ⚡ Extremely fast installs (3-10x faster than npm/yarn)
- 🔧 Native TypeScript support
- 🎯 Compatible with npm packages
- 📦 Built-in test runner
- 🔨 Built-in bundler

**Alternatives:**
- **npm** - Slowest, but universal
- **yarn** - Faster, PnP mode
- **pnpm** - Efficient disk usage, fast
- **bun** - Fastest, modern

### 8.6.2 Workspaces

**Pattern:** Monorepo with workspaces

```json
{
  "workspaces": ["apps/*", "packages/*"]
}
```

**Benefits:**
- Shared dependencies across packages
- Hoist common dependencies
- Link local packages automatically

## 8.7 Dependency Synchronization

**ACTUAL IMPLEMENTATION: Syncpack**

**Purpose:** Ensure consistent versions across workspace

**Commands:**

```json
{
  "scripts": {
    "deps:check": "syncpack list-mismatches",
    "deps:fix": "syncpack fix-mismatches"
  }
}
```

### 8.7.1 Why Syncpack?

**Problem:**
- Different packages use different versions of React
- `@types/react` version mismatch
- Duplicate dependencies in bundle

**Solution:**
- Syncpack identifies version mismatches
- Auto-fix to use consistent versions
- Reduces bundle size

**Example:**

```bash
$ bun run deps:check
❌ react: 18.2.0, 18.3.0, 19.0.0 (3 versions!)
❌ @types/react: 18.2.0, 18.3.0 (2 versions!)

$ bun run deps:fix
✅ Updated react to 19.0.0 across all packages
✅ Updated @types/react to 18.3.0 across all packages
```

## 8.8 Build Optimization

**Recommended practices for production builds**

### 8.8.1 Bundle Analysis

**Tools:**
- `rollup-plugin-visualizer` - Vite/Rollup
- `@next/bundle-analyzer` - Next.js
- `webpack-bundle-analyzer` - Webpack

**Setup:**

```bash
bun add -D rollup-plugin-visualizer
bun run build --analyze
```

### 8.8.2 Code Splitting

**Patterns:**
- Route-based splitting (automatic in Next.js)
- Component lazy loading with `React.lazy()`
- Vendor chunk splitting
- Dynamic imports for large libraries

### 8.8.3 Tree Shaking

**Requirements:**
- ES modules (not CommonJS)
- Named exports (not default)
- Side-effect-free code

**Mark as side-effect-free:**

```json
{
  "sideEffects": false
}
```

Or specify files with side effects:

```json
{
  "sideEffects": ["*.css", "*.scss"]
}
```

### 8.8.4 Performance Budgets

**Set size limits:**

```json
{
  "build": {
    "chunkSizeWarningLimit": 500,
    "rollupOptions": {
      "output": {
        "manualChunks": {
          "vendor": ["react", "react-dom"]
        }
      }
    }
  }
}
```

**Monitor:**
- Initial bundle < 200KB (gzip)
- Largest chunk < 500KB
- Total bundle < 1MB

> See [examples.md - Build Optimization](./examples.md#88-build-optimization) for implementation examples
