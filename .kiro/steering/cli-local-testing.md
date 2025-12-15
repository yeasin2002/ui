# Running and Testing the CLI Locally

## Quick Start

```bash
# 1. Install dependencies
pnpm install

# 2. Build the CLI
pnpm shadcn:build

# 3. Run CLI commands locally
pnpm shadcn <command> [options]
```

## Development Workflow

### Option 1: Dev Mode (Recommended for Development)

Run the CLI in watch mode - automatically rebuilds on changes:

```bash
# Terminal 1: Start CLI in dev mode
pnpm shadcn:dev

# Terminal 2: Test CLI commands
pnpm shadcn init -c ~/Desktop/test-app
pnpm shadcn add button -c ~/Desktop/test-app
```

### Option 2: Build and Run

For testing production-like behavior:

```bash
# Build once
pnpm shadcn:build

# Run commands
pnpm shadcn init
pnpm shadcn add button card
pnpm shadcn create my-app --template next
```

## Testing Against Different Registries

### Local Registry (Development)

Uses the local dev server at `http://localhost:4000`:

```bash
# Terminal 1: Start the docs site (serves local registry)
pnpm v4:dev

# Terminal 2: Run CLI against local registry
pnpm shadcn add button -c ~/Desktop/test-app
```

### Production Registry

Uses `https://ui.shadcn.com`:

```bash
pnpm shadcn:prod add button -c ~/Desktop/test-app
```

## Common Test Scenarios

### Test `init` Command

```bash
# Create a test directory
mkdir ~/Desktop/test-init && cd ~/Desktop/test-init

# Initialize a new Next.js project first
npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"

# Run shadcn init
pnpm shadcn init -c ~/Desktop/test-init
```

### Test `add` Command

```bash
pnpm shadcn add button -c ~/Desktop/test-init
pnpm shadcn add card dialog -c ~/Desktop/test-init
pnpm shadcn add "https://ui.shadcn.com/r/button" -c ~/Desktop/test-init
```

### Test `create` Command

```bash
# Basic create
pnpm shadcn create my-app --template next

# With preset URL
pnpm shadcn create my-app --preset "https://ui.shadcn.com/init?base=base&style=mira&baseColor=neutral&theme=pink&iconLibrary=lucide&font=inter&menuAccent=subtle&menuColor=default&radius=default" --template next

# With src directory
pnpm shadcn create my-app --template next --src-dir
```

### Test `diff` Command

```bash
pnpm shadcn diff button -c ~/Desktop/test-init
```

## Running Tests

```bash
# Run all CLI tests
pnpm shadcn:test

# Run specific test file
pnpm --filter=shadcn test src/commands/init.test.ts

# Run tests in watch mode
pnpm --filter=shadcn test --watch
```

## Debugging

### Enable Verbose Output

Most commands support `--silent=false` or verbose flags. Check command help:

```bash
pnpm shadcn init --help
pnpm shadcn add --help
```

### Debug with Node Inspector

```bash
# Add --inspect flag
node --inspect ./packages/shadcn/dist/index.js init -c ~/Desktop/test-app
```

### Check Built Output

The CLI builds to `packages/shadcn/dist/`:

```bash
# View built files
ls packages/shadcn/dist/
```

## Project Structure

```
packages/shadcn/
├── src/
│   ├── commands/       # CLI commands (init, add, create, diff, etc.)
│   ├── registry/       # Registry fetching and parsing
│   ├── utils/          # Utilities and helpers
│   │   ├── updaters/   # File update logic
│   │   └── transformers/ # Code transformers
│   └── index.ts        # CLI entry point
├── test/               # Test files and fixtures
│   └── fixtures/       # Test project fixtures
└── dist/               # Built output
```

## Troubleshooting

### "Command not found" Error

Make sure you've built the CLI:

```bash
pnpm shadcn:build
```

### Registry Connection Issues

Check if the registry is accessible:

```bash
# For local registry
curl http://localhost:4000/registry/index.json

# For production
curl https://ui.shadcn.com/registry/index.json
```

### Clean Rebuild

```bash
# Clean and rebuild
pnpm clean
pnpm install
pnpm shadcn:build
```
