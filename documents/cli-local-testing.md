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
pnpm shadcn init -c C:\Yeasin\experiment\resizable-issues-check
pnpm shadcn add resizable -c C:\Yeasin\experiment\resizable-issues-check
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
pnpm shadcn add button -c C:\Yeasin\experiment\resizable-issues-check
```

### Production Registry

Uses `https://ui.shadcn.com`:

```bash
pnpm shadcn:prod add button -c C:\Yeasin\experiment\resizable-issues-check
```

---

## Testing the `--src-dir` ENOENT Fix

After implementing the fix for the `--src-dir` ENOENT issue, run these tests to verify everything works.

### Prerequisites

```bash
# 1. Build the CLI with your changes
pnpm shadcn:build

# 2. Create a clean test directory
mkdir C:\Yeasin\experiment\fix-test
cd C:\Yeasin\experiment\fix-test
```

### Test Matrix

Run ALL of these tests. Each should complete without errors.

#### ✅ Test 1: Next.js with `--src-dir` (Previously FAILED)

```bash
# Clean directory first
rmdir /s /q C:\Yeasin\experiment\fix-test 2>nul
mkdir C:\Yeasin\experiment\fix-test

# Run from repo root
pnpm shadcn init --src-dir -c "C:/Yeasin/experiment/fix-test"
# Select: Next.js
# Name: test-next-src
# Color: Neutral

# Expected: Success! Project initialization completed.
```

#### ✅ Test 2: Vite with `--src-dir` (Previously FAILED)

```bash
rmdir /s /q C:\Yeasin\experiment\fix-test 2>nul
mkdir C:\Yeasin\experiment\fix-test

pnpm shadcn init --src-dir -c "C:/Yeasin/experiment/fix-test"
# Select: Vite
# Name: test-vite-src
# Color: Neutral

# Expected: Success! Project initialization completed.
```

#### ✅ Test 3: TanStack Start with `--src-dir` (Previously FAILED)

```bash
rmdir /s /q C:\Yeasin\experiment\fix-test 2>nul
mkdir C:\Yeasin\experiment\fix-test

pnpm shadcn init --src-dir -c "C:/Yeasin/experiment/fix-test"
# Select: TanStack Start
# Name: test-start-src
# Color: Neutral

# Expected: Success! Project initialization completed.
```

#### ✅ Test 4: Next.js WITHOUT `--src-dir` (Should still work)

```bash
rmdir /s /q C:\Yeasin\experiment\fix-test 2>nul
mkdir C:\Yeasin\experiment\fix-test

pnpm shadcn init -c "C:/Yeasin/experiment/fix-test"
# Select: Next.js
# Name: test-next-no-src
# Color: Neutral

# Expected: Success! Project initialization completed.
# Files should be in: lib/utils.ts, app/globals.css (no src folder)
```

#### ✅ Test 5: `create` command with `--src-dir`

```bash
rmdir /s /q C:\Yeasin\experiment\fix-test 2>nul
mkdir C:\Yeasin\experiment\fix-test

pnpm shadcn create test-create --template next --src-dir -c "C:/Yeasin/experiment/fix-test"

# Expected: Success! Project initialization completed.
```

#### ✅ Test 6: `create` with preset URL and `--src-dir`

```bash
rmdir /s /q C:\Yeasin\experiment\fix-test 2>nul
mkdir C:\Yeasin\experiment\fix-test

pnpm shadcn create test-preset --preset "https://ui.shadcn.com/init?base=base&style=mira&baseColor=neutral&theme=pink&iconLibrary=lucide&font=inter&menuAccent=subtle&menuColor=default&radius=default" --template next --src-dir -c "C:/Yeasin/experiment/fix-test"

# Expected: Success! Project initialization completed.
```

#### ✅ Test 7: Existing project (Regression test)

```bash
# First create a Next.js project manually
cd C:\Yeasin\experiment
npx create-next-app@latest existing-app --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"

cd existing-app

# Then run shadcn init
pnpm shadcn init -c "C:/Yeasin/experiment/existing-app"

# Expected: Success! Should detect Next.js framework automatically.
```

### Quick Test Script (PowerShell)

Save this as `test-fix.ps1` and run from repo root:

```powershell
$testDir = "C:\Yeasin\experiment\fix-test"

function Test-Command {
    param($Name, $Command)
    Write-Host "`n=== Testing: $Name ===" -ForegroundColor Cyan
    
    # Clean up
    if (Test-Path $testDir) { Remove-Item -Recurse -Force $testDir }
    New-Item -ItemType Directory -Path $testDir | Out-Null
    
    # Run command
    Invoke-Expression $Command
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✅ PASSED: $Name" -ForegroundColor Green
    } else {
        Write-Host "❌ FAILED: $Name" -ForegroundColor Red
    }
}

# Build first
Write-Host "Building CLI..." -ForegroundColor Yellow
pnpm shadcn:build

# Run tests (interactive - you'll need to select options)
Write-Host "`n`nRun these commands manually and verify each succeeds:`n" -ForegroundColor Yellow

Write-Host "1. Next.js + --src-dir:" -ForegroundColor Cyan
Write-Host "   pnpm shadcn init --src-dir -c `"C:/Yeasin/experiment/fix-test`""

Write-Host "`n2. Vite + --src-dir:" -ForegroundColor Cyan  
Write-Host "   pnpm shadcn init --src-dir -c `"C:/Yeasin/experiment/fix-test`""

Write-Host "`n3. TanStack + --src-dir:" -ForegroundColor Cyan
Write-Host "   pnpm shadcn init --src-dir -c `"C:/Yeasin/experiment/fix-test`""

Write-Host "`n4. Next.js without --src-dir:" -ForegroundColor Cyan
Write-Host "   pnpm shadcn init -c `"C:/Yeasin/experiment/fix-test`""

Write-Host "`n5. Create command:" -ForegroundColor Cyan
Write-Host "   pnpm shadcn create test-app --template next --src-dir -c `"C:/Yeasin/experiment/fix-test`""
```

### Expected Output for Each Test

Each test should show:

```
✔ Creating a new [Framework] project.
✔ Writing components.json.
✔ Checking registry.
✔ Updating CSS variables in [path]
✔ Installing dependencies.
✔ Created 1 file:
  - [src/]lib/utils.ts

Success! Project initialization completed.
You may now add components.
```

**NOT** this error:
```
⠋ Updating ..
ENOENT: no such file or directory, open ''
```

---

## Common Test Scenarios

### Test `init` Command

```bash
# Run shadcn init against your test project
pnpm shadcn init -c C:\Yeasin\experiment\resizable-issues-check
```

### Test `add` Command

```bash
pnpm shadcn add button -c C:\Yeasin\experiment\resizable-issues-check
pnpm shadcn add card dialog -c C:\Yeasin\experiment\resizable-issues-check
pnpm shadcn add "https://ui.shadcn.com/r/button" -c C:\Yeasin\experiment\resizable-issues-check
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
pnpm shadcn diff button -c C:\Yeasin\experiment\resizable-issues-check
```

---

## Running Unit Tests

```bash
# Run all CLI tests
pnpm shadcn:test

# Run specific test file
pnpm --filter=shadcn test src/commands/init.test.ts

# Run tests in watch mode
pnpm --filter=shadcn test --watch

# Run update-files tests specifically (relevant to the fix)
pnpm --filter=shadcn test src/utils/updaters/update-files.test.ts
```

---

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
node --inspect ./packages/shadcn/dist/index.js init -c C:\Yeasin\experiment\resizable-issues-check
```

### Check Built Output

The CLI builds to `packages/shadcn/dist/`:

```bash
# View built files
dir packages\shadcn\dist\
```

### Add Debug Logging

To debug the fix, add temporary logging in `update-files.ts`:

```typescript
// In resolveFilePath function
console.log('DEBUG resolveFilePath:', {
  fileType: file.type,
  filePath: file.path,
  target: file.target,
  framework: options.framework,
})

// In resolvePageTarget function  
console.log('DEBUG resolvePageTarget:', { target, framework })
```

---

## Project Structure

```
packages/shadcn/
├── src/
│   ├── commands/       # CLI commands (init, add, create, diff, etc.)
│   ├── registry/       # Registry fetching and parsing
│   ├── utils/          # Utilities and helpers
│   │   ├── updaters/   # File update logic (update-files.ts is here)
│   │   └── transformers/ # Code transformers
│   └── index.ts        # CLI entry point
├── test/               # Test files and fixtures
│   └── fixtures/       # Test project fixtures
└── dist/               # Built output
```

---

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

### Path Issues on Windows (Git Bash)

Use forward slashes or quote paths:

```bash
# Good
pnpm shadcn init -c "/c/Yeasin/experiment/test"
pnpm shadcn init -c "C:/Yeasin/experiment/test"

# Bad (backslashes get mangled)
pnpm shadcn init -c C:\Yeasin\experiment\test
```

---

## Checklist After Fix

- [ ] `init --src-dir` works for Next.js
- [ ] `init --src-dir` works for Vite
- [ ] `init --src-dir` works for TanStack Start
- [ ] `init` without `--src-dir` still works
- [ ] `create --src-dir` works
- [ ] `create` with preset URL works
- [ ] Existing project `init` still works
- [ ] `add` command still works
- [ ] Unit tests pass (`pnpm shadcn:test`)
