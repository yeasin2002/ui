# Project Structure

## Monorepo Layout

```
ui/
├── apps/
│   └── v4/                    # Documentation website (Next.js)
├── packages/
│   ├── shadcn/                # CLI package
│   └── tests/                 # Integration tests
├── templates/                 # Project templates
├── deprecated/                # Legacy code (www, cli)
└── scripts/                   # Build and utility scripts
```

## Apps

### v4 (Documentation Site)

```
apps/v4/
├── app/                       # Next.js App Router pages
│   ├── (app)/                 # Main app routes (docs, components, etc.)
│   ├── (create)/              # Component creation flows
│   ├── (examples)/            # Example pages
│   ├── (internal)/            # Internal tools
│   ├── (view)/                # Component preview routes
│   └── api/                   # API routes
├── components/                # Site-specific React components
├── content/                   # MDX documentation files
│   └── docs/                  # Documentation content
├── hooks/                     # Custom React hooks
├── lib/                       # Utility functions and helpers
├── registry/                  # Component registry (THE CORE)
│   ├── bases/                 # Base UI implementations
│   ├── new-york-v4/           # Radix UI implementations
│   ├── radix-*/               # Style-specific Radix components
│   ├── base-*/                # Style-specific Base components
│   ├── styles/                # Style definitions
│   ├── config.ts              # Registry configuration
│   └── __index__.tsx          # Generated registry index
├── public/                    # Static assets
│   └── r/                     # Registry JSON files (served publicly)
├── scripts/                   # Build scripts
│   ├── build-registry.mts     # Generates registry files
│   ├── capture-registry.mts   # Screenshots components
│   └── validate-registries.mts
└── styles/                    # Global CSS
```

## Packages

### shadcn (CLI)

```
packages/shadcn/
├── src/
│   ├── commands/              # CLI commands (init, add, etc.)
│   ├── mcp/                   # Model Context Protocol integration
│   ├── migrations/            # Version migration scripts
│   ├── preflights/            # Pre-flight checks
│   ├── registry/              # Registry client code
│   ├── schema/                # Zod schemas
│   ├── styles/                # Style definitions
│   └── utils/                 # Utility functions
└── test/                      # Unit tests
```

### tests

```
packages/tests/
├── fixtures/                  # Test fixtures (sample projects)
│   ├── next-app/
│   ├── vite-app/
│   └── registry/
└── src/
    └── tests/                 # Integration test suites
```

## Registry System

The registry is the heart of the project. It organizes components by:

1. **Base**: UI primitive library (Radix UI or Base UI)
2. **Style**: Visual design system (Vega, Nova, Maia, Lyra, Mira)
3. **Type**: Component category (ui, example, block, hook, lib, theme)

### Registry Structure

```
registry/
├── bases/                     # Base UI components
│   └── base/
│       ├── ui/                # UI components
│       └── examples/          # Usage examples
├── new-york-v4/               # Radix UI components (legacy naming)
│   ├── ui/
│   ├── examples/
│   └── blocks/
├── radix-{style}/             # Style-specific Radix components
│   ├── ui/
│   └── examples/
├── base-{style}/              # Style-specific Base components
│   ├── ui/
│   └── examples/
├── config.ts                  # Configuration and schemas
├── styles.tsx                 # Style definitions
├── themes.ts                  # Theme definitions
├── base-colors.ts             # Base color palettes
└── __index__.tsx              # Auto-generated registry index
```

## Key Conventions

### File Organization

- Components are organized by base and style
- Each component has a main implementation in `ui/` and examples in `examples/`
- Blocks are larger, composed components in `blocks/`

### Naming

- Registry items use kebab-case: `button.tsx`, `data-table.tsx`
- React components use PascalCase: `Button`, `DataTable`
- Style names: vega, nova, maia, lyra, mira
- Base names: radix, base

### Import Paths

The v4 app uses path aliases:

- `@/components/*` - Site components
- `@/registry/*` - Registry components
- `@/lib/*` - Utilities
- `@/hooks/*` - Custom hooks
- `@/config/*` - Configuration
- `@/styles/*` - Styles

### Registry Types

- `registry:ui` - UI components
- `registry:example` - Usage examples
- `registry:block` - Composed blocks
- `registry:hook` - Custom hooks
- `registry:lib` - Utility libraries
- `registry:theme` - Theme definitions
- `registry:base` - Base configurations
