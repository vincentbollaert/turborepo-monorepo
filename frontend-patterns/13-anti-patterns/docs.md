# 13. Anti-Patterns

> **Quick Guide:** Common mistakes compiled from all frontend patterns. Review these to avoid pitfalls.

This document compiles all anti-patterns identified across the frontend standards documentation.

## 13.1 State Management Anti-Patterns

**❌ NEVER:**
- Store server data in Zustand (use React Query instead)
- Store client UI state in React Query (use Zustand instead)
- Create unnecessary object references in Zustand selectors (causes re-renders)
- Use prop drilling instead of state management for global state
- Mutate state directly (use immutable updates)
- Create separate stores for related data (use one store with slices)

> See [State Management §4](../04-state-management/docs.md)

## 13.2 TypeScript Anti-Patterns

**❌ NEVER:**
- Use `any` without explicit justification comment
- Use `@ts-ignore` or `@ts-expect-error` without explaining why
- Skip type definitions for exported functions
- Use implicit `any` (enable `strict` mode)
- Define types far from their usage
- Use `I` prefix for interfaces (e.g., `IProduct`)
- Use `interface` for component props (use `type` instead)

> See [Code Conventions §3.4 & §3.6](../03-code-conventions/docs.md)

## 13.3 Component Anti-Patterns

**❌ NEVER:**
- Create God components (> 300 lines, > 10 props)
- Skip ref forwarding on interactive elements
- Skip className exposure (prevents customization)
- Use prop spreading without type safety
- Use inline styles instead of design tokens
- Use cva for components with no variants (over-engineering)
- Use default exports in library components (use named exports)
- Mix casing (Button.tsx vs button.module.scss)

> See [Code Conventions §3.1 & §3.2](../03-code-conventions/docs.md)

## 13.4 Performance Anti-Patterns

**❌ NEVER:**
- Memoize everything (premature optimization has overhead)
- Skip memoization for expensive operations
- Use inline function definitions in JSX props (causes re-renders)
- Import entire libraries (`import _ from 'lodash'` instead of `import debounce from 'lodash/debounce'`)
- Skip lazy loading for routes
- Skip code splitting
- Not optimize images (missing WebP, lazy loading, responsive sizes)
- Block main thread with heavy computation (use web workers)
- Not measure performance before optimizing
- Set bundle size budgets (sizes grow unnoticed)

> See [Performance Standards §11](../11-performance-standards/docs.md)

## 13.5 Testing Anti-Patterns

**❌ NEVER:**
- Test implementation details (test behavior, not implementation)
- Use brittle selectors (prefer `getByRole`, `getByLabelText`)
- Skip MSW setup for API tests
- Skip integration tests (only unit tests)
- Have incomplete test coverage (< 80%)
- Test private methods directly
- Mock too much (integration tests should use real dependencies)
- Skip accessibility testing

> See [Testing Standards §5](../05-testing-standards/docs.md)

## 13.6 API & Data Fetching Anti-Patterns

**❌ NEVER:**
- Hardcode API URLs (use environment variables)
- Skip error handling for API calls
- Skip loading states
- Skip retry logic for transient failures
- Mutate cache directly (use React Query mutation helpers)
- Fetch data on every render (use caching)
- Not handle race conditions
- Skip request deduplication

> See [API Client Architecture §2](../02-api-client-architecture/docs.md)

## 13.7 Styling Anti-Patterns

**❌ NEVER:**
- Use CSS-in-JS (styled-components, Emotion) - use SCSS Modules
- Use inline styles for anything other than dynamic values
- Hardcode colors/spacing (use design tokens)
- Create component variables unnecessarily
- Use className toggling for state (use data-attributes)
- Skip Ladle stories for design system components
- Use Tailwind classes directly in components (use design tokens)

> See [Design System §6](../06-design-system/docs.md)

## 13.8 Accessibility Anti-Patterns

**❌ NEVER:**
- Remove focus outlines without replacement
- Use `div` or `span` for buttons/links
- Add click handlers on non-interactive elements without role/keyboard support
- Use color-only error indicators
- Use placeholder text as label replacement
- Disable form submit buttons (show validation errors instead)
- Auto-play audio/video without controls
- Open new windows without warning
- Create form inputs without labels
- Skip keyboard navigation support

> See [Accessibility §7](../07-accessibility/docs.md)

## 13.9 Build & Tooling Anti-Patterns

**❌ NEVER:**
- Skip linting configuration
- Use multiple icon libraries (use lucide-react)
- Import entire lucide-react package
- Use magic numbers anywhere (use named constants)
- Skip TypeScript strict mode
- Modify generated files manually (regenerate from source)
- Skip pre-commit hooks
- Not use Turborepo caching (wastes build time)

> See [Build & Tooling §8](../08-build-tooling/docs.md)

## 13.10 Environment & Security Anti-Patterns

**❌ NEVER:**
- Commit secrets to repository
- Use `process.env.VARIABLE` directly without validation
- Hardcode environment values in code
- Share `.env` files via Slack/email
- Use production secrets in development
- Skip `.env.example` documentation
- Skip validation (missing variables cause runtime errors)
- Use different variable names across environments
- Not rotate secrets
- Expose secrets in client-side code (must use framework prefixes)
- Use `dangerouslySetInnerHTML` with user input

> See [Environment Management §10](../10-environment-management/docs.md) & [Security §12](../12-security-patterns/docs.md)

## 13.11 File & Directory Anti-Patterns

**❌ NEVER:**
- Use PascalCase for file names (use kebab-case)
- Mix casing (Button.tsx and button.module.scss)
- Use default exports in libraries
- Create barrel files in components (use package.json exports)
- Import from internal paths instead of package exports
- Use relative imports for cross-package imports

> See [Code Conventions §3.2 & §3.3](../03-code-conventions/docs.md)

## 13.12 Monorepo Anti-Patterns

**❌ NEVER:**
- Have version mismatches across packages (use syncpack)
- Skip dependency synchronization
- Not declare environment variables in turbo.json
- Modify turbo cache manually
- Run full test suite on every PR (use affected detection)
- Not use remote caching (wastes CI time)
- Not use affected builds

> See [Build & Tooling §8](../08-build-tooling/docs.md)

## 13.13 Quick Anti-Pattern Checklist

Review this checklist before submitting code:

**Code Quality:**
- [ ] No `any` without justification
- [ ] No magic numbers
- [ ] No hardcoded values
- [ ] Named exports only
- [ ] kebab-case file names

**Components:**
- [ ] Ref forwarding on interactive elements
- [ ] className prop exposed
- [ ] No God components (< 300 lines)
- [ ] Data-attributes for state styling
- [ ] Design tokens (no hardcoded colors/spacing)

**State & Data:**
- [ ] Server data in React Query
- [ ] UI state in Zustand
- [ ] Separate selectors (or `shallow` when destructuring)
- [ ] No prop drilling for global state

**Performance:**
- [ ] Lazy load routes
- [ ] No unnecessary memoization
- [ ] Optimized images
- [ ] No large libraries imported whole

**Testing:**
- [ ] MSW for API mocking
- [ ] Testing Library queries (getByRole)
- [ ] Integration tests
- [ ] Accessibility tests

**Security:**
- [ ] No committed secrets
- [ ] Validated environment variables
- [ ] No exposed API keys
- [ ] Input sanitization

**Accessibility:**
- [ ] Semantic HTML
- [ ] Keyboard navigation
- [ ] ARIA labels
- [ ] Focus management
- [ ] Screen reader support
