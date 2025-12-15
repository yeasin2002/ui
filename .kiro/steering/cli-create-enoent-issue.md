# CLI Create Command ENOENT Issue

## Problem

When running `pnpm dlx shadcn@latest create` with a preset URL and `--src-dir` flag, the CLI crashes with:

```
ENOENT: no such file or directory, open ''
```

This happens during the "Updating..." step after the project is created.

## Error Context

```bash
pnpm dlx shadcn@latest create --preset "https://ui.shadcn.com/init?base=base&style=mira&..." --template next --src-dir blades
```

The error occurs after:
- ✔ Creating a new Next.js project
- ✔ Writing components.json
- ✔ Checking registry
- ✔ Updating CSS variables
- ✔ Installing dependencies
- ✔ Updating fonts
- ✔ Created 1 file: src/lib/utils.ts
- ⠋ Updating... (CRASH)

## Root Cause

The issue is in `packages/shadcn/src/utils/updaters/update-files.ts`:

1. The `create` command calls `updateFiles()` with template files including a `registry:page` type file
2. `updateFiles()` calls `getProjectInfo()` to detect the framework
3. If framework detection fails or returns `null`, `projectInfo?.framework.name` becomes `undefined`
4. `resolveFilePath()` calls `resolvePageTarget(target, undefined)`
5. `resolvePageTarget()` returns an empty string `""` when framework is `undefined`
6. The empty string is passed to `fs.writeFile('')`, causing `ENOENT`

### Code Flow

```typescript
// packages/shadcn/src/commands/create.ts
const templateFiles = getTemplateFiles(template as Template)
// Returns: [{ type: "registry:page", path: "app/page.tsx", target: "app/page.tsx", content: "..." }]

await updateFiles(templateFiles, config, { overwrite: true, silent: true })

// packages/shadcn/src/utils/updaters/update-files.ts
const projectInfo = await getProjectInfo(config.resolvedPaths.cwd)
// projectInfo might be null if detection fails

let filePath = resolveFilePath(file, config, {
  framework: projectInfo?.framework.name,  // undefined if projectInfo is null
  ...
})

// resolveFilePath returns "" when framework is undefined for registry:page files
if (file.type === "registry:page") {
  target = resolvePageTarget(target, options.framework)
  if (!target) {
    return ""  // <-- Returns empty string!
  }
}

// Later, fs.writeFile is called with empty string
await fs.writeFile(filePath, content, "utf-8")  // filePath = ""
```

## Potential Causes

1. **Framework detection timing**: The newly created project might not have all files in place when `getProjectInfo()` runs
2. **Missing next.config file**: Framework detection looks for `next.config.*` files
3. **Path resolution issues**: `config.resolvedPaths.cwd` might not point to the correct directory

## Solution Options

### Option 1: Pass framework explicitly (Recommended)

In `create.ts`, pass the template/framework information to `updateFiles`:

```typescript
// packages/shadcn/src/commands/create.ts
await updateFiles(templateFiles, config, {
  overwrite: true,
  silent: true,
  framework: template === "next" ? "next-app" : template,  // Pass explicitly
})
```

### Option 2: Skip empty file paths

In `update-files.ts`, skip files with empty resolved paths:

```typescript
// packages/shadcn/src/utils/updaters/update-files.ts
let filePath = resolveFilePath(file, config, { ... })

if (!filePath) {
  continue  // Skip this file instead of crashing
}
```

### Option 3: Better error handling

Add validation before writing:

```typescript
if (!filePath || filePath === "") {
  throw new Error(`Could not resolve target path for file: ${file.path}`)
}
```

## Files to Fix

- `packages/shadcn/src/commands/create.ts` - Pass framework info to updateFiles
- `packages/shadcn/src/utils/updaters/update-files.ts` - Add validation for empty paths

## Testing

After fix, test with:

```bash
pnpm dlx shadcn@latest create --preset "https://ui.shadcn.com/init?base=base&style=mira&baseColor=neutral&theme=pink&iconLibrary=lucide&font=inter&menuAccent=subtle&menuColor=default&radius=default&template=next" --template next --src-dir my-app
```
