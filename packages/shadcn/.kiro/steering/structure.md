# Project Structure

## Top-Level Organization

```
src/
├── commands/          # CLI command implementations
├── registry/          # Registry API, fetching, parsing, validation
├── mcp/              # Model Context Protocol integration
├── utils/            # Shared utilities and helpers
├── styles/           # Style transformation utilities
├── migrations/       # Component migration logic
├── preflights/       # Pre-execution validation checks
├── schema/           # Zod schemas and type definitions
├── icons/            # Icon library definitions
└── index.ts          # CLI entry point

test/
├── fixtures/         # Test fixtures for various frameworks
└── utils/            # Utility tests
```

## Key Directories

### commands/
CLI command definitions using Commander.js. Each command exports a Command instance with options, arguments, and action handlers.

- Main commands: init, create, add, diff, view, search, migrate, info, build, mcp
- Nested commands in `registry/` subdirectory for registry-specific operations

### registry/
Core registry functionality for fetching and managing components:

- **api.ts**: Registry API client for fetching items, styles, colors
- **fetcher.ts**: HTTP fetching with proxy support and authentication
- **parser.ts**: Parse registry responses and local files
- **resolver.ts**: Resolve component dependencies and references
- **validator.ts**: Validate registry items against schemas
- **builder.ts**: Build URLs and headers for registry requests
- **search.ts**: Search across multiple registries
- **namespaces.ts**: Handle registry namespacing (e.g., `@registry/component`)
- **config.ts**: Registry configuration management
- **errors.ts**: Registry-specific error types

### utils/
Shared utilities organized by function:

- **transformers/**: Code transformations (imports, JSX, icons, CSS vars, etc.)
- **updaters/**: File updaters (dependencies, CSS, Tailwind config, fonts, env vars)
- **get-*.ts**: Information getters (config, project info, package manager)
- **logger.ts, spinner.ts, highlighter.ts**: CLI output utilities
- **errors.ts**: Error code constants
- **file-helper.ts**: File backup/restore operations

### preflights/
Pre-execution validation for each command to check prerequisites and project state.

### migrations/
Component migration logic for updating between versions or styles.

## Conventions

- Use `@/` path alias for all internal imports
- Export schemas alongside implementation files
- Separate concerns: fetching, parsing, validation, transformation
- Commands handle user interaction; utilities handle business logic
- Use Zod for all runtime validation and type inference
- Prefer `fs-extra` over native `fs` for file operations
- Use `execa` for spawning processes
- Spinner feedback for long-running operations
- Backup files with `.bak` suffix before destructive operations
