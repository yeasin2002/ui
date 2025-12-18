# Resizable Example Fix

Fix for `apps/v4/registry/bases/base/examples/resizable-example.tsx`

## Issue

The `ResizableControlled` component uses `onLayoutChange` incorrectly. In `react-resizable-panels` v4, `onLayoutChange` receives a `Layout` object (map of panel id to size), not an array.

**Current (incorrect):**

```tsx
function ResizableControlled() {
  const [sizes, setSizes] = React.useState([30, 70])

  return (
    <ResizablePanelGroup
      orientation="horizontal"
      className="min-h-[200px] rounded-lg border"
      onLayoutChange={(newSizes) => {
        setSizes(newSizes) // ❌ newSizes is Layout object, not array
      }}
    >
      <ResizablePanel defaultSize={30} minSize={20}>
        <div className="flex h-full flex-col items-center justify-center gap-2 p-6">
          <span className="font-semibold">{Math.round(sizes[0] ?? 30)}%</span>
        </div>
      </ResizablePanel>
      <ResizableHandle />
      <ResizablePanel defaultSize={70} minSize={30}>
        <div className="flex h-full flex-col items-center justify-center gap-2 p-6">
          <span className="font-semibold">{Math.round(sizes[1] ?? 70)}%</span>
        </div>
      </ResizablePanel>
    </ResizablePanelGroup>
  )
}
```

## Solution

Use panel `id` props and access sizes from the `Layout` object by id.

**Fixed:**

```tsx
interface ControlledLayout {
  left: number
  right: number
}

function ResizableControlled() {
  const [layout, setLayout] = React.useState<ControlledLayout>({
    left: 30,
    right: 70,
  })

  return (
    <Example title="Controlled">
      <ResizablePanelGroup
        orientation="horizontal"
        className="min-h-[200px] rounded-lg border"
        onLayoutChange={(newLayout) => {
          setLayout(newLayout as ControlledLayout)
        }}
      >
        <ResizablePanel id="left" defaultSize={30} minSize={20}>
          <div className="flex h-full flex-col items-center justify-center gap-2 p-6">
            <span className="font-semibold">{Math.round(layout.left)}%</span>
          </div>
        </ResizablePanel>
        <ResizableHandle />
        <ResizablePanel id="right" defaultSize={70} minSize={30}>
          <div className="flex h-full flex-col items-center justify-center gap-2 p-6">
            <span className="font-semibold">{Math.round(layout.right)}%</span>
          </div>
        </ResizablePanel>
      </ResizablePanelGroup>
    </Example>
  )
}
```

## Complete Diff

```diff
+interface ControlledLayout {
+  left: number
+  right: number
+}

 function ResizableControlled() {
-  const [sizes, setSizes] = React.useState([30, 70])
+  const [layout, setLayout] = React.useState<ControlledLayout>({
+    left: 30,
+    right: 70,
+  })

   return (
     <Example title="Controlled">
       <ResizablePanelGroup
         orientation="horizontal"
         className="min-h-[200px] rounded-lg border"
         onLayoutChange={(newSizes) => {
-          setSizes(newSizes)
+          setLayout(newSizes as ControlledLayout)
         }}
       >
-        <ResizablePanel defaultSize={30} minSize={20}>
+        <ResizablePanel id="left" defaultSize={30} minSize={20}>
           <div className="flex h-full flex-col items-center justify-center gap-2 p-6">
-            <span className="font-semibold">{Math.round(sizes[0] ?? 30)}%</span>
+            <span className="font-semibold">{Math.round(layout.left)}%</span>
           </div>
         </ResizablePanel>
         <ResizableHandle />
-        <ResizablePanel defaultSize={70} minSize={30}>
+        <ResizablePanel id="right" defaultSize={70} minSize={30}>
           <div className="flex h-full flex-col items-center justify-center gap-2 p-6">
-            <span className="font-semibold">{Math.round(sizes[1] ?? 70)}%</span>
+            <span className="font-semibold">{Math.round(layout.right)}%</span>
           </div>
         </ResizablePanel>
       </ResizablePanelGroup>
     </Example>
   )
 }
```

## Key Changes

| Before                 | After                                                 |
| ---------------------- | ----------------------------------------------------- |
| `useState([30, 70])`   | `useState<ControlledLayout>({ left: 30, right: 70 })` |
| No panel `id` props    | Added `id="left"` and `id="right"`                    |
| `sizes[0]`, `sizes[1]` | `layout.left`, `layout.right`                         |

## Why This Matters

In v4, `onLayoutChange` receives a `Layout` object where keys are panel IDs:

```ts
type Layout = {
  [panelId: string]: number  // percentage 0-100
}

// Example callback value:
{ left: 35.5, right: 64.5 }
```

Without `id` props on panels, the layout object uses auto-generated IDs which are not predictable.

## Also Applies To

Check if `apps/v4/registry/bases/radix/examples/resizable-example.tsx` has the same issue.

## References

- `documents/resizable-panels-usage-guide.md` - Type-safe layout patterns
- `documents/resizable-v4-migration-plan.md` - v4 breaking changes
