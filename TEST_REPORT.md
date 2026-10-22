# Laporan pengujian V3 — 23 Oktober 2026

## Hasil

| Pemeriksaan | Hasil |
|---|---|
| Backend Python 3.12.14 | **137 passed** |
| Backend Python 3.10.22 | **137 passed** |
| Chromium Playwright + API sebenarnya | **5 passed** |
| TypeScript strict/Vite production build | Berhasil, aset hash |
| Alembic upgrade/check/current | `b7c31ef05a92 (head)`, tidak ada perubahan model tertinggal |
| Migrasi database v1/v2 berisi data | Lulus, original rows/columns tetap sama sebelum langkah pemeliharaan antrean yang disengaja |
| SQLite integrity dan foreign keys | `ok`, tanpa pelanggaran pada fixture migrasi |
| Syntax Bash/Python | Lulus |
| Help Center | 22 artikel dalam 12 kategori; alur/aturan sumber dan pembatalan diperbarui |

Satu StarletteDeprecationWarning terkait httpx TestClient tetap muncul pada pytest; tidak ada test gagal. Data palsu dan pengaturan browser hanya berada dalam pengujian, bukan implementasi processing aplikasi.

## Apa yang rusak dan dikoreksi

1. `Day.metric()` mencari header unik sebelum memeriksa hari kosong; runner mengulang header lookup untuk source_cell, biaya dan kiriman. V3 membaca tata letak SUMBER tetap sesuai DOC DEPO/QRIS. Tidak mengubah resolver tujuan, perhitungan fee atau formula rekonsiliasi.
2. Deteksi transaksi bergantung pada posisi header kumulatif. V3 menggunakan payload kolom sumber yang diketahui; total kumulatif, saldo, dekorasi, formula kosong dan nol template bukan transaksi. Missing/ambiguous date, invalid numeric/formula dan error Google tetap error.
3. Satu tanggal UI mengendalikan massal/individual. Sekarang tiap WEB memiliki tanggal/statusnya sendiri dan dialog Start menangkap tanggal eksekusi yang dikonfirmasi; respons tanggal lama tidak dipakai ketika tanggal berubah.
4. Tidak ada pembatalan dan unexpected exception dapat meninggalkan job aktif untuk restart berulang. V3 menambahkan cancel/stop persisten, finalisasi aman, penjagaan konflik selama CANCEL_REQUESTED dan retry item belum selesai. Canceled job tidak di-resume; error tak terduga menjadi FAILED/PARTIAL dengan catatan.

## Cakupan keuangan/sumber

- DEPO J/K/L, H dan I digunakan langsung walau header hilang, ganda atau menyesatkan. M/BALANCE tidak digunakan sebagai saldo bank.
- QRIS I/J/K, G dan H terpisah dari tata letak DEPO; header() dipasang sebagai fungsi yang menggagalkan test untuk membuktikan semua jalur sumber tidak memanggilnya.
- Fixture 18 Oktober: pemisah 18 pada baris 309 dan 19 pada baris 310; closing row 309. DEPO 0, WD 0, saldo 2.355.000 dari L306, biaya/fee kosong nol dan tidak menciptakan kiriman.
- Hari bertransaksi tetap memakai closing kumulatif nonnol; baris berikutnya tidak ikut dihitung. Blank total hari kosong dibedakan dari error formula, missing date, ambiguous date, grid sempit dan saldo rusak.
- Biaya H DEPO dijumlah bertanda lintas tab sebelum ABS. Biaya G QRIZZ/E-POWER tetap terpisah dari fee H. Multiple FEE QRIS PERIODE mempertahankan teks/newline, dijumlah pada tanggal transaksi; FEE WD AUTO sekali meskipun beberapa baris cocok. Zero fee tetap normal.
- Transfer I DEPO/H QRIS, bank dari nama tab DEPO, warna cokelat/biru yang sudah disetujui, overflow BCA→MANDIRI→BRI→BNI dan Note asli tetap diuji. Slot penuh, TOTAL, formula, Note manual, area merged dan konten asing tetap dilindungi.
- Read-back, ownership, duplicate prevention, uncertain intent, selective retry, URL revision, resync dan Clear seluruh WEB tetap lulus regresi V2.
- Laporan deposit harian tetap baca saja. Formula, STATUS, KETERANGAN, Note dan format fixture tetap utuh; nonnol kecil, error formula dan konsolidasi nol dengan WEB berselisih tidak menjadi Clear.
- Status dua tanggal diperiksa terpisah: tanggal 15 saldo nol/KLOP, tanggal 16 selisih -30.278.535/TIDAK KLOP, tanpa salah memakai record tanggal lain.

## Cakupan pembatalan/worker

- API admin/CSRF melindungi cancel. Antrean dapat langsung dibatalkan; preview yang dibatalkan tidak menulis atau membuat ledger finansial.
- Stop ketika grid sumber sedang dibaca tidak memulai item berikutnya.
- Stop ketika POST sudah dijalankan tetap melakukan read-back dan menyimpan VERIFIED. Retry tidak mengulang sel/item yang sudah sukses.
- Koneksi putus setelah tulisan diterima meninggalkan PREPARED/error yang dapat direkonsiliasi saat retry, bukan dihapus atau diubah menjadi sukses palsu.
- Cancel batch antre/berjalan, selective retry WEB tersisa, recovery cancellation request setelah restart, serta unexpected run/batch exception difinalkan tanpa RUNNING/QUEUED abadi.
- Retry GET boleh berhenti di antara attempt; POST yang telah dikirim tetap mengikuti retry/read-back bounded untuk memastikan hasil.
- Recheck saldo dibatalkan tanpa mengubah run finansial sukses. Pembatalan saat konsolidasi tidak memakai saldo hijau lama untuk mengizinkan Clear.
- Fixture V2 di-upgrade tanpa perubahan history/status asli oleh migrasi. Langkah prepare-v3-upgrade kemudian sengaja membatalkan pekerjaan belum selesai, mempertahankan run sukses, hash akun dan hasil yang tersimpan, serta mencatat audit.
- Helper rollback stdlib diuji pada snapshot skema V2; tidak membutuhkan model V3 atau memulai worker.

Perbandingan langsung dengan ZIP V2 juga lulus: resolver tujuan, resolver laporan dan aturan angka (`types.py`) identik byte demi byte. AST fungsi fees, signed_sum, eligible_description, parse_date, separator_dates dan check_write tetap sama. Perubahan kolom sumber dan kontrol eksekusi tidak mengganti aturan bisnis tersebut.

## Pengukuran pembacaan sumber

Fixture full run mencatat **1 grid DEPO + 2 grid QRIS = 3 pengambilan grid sumber untuk 17 item yang memakai sumber**. Semua modul tab memakai Day/grid yang sama; sumber yang gagal juga dibaca satu kali untuk konteks itu. Ini membuktikan tidak ada fetch ulang per metrik. Angka tersebut adalah jumlah panggilan `grid()` pada fake, bukan jumlah HTTP Google atau benchmark waktu live. V2 sudah memiliki cache tab; laporan ini tidak mengklaim percepatan live V3 terhadap V2. Pembacaan tujuan untuk proteksi dan read-back tetap dilakukan.

## Browser

Lima skenario menguji login/persist URL/validasi tanpa key, autentikasi, Help Center, sidebar desktop/mobile/zoom dan alur V3. Skenario V3 memeriksa tanggal massal tidak mengubah tanggal individual, tanggal VELBETT tidak mengubah FANTASI/massal, request individual membawa tanggalnya, status berubah mengikuti tanggal, batch memakai tanggal massal untuk kedua WEB, serta cancel preview/run/batch melalui API/UI nyata di detail dan riwayat.

Skenario antrean memakai fixture konfigurasi yang dinyatakan valid dalam DB temporer; tidak ada worker atau Google yang berjalan pada browser suite. Fixture itu dibatasi secara eksplisit ke `/tmp/tutupkas-e2e.*` dengan password test acak, dan tidak dapat digunakan pada DB operasional. Pemrosesan/write/stop running diuji terpisah melalui engine dan FakeSheets.

Viewport 1920×1080, 1366×768, 1280×720, 1024×768, mobile 390×844, dan CSS zoom Chromium 125/150% tetap lulus. CSS zoom bukan pengukuran native zoom seluruh browser. Screenshot pengujian ada di docs.

## Belum diverifikasi

**Tidak ada pengujian live crawl + write Google Sheets untuk V3 yang dijalankan pengembang.** Key/SSH VPS tidak tersedia di lingkungan ini. Kesesuaian workbook nyata, izin Google, nilai warna aktual, hasil formula aktual, instalasi/rollback systemd/Nginx/HTTPS dan cache domain harus diperiksa melalui smoke test pada VPS pengguna. Tidak ada klaim penulisan live berhasil berdasarkan fixture.

Panduan: [UPGRADE_V3.md](UPGRADE_V3.md). Jalankan pada salinan kerja yang Anda izinkan; source/destination tidak otomatis ditarik ketika URL disimpan/divalidasi. Rincian berkas/perbaikan: [RELEASE_NOTES_V3.md](RELEASE_NOTES_V3.md).

Reproduksi lokal:

```bash
.venv/bin/pytest backend/tests -q
(cd backend && ../.venv/bin/alembic check)
npm --prefix frontend run build
PLAYWRIGHT_CHROMIUM_EXECUTABLE=/usr/bin/chromium bash scripts/e2e.sh
bash -n deploy/*.sh scripts/*.sh
```
