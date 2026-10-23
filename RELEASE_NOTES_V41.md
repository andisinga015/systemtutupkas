# Tutup Kas Lion Group V4.1 — koreksi akses tujuan

V4 menolak `SALDO BANK` karena `rowCount × columnCount` melebihi 250.000 sel, meskipun sebagian besar alokasi kosong. Setelah guard, V4 juga menumpuk hasil band menjadi grid padat dan menyimpannya sepanjang run. Writer mengulangi materialisasi sebelum/sesudah tulis; laporan mempunyai jalur yang sama. Source `L24` bukan penyebab penolakan tersebut.

V4.1 mengganti jalur tujuan dengan `DestinationReader`: discovery header values-only, scan global dalam rentang terbatas, indeks struktur temporer, pembacaan sel target eksplisit dan signature formula ringkas. Tidak ada guard berdasarkan produk dimensi tujuan. Tidak ada perubahan tata letak workbook atau koordinat finansial.

## Berkas/fungsi yang berubah

| Berkas | Fungsi dan tujuan |
|---|---|
| `backend/app/destination_reader.py` (baru) | `DestinationReader`, `_header_values`, `_scan`, `_groups`, `_responses`, `_structural`, `read_cells`, `prefetch`, `cell`, `inherited`, `refresh`, `close`; `MergeIndex`, `FormulaSignature`, `read_destination`, `close_view`: akses terbatas, indeks struktur dan resource cleanup |
| `backend/app/sheets.py` | `grid()` mendelegasikan ke reader; `destination_grid()` baru, menerima nama website untuk indeks. Request/auth/source/write transport tetap sama |
| `backend/app/engine/types.py` | `Grid.height`, `Grid.iter_cells()` untuk protokol koordinat absolut; constructor, Cell, Write, number/norm/fingerprint/column tetap sama |
| `backend/app/engine/resolver.py` | `website_block`, `date_row`, `transfer_slots`: iterator struktur menggantikan asumsi rows padat. Kondisi, batas blok, account, label, tanggal, bank dan alokasi tidak diganti |
| `backend/app/engine/report.py` | `markers`, `tables`, `resolve_balance`: iterator struktur/height; aturan marker, asosiasi, tanggal, Decimal dan nol tetap sama |
| `backend/app/engine/writer.py` | `structural`, `formula_map`, `check_write`, `execute`, `_execute_checked`, `_submit_and_verify`: signature global, query merge, fresh reader, target prefetch/read-back dan penutupan view. `get_ledger` dan `owner_key` tetap sama |
| `backend/app/engine/runner.py` | `process_run` memakai ExitStack; `target()` membuat indeks bernama dan mendaftarkan cleanup. Perhitungan source/fee/expense/transfer tidak berubah |
| `backend/app/reconciliation.py` | `fresh_balance()` memakai reader bernama dan menutup tiap observasi; pengulangan zero stabil serta aturan rekonsiliasi sama |
| `backend/app/config.py` | Batas batch 64.000 sel dan indeks temporer 64 MiB; cap HTTP 8 MiB serta nilai guard lama tidak dinaikkan |
| `backend/app/cli.py` | Alias pemeliharaan `prepare-v41-upgrade`, perilaku penahanan antrean lama tetap sama dan diaudit |
| `deploy/upgrade.sh`, `rollback.sh`, `app.env.example` | Rilis/backup V4.1, API saja setelah upgrade, worker manual; dokumentasi batas baru |
| `scripts/package.py` | Nama ZIP V4.1 |
| `backend/tests/helpers.py`, `range_api.py` | Adapter dan metadata sheetId pada transport uji |
| `backend/tests/test_v4_access.py` | Mengganti ekspektasi guard alokasi tujuan lama; seluruh tes sumber/proteksi lain dipertahankan |
| `backend/tests/test_migration.py` | Menambah kasus alias pemeliharaan V4.1 tanpa perubahan skema |
| `backend/tests/test_v41_destination.py`, `wide_api.py`, `fixtures/v4_grid_reader.py` (baru) | Fixture lebar lazy, oracle reader V4, koordinat/signature/output differential, semua tahap read/write/report dan memori |
| `backend/tests/performance/measure_destination.py`, `measure_v41_website.py`, `benchmark_memory.py` (baru) | Pengukuran fase, request GET/POST, byte respons, RSS/VmHWM dan proses terbatasi |
| `backend/tests/performance/measure_source.py`, `measure_website.py` | Hanya instrumentasi peak Linux: VmHWM agar tidak menghitung peak parent pytest yang diwariskan; batas tes tidak dilonggarkan |
| `README.md`, `docs/UPGRADE_V41.md`, `PERFORMANCE_V41.md`, kedua JSON V41, `TEST_REPORT.md`, `TEST_REPORT_V4.md`, `RELEASE_NOTES_V41.md`, `SCOPE_V41.md`, `V41_SPECIFICATION.md` | Panduan, laporan, bukti benchmark dan arsip V4 |

## Keselamatan mapping dan write

Koordinat pada indeks berasal dari `GridData.startRow/startColumn + offset`; tidak dinormalisasi terhadap slice. Row 74/column 1421 tetap **BBR75**, bukan offset lokal. Nama sheet tetap dikonfigurasi melalui URL/tab; sheetId diambil ulang dari metadata internal. Marker website duplikat, tanggal duplikat/hilang, boundary website tak terdaftar, header merged, account/alias dan kedua layout kiriman tetap memakai aturan V4.

Indeks berisi struktur yang dibutuhkan, bukan seluruh nilai transaksi. Sel yang tidak ada di indeks tidak dianggap blank untuk write. Nilai/formula/Note target dibaca eksplisit, dan merge global diperiksa. Signature struktur dihasilkan sebagai JSON stream yang **identik byte dengan fingerprint V4** pada fixture differential. Formula di seluruh sheet dibandingkan menggunakan hash canonical berurutan dengan koordinat dan count; perubahan formula di luar website aktif tetap terdeteksi.

Ledger ownership, PREPARED/VERIFIED, conflict manual, idempotency dan read-back tetap berlaku. Stop setelah POST tetap menyelesaikan verifikasi aman. `Laporan deposit harian` tetap read-only dan dibaca ulang melalui view baru untuk stabilitas zero; tidak ada nilai/formula yang dipaksa menjadi nol.

Source reader/parser V4, fungsi Sheets request/source/write, API routes, autentikasi, models, workflow, worker, batch/queue, job control dan frontend/aset **tetap identik**. Tidak ada migrasi skema, worker pool, perubahan MemoryMax, Nginx, SSL atau kredensial. API response fields dan fungsi finansial tetap sama.

## Hasil dan penerimaan

198 tes backend lulus pada Python 3.10/3.12; 5 tes Chromium lulus. Batas memori subprocess 1536 MiB, golden 22 output, far-right mapping, report lebar, kedua layout transfer dan proteksi global diuji. Lihat [TEST_REPORT.md](TEST_REPORT.md) serta [PERFORMANCE_V41.md](PERFORMANCE_V41.md) untuk angka aktual dan batas bukti.

Upgrade menahan antrean lama sesudah backup dan tidak memulai Run #7. Worker tetap berhenti sampai PREVIEW dan tinjauan operator. Live Google/workbook/VPS masih pending; tidak ada klaim produksi pasti berhasil berdasarkan fixture. Ikuti [UPGRADE_V41.md](UPGRADE_V41.md).

## Daftar lengkap fungsi pada modul reader baru

- `encode()`
- `FormulaSignature`: `__len__()`.
- `formula_signature()`
- `read_destination()`
- `close_view()`
- `MergeIndex`: `__init__()`, `__iter__()`, `__len__()`, `__eq__()`, `at()`.
- `_dispose()`
- `DestinationReader`: `__init__()`, `close()`, `refresh()`, `_sheets()`, `_merges()`, `_ranges()`, `_payload()`, `_cell()`, `_index_limit()`, `_header_entry()`, `_groups()`, `_responses()`, `_header_values()`, `_scan()`, `iter_cells()`, `_structural()`, `read_cells()`, `prefetch()`, `_size()`, `_remember()`, `cell()`, `inherited()`.
