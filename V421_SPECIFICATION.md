# TUTUP KAS LION GROUP — V4.2.1
# CRITICAL DESTINATION WRITER PERFORMANCE PATCH
# NO FINANCIAL LOGIC CHANGES

Work from the existing V4.2 source code.

## CONFIRMED PRODUCTION BOTTLENECK

Run #9, VELBETT, October 18, 2026.

The application has already completed:

- 8 source snapshots.
- 43 financial plans.
- Monthly destination mapping #4 is READY.
- 8 destination tabs mapped.
- 7,936 absolute targets stored.

But during the financial writing process, the worker repeatedly issues large Google Sheets GET requests of approximately 7.7–8.3 MB while processing one item:

- Module: `expense_total`
- Phase: `CHECKING_DESTINATION`
- Worker RAM: approximately 138 MB
- No OOM or worker restart observed.

The application is excessively slow because destination verification repeatedly reads large portions of the spreadsheet despite having an approved absolute destination registry.

## REQUIRED INVESTIGATION

Trace every actual call in:

- `backend/app/engine/writer.py`
- `backend/app/destination_reader.py`
- `backend/app/engine/runner.py`
- `backend/app/destination_registry.py`
- `backend/app/sheets.py`

Especially investigate:

- `grid.refresh()`
- Full destination rescanning
- Structural checks
- Formula scans
- `execute()`
- `_submit_and_verify()`
- Repeated GET requests during `CHECKING_DESTINATION` and `VERIFYING`

Determine precisely which functions produce the repeated multi-megabyte responses.

Do not assume every GET is redundant without tracing the caller and its safety purpose.

## MANDATORY CORRECTION

1. Use the existing READY monthly Destination Mapping Registry as the authoritative approved absolute coordinate map.
2. Never rebuild the entire destination grid for each item merely to resolve an already-approved coordinate.
3. Replace repeated global scans with a safety-equivalent bounded validation mechanism. Identify every global invariant that the existing implementation protects and preserve it. If an invariant cannot be verified safely without global scanning, report that limitation rather than silently dropping the protection.
4. Group compatible destination reads using bounded batch requests.
5. Group financially independent write intents into safe batches.
6. Preserve each item's independent success/error state.
7. Verify the actual written cells using grouped, bounded read-back operations.
8. Maintain atomic database claims, ownership ledger, Notes, formulas, merged-cell protections, manual data protections, and idempotent recovery.
9. Preserve explicit Admin conflict decisions: Masukkan Data Baru and Tetapkan Data Lama.
10. Preserve all KET PENGIRIMAN slot allocation and duplicate-protection rules.
11. Preserve the exact destination cells, date matching, account mapping, fee rules, and financial calculations.
12. Preserve read-only Laporan deposit harian and true KLOP requirements.
13. Do not modify source snapshots, date indexing, source tab registration, authentication, unrelated UI, or worker concurrency.
14. Do not increase MemoryMax, API response limits, or replace Google Sheets API with browser automation.

## SPECIAL SAFETY REQUIREMENTS

- Mapping registry must be READY and tied to the correct monthly destination revision.
- A changed destination cell must be detected before overwriting.
- Formula, Note, and protected content must not be silently removed.
- A Google write succeeding before a timeout must not lead to duplicate writes.
- Batches must not create partial mismatches between descriptions, amounts, or Notes.
- Do not issue a batch containing conflicting writes to the same cell.
- Do not cache mutable destination values indefinitely.
- Do not assume a cached header alone guarantees that no structural change occurred.
- Preview must remain fully read-only.
- Running production jobs must not be silently replayed during upgrade.

## PERFORMANCE ACCEPTANCE TESTS

Use a representative VELBETT fixture with 43 financial items.

Measure the number of destination GET requests and response bytes:

- During planning.
- During pre-write validation.
- During writing.
- During read-back verification.

Compare V4.2 against V4.2.1.

Specifically assert:

- Processing one `expense_total` item does not trigger an unnecessary full destination rescan.
- Multiple compatible writes can share a bounded validation pass.
- Repeated multi-megabyte GET responses are eliminated where unnecessary.
- Absolute write coordinates are identical to V4.2's approved mapping.
- All financial values, descriptions, Notes, and conflict outcomes remain identical.
- A source or destination change during processing cannot silently corrupt the result.
- Crash and timeout recovery remain idempotent.
- No new OOM exposure.
- Existing regression tests continue to pass.

Measure total wall-clock duration and peak RSS.

Target under 60 seconds per representative website where Google latency and quota permit, but NEVER weaken financial checks to meet that target.

## DELIVERY

Deliver:

1. Complete V4.2.1 source ZIP.
2. Detailed root-cause analysis showing exact expensive call paths.
3. Minimal changed-file list.
4. Automated tests and measured before/after API request counts.
5. Proof that all existing financial invariants remain protected.
6. V4.2 → V4.2.1 safe upgrade script.
7. Rollback instructions.
8. Honest limitations and unverified conditions.

The upgrade must preserve the production SQLite database, accounts, source settings, READY mapping revisions, previous financial writes, ledgers, and all Run history.

Leave the worker stopped after upgrade pending manual verification.

Do not add new unrelated features.

**PRIMARY GOAL: MAKE THE EXISTING FINANCIAL WRITER EFFICIENT WITHOUT CHANGING WHAT IT WRITES OR WHICH CELLS IT PROTECTS.**