# Block Viewer v4 Migration Fix

Guide for fixing the `react-resizable-panels` v4 migration issues in `apps/v4/components/block-viewer.tsx`.

## Current Issues

The file has TypeScript errors because it uses the old v3 imperative API:

```
Error: Module '"react-resizable-panels"' has no exported member 'ImperativePanelHandle'.
Error: Property 'ref' does not exist on type... (should be 'panelRef')
```

## Changes Required

### 1. Update Import (Line 19)

**Current:**

```tsx
import { type ImperativePanelHandle } from "react-resizable-panels"
```

**Change to:**

```tsx
import { usePanelRef, type PanelImperativeHandle } from "react-resizable-panels"
```

---

### 2. Update Context Type (Line 67)

**Current:**

```tsx
type BlockViewerContext = {
  // ...
  resizablePanelRef: React.RefObject<ImperativePanelHandle | null> | null
  // ...
}
```

**Change to:**

```tsx
type BlockViewerContext = {
  // ...
  resizablePanelRef: ReturnType<typeof usePanelRef> | null
  // ...
}
```

Or alternatively:

```tsx
import { type PanelImperativeHandle } from "react-resizable-panels"

type BlockViewerContext = {
  // ...
  resizablePanelRef: React.RefObject<PanelImperativeHandle | null> | null
  // ...
}
```

---

### 3. Update Ref Creation (Line 97)

**Current:**

```tsx
const resizablePanelRef = React.useRef<ImperativePanelHandle>(null)
```

**Change to:**

```tsx
const resizablePanelRef = usePanelRef()
```

---

### 4. Update Panel Ref Prop (Line 262)

**Current:**

```tsx
<ResizablePanel
  ref={resizablePanelRef}
  className="..."
  defaultSize={100}
  minSize={30}
>
```

**Change to:**

```tsx
<ResizablePanel
  panelRef={resizablePanelRef}
  className="..."
  defaultSize={100}
  minSize={30}
>
```

---

## Complete Diff

```diff
// Line 19: Update import
- import { type ImperativePanelHandle } from "react-resizable-panels"
+ import { usePanelRef, type PanelImperativeHandle } from "react-resizable-panels"

// Line 67: Update context type
  type BlockViewerContext = {
    item: z.infer<typeof registryItemSchema>
    view: "code" | "preview"
    setView: (view: "code" | "preview") => void
    activeFile: string | null
    setActiveFile: (file: string) => void
-   resizablePanelRef: React.RefObject<ImperativePanelHandle | null> | null
+   resizablePanelRef: ReturnType<typeof usePanelRef> | null
    tree: ReturnType<typeof createFileTreeForRegistryItemFiles> | null
    // ...
  }

// Line 97: Update ref creation
- const resizablePanelRef = React.useRef<ImperativePanelHandle>(null)
+ const resizablePanelRef = usePanelRef()

// Line 262: Update panel ref prop
  <ResizablePanel
-   ref={resizablePanelRef}
+   panelRef={resizablePanelRef}
    className="bg-background relative aspect-[4/2.5] overflow-hidden rounded-lg border md:aspect-auto md:rounded-xl"
    defaultSize={100}
    minSize={30}
  >
```

---

## Why These Changes?

### v3 → v4 Breaking Changes

| v3 (Old)                          | v4 (New)                 | Reason                                    |
| --------------------------------- | ------------------------ | ----------------------------------------- |
| `ImperativePanelHandle`           | `PanelImperativeHandle`  | Type renamed for consistency              |
| `useRef<ImperativePanelHandle>()` | `usePanelRef()`          | Convenience hook with proper typing       |
| `<Panel ref={...}>`               | `<Panel panelRef={...}>` | Prop renamed to avoid React ref conflicts |

### The `usePanelRef()` Hook

v4 provides a typed convenience hook that replaces manual ref creation:

```tsx
// v3 - Manual ref with type import
import { type ImperativePanelHandle } from "react-resizable-panels"
const panelRef = React.useRef<ImperativePanelHandle>(null)

// v4 - Convenience hook (recommended)
import { usePanelRef } from "react-resizable-panels"
const panelRef = usePanelRef()
```

The hook returns a properly typed `RefObject<PanelImperativeHandle>`.

---

## Usage After Fix

The imperative API usage in `BlockViewerToolbar` (line 163) will continue to work:

```tsx
onValueChange={(value) => {
  setView("preview")
  if (resizablePanelRef?.current) {
    resizablePanelRef.current.resize(parseInt(value))
  }
}}
```

The `resize()` method signature is the same in v4, so no changes needed there.

---

## Verification

After making these changes:

1. Run `pnpm typecheck` to verify no TypeScript errors
2. Run `pnpm v4:dev` and test the block viewer
3. Verify the Desktop/Tablet/Mobile toggle buttons work correctly

---

## Related Files

- `apps/v4/app/(create)/components/preview.tsx` - Also uses `ImperativePanelHandle` but the code appears unused and can be removed entirely

---

## References

- [react-resizable-panels v4 Changelog](https://github.com/bvaughn/react-resizable-panels/blob/main/CHANGELOG.md)
- `documents/imperative-panel-api-guide.md` - Complete imperative API guide
- `documents/resizable-v4-remaining-changes.md` - Migration checklist
