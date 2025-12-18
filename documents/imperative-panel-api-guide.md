# Imperative Panel API Guide

Complete guide for using the imperative API in `react-resizable-panels` v4, including migration from v3.

## Overview

The imperative API allows you to programmatically control panels and groups - collapse, expand, resize, and get current sizes without relying solely on user interaction.

## Breaking Changes from v3 to v4

### Type Names Changed

| v3 (Old)                     | v4 (New)               |
| ---------------------------- | ---------------------- |
| `ImperativePanelHandle`      | `PanelImperativeHandle` |
| `ImperativePanelGroupHandle` | `GroupImperativeHandle` |

### Ref Pattern Changed

| v3 (Old)                                | v4 (New)                              |
| --------------------------------------- | ------------------------------------- |
| `useRef<ImperativePanelHandle>(null)`   | `usePanelRef()` hook                  |
| `useRef<ImperativePanelGroupHandle>(null)` | `useGroupRef()` hook               |
| `<Panel ref={panelRef}>`                | `<Panel panelRef={panelRef}>`         |
| `<PanelGroup ref={groupRef}>`           | `<Group groupRef={groupRef}>`         |

### Method Changes

| v3 Panel Method      | v4 Panel Method                          |
| -------------------- | ---------------------------------------- |
| `getSize(): number`  | `getSize(): { asPercentage, inPixels }`  |
| `isExpanded(): boolean` | Removed (use `!isCollapsed()`)        |

## Migration Example

```tsx
// ❌ v3 (Old)
import { PanelGroup, Panel, PanelResizeHandle } from "react-resizable-panels";
import type { ImperativePanelGroupHandle, ImperativePanelHandle } from "react-resizable-panels";
import { useRef } from "react";

function OldComponent() {
  const panelRef = useRef<ImperativePanelHandle>(null);
  const groupRef = useRef<ImperativePanelGroupHandle>(null);

  return (
    <PanelGroup ref={groupRef} direction="horizontal">
      <Panel ref={panelRef}>Content</Panel>
      <PanelResizeHandle />
      <Panel>Content</Panel>
    </PanelGroup>
  );
}

// ✅ v4 (New)
import { Group, Panel, Separator, useGroupRef, usePanelRef } from "react-resizable-panels";

function NewComponent() {
  const panelRef = usePanelRef();
  const groupRef = useGroupRef();

  return (
    <Group groupRef={groupRef} orientation="horizontal">
      <Panel panelRef={panelRef}>Content</Panel>
      <Separator />
      <Panel>Content</Panel>
    </Group>
  );
}
```

## Type Definitions

### PanelImperativeHandle

```ts
import type { PanelImperativeHandle } from "react-resizable-panels";

interface PanelImperativeHandle {
  /**
   * Collapse the Panel to its `collapsedSize`.
   * Does nothing if Panel is not `collapsible` or already collapsed.
   */
  collapse: () => void;

  /**
   * Expand a collapsed Panel to its most recent size.
   * Does nothing if Panel is not currently collapsed.
   */
  expand: () => void;

  /**
   * Get the current size of the Panel.
   * @returns { asPercentage: number, inPixels: number }
   */
  getSize: () => PanelSize;

  /**
   * Check if the Panel is currently collapsed.
   */
  isCollapsed: () => boolean;

  /**
   * Update the Panel's size.
   * @param size - Number (pixels) or string ("50%", "100px", "10rem")
   */
  resize: (size: number | string) => void;
}

interface PanelSize {
  asPercentage: number; // 0-100
  inPixels: number;
}
```

### GroupImperativeHandle

```ts
import type { GroupImperativeHandle } from "react-resizable-panels";

interface GroupImperativeHandle {
  /**
   * Get the Group's current layout.
   * @returns Map of Panel id to percentage (0-100)
   */
  getLayout: () => { [panelId: string]: number };

  /**
   * Set a new layout for the Group.
   * @param layout - Map of Panel id to percentage (0-100)
   * @returns Applied layout (after validation)
   */
  setLayout: (layout: { [panelId: string]: number }) => Layout;
}
```

## Convenience Hooks

v4 provides typed hooks for creating refs:

```ts
import { usePanelRef, useGroupRef } from "react-resizable-panels";

// These are equivalent to:
// const panelRef = useRef<PanelImperativeHandle>(null);
// const groupRef = useRef<GroupImperativeHandle>(null);

const panelRef = usePanelRef();
const groupRef = useGroupRef();
```

### Callback Ref Variants

For cases where you need callback refs:

```ts
import { usePanelCallbackRef, useGroupCallbackRef } from "react-resizable-panels";

const panelCallbackRef = usePanelCallbackRef();
const groupCallbackRef = useGroupCallbackRef();
```

## Complete Usage Examples

### Panel Imperative API

```tsx
"use client";

import { useCallback } from "react";
import { Group, Panel, Separator, usePanelRef } from "react-resizable-panels";
import type { PanelSize } from "react-resizable-panels";

export function PanelImperativeExample() {
  const sidebarRef = usePanelRef();

  const collapseSidebar = useCallback(() => {
    sidebarRef.current?.collapse();
  }, []);

  const expandSidebar = useCallback(() => {
    sidebarRef.current?.expand();
  }, []);

  const toggleSidebar = useCallback(() => {
    if (sidebarRef.current?.isCollapsed()) {
      sidebarRef.current.expand();
    } else {
      sidebarRef.current?.collapse();
    }
  }, []);

  const getSidebarSize = useCallback(() => {
    const size: PanelSize | undefined = sidebarRef.current?.getSize();
    if (size) {
      console.log(`Sidebar: ${size.asPercentage.toFixed(1)}% (${size.inPixels}px)`);
    }
  }, []);

  const resizeSidebar = useCallback((newSize: number | string) => {
    sidebarRef.current?.resize(newSize);
  }, []);

  return (
    <div className="space-y-4">
      <div className="flex flex-wrap gap-2">
        <button onClick={collapseSidebar} className="px-3 py-1 border rounded">
          Collapse
        </button>
        <button onClick={expandSidebar} className="px-3 py-1 border rounded">
          Expand
        </button>
        <button onClick={toggleSidebar} className="px-3 py-1 border rounded">
          Toggle
        </button>
        <button onClick={getSidebarSize} className="px-3 py-1 border rounded">
          Log Size
        </button>
        <button onClick={() => resizeSidebar(15)} className="px-3 py-1 border rounded">
          Set 15%
        </button>
        <button onClick={() => resizeSidebar(30)} className="px-3 py-1 border rounded">
          Set 30%
        </button>
        <button onClick={() => resizeSidebar("200px")} className="px-3 py-1 border rounded">
          Set 200px
        </button>
      </div>

      <Group orientation="horizontal" className="min-h-[400px] rounded-lg border">
        <Panel
          panelRef={sidebarRef}
          id="sidebar"
          defaultSize={25}
          minSize={15}
          maxSize={40}
          collapsible
          collapsedSize={0}
        >
          <div className="flex h-full items-center justify-center bg-muted/50">
            <span>Sidebar</span>
          </div>
        </Panel>
        <Separator />
        <Panel id="content" defaultSize={75}>
          <div className="flex h-full items-center justify-center">
            <span>Main Content</span>
          </div>
        </Panel>
      </Group>
    </div>
  );
}
```

### Group Imperative API

```tsx
"use client";

import { useCallback } from "react";
import { Group, Panel, Separator, useGroupRef } from "react-resizable-panels";
import type { Layout } from "react-resizable-panels";

// Define typed layouts
interface EditorLayout {
  sidebar: number;
  editor: number;
  preview: number;
}

const LAYOUTS: Record<string, EditorLayout> = {
  default: { sidebar: 20, editor: 50, preview: 30 },
  focused: { sidebar: 0, editor: 70, preview: 30 },
  preview: { sidebar: 20, editor: 30, preview: 50 },
  equal: { sidebar: 33.33, editor: 33.33, preview: 33.34 },
};

export function GroupImperativeExample() {
  const groupRef = useGroupRef();

  const applyLayout = useCallback((layoutName: keyof typeof LAYOUTS) => {
    const layout = LAYOUTS[layoutName];
    groupRef.current?.setLayout(layout);
  }, []);

  const logCurrentLayout = useCallback(() => {
    const layout = groupRef.current?.getLayout() as EditorLayout | undefined;
    if (layout) {
      console.log("Current layout:", {
        sidebar: `${layout.sidebar?.toFixed(1)}%`,
        editor: `${layout.editor?.toFixed(1)}%`,
        preview: `${layout.preview?.toFixed(1)}%`,
      });
    }
  }, []);

  return (
    <div className="space-y-4">
      <div className="flex flex-wrap gap-2">
        <button onClick={() => applyLayout("default")} className="px-3 py-1 border rounded">
          Default Layout
        </button>
        <button onClick={() => applyLayout("focused")} className="px-3 py-1 border rounded">
          Focus Mode
        </button>
        <button onClick={() => applyLayout("preview")} className="px-3 py-1 border rounded">
          Preview Mode
        </button>
        <button onClick={() => applyLayout("equal")} className="px-3 py-1 border rounded">
          Equal Split
        </button>
        <button onClick={logCurrentLayout} className="px-3 py-1 border rounded">
          Log Layout
        </button>
      </div>

      <Group
        groupRef={groupRef}
        orientation="horizontal"
        defaultLayout={LAYOUTS.default}
        className="min-h-[400px] rounded-lg border"
      >
        <Panel id="sidebar" defaultSize={20} minSize={0} collapsible collapsedSize={0}>
          <div className="flex h-full items-center justify-center bg-muted/50">
            Sidebar
          </div>
        </Panel>
        <Separator />
        <Panel id="editor" defaultSize={50} minSize={20}>
          <div className="flex h-full items-center justify-center">
            Editor
          </div>
        </Panel>
        <Separator />
        <Panel id="preview" defaultSize={30} minSize={15}>
          <div className="flex h-full items-center justify-center bg-muted/30">
            Preview
          </div>
        </Panel>
      </Group>
    </div>
  );
}
```

### Combined Panel + Group Control

```tsx
"use client";

import { useCallback, useState } from "react";
import { Group, Panel, Separator, useGroupRef, usePanelRef } from "react-resizable-panels";
import type { Layout, PanelSize } from "react-resizable-panels";

interface AppLayout {
  nav: number;
  main: number;
  aside: number;
}

export function CombinedImperativeExample() {
  const groupRef = useGroupRef();
  const navRef = usePanelRef();
  const asideRef = usePanelRef();

  const [navCollapsed, setNavCollapsed] = useState(false);
  const [asideCollapsed, setAsideCollapsed] = useState(false);

  // Sync collapse state with panel state
  const updateCollapseState = useCallback(() => {
    setNavCollapsed(navRef.current?.isCollapsed() ?? false);
    setAsideCollapsed(asideRef.current?.isCollapsed() ?? false);
  }, []);

  const toggleNav = useCallback(() => {
    if (navRef.current?.isCollapsed()) {
      navRef.current.expand();
    } else {
      navRef.current?.collapse();
    }
    // Update state after a small delay to allow animation
    setTimeout(updateCollapseState, 50);
  }, [updateCollapseState]);

  const toggleAside = useCallback(() => {
    if (asideRef.current?.isCollapsed()) {
      asideRef.current.expand();
    } else {
      asideRef.current?.collapse();
    }
    setTimeout(updateCollapseState, 50);
  }, [updateCollapseState]);

  const resetLayout = useCallback(() => {
    const defaultLayout: AppLayout = { nav: 20, main: 60, aside: 20 };
    groupRef.current?.setLayout(defaultLayout);
    
    // Expand collapsed panels
    if (navRef.current?.isCollapsed()) navRef.current.expand();
    if (asideRef.current?.isCollapsed()) asideRef.current.expand();
    
    setTimeout(updateCollapseState, 50);
  }, [updateCollapseState]);

  const focusMode = useCallback(() => {
    navRef.current?.collapse();
    asideRef.current?.collapse();
    setTimeout(updateCollapseState, 50);
  }, [updateCollapseState]);

  return (
    <div className="space-y-4">
      <div className="flex flex-wrap gap-2">
        <button onClick={toggleNav} className="px-3 py-1 border rounded">
          {navCollapsed ? "Show" : "Hide"} Nav
        </button>
        <button onClick={toggleAside} className="px-3 py-1 border rounded">
          {asideCollapsed ? "Show" : "Hide"} Aside
        </button>
        <button onClick={focusMode} className="px-3 py-1 border rounded">
          Focus Mode
        </button>
        <button onClick={resetLayout} className="px-3 py-1 border rounded">
          Reset
        </button>
      </div>

      <Group
        groupRef={groupRef}
        orientation="horizontal"
        className="min-h-[400px] rounded-lg border"
      >
        <Panel
          panelRef={navRef}
          id="nav"
          defaultSize={20}
          minSize={10}
          maxSize={30}
          collapsible
          collapsedSize={0}
        >
          <div className="flex h-full items-center justify-center bg-blue-50">
            Navigation
          </div>
        </Panel>
        <Separator />
        <Panel id="main" defaultSize={60} minSize={30}>
          <div className="flex h-full items-center justify-center">
            Main Content
          </div>
        </Panel>
        <Separator />
        <Panel
          panelRef={asideRef}
          id="aside"
          defaultSize={20}
          minSize={10}
          maxSize={30}
          collapsible
          collapsedSize={0}
        >
          <div className="flex h-full items-center justify-center bg-green-50">
            Aside
          </div>
        </Panel>
      </Group>
    </div>
  );
}
```

## Using with shadcn/ui Wrapper

```tsx
"use client";

import { useCallback } from "react";
import { useGroupRef, usePanelRef } from "react-resizable-panels";
import {
  ResizableHandle,
  ResizablePanel,
  ResizablePanelGroup,
} from "@/components/ui/resizable";

export function ShadcnImperativeExample() {
  const groupRef = useGroupRef();
  const sidebarRef = usePanelRef();

  const toggleSidebar = useCallback(() => {
    if (sidebarRef.current?.isCollapsed()) {
      sidebarRef.current.expand();
    } else {
      sidebarRef.current?.collapse();
    }
  }, []);

  const resetLayout = useCallback(() => {
    groupRef.current?.setLayout({ sidebar: 25, content: 75 });
  }, []);

  return (
    <div className="space-y-4">
      <div className="flex gap-2">
        <button onClick={toggleSidebar} className="px-3 py-1 border rounded">
          Toggle Sidebar
        </button>
        <button onClick={resetLayout} className="px-3 py-1 border rounded">
          Reset Layout
        </button>
      </div>

      <ResizablePanelGroup
        groupRef={groupRef}
        orientation="horizontal"
        className="min-h-[400px] rounded-lg border"
      >
        <ResizablePanel
          panelRef={sidebarRef}
          id="sidebar"
          defaultSize={25}
          minSize={15}
          collapsible
          collapsedSize={0}
        >
          <div className="flex h-full items-center justify-center p-6">
            Sidebar
          </div>
        </ResizablePanel>
        <ResizableHandle withHandle />
        <ResizablePanel id="content" defaultSize={75}>
          <div className="flex h-full items-center justify-center p-6">
            Content
          </div>
        </ResizablePanel>
      </ResizablePanelGroup>
    </div>
  );
}
```

## Size Units in v4

v4 supports multiple size units for the `resize()` method:

```tsx
const panelRef = usePanelRef();

// Percentage (0-100)
panelRef.current?.resize(25);      // 25%
panelRef.current?.resize("25%");   // 25%

// Pixels
panelRef.current?.resize("200px"); // 200 pixels

// Relative units
panelRef.current?.resize("10rem"); // 10rem
panelRef.current?.resize("5em");   // 5em

// Viewport units
panelRef.current?.resize("25vh");  // 25% of viewport height
panelRef.current?.resize("25vw");  // 25% of viewport width
```

## Best Practices

1. **Always use the convenience hooks** (`usePanelRef`, `useGroupRef`) for type safety
2. **Check for null** before calling methods: `panelRef.current?.collapse()`
3. **Use `useCallback`** for handlers to prevent unnecessary re-renders
4. **Provide panel `id`s** when using imperative API with `setLayout()`
5. **Handle SSR** - refs are null during server rendering

## Common Patterns

### Double-click to collapse

```tsx
<Separator
  onDoubleClick={() => {
    if (panelRef.current?.isCollapsed()) {
      panelRef.current.expand();
    } else {
      panelRef.current?.collapse();
    }
  }}
/>
```

### Keyboard shortcuts

```tsx
useEffect(() => {
  const handleKeyDown = (e: KeyboardEvent) => {
    if (e.key === "b" && (e.metaKey || e.ctrlKey)) {
      e.preventDefault();
      if (sidebarRef.current?.isCollapsed()) {
        sidebarRef.current.expand();
      } else {
        sidebarRef.current?.collapse();
      }
    }
  };

  window.addEventListener("keydown", handleKeyDown);
  return () => window.removeEventListener("keydown", handleKeyDown);
}, []);
```

## References

- [react-resizable-panels GitHub](https://github.com/bvaughn/react-resizable-panels)
- [Official Documentation](https://react-resizable-panels.vercel.app/)
- [Changelog](https://github.com/bvaughn/react-resizable-panels/blob/main/CHANGELOG.md)
