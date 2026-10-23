# Tutup Kas Lion Group V4

V3 dapat OOM sebelum item pertama ketika membaca seluruh tab sumber terformat. V4 mengganti jalur itu dengan discovery nilai dalam chunk, konteks satu hari yang kecil, metadata format sel terpilih, batas respons streaming, dan pemulihan yang menunggu retry admin setelah crash. Tidak ada perubahan UI, kontrak API finansial, koordinat sumber, pemetaan tujuan, perhitungan, Note, atau syarat Clear.

## Berkas yang berubah

| Berkas | Perubahan |
|---|---|
| `backend/app/sheets.py` | Streaming respons dengan batas byte/waktu; batchGet nilai; helper range/field mask; pembacaan tujuan/laporan dalam band dan batas total sel |
| `backend/app/source_reader.py` (baru) | Discovery tanggal, validasi pemisah, selected-day payload, saldo backward bounded, format kandidat kiriman, sparse absolute rows |
| `backend/app/config.py` | Default dan validasi batas sumber, chunk, tujuan serta byte respons |
| `backend/app/engine/runner.py` | Menggunakan source_day; berbagi konteks antar modul satu tab; melepas konteks/rencana; konteks diagnostik VPS |
| `backend/app/diagnostics.py` (baru) | RSS/high-water Linux dan log worker rotasi privat, sekitar 20 MiB maksimum |
| `backend/app/job_control.py` | Menambahkan pengukuran RSS ke log fase yang sudah ada |
| `backend/app/worker.py` | Job terputus FAILED/PARTIAL tanpa auto-replay; cancellation tetap berlaku; mengaktifkan log rotasi |
| `backend/app/batches.py` | Pemulihan batch mempertahankan child SUCCESS/CANCELED dan tidak memutar ulang child terminal |
| `backend/app/cli.py` | Perintah pemeliharaan `prepare-v4-upgrade` setelah backup dan layanan berhenti; mempertahankan command V3 |
| `deploy/upgrade.sh`, `rollback.sh` | Rilis/backup V4; tahan antrean lama; API dimulai, worker tetap berhenti sampai penerimaan manual |
| `deploy/app.env.example` | Dokumentasi default batas; app.env VPS tidak ditulis ulang |
| `scripts/package.py` | Membuat ZIP V4 tanpa state/key/DB |
| `scripts/preview-source-v4.py` (baru) | Probe sumber referensi baca saja pada VPS, worker harus berhenti; tidak menggantikan PREVIEW seluruh WEB |
| `backend/tests/helpers.py` | Adapter source_day pada fake lama; jalur produksi tidak mempunyai fallback full-grid sumber |
| `backend/tests/test_worker.py`, `test_reconciliation.py` | Ekspektasi pemulihan baru: perlu retry manual, hasil sukses dipertahankan |
| `backend/tests/test_migration.py` | Membuktikan pemeliharaan V3/V4 menyimpan akun/history dan mencatat audit |
| `backend/tests/test_v4_access.py`, `range_api.py`, `performance/`, `fixtures/v3_financial_outputs.json` (baru) | API ranges, regresi output V3, crash setelah POST, preview, log, memori subprocess dan benchmark V3/V4 |
| `README.md`, `docs/UPGRADE_V4.md`, `PERFORMANCE_V4.md`, `PERFORMANCE_V4.json`, `PERFORMANCE_WEBSITE_V4.json`, `TEST_REPORT.md`, `TEST_REPORT_V3.md`, `RELEASE_NOTES_V4.md`, `V4_SPECIFICATION.md`, `SCOPE_V4.md` | Dokumentasi, bukti pengujian, arsip laporan V3 dan batas penerimaan |

Tidak ada migrasi DB baru. Frontend source, CSS, navigasi, label dan **seluruh 59 aset public** identik byte dengan V3. Footer tetap v3.0 sesuai batas tidak mengubah frontend. `engine/parser.py`, `resolver.py`, `writer.py`, `report.py`, `types.py`, `reconciliation.py`, `operations.py`, `workflow.py` dan `models.py` juga identik byte dengan ZIP V3.

## Pemulihan dan penerimaan

Upgrade tidak memulai ulang Run #5. Antrean lama yang belum selesai dibatalkan secara diaudit setelah backup; hasil dan write intents disimpan. Job yang terputus setelah V4 berjalan tidak diulang otomatis saat restart. Retry eksplisit tetap menggunakan read-before-write/read-back dan menolak uncertain write yang berubah, formula atau konten manual.

Lihat [UPGRADE_V4.md](UPGRADE_V4.md). Live Google/VPS masih pending, dimulai probe sumber dan PREVIEW penuh. Angka sekitar 45 MiB pada benchmark hanya untuk probe sumber sintetis, bukan ukuran seluruh worker produksi. Jumlah request discovery dapat naik karena chunk dan bukti keunikan; batas pemetaan tanggal tidak dipersempit secara diam-diam. Lihat [PERFORMANCE_V4.md](PERFORMANCE_V4.md) untuk pengukuran dan konflik optimisasi yang dihentikan.
