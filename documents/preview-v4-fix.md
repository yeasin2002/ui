# Preview Component v4 Migration Fix

Guide for fixing the `react-resizable-panels` v4 migration issues in `apps/v4/app/(create)/components/preview.tsx`.

## Current Issues

The file has TypeScript errors and unused code:

```
Error: Module '"react-resizable-panels"' has no exported member 'ImperativePanelHandle'.
Warning: 'setIframeKey' is declared but its value is never read.
```

## Analysis

Looking at the component:

1. **Line 4**: Imports `type ImperativePanelHandle` from `react-resizable-panels` (type no longer exists in v4)
2. **Line 17**: Creates `resizablePanelRef` using the old `React.useRef<ImperativePanelHandle>(null)` pattern
3. **Lines 21-25**: Has a `useEffect` that uses `resizablePanelRef.current.resize()`
4. **Line 19**: `setIframeKey` is declared but never used
5. **No `ResizablePanel`**: The component renders an `<iframe>` but **no `ResizablePanel`** component

**Key Finding**: The `resizablePanelRef` and its associated effect are **dead code** - there's no `ResizablePanel` to attach the ref to.

---

## Recommended Fix: Remove Unused Code

Since there's no `ResizablePanel` in this component, the cleanest fix is to remove the unused imports, ref, and effect entirely.

### Changes Required

#### 1. Remove Unused Import (Line 4)

**Current:**

```tsx
import { type ImperativePanelHandle } from "react-resizable-panels"
```

**Change to:**

```tsx
// Remove this line entirely
```

---

#### 2. Remove Unused Ref (Line 17)

**Current:**

```tsx
const resizablePanelRef = React.useRef<ImperativePanelHandle>(null)
```

**Change to:**

```tsx
// Remove this line entirely
```

---

#### 3. Remove Unused Effect (Lines 21-25)

**Current:**

```tsx
// Sync resizable panel with URL param changes.
React.useEffect(() => {
  if (resizablePanelRef.current && params.size) {
    resizablePanelRef.current.resize(params.size)
  }
}, [params.size])
```

**Change to:**

```tsx
// Remove this entire block
```

---

#### 4. Fix Unused `setIframeKey` (Line 19)

The `setIframeKey` is declared but never used. Either:

**Option A - Remove it:**

```tsx
// Current
const [iframeKey, setIframeKey] = React.useState(0)

// Change to
const [iframeKey] = React.useState(0)
```

**Option B - Keep for future use** (if iframe refresh functionality is planned)

---

## Complete Diff

```diff
  "use client"

  import * as React from "react"
- import { type ImperativePanelHandle } from "react-resizable-panels"

  import { DARK_MODE_FORWARD_TYPE } from "@/components/mode-switcher"
  import { Badge } from "@/registry/new-york-v4/ui/badge"
  import { RANDOMIZE_FORWARD_TYPE } from "@/app/(create)/components/customizer-controls"
  import { CMD_K_FORWARD_TYPE } from "@/app/(create)/components/item-picker"
  import { useDesignSystemSync } from "@/app/(create)/hooks/use-design-system"

  const MESSAGE_TYPE = "design-system-params"

  export function Preview() {
    const params = useDesignSystemSync()
    const iframeRef = React.useRef<HTMLIFrameElement>(null)
-   const resizablePanelRef = React.useRef<ImperativePanelHandle>(null)
    const [initialParams] = React.useState(params)
-   const [iframeKey, setIframeKey] = React.useState(0)
+   const [iframeKey] = React.useState(0)

-   // Sync resizable panel with URL param changes.
-   React.useEffect(() => {
-     if (resizablePanelRef.current && params.size) {
-       resizablePanelRef.current.resize(params.size)
-     }
-   }, [params.size])

    React.useEffect(() => {
      // ... rest of the component unchanged
```

---

## Alternative Fix: If Resizable Panel Is Needed

If the component **should** have a `ResizablePanel` (perhaps it was removed or is planned), here's how to properly implement it:

```tsx
"use client"

import * as React from "react"
import { usePanelRef } from "react-resizable-panels"

import {
  ResizableHandle,
  ResizablePanel,
  ResizablePanelGroup,
} from "@/registry/new-york-v4/ui/resizable"

// ... other imports

export function Preview() {
  const params = useDesignSystemSync()
  const iframeRef = React.useRef<HTMLIFrameElement>(null)
  const resizablePanelRef = usePanelRef() // Use the hook, not useRef
  const [initialParams] = React.useState(params)

  // Sync resizable panel with URL param changes.
  React.useEffect(() => {
    if (resizablePanelRef.current && params.size) {
      resizablePanelRef.current.resize(params.size)
    }
  }, [params.size])

  // ... rest of component

  return (
    <ResizablePanelGroup orientation="horizontal">
      <ResizablePanel
        panelRef={resizablePanelRef} // Use panelRef, not ref
        defaultSize={50}
      >
        {/* iframe content */}
      </ResizablePanel>
      <ResizableHandle />
      <ResizablePanel>{/* other content */}</ResizablePanel>
    </ResizablePanelGroup>
  )
}
```

---

## Verification

After making these changes:

1. Run `pnpm typecheck` to verify no TypeScript errors
2. Run `pnpm v4:dev` and test the create/preview page
3. Verify the preview iframe still works correctly

---

## Summary

| Issue                                | Fix                                     |
| ------------------------------------ | --------------------------------------- |
| `ImperativePanelHandle` not exported | Remove - type no longer exists in v4    |
| `resizablePanelRef` ref              | Remove - no ResizablePanel to attach to |
| `useEffect` for resize               | Remove - dead code                      |
| `setIframeKey` unused                | Remove or keep for future use           |

---

## References

- `documents/imperative-panel-api-guide.md` - Complete imperative API guide
- `documents/block-viewer-v4-fix.md` - Similar fix for block-viewer.tsx
- `documents/resizable-v4-remaining-changes.md` - Migration checklist
