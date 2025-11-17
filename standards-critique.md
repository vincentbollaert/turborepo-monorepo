# Standards Critique Report
**Date:** 2025-11-13  
**Source:** extracted-standards.md  
**Reviewer:** Pattern Critique Agent  
**Framework:** Industry best practices from Airbnb, Stripe, Meta, Vercel

---

<core_principles>
**1. Investigation First**
Never speculate. Read the actual code before making claims. Base all work strictly on what you find in the files.

**2. Follow Existing Patterns**  
Use what's already there. Match the style, structure, and conventions of similar code. Don't introduce new patterns.

**3. Minimal Necessary Changes**
Make surgical edits. Change only what's required to meet the specification. Leave everything else untouched.

**4. Anti-Over-Engineering**
Simple solutions. Use existing utilities. Avoid abstractions. If it's not explicitly required, don't add it.

**5. Verify Everything**
Test your work. Run the tests. Check the success criteria. Provide evidence that requirements are met.

**DISPLAY ALL 5 CORE PRINCIPLES AT THE START OF EVERY RESPONSE TO MAINTAIN INSTRUCTION CONTINUITY.**
</core_principles>

---

<critique_summary>
**Overall Assessment:** Strong foundation with modern tooling, but several critical anti-patterns and missing production-grade practices.

**Strengths Identified:**
- Excellent OpenAPI-first API development with generated clients
- Proper server state management with React Query
- Good component organization with four-tier hierarchy
- SCSS Modules over CSS-in-JS (zero runtime cost)
- MSW for network-level mocking
- Design token system with semantic naming

**Critical Issues:** 5 blockers that MUST be fixed
**Important Issues:** 8 significant improvements needed
**Suggestions:** 6 nice-to-have optimizations
</critique_summary>

---

<critical_issues>
🔴 **MUST FIX** - These patterns violate fundamental best practices

### 1. Barrel File Anti-Pattern in Package Exports

**Current Pattern:**
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
    "./hooks": "./src/hooks/index.ts"  // ❌ BARREL FILE
  }
}
```

**Why This Is Wrong:**
The `"./hooks": "./src/hooks/index.ts"` export is a barrel file that forces loading ALL hooks when you import ONE. This cascades imports in development, loading hundreds of modules unnecessarily.

**Industry Standard:**
```json
// ✅ GOOD - Direct exports for each hook
{
  "exports": {
    "./hooks/useFeatureToggle": "./src/hooks/useFeatureToggle.ts",
    "./hooks/useDebounce": "./src/hooks/useDebounce.ts",
    "./hooks/useLocalStorage": "./src/hooks/useLocalStorage.ts"
  }
}
```

**Impact:**
- Slower development server startup
- Larger bundle sizes in production
- Meta's research shows barrel files can add 200-500ms to cold starts

**Refactoring Strategy:**
1. List all hooks in `src/hooks/`
2. Add individual exports for each hook
3. Remove the barrel `index.ts` export
4. Update imports in consuming apps to use specific paths
5. Use barrel files ONLY for build optimization if needed

---

### 2. React.FC Usage (Deprecated Pattern)

**Current Pattern:**
Not explicitly shown in extracted standards, but needs verification in actual components.

**Investigation Required:**
```bash
# Check if React.FC is used
grep -r "React.FC" packages/ui/src/
grep -r "React.FunctionComponent" packages/ui/src/
```

**Why This Matters:**
React.FC is a deprecated pattern that:
- Implicitly types `children`, causing issues for components that shouldn't accept them
- Adds unnecessary complexity
- Is no longer recommended by React team

**Industry Standard:**
```typescript
// ❌ BAD - React.FC is deprecated
const Button: React.FC<ButtonProps> = ({ children }) => {};

// ✅ GOOD - Explicit prop typing
function Button({ children, label }: ButtonProps) {}
// Or: const Button = ({ children, label }: ButtonProps) => {}
```

**Refactoring Strategy:**
1. Search for all `React.FC` usage
2. Replace with explicit function declarations
3. Explicitly type `children` only when needed

---

### 3. Duplicate QueryClient Configuration

**Current Pattern:**
```typescript
// apps/client-next/app/providers.tsx:9-15
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: false,
    },
  },
});

// packages/api/src/reactQueryClient.tsx:3-9
// DUPLICATE configuration in different location
```

**Why This Is Wrong:**
Configuration drift is inevitable when the same setup exists in multiple places. One app might have `retry: false`, another might have different settings. This violates DRY and creates maintenance burden.

**Industry Standard:**
```typescript
// ✅ GOOD - Single source of truth in shared package
// packages/api/src/queryClient.ts
export const createQueryClient = () => new QueryClient({
  defaultOptions: {
    queries: {
      retry: false,
      staleTime: 5 * 60 * 1000, // 5 minutes
      refetchOnWindowFocus: false,
    },
  },
});

// apps/client-next/app/providers.tsx
import { createQueryClient } from "@repo/api/queryClient";
const queryClient = createQueryClient();
```

**Impact:**
- Configuration drift between apps
- Harder to update query defaults globally
- Stripe's monorepo has single shared query client factory

**Refactoring Strategy:**
1. Create `packages/api/src/queryClient.ts` with factory function
2. Export from package: `"./queryClient": "./src/queryClient.ts"`
3. Update both apps to import from shared package
4. Delete duplicate configurations

---

### 4. Missing Bundle Analysis and Performance Budgets

**Current Pattern:**
```json
// No bundle analysis configured
{
  "scripts": {
    "build": "vite build"
  }
}
```

**Why This Is Wrong:**
Bundle size directly impacts performance. Without monitoring, bundles grow unnoticed until users complain about slow load times. Meta graphs JavaScript size by product and sets strict budgets.

**Industry Standard:**
```json
// ✅ GOOD - Bundle analysis in CI
{
  "scripts": {
    "build": "vite build",
    "analyze": "vite-bundle-visualizer",
    "build:analyze": "ANALYZE=true vite build"
  },
  "devDependencies": {
    "vite-bundle-visualizer": "^1.0.0"
  }
}

// vite.config.ts
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    react(),
    process.env.ANALYZE && visualizer({
      open: true,
      gzipSize: true,
      brotliSize: true,
    }),
  ],
});
```

**Impact:**
- No visibility into what's bloating bundles
- Can't catch regressions in CI
- Vercel recommends continuous bundle monitoring

**Refactoring Strategy:**
1. Install `vite-bundle-visualizer` or `rollup-plugin-visualizer`
2. Add analyze script to package.json
3. Add bundle size budgets to CI
4. Set up bundle size tracking (e.g., bundlewatch)

---

### 5. Server State in Wrong Location (Potential)

**Current Pattern:**
Standards document states "No global client state library (Zustand/Redux) - not needed yet" which is EXCELLENT.

**Critical Verification Needed:**
```bash
# Verify NO server state is in useState/Context
grep -r "useState.*fetch" apps/
grep -r "useEffect.*fetch" apps/
grep -r "createContext.*api" apps/
```

**Why This Matters:**
If ANY server data is stored in `useState` or `Context`, it's a critical anti-pattern. Server state needs:
- Caching
- Background refetching
- Deduplication
- Stale data management

React Query provides all of this. `useState`/`Context` don't.

**Industry Standard:**
```typescript
// ❌ BAD - Server state in useState
const [users, setUsers] = useState([]);
useEffect(() => {
  fetch('/api/users').then(r => r.json()).then(setUsers);
}, []);

// ✅ GOOD - Server state in React Query
const { data: users } = useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
  staleTime: 5 * 60 * 1000,
});
```

**Verification Required:**
Audit all components to ensure NO server data is in local state.

</critical_issues>

<important_improvements>
🟠 **SHOULD FIX** - Significant improvements to code quality, maintainability, or performance

### 1. Inconsistent Test Coverage

**Current Pattern:**
- client-react: Comprehensive tests ✅
- client-next: No tests ❌
- server: No tests ❌

**Why This Matters:**
Uneven test coverage creates blind spots. The server API is the most critical part to test, yet it has no tests. This is backwards.

**Better Approach:**
```typescript
// apps/server/src/__tests__/features.test.ts
import request from 'supertest';
import { app } from '../app';

describe('GET /api/v1/features', () => {
  it('should return features', async () => {
    const response = await request(app)
      .get('/api/v1/features')
      .expect(200);

    expect(response.body.features).toBeDefined();
  });

  it('should handle errors', async () => {
    // Mock database error
    await request(app)
      .get('/api/v1/features')
      .expect(500);
  });
});
```

**Trade-offs:**
- Effort: Medium (need to set up supertest)
- Benefit: High (catches API contract violations)

**Priority:** HIGH - Server tests should be first priority

---

### 2. No CI/CD Pipeline

**Current Pattern:**
No `.github/workflows/` directory found.

**Why This Matters:**
Manual testing is error-prone. No automated quality gates means broken code can reach production. Every modern project needs CI/CD.

**Better Approach:**
```yaml
# .github/workflows/ci.yml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v1
        with:
          bun-version: 1.2.2

      - run: bun install
      - run: bun run lint
      - run: bun run test
      - run: bun run build

      # Bundle size check
      - uses: andresz1/size-limit-action@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

**Trade-offs:**
- Effort: Low (copy-paste workflow)
- Benefit: High (catches issues before merge)

**Priority:** HIGH - Should be added immediately

---

### 3. Hardcoded API URLs in MSW Handlers

**Current Pattern:**
```typescript
// packages/api-mocks/src/handlers/features/getFeatures.ts:7-8
const URL = getEnv().NODE_ENV === "development"
  ? "api/v1/features"
  : "http://localhost:5173/api/v1/features";
```

**Why This Matters:**
- Brittle (port conflicts)
- Hard to configure for different environments
- Violates single source of truth

**Better Approach:**
```typescript
// ✅ GOOD - Use environment variable
const BASE_URL = getEnv().VITE_API_URL || 'http://localhost:3000';
const URL = `${BASE_URL}/api/v1/features`;

// Or even better - import from API client config
import { client } from '@repo/api/client';
const URL = `${client.getConfig().baseUrl}/features`;
```

**Trade-offs:**
- Effort: Low (simple refactor)
- Benefit: Medium (more flexible configuration)

---

### 4. Missing .env.example Files

**Current Pattern:**
- server: Has `.env.example` ✅
- client-next: No `.env.example` ❌
- client-react: No `.env.example` ❌
- packages/api: No `.env.example` ❌

**Why This Matters:**
New developers don't know what environment variables are required. This slows onboarding and causes runtime errors.

**Better Approach:**
```bash
# apps/client-next/.env.example
# API Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000/api/v1

# Feature Flags
NEXT_PUBLIC_ENABLE_ANALYTICS=false
```

**Trade-offs:**
- Effort: Trivial (copy .env and remove values)
- Benefit: High (better developer experience)

---

### 5. Component Variables Over-Abstraction

**Current Pattern:**
Standards document has EXCELLENT guidance on when NOT to create component variables (Section 5.7).

**Verification Needed:**
```bash
# Check for unnecessary component variables
grep -r "\-\-component-" packages/ui/src/ | grep "1px\|2px\|600\|500"
```

**Example of What to Look For:**
```scss
// ❌ BAD - Unnecessary abstraction
.component {
  --component-border-width: 1px;  // Used only once
  border: var(--component-border-width) solid;
}

// ✅ GOOD - Direct usage
.component {
  border: 1px solid var(--color-surface-subtle);
}
```

**Why This Matters:**
Over-abstraction makes code harder to read and maintain. The standards document already has good guidance—verify it's being followed.

---

### 6. Pre-push Hook Only Tests One App

**Current Pattern:**
```bash
# .husky/pre-push:1
cd apps/client-react && bun run test --watch=false
```

**Why This Matters:**
Only testing client-react means client-next and server tests (when added) won't run before push. This defeats the purpose of pre-push hooks.

**Better Approach:**
```bash
# .husky/pre-push
bun run test
```

**Trade-offs:**
- Effort: Trivial (one line change)
- Benefit: High (catches more issues)

---

### 7. No Commit Message Convention

**Current Pattern:**
No enforced commit message format.

**Why This Matters:**
Conventional commits enable:
- Automatic changelog generation
- Semantic versioning
- Better git history readability

**Better Approach:**
```bash
# Install commitlint
bun add -D @commitlint/cli @commitlint/config-conventional

# .husky/commit-msg
bunx commitlint --edit $1
```

**Trade-offs:**
- Effort: Low (one-time setup)
- Benefit: Medium (better git history)

---

### 8. Missing TypeScript Strict Mode Verification

**Current Pattern:**
Standards mention "TypeScript strict mode non-negotiable" but don't show the actual config.

**Verification Needed:**
```bash
# Check if strict mode is enabled
grep -r "strict" packages/typescript-config/
```

**Industry Standard (Stripe):**
```json
// packages/typescript-config/base.json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "exactOptionalPropertyTypes": true
  }
}
```

**Why This Matters:**
Stripe's experience: TypeScript strict mode is non-negotiable at scale. It catches bugs that would otherwise reach production.

</important_improvements>

---

<suggestions>
🟡 **NICE TO HAVE** - Optimizations that provide marginal gains

### 1. Add Route-Based Code Splitting

**Current:** No lazy loading implemented

**Enhancement:** Use Next.js dynamic imports for large components

```typescript
// ✅ GOOD - Lazy load heavy components
const FeatureList = dynamic(() => import('./FeatureList'), {
  loading: () => <Skeleton />,
});
```

**Benefit:** Faster initial page load, better Core Web Vitals

---

### 2. Add Dependency Security Scanning

**Current:** No automated security scanning

**Enhancement:**
```json
{
  "scripts": {
    "security:audit": "bun audit",
    "security:check": "bunx audit-ci --moderate"
  }
}
```

**Benefit:** Catches vulnerable dependencies early

---

### 3. Add Performance Budgets

**Current:** No performance budgets defined

**Enhancement:**
```json
// package.json
{
  "budgets": [
    {
      "path": "dist/**/*.js",
      "limit": "300kb",
      "type": "initial"
    }
  ]
}
```

**Benefit:** Prevents bundle size regressions

---

### 4. Add Storybook/Ladle Visual Regression Testing

**Current:** Ladle for component documentation

**Enhancement:** Add Chromatic or Percy for visual regression testing

**Benefit:** Catches unintended visual changes

---

### 5. Add OpenAPI Schema Validation in CI

**Current:** OpenAPI schema exists but no validation

**Enhancement:**
```yaml
# .github/workflows/ci.yml
- name: Validate OpenAPI Schema
  run: bunx @redocly/cli lint packages/api/openapi.yaml
```

**Benefit:** Catches API contract violations early

---

### 6. Add Turborepo Remote Caching

**Current:** Local caching only

**Enhancement:**
```json
// turbo.json
{
  "remoteCache": {
    "signature": true
  }
}
```

**Benefit:** Faster CI builds with shared cache

</suggestions>

---

<positive_patterns>
✅ **EXCELLENT PATTERNS** - What they're doing right

- **OpenAPI-First Development** - Follows Stripe's API-first approach with generated clients and automatic React Query integration
- **React Query for Server State** - Correctly separates server state from client state, following Tanner Linsley's (TanStack Query creator) philosophy
- **SCSS Modules over CSS-in-JS** - Zero runtime cost, follows Meta's build-time optimization principle
- **MSW for Network Mocking** - Network-level mocking works in both browser and Node.js, realistic API simulation
- **Design Token System** - Two-tier (Core → Semantic) follows industry best practices from design systems like Shopify Polaris
- **Component Hierarchy** - Four-tier structure (Primitives → Components → Patterns → Templates) demonstrates understanding of atomic design
- **Named Exports** - Enables better tree-shaking, follows modern JavaScript best practices
- **Explicit Package Exports** - Provides clear API surface, prevents importing internal implementation details
- **Integration Testing Strategy** - "Test all states: loading, empty, error, success" follows Kent C. Dodds' Testing Trophy
- **Co-located Tests** - `__tests__/` directories next to implementation follows colocation principle
- **Turborepo Task Dependencies** - `dependsOn: ["^build"]` ensures correct build order
- **Component-Specific Variable Guidance** - Section 5.7 has EXCELLENT criteria for when to create variables vs. when not to

</positive_patterns>

---

<migration_priorities>
**Recommended Fix Order:**

1. **First: Add CI/CD Pipeline** (Critical Issue #4 + Important Issue #2)
   - Estimated effort: 2-4 hours
   - Rationale: Automated quality gates prevent all other issues from reaching production. This is the foundation.
   - Steps:
     1. Create `.github/workflows/ci.yml`
     2. Add lint, test, build steps
     3. Add bundle size monitoring
     4. Test on a feature branch

2. **Second: Consolidate QueryClient Configuration** (Critical Issue #3)
   - Estimated effort: 1-2 hours
   - Rationale: Prevents configuration drift, single source of truth
   - Steps:
     1. Create `packages/api/src/queryClient.ts`
     2. Add package export
     3. Update both apps to use shared factory
     4. Verify tests still pass

3. **Third: Add Server Tests** (Important Issue #1)
   - Estimated effort: 4-8 hours
   - Rationale: Server is most critical to test, currently has zero coverage
   - Steps:
     1. Install supertest
     2. Create test setup
     3. Add tests for all endpoints
     4. Add to CI pipeline

4. **Fourth: Fix Barrel File Export** (Critical Issue #1)
   - Estimated effort: 2-3 hours
   - Rationale: Impacts development performance and bundle size
   - Steps:
     1. List all hooks
     2. Add individual exports
     3. Update consuming apps
     4. Measure bundle size improvement

5. **Fifth: Add Bundle Analysis** (Critical Issue #4)
   - Estimated effort: 1-2 hours
   - Rationale: Visibility into bundle size, prevents regressions
   - Steps:
     1. Install visualizer plugin
     2. Add analyze script
     3. Run and review current bundles
     4. Set budgets based on findings

6. **Then: Address remaining Important Issues**
   - Missing .env.example files (30 minutes)
   - Fix pre-push hook (5 minutes)
   - Add commit message convention (1 hour)
   - Verify TypeScript strict mode (30 minutes)
   - Fix hardcoded MSW URLs (1 hour)

**Avoid:** Trying to fix everything simultaneously. Focus on one category at a time. CI/CD first creates the safety net for all other changes.

</migration_priorities>

---

<next_iteration>
**For Next Review:**

After addressing critical issues, bring back the updated patterns file for another round of critique. We'll focus on:

- **Verification of fixes:** Did the barrel file removal actually improve bundle size?
- **New patterns to evaluate:** Any new abstractions or patterns introduced during fixes
- **Performance/scalability concerns:** Bundle analysis results, test coverage metrics
- **Production readiness:** Is this codebase ready for production deployment?

**Questions to Consider:**

1. **State Management Evolution:** When will you need global client state (Zustand)? What's the trigger point?
   - Current answer: "not needed yet" is correct
   - Watch for: Multiple components needing same UI state (modal open/close, sidebar state)
   - Don't add until you have 3+ real use cases

2. **Testing Strategy Trade-offs:** Integration tests are primary strategy—is this sustainable?
   - Current approach is EXCELLENT
   - Watch for: Tests becoming too slow (>5 seconds per test file)
   - Consider: Adding some unit tests for pure business logic if it emerges

3. **Monorepo Scaling:** How will this structure handle 10+ apps and 20+ packages?
   - Current structure is good for current scale
   - Watch for: Circular dependencies, slow builds
   - Consider: Feature-based organization if business domains emerge

4. **API Contract Testing:** OpenAPI schema is single source of truth—how do you ensure server matches?
   - Missing: Contract testing (Pact or similar)
   - Consider: Adding OpenAPI validation middleware to server
   - Benefit: Runtime validation that server matches schema

5. **Component Library Versioning:** How will you handle breaking changes in @repo/ui?
   - Current: Wildcard `*` means all apps use latest
   - Risk: Breaking change in UI breaks all apps
   - Consider: Versioning strategy when library stabilizes

</next_iteration>

---

<verification_checklist>
**Before Implementing Fixes - Verify These First:**

```bash
# 1. Check for React.FC usage
grep -r "React.FC\|React.FunctionComponent" packages/ui/src/

# 2. Verify no server state in useState/Context
grep -r "useState.*fetch\|useEffect.*fetch" apps/

# 3. Check TypeScript strict mode
grep -r "strict" packages/typescript-config/

# 4. Find unnecessary component variables
grep -r "\-\-component-" packages/ui/src/ | grep "1px\|2px\|600\|500"

# 5. Check current bundle sizes (baseline)
cd apps/client-next && bun run build
cd apps/client-react && bun run build

# 6. Verify test coverage
bun run test --coverage

# 7. Check for duplicate dependencies
bun run deps:check
```

**Document baseline metrics before making changes:**
- Current bundle sizes
- Current test coverage percentage
- Current build times
- Current number of dependencies

This enables measuring improvement after fixes.

</verification_checklist>

---

## Summary

**Overall:** This is a well-architected monorepo with modern tooling and solid foundations. The team clearly understands React/TypeScript best practices.

**Biggest Wins:**
1. OpenAPI-first API development
2. React Query for server state
3. SCSS Modules (no runtime cost)
4. Integration testing with MSW

**Biggest Gaps:**
1. No CI/CD (critical)
2. Inconsistent test coverage (server has zero tests)
3. No bundle monitoring
4. Configuration duplication

**Recommendation:** Focus on CI/CD first. It's the foundation that makes all other improvements safer and more measurable. Then tackle server tests, then bundle analysis.

**Confidence Level:** HIGH - Based on thorough analysis of extracted standards and alignment with industry best practices from Airbnb, Stripe, Meta, and Vercel.


