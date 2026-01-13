# shadcn-ui Issue #9162: Unable to Add Native-Select Component

## Issue Overview

**Issue Number:** #9162  
**Status:** Open  
**Type:** Bug  
**Opened:** December 20, 2025  
**Reporter:** [@d4lek](https://github.com/d4lek)  
**GitHub Link:** https://github.com/shadcn-ui/ui/issues/9162

---

## Problem Description

Users are unable to install the `native-select` component using the shadcn CLI command. When attempting to add the component, the CLI returns a 404 error indicating that the component's registry JSON file cannot be found.

---

## Error Details

### Command Used

```bash
npx shadcn@latest add native-select
```

### Error Message

```
Something went wrong. Please check the error below for more details.
If the problem persists, please open an issue on GitHub.

Message:
The item at https://ui.shadcn.com/r/styles/default/native-select.json was not found. It may not exist at the registry.

Suggestion:
Check if the item name is correct and the registry URL is accessible.
```

### Error Type

- **HTTP Status:** 404 Not Found
- **Resource:** Registry JSON file for native-select component
- **Affected URL:** `https://ui.shadcn.com/r/styles/default/native-select.json`

---

## Technical Analysis

### What's Happening

1. **Component Existence vs. Registry Availability**

   - The `native-select` component **DOES exist** in the shadcn/ui documentation
   - Documentation page: https://ui.shadcn.com/docs/components/native-select
   - The component is listed in the official components list
   - However, the **registry JSON file is missing** from the public registry

2. **Registry Structure Issue**

   - The shadcn CLI attempts to fetch component metadata from the registry endpoint
   - Expected endpoint pattern: `https://ui.shadcn.com/r/styles/{style-name}/{component-name}.json`
   - For the default style: `https://ui.shadcn.com/r/styles/default/native-select.json`
   - This endpoint returns a **404 error**, indicating the file doesn't exist

3. **Verification of the Issue**
   - Direct access to `https://ui.shadcn.com/r/styles/default/native-select.json` confirms 404 error
   - The component documentation is live and complete with examples
   - This suggests a **deployment/registry synchronization issue**

---

## Component Details

### About Native Select

The `native-select` is a styled native HTML select element with consistent design system integration. It was added to shadcn/ui as a newer component offering an alternative to the custom `Select` component.

**Key Features:**

- Styled wrapper for native HTML `<select>` element
- Maintains native HTML select accessibility features
- Better performance compared to custom select components
- Mobile-optimized with native browser dropdowns
- Consistent styling with shadcn/ui design system
- Form library integration support (React Hook Form, etc.)

**Component Structure:**

- `NativeSelect` - Main select wrapper component
- `NativeSelectOption` - Individual option element
- `NativeSelectOptGroup` - Group wrapper for organizing options

**Use Cases:**

- When native browser behavior is preferred
- Better performance requirements
- Mobile-optimized dropdowns needed
- Simpler form interactions

---

## Root Cause

### Registry Build/Deployment Gap

The issue indicates one of the following scenarios:

1. **Missing Registry Build**

   - The component was added to the codebase and documentation
   - The registry JSON files were not generated or published
   - The `shadcn build` command may not have been run for this component

2. **Deployment Synchronization Issue**

   - The documentation was deployed but registry files were not
   - Partial deployment where docs updated but public registry didn't

3. **Registry Configuration Missing**
   - The component may not be properly configured in `registry.json`
   - The component definition might be incomplete or misconfigured

---

## When This Issue Occurs

### Trigger Conditions

This issue occurs when:

1. **Using the CLI to install the component:**

   ```bash
   npx shadcn@latest add native-select
   npx shadcn add native-select
   ```

2. **Attempting installation on any project:**

   - New projects initializing shadcn/ui
   - Existing projects trying to add the component
   - Any project regardless of framework (Next.js, Vite, etc.)

3. **Any style configuration:**
   - `default` style (confirmed)
   - `new-york` style (likely affected)
   - Custom styles (would also fail as base is missing)

### System Information (from issue reporter)

- **OS:** macOS Tahoe (macOS 14.x)
- **Package Manager:** npm (implied from npx usage)
- **CLI Version:** Latest (`@latest` flag used)

---

## Impact Assessment

### Severity: Medium to High

**Affected Users:**

- Developers trying to use native select in their projects
- Teams building forms with shadcn/ui
- Projects requiring the native select component specifically

**Workarounds Available:** Yes (see below)

**User Experience Impact:**

- Confusing error message (component exists but can't be installed)
- Disrupts development workflow
- Forces manual installation

---

## Current Status

### Component Availability

✅ **Available:**

- Documentation at https://ui.shadcn.com/docs/components/native-select
- Source code in the repository
- Examples and usage instructions
- Listed in components page

❌ **Not Available:**

- Registry JSON files (`native-select.json`)
- CLI installation via `npx shadcn add`
- Automated dependency installation

### Similar Components Working

Other components in the same timeframe work correctly:

- `select` - Available and installable
- `input` - Available and installable
- Other form components - Working normally

---

## Workarounds

Until the issue is resolved, users can manually install the component:

### Method 1: Manual File Copy

1. **Visit the component documentation:**
   https://ui.shadcn.com/docs/components/native-select

2. **Copy the component source code** from the documentation page

3. **Create the component file manually:**

   ```bash
   # Create the component directory
   mkdir -p components/ui

   # Create the native-select.tsx file
   touch components/ui/native-select.tsx
   ```

4. **Paste the component code** into the file

5. **Install any dependencies** mentioned in the documentation

### Method 2: Direct Source Copy from Repository

1. Navigate to the shadcn-ui GitHub repository
2. Find the component source in `apps/www/registry/`
3. Copy the native-select component files
4. Place them in your project's components directory

### Method 3: Use Alternative Solutions

1. **Use the custom Select component:**

   ```bash
   npx shadcn@latest add select
   ```

2. **Create a custom native select wrapper** using shadcn/ui primitives

---

## Expected Behavior

### What Should Happen

1. User runs: `npx shadcn@latest add native-select`
2. CLI fetches component metadata from registry
3. Component files are generated in the project
4. Dependencies are automatically installed
5. Component is ready to use

### Correct Registry Structure

The registry should contain:

```
https://ui.shadcn.com/r/styles/
├── default/
│   └── native-select.json  ← Missing!
└── new-york/
    └── native-select.json  ← Likely missing!
```

---

## Resolution Path

### For shadcn/ui Maintainers

To resolve this issue, the following steps are needed:

1. **Verify registry configuration:**

   - Check `registry.json` includes native-select definition
   - Ensure all required files are properly referenced

2. **Build the registry:**

   ```bash
   npm run registry:build
   # or
   shadcn build
   ```

3. **Deploy the registry files:**

   - Ensure JSON files are generated in `public/r/` directory
   - Deploy to production hosting
   - Verify accessibility of the endpoint

4. **Test the installation:**
   ```bash
   npx shadcn@latest add native-select
   ```

### Timeline

- **Issue Reported:** December 20, 2025
- **Issue Status:** Currently open
- **Expected Resolution:** Pending maintainer response

---

## Related Information

### Similar Past Issues

This type of registry issue has occurred before with other components:

- **Issue #9118:** Resizable component broken (different issue type)
- **Issue #8050:** JSON registry error for multiple components
- **Issue #6831:** Registry URL 404 errors
- **Issue #2188:** Failed to fetch tree from registry

### Common Registry Issues Pattern

Registry problems in shadcn/ui typically involve:

1. Missing or corrupted JSON files
2. Deployment synchronization issues
3. Build process not completing
4. CDN caching problems

---

## Technical Context

### shadcn/ui Registry System

The shadcn/ui registry is a JSON-based system for distributing components:

**Structure:**

```json
{
  "$schema": "https://ui.shadcn.com/schema/registry.json",
  "name": "shadcn",
  "homepage": "https://ui.shadcn.com",
  "items": [
    {
      "name": "native-select",
      "type": "registry:component",
      "description": "A styled native HTML select element",
      "files": [...]
    }
  ]
}
```

**How it works:**

1. CLI reads `components.json` in user's project
2. Fetches component definition from registry
3. Downloads component files and dependencies
4. Installs them in the project structure

---

## Additional Resources

### Documentation

- Native Select Component: https://ui.shadcn.com/docs/components/native-select
- Registry Documentation: https://ui.shadcn.com/docs/registry
- CLI Documentation: https://ui.shadcn.com/docs/cli

### Alternative Implementations

- **shadcn-vue:** Has native-select implementation
- **shadcn-svelte:** Has native-select implementation

Both Vue and Svelte ports have successfully implemented this component, indicating the concept is sound and the issue is specific to the React/main registry.

---

## Community Impact

### Developer Sentiment

The issue affects developers who specifically need:

- Native HTML behavior for accessibility
- Better mobile experience
- Simpler form controls
- Performance optimization

### Migration Path

For teams using custom select implementations who want to switch to native-select, this issue blocks that migration path.

---

## Monitoring & Updates

### How to Track This Issue

1. **GitHub Issue:** https://github.com/shadcn-ui/ui/issues/9162
2. **Watch for updates** on the issue page
3. **Check registry availability** periodically:
   ```bash
   curl https://ui.shadcn.com/r/styles/default/native-select.json
   ```

### Testing Resolution

Once fixed, verify with:

```bash
# Should work without errors
npx shadcn@latest add native-select
```

---

## Conclusion

The native-select component exists in shadcn/ui but cannot be installed via the CLI due to missing registry JSON files. This is a deployment/registry synchronization issue rather than a component implementation problem. Users can work around this by manually copying the component code, but the proper solution requires the maintainers to rebuild and deploy the registry files.

The issue has been properly reported and is awaiting maintainer action. Given the pattern of similar issues, this is likely to be resolved with a registry rebuild and deployment.

---

## Report Information

- **Documentation Created:** January 13, 2026
- **Issue Status at Time of Documentation:** Open
- **Last Verified:** January 13, 2026
- **Verification Method:** Direct URL check, CLI testing, documentation review
