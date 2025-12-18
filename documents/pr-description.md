# PR Description

## fix(resizable): migrate to react-resizable-panels v4

Closes #9118

### Problem

The `react-resizable-panels` library released v4.0.0 with breaking changes. The current codebase uses v3 APIs, causing:

1. **TypeScript errors** - `ImperativePanelHandle` type no longer exported
2. **Runtime failures** - `direction` prop renamed to `orientation`, breaking all resizable examples
3. **Styling broken** - CSS selectors changed from `data-[panel-group-direction=...]` to `aria-[orientation=...]`

### Solution

- Update all resizable UI components to use `aria-[orientation=...]` CSS selectors
- Change `direction` → `orientation` prop in all examples and documentation
- Replace `ImperativePanelHandle` type with `usePanelRef()`
- Remove unused resizable code from `preview.tsx` (dead code - no ResizablePanel rendered)
- Update documentation code examples

### Breaking Changes in v4

| v3                                 | v4                       |
| ---------------------------------- | ------------------------ |
| `direction` prop                   | `orientation` prop       |
| `data-[panel-group-direction=...]` | `aria-[orientation=...]` |
| `ImperativePanelHandle` type       | `usePanelRef()` hook     |
| `<Panel ref={...}>`                | `<Panel panelRef={...}>` |
