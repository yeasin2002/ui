# Technology Stack

## Build System

- **tsup**: Build tool for TypeScript libraries with ESM output
- **TypeScript**: Strict typing with ES modules
- **Vitest**: Testing framework with 8s timeout for integration tests

## Core Dependencies

- **commander**: CLI framework for command parsing
- **prompts**: Interactive CLI prompts
- **zod**: Runtime schema validation
- **ts-morph**: TypeScript AST manipulation
- **@babel/parser**: JavaScript/TypeScript parsing
- **recast**: Code transformation with formatting preservation
- **cosmiconfig**: Configuration file discovery

## UI/Styling

- **Tailwind CSS**: Utility-first CSS framework (v3 and v4 support)
- **postcss**: CSS processing and transformation
- **Radix UI**: Headless UI component primitives

## Utilities

- **fs-extra**: Enhanced file system operations
- **fast-glob**: File pattern matching
- **execa**: Process execution
- **ora**: Terminal spinners
- **kleur**: Terminal colors
- **diff**: Text diffing
- **fuzzysort**: Fuzzy search

## Common Commands

```bash
# Development
pnpm dev              # Watch mode build
pnpm build            # Production build
pnpm typecheck        # Type checking without emit

# Testing
pnpm test             # Run all tests
pnpm test:dev         # Run tests with local registry

# Formatting
pnpm format:write     # Format code with Prettier
pnpm format:check     # Check formatting

# CLI Usage
pnpm start            # Run CLI with production registry
pnpm start:dev        # Run CLI with local registry
pnpm start:prod       # Run CLI with production registry

# MCP
pnpm mcp:inspect      # Inspect MCP server with debugger
```

## Module System

- ES modules only (type: "module")
- Path aliases using `@/*` for imports
- Multiple entry points exported: main, registry, schema, mcp, utils, icons
