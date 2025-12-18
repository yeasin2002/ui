# Resizable v4 Migration - Remaining Changes

**Date:** December 17, 2025  
**Status:** In Progress (Core UI + Examples completed)

Based on a deep scan of the codebase, here are the remaining changes needed to complete the `react-resizable-panels` v4 migration.

---

## Summary

| Category               | Total Files | ✅ Done | ❌ Remaining |
| ---------------------- | ----------- | ------- | ------------ |
| Core UI Components     | 3           | ✅ 3    | 0            |
| Examples (new-york-v4) | 4           | ✅ 4    | 0            |
| Examples (bases)       | 2           | ✅ 2    | 0            |
| Sink Demo              | 1           | ✅ 1    | 0            |
| Website Components     | 2           | ✅ 1    | 1            |
| Documentation          | 1           | ✅ 1    | 0            |
| Dependencies           | 1           | ✅      | 0            |

**Package version:** ✅ Already updated to `^4.0.1` in `apps/v4/package.json`

---

## 1. Core UI Components (CSS Classes)

These files still use the old `data-[panel-group-direction=vertical]` CSS classes instead of `aria-[orientation=...]`.

### `apps/v4/registry/new-york-v4/ui/resizable.tsx`

**Current (line 17):**

```tsx
"flex h-full w-full data-[panel-group-direction=vertical]:flex-col",
```

**Change to:**

```tsx
"flex h-full w-full aria-[orientation=vertical]:flex-col",
```

**Current (line 42) - ResizableHandle className:**

```tsx
"... data-[panel-group-direction=vertical]:h-px data-[panel-group-direction=vertical]:w-full data-[panel-group-direction=vertical]:after:left-0 data-[panel-group-direction=vertical]:after:h-1 data-[panel-group-direction=vertical]:after:w-full data-[panel-group-direction=vertical]:after:translate-x-0 data-[panel-group-direction=vertical]:after:-translate-y-1/2 [&[aria-orientation=vertical]>div]:rotate-90"
```

**Change to:**

```tsx
"... aria-[orientation=horizontal]:h-px aria-[orientation=horizontal]:w-full aria-[orientation=horizontal]:after:left-0 aria-[orientation=horizontal]:after:h-1 aria-[orientation=horizontal]:after:w-full aria-[orientation=horizontal]:after:translate-x-0 aria-[orientation=horizontal]:after:-translate-y-1/2 [&[aria-orientation=horizontal]>div]:rotate-90"
```

> **Important:** For `Separator` (handle), use `horizontal` because separator orientation is **opposite** of group orientation. A vertical group has horizontal separators.

---

### `apps/v4/registry/bases/base/ui/resizable.tsx`

Same changes as above:

- Line 22: `data-[panel-group-direction=vertical]:flex-col` → `aria-[orientation=vertical]:flex-col`
- Line 47: All `data-[panel-group-direction=vertical]:...` → `aria-[orientation=horizontal]:...`

---

### `apps/v4/registry/bases/radix/ui/resizable.tsx`

Same changes as above:

- Line 16: `data-[panel-group-direction=vertical]:flex-col` → `aria-[orientation=vertical]:flex-col`
- Line 41: All `data-[panel-group-direction=vertical]:...` → `aria-[orientation=horizontal]:...`

---

## 2. Example Files (`direction` → `orientation`)

### ✅ `apps/v4/registry/new-york-v4/examples/resizable-demo.tsx`

Already updated to use `orientation="horizontal"` and `orientation="vertical"`.

### ✅ `apps/v4/registry/new-york-v4/examples/resizable-handle.tsx`

**Line 10:**

```diff
- direction="horizontal"
+ orientation="horizontal"
```

### ✅ `apps/v4/registry/new-york-v4/examples/resizable-vertical.tsx`

**Line 10:**

```diff
- direction="vertical"
+ orientation="vertical"
```

### ✅ `apps/v4/registry/new-york-v4/examples/resizable-demo-with-handle.tsx`

**Lines 10, 20:**

```diff
- direction="horizontal"
+ orientation="horizontal"

- direction="vertical"
+ orientation="vertical"
```

---

## 3. Base Example Files

### ✅ `apps/v4/registry/bases/base/examples/resizable-example.tsx`

Multiple instances (lines 31, 54, 77, 99, 107, 133):

```diff
- direction="horizontal"
+ orientation="horizontal"

- direction="vertical"
+ orientation="vertical"
```

### ✅ `apps/v4/registry/bases/radix/examples/resizable-example.tsx`

Same changes as base examples above.

---

## 4. Sink Demo ✅ COMPLETED

### ✅ `apps/v4/app/(internal)/sink/components/resizable-demo.tsx`

Already updated to use `orientation="horizontal"` and `orientation="vertical"`.

---

## 5. Website Components (Imperative API)

### ✅ `apps/v4/components/block-viewer.tsx`

**COMPLETED** - All changes have been applied:

- ✅ Import changed to `usePanelRef` from `react-resizable-panels`
- ✅ Context type uses `ReturnType<typeof usePanelRef> | null`
- ✅ Uses `usePanelRef()` hook instead of `useRef`
- ✅ Uses `panelRef={resizablePanelRef}` prop
- ✅ Uses `orientation="horizontal"`

---

### ❌ `apps/v4/app/(create)/components/preview.tsx`

**HAS TYPESCRIPT ERROR** - This file imports `ImperativePanelHandle` but **doesn't actually render** a `ResizablePanel`. The ref is created but appears unused in the current code.

**Current (line 4):**

```tsx
import { type ImperativePanelHandle } from "react-resizable-panels"
```

**Current (line 17):**

```tsx
const resizablePanelRef = React.useRef<ImperativePanelHandle>(null)
```

**Current (lines 21-25):**

```tsx
React.useEffect(() => {
  if (resizablePanelRef.current && params.size) {
    resizablePanelRef.current.resize(params.size)
  }
}, [params.size])
```

**Recommended Fix:** Remove unused code since no `ResizablePanel` is rendered:

```diff
- import { type ImperativePanelHandle } from "react-resizable-panels"
...
- const resizablePanelRef = React.useRef<ImperativePanelHandle>(null)
...
- React.useEffect(() => {
-   if (resizablePanelRef.current && params.size) {
-     resizablePanelRef.current.resize(params.size)
-   }
- }, [params.size])
```

**Note:** Also remove the unused `setIframeKey` variable or use it (currently declared but never read).

---

## 6. Documentation

### ✅ `apps/v4/content/docs/components/resizable.mdx`

**COMPLETED** - All code examples and text have been updated:

- ✅ Usage example uses `orientation="horizontal"`
- ✅ Description text says "Use the `orientation` prop..."
- ✅ Vertical code example uses `orientation="vertical"`
- ✅ Handle example uses `orientation="horizontal"`

---

## Quick Reference: CSS Class Mapping

| Old (v3)                                                 | New (v4)                                         | Notes           |
| -------------------------------------------------------- | ------------------------------------------------ | --------------- |
| `data-[panel-group-direction=vertical]:flex-col`         | `aria-[orientation=vertical]:flex-col`           | For Group       |
| `data-[panel-group-direction=vertical]:h-px`             | `aria-[orientation=horizontal]:h-px`             | For Separator\* |
| `data-[panel-group-direction=vertical]:w-full`           | `aria-[orientation=horizontal]:w-full`           | For Separator\* |
| `[&[data-panel-group-direction=vertical]>div]:rotate-90` | `[&[aria-orientation=horizontal]>div]:rotate-90` | For Separator\* |

\*Separator orientation is **opposite** of Group orientation

---

## Checklist

### Core Components ✅ COMPLETED

- [x] `apps/v4/registry/new-york-v4/ui/resizable.tsx` - CSS classes ✅
- [x] `apps/v4/registry/bases/base/ui/resizable.tsx` - CSS classes ✅
- [x] `apps/v4/registry/bases/radix/ui/resizable.tsx` - CSS classes ✅

### Examples (new-york-v4) ✅ COMPLETED

- [x] `apps/v4/registry/new-york-v4/examples/resizable-demo.tsx` ✅
- [x] `apps/v4/registry/new-york-v4/examples/resizable-handle.tsx` ✅
- [x] `apps/v4/registry/new-york-v4/examples/resizable-vertical.tsx` ✅
- [x] `apps/v4/registry/new-york-v4/examples/resizable-demo-with-handle.tsx` ✅

### Examples (bases) ✅ COMPLETED

- [x] `apps/v4/registry/bases/base/examples/resizable-example.tsx` ✅
- [x] `apps/v4/registry/bases/radix/examples/resizable-example.tsx` ✅

### Sink Demo ✅ COMPLETED

- [x] `apps/v4/app/(internal)/sink/components/resizable-demo.tsx` ✅

### Website Components

- [x] `apps/v4/components/block-viewer.tsx` - Imperative API + orientation prop ✅
- [ ] `apps/v4/app/(create)/components/preview.tsx` - Remove unused code

### Documentation

- [x] `apps/v4/content/docs/components/resizable.mdx` ✅

### Post-Migration

- [ ] Run `pnpm install` to ensure lockfile is correct
- [ ] Run `pnpm registry:build` to rebuild component registry
- [ ] Run `pnpm typecheck` to verify no TypeScript errors
- [ ] Run `pnpm v4:dev` and manually test resizable components
- [ ] Test all resizable examples in the docs

---

## Files with TypeScript Errors (Must Fix)

1. **`apps/v4/app/(create)/components/preview.tsx`**
   - `ImperativePanelHandle` no longer exported
   - Unused ref and effect should be removed

---

## Priority Order

1. ~~**High:** Fix `block-viewer.tsx` (has TypeScript errors, will break build)~~ ✅ DONE
2. ~~**High:** Update core UI components CSS classes (affects all users)~~ ✅ DONE
3. ~~**Medium:** Update example files (affects documentation)~~ ✅ DONE
4. ~~**Medium:** Update sink demo file~~ ✅ DONE
5. ~~**Medium:** Update documentation MDX~~ ✅ DONE
6. **Low:** Clean up `preview.tsx` (unused code, has TypeScript error)
