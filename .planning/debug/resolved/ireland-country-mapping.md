---
status: resolved
trigger: "Ireland country mapping should produce IE but showing ZZ instead"
created: 2026-02-22T00:00:00Z
updated: 2026-02-22T00:02:00Z
---

## Current Focus

hypothesis: CONFIRMED - Ireland was missing from COUNTRY_MAP
test: Added Ireland: IE to COUNTRY_MAP, verified line 59 of src/App.tsx
expecting: mapCountry("Ireland") now returns "IE" instead of "ZZ"
next_action: complete

## Symptoms

expected: When "Ireland" appears in the CSV data, the country code should map to "IE"
actual: Ireland is being mapped to "ZZ" (unknown/unmapped country code)
errors: No error messages - it silently maps to wrong code
reproduction: Upload a CSV with Ireland as a country value and observe the output country code
started: Unknown - likely has always been this way (mapping data issue)

## Eliminated

## Evidence

- timestamp: 2026-02-22T00:01:00Z
  checked: src/App.tsx line 59 - COUNTRY_MAP constant
  found: "const COUNTRY_MAP: Record<string, string> = { Myanmar: \"MM\", Thailand: \"TH\" };"
  implication: Ireland had no entry in the map whatsoever

- timestamp: 2026-02-22T00:01:00Z
  checked: src/App.tsx line 77 - mapCountry function
  found: "const mapCountry = (name: string) => COUNTRY_MAP[name.trim()] ?? \"ZZ\";"
  implication: Any country name not in COUNTRY_MAP falls back to "ZZ" - this is why Ireland produced "ZZ"

- timestamp: 2026-02-22T00:02:00Z
  checked: src/App.tsx line 59 after fix
  found: "const COUNTRY_MAP: Record<string, string> = { Ireland: \"IE\", Myanmar: \"MM\", Thailand: \"TH\" };"
  implication: Ireland now maps to "IE"; mapCountry("Ireland") returns "IE"

## Resolution

root_cause: "Ireland" was not present as a key in COUNTRY_MAP (line 59, src/App.tsx). The map only contained Myanmar->MM and Thailand->TH. The mapCountry function falls back to "ZZ" for any unmapped key via the nullish coalescing operator (?? "ZZ").
fix: Added "Ireland: IE" to COUNTRY_MAP on line 59 of src/App.tsx.
verification: Read line 59 post-edit confirms { Ireland: "IE", Myanmar: "MM", Thailand: "TH" }. The mapCountry function is unchanged - it will now resolve "Ireland" to "IE" instead of falling back to "ZZ".
files_changed:
  - src/App.tsx
