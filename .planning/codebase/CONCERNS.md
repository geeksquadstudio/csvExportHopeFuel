# Codebase Concerns

**Analysis Date:** 2026-02-22

## Critical Issues

### Duplicate File Structure

**Issue:** Parallel component directories causing confusion and potential sync issues
- Files: `src/components/ui/*.{ts,tsx,js}` AND `components/ui/*.{ts,tsx,js}` (exact duplicates)
- Files: `src/App.tsx` AND `src/App.js` (JavaScript output alongside TypeScript source)
- Files: `src/main.tsx` AND `src/main.js` (JavaScript output alongside TypeScript source)
- Files: `src/lib/utils.ts` AND `src/lib/utils.js` (JavaScript output alongside TypeScript source)
- Impact: **Build confusion, IDE issues, maintenance burden.** Both tsconfig path alias `@/components/*` points to `./components/*` (root), not `./src/components/ui/*`. Components are imported from wrong location. Developers must maintain sync between two directories.
- Fix approach: Remove all `.js` files from source tree. Remove `/components` root directory entirely. Update tsconfig path alias to point to `./src/components/*` instead of `./components/*`. Ensure all imports use `@/` prefix pointing to `./src/*`.

### TypeScript Strict Mode with `as any` Casts

**Issue:** Multiple unsafe type assertions undermine strict mode benefits
- Files: `src/App.tsx` (lines 82, 331, 539-570)
- Pattern: `(saveAs as any)(blob, filename)`, `(blob as any).type`, `(window as any).__runMvpTests__`
- Impact: **Type safety gaps.** Compiler cannot catch errors in these sections. Particularly problematic for the file-saver integration where type mismatches hide API compatibility issues.
- Fix approach: Import and use proper type definitions. For file-saver, use `@types/file-saver` (already installed but not leveraged). For blob type check, use proper Blob instanceof check. For window extension, use interface module augmentation instead of casting.

### Missing Error Handling in Async Pipeline

**Issue:** CSV processing pipeline lacks error recovery and validation gaps
- Files: `src/App.tsx` (lines 167-340, especially readFileText and Papa.parse)
- Problems:
  - No try-catch wrapper around `readFileText()` call (line 174)
  - Papa.parse error checking only examines `parsed.errors[0]` but doesn't validate row structure before processing
  - No error handling for ZIP generation (`zip.generateAsync()` at line 329)
  - Download failures silently caught with empty try-catch (line 333)
- Impact: **Silent failures.** If file reading fails, ZIP generation fails, or download fails, users get stuck on "Complete" status without knowing what went wrong.
- Fix approach: Wrap readFileText in try-catch with proper error message. Add validation of Papa.parse row array structure. Wrap zip.generateAsync() with error boundary. Log/display download errors.

### Untested Manual Test Function

**Issue:** Self-test harness not integrated into automated testing
- Files: `src/App.tsx` (lines 534-572)
- Code: `__runMvpTests__()` function exposed to window, must be manually run in browser console
- Coverage: Only 5 test cases for a complex CSV validation pipeline
- Impact: **No CI/CD integration.** Validation, transformation, and filename generation logic has no regression protection. Tests never run during build. New developers don't know tests exist.
- Fix approach: Move tests to proper Jest/Vitest setup. Add npm scripts for automated testing. Increase coverage (at minimum: test all error codes, month validation edge cases, country mapping, duplicate detection, filename sequence logic).

### Hard-Coded Country Mapping

**Issue:** Country mapping is fixed to Myanmar and Thailand only
- Files: `src/App.tsx` (line 59)
- Code: `const COUNTRY_MAP: Record<string, string> = { Myanmar: "MM", Thailand: "TH" };`
- Limitation: Any other country unmapped defaults to "ZZ". No admin panel to add countries. No database lookup.
- Impact: **Not scalable.** Each new country requires code change and redeploy. Users in other countries cannot use the system properly.
- Fix approach: Move country mapping to external JSON config or database. Add admin UI for managing country codes. Load mapping at startup instead of compile time.

## Tech Debt

### Mixed File Extensions in Source Tree

**Issue:** Both TypeScript (.tsx/.ts) and JavaScript (.js) files coexist
- Files: All source files have parallel .js versions
- Cause: Build artifact left in source control instead of .gitignore
- Impact: **Confusing to developers.** Import resolution ambiguous. Git history bloated. Build consistency unclear.
- Fix approach: Add `*.js` to .gitignore immediately. Remove all .js files from source tree. Keep only source TypeScript files. JavaScript generated only during `npm run build` to `dist/`.

### Path Alias Misconfiguration

**Issue:** Incomplete and conflicting tsconfig path aliases
- Files: `tsconfig.json` (lines 13-15)
- Problem:
  - `@/*` maps to `./src/*` (correct)
  - `@/components/*` maps to `./components/*` (wrong - should be `./src/components/*`)
  - No alias for utilities, lib, or other directories
- Current imports work by accident because `@/components/*` finds files in root `/components/` directory (which is duplicated from src)
- Impact: **Maintainability issue.** Import paths don't match actual file structure. If root `/components/` is ever removed, all imports break. IDE autocomplete may be confused.
- Fix approach: Correct `@/components/*` to `["./src/components/*"]`. Remove root `/components/` directory. Test all imports still resolve.

### Missing Test Infrastructure

**Issue:** No automated testing framework configured despite having test file patterns in .gitignore
- Files: `.gitignore` references `*.spec.ts` and `*.test.ts` but no testing setup
- Config: No jest.config.js, vitest.config.ts, or test runner scripts
- Impact: **Zero test coverage for regression protection.** Manual browser console tests are not scalable. No CI/CD testing gate.
- Fix approach: Install and configure Vitest (lightweight, works with Vite). Create test files alongside source. Add `npm run test` and `npm run test:watch` scripts. Configure coverage thresholds.

### Unsafe Empty Catch Blocks

**Issue:** Multiple empty catch blocks hide errors
- Files: `src/App.tsx` (lines 82, 333, 570)
- Pattern: `try { ... } catch {}` or `catch (e: any) { ... }` with no error handling
- Impact: **Silent failures.** Download errors, file system errors, type errors in test helpers don't surface to user or console.
- Fix approach: Log all caught errors to console.error() at minimum. Better: display user-facing error messages for file operations. For test helpers, throw the error or return failure status.

### Validation Logic Scattered in Single Component

**Issue:** All CSV parsing, validation, transformation, and filename generation in one 500+ line component
- Files: `src/App.tsx` (all business logic mixed with React state management and UI)
- Symptoms: Hard to test, hard to reuse, hard to reason about data flow
- Impact: **Difficult to maintain and extend.** Future features (batch processing, streaming, webhooks) require refactoring.
- Fix approach: Extract validation to `src/lib/validation.ts`. Extract transformation to `src/lib/transform.ts`. Extract filename generation to `src/lib/filenames.ts`. Leave App.tsx for UI state and orchestration only. This enables separate testing and reuse.

### No Input Sanitization

**Issue:** CSV data passed through Papa.parse with minimal pre-validation
- Files: `src/App.tsx` (lines 174-186)
- Risk: Papa.parse configured with `skipEmptyLines: true` only. No validation of malformed CSV before parsing. Large files could cause memory issues.
- Impact: **Potential for DoS or unexpected behavior.** Malformed but valid CSV could cause parsing errors that aren't gracefully handled.
- Fix approach: Add file header sniffing before parsing. Validate CSV structure in first pass (row count, column consistency) before full parse. Add memory usage checks.

## Performance Bottlenecks

### Synchronous DOM Manipulation During Processing

**Issue:** File download attempted immediately after ZIP generation
- Files: `src/App.tsx` (lines 329-333)
- Problem: `downloadBlob()` triggers immediately after ZIP blob created. Browser may not have finished rendering progress updates. Large files (25MB) could freeze UI during download initiation.
- Impact: **Slow perceived performance.** UI feels janky during final stages.
- Fix approach: Defer downloadBlob to microtask (Promise.resolve().then()). Add explicit UI update for "Download complete" state. Consider background worker for ZIP generation to keep UI responsive.

### Large State Object in Single Component

**Issue:** Component state holds entire processed dataset in memory
- Files: `src/App.tsx` (lines 126-138)
- Data: `msgsErr`, `msgsWarn` arrays can grow to 50,000+ items. `summary` contains all file names.
- Impact: **Memory bloat with large files.** 50,000 errors in state array causes re-render overhead. Browser becomes sluggish.
- Fix approach: Paginate error/warning tables. Keep only last 1000 items in state. Stream processing results instead of accumulating.

### Regex Compilation on Every Call

**Issue:** Validation regex patterns compiled on each field validation
- Files: `src/App.tsx` (lines 68-76, 115-116)
- Pattern: `isValidEmail`, `isValidCurrency`, etc. create new RegExp on each call during tight loops
- Impact: **Unnecessary CPU during transformation loop.** 50,000 rows × 10 validations = 500,000 regex compilations.
- Fix approach: Pre-compile all regex patterns as constants (already done) but verify they're used as constant references, not recreated.

## Security Considerations

### Client-Side Only Validation

**Issue:** All validation happens in browser, no server-side verification
- Files: `src/App.tsx` (entire validation pipeline)
- Problem: User can disable JavaScript, modify CSV in-memory, or patch network requests
- Impact: **Data integrity at risk.** CSV could be modified before reaching backend (if one exists). Counts and validations cannot be trusted.
- Fix approach: Document that client-side validation is for UX only. Implement server-side validation before importing. Add checksum/signature verification if data is sent to backend.

### No CORS Headers or Rate Limiting

**Issue:** No server infrastructure mentioned; file processed entirely in browser
- Files: All processing in `src/App.tsx`
- Implication: If API endpoints added in future, no default CORS or rate limiting. Users could be exploited.
- Fix approach: If backend added, implement CORS policy, request rate limiting, file size limits server-side. Document security requirements in README.

### Sensitive Data in Manifest

**Issue:** Manifest JSON in ZIP contains counts but not data values; however, file contains PII
- Files: `src/App.tsx` (lines 318-327, 327)
- Data: Exported CSV files contain Name, Email, Country - all personal identifiable information
- Concern: ZIP downloaded to user's machine but no guidance on secure storage or deletion
- Fix approach: Add warning in UI about secure storage. Consider encryption option for ZIP. Document data retention policy.

## Fragile Areas

### File-Saver Fallback Implementation

**Issue:** Unsafe type casting and incomplete fallback logic for download
- Files: `src/App.tsx` (lines 81-88)
- Problem:
  - Type casting `(saveAs as any)(blob, filename)` hides API mismatches
  - Fallback creates <a> element and appends to document.body (could cause layout shift)
  - setTimeout cleanup assumes document still intact (could fail in SPA navigation)
  - openFallback feature opens blob URL in new tab but URL revoked after 30s (race condition)
- Impact: **Download could fail silently or partially.** Users may not get files in some browsers.
- Safe modification:
  1. Remove `as any` cast - use proper file-saver types
  2. Create detached <a> element instead of appending to body
  3. Use Promise-based cleanup instead of setTimeout
  4. Test thoroughly in different browsers (Safari, Firefox, Chrome, Edge)
  5. Add user feedback for successful download

### Duplicate Detection via Concatenation

**Issue:** Duplicate detection uses manual string concatenation
- Files: `src/App.tsx` (line 76)
- Code: `const dupKeyOf = (arr: string[]) => arr.map((v) => (v ?? "").trim()).join("\u001F");`
- Risk: If data contains the Unicode separator character (U+001F), false duplicates detected. Custom separator is non-printable but still possible in data.
- Impact: **Data loss.** Valid rows marked as duplicates and dropped silently.
- Safe modification: Use JSON.stringify() with sorted keys instead, or use crypto.subtle.digest() for hash-based deduplication.

### Filename Generation Sequence Overflow

**Issue:** Sequence number generation doesn't validate against maximum integer
- Files: `src/App.tsx` (lines 99-107)
- Code: `let n = parseInt(startSeq, 10); return () => String(n++).padStart(width, "0");`
- Risk: If startSeq = "999999999999999999" and user adds more files, integer overflow wraps to negative
- Impact: **Corrupted filename sequences.** Files generated with invalid names.
- Safe modification: Add validation that parsed integer is within safe JavaScript number range. Reject startSeq > Number.MAX_SAFE_INTEGER.

### Month Validation Edge Case

**Issue:** Month validation accepts 1-12 but doesn't prevent invalid dates
- Files: `src/App.tsx` (lines 74, 256)
- Code: `isValidMonth = (s: string) => /^(?:[1-9]|1[0-2])$/.test(normalizeMonth(s));`
- Caveat: No check if the month is valid for the transaction date. User can provide Month=2, TransactionDate=2026-02-30.
- Impact: **Downstream system may reject data.** Inconsistent state in output CSV.
- Fix approach: Validate month against transaction date month, or document that month field is independent reference only.

## Missing Critical Features

### No Audit Trail

**Issue:** No logging of processed data for compliance
- Problem: Each job processes member data but no record kept of what was processed, who processed it, or when
- Blocks: Compliance requirements, debugging user issues, security investigations
- Fix approach: Add server-side audit logging. Log job IDs, file hashes, counts, user (if auth added). Retain for X days.

### No Rate Limiting on Uploads

**Issue:** User can upload 25MB files repeatedly without throttle
- Problem: No quota per user. Could be exploited to consume server resources (if backend added).
- Blocks: Production deployment in multi-user scenario
- Fix approach: Add upload quota (files per hour/day). Add throttle to file processing. Document limits clearly to users.

### No Data Export Format Flexibility

**Issue:** Output format fixed to CSV only
- Problem: No option for Excel, JSON, or other formats. Splitting fixed at 300 rows, can't adjust.
- Blocks: Use cases needing different output formats or chunk sizes
- Fix approach: Add settings panel for output format selection. Make chunk size configurable.

## Test Coverage Gaps

### Zero Automated Tests

**Issue:** No test framework integration despite complex validation logic
- What's not tested:
  - Email validation (edge cases: +, -, numbers, special TLDs)
  - Amount validation (decimal precision, very large numbers)
  - Currency code validation (non-ASCII input)
  - Date parsing (non-ISO formats, leap years, timezone handling)
  - Card ID transformations (padding logic, length > 7 handling)
  - Country mapping (unmapped countries)
  - Duplicate detection (edge cases with separator character)
  - File chunk splitting (boundary conditions)
  - Filename sequence generation (overflow, padding)
  - Header reordering logic
- Files: `src/App.tsx` (all validation functions)
- Risk: **Regressions when modifying validation.** Bugs in edge cases discovered only in production.
- Priority: **High** - Core business logic with no regression protection

### No Integration Tests

**Issue:** No end-to-end CSV processing test
- Problem: Can't verify entire pipeline (read → validate → transform → split → package → download) works
- Risk: Changes to one stage may break downstream stages silently
- Fix approach: Add test with sample CSV file. Verify output ZIP structure and manifest. Check all files generated with correct headers.

### No UI Component Tests

**Issue:** React components (`StageChip`, `Stat`, `MsgTable`) untested
- Problem: Component props could break without notice. Styling regression invisible.
- Fix approach: Add Vitest with @testing-library/react. Test component rendering and interactions.

---

*Concerns audit: 2026-02-22*
