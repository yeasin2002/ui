# Technology Stack

## Build System

- **Package Manager**: pnpm (v9.0.6) with workspaces
- **Monorepo Tool**: Turborepo for build orchestration
- **Bundler**: tsup for package builds, Next.js for the web app

## Core Technologies

### Frontend

- **Framework**: Next.js 16 (App Router) with React 19
- **Styling**: Tailwind CSS v4 with PostCSS
- **UI Primitives**: Radix UI and Base UI (@base-ui/react)
- **Icons**: Multiple libraries supported (Lucide, Hugeicons, Tabler, Phosphor, Radix Icons)
- **Animations**: Motion (Framer Motion successor)
- **Forms**: React Hook Form with Zod validation
- **State**: Jotai for state management

### Documentation

- **Content**: Fumadocs (MDX-based documentation framework)
- **Syntax Highlighting**: Shiki with rehype-pretty-code
- **Code Examples**: Component preview system with live demos

### CLI Package

- **Runtime**: Node.js with TypeScript
- **CLI Framework**: Commander.js
- **Code Transformation**: ts-morph, Babel, Recast
- **MCP Support**: Model Context Protocol SDK for AI integration

## Code Quality

- **TypeScript**: Strict mode enabled across all packages
- **Linting**: ESLint with Next.js, Turbo, and Tailwind plugins
- **Formatting**: Prettier with import sorting
- **Testing**: Vitest for unit and integration tests
- **Changesets**: For version management and changelogs

## Common Commands

### Development

```bash
# Run the documentation site
pnpm --filter=v4 dev
# or
pnpm v4:dev

# Run the CLI in dev mode
pnpm --filter=shadcn dev
# or
pnpm shadcn:dev

# Run all workspaces in parallel
pnpm dev
```

### Building

```bash
# Build all packages
pnpm build

# Build specific workspace
pnpm --filter=shadcn build
pnpm --filter=v4 build

# Build registry (generates component metadata)
pnpm registry:build
```

### Testing

```bash
# Run all tests
pnpm test

# Run tests for shadcn package
pnpm shadcn:test

# Run tests in dev mode (requires v4 dev server running)
pnpm test:dev
```

### Code Quality

```bash
# Lint all packages
pnpm lint

# Fix linting issues
pnpm lint:fix

# Type check
pnpm typecheck

# Format code
pnpm format:write

# Check formatting
pnpm format:check

# Run all checks
pnpm check
```

### CLI Testing

```bash
# Test CLI locally (points to local registry)
pnpm shadcn

# Test CLI with specific app
pnpm shadcn init -c ~/path/to/app
pnpm shadcn add button -c ~/path/to/app
```

### Registry Management

```bash
# Build registry files
pnpm registry:build

# Capture registry screenshots
pnpm registry:capture

# Validate registries
pnpm validate:registries
```

## Environment Variables

The v4 app uses several environment variables (see `apps/v4/.env.example`):

- `NEXT_PUBLIC_APP_URL`: Public URL for the app
- `COMPONENTS_REGISTRY_URL` / `REGISTRY_URL`: Registry endpoint URLs
- `V0_URL` / `V0_EDIT_SECRET`: Integration with v0.dev
- `UPSTASH_REDIS_REST_URL` / `UPSTASH_REDIS_REST_TOKEN`: Redis for caching
