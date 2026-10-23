# Pemeriksaan scope V4

Perbandingan dilakukan terhadap source di ZIP V3 yang telah dikirim, dalam repository Git review temporer terpisah. Working tree/index/HEAD proyek pengguna tidak diganti. Full diff diperiksa mencakup backend, tests, deployment dan dokumentasi. Daftar berkas dan tujuan perubahan ada di [RELEASE_NOTES_V4.md](RELEASE_NOTES_V4.md).

| Batas | Hasil |
|---|---|
| Tidak ada full source-tab CellData/grid di RAM | Lulus; runner hanya memakai source_day, selected-day sparse rows |
| Discovery values-only dan chunk bounded | Lulus; default 200 baris, batch rentang kolom tunggal |
| Tidak mengunduh seluruh nilai histori sama sekali | **Belum terpenuhi secara literal:** discovery masih memindai nilai global demi semantik tanggal V3; pembatasan marker region dihentikan karena konflik, lihat PERFORMANCE_V4.md |
| Posisi tanggal cache stale | Tidak ada cache lintas run; anchor diverifikasi ulang |
| Kolom, hari kosong, saldo, expense dan fee | Parser V3 identik byte; hasil fixture/differential lulus |
| Brown/blue, bank, overflow, Notes, ownership | Parser/resolver/writer sama; tes lama dan golden output lulus |
| Tujuan, formula, entri manual dan verifikasi | Resolver/writer sama; reads dalam band tetapi snapshot/proteksi tetap lengkap dalam batas aman |
| Laporan read-only, KLOP dan Clear | Resolver laporan dan reconciliation sama; write ditolak sebelum API |
| UI/CSS/API contracts | Source frontend dan 59 aset identik; operasi API/models/workflow sama |
| Worker pool, paralelisasi website/modul | Tidak ditambahkan; tetap antrean/single worker |
| OOM restart/retry | Interrupted run ditahan FAILED/PARTIAL, perlu retry manual; PREPARED/VERIFIED disimpan |
| Cancel dan Stop | Permintaan tersimpan dihormati; POST yang sudah dikirim tetap dibaca ulang |
| Memori dan response bounds | Batas byte, chunk, sel konteks; tes subprocess RLIMIT_AS 1536 MiB lulus |
| Metrik hanya VPS | Structured logs, rotasi privat 5 MiB × 4; UI tidak diubah |
| DB/histori/akun/key/Nginx/HTTPS | Tidak dihapus/diganti; queue maintenance diaudit sesudah backup |
| MemoryMax/swap/OOM killer | Tidak diubah oleh upgrade |
| Live sumber/website PREVIEW dan deploy VPS | **Pending operator**; belum ada akses Google/key/SSH di lingkungan pengembangan |

V4 adalah paket koreksi backend untuk ditinjau dan diuji live sesuai panduan, bukan pernyataan bahwa semua target produksi telah terbukti. Tidak ada perubahan aturan finansial untuk mencapai angka benchmark. Tidak ada klaim jumlah request total turun, full website memakai 45 MiB, atau OOM produksi definitif teratasi.
