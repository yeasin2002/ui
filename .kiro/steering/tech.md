# Tech Stack

## Build System

- **Monorepo**: pnpm workspaces + Turborepo
- **Package Manager**: pnpm (v9.0.6)
- **Releases**: Changesets for versioning and changelogs

## Main Packages

| Package           | Tech                                              |
| ----------------- | ------------------------------------------------- |
| `apps/v4`         | Next.js 16, React 19, Tailwind CSS v4, TypeScript |
| `packages/shadcn` | Node.js CLI, Commander, ts-morph, Zod             |

## Core Dependencies

- **UI Primitives**: Radix UI, Base UI
- **Styling**: Tailwind CSS v4, class-variance-authority (cva), tailwind-merge, clsx
- **Forms**: react-hook-form, @hookform/resolvers, zod
- **Documentation**: Fumadocs (MDX-based docs framework)
- **Charts**: Recharts
- **Icons**: Lucide, Hugeicons, Tabler
- **Animation**: Motion (Framer Motion)

## Common Commands

```bash
# Install dependencies
pnpm install

# Development
pnpm dev                    # Run all workspaces in dev mode
pnpm v4:dev                 # Run docs website (port 4000)
pnpm shadcn:dev             # Run CLI in dev mode

# Building
pnpm build                  # Build all packages
pnpm v4:build               # Build docs website
pnpm shadcn:build           # Build CLI

# Registry
pnpm registry:build         # Rebuild component registry
pnpm registry:capture       # Capture registry screenshots

# Testing
pnpm test                   # Run all tests
pnpm shadcn:test            # Run CLI tests

# Code Quality
pnpm lint                   # Lint all packages
pnpm lint:fix               # Fix lint issues
pnpm format:write           # Format code with Prettier
pnpm typecheck              # TypeScript type checking

# CLI Testing (local)
pnpm shadcn                 # Run CLI against local registry
pnpm shadcn:prod            # Run CLI against production registry
```

## Code Style

- TypeScript with strict mode
- Prettier for formatting (no semicolons, double quotes, 2-space indent)
- ESLint with Next.js and TypeScript rules
- Consistent type imports: `import { type Foo }` (inline style)
- Import order: react → next → third-party → workspace → types → local
