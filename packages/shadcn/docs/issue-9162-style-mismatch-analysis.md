# Issue #9162: Style Mismatch Causing 404 Errors for New Components

## Executive Summary

Users with `"style": "default"` or `"style": "new-york"` in their `components.json` receive confusing 404 errors when trying to add newer components like `native-select`. The root cause is that these components only exist in the `new-york-v4` style (Tailwind v4), but the CLI doesn't detect this mismatch and provides a generic "not found" error.

**This is NOT a registry deployment issue** - the component exists, just at a different style path.

---

## Root Cause Analysis

### The Problem

1. **Style Deprecation**: The `default` style has been officially deprecated. New projects use `new-york` or `new-york-v4`.

2. **New Components Only in v4**: Components added after the Tailwind v4 migration (like `native-select`) only exist in `new-york-v4`:
   ```
   ✅ https://ui.shadcn.com/r/styles/new-york-v4/native-select.json  → 200 OK
   ❌ https://ui.shadcn.com/r/styles/new-york/native-select.json     → 404
   ❌ https://ui.shadcn.com/r/styles/default/native-select.json      → 404
   ```

3. **Legacy Projects**: Users who initialized their projects before the v4 migration have `"style": "default"` or `"style": "new-york"` in their `components.json`.

4. **URL Construction**: The CLI builds URLs using the style from config:
   ```
   https://ui.shadcn.com/r/styles/{style}/{name}.json
   ```

### Code Flow

```
User runs: npx shadcn add native-select
                    ↓
CLI reads components.json → style: "default"
                    ↓
buildUrlFromRegistryConfig() constructs URL:
  https://ui.shadcn.com/r/styles/default/native-select.json
                    ↓
fetchRegistry() → 404 Not Found
                    ↓
RegistryNotFoundError thrown with generic message
```

---

## Affected Files in CLI Codebase

### 1. `src/registry/constants.ts`
```typescript
export const FALLBACK_STYLE = "new-york-v4"

export const BUILTIN_REGISTRIES: z.infer<typeof registryConfigSchema> = {
  "@shadcn": `${REGISTRY_URL}/styles/{style}/{name}.json`,
}
```
- The `{style}` placeholder is replaced with the user's configured style
- `FALLBACK_STYLE` is only used when no style is configured

### 2. `src/registry/builder.ts`
```typescript
export function buildUrlFromRegistryConfig(
  item: string,
  registryConfig: z.infer<typeof registryConfigItemSchema>,
  config?: Config
) {
  // ...
  if (config?.style && url.includes(STYLE_PLACEHOLDER)) {
    url = url.replace(STYLE_PLACEHOLDER, config.style)
  }
  // ...
}
```
- Directly uses `config.style` without validation
- No check if the style actually has the requested component

### 3. `src/registry/errors.ts`
```typescript
export class RegistryNotFoundError extends RegistryError {
  constructor(public readonly url: string, cause?: unknown) {
    const message = `The item at ${url} was not found. It may not exist at the registry.`
    // ...
    suggestion: "Check if the item name is correct and the registry URL is accessible.",
  }
}
```
- Generic error message doesn't help users understand the style mismatch
- No context about deprecated styles or v4-only components

### 4. `src/registry/config.ts`
```typescript
function resolveStyleFromConfig(config: DeepPartial<Config>) {
  if (!config.style) {
    return FALLBACK_STYLE
  }

  // Check if we should use new-york-v4 for Tailwind v4.
  // We assume that if tailwind.config is empty, we're using Tailwind v4.
  if (config.style === "new-york" && config.tailwind?.config === "") {
    return FALLBACK_STYLE
  }

  return config.style
}
```
- Only upgrades `new-york` → `new-york-v4` if `tailwind.config` is empty
- Doesn't handle `default` style at all
- Doesn't handle cases where user has Tailwind v4 but old style config

### 5. `src/registry/fetcher.ts`
```typescript
if (response.status === 404) {
  throw new RegistryNotFoundError(url, messageFromServer)
}
```
- Throws generic 404 error without additional context
- No attempt to check alternative styles

---

## Current Error Message (Poor UX)

```
Something went wrong. Please check the error below for more details.
If the problem persists, please open an issue on GitHub.

Message:
The item at https://ui.shadcn.com/r/styles/default/native-select.json was not found. 
It may not exist at the registry.

Suggestion:
Check if the item name is correct and the registry URL is accessible.
```

**Problems:**
1. User doesn't know the component exists in a different style
2. No mention of style deprecation
3. No actionable fix provided
4. Suggests the component might not exist (it does!)

---

## Proposed Solutions

### Solution 1: Improved Error Detection and Messaging (Recommended)

**Location:** `src/registry/fetcher.ts` and `src/registry/errors.ts`

When a 404 occurs, check if the component exists in `new-york-v4`:

```typescript
// In fetcher.ts - after catching 404
if (response.status === 404) {
  // Check if this might be a style mismatch issue
  const styleMismatchInfo = await checkStyleMismatch(url, config)
  if (styleMismatchInfo.existsInV4) {
    throw new RegistryStyleMismatchError(url, config.style, styleMismatchInfo)
  }
  throw new RegistryNotFoundError(url, messageFromServer)
}
```

**New Error Class:**
```typescript
export class RegistryStyleMismatchError extends RegistryError {
  constructor(
    public readonly url: string,
    public readonly currentStyle: string,
    public readonly info: { existsInV4: boolean; availableStyles: string[] }
  ) {
    const message = `The component was not found for style "${currentStyle}".`
    
    super(message, {
      code: RegistryErrorCode.NOT_FOUND,
      statusCode: 404,
      context: { url, currentStyle, ...info },
      suggestion: this.buildSuggestion(currentStyle, info),
    })
    this.name = "RegistryStyleMismatchError"
  }

  private buildSuggestion(style: string, info: StyleMismatchInfo): string {
    const suggestions: string[] = []
    
    if (style === "default") {
      suggestions.push(
        'The "default" style has been deprecated.',
        'Update your components.json to use "style": "new-york" or "style": "new-york-v4".'
      )
    }
    
    if (info.existsInV4) {
      suggestions.push(
        'This component is available in the "new-york-v4" style (Tailwind v4).',
        'To use it, either:',
        '  1. Migrate your project to Tailwind v4 and update style to "new-york-v4"',
        '  2. Manually copy the component from https://ui.shadcn.com/docs/components/'
      )
    }
    
    return suggestions.join('\n')
  }
}
```

**Improved Error Output:**
```
Something went wrong. Please check the error below for more details.

Message:
The component was not found for style "default".

Suggestion:
The "default" style has been deprecated.
Update your components.json to use "style": "new-york" or "style": "new-york-v4".

This component is available in the "new-york-v4" style (Tailwind v4).
To use it, either:
  1. Migrate your project to Tailwind v4 and update style to "new-york-v4"
  2. Manually copy the component from https://ui.shadcn.com/docs/components/native-select
```

---

### Solution 2: Automatic Style Fallback with Warning

**Location:** `src/registry/config.ts`

```typescript
const DEPRECATED_STYLES = ["default"]
const V4_ONLY_COMPONENTS = ["native-select", /* add others as discovered */]

function resolveStyleFromConfig(config: DeepPartial<Config>) {
  if (!config.style) {
    return FALLBACK_STYLE
  }

  // Warn about deprecated styles
  if (DEPRECATED_STYLES.includes(config.style)) {
    logger.warn(
      `The "${config.style}" style is deprecated. ` +
      `Consider updating to "new-york" or "new-york-v4".`
    )
  }

  // Auto-upgrade for Tailwind v4 detection
  if (config.style === "new-york" && config.tailwind?.config === "") {
    return FALLBACK_STYLE
  }

  return config.style
}
```

---

### Solution 3: Pre-fetch Validation

**Location:** `src/registry/resolver.ts` or new `src/registry/validator.ts`

Before fetching, validate that the component exists for the configured style:

```typescript
async function validateComponentAvailability(
  componentName: string,
  style: string
): Promise<ValidationResult> {
  const stylesToCheck = [style]
  
  // If using legacy style, also check v4
  if (style === "default" || style === "new-york") {
    stylesToCheck.push("new-york-v4")
  }
  
  const results = await Promise.all(
    stylesToCheck.map(async (s) => {
      const url = `${REGISTRY_URL}/styles/${s}/${componentName}.json`
      const response = await fetch(url, { method: "HEAD" })
      return { style: s, exists: response.ok }
    })
  )
  
  return {
    requestedStyle: style,
    existsInRequested: results.find(r => r.style === style)?.exists ?? false,
    availableIn: results.filter(r => r.exists).map(r => r.style),
  }
}
```

---

### Solution 4: Add Deprecation Constants

**Location:** `src/registry/constants.ts`

```typescript
export const DEPRECATED_STYLES = {
  default: {
    deprecatedSince: "2025-03",
    replacement: "new-york",
    message: 'The "default" style has been deprecated. Use "new-york" instead.',
  },
} as const

export const STYLE_REQUIREMENTS = {
  "new-york-v4": {
    tailwindVersion: "v4",
    description: "Tailwind CSS v4 style with OKLCH colors",
  },
  "new-york": {
    tailwindVersion: "v3",
    description: "Tailwind CSS v3 style with HSL colors",
  },
} as const
```

---

## Implementation Priority

| Priority | Solution | Effort | Impact |
|----------|----------|--------|--------|
| 1 | Solution 1: Better error messages | Medium | High |
| 2 | Solution 4: Deprecation constants | Low | Medium |
| 3 | Solution 2: Auto-fallback with warning | Low | Medium |
| 4 | Solution 3: Pre-fetch validation | High | High |

---

## Affected Components (Known v4-Only)

Components that only exist in `new-york-v4`:
- `native-select`
- (More may be added - need to audit registry)

Components that exist in all styles:
- `button`, `input`, `select`, `card`, etc. (legacy components)

---

## Testing Recommendations

1. **Unit Tests:**
   - Test `RegistryStyleMismatchError` message generation
   - Test style resolution logic with deprecated styles
   - Test fallback behavior

2. **Integration Tests:**
   - Test `add` command with deprecated style + v4-only component
   - Test error message output formatting
   - Test with various `components.json` configurations

3. **Manual Testing:**
   ```bash
   # Create test project with deprecated style
   echo '{"style":"default",...}' > components.json
   npx shadcn add native-select
   # Should show helpful error message
   ```

---

## Related Issues

- Issue #9162: Unable to Add Native-Select Component
- Issue #6856: Update theme for tailwind v4
- Issue #2570: Switch to newyork

---

## References

- [Tailwind v4 Migration Guide](https://ui.shadcn.com/docs/tailwind-v4)
- [components.json Documentation](https://ui.shadcn.com/docs/components-json)
- [Style Deprecation Notice](https://ui.shadcn.com/docs/components-json#style)

---

## Document Info

- **Created:** January 13, 2026
- **Author:** CLI Analysis
- **Status:** Ready for Review
- **Related PR:** TBD


---

## Appendix A: Quick Reference - Files to Modify

### For Improved Error Messages

```
src/registry/errors.ts
├── Add: RegistryStyleMismatchError class
├── Add: StyleMismatchInfo type
└── Update: Error code enum

src/registry/fetcher.ts
├── Update: fetchRegistry() to detect style mismatch on 404
└── Add: checkStyleMismatch() helper function

src/registry/constants.ts
├── Add: DEPRECATED_STYLES constant
├── Add: V4_ONLY_COMPONENTS list (optional)
└── Add: STYLE_REQUIREMENTS constant
```

### For Style Resolution Improvements

```
src/registry/config.ts
├── Update: resolveStyleFromConfig() to warn on deprecated styles
└── Add: isDeprecatedStyle() helper

src/utils/handle-error.ts
├── Update: handleError() to format RegistryStyleMismatchError nicely
└── Add: Special formatting for style-related errors
```

---

## Appendix B: Verification Commands

```bash
# Check if component exists in different styles
curl -s -o /dev/null -w "%{http_code}" "https://ui.shadcn.com/r/styles/default/native-select.json"
# Expected: 404

curl -s -o /dev/null -w "%{http_code}" "https://ui.shadcn.com/r/styles/new-york/native-select.json"
# Expected: 404

curl -s -o /dev/null -w "%{http_code}" "https://ui.shadcn.com/r/styles/new-york-v4/native-select.json"
# Expected: 200

# Check available styles
curl -s "https://ui.shadcn.com/r/styles/index.json"
# Returns: [{"name":"new-york","label":"New York"},{"name":"default","label":"Default"}]

# Check main index (v3 components only)
curl -s "https://ui.shadcn.com/r/index.json" | grep -c "native-select"
# Expected: 0 (not in v3 index)
```

---

## Appendix C: Sample Code Changes

### New Error Class (src/registry/errors.ts)

```typescript
export interface StyleMismatchInfo {
  existsInV4: boolean
  availableStyles: string[]
  componentName: string
}

export class RegistryStyleMismatchError extends RegistryError {
  constructor(
    public readonly url: string,
    public readonly currentStyle: string,
    public readonly info: StyleMismatchInfo
  ) {
    const componentName = info.componentName
    const message = `Component "${componentName}" is not available for style "${currentStyle}".`

    let suggestion = ""
    
    if (currentStyle === "default") {
      suggestion = `The "default" style has been deprecated.\n`
    }
    
    if (info.existsInV4) {
      suggestion += `This component is only available in the "new-york-v4" style (Tailwind v4).\n\n`
      suggestion += `To resolve this:\n`
      suggestion += `  1. Migrate to Tailwind v4: https://ui.shadcn.com/docs/tailwind-v4\n`
      suggestion += `  2. Update components.json: "style": "new-york-v4"\n`
      suggestion += `  3. Or manually copy from: https://ui.shadcn.com/docs/components/${componentName}`
    } else if (info.availableStyles.length > 0) {
      suggestion += `Available in styles: ${info.availableStyles.join(", ")}`
    } else {
      suggestion += `This component may not exist in the registry.`
    }

    super(message, {
      code: RegistryErrorCode.NOT_FOUND,
      statusCode: 404,
      context: { url, currentStyle, ...info },
      suggestion,
    })
    this.name = "RegistryStyleMismatchError"
  }
}
```

### Style Mismatch Detection (src/registry/fetcher.ts)

```typescript
import { REGISTRY_URL } from "@/src/registry/constants"

async function checkStyleMismatch(
  originalUrl: string,
  config: Config
): Promise<StyleMismatchInfo | null> {
  // Extract component name from URL
  const match = originalUrl.match(/\/styles\/[^/]+\/([^.]+)\.json/)
  if (!match) return null
  
  const componentName = match[1]
  const currentStyle = config.style
  
  // Only check if using a potentially outdated style
  if (!["default", "new-york"].includes(currentStyle)) {
    return null
  }
  
  // Check if exists in new-york-v4
  const v4Url = `${REGISTRY_URL}/styles/new-york-v4/${componentName}.json`
  try {
    const response = await fetch(v4Url, { method: "HEAD", agent })
    if (response.ok) {
      return {
        existsInV4: true,
        availableStyles: ["new-york-v4"],
        componentName,
      }
    }
  } catch {
    // Ignore fetch errors
  }
  
  return null
}
```

### Updated Fetch Handler (src/registry/fetcher.ts)

```typescript
if (response.status === 404) {
  // Check for style mismatch before throwing generic error
  const styleMismatch = await checkStyleMismatch(url, config)
  if (styleMismatch) {
    throw new RegistryStyleMismatchError(
      url,
      config.style,
      styleMismatch
    )
  }
  throw new RegistryNotFoundError(url, messageFromServer)
}
```
