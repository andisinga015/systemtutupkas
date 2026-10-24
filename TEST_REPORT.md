# Laporan pengujian V4.2 — 24 Oktober 2026

**283 passed pada Python 3.12.14 (174,80 detik) dan 283 passed pada Python 3.10.22 (215,55 detik).** Satu warning Starlette/httpx deprecation pada kedua versi; dependensi terkunci tidak diubah. Seluruh **198 kasus V4.1 dipertahankan**, dengan 85 kasus tambahan. Tidak ada test gagal yang dihapus.

## Hasil verifikasi

| Pemeriksaan | Hasil |
|---|---|
| Backend full suite Python 3.12 | 283 passed |
| Backend full suite Python 3.10 | 283 passed |
| TypeScript strict + Vite build | Lulus, frontend prebuilt disertakan |
| Chromium Playwright | 8 passed, 26,7 detik |
| Alembic current/check | c92448ce233b (head); no new upgrade operations |
| Shell syntax upgrade/rollback/install/e2e | Lulus |
| Migrasi populated V1/V2/V4.1 → V4.2 | Data lama/integritas/FK dipertahankan |
| Downgrade kosong / rollback backup | Lulus; downgrade tabel V4.2 berisi data ditolak |
| Large-source/wide/preformatted memory tests lama | Lulus dalam full suite |
| Fixture website VELBETT 18 Oktober | 22 nilai/koordinat/Notes/bank cocok golden |
| Live Google/VPS pengguna | **Pending; tidak ada akses/penulisan asli** |

Browser mempertahankan lima kasus lama: admin/login, persistensi setting/periode, Help Center, sidebar desktop/zoom/mobile dan enqueue/cancel nyata. Tiga kasus baru menguji retry item ketika Run aktif serta kedua mode keputusan konflik/Preview. Google di kasus baru browser memakai mock kontrak UI secara eksplisit; server source/mapping/decision/readback diuji oleh backend dengan wire fixture. Uji API restart benar-benar menjalankan dua generasi proses uvicorn, mempertahankan sesi login dan retry QUEUED yang sama di SQLite. Uji worker subprocess mempertahankan flock dan menolak worker kedua.

## Cakupan tambahan

- Satu snapshot lengkap bersama, fixed columns, checksum/completeness, indeks forward/backward, invalidasi edit/insert/append, missing/duplicate, fallback tanpa token, tab inactive/rename/recreated, alias aktif setelah rename dan kepemilikan lama tanpa duplikasi.
- Registry bulanan seluruh lebar, target absolut terpin, struktur/identity/merge/header/date yang berubah ditolak, source rename tidak mengubah target, revalidation mempertahankan histori/ledger.
- Retry satu item di antara 42 item tambahan yang sudah sukses; raw sumber baru, current URL/alias/explicit-tab override hanya untuk item tersebut, prioritas saat membaca/menulis/menjelang selesai, klik ganda/claim/restart/cancel, stale BIAYA total tanpa auto-replay.
- Actual old/new, source provenance, kelompok fee/transfer, NEW/KEEP, stale approval, CSRF/idempotency, formula/Notes, Preview zero financial writes, KEEP tidak mengakui ledger PREPARED asing/Preview, seluruh kelompok tetap unresolved bila readback gagal.
- Header fee di bawah dua baris awal dan merged origin, batas website, beberapa fee, no-op source tanpa fee, konflik no-fee vs data lama, clear hanya robot-owned, transfer berubah/hilang/restored pada slot lama, legacy proof administratif.
- Worker mati setelah accepted POST pada angka/fee/transfer: intent tersimpan dan sel yang sama tidak dikirim ulang; global formula berubah saat PREPARED tetap ditolak. Tidak ada finalisasi palsu KLOP.

Pemetaan setiap nomor pada 80 persyaratan §21 terdapat dalam [COVERAGE_V42.md](COVERAGE_V42.md). Fixture golden asli, parser finansial, source reader dasar, stylesheet/navigation dan tiga migrasi sebelumnya dibandingkan byte-for-byte terhadap ZIP V4.1.

## Penyesuaian test lama yang memang mengikuti perilaku V4.2

Kasus tidak dihapus atau dilemahkan:

- Lost reply POST yang hasilnya langsung terbukti kini SUCCESS tanpa POST ulang; untuk kasus uncertain PREPARED, fixture juga membuat pembacaan verifikasi tidak tersedia. Keduanya diuji terpisah.
- POST HTTP 503 kini berhenti setelah satu percobaan dan meminta readback; assertion lama yang mengharuskan POST kedua diganti dengan bukti tidak replay.
- Run identik berikutnya tetap mempunyai 22 rencana golden tetapi mengirim 0 POST bila semua sel sudah cocok dengan ledger terverifikasi.
- Resync ke workbook sumber berbeda tidak dianggap bukti identitas lama; test mengharuskan konflik kiriman dan keputusan NEW eksplisit, lalu membuktikan jumlah ownership/slot tidak bertambah.
- Fixture yang sengaja mengubah tanggal/layout tujuan sesudah konfigurasi kini mengotorisasi registry baru terlebih dahulu; normal Crawl/Retry tidak diberi remapping otomatis. Fake sheetIds dibuat konsisten dengan metadata aslinya.
- Lock test diisolasi per direktori sementara agar dua versi Python/subprocess tidak berbagi lock produksi.

## Performa dan batas produksi

Benchmark memakai sumber lazy masing-masing 40.000 baris, delapan tab tujuan alokasi 1.000 × 1.500, RLIMIT_AS 1536 MiB. Peak RSS sekitar 133 MiB; respons processing terbesar 44.266 byte. Cold mapping 218 GET; processing awal 1.381 GET + 16 POST; steady warm Preview 49 GET + 0 POST; retry satu item 28 GET + 0 POST. Semua angka offline tanpa pacing/latensi Google. Detail per tahap, fallback tanpa token serta JSON lengkap ada di [PERFORMANCE_V42.md](PERFORMANCE_V42.md).

Cold proof/write masih mahal. Target Preview live <60 detik belum terbukti; 49 request dengan pacing 1,2 detik saja sudah sekitar 57,6 detik. Tanpa token versi, Preview dan retry lebih lambat. Cold discovery tetap melakukan values-only scan bounded sepanjang histori untuk membuktikan tanggal unik; tidak ada full historical grid materialization. Tidak ada klaim dapat melewati histori secara aman tanpa bukti.

Sheets tidak menyediakan transaksi compare-and-set lintas pembacaan dan batchUpdate. Sistem memeriksa kembali anchor/nilai/versi/approval sebelum POST, tetapi tidak menjamin kebal terhadap editor eksternal yang mengubah sel pada celah GET→POST. Tinjau penerimaan dengan akses edit terkontrol. Histori Google setelah backup tidak di-undo oleh rollback SQLite; cocokkan sel aktual sebelum melanjutkan intent.

Snapshot maksimum 8 MiB per tab/tanggal dan akumulasi 64 MiB per Run; mencapai batas adalah error eksplisit, bukan izin menghapus audit/intent atau memotong data. Tidak ada cleanup otomatis financial records. Identitas lama tanpa bukti memerlukan konfirmasi administratif; konflik formula/merge/manual Note dapat tetap blocked walaupun admin meminta NEW. Tidak ada penggantian key, kenaikan MemoryMax/HTTP cap, atau klaim deployment VPS sudah dilakukan.

## Paket

Distribusi memuat source, frontend prebuilt, test, dokumentasi, migrasi additive dan skrip upgrade/rollback. ZIP diperiksa CRC, kelengkapan aset dan pengecualian private key/database/environment/cache. SHA256 disertakan terpisah. Versi final di GitHub diverifikasi lewat fetch baru dan checksum blob yang sama; arsip rilis lama tetap dipertahankan.
