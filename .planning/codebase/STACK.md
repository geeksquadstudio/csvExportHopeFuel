# Technology Stack

**Analysis Date:** 2026-02-22

## Languages

**Primary:**
- TypeScript 5.8.3 - Main language for source code and configuration
- JavaScript - Build configuration and scripts

**Secondary:**
- CSS (via Tailwind) - Styling and layout

## Runtime

**Environment:**
- Node.js - Required for development and build processes

**Package Manager:**
- npm - Dependency management
- Lockfile: `package-lock.json` (present)

## Frameworks

**Core:**
- React 19.1.1 - UI framework and component library
- React DOM 19.1.1 - DOM rendering for React components

**Build/Dev:**
- Vite 7.1.2 - Build tool and dev server with React plugin (`@vitejs/plugin-react` 5.0.0)
- TypeScript Compiler (tsc) - Type checking and transpilation (run via `tsc -b` in build pipeline)

**Styling:**
- Tailwind CSS 3.4.17 - Utility-first CSS framework
- PostCSS 8.5.6 - CSS transformation pipeline
- Autoprefixer 10.4.21 - Automatic vendor prefix injection
- tailwindcss-animate 1.0.7 - Animation utilities for Tailwind

**UI Components:**
- shadcn/ui 3.0.0 - Pre-built, customizable component library (headless)
- Radix UI (@radix-ui/react-*) - Unstyled, accessible primitives:
  - `@radix-ui/react-label` 2.1.7 - Label component
  - `@radix-ui/react-progress` 1.1.7 - Progress indicator
  - `@radix-ui/react-separator` 1.1.7 - Separator/divider
  - `@radix-ui/react-slot` 1.2.3 - Slot composition pattern

**Animation:**
- Framer Motion 12.23.12 - React animation library for motion effects and stage transitions

## Key Dependencies

**Critical:**
- PapaParse 5.5.3 - CSV parsing and generation for bulk import processing
- JSZip 3.10.1 - ZIP file creation for packaging processed CSV outputs
- file-saver 2.0.5 - Browser file download functionality for CSV and ZIP exports
- lucide-react 0.542.0 - Icon library integrated with React
- clsx 2.1.1 - Conditional CSS class name utilities
- class-variance-authority 0.7.1 - Component variant management
- tailwind-merge 3.3.1 - Tailwind CSS class merging and conflict resolution

**Infrastructure:**
- Git 0.1.5 - Version control integration (minor utility)

## Configuration

**Environment:**
- No environment configuration files detected (dev-only, no API keys or external services required)
- All configuration hardcoded in source code (country mappings, validation rules, file limits)
- Build targets: ES2020 with JSX React transform

**Build:**
- `vite.config.ts` - Vite configuration with:
  - React plugin enabled
  - Base path: `/csvHopeFuel` (for GitHub Pages deployment)
  - Path aliases: `@` → `./src`, `@/components` → `./components`
- `tsconfig.json` - TypeScript configuration:
  - Target: ES2020
  - Module: ESNext
  - Module resolution: Bundler
  - Strict mode enabled
  - Path aliases configured for imports
- `tsconfig.app.json` - Application TypeScript config (extends base)
- `tsconfig.node.json` - Build configuration TypeScript config
- `tailwind.config.js` - Tailwind CSS customization with:
  - Dark mode via class selector
  - Custom animations (accordion-up, accordion-down)
  - Custom theme variables for colors and border radius
- `postcss.config.js` - PostCSS plugins (Tailwind, Autoprefixer)
- `eslint.config.js` - ESLint configuration:
  - Base JS recommended rules
  - TypeScript ESLint rules
  - React Hooks linting
  - React Refresh plugin for HMR
  - Global ignore: `dist/`
- `components.json` - shadcn/ui configuration:
  - Style preset: "new-york"
  - Icon library: "lucide"
  - TSX output
  - CSS variable theming

## Scripts

**Development:**
- `npm run dev` - Start Vite dev server with HMR

**Production:**
- `npm run build` - Compile TypeScript and build with Vite
- `npm run preview` - Preview production build locally
- `npm run predeploy` - Pre-deployment hook (runs build)
- `npm run deploy` - Deploy to GitHub Pages via gh-pages

**Code Quality:**
- `npm run lint` - Run ESLint on all files

## Platform Requirements

**Development:**
- Node.js with npm
- Modern browser with ES2020 support
- 25MB+ available memory (for large CSV processing)

**Production:**
- Deployment target: GitHub Pages
- Base URL: `https://[username].github.io/csvHopeFuel`
- Static hosting only (no backend required)

---

*Stack analysis: 2026-02-22*
