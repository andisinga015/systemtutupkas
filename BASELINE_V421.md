# Baseline V4.2.1 — 25 Oktober 2026 (Asia/Hovd)

Patch dikerjakan terhadap source ZIP V4.2 yang sudah dirilis, bukan hasil rekonstruksi writer lama dari patch baru.

| Bukti | Nilai |
|---|---|
| Parent GitHub V4.2 | `7fa006e25a458842ffbd6befbfb9a6c2a5d7a079` |
| Arsip | `tutupkas-lion-group-v4.2.zip` |
| SHA256 | `35eb89eea6da7fc05e200ccd807b8607042b07ac8804e2c4ec65e11ddf36dc45` |
| Schema sebelum/sesudah | `c92448ce233b` |
| Suite sebelum perubahan | 283 passed, Python 3.12.14, 168,92 detik |
| Runtime pembanding | Python 3.12.14; dependensi terkunci yang sama |

Laporan Run #9 pengguna: VELBETT 18 Oktober 2026, 8 snapshot, 43 rencana, mapping #4 READY, 8 tab dan 7.936 target absolut; `expense_total/CHECKING_DESTINATION` berulang GET sekitar 7,7–8,3 MB, RAM sekitar 138 MB, tanpa OOM. Database/log request lengkap/VPS/Google produksi tidak tersedia di lingkungan pengembangan. Angka itu merupakan observasi pengguna, bukan benchmark live yang dilakukan di sini.

Fixture pembanding menggunakan API registrasi/identitas/READY mapping asli dan 8 sumber (6 DEPO, 2 QRIS), 43 item finansial, 57 update sel dan 13 kiriman. BIAYA berukuran 256 × 64 dengan formatted blanks dan Note historis panjang di luar sel target. Fixture mereproduksi respons besar yang diminta lalu dibuang oleh indeks struktur. Fixture tidak meniru seluruh 7.936 target produksi. Cold registration dikerjakan sebelum pengukuran; Stage A/B/C Run tetap dijalankan. Sumber/Google memakai transport terisolasi, tanpa latensi/pacing/quota nyata.

Baseline yang diukur ulang langsung dari ZIP asli: **660 GET, 36 POST, 100.971.103 byte respons tujuan**, 12 GET minimal 2 MiB, 3.024,637 ms, peak RSS 155,29 MiB. Termasuk byte percobaan respons yang ditolak batas 8 MiB. Lihat JSON dan rincian per tahap pada [PERFORMANCE_V421.md](PERFORMANCE_V421.md).

Seluruh 283 test lama tetap ada tanpa perubahan. Source snapshots, date index, identitas/registrasi tab, parser/perhitungan, resolver, models/migrations, autentikasi, UI, dependency locks dan unit layanan dibandingkan dengan arsip V4.2. Daftar perubahan patch terdapat pada [CHANGED_FILES_V421.md](CHANGED_FILES_V421.md).
