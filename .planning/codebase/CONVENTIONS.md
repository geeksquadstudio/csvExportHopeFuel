# Coding Conventions

**Analysis Date:** 2026-02-22

## Naming Patterns

**Files:**
- React components: PascalCase with `.tsx` extension (e.g., `App.tsx`, `Button.tsx`)
- UI components: lowercase with `.tsx` extension (e.g., `button.tsx`, `card.tsx`)
- Utility modules: lowercase with `.ts` extension (e.g., `utils.ts`)
- Type definition files: `*.d.ts` (e.g., `vite-env.d.ts`)

**Functions:**
- Component functions: PascalCase (e.g., `App`, `StageChip`, `MsgTable`)
- Helper functions: camelCase (e.g., `normHeader`, `isValidEmail`, `downloadBlob`)
- Private/internal functions: camelCase with leading underscore discouraged; use standard camelCase
- Utility functions exported as named exports or in utility modules (e.g., `cn`, `normalizeCurrency`)

**Variables:**
- State variables: camelCase (e.g., `csvFile`, `startSeq`, `msgsErr`)
- Constants: SCREAMING_SNAKE_CASE (e.g., `INPUT_HEADERS`, `MAX_BYTES`, `CODES`)
- Type/interface naming: PascalCase (e.g., `JobStatus`, `RowMsg`, `ProcessCounts`)

**Types:**
- Interfaces/Types: PascalCase (e.g., `ButtonProps`, `CardProps`)
- Union types: clear semantic naming (e.g., `JobStatus = "Idle" | "Validating" | ...`)
- Generics: Single uppercase letter or descriptive (e.g., `<T>`, `<ButtonProps>`)

## Code Style

**Formatting:**
- No explicit formatter configured (Prettier not present)
- Indentation: 2 spaces (observed in all files)
- Line length: No hard limit enforced, but generally kept reasonable
- Trailing semicolons: Present and required throughout (TypeScript/JavaScript standard)
- String quotes: Double quotes preferred in JSX attributes and string literals

**Linting:**
- ESLint config: `eslint.config.js` (Flat config format)
- Extends: `@eslint/js`, `typescript-eslint`, `eslint-plugin-react-hooks`, `eslint-plugin-react-refresh`
- Target: ES2020, browser globals
- Runs on all `**/*.{ts,tsx}` files
- Ignores: `dist/` directory

**Lint Command:**
```bash
npm run lint              # Run ESLint on entire codebase
```

## Import Organization

**Order:**
1. React and React ecosystem imports (e.g., `import React`, `import { useMemo }`)
2. Third-party library imports (e.g., `import Papa`, `import JSZip`, `import { motion }`)
3. External package imports (e.g., `import saveAs from "file-saver"`)
4. Icon/component library imports (e.g., `import { Upload, Download } from "lucide-react"`)
5. Internal imports from `@/` path aliases (e.g., `import { Card } from "@/components/ui/card"`)
6. Relative imports from utils/helpers (less common, prefer path aliases)

**Path Aliases:**
- `@/*` → `./src/*` (base src directory)
- `@/components/*` → `./components/*` (shadcn UI components)

Example from `src/App.tsx`:
```typescript
import React, { useMemo, useState } from "react";
import { motion, Variants } from "framer-motion";
import Papa from "papaparse";
import JSZip from "jszip";
import saveAs from "file-saver";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Upload, Download, FileWarning } from "lucide-react";
```

## Error Handling

**Patterns:**
- Explicit error codes using object constants (see `CODES` in `src/App.tsx`)
- Error/warning messages collected in arrays with structure: `{ line: number; code: string; message: string }`
- Validation uses guard clauses with early returns
- Try-catch blocks for file operations (e.g., `readFileText`, `downloadBlob`)
- Graceful fallbacks for file operations (e.g., manual blob download if `saveAs` fails in `downloadBlob`)

Example error code structure:
```typescript
const CODES = {
  ERR: {
    HEADERS_MISSING: "E-HEADERS-MISSING",
    EMAIL: "E-EMAIL-FORMAT",
    AMOUNT_NUM: "E-AMOUNT-NUM",
  },
  WARN: {
    HEADERS_REORDERED: "W-HEADERS-REORDERED",
    COUNTRY_UNMAPPED: "W-COUNTRY-UNMAPPED",
  },
} as const;
```

## Logging

**Framework:** Console-based (no structured logging library)

**Patterns:**
- `console.log()`, `console.table()` for debug output
- Self-test function `__runMvpTests__()` uses `console.table()` and `console.log()`
- Attached to window object for browser console access (e.g., `window.__runMvpTests__()`)
- No production logging observed

## Comments

**When to Comment:**
- File-level headers describing purpose (e.g., "File: src/App.tsx")
- Section dividers for logical grouping (e.g., "--- Filename helpers ---")
- Complex business logic (e.g., CSV validation rules)
- Non-obvious constraints or requirements

**JSDoc/TSDoc:**
- Not used in this codebase
- Type annotations handle documentation through TypeScript

Example comment style from `src/App.tsx`:
```typescript
// File: src/App.tsx
// Hope Fuel PRF Bulk Import & Member Categorization – MVP (React + TS)

// --- Filename helpers (PRD exact shapes — NO chunk suffix) ---
```

## Function Design

**Size:** Functions vary from inline helpers (1-2 lines) to larger async handlers (100+ lines)
- Utility helpers: 1-2 lines, single responsibility
- Validation functions: 1-5 lines each
- Complex async handlers: Organized with clear stage progression

**Parameters:**
- Typed with TypeScript (no implicit `any`)
- Destructuring for component props is standard
- Rest parameters used for spreading props (e.g., `...props`)

**Return Values:**
- Explicit return types for functions (TypeScript strict mode)
- Async functions return Promises
- Helper functions return primitives or objects with clear structure

Example function style:
```typescript
const isValidEmail = (s: string) => /^(?!.{255,})[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i.test(s.trim());

const normHeader = (h: string) => h.toLowerCase().replace(/\s+|_/g, "");

function downloadBlob(blob: Blob, filename: string, opts?: { openFallback?: boolean }) {
  try { (saveAs as any)(blob, filename); return; } catch {}
  // ... fallback logic
}
```

## Module Design

**Exports:**
- Named exports for components and utilities (e.g., `export { Button, buttonVariants }`)
- Default export for main App component (e.g., `export default function App()`)
- Barrel files used for UI components (multi-export pattern in `card.tsx`, `button.tsx`)

**Barrel Files:**
- `src/components/ui/` contains multiple component exports per file
- Example: `card.tsx` exports `Card`, `CardHeader`, `CardTitle`, `CardContent`, etc.

Example barrel pattern from `src/components/ui/card.tsx`:
```typescript
export { Card, CardHeader, CardFooter, CardTitle, CardDescription, CardContent }
```

## TypeScript Strict Mode

**Configuration:** `strict: true` in `tsconfig.json`

**Implications:**
- All variables must be explicitly typed or type-inferred
- No implicit `any` allowed
- Null/undefined checks required
- Type assertions used sparingly (e.g., `(saveAs as any)` for legacy library)

---

*Convention analysis: 2026-02-22*
