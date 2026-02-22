# Codebase Structure

**Analysis Date:** 2026-02-22

## Directory Layout

```
csvExportHopeFuel/
├── src/                           # Source TypeScript/React code
│   ├── main.tsx                  # Entry point; mounts App to DOM
│   ├── App.tsx                   # Monolithic main component (573 lines)
│   ├── App.css                   # Legacy CSS (unused; Tailwind primary)
│   ├── index.css                 # Global styles (Tailwind imports)
│   ├── vite-env.d.ts             # Vite environment types
│   ├── components/
│   │   └── ui/                   # shadcn/ui primitive components
│   │       ├── button.tsx        # Button variant wrapper
│   │       ├── card.tsx          # Card, CardHeader, CardTitle, CardContent, CardDescription, CardFooter
│   │       ├── badge.tsx         # Badge variant wrapper
│   │       ├── input.tsx         # Input field
│   │       ├── label.tsx         # Form label
│   │       ├── progress.tsx      # Progress bar
│   │       ├── separator.tsx     # Visual separator
│   │       └── *.js              # Duplicate .js versions (legacy; not used)
│   ├── lib/
│   │   └── utils.ts              # Utility functions (cn class merger)
│   └── assets/
│       └── react.svg             # React logo
├── components/                    # Root components directory (legacy shadcn structure)
│   └── ui/                        # Mirrors src/components/ui
├── public/                        # Static assets directory (empty)
├── .planning/
│   └── codebase/                 # Generated codebase documentation (this directory)
├── index.html                     # HTML entry point
├── vite.config.ts                # Vite configuration with aliases
├── tsconfig.json                 # TypeScript configuration (ES2020, strict mode)
├── tsconfig.app.json             # App-specific TypeScript settings
├── tsconfig.node.json            # Node tools TypeScript settings
├── eslint.config.js              # ESLint configuration
├── tailwind.config.js            # Tailwind CSS configuration
├── postcss.config.js             # PostCSS configuration (for Tailwind)
├── components.json               # shadcn/ui CLI configuration
├── package.json                  # Project metadata and dependencies
├── package-lock.json             # Lock file for dependencies
├── .gitignore                    # Git ignore rules
└── README.md                      # Project README
```

## Directory Purposes

**src/:**
- Purpose: All TypeScript/React source code
- Contains: Main application component, UI primitives, utilities, global styles
- Key files: `App.tsx` (core logic), `main.tsx` (bootstrap), `index.css` (global styles)

**src/components/ui/:**
- Purpose: Reusable shadcn/ui component library
- Contains: Button, Card, Input, Label, Progress, Separator, Badge components
- Key files: Base exports used by App.tsx
- Note: Each component has both .tsx and .js versions (legacy)

**src/lib/:**
- Purpose: Shared utility functions
- Contains: `utils.ts` with `cn()` for Tailwind class merging using clsx + tailwind-merge
- Key files: `utils.ts`

**components/:**
- Purpose: Legacy shadcn/ui root directory (deprecated in favor of src/components/ui/)
- Contains: Duplicate component definitions
- Note: Not actively used; maintained for compatibility

**public/:**
- Purpose: Static assets served at root
- Contains: Empty in current setup
- Note: Would contain favicon, images, etc.

**.planning/codebase/:**
- Purpose: Generated codebase analysis documentation
- Contains: ARCHITECTURE.md, STRUCTURE.md, and other GSD analysis docs
- Generated: Yes (by GSD mapper)
- Committed: Yes (to git)

## Key File Locations

**Entry Points:**
- `index.html`: HTML entry point; defines `<div id="root">` and loads `/src/main.tsx`
- `src/main.tsx`: React bootstrap; mounts App component via ReactDOM.createRoot()
- `src/App.tsx`: Main React component; contains all UI and business logic

**Configuration:**
- `vite.config.ts`: Build configuration; defines `@` alias to `./src`, `@/components` alias to `./components`
- `tsconfig.json`: TypeScript compiler options; enables strict mode, JSX react-jsx, path aliases
- `tailwind.config.js`: Tailwind theme and plugin configuration
- `components.json`: shadcn/ui CLI config; specifies UI library, icon library (lucide), style (new-york)

**Core Logic:**
- `src/App.tsx`: Single-file monolith containing:
  - Type definitions (RowMsg, JobStatus, ProcessCounts)
  - Constants (INPUT_HEADERS, CODES, COUNTRY_MAP, limits)
  - Validator functions (email, currency, amount, date, month, etc.)
  - Helper functions (CSV generation, file download, chunking, naming)
  - React component with hooks (useState, useMemo)
  - Event handlers (startProcessing, acceptCsvFile, handleReset)
  - Render logic (layout, cards, tables, progress)
  - Test functions (self-tests for manual browser console execution)

**Utilities:**
- `src/lib/utils.ts`: `cn()` function for merging Tailwind classes
- `src/components/ui/*.tsx`: Reusable styled components (Button, Card, Input, etc.)

**Testing:**
- No test files in src/
- Self-tests defined in App.tsx (lines 534-572) as `__runMvpTests__()` function
- Exposed on window object for manual execution in browser console

## Naming Conventions

**Files:**
- Components: PascalCase (.tsx) → `Button.tsx`, `Card.tsx`
- Utilities: camelCase (.ts) → `utils.ts`
- Configuration: kebab-case or camelCase → `vite.config.ts`, `tailwind.config.js`
- Entry points: Explicit names → `main.tsx`, `index.html`, `App.tsx`

**Directories:**
- Component libraries: plural lowercase → `components`, `ui`
- Feature grouping: plural lowercase → `lib`, `assets`
- Internal structure: lowercase short names → `src`, `public`, `.planning`

**Functions/Variables:**
- Public functions: camelCase → `startProcessing()`, `acceptCsvFile()`, `normHeader()`
- Helper functions: camelCase with action prefix → `isValidEmail()`, `mapCountry()`, `buildNewCardId()`
- Constants: UPPER_SNAKE_CASE → `INPUT_HEADERS`, `COUNTRY_MAP`, `MAX_BYTES`
- Type names: PascalCase → `RowMsg`, `JobStatus`, `ProcessCounts`, `ExportRowNew`
- State variables: camelCase → `csvFile`, `startSeq`, `status`, `msgsErr`

**Paths/Imports:**
- Alias `@/` resolves to `./src/`
- Alias `@/components/` resolves to `./components/`
- UI components imported as: `import { Button } from "@/components/ui/button"`
- Utilities imported as: `import { cn } from "@/lib/utils"`

## Where to Add New Code

**New Feature (e.g., filtering, enrichment):**
- Primary code: Add functions/logic to `src/App.tsx` (alongside startProcessing and validation helpers)
- UI: Add new Card/Panel sections to render output in `src/App.tsx` JSX (after line 370)
- Tests: Add test cases to `__runMvpTests__()` self-test function (line 536+)

**New Component/Module:**
- If reusable across multiple features: `src/components/ui/{ComponentName}.tsx`
- Follow shadcn/ui pattern: export as named export, use cn() for class merging, forward refs for DOM components
- Add to `src/App.tsx` imports and use in render

**Utilities:**
- Math/formatting helpers: `src/lib/utils.ts`
- Domain-specific helpers (CSV, validation, etc.): Keep in `src/App.tsx` near App component
- Validation functions: Define in App.tsx alongside CODES constants (lines 64-77)

**UI Primitives:**
- Button variants, card layouts, input styles: Use shadcn/ui components from `src/components/ui/`
- Custom styling: Use Tailwind classes in className; merge with cn() if overriding base styles
- Theme colors: Defined in `tailwind.config.js`; reference in components via Tailwind color names

**Styling:**
- Global styles: `src/index.css` (Tailwind imports, CSS variables)
- Component-scoped styles: Use Tailwind className directly; no scoped CSS files
- Responsive breakpoints: `lg:` (1024px+) for grid changes; see line 367 for example
- Animations: Use framer-motion (already imported); see StageChip component (lines 488-506)

## Special Directories

**node_modules/:**
- Purpose: Installed npm dependencies
- Generated: Yes (by npm install)
- Committed: No (in .gitignore)
- Contains: 400+ packages including React, Vite, TypeScript, Tailwind, shadcn/ui, PapaParse, JSZip, file-saver

**dist/:**
- Purpose: Built production bundle
- Generated: Yes (by `npm run build`)
- Committed: No (in .gitignore)
- Created at: Root after build

**.git/:**
- Purpose: Git repository metadata
- Generated: Yes (by git init)
- Committed: N/A (system directory)

**.planning/:**
- Purpose: GSD planning and analysis output
- Generated: Yes (by GSD mapper and planner)
- Committed: Yes (to git)
- Contains: ARCHITECTURE.md, STRUCTURE.md, CONVENTIONS.md, TESTING.md, CONCERNS.md, etc.

---

*Structure analysis: 2026-02-22*
