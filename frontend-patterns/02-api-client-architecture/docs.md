# 02. API Client Architecture

> **Quick Guide:** Using OpenAPI? See §2.1. Need type safety? See §2.3. Integrating with React Query? See §2.5. Setting up MSW mocks? See §2.6.

## 2.1 OpenAPI/Swagger Integration

- Schema-first development approach
- **Using hey-api (@hey-api/openapi-ts)** with plugins:
  - `@hey-api/client-fetch` for fetch-based client
  - `@tanstack/react-query` for automatic React Query hooks
- Type generation from OpenAPI specs
- Generated query options and query keys
- Keeping schemas in sync with backend
- Manual regeneration when schema changes (`bun run build` in packages/api)

## 2.2 Client Configuration

- Base URL configuration via `client.setConfig()` (environment-specific)
- Configure in app providers before React Query setup
- No interceptors (fetch-based client)
- Authentication handling via fetch headers (if needed)
- Timeout configuration at fetch level
- Retry logic handled by React Query (not at client level)
- Global defaults set in QueryClient configuration

## 2.3 Type Safety

- **Generated types from OpenAPI** (never manual)
- All types auto-generated from schema (types, requests, responses)
- Type inference from generated query options
- Full autocomplete in components
- Breaking change detection via TypeScript errors
- Optional: Runtime validation with Zod schemas (if generated from OpenAPI)

## 2.4 Error Handling

- Component-level error handling (isPending, error, isSuccess states)
- React Query's built-in retry logic (exponential backoff)
- Global error defaults in QueryClient configuration
- Per-query error handling and retry customization
- Network error handling via React Query
- No interceptors (fetch-based client doesn't support them)
- Display errors where they occur (better UX)

## 2.5 Integration with React Query

- **Generated query options** from `@tanstack/react-query` plugin
- Use `getFeaturesOptions()` instead of writing custom hooks
- Generated query keys automatically namespaced
- Type inference works automatically
- Customize generated options by spreading them
- No manual query key factories needed
- Mutations follow same generated pattern

## 2.6 MSW for API Mocking

- Centralized mock package (@repo/api-mocks)
- Separation of handlers and mock data
- Variant switching (default, empty, error scenarios)
- Shared mocks between browser (dev) and Node (tests)
- Type-safe mocks using generated types
- Per-test handler overrides

**RED FLAGS:**

- ❌ Manual API type definitions (should be generated from OpenAPI)
- ❌ Manual React Query hooks (should use generated query options)
- ❌ Hardcoded API URLs (should use environment variables)
- ❌ Not using generated types from `@repo/api/types`
- ❌ Writing custom query key factories (should use generated keys)
- ❌ Inconsistent error handling patterns
- ❌ Not separating handlers and mock data in MSW setup
- ❌ Trying to use interceptors with fetch-based client
