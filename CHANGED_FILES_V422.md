# Perubahan per file V4.2.2

Basis: ZIP V4.2.1 SHA256 `67e8dc5656c398ee8cae63b892529799dd19588e7fe9f036fe4dc122c14e9180`. Semua perubahan berikut berkaitan dengan periode/otorisasi sumber dan pengiriman rilisnya.

| File produksi | Alasan |
|---|---|
| `backend/app/models.py` | SourceDocument period wajib, unique key periode dan check kombinasi valid |
| `backend/app/source_periods.py` (baru) | Kalender A/B deterministik dan label/identitas period, tanpa inferensi spreadsheet |
| `backend/app/api.py` | Save/history tiga sumber, legacy assignment, current-revision counts, exact doc/revision tab mutations dan stale metadata guards |
| `backend/app/workflow.py` | Filter sumber sebelum snapshot/items; period-aware signature/readiness; missing required half tidak fallback |
| `backend/app/source_authorization.py` | Retry exact source ID/period, aggregate legacy guard dan coherent period context signatures |
| `backend/app/snapshots.py` | Menyimpan period pada provenance saja; payload/perhitungan tidak diubah |
| `backend/app/operations.py` | Range resync lintas 15/16 mengambil masing-masing tanggal dari histori exact document records, bukan signature DOC A untuk DOC B |
| `backend/migrations/versions/d422a61e908f_source_periods.py` (baru) | Parent-table rebuild SQLite atomik, old DEPO UNASSIGNED/QRIS MONTHLY, ID/FK preservation, resume/rollback guards |
| `frontend/src/Settings.tsx` | Tiga URL cards/history, explicit legacy controls, isolated parent requests; menghapus lookup DEPO ambigu |
| `frontend/src/SourceTabs.tsx` (baru) | Tiga document choices, doc/revision-bound list/actions, checkbox reset dan stale/ABA response guards |
| `frontend/src/sourcePeriods.ts` (baru) | Descriptor/label/range kalender dengan last day aktual |
| `frontend/src/sourcePeriods.css` (baru) | Gaya terkait pilihan sumber/legacy assignment, mengikuti komponen lama |
| `frontend/src/api.ts` | Type metadata source period/revision/counts untuk UI |

| Delivery | Alasan |
|---|---|
| `deploy/upgrade.sh` | Deteksi release/schema V4.2/V4.2.1; stop memakai installed code sebelum schema baru, backup/migrate/verify V4.2.2; worker tetap berhenti |
| `deploy/rollback.sh` | Menerima backup prefix V4.2.2; rollback app+DB bersama tetap dipertahankan |
| `scripts/verify-v422-upgrade.py` (baru) | Verifikasi read-only semua protected old columns/financial history dan hanya perubahan migrasi/hold yang diizinkan |
| `scripts/package.py` | Nama ZIP V4.2.2 |

| Test/fixture | Alasan |
|---|---|
| `backend/tests/test_v422_periods.py` (baru) | Routing kalender, status independen, IDs/revision/races, snapshot/retry, index isolation, legacy/currentness/Crawl All dan performance gate |
| `backend/tests/test_v422_migration.py` (baru) | Populated graph migration/backup/hold/verifier, valid combinations/unique slots, downgrade refusal dan failure/resume |
| `backend/tests/performance/measure_v422_periods.py` (baru) | Oracle V4.2.1 vs V4.2.2, 43 item ditambah 10 unused active A tabs |
| `backend/tests/test_engine.py` | Fixture mendeklarasikan period secara eksplisit dan helper DOC B untuk regression dua tanggal |
| `backend/tests/writer_fixture.py` | Fixture Oct 18 mendeklarasikan SECOND_HALF; golden output tidak diubah |
| `backend/tests/performance/measure_v42_website.py` | Fixture Oct 18 juga memilih SECOND_HALF |
| `backend/tests/test_api.py`, `test_reconciliation.py`, `test_v42_workflow.py` | Payload simpan fixture menyertakan FIRST_HALF; assertion finansial tetap |
| `backend/tests/test_v3_source.py` | Fixture Oct 18 memakai B; regression 15/16 mempunyai kedua dokumen eksplisit |
| `backend/tests/test_migration.py` | Membandingkan seluruh kolom lama yang sama setelah penambahan period, ditambah assertion UNASSIGNED; tidak melemahkan preservasi |
| `frontend/e2e/application.spec.ts` | Selector judul DOC DEPO lama diganti nama DOC A yang diminta; skenario/assertion operasi tetap |
| `frontend/e2e/v422.spec.ts` (baru) | Sembilan skenario browser dokumen/status/race/period/legacy/website/revision, termasuk crafted request ditolak |
| `scripts/e2e-seed-v3.py` | Kedua periode eksplisit untuk regression enqueue tanggal 15/16 |
| `scripts/e2e-seed-v422.py` (baru) | Data browser terisolasi; guard hanya DB sementara E2E, tidak menghubungi Google |

README/active TEST_REPORT diperbarui; laporan V4.2.1 diarsipkan sebagai TEST_REPORT_V421. Docs V422 mencakup specification, baseline, architecture, 79-requirement coverage, performance JSON, upgrade/rollback dan known limitations. Semua screenshot dokumentasi lama dipertahankan.

Frontend prebuilt `public/index.html` dan hashed JS/CSS dibangun ulang dari source terkait; font/dependency locks tetap. Tidak ada perubahan pada seluruh `backend/app/engine/*`, destination registry/reader/Sheets transport, source reader/index/lineage, reconciliation, auth/session/CSRF, quota/locks/worker concurrency, dependency manifests, migrations lama, golden fixtures, systemd units, Nginx atau kredensial.
