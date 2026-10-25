# Pengujian V4.2.2 — 25 Oktober 2026 (Asia/Hovd)

**414 passed pada Python 3.12.14 (388,27 detik), 414 passed pada Python 3.10.22 (481,12 detik), dan 17 passed pada Chromium Playwright.** TypeScript strict + Vite build lulus. Satu warning Starlette/httpx deprecation pada setiap suite backend; dependency locks tidak diubah.

Seluruh **343 kasus V4.2.1 dipertahankan**, ditambah **71 kasus backend** (57 source/routing/retry/performance dan 14 migration/recovery). Delapan browser cases lama tetap berjalan, ditambah sembilan kasus V4.2.2. Fixture lama kini menyatakan source_period dan kedua dokumen untuk skenario yang melintasi 15/16; tidak ada financial assertion/golden yang dihapus atau dilemahkan. Test migrasi membandingkan semua kolom lama, ditambah hasil field period baru.

## Hasil

| Pemeriksaan | Bukti |
|---|---|
| Baseline sebelum perubahan core | 343 passed, Python 3.12.14, 306,62 detik |
| Full backend final Python 3.12 | 414 passed, 388,27 detik |
| Full backend final Python 3.10 | 414 passed, 481,12 detik |
| Browser | 17 passed, sekitar 1,1 menit |
| Build frontend | TypeScript strict dan Vite lulus; public prebuilt disertakan |
| Migration current/check | d422a61e908f head; no new upgrade operations |
| SQLite populated graph | Semua original columns/IDs/FKs/history tetap; DEPO UNASSIGNED/QRIS MONTHLY |
| Upgrade hold/verifier | Protected financial state tetap; deliberate ledger corruption ditolak |
| Pre-stop pada schema lama | Embedded stop body dieksekusi dengan source V4.2.1 asli pada c924; recheck batch QUEUED/RUNNING aman, Run SUCCESS tetap; post-migration verifier lulus |
| Failure/recovery | Parent rebuild gagal rollback atomik; completed schema dapat resume sebelum revision stamp |
| Backup rollback | Original database/schema/data dipulihkan utuh; assigned-period destructive downgrade ditolak |
| Runtime source routing | A10/B18 hanya sumber relevan + monthly QRIS; unused active period tidak punya item/read/snapshot |
| Writer preservation | Semua engine/destination reader/registry/Sheets transport byte-identik V4.2.1 |
| Live Google/VPS/systemd execution | Pending; tidak ada akses/tulisan produksi |

## Cakupan sumber dan UI

- Tiga independent URL saves, idempotency, validation/history/refresh, actual last day untuk Februari/leap year/April/Oktober, website/month/year context terpisah.
- Tiga pilihan sheet aktif/nonaktif, nama sama A/B dengan lineage berbeda, actual server states dan bulk activation/deactivation doc ID tepat, current revision counts, inactive/new-month/new-URL defaults.
- Cross-document, cross-revision, unavailable/identity-conflicted tab IDs ditolak; metadata respons lama setelah URL berubah tidak menandai URL baru valid. Source edits dan tab mutations memakai SQLite serialization/currentness checks.
- Checkbox/action reset saat dokumen/bulan/tahun/website/revision berubah. Browser menguji delayed A→B dan A→B→A dengan transport yang sengaja mengabaikan abort; daftar terbaru tetap benar. Missing B tidak menggunakan A dan action disabled.
- Hari 1/15/16/18/28/29/30/31, optional QRIS absent vs configured invalid, required half missing/no usable active tabs, exact selected source snapshot/item generation tanpa double BIAYA/kiriman.
- Date Index A row 290 dan B row 340 memakai identity/doc berbeda meski version token sama. Reader/index/separator financial implementation lama tidak diubah.
- Failed-item retry A10/B18/QRIS hanya membaca fresh source yang tepat; successful result JSON, snapshot lain dan mapping tetap. Legacy item/BIAYA total tidak dapat melewati explicit period assignment atau berpindah A↔B berdasarkan nama.
- UNASSIGNED migration, assignment A/B retaining same ID/revisions/active/disabled states, occupied slot/stale assignment rejected, historical Run JSON tetap readable tanpa mutation.
- Source signature/currentness tidak berubah akibat edit half yang tidak dipilih. Explicit resync range memakai histori masing-masing tanggal/documents. Crawl All memproses valid WEB walaupun WEB lain missing B.

Pemetaan semua nomor 1–79 terdapat pada [COVERAGE_V422.md](COVERAGE_V422.md). Semua regression finansial V4.2.1 ikut berjalan: fixed columns/carry-forward/fees/colors/transfers, Notes/formula/merge/manual protection, NEW/KEEP, ownership/duplicate protection, accepted-POST crash/timeout recovery, Preview nol financial writes, report read-only, true KLOP/Clear, retry claims/flock/auth/CSRF dan populated legacy migrations.

## Benchmark yang diukur

Oracle adalah ZIP V4.2.1 SHA256 `67e8dc5656c398ee8cae63b892529799dd19588e7fe9f036fe4dc122c14e9180`. Fixture memakai 8 snapshot, 43 plans, 57 cells dan 13 kiriman; sesudah patch ada 10 unused active DOC A tabs.

| Ukuran | V4.2.1 | V4.2.2 |
|---|---:|---:|
| Destination GET | 77 | 77 |
| Financial POST | 1 | 1 |
| Destination response bytes | 117.986 | 117.986 |
| Unused period source reads | 0 | 0 |
| GET ≥2 MiB | 0 | 0 |
| Wall clock offline, ms | 1.329,188 | 1.293,431 |
| Peak RSS, MiB | 152,89 | 157,32 |

Seluruh koordinat, nilai, deskripsi, Notes, bank, transaksi dan 43 status item sama persis. Output SHA `32c0ec33ef1164c5d971b66a16d6cd74ab842e6155053b5a34e48a59fc93f095`. Fallback tanpa token: 114 GET/7 POST/221.906 byte, 1.527,273 ms, peak 157,38 MiB, himpunan output/status tetap sama. Semua probe memakai RLIMIT_AS 1536 MiB. RSS sesudah mencakup tambahan cloned source grids/state fixture; kenaikan tidak disamarkan sebagai nol.

Per-stage JSON dan metode tersedia pada [PERFORMANCE_V422.md](PERFORMANCE_V422.md). Source Google network/cold registration/7.936 target produksi tidak diukur. Fixture tanpa latency/pacing bukan janji waktu Google. Default 1,2 detik tetap; 78 destination requests saja memerlukan sekitar 92,4 detik jarak start sebelum source/latency. Tidak ada klaim live <60 detik atau OOM mustahil untuk semua workbook.

## Batas dan penerimaan

Schema/migrasi khusus SQLite. Invalid legacy data/FK atau source identity yang belum terbukti tetap memerlukan pemeriksaan, tidak ditebak/dihapus. Proven rename menjaga lineage/history tetapi nama baru tetap memerlukan aktivasi Admin sesuai aturan lama. DEPO belum ditetapkan tidak dipilih sebagai sumber normal/retry. Source separator berikutnya wajib terbukti; tidak ada boundary buatan pada 15/16/akhir bulan.

Penerimaan VPS/systemd/Nginx dan Google asli, quota, concurrent external editors dan kondisi produksi masih pending. Sheets tidak menyediakan compare-and-set pada celah GET→POST; guard/readback V4.2.1 tetap dipertahankan. Rollback SQLite tidak menghapus perubahan Google setelah backup. Worker harus tetap berhenti sampai settings, legacy assignment, PREPARED intents dan antrean ditinjau.

Integration probe tambahan `UPGRADE_PROBE_V422.json` menguji exact pre-stop body skrip pada schema c924 dengan modul original V4.2.1, lalu migration/hold/verifier baru. Model, workflow dan cancellation code original V4.2 sama dengan V4.2.1. Pre-stop sengaja memakai kode terpasang, sementara post-migration memakai kode baru; pengujian ini tidak menjalankan systemd/Nginx asli.

Paket lengkap diperiksa scope/CRC/private-state exclusions, prebuilt assets, dan extraction smoke tests/migration check sebelum publikasi. Checksum terpisah disertakan. GitHub artifact diverifikasi lewat fresh fetch serta HTTP download dengan checksum sama; arsip lama dipertahankan. Laporan V4.2.1 asli ada pada [TEST_REPORT_V421.md](TEST_REPORT_V421.md).
