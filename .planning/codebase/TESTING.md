# Testing Patterns

**Analysis Date:** 2026-02-22

## Test Framework

**Status:** No external test framework configured

**Current Situation:**
- Zero `.test.*` or `.spec.*` files found in codebase
- No Jest, Vitest, or similar test runner configured in `package.json`
- No test-related dependencies present

**Self-Test Implementation:**
- Manual browser-console tests via `__runMvpTests__()` function (see `src/App.tsx` lines 534-572)
- Function attached to `window` object for accessibility: `window.__runMvpTests__()`

## Manual Test Implementation

**Location:** `src/App.tsx` (lines 534-572)

**Structure:**
```typescript
function __runMvpTests__() {
  const results: { name: string; ok: boolean; detail?: string }[] = [];
  const t = (name: string, fn: () => void) => {
    try { fn(); results.push({ name, ok: true }); }
    catch (e: any) { results.push({ name, ok: false, detail: e?.message }); }
  };
  const eq = (a: any, b: any) => {
    if (JSON.stringify(a) !== JSON.stringify(b)) throw new Error(...);
  };
  const ok = (v: any) => { if (!v) throw new Error(...); };
  const no = (v: any) => { if (v) throw new Error(...); };

  // Test cases using t()
  // ...

  console.table(results);
  const passed = results.filter(r => r.ok).length;
  const failed = results.length - passed;
  console.log(`MVP tests: ${passed} passed, ${failed} failed`);
  return { passed, failed, results };
}
```

**Assertion Helpers:**
- `t(name, fn)` - Test case wrapper with try-catch error handling
- `eq(a, b)` - Equality assertion using JSON.stringify comparison
- `ok(v)` - Truthy assertion
- `no(v)` - Falsy assertion

**Run Command:**
```javascript
// In browser console:
window.__runMvpTests__()

// Output:
// - console.table(results) - Renders results as table
// - console.log() - Summary with passed/failed counts
// - Returns object: { passed: number, failed: number, results: array }
```

## Test Coverage

**Currently Tested Functions (in `__runMvpTests__()`):**
- Month validation (`isValidMonth()`)
- CSV filename validation (`isCsvFileName()`)
- Filename generation (`buildFileNames()`)

**Example Test Cases:**
```typescript
t("month: accepts 12", () => ok(isValidMonth("12")));
t("month: rejects 13", () => no(isValidMonth("13")));

t("csv filename ok", () => ok(isCsvFileName("data.csv")));
t("csv filename bad", () => no(isCsvFileName("data.xlsx")));

t("filenames: single new + single old", () => {
  const { newNames, oldNames } = buildFileNames(1, 1, "098", "20250830");
  eq(newNames, ["098_prf_bulk_import_20250830.csv"]);
  eq(oldNames, ["099_extension_prf_bulk_import_20250830.csv"]);
});
```

**Gaps:**
- Email validation (`isValidEmail()`) - not tested
- Amount validation (`isValidAmount()`) - not tested
- Currency validation (`isValidCurrency()`) - not tested
- Date validation (`isValidISODate()`) - not tested
- Card ID parsing (`buildNewCardId()`) - not tested
- Complex CSV transformation logic - not tested
- Component rendering - not tested
- State management and effects - not tested
- Error handling flows - not tested
- File reading and parsing - not tested

## Validation Logic (Primary Testing Subject)

The codebase relies heavily on synchronous validation helper functions. These are the critical paths that should be tested if a formal test framework is added:

**Core Validation Functions in `src/App.tsx`:**
- `normHeader()` - Line 66: Normalize column headers
- `isValidEmail()` - Line 68: Email format validation
- `isValidCurrency()` - Line 70: ISO 4217 currency code validation
- `isValidAmount()` - Line 71: Monetary amount format validation
- `isValidISODate()` - Line 72: ISO date string validation
- `normalizeMonth()` - Line 73: Month string normalization
- `isValidMonth()` - Line 74: Month range validation (1-12)
- `mapCountry()` - Line 77: Country name to ISO 3166-1 alpha-2 mapping
- `buildNewCardId()` - Line 78: Card ID formatting with length check

## Testing Patterns (When Implemented)

**What to Test:**
1. Validation functions - All should have unit test coverage
2. CSV parsing and transformation logic - Integration tests
3. File generation (names, headers, content) - Unit tests
4. Error message generation and codes - Unit tests
5. Component behavior (e.g., drag-drop acceptance, progress states) - Component tests

**What NOT to Mock:**
- Validation functions themselves (test them as-is)
- Error code constants (test against actual constants)
- CSV formatting logic (PapaParse integration)
- Utility functions like `normHeader`, `mapCountry`

**What to Mock:**
- File system operations (`FileReader`, `Blob` creation)
- Browser APIs (`window.open`, `URL.createObjectURL`)
- External libraries when testing component behavior (optional)
- DOM manipulation for component tests

## Browser Environment

**Self-Test Access:**
```javascript
// Window attachment (from src/App.tsx lines 569-572):
if (typeof window !== "undefined") {
  (window as any).__runMvpTests__ = __runMvpTests__;
}
```

**Run via:**
1. Open application in browser
2. Open Developer Console (F12 or Cmd+Option+J)
3. Execute: `window.__runMvpTests__()`
4. View results in console table

## Future Test Setup Recommendations

**If Adding Vitest (lightweight for React components):**
```bash
npm install -D vitest @vitest/ui @testing-library/react @testing-library/jest-dom
```

**If Adding Jest (more comprehensive):**
```bash
npm install -D jest @testing-library/react @testing-library/jest-dom @types/jest ts-jest
```

**Config Location:** Would typically be `vitest.config.ts` or `jest.config.ts`

**Test File Organization:** Would follow pattern of `src/**/*.test.tsx` for components and `src/**/*.test.ts` for utilities

---

*Testing analysis: 2026-02-22*
