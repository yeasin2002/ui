ENOENT # CLI `--src-dir` Flag ENOENT Issue

## Summary

**Issue**: When running `shadcn init` or `shadcn create` with the `--src-dir` flag on a **new project**, the CLI crashes with `ENOENT: no such file or directory, open ''`

**Status**: Open Bug  
**GitHub Issues**: 
- [#9081](https://github.com/shadcn-ui/ui/issues/9081) - `--src-dir flag on create errors` (Dec 15, 2025)
- [#6830](https://github.com/shadcn-ui/ui/issues/6830) - `CLI not working properly` (Mar 2, 2025, 9 upvotes)

**Affected Commands**: `init`, `create`  
**Affected Frameworks**: Next.js, Vite, TanStack Start (ALL frameworks)  
**Affected Versions**: shadcn@3.6.1 and likely earlier versions  
**Affected Platforms**: macOS, Windows, Ubuntu (cross-platform issue)

---

## Key Observations

### What Works ✅
- `init` without `--src-dir` flag (new project)
- `init` on existing projects (with or without `--src-dir`)
- `create` without `--src-dir` flag

### What Fails ❌
- `init --src-dir` on new projects (all frameworks)
- `create --src-dir` on new projects (all frameworks)

---

## Reproduction Examples

### Example 1: Next.js with `--src-dir` (FAILS)

```bash
PS C:\Yeasin\experiment> bunx shadcn@latest init --src-dir
√ Would you like to start a new project? » Next.js
√ What is your project named? ... ww2
✔ Creating a new Next.js project.
√ Which color would you like to use as the base color? » Neutral
✔ Writing components.json.
✔ Checking registry.
✔ Updating CSS variables in src\app\globals.css
✔ Installing dependencies.
✔ Created 1 file:
  - src\lib\utils.ts

⠋ Updating ..

Something went wrong. Please check the error below for more details.
ENOENT: no such file or directory, open ''
```

### Example 2: Vite with `--src-dir` (FAILS)

```bash
PS C:\Yeasin\experiment> bunx shadcn@latest init --src-dir
√ Would you like to start a new project? » Vite
√ What is your project named? ... ww3
✔ Creating a new Vite project.
✔ Writing components.json.
✔ Updating CSS variables in src\index.css
✔ Installing dependencies.
✔ Created 1 file:
  - src\lib\utils.ts

⠋ Updating ..

ENOENT: no such file or directory, open ''
```

### Example 3: TanStack Start with `--src-dir` (FAILS)

```bash
PS C:\Yeasin\experiment> bunx shadcn@latest init --src-dir
√ Would you like to start a new project? » TanStack Start
√ What is your project named? ... ww4
✔ Creating a new TanStack Start project.
✔ Writing components.json.
✔ Updating CSS variables in src\styles.css
✔ Installing dependencies.
✔ Created 1 file:
  - src\lib\utils.ts

⠋ Updating ..

ENOENT: no such file or directory, open ''
```

### Example 4: Next.js WITHOUT `--src-dir` (WORKS)

```bash
PS C:\Yeasin\experiment> bunx shadcn@latest init
√ Would you like to start a new project? » Next.js
√ What is your project named? ... ww
✔ Creating a new Next.js project.
✔ Writing components.json.
✔ Updating CSS variables in app\globals.css
✔ Installing dependencies.
✔ Created 1 file:
  - lib\utils.ts

Success! Project initialization completed.
```

### Example 5: Existing Project (WORKS)

```bash
PS C:\Yeasin\experiment\my-app> pnpm dlx shadcn@latest init
√ Preflight checks.
√ Verifying framework. Found Next.js.
√ Validating Tailwind CSS config. Found v4.
√ Which color would you like to use as the base color? » Neutral
✔ Writing components.json.
✔ Created 1 file:
  - src\lib\utils.ts

Success! Project initialization completed.
```

---

## Root Cause Analysis

### The Bug Chain

The bug occurs through this chain of events:

```
init/create --src-dir
    ↓
createProject() creates new project
    ↓
runInit() → addComponents() → updateFiles()
    ↓
getProjectInfo() tries to detect framework
    ↓
Framework detection FAILS (returns "manual")
    ↓
resolveFilePath() for registry:page files
    ↓
resolvePageTarget(target, undefined) returns ""
    ↓
fs.writeFile("") → ENOENT error
```

### Detailed Code Flow

#### Step 1: Command Entry Point

**File**: `packages/shadcn/src/commands/init.ts` (lines 268-300)

```typescript
export async function runInit(options) {
  let projectInfo
  let newProjectTemplate
  
  if (!options.skipPreflight) {
    const preflight = await preFlightInit(options)
    if (preflight.errors[ERRORS.MISSING_DIR_OR_EMPTY_PROJECT]) {
      // NEW PROJECT CREATION
      const { projectPath, template } = await createProject(options)
      options.cwd = projectPath
      options.isNewProject = true
      newProjectTemplate = template  // <-- Template is known here!
      projectInfo = await getProjectInfo(options.cwd)
    }
  }
  
  // ... later calls addComponents()
}
```

**Key Issue**: The `newProjectTemplate` variable holds the framework type (`"next"`, `"vite"`, `"start"`), but this information is **NOT passed** to `addComponents()` or `updateFiles()`.

#### Step 2: addComponents Calls updateFiles

**File**: `packages/shadcn/src/utils/add-components.ts` (lines 119-124)

```typescript
await updateFiles(tree.files, config, {
  overwrite: options.overwrite,
  silent: options.silent,
  path: options.path,
  // NOTE: No framework parameter passed!
})
```

#### Step 3: updateFiles Tries to Detect Framework

**File**: `packages/shadcn/src/utils/updaters/update-files.ts` (lines 63-67)

```typescript
const [projectInfo, baseColor] = await Promise.all([
  getProjectInfo(config.resolvedPaths.cwd),  // <-- Framework detection
  // ...
])
```

#### Step 4: Framework Detection Fails

**File**: `packages/shadcn/src/utils/get-project-info.ts` (lines 50-92)

```typescript
export async function getProjectInfo(cwd: string): Promise<ProjectInfo | null> {
  const [configFiles, ...] = await Promise.all([
    fg.glob(
      "**/{next,vite,astro,app}.config.*|gatsby-config.*|composer.json|react-router.config.*",
      { cwd, deep: 3, ignore: PROJECT_SHARED_IGNORE }
    ),
    // ...
  ])

  // Next.js detection - looks for next.config.* files
  if (configFiles.find((file) => file.startsWith("next.config."))?.length) {
    type.framework = isUsingAppDir ? FRAMEWORKS["next-app"] : FRAMEWORKS["next-pages"]
    return type
  }

  // Vite detection - looks for vite.config.* files
  if (configFiles.find((file) => file.startsWith("vite.config."))?.length) {
    type.framework = FRAMEWORKS["vite"]
    return type
  }

  // If no config file found, returns framework: FRAMEWORKS["manual"]
  return type
}
```

**Why Detection Fails**:

1. **Next.js 15+**: `create-next-app` with `--no-import-alias` flag (used in `createNextProject`) may not create `next.config.ts` immediately
2. **Vite**: The template downloaded from GitHub may have different config file timing
3. **TanStack Start**: Similar timing issues with config file creation

The glob pattern `**/{next,vite,astro,app}.config.*` finds nothing because:
- The config files may not exist yet
- Or they're in a different location than expected

#### Step 5: resolveFilePath Returns Empty String

**File**: `packages/shadcn/src/utils/updaters/update-files.ts` (lines 363-372)

```typescript
export function resolveFilePath(file, config, options) {
  if (file.target) {
    let target = file.target

    if (file.type === "registry:page") {
      target = resolvePageTarget(target, options.framework)  // framework = "manual" or undefined
      if (!target) {
        return ""  // <-- Returns empty string!
      }
    }
    // ...
  }
}
```

#### Step 6: resolvePageTarget Returns Empty for Unknown Framework

**File**: `packages/shadcn/src/utils/updaters/update-files.ts` (lines 467-493)

```typescript
export function resolvePageTarget(
  target: string,
  framework?: ProjectInfo["framework"]["name"]
) {
  if (!framework) {
    return ""  // <-- Empty when framework is undefined
  }

  if (framework === "next-app") {
    return target
  }

  if (framework === "next-pages") {
    // Transform path
    return result
  }

  if (framework === "react-router") {
    // Transform path
    return result
  }

  if (framework === "laravel") {
    // Transform path
    return result
  }

  return ""  // <-- Empty for "manual", "vite", "tanstack-start", etc.
}
```

**Critical Bug**: `resolvePageTarget` only handles these frameworks:
- `next-app` ✅
- `next-pages` ✅
- `react-router` ✅
- `laravel` ✅

It returns `""` for:
- `manual` ❌
- `vite` ❌
- `tanstack-start` ❌
- `undefined` ❌

#### Step 7: fs.writeFile Called with Empty Path

```typescript
await fs.writeFile(filePath, content, "utf-8")  // filePath = ""
```

This causes: `ENOENT: no such file or directory, open ''`

---

## Why `--src-dir` Triggers the Bug

The `--src-dir` flag itself is **not the direct cause**, but it correlates with the issue because:

1. **Timing**: When `--src-dir` is used, the project structure is slightly different
2. **Config file location**: With `src` directory, config files may be detected differently
3. **Framework detection timing**: The `getProjectInfo()` call happens before all files are fully in place

**However**, the real issue is that:
1. Framework detection fails for newly created projects
2. `resolvePageTarget` doesn't handle all framework types
3. There's no validation before writing to empty file paths

---

## Impact Assessment

### Affected Users
- Anyone using `shadcn init --src-dir` on new projects
- Anyone using `shadcn create --src-dir`
- All frameworks: Next.js, Vite, TanStack Start

### Severity: **High**
- Blocks new project creation workflow with `--src-dir`
- No workaround except manually creating projects without `--src-dir`

---

## Solution Options

### Option 1: Pass Framework from Template (Recommended)

**Files to modify**:
- `packages/shadcn/src/commands/init.ts`
- `packages/shadcn/src/commands/create.ts`
- `packages/shadcn/src/utils/add-components.ts`
- `packages/shadcn/src/utils/updaters/update-files.ts`

**Approach**: Pass the known template/framework through the call chain.

```typescript
// In init.ts - runInit function
const config = await runInit(options)

// Pass template info to addComponents
await addComponents(components, fullConfig, {
  overwrite: true,
  silent: options.silent,
  baseStyle: options.baseStyle,
  isNewProject: options.isNewProject,
  framework: newProjectTemplate === "next" ? "next-app" 
           : newProjectTemplate === "vite" ? "vite"
           : newProjectTemplate === "start" ? "tanstack-start"
           : undefined,
})
```

```typescript
// In add-components.ts
export async function addComponents(
  components: string[],
  config: Config,
  options: {
    // ... existing options
    framework?: ProjectInfo["framework"]["name"]  // Add this
  }
)

// Pass to updateFiles
await updateFiles(tree.files, config, {
  overwrite: options.overwrite,
  silent: options.silent,
  path: options.path,
  framework: options.framework,  // Add this
})
```

```typescript
// In update-files.ts
export async function updateFiles(
  files: RegistryItem["files"],
  config: Config,
  options: {
    // ... existing options
    framework?: ProjectInfo["framework"]["name"]  // Add this
  }
)

// Use passed framework or detected framework
const framework = options.framework ?? projectInfo?.framework.name
```

**Pros**:
- Fixes the root cause
- No guessing - we know the template type
- Works for all frameworks

**Cons**:
- Requires changes to multiple files
- Need to update function signatures

---

### Option 2: Fix resolvePageTarget to Handle All Frameworks

**File**: `packages/shadcn/src/utils/updaters/update-files.ts`

```typescript
export function resolvePageTarget(
  target: string,
  framework?: ProjectInfo["framework"]["name"]
) {
  if (!framework) {
    return ""
  }

  // Handle Next.js App Router
  if (framework === "next-app") {
    return target
  }

  // Handle Next.js Pages Router
  if (framework === "next-pages") {
    let result = target.replace(/^app\//, "pages/")
    result = result.replace(/\/page(\.[jt]sx?)$/, "$1")
    return result
  }

  // Handle React Router
  if (framework === "react-router") {
    let result = target.replace(/^app\//, "app/routes/")
    result = result.replace(/\/page(\.[jt]sx?)$/, "$1")
    return result
  }

  // Handle Laravel
  if (framework === "laravel") {
    let result = target.replace(/^app\//, "resources/js/pages/")
    result = result.replace(/\/page(\.[jt]sx?)$/, "$1")
    return result
  }

  // ADD: Handle Vite - use src/pages or just return target
  if (framework === "vite") {
    return target.replace(/^app\//, "src/")
  }

  // ADD: Handle TanStack Start
  if (framework === "tanstack-start") {
    return target.replace(/^app\//, "src/routes/")
  }

  // ADD: Handle manual/unknown - return target as-is instead of empty
  return target
}
```

**Pros**:
- Simple fix in one location
- Handles all frameworks

**Cons**:
- May not be the correct path transformation for all frameworks
- Doesn't fix the framework detection issue

---

### Option 3: Add Empty Path Validation (Defensive)

**File**: `packages/shadcn/src/utils/updaters/update-files.ts`

```typescript
let filePath = resolveFilePath(file, config, {
  isSrcDir: projectInfo?.isSrcDir,
  framework: projectInfo?.framework.name,
  // ...
})

// ADD: Validate file path before writing
if (!filePath) {
  logger.warn(`Could not resolve path for file: ${file.path} (framework: ${projectInfo?.framework.name}), skipping...`)
  continue
}
```

**Pros**:
- Prevents crash
- Provides useful warning message

**Cons**:
- Doesn't fix the underlying issue
- User won't get the expected file created

---

### Option 4: Improve Framework Detection

**File**: `packages/shadcn/src/utils/get-project-info.ts`

Add fallback detection using `package.json`:

```typescript
// After checking for config files, add fallback
if (packageJson?.dependencies?.next || packageJson?.devDependencies?.next) {
  type.framework = isUsingAppDir ? FRAMEWORKS["next-app"] : FRAMEWORKS["next-pages"]
  type.frameworkVersion = await getFrameworkVersion(type.framework, packageJson)
  return type
}

if (packageJson?.devDependencies?.vite) {
  type.framework = FRAMEWORKS["vite"]
  return type
}

if (packageJson?.dependencies?.["@tanstack/react-start"]) {
  type.framework = FRAMEWORKS["tanstack-start"]
  return type
}
```

**Pros**:
- More robust framework detection
- Works even without config files

**Cons**:
- More complex change
- May have unintended side effects in other scenarios

---

## Recommended Fix

**Implement Option 1 + Option 3 together**:

1. **Option 1**: Pass framework explicitly through the call chain - fixes the root cause
2. **Option 3**: Add empty path validation - defensive programming for edge cases

### Implementation Steps

1. Add `framework` parameter to `addComponents` options interface
2. Add `framework` parameter to `updateFiles` options interface
3. In `init.ts`, map template to framework and pass to `addComponents`
4. In `create.ts`, map template to framework and pass to `updateFiles`
5. In `updateFiles`, prefer passed framework over detected
6. Add validation to skip/warn on empty file paths

---

## Testing

### Manual Testing

After implementing the fix, test with:

```bash
# From the shadcn/ui repo root
pnpm shadcn:build

# Test init with --src-dir for all frameworks
pnpm shadcn init --src-dir -c /path/to/test/dir  # Select Next.js
pnpm shadcn init --src-dir -c /path/to/test/dir  # Select Vite
pnpm shadcn init --src-dir -c /path/to/test/dir  # Select TanStack Start

# Test create with --src-dir
pnpm shadcn create test-app --template next --src-dir -c /path/to/test/dir
pnpm shadcn create test-app --template vite --src-dir -c /path/to/test/dir
pnpm shadcn create test-app --template start --src-dir -c /path/to/test/dir

# Test without --src-dir (should still work)
pnpm shadcn init -c /path/to/test/dir
pnpm shadcn create test-app --template next -c /path/to/test/dir
```

### Expected Results

- All commands should complete successfully
- Appropriate files should be created in correct locations
- No ENOENT errors

---

## Files to Modify

| File | Change |
|------|--------|
| `packages/shadcn/src/commands/init.ts` | Pass `framework` to `addComponents` based on `newProjectTemplate` |
| `packages/shadcn/src/commands/create.ts` | Pass `framework` to `updateFiles` based on `template` |
| `packages/shadcn/src/utils/add-components.ts` | Accept `framework` in options, pass to `updateFiles` |
| `packages/shadcn/src/utils/updaters/update-files.ts` | Accept `framework` in options, add empty path validation |

---

## Potential Side Effects

### Commands to Test After Fix

1. **`init` on existing projects** - Should still work (framework detected from config)
2. **`add` command** - Should still work (uses same `updateFiles`)
3. **`diff` command** - Should not be affected
4. **Workspace/monorepo scenarios** - Test `addWorkspaceComponents`

### Edge Cases to Consider

1. What if user has both `next.config.ts` AND passes `--template vite`?
2. What if framework detection succeeds but returns different framework than template?
3. What about `next-monorepo` template?

---

## Related Issues

- Issue #6830 mentions `EPERM: operation not permitted, scandir` which may be a different but related path resolution issue on Windows
- Both issues involve the `--src-dir` flag and project creation

---

## Timeline

- **Mar 2, 2025**: Issue #6830 opened (9 upvotes, 11 comments)
- **Dec 15, 2025**: Issue #9081 opened with clearer reproduction steps
- **Dec 17, 2025**: Root cause analysis completed

---

## References

- [GitHub Issue #9081](https://github.com/shadcn-ui/ui/issues/9081)
- [GitHub Issue #6830](https://github.com/shadcn-ui/ui/issues/6830)
- `packages/shadcn/src/commands/init.ts`
- `packages/shadcn/src/commands/create.ts`
- `packages/shadcn/src/utils/add-components.ts`
- `packages/shadcn/src/utils/updaters/update-files.ts`
- `packages/shadcn/src/utils/get-project-info.ts`
