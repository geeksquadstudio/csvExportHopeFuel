# Architecture

**Analysis Date:** 2026-02-22

## Pattern Overview

**Overall:** Single-file React application with integrated data processing pipeline

**Key Characteristics:**
- Monolithic component (`App.tsx`) containing all business logic, UI rendering, and state management
- Linear processing pipeline: File validation → Data transformation → Chunking → Naming → ZIP packaging
- Client-side only, no backend dependencies
- Real-time stage-based progress tracking with visual feedback

## Layers

**Presentation Layer:**
- Purpose: Render UI components and handle user interactions
- Location: `src/App.tsx` (lines 353-486), `src/components/ui/*`
- Contains: React components using shadcn/ui primitives (Card, Button, Input, Badge, Progress, Label, Separator)
- Depends on: State hooks (useState, useMemo), framer-motion for animations, lucide-react icons
- Used by: Browser DOM rendering

**Business Logic Layer:**
- Purpose: Orchestrate CSV processing pipeline and manage job state
- Location: `src/App.tsx` (lines 125-340, startProcessing function)
- Contains: Validation logic, data transformation, chunking, file naming, manifest generation
- Depends on: PapaParse for CSV parsing, JSZip for ZIP creation, file-saver for downloads
- Used by: startProcessing event handler

**Validation Layer:**
- Purpose: Enforce data integrity and format rules
- Location: `src/App.tsx` (lines 64-77, validator functions)
- Contains: Helper functions for validating email, currency, amount, dates, month, filenames, MIME types
- Depends on: Regular expressions and parsing functions
- Used by: startProcessing transformation loop

**Utility Layer:**
- Purpose: Provide common helpers for formatting, transformation, and output
- Location: `src/lib/utils.ts` (cn function), `src/App.tsx` (lines 64-121)
- Contains: CSV generation, file downloading, date formatting, header normalization, deduplication key generation
- Depends on: clsx, tailwind-merge (for styling utilities)
- Used by: All layers

## Data Flow

**File Upload → Processing → Download:**

1. **File Input Phase:**
   - User selects CSV via drag-drop or file input (`acceptCsvFile`)
   - File validated for: existence, CSV extension, CSV MIME type, file size ≤ 25MB
   - If invalid, uploadError state set; user sees error message

2. **Validation Phase:**
   - CSV parsed using PapaParse with skipEmptyLines: true
   - Header row extracted and normalized (lowercase, whitespace/underscore removed)
   - Headers validated: all 13 required INPUT_HEADERS present, no extra columns allowed
   - Body rows validated: each row must have correct column count
   - Max 50k data rows enforced
   - If headers invalid, errors.csv auto-downloaded; status set to Failed

3. **Transformation Phase:**
   - Each data row processed through validation checks:
     - Email format validation (regex)
     - Amount validation (numeric, 2 decimals max, positive)
     - Currency validation (3-letter ISO code)
     - Month validation (1-12)
     - CardID validation (empty=new member, digits-only=existing member)
     - Country mapping (hardcoded: Myanmar→MM, Thailand→TH, else→ZZ)
     - Date validation (ISO format, PaymentCheckDate)
   - Exact duplicates detected via dupKeyOf() key generation; dropped with warning
   - Rows split into two output types:
     - **New members** (cardTrim === ""): 8-column format
     - **Existing members** (cardTrim !== ""): 6-column format with PRF-prefixed CardID
   - Errors and warnings accumulated; non-error rows counted as valid

4. **Splitting Phase:**
   - Valid new member rows chunked into groups of 300
   - Valid existing member rows chunked into groups of 300
   - Prevents single oversized output file

5. **Naming Phase:**
   - Sequential filename generation using startSeq (user-provided 3+ digit seed)
   - New member files: `{SEQ}_prf_bulk_import_{DATEUTC}.csv`
   - Existing member files: `{SEQ}_extension_prf_bulk_import_{DATEUC}.csv`
   - SEQ increments for each chunk; leading zeros preserved

6. **Packaging Phase:**
   - New member CSVs created with headers: [Name, Email, Country, Total Amount, Currency, Month, SupportRegion, Note]
   - Existing member CSVs created with headers: [PRF Card No, TotalAmount, Currency, Month, SupportRegion, Note]
   - All CSVs generated with quotes=true, CRLF newlines
   - ZIP assembled containing: all CSVs + warnings.csv (if any) + errors.csv (if any) + manifest.json
   - manifest.json includes: jobId, dateUTC, startSeq, sequence range, file lists, counts
   - ZIP downloaded to user; summary displayed on-screen

**State Management:**
- Component state (useState): csvFile, startSeq, status, msgsErr, msgsWarn, counts, summary, uploadError, dndActive, zipBlob, zipFileName, fileInputKey, stageIdx
- Computed state (useMemo): startSeqValid, canStart, progressValue
- Progress calculated from JobStatus enum; stage state computed from stageIdx
- Reset clears all state and re-renders upload form

## Key Abstractions

**JobStatus Enum:**
- Purpose: Represent processing pipeline stages
- Examples: "Idle", "Validating", "Transforming", "Splitting", "Naming", "Packaging", "Complete", "Failed"
- Pattern: String literal union type controls UI rendering, progress calculation, button enable/disable

**ProcessCounts Type:**
- Purpose: Track row statistics during processing
- Schema: { total, valid, newCount, oldCount, warnings, errors }
- Pattern: Updated during transformation; displayed in live summary panel

**RowMsg Type:**
- Purpose: Represent a validation error or warning with source context
- Schema: { line (physical CSV line number), code (error code), message (human-readable text) }
- Pattern: Accumulated during transformation; displayed in error/warning tables; exported to error.csv

**Error Code System (CODES):**
- Purpose: Categorize failures and warnings with machine-readable identifiers
- Categories:
  - E-HEADERS-* / E-ROW-* / E-EMAIL-* / E-AMOUNT-* / E-CURR-* / E-CARDID-* / E-FILE-* (errors)
  - W-HEADERS-* / W-CARDID-* / W-HQID-* / W-COUNTRY-* / W-DUP-* / W-MONTH-* (warnings)
- Pattern: Error codes included in RowMsg; auto-downloaded in errors.csv for downstream processing

**Country Mapping (COUNTRY_MAP):**
- Purpose: Convert country names to ISO-2 codes with fallback
- Hardcoded: Myanmar→MM, Thailand→TH, unknown→ZZ
- Pattern: Called during transformation; failures logged as W-COUNTRY-UNMAPPED warning

**File Helpers:**
- `isCsvFileName()`: Regex test for .csv extension
- `isCsvMime()`: Accept text/csv, application/vnd.ms-excel, or empty MIME
- `downloadBlob()`: Cross-browser blob download with fallback object URL
- `openBlobInNewTab()`: Open blob in new tab with cleanup

## Entry Points

**React Mount:**
- Location: `src/main.tsx`
- Triggers: Page load
- Responsibilities: Render App component into DOM root element

**User File Selection:**
- Location: `acceptCsvFile()` function (triggered by Input onChange or onDrop)
- Triggers: File input change event or drag-drop event
- Responsibilities: Validate file; set csvFile state or uploadError

**Start Processing:**
- Location: `startProcessing()` async function (triggered by "Start Processing" button)
- Triggers: canStart condition met (csvFile && startSeqValid && no errors && not processing)
- Responsibilities: Execute full pipeline; manage status, stage, messages, counts, download

**Reset:**
- Location: `handleReset()` function (triggered by "Reset" button)
- Triggers: User clicks Reset
- Responsibilities: Clear all state; reset file input; return to Idle

## Error Handling

**Strategy:** Fail-fast with detailed error reporting

**Patterns:**
- **CSV Parse Errors:** Caught from PapaParse; single RowMsg generated; status→Failed; errors.csv auto-downloaded
- **Header Validation:** All-or-nothing; missing headers block processing; extra headers logged but don't block
- **Data Row Errors:** Per-row validation; line number and code included; rows skipped (not added to output)
- **File Size Limits:** Checked before processing; sets uploadError (UI only) or blocks if rows exceed 50k
- **Duplicate Detection:** Warning level; exact row duplicates dropped silently with W-DUP-EXACT message
- **Missing Optional Fields:** Allowed; empty strings passed through (except CardID empty = new member)

## Cross-Cutting Concerns

**Logging:** No external logging; only UI-based feedback via RowMsg tables and status indicators

**Validation:** Multi-stage validation with regex, type checking, range checks; hardcoded rules for country mapping and currency codes

**Authentication:** Not applicable; client-side only

**Date/Time:** UTC dates in YYYYMMDD format; all timestamps generated client-side; no timezone conversion

**File Operations:** All in-browser using File API, PapaParse, JSZip, file-saver; no server upload

**Performance Optimization:** Stage delays (800-1600ms) added for UX richness; chunking at 300 rows per file; no optimization for large datasets (50k limit)

---

*Architecture analysis: 2026-02-22*
