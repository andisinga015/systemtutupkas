# Tutup Kas Lion Group V4.2

Satu snapshot sumber lengkap dipakai semua modul pada tanggal/tab yang sama. Indeks tanggal persisten memakai bukti versi isi untuk navigasi aman. Registry bulanan menyimpan koordinat absolut tujuan saat URL disimpan/divalidasi, lalu Crawl/retry memvalidasi target yang sama tanpa memetakan ulang.

Retry satu item dapat diantrikan ketika Run aktif, memakai raw sumber terkini, dengan claim/idempotency persisten. Item sukses tetap disimpan; perubahan input BIAYA menandai dependensi total untuk otorisasi terpisah. Identitas tab mengikuti spreadsheetId/sheetId dan rename terbukti; recreated atau legacy tanpa bukti tidak ditebak.

Konflik menampilkan kelompok data Google aktual dan hasil sumber. Admin dapat memilih Masukkan Data Baru atau Tetapkan Data Lama; persetujuan stale ditolak dan error hanya ditutup sesudah verifikasi. Formula, merge, Note manual, Preview dan ledger PREPARED tetap dilindungi. Resolver fee dibatasi dua modul fee, mendukung header lebih rendah/merged; no-fee kosong menjadi no-op, sedangkan fee lama memerlukan konflik/keputusan.

Kiriman tetap DEPO sebelum QRIS dengan urutan bank/warna/nominal/Notes lama. Slot aktual diperiksa di batas tulis; transaksi berubah/hilang dikoreksi pada kelompok slot lama setelah persetujuan. Kelompok accepted POST dengan reply hilang diverifikasi sebelum pengiriman lain; recovery tidak menghapus histori token/ownership. Rekonsiliasi individual/konsolidasi tetap GET dan exact-zero.

Migrasi menambah 15 tabel tanpa mengubah tabel lama. Upgrade V4.1 membackup DB/aplikasi/frontend dan memulai API saja; worker ditahan untuk penerimaan manual. Tidak ada perubahan stylesheet/navigation, kolom finansial, key, Nginx/HTTPS atau MemoryMax. Perubahan UI hanya status mapping/progress, retry satu item/pilihan sumber dan keputusan konflik.

283 tes backend lulus pada Python 3.10/3.12, delapan tes browser lulus. Peak benchmark sekitar 133 MiB. Steady warm Preview 49 GET dan retry item 28 GET; cold processing tetap mahal dan durasi Google asli belum terbukti. Akses Google/VPS asli tidak tersedia, sehingga penerimaan live dan sasaran Preview <60 detik tetap pending. Lihat TEST_REPORT.md, PERFORMANCE_V42.md dan UPGRADE_V42.md.
