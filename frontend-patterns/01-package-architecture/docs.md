# 01. Package Architecture

> **Quick Guide:** Setting up a monorepo? See §1.1. Need package naming standards? See §1.2. Enforcing boundaries? See §1.3. Organizing UI components? See §1.6.

## 1.1 Workspace Structure

- Workspace organization (packages/, apps/, tools/)
- Directory naming conventions
- Monorepo tool identification (Turborepo/Nx/Lerna)

## 1.2 Package Naming Conventions

- Internal package prefixes (@repo/, @app/, etc.)
- Naming patterns (kebab-case, PascalCase)
- Scoping strategies

## 1.3 Dependency Boundaries

- Package dependency rules
- ESLint enforcement
- Circular dependency prevention
- Type tags (type:ui, type:util, type:data-access, type:feature)

## 1.4 Import Conventions

- Package names vs path aliases
- Import path standards
- Explicit package.json exports (preferred over barrel exports)
- Tree-shaking and API surface control

## 1.5 Versioning Strategy

- Changesets, Lerna, or manual versioning
- Version bumping workflow
- Changelog generation

## 1.6 UI Library Organization

- Component hierarchy (Primitives → Components → Patterns → Templates)
- Directory structure within UI packages
- When to use each tier (decision tree)
- Composition patterns across tiers
- Primitives and Components (essential foundation)
- Patterns and Templates (recommended for scale)
