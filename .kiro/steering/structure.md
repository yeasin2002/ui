# Project Structure

```
├── apps/
│   └── v4/                      # Main documentation website (ui.shadcn.com)
│       ├── app/                 # Next.js App Router pages
│       │   ├── (app)/           # Main site routes (docs, blocks, charts, etc.)
│       │   ├── (create)/        # Component creation flow
│       │   ├── (examples)/      # Example dashboards
│       │   └── api/             # API routes
│       ├── components/          # Website-specific components
│       ├── content/docs/        # MDX documentation files
│       ├── hooks/               # Website hooks
│       ├── lib/                 # Utilities and helpers
│       ├── registry/            # Component registry source
│       │   └── new-york-v4/     # Main component style
│       │       ├── ui/          # UI components (button, card, etc.)
│       │       ├── blocks/      # Full-page block components
│       │       ├── charts/      # Chart components
│       │       ├── examples/    # Component examples
│       │       ├── hooks/       # Reusable hooks
│       │       └── lib/         # Utility functions
│       └── styles/              # Global CSS
│
├── packages/
│   ├── shadcn/                  # CLI package (npx shadcn)
│   │   ├── src/
│   │   │   ├── commands/        # CLI commands (init, add, diff, etc.)
│   │   │   ├── registry/        # Registry fetching/parsing
│   │   │   ├── utils/           # CLI utilities
│   │   │   └── mcp/             # Model Context Protocol server
│   │   └── test/                # CLI tests
│   └── tests/                   # Integration tests
│
├── templates/                   # Starter templates
│   ├── monorepo-next/           # Turborepo + Next.js template
│   ├── start-app/               # TanStack Start template
│   └── vite-app/                # Vite + React template
│
└── deprecated/                  # Legacy code (v3 website, old CLI)
```

## Key Conventions

### Component Files

- UI components live in `apps/v4/registry/new-york-v4/ui/`
- Each component is a single `.tsx` file
- Components use `data-slot` attributes for styling hooks
- Export both component and variants (e.g., `Button`, `buttonVariants`)

### Registry System

- `_registry.ts` files define component metadata (dependencies, files)
- `registry.ts` aggregates all registry items
- Run `pnpm registry:build` after modifying components

### Path Aliases

- `@/*` maps to `apps/v4/*` in the v4 app
- Components import utils via `@/lib/utils`
- UI components via `@/registry/new-york-v4/ui/*`
