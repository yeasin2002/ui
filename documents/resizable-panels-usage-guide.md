# Resizable Panels Usage Guide

A comprehensive guide for using `react-resizable-panels` v4 with shadcn/ui in a type-safe manner.

## Installation

```bash
pnpm add react-resizable-panels
```

## Core Components

| Component                | Description                              |
| ------------------------ | ---------------------------------------- |
| `ResizablePanelGroup`    | Container that wraps resizable panels    |
| `ResizablePanel`         | Individual panel that can be resized     |
| `ResizableHandle`        | Draggable handle between panels          |

## Basic Usage

```tsx
import {
  ResizableHandle,
  ResizablePanel,
  ResizablePanelGroup,
} from "@/components/ui/resizable";

export function BasicExample() {
  return (
    <ResizablePanelGroup orientation="horizontal" className="min-h-[200px]">
      <ResizablePanel defaultSize={50}>
        <div className="p-4">Left Panel</div>
      </ResizablePanel>
      <ResizableHandle />
      <ResizablePanel defaultSize={50}>
        <div className="p-4">Right Panel</div>
      </ResizablePanel>
    </ResizablePanelGroup>
  );
}
```

## Type Definitions

```ts
import type { Layout } from "react-resizable-panels";

// Layout is a map of panel id to size (0-100)
type Layout = {
  [panelId: string]: number;
};

// Define your own typed layout for better type safety
interface MyLayout {
  sidebar: number;
  content: number;
  preview?: number;
}
```

## Type-Safe Layout State

### Option 1: Using the Library's Layout Type

```tsx
"use client";

import { useState } from "react";
import type { Layout } from "react-resizable-panels";
import {
  ResizableHandle,
  ResizablePanel,
  ResizablePanelGroup,
} from "@/components/ui/resizable";

export function TypeSafeResizable() {
  const [layout, setLayout] = useState<Layout>({
    sidebar: 25,
    content: 75,
  });

  return (
    <ResizablePanelGroup
      orientation="horizontal"
      defaultLayout={layout}
      onLayoutChange={setLayout}
      className="min-h-[400px] rounded-lg border"
    >
      <ResizablePanel id="sidebar" defaultSize={25} minSize={15} maxSize={40}>
        <div className="flex h-full items-center justify-center p-6">
          <span>Sidebar: {layout.sidebar?.toFixed(1)}%</span>
        </div>
      </ResizablePanel>
      <ResizableHandle withHandle />
      <ResizablePanel id="content" defaultSize={75}>
        <div className="flex h-full items-center justify-center p-6">
          <span>Content: {layout.content?.toFixed(1)}%</span>
        </div>
      </ResizablePanel>
    </ResizablePanelGroup>
  );
}
```

### Option 2: Custom Typed Layout (Recommended)

```tsx
"use client";

import { useState, useCallback } from "react";
import type { Layout } from "react-resizable-panels";
import {
  ResizableHandle,
  ResizablePanel,
  ResizablePanelGroup,
} from "@/components/ui/resizable";

// Define panel IDs as constants for type safety
const PANEL_IDS = {
  SIDEBAR: "sidebar",
  MAIN: "main",
  PREVIEW: "preview",
} as const;

type PanelId = (typeof PANEL_IDS)[keyof typeof PANEL_IDS];

// Typed layout interface
interface EditorLayout {
  [PANEL_IDS.SIDEBAR]: number;
  [PANEL_IDS.MAIN]: number;
  [PANEL_IDS.PREVIEW]: number;
}

// Default layout
const DEFAULT_LAYOUT: EditorLayout = {
  [PANEL_IDS.SIDEBAR]: 20,
  [PANEL_IDS.MAIN]: 50,
  [PANEL_IDS.PREVIEW]: 30,
};

export function TypedEditorLayout() {
  const [layout, setLayout] = useState<EditorLayout>(DEFAULT_LAYOUT);

  // Type-safe layout change handler
  const handleLayoutChange = useCallback((newLayout: Layout) => {
    setLayout(newLayout as EditorLayout);
  }, []);

  // Helper to get panel size safely
  const getPanelSize = (id: PanelId): string => {
    return layout[id]?.toFixed(1) ?? "0";
  };

  return (
    <ResizablePanelGroup
      orientation="horizontal"
      defaultLayout={DEFAULT_LAYOUT}
      onLayoutChange={handleLayoutChange}
      className="min-h-[500px] rounded-lg border"
    >
      <ResizablePanel
        id={PANEL_IDS.SIDEBAR}
        defaultSize={DEFAULT_LAYOUT[PANEL_IDS.SIDEBAR]}
        minSize={15}
        maxSize={30}
      >
        <div className="flex h-full flex-col p-4">
          <h3 className="font-semibold">Sidebar</h3>
          <p className="text-muted-foreground text-sm">
            Size: {getPanelSize(PANEL_IDS.SIDEBAR)}%
          </p>
        </div>
      </ResizablePanel>

      <ResizableHandle withHandle />

      <ResizablePanel
        id={PANEL_IDS.MAIN}
        defaultSize={DEFAULT_LAYOUT[PANEL_IDS.MAIN]}
        minSize={30}
      >
        <div className="flex h-full flex-col p-4">
          <h3 className="font-semibold">Main Content</h3>
          <p className="text-muted-foreground text-sm">
            Size: {getPanelSize(PANEL_IDS.MAIN)}%
          </p>
        </div>
      </ResizablePanel>

      <ResizableHandle withHandle />

      <ResizablePanel
        id={PANEL_IDS.PREVIEW}
        defaultSize={DEFAULT_LAYOUT[PANEL_IDS.PREVIEW]}
        minSize={20}
        collapsible
        collapsedSize={0}
      >
        <div className="flex h-full flex-col p-4">
          <h3 className="font-semibold">Preview</h3>
          <p className="text-muted-foreground text-sm">
            Size: {getPanelSize(PANEL_IDS.PREVIEW)}%
          </p>
        </div>
      </ResizablePanel>
    </ResizablePanelGroup>
  );
}
```

## Nested Panels with Type Safety

```tsx
"use client";

import { useState, useCallback } from "react";
import type { Layout } from "react-resizable-panels";
import {
  ResizableHandle,
  ResizablePanel,
  ResizablePanelGroup,
} from "@/components/ui/resizable";

interface HorizontalLayout {
  left: number;
  right: number;
}

interface VerticalLayout {
  top: number;
  bottom: number;
}

const DEFAULT_H_LAYOUT: HorizontalLayout = { left: 30, right: 70 };
const DEFAULT_V_LAYOUT: VerticalLayout = { top: 40, bottom: 60 };

export function NestedResizable() {
  const [hLayout, setHLayout] = useState<HorizontalLayout>(DEFAULT_H_LAYOUT);
  const [vLayout, setVLayout] = useState<VerticalLayout>(DEFAULT_V_LAYOUT);

  const handleHLayoutChange = useCallback((layout: Layout) => {
    setHLayout(layout as HorizontalLayout);
  }, []);

  const handleVLayoutChange = useCallback((layout: Layout) => {
    setVLayout(layout as VerticalLayout);
  }, []);

  return (
    <ResizablePanelGroup
      orientation="horizontal"
      defaultLayout={DEFAULT_H_LAYOUT}
      onLayoutChange={handleHLayoutChange}
      className="min-h-[400px] rounded-lg border"
    >
      <ResizablePanel id="left" defaultSize={30} minSize={20}>
        <div className="flex h-full items-center justify-center">
          <span>Left: {hLayout.left?.toFixed(1)}%</span>
        </div>
      </ResizablePanel>

      <ResizableHandle />

      <ResizablePanel id="right" defaultSize={70}>
        <ResizablePanelGroup
          orientation="vertical"
          defaultLayout={DEFAULT_V_LAYOUT}
          onLayoutChange={handleVLayoutChange}
        >
          <ResizablePanel id="top" defaultSize={40}>
            <div className="flex h-full items-center justify-center">
              <span>Top: {vLayout.top?.toFixed(1)}%</span>
            </div>
          </ResizablePanel>

          <ResizableHandle />

          <ResizablePanel id="bottom" defaultSize={60}>
            <div className="flex h-full items-center justify-center">
              <span>Bottom: {vLayout.bottom?.toFixed(1)}%</span>
            </div>
          </ResizablePanel>
        </ResizablePanelGroup>
      </ResizablePanel>
    </ResizablePanelGroup>
  );
}
```

## Persistent Layout with localStorage

```tsx
"use client";

import { useState, useEffect, useCallback } from "react";
import type { Layout } from "react-resizable-panels";
import {
  ResizableHandle,
  ResizablePanel,
  ResizablePanelGroup,
} from "@/components/ui/resizable";

const STORAGE_KEY = "resizable-layout";

interface AppLayout {
  nav: number;
  content: number;
}

const DEFAULT_LAYOUT: AppLayout = { nav: 20, content: 80 };

function loadLayout(): AppLayout {
  if (typeof window === "undefined") return DEFAULT_LAYOUT;
  
  try {
    const saved = localStorage.getItem(STORAGE_KEY);
    if (saved) {
      return JSON.parse(saved) as AppLayout;
    }
  } catch (e) {
    console.error("Failed to load layout:", e);
  }
  return DEFAULT_LAYOUT;
}

function saveLayout(layout: AppLayout): void {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(layout));
  } catch (e) {
    console.error("Failed to save layout:", e);
  }
}

export function PersistentLayout() {
  const [layout, setLayout] = useState<AppLayout>(DEFAULT_LAYOUT);
  const [isLoaded, setIsLoaded] = useState(false);

  // Load layout on mount
  useEffect(() => {
    setLayout(loadLayout());
    setIsLoaded(true);
  }, []);

  const handleLayoutChange = useCallback((newLayout: Layout) => {
    const typedLayout = newLayout as AppLayout;
    setLayout(typedLayout);
    saveLayout(typedLayout);
  }, []);

  // Prevent hydration mismatch
  if (!isLoaded) {
    return <div className="min-h-[400px] animate-pulse bg-muted rounded-lg" />;
  }

  return (
    <ResizablePanelGroup
      orientation="horizontal"
      defaultLayout={layout}
      onLayoutChange={handleLayoutChange}
      className="min-h-[400px] rounded-lg border"
    >
      <ResizablePanel id="nav" defaultSize={layout.nav} minSize={15} maxSize={30}>
        <div className="p-4">Navigation</div>
      </ResizablePanel>
      <ResizableHandle withHandle />
      <ResizablePanel id="content" defaultSize={layout.content}>
        <div className="p-4">Content</div>
      </ResizablePanel>
    </ResizablePanelGroup>
  );
}
```

## Using the Built-in useDefaultLayout Hook

```tsx
"use client";

import {
  Group,
  Panel,
  Separator,
  useDefaultLayout,
} from "react-resizable-panels";

export function WithDefaultLayoutHook() {
  const { defaultLayout, onLayoutChange } = useDefaultLayout({
    groupId: "my-unique-layout",
    storage: typeof window !== "undefined" ? localStorage : undefined,
  });

  return (
    <Group
      orientation="horizontal"
      defaultLayout={defaultLayout}
      onLayoutChange={onLayoutChange}
      className="min-h-[400px]"
    >
      <Panel id="sidebar" defaultSize={25}>
        Sidebar
      </Panel>
      <Separator />
      <Panel id="main" defaultSize={75}>
        Main
      </Panel>
    </Group>
  );
}
```

## Imperative API with Type Safety

```tsx
"use client";

import { useCallback } from "react";
import { useGroupRef, usePanelRef } from "react-resizable-panels";
import {
  ResizableHandle,
  ResizablePanel,
  ResizablePanelGroup,
} from "@/components/ui/resizable";

interface MyLayout {
  left: number;
  right: number;
}

export function ImperativeExample() {
  const groupRef = useGroupRef();
  const leftPanelRef = usePanelRef();

  const resetLayout = useCallback(() => {
    const newLayout: MyLayout = { left: 50, right: 50 };
    groupRef.current?.setLayout(newLayout);
  }, [groupRef]);

  const collapseLeft = useCallback(() => {
    leftPanelRef.current?.collapse();
  }, [leftPanelRef]);

  const expandLeft = useCallback(() => {
    leftPanelRef.current?.expand();
  }, [leftPanelRef]);

  const logLayout = useCallback(() => {
    const layout = groupRef.current?.getLayout() as MyLayout | undefined;
    if (layout) {
      console.log(`Left: ${layout.left}%, Right: ${layout.right}%`);
    }
  }, [groupRef]);

  return (
    <div className="space-y-4">
      <div className="flex gap-2">
        <button onClick={resetLayout} className="px-3 py-1 border rounded">
          Reset (50/50)
        </button>
        <button onClick={collapseLeft} className="px-3 py-1 border rounded">
          Collapse Left
        </button>
        <button onClick={expandLeft} className="px-3 py-1 border rounded">
          Expand Left
        </button>
        <button onClick={logLayout} className="px-3 py-1 border rounded">
          Log Layout
        </button>
      </div>

      <ResizablePanelGroup
        groupRef={groupRef}
        orientation="horizontal"
        className="min-h-[300px] rounded-lg border"
      >
        <ResizablePanel
          id="left"
          panelRef={leftPanelRef}
          defaultSize={30}
          minSize={15}
          collapsible
          collapsedSize={0}
        >
          <div className="p-4">Left Panel</div>
        </ResizablePanel>
        <ResizableHandle withHandle />
        <ResizablePanel id="right" defaultSize={70}>
          <div className="p-4">Right Panel</div>
        </ResizablePanel>
      </ResizablePanelGroup>
    </div>
  );
}
```

## Props Reference

### ResizablePanelGroup

| Prop             | Type                       | Description                          |
| ---------------- | -------------------------- | ------------------------------------ |
| `orientation`    | `"horizontal" \| "vertical"` | Layout direction                   |
| `defaultLayout`  | `Layout`                   | Initial panel sizes                  |
| `onLayoutChange` | `(layout: Layout) => void` | Called when sizes change             |
| `groupRef`       | `Ref<GroupImperativeHandle>` | Imperative API access              |
| `disabled`       | `boolean`                  | Disable resizing                     |
| `disableCursor`  | `boolean`                  | Disable cursor style changes         |

### ResizablePanel

| Prop            | Type                        | Description                         |
| --------------- | --------------------------- | ----------------------------------- |
| `id`            | `string`                    | Unique panel identifier             |
| `defaultSize`   | `number`                    | Initial size (0-100)                |
| `minSize`       | `number`                    | Minimum size (0-100)                |
| `maxSize`       | `number`                    | Maximum size (0-100)                |
| `collapsible`   | `boolean`                   | Allow collapsing                    |
| `collapsedSize` | `number`                    | Size when collapsed (default: 0)    |
| `panelRef`      | `Ref<PanelImperativeHandle>` | Imperative API access              |
| `onResize`      | `(size: number) => void`    | Called when this panel resizes      |

### ResizableHandle

| Prop         | Type      | Description                              |
| ------------ | --------- | ---------------------------------------- |
| `withHandle` | `boolean` | Show visible grip handle                 |
| `disabled`   | `boolean` | Disable this specific handle             |
