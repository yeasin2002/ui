# Resizable Component v4 Migration Plan

## Overview

The `react-resizable-panels` library released v4.0.0 with breaking changes. This document outlines all files in the shadcn/ui project that need to be updated.

**Related Issue:** [GitHub Issue #9118](https://github.com/shadcn-ui/ui/issues/9118)

---

## Breaking Changes Summary

| v3 (Old)                        | v4 (New)                    | Notes                                      |
| ------------------------------- | --------------------------- | ------------------------------------------ |
| `PanelGroup`                    | `Group`                     | Component renamed                          |
| `PanelResizeHandle`             | `Separator`                 | Aligns with ARIA "separator" role          |
| `direction` prop                | `orientation` prop          | Aligns with ARIA `orientation` attribute   |
| `data-panel-group-direction`    | `aria-orientation`          | Data attribute → ARIA attribute            |
| `ImperativePanelHandle`         | `usePanelRef()` hook        | Imperative API changed                     |
| `ImperativePanelGroupHandle`    | `useGroupRef()` hook        | Imperative API changed                     |
| `autoSaveId`                    | `useDefaultLayout()` hook   | Persistence API changed                    |
| `onCollapse` / `onExpand`       | Use `onResize` instead      | Callbacks consolidated                     |

### Important: Separator Orientation is Opposite

The `Separator`'s `aria-orientation` is the **opposite** of the `Group`'s orientation:
- Horizontal group → Vertical separators (`aria-orientation="vertical"`)
- Vertical group → Horizontal separators (`aria-orientation="horizontal"`)

---

## Files to Update

### 1. UI Component Files (Core)

These are the main component definitions that get distributed via the registry.

#### `apps/v4/registry/new-york-v4/ui/resizable.tsx`

**Current:**
```tsx
import * as ResizablePrimitive from "react-resizable-panels"

// Uses: ResizablePrimitive.PanelGroup, ResizablePrimitive.Panel, ResizablePrimitive.PanelResizeHandle
// CSS: data-[panel-group-direction=vertical]:...
```

**Change to:**
```tsx
import { Group, Panel, Separator } from "react-resizable-panels"

function ResizablePanelGroup({
  className,
  ...props
}: React.ComponentProps<typeof Group>) {
  return (
    <Group
      data-slot="resizable-panel-group"
      className={cn(
        "flex h-full w-full aria-[orientation=vertical]:flex-col",
        className
      )}
      {...props}
    />
  )
}

function ResizablePanel({ ...props }: React.ComponentProps<typeof Panel>) {
  return <Panel data-slot="resizable-panel" {...props} />
}

function ResizableHandle({
  withHandle,
  className,
  ...props
}: React.ComponentProps<typeof Separator> & {
  withHandle?: boolean
}) {
  return (
    <Separator
      data-slot="resizable-handle"
      className={cn(
        "bg-border focus-visible:ring-ring relative flex w-px items-center justify-center after:absolute after:inset-y-0 after:left-1/2 after:w-1 after:-translate-x-1/2 focus-visible:ring-1 focus-visible:ring-offset-1 focus-visible:outline-hidden aria-[orientation=horizontal]:h-px aria-[orientation=horizontal]:w-full aria-[orientation=horizontal]:after:left-0 aria-[orientation=horizontal]:after:h-1 aria-[orientation=horizontal]:after:w-full aria-[orientation=horizontal]:after:translate-x-0 aria-[orientation=horizontal]:after:-translate-y-1/2 [&[aria-orientation=horizontal]>div]:rotate-90",
        className
      )}
      {...props}
    >
      {withHandle && (
        <div className="bg-border z-10 flex h-4 w-3 items-center justify-center rounded-xs border">
          <GripVerticalIcon className="size-2.5" />
        </div>
      )}
    </Separator>
  )
}
```

#### `apps/v4/registry/bases/base/ui/resizable.tsx`

Same changes as above, with `cn-resizable-*` class prefixes.

#### `apps/v4/registry/bases/radix/ui/resizable.tsx`

Same changes as above, with `cn-resizable-*` class prefixes.

---

### 2. Example Files

All example files need `direction` → `orientation` prop change.

#### `apps/v4/registry/new-york-v4/examples/resizable-demo.tsx`

```diff
- <ResizablePanelGroup direction="horizontal" ...>
+ <ResizablePanelGroup orientation="horizontal" ...>

- <ResizablePanelGroup direction="vertical">
+ <ResizablePanelGroup orientation="vertical">
```

#### `apps/v4/registry/new-york-v4/examples/resizable-demo-with-handle.tsx`

```diff
- <ResizablePanelGroup direction="horizontal" ...>
+ <ResizablePanelGroup orientation="horizontal" ...>

- <ResizablePanelGroup direction="vertical">
+ <ResizablePanelGroup orientation="vertical">
```

#### `apps/v4/registry/new-york-v4/examples/resizable-vertical.tsx`

```diff
- <ResizablePanelGroup direction="vertical" ...>
+ <ResizablePanelGroup orientation="vertical" ...>
```

#### `apps/v4/registry/new-york-v4/examples/resizable-handle.tsx`

```diff
- <ResizablePanelGroup direction="horizontal" ...>
+ <ResizablePanelGroup orientation="horizontal" ...>
```

#### `apps/v4/registry/bases/base/examples/resizable-example.tsx`

Multiple instances of `direction` → `orientation`.

#### `apps/v4/registry/bases/radix/examples/resizable-example.tsx`

Multiple instances of `direction` → `orientation`.

#### `apps/v4/app/(internal)/sink/components/resizable-demo.tsx`

Multiple instances of `direction` → `orientation`.

---

### 3. Website Components (Imperative API)

These files use `ImperativePanelHandle` which needs to change to `usePanelRef()`.

#### `apps/v4/components/block-viewer.tsx`

**Current:**
```tsx
import { type ImperativePanelHandle } from "react-resizable-panels"

const resizablePanelRef = React.useRef<ImperativePanelHandle>(null)

<ResizablePanel ref={resizablePanelRef} ...>
```

**Change to:**
```tsx
import { usePanelRef } from "react-resizable-panels"

const resizablePanelRef = usePanelRef()

<ResizablePanel panelRef={resizablePanelRef} ...>
```

Also update:
```diff
- <ResizablePanelGroup direction="horizontal" ...>
+ <ResizablePanelGroup orientation="horizontal" ...>
```

#### `apps/v4/app/(create)/components/preview.tsx`

**Current:**
```tsx
import { type ImperativePanelHandle } from "react-resizable-panels"

const resizablePanelRef = React.useRef<ImperativePanelHandle>(null)
```

**Change to:**
```tsx
import { usePanelRef } from "react-resizable-panels"

const resizablePanelRef = usePanelRef()
```

Note: This file doesn't render a `ResizablePanel` directly, so verify if the ref is actually used.

---

### 4. Documentation Files

#### `apps/v4/content/docs/components/resizable.mdx`

Update all code examples:

```diff
- <ResizablePanelGroup direction="horizontal">
+ <ResizablePanelGroup orientation="horizontal">

- <ResizablePanelGroup direction="vertical">
+ <ResizablePanelGroup orientation="vertical">
```

Update the description text:
```diff
- Use the `direction` prop to set the direction of the resizable panels.
+ Use the `orientation` prop to set the orientation of the resizable panels.
```

---

### 5. Package Dependencies

#### `apps/v4/package.json`

```diff
- "react-resizable-panels": "^3.0.6",
+ "react-resizable-panels": "^4.0.0",
```

---

### 6. Registry Files (No Code Changes Needed)

These files reference the component but don't need code changes:
- `apps/v4/registry.json`
- `apps/v4/registry/__index__.tsx`
- `apps/v4/registry/bases/__index__.tsx`
- `apps/v4/registry/new-york-v4/ui/_registry.ts`
- `apps/v4/registry/bases/base/ui/_registry.ts`
- `apps/v4/registry/bases/radix/ui/_registry.ts`

---

## CLI Package

The CLI (`packages/shadcn`) doesn't contain resizable-specific code. It fetches components from the registry, so updating the registry components is sufficient.

---

## Templates

No resizable components found in the `templates/` folder. No changes needed.

---

## Migration Checklist

### Core Components
- [ ] `apps/v4/registry/new-york-v4/ui/resizable.tsx`
- [ ] `apps/v4/registry/bases/base/ui/resizable.tsx`
- [ ] `apps/v4/registry/bases/radix/ui/resizable.tsx`

### Examples
- [ ] `apps/v4/registry/new-york-v4/examples/resizable-demo.tsx`
- [ ] `apps/v4/registry/new-york-v4/examples/resizable-demo-with-handle.tsx`
- [ ] `apps/v4/registry/new-york-v4/examples/resizable-vertical.tsx`
- [ ] `apps/v4/registry/new-york-v4/examples/resizable-handle.tsx`
- [ ] `apps/v4/registry/bases/base/examples/resizable-example.tsx`
- [ ] `apps/v4/registry/bases/radix/examples/resizable-example.tsx`
- [ ] `apps/v4/app/(internal)/sink/components/resizable-demo.tsx`

### Website Components
- [ ] `apps/v4/components/block-viewer.tsx`
- [ ] `apps/v4/app/(create)/components/preview.tsx`

### Documentation
- [ ] `apps/v4/content/docs/components/resizable.mdx`

### Dependencies
- [ ] `apps/v4/package.json` - bump `react-resizable-panels` to `^4.0.0`

### Post-Migration
- [ ] Run `pnpm install` to update lockfile
- [ ] Run `pnpm registry:build` to rebuild component registry
- [ ] Run `pnpm test` to verify no regressions
- [ ] Run `pnpm v4:dev` and manually test resizable components

---

## CSS Class Changes Reference

| Old Class                                              | New Class                                        |
| ------------------------------------------------------ | ------------------------------------------------ |
| `data-[panel-group-direction=vertical]:flex-col`       | `aria-[orientation=vertical]:flex-col`           |
| `data-[panel-group-direction=vertical]:h-px`           | `aria-[orientation=horizontal]:h-px`*            |
| `data-[panel-group-direction=vertical]:w-full`         | `aria-[orientation=horizontal]:w-full`*          |
| `[&[data-panel-group-direction=vertical]>div]:rotate-90` | `[&[aria-orientation=horizontal]>div]:rotate-90`* |

*Note: For `Separator`, use `horizontal` because separator orientation is opposite of group orientation.

---

## Testing Notes

After migration, verify:
1. Horizontal resizing works correctly
2. Vertical resizing works correctly
3. Nested panels work correctly
4. Handle visibility toggle works
5. Keyboard navigation works (arrow keys)
6. Touch/mobile resizing works
7. Block viewer preview resizing works
8. Create page preview resizing works
