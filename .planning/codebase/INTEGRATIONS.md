# External Integrations

**Analysis Date:** 2026-02-22

## APIs & External Services

**None detected** - This is a standalone client-side application with no external API integrations. The application processes data entirely in the browser.

## Data Storage

**Databases:**
- Not applicable - No persistent data storage

**File Storage:**
- Local filesystem only - Application generates and downloads files to user's local machine:
  - CSV files (processed bulk import data)
  - ZIP archives (packaged outputs with manifest)
  - Error/warning CSV reports
  - Browser file download via Blob API and file-saver

**Caching:**
- None - No caching mechanism required

## Authentication & Identity

**Auth Provider:**
- Not applicable - No user authentication or identity management

**Access Control:**
- Public static content (GitHub Pages)
- No user sessions or credentials

## Monitoring & Observability

**Error Tracking:**
- None - No external error tracking service

**Logs:**
- Browser console only - Client-side validation errors and messages logged to console
- All processing happens in-memory with no remote logging

**Data Processing:**
- All data processing occurs client-side in browser
- No data transmitted to external servers
- No webhooks or callbacks

## CI/CD & Deployment

**Hosting:**
- GitHub Pages
- Repository: `https://github.com/geeksquadstudio/csvHopeFuel.git`
- Deployed to: `https://[username].github.io/csvHopeFuel`

**CI Pipeline:**
- Not detected - Manual deployment via `npm run deploy` (gh-pages package)
- No GitHub Actions workflows present
- Build process: `npm run build && npm run deploy`

**Deployment Method:**
- gh-pages npm package handles Git push to gh-pages branch
- Static site serving via GitHub Pages

## Environment Configuration

**Required env vars:**
- None - No environment configuration needed

**Secrets location:**
- Not applicable - No secrets or API keys required

**Build Environment:**
- Public repository
- No credentials in version control

## Webhooks & Callbacks

**Incoming:**
- None - No webhook endpoints

**Outgoing:**
- None - No external service callbacks

## Data Flow

**User Interaction:**
1. User uploads CSV file via browser UI
2. File parsed client-side using PapaParse
3. Data validated with hardcoded rules (country mappings, email format, currency codes)
4. Valid rows transformed into separate new/existing member CSV exports
5. Rows grouped into chunks (300 rows per file)
6. CSV files packaged into ZIP with manifest.json
7. ZIP file generated in-memory using JSZip
8. User downloads ZIP to local machine via browser download

**No External Communication:**
- All processing stays in browser memory
- No API calls to external services
- No data storage beyond local browser session
- Files only leave user's machine via browser download

## Key Characteristics

**Client-Side Only:**
- 100% browser-based processing
- No server infrastructure required
- No data leaves user's machine unless explicitly downloaded
- Zero dependencies on external services or APIs

**File Processing:**
- Formats: CSV input/output
- Max file size: 25MB
- Max rows: 50,000
- Output: ZIP archive containing multiple CSV files and manifest

**Country Mapping:**
- Hardcoded mapping: `{ Myanmar: "MM", Thailand: "TH" }`
- Unmapped countries get code `"ZZ"`
- No external geolocation or reference data

**Error Handling:**
- Client-side validation only
- Error messages exported as CSV for user review
- No error reporting to external services

---

*Integration audit: 2026-02-22*
