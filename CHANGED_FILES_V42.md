# Perubahan per file — V4.1 → V4.2

Baseline review adalah source dalam ZIP V4.1 SHA256 `637e62b2f6af46fa8f1fde9f86c28e3c7455d29d3eec863d1b227daa81823344`. Repository GitHub menyimpan distribusi ZIP; untuk review source penuh, baseline diekstrak ke checkout Git terisolasi lalu dibandingkan dengan semua file source akhir. Index/HEAD workspace pengguna tidak diganti. Tidak ada source file dihapus.

Seluruh perbedaan distribusi source dicocokkan dengan whitelist manifest di bawah. Golden finansial V4.1, parser, SourceReader dasar, stylesheet, App/navigation dan semua tiga migrasi lama sama byte-for-byte. Screenshot lama dipertahankan; hasil browser tidak mengubah dokumen visual lama. Aset `public/` di ZIP dibangun dari source frontend terakhir dan tidak diedit manual.

| Status | File | Alasan |
|---|---|---|
| M | [README.md](../README.md) | Petunjuk V4.2, operasi, verifikasi, upgrade dan batas penerimaan live. |
| M | [backend/app/api.py](../backend/app/api.py) | DTO retry ber-idempotency/tab eksplisit dan refresh/validasi identitas sumber. |
| M | [backend/app/cli.py](../backend/app/cli.py) | prepare-v42-upgrade menahan antrean lama setelah backup tanpa menghapus intent. |
| M | [backend/app/config.py](../backend/app/config.py) | Batas snapshot 8 MiB/64 MiB per Run dan jumlah target registry; cap HTTP/RAM lama tetap. |
| A | [backend/app/conflicts.py](../backend/app/conflicts.py) | Observasi old/new aktual, keputusan immutable, approval stale, KEEP/NEW dan retained-cell aman. |
| M | [backend/app/destination_reader.py](../backend/app/destination_reader.py) | Indeks label fee rendah dan predicate huruf laporan yang sudah terbukti tanpa target GET berulang. |
| A | [backend/app/destination_registry.py](../backend/app/destination_registry.py) | Builder/version/target absolut bulanan, anchor/global validation, recovery proof dan metrik cold. |
| M | [backend/app/engine/report.py](../backend/app/engine/report.py) | Pisahkan resolusi koordinat BALANCE dari pembacaan angkanya; discovery huruf memakai proof indeks. |
| M | [backend/app/engine/resolver.py](../backend/app/engine/resolver.py) | Perbaikan hanya dua resolver fee untuk header rendah/merged dalam website sendiri. |
| M | [backend/app/engine/runner.py](../backend/app/engine/runner.py) | Coordinator snapshot→mapped result→writer, prioritas retry, dependensi dan slot aktual Stage C. |
| M | [backend/app/engine/writer.py](../backend/app/engine/writer.py) | Stable owner, grouped approval/readback, PREPARED dan no-repost untuk intent yang sudah cocok. |
| M | [backend/app/job_control.py](../backend/app/job_control.py) | Cancel/interruption mempertahankan antrean retry dan mengembalikan keputusan pending secara aman. |
| A | [backend/app/lineage.py](../backend/app/lineage.py) | Identity/binding berdasarkan spreadsheetId/sheetId; rename terbukti dan recreated yang nonaktif. |
| M | [backend/app/models.py](../backend/app/models.py) | 15 model additive untuk identity/index/snapshot/map/retry/decision/lineage/retained-cell. |
| M | [backend/app/operations.py](../backend/app/operations.py) | Registrasi mapping pada URL, antrean retry aktif, opsi sumber, endpoint konflik/legacy proof/progress. |
| A | [backend/app/quota.py](../backend/app/quota.py) | Pacing bersama lintas API/worker dengan file lock privat dan cancellation saat menunggu. |
| M | [backend/app/reconciliation.py](../backend/app/reconciliation.py) | GET BALANCE dari koordinat registry dengan validasi lama dan pengukuran tahap. |
| A | [backend/app/retry_queue.py](../backend/app/retry_queue.py) | Durable enqueue/claim/idempotency/cancel/restart, fairness boundary dan stale dependency notice. |
| M | [backend/app/sheets.py](../backend/app/sheets.py) | Metadata versi Drive opsional dengan key sama, telemetry/pacing dan inspeksi sebelum retry POST. |
| A | [backend/app/snapshots.py](../backend/app/snapshots.py) | Snapshot tanggal lengkap/checksum/provenance dan completeness/resource caps. |
| A | [backend/app/source_authorization.py](../backend/app/source_authorization.py) | Retry mengotorisasi konfigurasi sumber terkini hanya untuk item terpilih; histori Run tetap. |
| A | [backend/app/source_index.py](../backend/app/source_index.py) | Navigasi tanggal terikat versi/shape, cold proof bounded dan invalidasi/revalidasi. |
| A | [backend/app/telemetry.py](../backend/app/telemetry.py) | Timer exclusive/inclusive, request attempts/retries/bytes/RSS dengan data metrik bounded. |
| A | [backend/app/transfer_intents.py](../backend/app/transfer_intents.py) | Stable token/lineage, runtime slots, koreksi/retirement kelompok lama dan recovery. |
| M | [backend/app/worker.py](../backend/app/worker.py) | Pemulihan retry/mapping dan layanan builder queued sesudah API restart; tetap satu coordinator. |
| M | [backend/app/workflow.py](../backend/app/workflow.py) | Pin mapping/identity, context alias aktif setelah rename dan kelengkapan keputusan KEEP. |
| A | [backend/migrations/versions/c92448ce233b_v42_durable_snapshots_mapping_retries_.py](../backend/migrations/versions/c92448ce233b_v42_durable_snapshots_mapping_retries_.py) | 15 tabel/indeks baru, tanpa rebuild legacy; downgrade berisi data ditolak. |
| M | [backend/tests/conftest.py](../backend/tests/conftest.py) | Isolasi worker/mapping lock per test supaya proses dan dua versi Python tidak berbagi state. |
| M | [backend/tests/helpers.py](../backend/tests/helpers.py) | Fake sheetId konsisten dan bounded target wire GET, serta helper registrasi fixture. |
| A | [backend/tests/performance/measure_v42_website.py](../backend/tests/performance/measure_v42_website.py) | Benchmark cold/warm/retry website 18 Oktober dengan source besar dan tujuan lebar. |
| M | [backend/tests/test_cancellation.py](../backend/tests/test_cancellation.py) | Bedakan immediate readback success dari verification-unavailable PREPARED; cancellation tetap diuji. |
| M | [backend/tests/test_engine.py](../backend/tests/test_engine.py) | Lost reply yang terbukti langsung SUCCESS tanpa duplicate POST; modul lama tetap diuji. |
| M | [backend/tests/test_migration.py](../backend/tests/test_migration.py) | Upgrade populated/prepare V4.2, kosong downgrade, guard histori dan backup rollback. |
| M | [backend/tests/test_reconciliation.py](../backend/tests/test_reconciliation.py) | Otorisasi ulang fixture layout dan keputusan eksplisit ketika source identity resync berbeda. |
| M | [backend/tests/test_sheets_cells.py](../backend/tests/test_sheets_cells.py) | POST 503 tidak langsung diulang; GET retry/cancellation tetap dibuktikan. |
| M | [backend/tests/test_v3_source.py](../backend/tests/test_v3_source.py) | Fixture identity konsisten dan mapping diotorisasi setelah layout tanggal sengaja diubah. |
| M | [backend/tests/test_v41_destination.py](../backend/tests/test_v41_destination.py) | Wire target GET dan registry baru untuk fixture yang sengaja digeser; golden tetap. |
| A | [backend/tests/test_v42_conflicts.py](../backend/tests/test_v42_conflicts.py) | Actual old/new, stale/KEEP/NEW/Preview, Notes/formula, readback/timeout/CSRF dan PREPARED asing. |
| A | [backend/tests/test_v42_sources.py](../backend/tests/test_v42_sources.py) | Snapshot/index/identity lifecycle dan prioritas alias setelah rename. |
| A | [backend/tests/test_v42_workflow.py](../backend/tests/test_v42_workflow.py) | Active retry/race/queue/restart/dependency, fee, transfer, stale map, crash dan API subprocess. |
| M | [backend/tests/test_v4_access.py](../backend/tests/test_v4_access.py) | Repeated run tetap 22 rencana golden, 0 repeated POST dan retry hanya source terpilih. |
| M | [deploy/rollback.sh](../deploy/rollback.sh) | Terima backup upgrade-v4.2; tetap backup database baru dan pulihkan release/DB bersama. |
| M | [deploy/upgrade.sh](../deploy/upgrade.sh) | Deteksi release, backup/migrasi additive, API-only restart, worker held dan perintah verifikasi. |
| A | [docs/ARCHITECTURE_V42.md](../docs/ARCHITECTURE_V42.md) | Arsitektur, state/recovery, batas resource dan dependensi. |
| A | [docs/BASELINE_V42.md](../docs/BASELINE_V42.md) | Baseline 198 test dan temuan/rencana sebelum perubahan kode. |
| A | [docs/CHANGED_FILES_V42.md](../docs/CHANGED_FILES_V42.md) | Manifest perubahan per file beserta alasan dan baseline review. |
| A | [docs/COVERAGE_V42.md](../docs/COVERAGE_V42.md) | Bukti test untuk seluruh 80 persyaratan §21. |
| A | [docs/PERFORMANCE_FALLBACK_V42.json](../docs/PERFORMANCE_FALLBACK_V42.json) | Hasil aktual benchmark tanpa token versi. |
| A | [docs/PERFORMANCE_V42.md](../docs/PERFORMANCE_V42.md) | Waktu/counter tahap, metode, pacing dan keterbatasan live. |
| A | [docs/PERFORMANCE_WEBSITE_V42.json](../docs/PERFORMANCE_WEBSITE_V42.json) | Hasil aktual benchmark bertoken cold/warm/retry. |
| A | [docs/RELEASE_NOTES_V42.md](../docs/RELEASE_NOTES_V42.md) | Perilaku akhir dan penerimaan V4.2. |
| A | [docs/SCOPE_V42.md](../docs/SCOPE_V42.md) | Batas perubahan serta aturan/data yang dipertahankan. |
| M | [docs/TEST_REPORT.md](../docs/TEST_REPORT.md) | Hasil final 283 test pada dua Python, delapan browser cases dan risiko yang belum diverifikasi. |
| A | [docs/TEST_REPORT_V41.md](../docs/TEST_REPORT_V41.md) | Arsip laporan V4.1 sebelum laporan aktif diperbarui. |
| A | [docs/UPGRADE_V42.md](../docs/UPGRADE_V42.md) | Langkah VPS, additive migration, API-only acceptance dan rollback konsisten. |
| A | [docs/V42_SPECIFICATION.md](../docs/V42_SPECIFICATION.md) | Salinan spesifikasi lengkap pengguna untuk keterlacakan rilis. |
| A | [frontend/e2e/v42.spec.ts](../frontend/e2e/v42.spec.ts) | Tiga browser cases kontrak UI untuk retry saat aktif dan old/new/Preview. |
| M | [frontend/src/Dashboard.tsx](../frontend/src/Dashboard.tsx) | Kontrol item retry/sumber, progress/notices dan modal konflik; layout/navigation/CSS tetap. |
| M | [frontend/src/Settings.tsx](../frontend/src/Settings.tsx) | Status/polling registry dan masalah mapping pada pengaturan tujuan yang sama. |
| M | [frontend/src/api.ts](../frontend/src/api.ts) | Tipe DTO mapping/progress/retry/conflict yang dipakai kontrol V4.2. |
| M | [scripts/e2e-seed-v3.py](../scripts/e2e-seed-v3.py) | Fixture enqueue/cancel browser mempunyai identity dan registry V4.2; tidak menjalankan Google/worker. |
| M | [scripts/package.py](../scripts/package.py) | Nama paket/checksum V4.2, tetap mengecualikan runtime/private state. |
| M | [scripts/verify-real-run.py](../scripts/verify-real-run.py) | Readback nyata lewat registry yang dipin, laporan read-only dan cleanup view. |

A = file baru, M = perubahan pada file yang ada. Batas Google response/memori, service units, Nginx/HTTPS, credential path dan dependensi terkunci sama. File test lama dipertahankan; penyesuaian assertion/setup mengikuti perilaku V4.2 yang diotorisasi dan dijelaskan dalam TEST_REPORT.md. Tidak ada penghapusan ledger/audit atau fitur di luar ruang lingkup.
