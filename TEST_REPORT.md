# Laporan pengujian v2 — 22 Oktober 2026

## Hasil lokal

| Pemeriksaan | Hasil |
|---|---|
| Backend Python 3.12.14 | **106 passed** |
| Backend Python 3.10.22 (kompatibilitas VPS) | **106 passed** |
| Chromium Playwright dengan API sebenarnya | **4 passed** |
| TypeScript strict + Vite build | Berhasil; aset JavaScript/CSS memakai hash |
| Alembic `check` dan `current` | Tidak ada perubahan model tertinggal; `daf814b2a736 (head)` |
| Migrasi DB v1 berisi data | Berhasil; semua kolom/baris asli pada tabel yang diisi tetap identik |
| SQLite integrity dan foreign_key_check setelah migrasi | `ok`, tanpa pelanggaran |
| Registry panduan | 21 artikel, 12 kategori; related/context links dan query wajib valid |
| Bash syntax + Python compile | Berhasil untuk skrip deploy/setup dan modul aplikasi |

Pytest menghasilkan satu StarletteDeprecationWarning tentang httpx TestClient, tanpa test gagal. Dependensi aplikasi tidak diubah untuk menyembunyikan peringatan itu.

Perintah reproduksi:

```bash
.venv/bin/pytest backend/tests -q
(cd backend && ../.venv/bin/alembic check)
npm --prefix frontend run build
PLAYWRIGHT_CHROMIUM_EXECUTABLE=/usr/bin/chromium bash scripts/e2e.sh
bash -n deploy/*.sh scripts/*.sh
```

## Cakupan backend

- Sesi admin, CSRF/origin, logout, expiry, limiter, lifecycle WEB, revisi sumber, validasi/tab dan perbedaan bulan.
- Aturan sumber dan ketujuh modul tetap diuji, termasuk closing/carry-forward, biaya bertanda, fee, parsing tanggal, slot kiriman bank, warna, formula kosong, Note manual, sel merged dan konflik sumber.
- Baca ulang tulisan, intent setelah transport crash, kegagalan satu modul, retry selektif, dependensi agregat dan pencegahan append/duplikasi.
- Laporan VELBETT/FANTASI/BBTOTO pada blok horizontal bergeser/bersebelahan dengan kolom kosong; penanda case/whitespace/newline, huruf vertikal, merged label serta tabel bertumpuk. Marker hilang, blok ganda, header/date hilang/ganda dan konsolidasi tidak unik harus berhenti.
- Nol numerik serta teks nol/desimal/negative zero diterima; selisih sekecil `0.000000000001` tetap nonnol. Kosong/bool/formula error/NaN/Infinity ditolak. Nilai formula stale dicoba ulang, dan satu pengamatan nol terakhir saja belum cukup.
- Laporan hanya dibaca: nilai/formula/Note/warna/STATUS/KETERANGAN tetap identik, tidak ada ledger laporan, writer dan SheetsClient menolak penulisan sebelum HTTP.
- Penulisan SUCCESS dapat mempunyai rekonsiliasi NOT_BALANCED; error per WEB memuat pesan, saldo/selisih dan sel yang benar. Konsolidasi nol tidak menjadikan status keseluruhan KLOP bila satu WEB tidak klop. Ledger belum VERIFIED atau sumber parsial tidak dapat Clear.
- Recheck saldo menggunakan job persisten dan hanya membaca laporan; jumlah write/transfer/Note tetap. Koreksi menjadi nol menyelesaikan error rekonsiliasi tanpa replay tulisan.
- Crawl All memakai engine nyata terhadap FakeSheets: dua WEB aktif berurutan, nonaktif dikecualikan, progress aktual, partial success karena konfigurasi hilang, retry hanya WEB gagal dan melanjutkan entry setelah restart. Race individual vs batch diuji dengan dua session/thread SQLite; hanya satu start diterima.
- URL baru mereset validasi, mempertahankan snapshot/history/ledger lama, tidak membuat run saat Save/Validate, menandai Perlu Sinkronisasi dan menulis ke tujuan baru hanya melalui resync manual. Source baru mereset tab; resync sumber yang sama mengadopsi ownership yang sah tanpa transfer duplikat, termasuk rentang tanggal dengan riwayat masing-masing tanggal. Dua URL lama tidak dipilih secara arbitrer dan bulan lain tetap terpisah.
- Migrasi v1 membandingkan data asli pada akun, sesi, WEB, source/revision/tab, destination/history, run/item/error, ledger, transfer ownership, Clear lama, alias, validation dan audit. Hanya bulan yang memiliki satu tujuan lama diadopsi; dua URL lama tetap menunggu pilihan admin. Clear global baru tetap kosong.
- Worker subprocess benar-benar dijalankan untuk memulihkan job terotorisasi dan mencatat key tidak tersedia; lock menolak worker kedua. Recovery batch diuji dari state RUNNING persisten tanpa mengulang entry sukses.

FakeSheets hanya ada pada kode pengujian. Pengujian engine ini tidak menggunakan API Google asli dan bukan bukti kesesuaian workbook asli.

## Cakupan browser

Empat skenario menjalankan UI React dengan FastAPI dan DB temporer sebenarnya, bukan respons API tiruan:

1. Login, tambah WEB, simpan/persist URL, kegagalan validasi tanpa key, daftar sheet, history, pemblokiran Start tanpa tujuan dan tampilan satu tujuan bulanan.
2. Password salah ditolak; tidak ada registrasi publik.
3. Help Center dari sidebar, search isi/keywords case-insensitive untuk `validasi`, `sheet aktif`, `crawl semua`, `balance tidak klop`, `url baru`, hasil kosong, navigasi artikel dan bantuan kontekstual.
4. Sidebar pada 1920×1080, 1366×768, 1280×720, 1024×768, zoom CSS Chromium 125%/150% dan mobile 390×844. Scroll sidebar tidak memindahkan halaman utama, Help terjangkau, tidak ada overflow horizontal, focus trap/restore/Escape/close berfungsi. CSS zoom menguji tekanan layout; ini bukan pengukuran zoom native semua browser.

Screenshot tersimpan di folder docs. Browser E2E tidak menulis Google; alur finansial penuh diuji pada integrasi backend terhadap FakeSheets.

## Belum dijalankan dari lingkungan pengembang

| Pemeriksaan | Status |
|---|---|
| Akses service account dan workbook Google aktual | Belum tersedia di cloud; key tetap pada VPS pengguna |
| Kecocokan struktur, warna, rekening dan formula workbook aktual | Memerlukan smoke test pada salinan kerja yang pengguna otorisasi |
| Penulisan/read-back/Note pada Google asli | Belum dijalankan oleh pengembang untuk v2 |
| Eksekusi upgrade/rollback sebagai root pada VPS sebenarnya | Belum dijalankan; skrip diperiksa sintaks dan panduan disediakan |
| Nginx, Certbot, domain HTTPS dan cache aktual | Tidak diubah atau diuji dari cloud |
| Restore cloud pada task baru setelah Publish | Belum diverifikasi; konfigurasi tersimpan sebagai draft |

Ikuti [UPGRADE_V2.md](UPGRADE_V2.md) untuk backup, staging, migrasi, cache, health, smoke test, verifikasi Google hanya baca dan rollback. Jangan menganggap test lokal sebagai bukti deployment berhasil. Pengguna dapat menjalankan `scripts/verify-real-run.py RUN_ID` pada VPS setelah Start nyata yang disetujui; helper hanya membaca dan membandingkan ledger serta laporan, tanpa membuat tulisan atau otomatis Clear.
