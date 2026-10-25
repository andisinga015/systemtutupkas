# TUTUP KAS LION GROUP — V4.2.2
## DUAL-PERIOD DOC DEPO/WD, MONTHLY QRIS, INDEPENDENT ACTIVE/INACTIVE TABS
## STRICT FINANCIAL AND V4.2.1 PERFORMANCE PRESERVATION

# 0. MANDATORY INSTRUCTION

Work from the complete existing Tutup Kas Lion Group **V4.2.1** source code supplied in the ZIP.

Create a complete, production-ready **V4.2.2** release.

This is a targeted correction to:

1. Source document registration.
2. Half-month DOC DEPO/WD selection.
3. Monthly DOC QRIS selection.
4. Independent source-tab activation and deactivation.
5. Source configuration UI and API.
6. Run creation and source snapshots.
7. Item-specific retry.
8. Safe migration of historical configurations.

Do not redesign the financial calculation engine.

Do not change the existing destination mapping architecture.

Do not remove the V4.2.1 writer performance optimizations.

Do not add unrelated features.

## Core requirements

For each WEBSITE and calendar MONTH:

- **DOC A**: one DEPO/WD Google Spreadsheet for dates 1–15.
- **DOC B**: one DEPO/WD Google Spreadsheet for dates 16–the last calendar day.
- **DOC QRIS**: one Google Spreadsheet for the entire month.

Each document contains its own tabs.

Each document must have its own independent active/inactive tab configuration.

A normal Run uses exactly ONE date-appropriate DEPO document plus the applicable monthly QRIS source.

The existing monthly Kas Kecil mapping remains unchanged.

**NEVER GUESS SOURCE IDENTITIES, NEVER MIX DOC A AND DOC B, AND NEVER REACTIVATE DISABLED TABS AUTOMATICALLY.**

---

# 1. VERIFIED CURRENT IMPLEMENTATION

The existing V4.2.1 code contains the following relevant structures.

## backend/app/models.py

`SourceDocument` currently has:

`UniqueConstraint("website_id", "document_type", "year", "month")`

This permits only one DEPO source per month.

`SourceSheet` currently stores:

- `source_document_id`
- `source_revision_id`
- `tab_name`
- `is_active`
- `is_available`

Its existing relationship to source revisions must be preserved.

## backend/app/api.py

`SourceSave` currently accepts:

- year
- month
- document_type
- spreadsheet_url

The existing endpoint:

`PUT /websites/{id}/sources`

looks up the source by website, document type, year and month.

This must become period-aware.

Existing source-tab endpoints include:

- `GET /sources/{id}/tabs`
- `POST /sources/{id}/tabs`
- `POST /sources/{id}/tabs/refresh`

Preserve their safe existing behavior while supporting DOC A, DOC B and QRIS independently.

## frontend/src/Settings.tsx

The current source configuration renders only:

- DOC DEPO
- DOC QRIS

The current `TabManagement()` uses:

`docs.find(d => d.document_type === type)`

and offers only:

`['DEPO', 'QRIS']`

This is insufficient for two DEPO spreadsheets.

**Simply adding another DEPO source card without modifying TabManagement correctly is a critical bug.**

Both DEPO documents would share `document_type='DEPO'`, making `.find()` ambiguous.

Replace this selection logic with unambiguous document type + source period identity.

## backend/app/workflow.py

Inspect and update:

- `source_configuration()`
- `make_snapshot()`
- `create_run()`
- Source configuration signatures.
- Run currentness rules.
- Existing readiness validation.

Also inspect affected source authorization, snapshots, indexing, source identity, batch creation and selective retry paths.

Do not rewrite unrelated modules.

---

# 2. EXACT SOURCE DOCUMENT MODEL

Use the following administrator-facing names:

| Document | Internal type | Source period | Date range |
|---|---|---|---|
| DOC A — DEPO/WD | DEPO | FIRST_HALF | 1–15 |
| DOC B — DEPO/WD | DEPO | SECOND_HALF | 16–last day |
| DOC QRIS | QRIS | MONTHLY | 1–last day |

`DOC A` and `DOC B` are clear administrator-facing labels.

Keep `DEPO` and `QRIS` as the existing logical financial source document types.

Do not invent new financial module types.

Add a period identifier such as:

`source_period`

Allowed persisted states:

- FIRST_HALF
- SECOND_HALF
- MONTHLY
- UNASSIGNED, used only for legacy DEPO configurations awaiting explicit assignment.

Required combinations:

- DEPO + FIRST_HALF
- DEPO + SECOND_HALF
- QRIS + MONTHLY
- DEPO + UNASSIGNED only during legacy migration

Reject invalid combinations.

The normal source identity is:

`website_id + year + month + document_type + source_period`

This must distinguish DOC A from DOC B even though both have document_type DEPO.

---

# 3. THREE SOURCE URL CARDS IN ADMIN SETTINGS

In:

`/admin-setting/<website-slug>`

For each selected month/year, display three independent source configuration cards.

### CARD 1

**DOC A — DEPO/WD (Tanggal 1–15)**

Includes:

- Google Spreadsheet URL.
- Save/Edit URL.
- Validate URL.
- Open Google Spreadsheet.
- Validation status.
- URL history.

### CARD 2

**DOC B — DEPO/WD (Tanggal 16–Akhir Bulan)**

Includes exactly the same controls, separately from DOC A.

### CARD 3

**DOC QRIS (Tanggal 1–Akhir Bulan)**

Includes exactly the same controls.

### Independent changes

Changing DOC A must never change DOC B or DOC QRIS.

Changing DOC B must never change DOC A or DOC QRIS.

Changing DOC QRIS must never change either DEPO source.

A period change in the Admin date/month selector must load that month's three document records, without mixing other months.

Use clear labels such as:

- Oktober 2026 — DOC A — 1–15 Oktober.
- Oktober 2026 — DOC B — 16–31 Oktober.
- Oktober 2026 — DOC QRIS — 1–31 Oktober.

For February, April, and other months, calculate the actual last date correctly.

Do not create another QRIS document for the second half.

Saving or validating a source URL must not automatically execute a financial Crawl.

---

# 4. CRITICAL: INDEPENDENT ACTIVE / INACTIVE TAB MANAGEMENT

THIS SECTION IS MANDATORY AND MUST NOT BE OMITTED.

The administrator requires separate active/inactive tab management for DOC A, DOC B and DOC QRIS.

## 4.1 Source document selector

In:

`/admin-setting/<website-slug>/sheet-aktif`

Replace the current two-option selector with THREE clearly labeled options:

1. **DOC A — DEPO/WD (1–15)**
2. **DOC B — DEPO/WD (16–akhir bulan)**
3. **DOC QRIS — Bulanan**

Use tabs, segmented buttons or another existing UI component consistent with the current design.

Do not combine both DEPO document lists.

## 4.2 Correct document lookup

Do not use:

`docs.find(d => d.document_type === 'DEPO')`

because DOC A and DOC B both have document_type DEPO.

Use explicit unique selection:

- DOC A → DEPO + FIRST_HALF.
- DOC B → DEPO + SECOND_HALF.
- DOC QRIS → QRIS + MONTHLY.

Resolve the actual SourceDocument ID.

Call the existing source-tab API with that exact document ID.

Source-tab records must be loaded for the selected document's CURRENT source revision only.

## 4.3 Example

October 2026, VELBETT.

DOC A contains:

- WD BCA 1 / IVAN — ACTIVE.
- WD BCA 2 / ALIN — INACTIVE.

DOC B contains:

- WD BCA UTAMA / IVAN — ACTIVE.
- WD BCA 2 / ALIN — ACTIVE.
- WD DANA 1 / FATMAWATI — INACTIVE.

DOC QRIS contains:

- QRIS E-POWER — ACTIVE.
- QRIS QRIZZ — ACTIVE.

If Admin selects DOC A:

Only display DOC A's source tabs and their exact statuses.

If Admin selects DOC B:

Only display DOC B's source tabs and their exact statuses.

If Admin selects DOC QRIS:

Only display DOC QRIS source tabs and their exact statuses.

The website, month and selected document must remain clearly visible.

## 4.4 Independent activation

If an administrator activates:

`DOC A → WD BCA 2 / ALIN`

then ONLY that SourceSheet record under DOC A becomes active.

The similarly named source tab in DOC B must remain unchanged.

If an administrator disables:

`DOC B → WD BCA 2 / ALIN`

the DOC A tab with the same name must remain unchanged.

Names are NOT proof that two tabs from two documents are identical.

## 4.5 Active/inactive operations

Preserve the existing controls for:

- Select one tab.
- Select multiple tabs.
- Activate selected tabs.
- Deactivate selected tabs.
- Refresh tab list.
- Display current availability.
- Display current activation status.

The action must apply ONLY to the currently selected SourceDocument ID and source revision.

Never apply one activation operation to both DEPO documents.

Do not add a global switch that accidentally activates or disables every tab in DOC A and DOC B together.

## 4.6 Frontend state isolation

When switching from DOC A to DOC B:

- Clear previous selected checkboxes.
- Load DOC B's current tab list.
- Show DOC B's active/inactive states.
- Reset any pending bulk action tied to DOC A.
- Do not retain DOC A's selected tab IDs.

When switching to QRIS, follow the same behavior.

Also reset and refetch correctly when:

- Website changes.
- Month changes.
- Year changes.
- Source URL changes.
- Source document revision changes.

Protect against stale asynchronous API responses.

An old DOC A fetch must not overwrite the visible DOC B tab list when the user switches quickly.

Disable or safely reject actions while the selected source document is loading or has changed.

## 4.7 Missing document

If DOC B has not been configured:

Display a clear state such as:

"DOC B tanggal 16–akhir bulan belum memiliki URL."

Disable Activate, Deactivate and Refresh actions for DOC B.

Do NOT show DOC A's tabs as a fallback.

Do NOT silently select another source.

## 4.8 New month and newly discovered tabs

Keep the existing business requirement:

**New tabs default to INACTIVE.**

A newly added month must not automatically activate tabs based on last month's configuration.

When a new DOC A or DOC B URL is registered and its tabs are discovered, they must default to inactive unless the application has verified and explicitly preserved activation from the same document's valid history under an existing approved rule.

Do not copy activation settings from DOC A into DOC B.

Do not copy activation settings from DOC B into DOC A.

Do not copy activation settings from one month into another.

Do not activate newly discovered tabs automatically.

## 4.9 Rename, deletion and recreation

If a tab is renamed in the same spreadsheet:

- Follow the existing verified lineage rules.
- Preserve activation only if continuity can be proven.
- Otherwise require Admin review.

If a tab is deleted and recreated with the same name but a different internal identity:

- Do not guess identity continuity.
- Do not automatically inherit activation or historical ownership.
- Report a source identity error requiring revalidation.

Internal Google sheet IDs may be used as evidence, but Admin must not be required to enter them manually.

## 4.10 Backend ownership checks

For activation/deactivation requests, validate that EVERY submitted tab ID belongs to:

- The requested SourceDocument.
- The current SourceRevision.
- The current website/month context where applicable.

Reject cross-document or stale-revision IDs.

Never allow a crafted or stale frontend request to activate DOC A tabs through DOC B.

Use database-safe update operations.

Return refreshed authoritative active/inactive states after updates.

## 4.11 Counts

Show correct per-document counts:

- DOC A: number active / inactive / unavailable.
- DOC B: number active / inactive / unavailable.
- DOC QRIS: number active / inactive / unavailable.

Do not combine those counts in a misleading way.

Any existing Dashboard aggregate counts must continue to represent actual registered SourceSheet states without double-counting stale revisions.

---

# 5. DATE-BASED SOURCE ROUTING

Implement deterministic source selection using the exact selected calendar date.

### Days 1–15

Select:

- DOC A — DEPO FIRST_HALF.
- DOC QRIS — MONTHLY, if configured and applicable.

Do NOT select DOC B.

### Days 16–last calendar day

Select:

- DOC B — DEPO SECOND_HALF.
- DOC QRIS — MONTHLY, if configured and applicable.

Do NOT select DOC A.

### Examples

October 15, 2026:

- DEPO → DOC A.
- QRIS → Monthly QRIS.

October 16, 2026:

- DEPO → DOC B.
- QRIS → Monthly QRIS.

October 18, 2026:

- DEPO → DOC B.
- QRIS → Monthly QRIS.

October 31, 2026:

- DEPO → DOC B.
- QRIS → Monthly QRIS.

February 28, 2026:

- DEPO → DOC B.

February 29, 2028:

- DEPO → DOC B.

The half-month boundary is always between day 15 and day 16.

Do not infer the period from spreadsheet titles or file names.

Do not inspect both DEPO documents to choose which one contains the date.

Do not silently fall back to DOC A if DOC B is missing.

Do not silently fall back to DOC B if DOC A is missing.

A missing relevant document is a configuration error for the selected date.

---

# 6. HOW ACTIVE/INACTIVE AFFECTS ROBOT EXECUTION

This is the operational acceptance rule.

### Run dated October 10

Robot must:

1. Resolve DOC A.
2. Read ONLY active, available DOC A tabs.
3. Ignore inactive DOC A tabs.
4. Ignore all DOC B tabs, regardless of activation status.
5. Read active applicable monthly QRIS tabs.
6. Calculate and write according to existing rules.

### Run dated October 18

Robot must:

1. Resolve DOC B.
2. Read ONLY active, available DOC B tabs.
3. Ignore inactive DOC B tabs.
4. Ignore all DOC A tabs, regardless of activation status.
5. Read active applicable monthly QRIS tabs.
6. Calculate and write according to existing rules.

### Important

An inactive source tab must not create normal financial Run items.

An unavailable or identity-conflicted tab must not be silently treated as active.

If a required source document has no usable active tabs, use the existing configuration error behavior, with a clear period-specific message. Do not fabricate zero-valued financial results.

Preserve the existing optional QRIS behavior where applicable.

An invalid configured QRIS file must not be interpreted as a valid day without QRIS transactions.

---

# 7. RUN SNAPSHOT AND ITEM GENERATION

Modify the existing workflow so source selection happens BEFORE creating the Run snapshot or individual items.

The selected Run must contain:

- Exact website.
- Exact selected date.
- Exact DOC A or DOC B record.
- Exact monthly QRIS record where applicable.
- Exact source IDs.
- Source revisions.
- Spreadsheet IDs.
- Active tab names.
- Proven tab identities.
- Existing aliases.
- Source period identity.

Only the selected half-month DEPO document may contribute Run items.

The unused half-month document must not contribute:

- DEPO items.
- WD items.
- SALDO items.
- Expense items.
- Transfer items.
- Snapshot reads.
- Source index reads.

QRIS continues to contribute only its applicable existing module items.

### Critical example

If DOC A has 10 active tabs and DOC B has 8 active tabs:

For October 18:

Robot reads the 8 active DOC B tabs.

Robot does NOT read the 10 DOC A tabs.

The number of Run items must reflect selected sources only.

Do not append DOC A and DOC B item lists.

Do not double-count expenses or transfers.

---

# 8. DATABASE MIGRATION

Extend `source_documents` with a validated `source_period` field or equivalent properly normalized representation.

Update the uniqueness constraint from:

`website_id + document_type + year + month`

to:

`website_id + document_type + year + month + source_period`

Use a safe SQLite/Alembic migration.

Preserve original SourceDocument IDs and foreign-key references wherever possible.

Preserve:

- Existing users.
- Websites.
- SourceDocument IDs.
- Source revision history.
- SourceSheet records.
- Active/inactive states.
- Source lineage.
- Source Date Index.
- Historical snapshots.
- Run history.
- Retry state.
- Financial ledger.
- Transfer ownership.
- Destination mapping.
- Audit history.

Do not clear or truncate any historical financial records.

### Legacy DEPO records

Existing V4.2/V4.2.1 installations may have one legacy DEPO URL per website/month.

It is unknown whether that URL corresponds to DOC A or DOC B.

Migrate it to an explicit UNASSIGNED state.

Display the old URL in Admin.

Provide two deliberate Admin actions:

- "Tetapkan sebagai DOC A — tanggal 1–15"
- "Tetapkan sebagai DOC B — tanggal 16–akhir bulan"

The assignment must preserve its source record identity and tab activation history.

If the administrator assigns the legacy DEPO to DOC B, the original active/inactive tab settings must remain associated with that same record in DOC B.

Do not clone its tabs to DOC A.

Do not guess the period from past Run dates.

Do not replace an existing configured period record silently.

### Legacy QRIS

Migrate existing QRIS source records to MONTHLY.

Preserve URL, validation, revisions and active/inactive tabs.

---

# 9. SOURCE VALIDATION AND HISTORY

Each source document has an independent URL and validation lifecycle.

For DOC A:

- Validate DOC A's exact URL.
- Refresh only DOC A's source tabs.
- Preserve DOC B and QRIS settings.

For DOC B:

- Validate DOC B's exact URL.
- Refresh only DOC B's source tabs.
- Preserve DOC A and QRIS settings.

For QRIS:

- Validate QRIS's exact URL.
- Refresh only QRIS tabs.
- Preserve both DEPO documents.

URL history must clearly label source period.

Example:

"VELBETT — Oktober 2026 — DOC B — Revisi 3"

Changing a source URL must invalidate only the affected source revision and dependent processing evidence.

It must not rebuild unrelated source documents.

It must not invalidate the READY monthly Kas Kecil mapping merely because the source URL changed.

---

# 10. SOURCE DATE INDEX

Preserve the current optimized date indexing implementation.

Date indexes must be isolated by actual source document and tab identity.

DOC A and DOC B have independent row positions.

If October 15 in DOC A ends at row 350, do not assume October 16 in DOC B starts at row 351.

DOC B is a different Google Spreadsheet.

Every source index must preserve its own:

- Spreadsheet ID.
- SourceDocument ID.
- Source revision.
- Internal tab identity.
- Selected date.
- Verified separator boundaries.

Follow the original following-date separator logic.

Do not invent transaction boundaries at day 15 or 16.

If a source document ends without the usual next-date marker, use only demonstrably safe source evidence under existing rules; otherwise report an explicit boundary error.

Never fabricate a balance or a transaction range.

---

# 11. RETRY — ONLY THE FAILED ITEM

Preserve the existing selective retry implementation.

When retrying an item:

- Identify the original logical item and selected date.
- Determine the correct source period.
- Use its exact SourceDocument identity.
- Fetch fresh raw data required ONLY for that failed item.
- Do not rescan every source tab.
- Do not process successful items again.
- Do not rebuild the READY destination mapping.
- Do not replay verified writes.

For October 18, a DEPO-related failed item must use DOC B.

For October 10, a DEPO-related failed item must use DOC A.

For QRIS, use monthly QRIS.

If the relevant source document URL or identity has changed, use the existing explicit source reauthorization safeguards.

Do not silently switch old historical items from DOC A to DOC B or vice versa.

Preserve:

- Masukkan Data Baru.
- Tetapkan Data Lama.
- Old/new actual destination comparisons.
- Formula protection.
- Notes protection.
- Ledger ownership.
- Idempotent recovery.

---

# 12. SOURCE SIGNATURES AND CURRENTNESS

Inspect and update all affected source context signatures and validation functions, including:

- source_configuration()
- source_signature()
- current_source_signature()
- run_source_signature()
- context_signature()
- is_current_run()
- source_authorization functions.

A Run for October 18 depends on:

- DOC B.
- Monthly QRIS where applicable.

It must NOT become stale merely because DOC A was edited.

Likewise, an October 10 Run must not become stale merely because DOC B was edited.

The destination mapping signature remains separate and unaffected by source period selection.

---

# 13. CRAWL ALL

The existing Crawl All functionality must keep working.

For each website and selected date, use that website's own:

- DOC A or DOC B.
- Monthly QRIS.
- Active/inactive tab settings.

Do not reuse VELBETT's source configuration for another website.

If FANTASI's DOC B is missing, report FANTASI's configuration error.

Do not guess or substitute another document.

Do not prevent safe independent website processing merely because another website has an invalid source configuration.

Do not add new five-worker concurrency in this release.

Do not redesign the existing queue system.

---

# 14. FINANCIAL RULES — DO NOT CHANGE

Absolutely preserve all existing financial processing rules.

## DOC DEPO

- D: description.
- E: DEPO.
- F: WD.
- H: expense.
- I: KIRIMAN.
- J: cumulative DEPO.
- K: cumulative WD.
- L: SALDO.

## DOC QRIS

- D: description.
- E: DEPO.
- F: WD.
- G: expense.
- H: KIRIMAN.
- I: cumulative DEPO.
- J: cumulative WD.
- K: SALDO.

Preserve all existing:

- Source separator rules.
- Empty-date logic.
- Closing saldo calculations.
- BIAYA aggregation.
- FEE QRIS DEPO.
- FEE QRIS WD.
- KET PENGIRIMAN.
- Brown/blue transfer recognition.
- QRIS bank allocation.
- Original descriptions.
- Google Notes.
- Existing write ownership.
- Duplicate prevention.
- Existing conflict resolution.
- Financial reconciliation.

`Laporan deposit harian` must remain STRICTLY READ ONLY.

Do not force KLOP.

---

# 15. ABSOLUTE KAS KECIL MAPPING — NO CHANGES

The existing monthly destination mapping is based on website/date/module/account.

It must remain independent of source document period and source tab names.

Do not:

- Rebuild Kas Kecil mapping just because DOC A or DOC B is registered.
- Change absolute row or column coordinates.
- Modify destination Google Sheet structure.
- Change existing formulas.
- Delete Notes.
- Alter destination website boundaries.
- Automatically rewrite existing data.

Preserve READY destination mapping versions where still valid.

---

# 16. V4.2.1 PERFORMANCE — MUST BE PRESERVED

The V4.2.1 source ZIP already includes improvements for:

- Batch destination validation.
- Bounded destination requests.
- Batch financial writes.
- Grouped read-back verification.
- Reduced destination scanning.

Keep these improvements.

Do not revert to the older V4.2 writer.

Do not change:

- Worker MemoryMax.
- Server architecture.
- Google credentials.
- API quotas as an untested shortcut.
- Source financial calculations.
- Unrelated frontend components.

Do not introduce Chromium scraping.

The new source routing must not cause both half-month DEPO files to be read for a single-date Run.

Performance must not regress due to this change.

---

# 17. REQUIRED REGRESSION TESTS — ESPECIALLY ACTIVE/INACTIVE

These tests are mandatory.

## A. Three source cards

1. Display DOC A for days 1–15.
2. Display DOC B for days 16–end-of-month.
3. Display monthly DOC QRIS.
4. Save three independent URLs.
5. Editing DOC A does not change DOC B.
6. Editing DOC B does not change DOC A.
7. Editing QRIS does not change DOC A or B.
8. Repeated identical URL saves are idempotent.
9. Period/year change loads the correct records.
10. History displays the correct document and period.

## B. Active/Inactive UI — critical

11. Source tab selector shows DOC A, DOC B and QRIS.
12. Selecting DOC A loads only DOC A tabs.
13. Selecting DOC B loads only DOC B tabs.
14. Selecting QRIS loads only QRIS tabs.
15. DOC A and DOC B can contain the same tab name without ambiguity.
16. Activating DOC A's tab never activates the same-named DOC B tab.
17. Disabling DOC B's tab never disables DOC A's tab.
18. QRIS activation does not change DEPO activation.
19. Switching documents clears selected checkbox IDs.
20. Switching documents reloads actual active/inactive states.
21. Switching months clears old selections.
22. Rapid document switching cannot display stale results.
23. A stale API response cannot replace the currently selected document's tabs.
24. Missing DOC B displays an explicit empty/configuration state.
25. Missing DOC B cannot fall back to DOC A.
26. New tabs default to inactive.
27. New month's tabs default to inactive.
28. Activations are not copied between half-month documents.
29. Refresh DOC A leaves DOC B unchanged.
30. Refresh DOC B leaves DOC A unchanged.
31. Refresh QRIS leaves both DEPO documents unchanged.
32. Bulk activate affects only the selected document.
33. Bulk deactivate affects only the selected document.
34. Backend rejects tab IDs belonging to a different source document.
35. Backend rejects stale source-revision tab IDs.
36. Status counters accurately show active, inactive and unavailable tabs.
37. Proven rename preserves applicable history safely.
38. Deleted/recreated tabs do not inherit ownership automatically.

## C. Date routing

39. Day 1 uses DOC A.
40. Day 15 uses DOC A.
41. Day 16 uses DOC B.
42. Day 18 uses DOC B.
43. Day 28 uses DOC B.
44. Day 29 in leap-year February uses DOC B.
45. Day 30 uses DOC B.
46. Day 31 uses DOC B.
47. QRIS remains the same monthly document.
48. Missing relevant DEPO period raises a specific error.
49. Inactive tabs are excluded from Run items.
50. Unused-period tabs are excluded even if active.
51. Source configuration never includes both DOC A and DOC B for one date.

## D. Database migration

52. Existing DEPO becomes UNASSIGNED.
53. Admin explicitly assigns legacy DEPO to DOC A.
54. Admin explicitly assigns legacy DEPO to DOC B.
55. Existing SourceDocument ID is retained.
56. Historical source revisions remain.
57. Existing active/inactive tab states remain with the assigned document.
58. No old tabs are cloned to the other period.
59. Existing QRIS becomes MONTHLY without losing its settings.
60. Old Runs remain readable.
61. Write ledger and transfer ownership remain intact.
62. Migration rollback and backup procedures are valid.

## E. Financial workflow

63. October 10 processes only active DOC A tabs.
64. October 18 processes only active DOC B tabs.
65. Both dates use the monthly QRIS as applicable.
66. No duplicated DEPO/WD items.
67. BIAYA is not aggregated from both DEPO documents.
68. Separate Date Index is used per source document.
69. Retrying a DOC A failed item uses DOC A fresh raw data.
70. Retrying a DOC B failed item uses DOC B fresh raw data.
71. Retrying a QRIS failed item uses monthly QRIS.
72. Successful items remain untouched.
73. Existing destination mapping remains unchanged.
74. Existing V4.2.1 writer batching remains intact.
75. Existing financial calculations match approved fixtures.
76. Notes and formula protections remain.
77. KET PENGIRIMAN duplicate protection remains.
78. Reconciliation remains read-only and truthful.
79. Crawl All uses each website's independent source settings.

Run the full existing V4.2.1 regression suite in addition to these tests.

Do not delete or weaken existing tests.

---

# 18. SAFE UPGRADE FROM PRODUCTION

The production VPS may still be running V4.2.

The V4.2.1 ZIP is available but may not yet have been installed.

Therefore, build V4.2.2 from the V4.2.1 source code while supplying a safe upgrade path from:

- V4.2 → V4.2.2.
- V4.2.1 → V4.2.2.

For the V4.2 path, all V4.2.1 writer optimizations must be included.

Detect the installed schema and release.

Do not assume the current version solely from the staging folder name.

Required deployment behavior:

1. Check for active financial Runs.
2. Stop the worker safely.
3. Back up the SQLite database consistently.
4. Preserve all existing source and destination configuration.
5. Preserve historical Run data and ledger state.
6. Apply the correct Alembic migrations.
7. Install V4.2.2.
8. Verify API and schema.
9. Preserve Nginx, HTTPS, credentials and RAM settings.
10. Leave the worker STOPPED.
11. Never automatically resume historical canceled or partial Runs.
12. Provide rollback instructions.

After upgrade, Admin must assign the legacy DEPO URL to DOC A or DOC B explicitly before using that period for a new Run.

Do not start a Crawl automatically.

---

# 19. REQUIRED DELIVERABLES

Deliver a complete:

**tutupkas-lion-group-v4.2.2.zip**

Include:

1. Complete updated source code.
2. Database migration.
3. Explicit legacy DOC period assignment.
4. Three independently editable source URL cards.
5. Three separate choices in the Active/Inactive tab menu.
6. Independent SourceSheet status management for DOC A, DOC B and QRIS.
7. Backend cross-document activation protection.
8. Date-based source routing.
9. Correct Run snapshot and item creation.
10. Correct item-specific retry routing.
11. Preserved V4.2.1 destination writer optimizations.
12. Preserved Kas Kecil absolute mapping.
13. All regression tests.
14. Frontend build and test results.
15. Database migration test results.
16. Safe upgrade script.
17. Rollback instructions.
18. Changed-file summary.
19. Honest known limitations.

Explain every changed file and why it was necessary.

Do not include secrets or production databases in the ZIP.

---

# 20. FINAL ACCEPTANCE GATE

The release is acceptable ONLY if all of the following are true.

- [ ] Admin displays DOC A, DOC B and DOC QRIS independently.
- [ ] Admin Active/Inactive menu has three clear document choices.
- [ ] Selecting DOC A displays DOC A's real tab statuses.
- [ ] Selecting DOC B displays DOC B's real tab statuses.
- [ ] Selecting QRIS displays QRIS's real tab statuses.
- [ ] Identical tab names across A and B do not mix source identities.
- [ ] Activating/disabling a tab affects only the selected document.
- [ ] Refreshing one source does not change another source.
- [ ] New tabs remain inactive by default.
- [ ] Switching documents cannot submit stale selected tab IDs.
- [ ] Backend rejects cross-document tab activation.
- [ ] DOC A is selected only for dates 1–15.
- [ ] DOC B is selected only for dates 16–month end.
- [ ] Monthly QRIS is selected for applicable dates throughout the month.
- [ ] A Run never includes both half-month DEPO documents.
- [ ] Retry only re-reads failed-item data from the correct document.
- [ ] Legacy DEPO records are not assigned to a period by guessing.
- [ ] All historical financial records remain intact.
- [ ] V4.2.1 destination writer optimizations remain intact.
- [ ] Absolute Kas Kecil mapping remains unchanged.
- [ ] Financial calculations remain identical.
- [ ] Formula and Notes protections remain intact.
- [ ] Existing regression tests pass.
- [ ] Safe upgrade and rollback are provided.

## FINAL DIRECTIVE

**DOC A AND DOC B ARE DIFFERENT SOURCE DOCUMENTS, EVEN WHEN THEY CONTAIN TABS WITH IDENTICAL NAMES.**

**THE ACTIVE/INACTIVE SELECTION MUST ALWAYS TARGET ONE EXACT DOCUMENT ID AND ONE CURRENT SOURCE REVISION.**

**DOC A ACTIVE TABS AND DOC B ACTIVE TABS MUST NEVER INTERFERE WITH EACH OTHER.**

**SELECT THE SOURCE BY THE REQUESTED DATE BEFORE CREATING THE RUN.**

**PRESERVE THE ENTIRE V4.2.1 FINANCIAL ENGINE AND DESTINATION WRITER PERFORMANCE.**

Implement this completely.

Deliver working code, migration, Admin controls, tests and the final ZIP.

Do not deliver only a design proposal.

# END OF V4.2.2 INSTRUCTIONS