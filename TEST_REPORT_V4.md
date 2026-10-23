# Laporan pengujian V4 — 23 Oktober 2026

V4 adalah koreksi backend pembacaan Google Sheets/memori. Pengujian memakai fixture terisolasi; belum ada akses key/Google/SSH VPS produksi.

| Pemeriksaan | Hasil |
|---|---|
| Backend Python 3.12.14 | **163 passed** |
| Backend Python 3.10.22 | **163 passed** |
| Chromium Playwright + API sebenarnya | **5 passed** |
| TypeScript strict / Vite build | Berhasil |
| Source frontend/CSS dan 59 aset public | Identik byte dengan ZIP V3 |
| Parser, resolver tujuan/laporan, writer, types, reconciliation, operations, workflow, models | Identik byte dengan ZIP V3 |
| Alembic check/current | Tidak ada operasi baru, `b7c31ef05a92 (head)` |
| Migrasi populated fixture / pemeliharaan antrean V3 dan V4 | Akun, history, hasil dan ownership dipertahankan; cancellation diaudit |
| Syntax Python/Bash dan full Git diff review terpisah | Lulus |
| Streaming response bounds, recovery setelah crash, log privat/rotasi | Lulus |
| Probe sumber dan satu website, RLIMIT_AS 1536 MiB | Lulus, tanpa OOM |

Satu StarletteDeprecationWarning mengenai httpx TestClient masih muncul. Dependensi tidak diganti untuk patch ini.

## Regresi finansial

Golden output diekspor dengan menjalankan engine V3 dari ZIP terkirim SHA256 `2437bc71fd4369b4525700e3f3bdfd8239d650abfaa1ec5173b68b640c68bc65`, pada satu DOC DEPO dan dua DOC QRIS. V4 dibandingkan terhadap **22 cell writes**: tab/alamat/nilai/deskripsi/Note/bank/fingerprint transaksi sama persis. Penggantian sumber dengan ranges API V4 menghasilkan output yang sama. Retry satu item hanya membaca sumber yang diperlukan.

Parser V3 juga dibandingkan langsung dengan source_day untuk hari aktif/kosong, QRIZZ/E-POWER, biaya bertanda, multi-fee dan brown/blue. Referensi 18 Oktober memberikan **0 DEPO, 0 WD, saldo 2.355.000**, dengan sumber saldo pada baris absolut yang sama. Numeric date diformat, missing/duplicate/ambiguous date, formula error dan nominal rusak tetap dibedakan dari hari kosong. Batas resource menghasilkan error, tanpa memotong transaksi atau nol palsu.

Seluruh cakupan V3 tetap dijalankan: account/WEB/date matching, DEPO J/K/L dan QRIS I/J/K, signed biaya, fee/newline asli, Note QRIS, DEPO sebelum QRIS, priority BCA→MANDIRI→BRI→BNI, slot overflow, formula/merge/manual-note conflict, ownership, idempotency, selective retry/resync, read-back, laporan read-only, exact zero per-WEB/konsolidasi dan Clear. Rincian V3 diarsipkan dalam [TEST_REPORT_V3.md](TEST_REPORT_V3.md).

## Resource dan pemulihan

Tes memastikan discovery values-only bounded, metadata format sel tunggal, konteks tab dipakai bersama, serta kegagalan dicache tanpa traceback respons besar. Band tujuan mempertahankan setiap nilai, formula, Note, warna dan merge. Respons terlalu besar diputus saat streaming dan ditutup; body parsial dilepas. Disconnect body menggunakan retry terbatas. `Accept-Encoding: identity` menghindari buffer dekompresi besar sebelum byte cap. Log privat 0600 diuji dengan rotasi 5 MiB dan tiga backup; tidak ada metrik UI.

Di bawah RLIMIT_AS 1536 MiB, histori 4.000→40.000 baris mempertahankan peak probe sumber sekitar **45 MiB** dan konteks tiga baris untuk hari kosong. Run lengkap tiga tab sumber mencapai sekitar **129 MiB**, SUCCESS dengan 22 output sesuai acuan. Angka/durasi/calls lengkap tersedia di [PERFORMANCE_V4.json](PERFORMANCE_V4.json) dan [PERFORMANCE_WEBSITE_V4.json](PERFORMANCE_WEBSITE_V4.json). Ini benchmark offline tanpa jaringan/throttle Google, bukan ukuran worker produksi.

Tes interruption setelah POST fixture diterima menyisakan PREPARED. Dua recovery/start berikutnya tidak membaca/menulis ulang; retry admin memeriksa tujuan lalu menyelesaikan verifikasi. Run/batch terputus menjadi FAILED/PARTIAL dan hasil sukses disimpan. CANCEL_REQUESTED diselesaikan CANCELED. Stop saat POST sudah dikirim tetap menyelesaikan read-back; retry tidak mengulang item sukses. CLI prepare-v4-upgrade mempertahankan akun/history/intents dan mencatat audit setelah pemeliharaan.

## Batas bukti dan target pending

Jumlah request total meningkat karena chunk. Discovery masih menelusuri nilai global demi keunikan separator V3; larangan literal membaca semua nilai histori belum terpenuhi pada tahap discovery. Pembatasan kolom marker lebih agresif dihentikan karena dapat mengubah aturan tanggal. Lihat [PERFORMANCE_V4.md](PERFORMANCE_V4.md) dan [SCOPE_V4.md](SCOPE_V4.md); tidak ada klaim semua target sudah dipenuhi.

Belum diuji live: workbook referensi Run #5, seluruh 43 item website, Notes/warna/merge/formula aktual, quota/latensi, RSS cgroup worker pada VPS, instalasi systemd/rollback serta cache domain. Tujuan fixture kecil; memori semua ukuran tujuan produksi belum terbukti. Skrip upgrade diperiksa syntax/diff, belum dijalankan pada VPS pengguna. Tidak ada klaim OOM produksi definitif hilang.

Ikuti [UPGRADE_V4.md](UPGRADE_V4.md): backup, tahan antrean lama, API dimulai tetapi worker tetap berhenti, probe sumber baca saja, PREVIEW penuh, tinjau intent, lalu operator secara eksplisit mengizinkan penulisan. Daftar perubahan ada di [RELEASE_NOTES_V4.md](RELEASE_NOTES_V4.md).

Reproduksi:

```bash
.venv/bin/pytest backend/tests -q
(cd backend && ../.venv/bin/alembic check)
npm --prefix frontend run build
PLAYWRIGHT_CHROMIUM_EXECUTABLE=/usr/bin/chromium bash scripts/e2e.sh
.venv/bin/python backend/tests/performance/measure_source.py --history 40000 --limit-mib 1536
.venv/bin/python backend/tests/performance/measure_website.py --history 40000
bash -n deploy/*.sh scripts/*.sh
python3 scripts/package.py
```
