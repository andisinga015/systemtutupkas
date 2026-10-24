# Laporan pengujian V4.1 — 23 Oktober 2026

V4.1 mengganti akses tujuan dan laporan yang padat menjadi reader terbatas dengan indeks struktur temporer. Pengujian memakai fixture terisolasi; pengembang belum mengakses key Google atau SSH VPS pengguna.

| Pemeriksaan | Hasil aktual |
|---|---|
| Backend Python 3.12.14 | **198 passed**, 76,76 detik |
| Backend Python 3.10.22 | **198 passed**, 88,61 detik |
| Chromium Playwright dengan API sebenarnya | **5 passed**, 34,6 detik |
| TypeScript strict / Vite build | Berhasil |
| Frontend sources dan seluruh 59 public assets | Identik byte dengan ZIP V4 |
| Source reader/parser, auth/API, models, workflow, worker, operations, batches, job control, diagnostics | Identik byte dengan ZIP V4 |
| Transport request/auth/source/write | AST fungsi identik V4; hanya jalur grid tujuan diganti |
| Alembic check/current | Tidak ada operasi baru; `b7c31ef05a92 (head)` |
| Maintenance V3/V4/V4.1 pada populated DB fixture | Akun/history/intents/ownership dipertahankan, penahanan antrean diaudit |
| Python/Bash syntax dan diff terhadap ZIP V4 | Lulus |
| Probe sparse/terformat dan satu website lengkap | Tanpa OOM di RLIMIT_AS 1536 MiB |

Tidak ada tes backend gagal, skipped atau xfailed pada kedua full-suite run. Satu `StarletteDeprecationWarning` mengenai httpx TestClient muncul; dependensi tidak diganti untuk patch ini. Lima tes browser mencakup autentikasi, konfigurasi, dashboard, riwayat, tanggal individual/batch dan cancel. UI tidak diubah, termasuk footer v3.0.

## Regresi yang dipertahankan

Seluruh 163 kasus V4 tetap dijalankan. Satu kasus tujuan yang dahulu mengharapkan guard berdasarkan alokasi diganti dengan pembacaan bounded yang berhasil; assertion guard sumber tidak dihapus. Tambahan 34 kasus reader V4.1 dan satu kasus CLI V4.1 menghasilkan 198 kasus total.

Golden 22 output V3/V4 tidak diganti: TOTAL DEPO, TOTAL WD, SALDO BANK, BIAYA, dua tab fee dan KET PENGIRIMAN dibandingkan untuk tab, alamat absolut, nilai, deskripsi, Note, bank dan identitas transaksi. Satu DOC DEPO dan dua DOC QRIS berjalan lewat transport sumber V4 serta tujuan lebar baru sampai SUCCESS dan rekonsiliasi zero. DEPO sebelum QRIS, prioritas bank, fee bertanda/newline, ownership slot dan overflow tetap diuji. Parser sumber, day discovery/carry-forward, error/missing/duplicate date dan Referensi 18 Oktober tetap lulus tanpa perubahan kode sumber.

## Tujuan lebar dan mapping

- Alokasi 1.000 × 1.000 dan 1.000 × 1.500 tidak ditolak; pemindaian hingga 1.800 kolom, 40+ blok dan marker dekat ujung diuji dengan fixture lazy, tanpa daftar sel untuk alokasi penuh.
- Row 74/column 1421 tetap **BBR75**. Situs tetangga dengan account mirip, boundary website tidak terdaftar, duplicate marker, tanggal hilang/duplikat/numeric, dan merged header lintas batas chunk diuji.
- Reader V4 lama diekstrak sebagai oracle offline. Mapping, struktur/signature dan kedua layout kiriman (calendar-date dan monthly side-total) dibandingkan langsung pada fixture identik.
- Request memakai rentang eksplisit dan sheetId sesuai. Byte cap tetap, batch over-cap dipecah bounded, merge metadata global difilter ke sheet tepat, struktur terlalu besar dan cancel gagal/berhenti secara akurat.

## Proteksi tulis dan laporan

Sel target diambil eksplisit untuk nilai/formula/Note; tidak ada asumsi blank akibat sparse index. Konflik manual value, Note, formula, merge, ownership, PREPARED uncertain source dan VERIFIED retry diuji. Perubahan header/merge/target antara planning dan pre-write terdeteksi. Formula jauh di luar website aktif yang berubah sesudah POST menggagalkan verifikasi global. Read-back nilai/Note mismatch menyisakan intent yang tepat untuk recovery; Stop setelah POST tetap menyelesaikan verifikasi sebelum cancellation berlaku kembali.

`Laporan deposit harian` tetap read-only. Marker horizontal/vertikal, tabel tersebar lebar, website associations, date, BALANCE hilang/duplikat dan stable exact zero melalui fresh reads diuji. Rekonsiliasi per-WEB/konsolidasi dan syarat Clear lama tetap lulus. Tidak ada POST tulis laporan atau formula yang dipaksa nol.

Direktori/file indeks temporer 0700/0600, cleanup sukses/error/cancel, cache count/byte serta batas memori proses diuji. Reader produksi tidak mempunyai `rows`; ordinary transaction payload dibuang setelah chunk. Global formula/struktur tetap diperiksa.

## Benchmark dan batas bukti

Probe sparse 1.000 × 1.500: 83 GET + 1 POST, peak proses **128,47 MiB**, target BBR75 VERIFIED. Versi seluruh blank terformat: peak **172,79 MiB**, respons terbesar 4.431.506 B, total 23,55 detik offline. Run penuh tiga tab sumber/22 output: peak **129,41 MiB**, SUCCESS, 1.319 request gabungan. Peak mencakup harness, bukan reader saja. RLIMIT_AS berbeda dari systemd MemoryMax. Rincian fase, instrumen, biaya global scan dan data mentah ada di [PERFORMANCE_V41.md](PERFORMANCE_V41.md).

Belum diuji live: workbook VELBETT/Run #7, FATMAWATI L24, seluruh 43 item aktual, respons/merge/Notes aktual Google, quota/latensi, RAM cgroup 1,5 GiB pada VPS, upgrade dan rollback systemd. Skrip upgrade diperiksa lokal; tidak ada klaim OOM produksi pasti hilang. Banyak request global dapat membuat run memerlukan puluhan menit pada throttle V4.

Upgrade membackup sebelum menahan antrean lama dan **tidak menjalankan ulang Run #7**. API dimulai, worker tetap berhenti. Ikuti [UPGRADE_V41.md](UPGRADE_V41.md) untuk pemeriksaan intent, PREVIEW baca saja, izin tulis operator dan rollback. Daftar file/fungsi di [RELEASE_NOTES_V41.md](RELEASE_NOTES_V41.md), gate di [SCOPE_V41.md](SCOPE_V41.md), laporan V4 di [TEST_REPORT_V4.md](TEST_REPORT_V4.md).

## Reproduksi

```bash
.venv/bin/pytest backend/tests -q
(cd backend && ../.venv/bin/alembic check)
npm --prefix frontend run build
PLAYWRIGHT_CHROMIUM_EXECUTABLE=/usr/bin/chromium bash scripts/e2e.sh
.venv/bin/python backend/tests/performance/measure_destination.py --width 1500 --formatted-blanks
.venv/bin/python backend/tests/performance/measure_v41_website.py --history 4000
bash -n deploy/upgrade.sh deploy/rollback.sh
python3 scripts/package.py
```
