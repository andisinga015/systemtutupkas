# Daftar perubahan V4.2 → V4.2.1

Scope produksi **7 file**, tanpa schema/perubahan finansial:

| File | Alasan |
|---|---|
| `backend/app/engine/batch_writer.py` (baru) | Window bounded, shared preflight, checksum/source guards, final actual target GET, atomic POST dan closed-epoch readback per kelompok |
| `backend/app/engine/writer.py` | Ekstraksi validasi grup dan defer-submit setelah PREPARED; guard/verify lama digunakan ulang |
| `backend/app/engine/runner.py` | Pemanggilan coordinator pada mapped financial run, bounded KET planner prefetch dan penundaan QRIS transfer retry sampai DEPO PENDING selesai |
| `backend/app/retry_queue.py` | Filter dependency sebelum klaim atomik, tanpa perubahan protokol/klaim/concurrency |
| `backend/app/destination_registry.py` | Refresh durable signatures, conditional field projection, shared properties/merge epoch proof; otorisasi/koordinat registry tetap |
| `backend/app/destination_reader.py` | Parameter invariant validation; full empat baris awal, conditional body field mask dan exact full fallback; scan predicate/formula global tetap |
| `backend/app/sheets.py` | Transport atomic write_batch, duplicate/resource guards, request shape diagnostics dan byte telemetry partial/rejected |

Delivery **4 file**: `deploy/upgrade.sh` (safe V4.2→V4.2.1, no-op schema, hold jobs, worker stopped), `deploy/rollback.sh` (prefix backup V4.2.1), `scripts/package.py` (nama ZIP), `scripts/verify-v421-upgrade.py` (baru; read-only protected DB comparison).

Test/fixture **3 file baru**: `backend/tests/test_v421_writer.py`, `backend/tests/writer_fixture.py`, `backend/tests/performance/measure_v421_writer.py`. Semua file test V4.2 lama tetap byte-identik; tidak ada test lama dihapus/dilemahkan.

Dokumentasi: README, active TEST_REPORT, arsip TEST_REPORT_V42, specification V4.2.1, BASELINE/ROOT_CAUSE/INVARIANTS/PERFORMANCE/UPGRADE/RELEASE_NOTES/CHANGED_FILES V421, serta tiga JSON benchmark sebelum/sesudah/fallback. Documentation screenshots lama dipertahankan byte-identik.

Audit terhadap ZIP V4.2 memastikan **tidak ada perubahan** pada model/migrasi/schema, parser/resolver/report/reconciliation, source reader/date index/snapshot/identity/tab registration, auth/session/CSRF/API UI, frontend source/prebuilt, dependency manifests/locks, golden fixtures, quota/locks/worker concurrency, install script, unit systemd dan konfigurasi Nginx. Source registration/mapping builder tidak dibangun ulang sebagai langkah upgrade.

Daftar per-file/hash aktual juga dapat diverifikasi langsung dengan membandingkan dua ZIP lengkap di GitHub. Rilis V4.2 sebelumnya tidak ditimpa.
