# Resizable v4 Migration - Pre-PR Verification Checklist

**Date:** December 18, 2025  
**Purpose:** Verify all changes are correct before submitting PR for `react-resizable-panels` v4 migration.

---

## Problem Statement

**Issue:** [GitHub Issue #9118](https://github.com/shadcn-ui/ui/issues/9118)

The `react-resizable-panels` library released v4.0.0 with breaking changes. The shadcn/ui codebase was using v3 APIs which caused:

1. **TypeScript errors** - `ImperativePanelHandle` type no longer exported
2. **Runtime failures** - `direction` prop renamed to `orientation`
3. **Styling broken** - CSS selectors changed from `data-[panel-group-direction=...]` to `aria-[orientation=...]`

### Breaking Changes in v4

| v3 (Old) | v4 (New) | Impact |
|----------|----------|--------|
| `direction` prop | `orientation` prop | All usages break |
| `data-[panel-group-direction=...]` | `aria-[orientation=...]` | CSS styling breaks |
| `ImperativePanelHandle` type | `usePanelRef()` hook | TypeScript errors |
| `<Panel ref={...}>` | `<Panel panelRef={...}>` | Imperative API breaks |
| `onLayout` callback | `onResize` callback | Event handlers break |

### Solution

Update all resizable component files to use v4 API:
- Change `direction` → `orientation` prop in all examples and docs
- Update CSS classes to use `aria-[orientation=...]` selectors
- Replace `ImperativePanelHandle` with `usePanelRef()` hook
- Change `ref` → `panelRef` prop for imperative control
- Remove unused resizable code from `preview.tsx`

---

## Quick Commands Summary

Run these commands in order from the repository root:

```bash
# 1. Install dependencies (ensure lockfile is updated)
pnpm install

# 2. TypeScript type checking (CRITICAL - must pass)
 ❌pnpm typecheck

# 3. Lint check
 ✅pnpm lint

# 4. Build the registry (CRITICAL - regenerates registry.json)
pnpm registry:build

# 5. Build all packages
pnpm build

# 6. Run tests
pnpm test

# 7. Start dev server for manual testing
pnpm v4:dev
```

---

## Step-by-Step Verification

### Step 1: Install Dependencies

```bash
pnpm install
```

**What to check:**

- No errors during installation
- `pnpm-lock.yaml` is updated if needed
- `react-resizable-panels` version is `^4.0.1` in `apps/v4/package.json`

---

### Step 2: TypeScript Type Checking (CRITICAL)

```bash
pnpm typecheck
```

**What to check:**

- ✅ No TypeScript errors
- Specifically verify no errors in:
  - `apps/v4/registry/new-york-v4/ui/resizable.tsx`
  - `apps/v4/registry/bases/base/ui/resizable.tsx`
  - `apps/v4/registry/bases/radix/ui/resizable.tsx`
  - `apps/v4/components/block-viewer.tsx`
  - `apps/v4/app/(create)/components/preview.tsx`

**Common errors to watch for:**

- `Module '"react-resizable-panels"' has no exported member 'ImperativePanelHandle'`
- `Property 'direction' does not exist` (should be `orientation`)
- `Property 'ref' does not exist` (should be `panelRef`)

---

### Step 3: Lint Check

```bash
pnpm lint
```

**What to check:**

- No lint errors
- No unused imports warnings (especially for removed `ImperativePanelHandle`)

**Fix lint issues:**

```bash
pnpm lint:fix
```

---

### Step 4: Build Registry (CRITICAL)

```bash
pnpm registry:build
```

**What to check:**

- Build completes without errors
- `apps/v4/registry.json` is regenerated
- Registry includes updated resizable components

**Why this matters:**

- The CLI fetches components from the registry
- If registry isn't rebuilt, users will get old v3 code

---

### Step 5: Build All Packages

```bash
pnpm build
```

**What to check:**

- All packages build successfully
- No build errors in `apps/v4`
- No build errors in `packages/shadcn`

---

### Step 6: Run Tests

```bash
pnpm test
```

**What to check:**

- All tests pass
- No regressions in CLI tests

**Run specific test suites:**

```bash
pnpm shadcn:test        # CLI tests only
pnpm v4:test            # v4 app tests (if any)
```

---

### Step 7: Manual Testing (CRITICAL)

Start the development server:

```bash
pnpm v4:dev
```

Then open http://localhost:4000 and test:

#### 7.1 Documentation Page

Navigate to: `/docs/components/resizable`

**Check:**

- [ ] Page loads without errors
- [ ] All code examples show `orientation` (not `direction`)
- [ ] Interactive demos work correctly

#### 7.2 Resizable Demo Examples

Test each demo on the docs page:

**Horizontal Resizing:**

- [ ] Drag handle left/right works
- [ ] Panels resize smoothly
- [ ] Min/max constraints respected

**Vertical Resizing:**

- [ ] Drag handle up/down works
- [ ] Panels resize smoothly
- [ ] Nested panels work correctly

**Handle Visibility:**

- [ ] `withHandle` prop shows grip icon
- [ ] Handle rotates correctly for vertical orientation

#### 7.3 Block Viewer (Blocks Page)

Navigate to: `/blocks`

**Check:**

- [ ] Block previews load correctly
- [ ] Desktop/Tablet/Mobile toggle buttons work
- [ ] Resizing the preview panel works
- [ ] No console errors

#### 7.4 Keyboard Navigation

- [ ] Arrow keys resize panels
- [ ] Tab navigates between handles
- [ ] Enter/Space activates handles

#### 7.5 Touch/Mobile (Optional)

- [ ] Touch drag works on mobile devices
- [ ] Responsive layout works

---

## Files Changed Summary

### Core UI Components (CSS classes updated)

- [x] `apps/v4/registry/new-york-v4/ui/resizable.tsx`
- [x] `apps/v4/registry/bases/base/ui/resizable.tsx`
- [x] `apps/v4/registry/bases/radix/ui/resizable.tsx`

### Examples (`direction` → `orientation`)

- [x] `apps/v4/registry/new-york-v4/examples/resizable-demo.tsx`
- [x] `apps/v4/registry/new-york-v4/examples/resizable-handle.tsx`
- [x] `apps/v4/registry/new-york-v4/examples/resizable-vertical.tsx`
- [x] `apps/v4/registry/new-york-v4/examples/resizable-demo-with-handle.tsx`
- [x] `apps/v4/registry/bases/base/examples/resizable-example.tsx`
- [x] `apps/v4/registry/bases/radix/examples/resizable-example.tsx`

### Sink Demo

- [x] `apps/v4/app/(internal)/sink/components/resizable-demo.tsx`

### Website Components (Imperative API)

- [x] `apps/v4/components/block-viewer.tsx`
- [x] `apps/v4/app/(create)/components/preview.tsx`

### Documentation

- [x] `apps/v4/content/docs/components/resizable.mdx`

---

## CSS Class Changes Verification

Verify these CSS class changes in the resizable components:

### ResizablePanelGroup

| Old                                              | New                                    |
| ------------------------------------------------ | -------------------------------------- |
| `data-[panel-group-direction=vertical]:flex-col` | `aria-[orientation=vertical]:flex-col` |

### ResizableHandle (Separator)

| Old                                                      | New                                              |
| -------------------------------------------------------- | ------------------------------------------------ |
| `data-[panel-group-direction=vertical]:h-px`             | `aria-[orientation=horizontal]:h-px`             |
| `data-[panel-group-direction=vertical]:w-full`           | `aria-[orientation=horizontal]:w-full`           |
| `[&[data-panel-group-direction=vertical]>div]:rotate-90` | `[&[aria-orientation=horizontal]>div]:rotate-90` |

**Important:** Separator orientation is **opposite** of group orientation!

---

## Prop Changes Verification

Verify these prop changes in all files:

| Old                      | New                        |
| ------------------------ | -------------------------- |
| `direction="horizontal"` | `orientation="horizontal"` |
| `direction="vertical"`   | `orientation="vertical"`   |
| `ref={panelRef}`         | `panelRef={panelRef}`      |
| `onLayout={...}`         | `onResize={...}`           |

---

## Imperative API Changes Verification

### block-viewer.tsx

```tsx
// Should use:
import { usePanelRef } from "react-resizable-panels"
const resizablePanelRef = usePanelRef()
<ResizablePanel panelRef={resizablePanelRef} ...>
```

### preview.tsx

```tsx
// Should have removed:
// - import { type ImperativePanelHandle }
// - const resizablePanelRef = React.useRef<ImperativePanelHandle>(null)
// - The useEffect that calls resize()
```

---

## Git Diff Check

Before committing, review the diff:

```bash
git diff --stat
git diff apps/v4/registry/
git diff apps/v4/components/block-viewer.tsx
git diff apps/v4/app/\(create\)/components/preview.tsx
git diff apps/v4/content/docs/components/resizable.mdx
```

---

## Commit Message Template

```
fix(resizable): migrate to react-resizable-panels v4

BREAKING CHANGES:
- `direction` prop renamed to `orientation`
- CSS selectors changed from `data-[panel-group-direction=...]` to `aria-[orientation=...]`
- Imperative API: `ImperativePanelHandle` → `usePanelRef()` hook
- Imperative API: `ref` prop → `panelRef` prop

Changes:
- Updated all resizable UI components (new-york-v4, base, radix)
- Updated all resizable examples
- Updated block-viewer.tsx imperative API usage
- Removed unused resizable code from preview.tsx
- Updated documentation

Closes #9118
```

---

## Troubleshooting

### TypeScript errors about `ImperativePanelHandle`

The type was renamed. Use `usePanelRef()` hook instead:

```tsx
// Old
import { type ImperativePanelHandle } from "react-resizable-panels"
const ref = React.useRef<ImperativePanelHandle>(null)

// New
import { usePanelRef } from "react-resizable-panels"
const ref = usePanelRef()
```

### Resizing doesn't work visually

Check CSS classes are updated:

- Group: `aria-[orientation=vertical]:flex-col`
- Handle: `aria-[orientation=horizontal]:...` (opposite!)

### Registry build fails

Ensure all TypeScript errors are fixed first:

```bash
pnpm typecheck
pnpm registry:build
```

### Tests fail

Check if any tests reference old API:

```bash
pnpm test -- --grep resizable
```

---

## Final Checklist

Before creating PR:

- [ ] `pnpm install` - no errors
- [ ] `pnpm typecheck` - no errors
- [ ] `pnpm lint` - no errors
- [ ] `pnpm registry:build` - no errors
- [ ] `pnpm build` - no errors
- [ ] `pnpm test` - all pass
- [ ] Manual testing - all features work
- [ ] Git diff reviewed
- [ ] Commit message follows convention

---

## References

- [react-resizable-panels v4 Changelog](https://github.com/bvaughn/react-resizable-panels/blob/main/CHANGELOG.md)
- [GitHub Issue #9118](https://github.com/shadcn-ui/ui/issues/9118)
- `documents/resizable-v4-migration-plan.md`
- `documents/resizable-v4-remaining-changes.md`
- `documents/imperative-panel-api-guide.md`
