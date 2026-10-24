# Arsitektur dan batas keselamatan V4.2

Satu worker/coordinator tetap memegang flock. Scanner, Mapper dan Writer adalah tahap logis dalam proses tersebut; tidak ada lima worker website atau tiga proses Python finansial yang berjalan bebas. Background mapping URL adalah pembacaan terpisah melalui API, memakai lock builder serta pacing Google yang sama.

## Sumber dan snapshot

`lineage.py` merekam spreadsheetId, Google sheetId, revisi konfigurasi, nama yang pernah terbukti, dan binding ke aktivasi. Rename dengan ID sama terhubung ke lineage lama; nama baru tetap perlu diaktifkan. Nama sama dengan ID baru dinonaktifkan dan ditandai berubah, bukan dianggap rename. Legacy tanpa bukti memerlukan konfirmasi administratif yang tercatat.

`source_index.py` menyimpan posisi separator, separator berikutnya, shape dan bukti yang terikat versi isi Drive. Indeks hanya menavigasi, bukan menyimpan truth finansial. Forward/backward memakai indeks yang sudah terbukti; payload dan separator aktual tetap GET. Insert/append/edit atau pergantian versi membatalkan asumsi lama. Tanpa token tepercaya, sistem melakukan discovery bounded lagi. Cold proof masih membaca nilai pada pita kolom sempit sepanjang histori untuk membuktikan separator unik, termasuk tanggal yang mungkin berada di kolom berbeda. Ini **bukan** query tanggal di sisi Google dan bukan pembacaan nol histori. Tidak ada materialisasi seluruh grid/formating historis.

`snapshots.py` merekam interval terpilih, penutupan/carry-forward yang diperlukan, posisi absolut sumber, warna transaksi yang relevan, identity, waktu, provenance dan checksum. State SCANNING dikomit sebelum HTTP; SNAPSHOT_READY hanya setelah semua bukti lengkap. Mapper memakai Day/parser dan kolom tetap V4.1; tiap modul pada tab yang sama berbagi snapshot normal. Hasil mapping berisi seluruh kelompok write, mapping ID, snapshot ID, provenance dan checksum. Writer memverifikasi checksum/completeness sebelum menghubungi Google.

Batas respons tetap 8 MiB. Snapshot satu tab/tanggal ≤8 MiB dan akumulasi snapshot satu Run ≤64 MiB. Melebihi batas menghasilkan error eksplisit, bukan pemotongan data. Snapshot lengkap dipertahankan untuk audit/recovery; versi ini tidak otomatis menghapus snapshot, intent PREPARED, ledger atau keputusan untuk menghemat ruang. Retry sangat banyak pada satu Run dapat mencapai batas tersebut dan perlu tinjauan administratif/Run baru. Cache RAM target dibatasi 128 sel/4 MiB; indeks tujuan temporer tetap privat dan dibersihkan saat close.

## Registry tujuan bulanan

Save URL/validate membuat versi mapping dan mengantrekan builder. Builder memakai resolver bounded V4.1 pada seluruh lebar horizontal, semua website/tanggal yang relevan, delapan tab wajib, serta kategori rekening/fee/BIAYA/KET/report. Hasil disimpan sebagai koordinat absolut dan scope validasi; tidak ada key mapping berdasarkan nama tab sumber. Source alias digunakan untuk memilih kategori rekening yang sudah terdaftar, bukan menggeser target.

Normal Crawl/retry membaca registry yang dipin Run. Tidak ada mapping ulang otomatis, insert baris, append atau perubahan format. Registry READY dapat memuat masalah per modul/tanggal; entri invalid tetap ditolak dan pesan masalah tersedia pada pengaturan. Revalidasi eksplisit membuat versi baru dan mempertahankan versi/ownership/ledger lama. URL/revisi/mapping yang berubah tidak mengarahkan Run lama secara diam-diam.

Fast validation memerlukan token versi workbook yang tidak berubah, lalu membaca target dan anchor aktual. Token tidak tersedia/berubah memakai pembuktian global bounded serta critical resolver predicate untuk membandingkan **koordinat yang sama**, tidak mengadopsi koordinat baru. Ini validasi, bukan remapping. Semua struktur global V4.1 (dua header awal, tanggal/bank/TOTAL, merges) dan formula global tetap diperiksa; setelah POST pembuktian global selalu dilakukan. Perubahan sumber tidak meniadakan pemeriksaan target/manual Notes/ownership.

Deskripsi fee yang mengandung tanggal dapat termasuk predicate struktur global V4.1. Setelah seluruh write dan readback cocok, hanya perubahan finansial yang diotorisasi diperbarui pada bukti struktur. Jika worker mati sebelum pengakuan, perubahan itu hanya boleh dijelaskan oleh snapshot sel PREPARED yang cocok persis, tidak formula/merge, dan hash seluruh struktur lain tetap sama. Formula global yang berubah saat ada PREPARED ditolak. Pemulihan struktur tidak otomatis mengakui hasil finansial.

## Retry dan dependensi

`item_retry_requests` adalah antrean persisten dengan satu permintaan terbuka per item, key idempotensi unik, claim conditional dalam transaksi pendek, dan status QUEUED/CLAIMED/SUCCEEDED/FAILED/CANCELED/INTERRUPTED. API tidak mengubah RUNNING menjadi QUEUED ketika item gagal diminta ulang. Coordinator mengklaim sampai tiga retry prioritas per batas aman, lalu memberi giliran pekerjaan normal; pemeriksaan akhir Run menangkap permintaan menjelang selesai. Worker flock mencegah dua proses coordinator.

Retry SOURCE_RETRY membuat snapshot baru dari raw source saat ini untuk satu item. Otorisasi sumber disimpan per item; `Run.sources` historis tidak ditulis ulang. Perubahan URL/alias/rename terbukti dapat dipakai sesuai konfigurasi aktif; nama tanpa bukti memerlukan pemilihan eksplisit. Tujuan tetap dipin. Sumber yang berubah lagi setelah otorisasi ditolak sebelum tulisan. Klik ganda tidak mengganti otorisasi permintaan yang sudah menunggu/berjalan.

Item sukses tidak dibaca/ditulis ulang. Bila sumber baru berbeda dari snapshot sukses lain, audit/notifikasi menjelaskan hasil yang tetap memakai snapshot semula. Jika input BIAYA diperbarui setelah total berhasil, total diberi DEPENDENCY_STALE dan tidak dihitung ulang otomatis; tombol retry total memerlukan otorisasi terpisah. DOC-wide retry lama boleh memasukkan total yang **sudah gagal**, sementara retry item spesifik tidak memasukkan item lain. Clear tetap diblokir ketika ada dependensi stale atau konteks sumber campuran.

Pembatalan menghentikan pembacaan berikutnya; POST yang sudah dikirim tetap diperiksa. Restart worker menahan Run terputus dan mempertahankan intent; permintaan CLAIMED menjadi INTERRUPTED, QUEUED tidak dihapus. Tidak ada financial replay otomatis akibat restart. API restart tidak kehilangan permintaan SQLite. Retry conflict memakai keputusan serta snapshot yang telah diotorisasi; SOURCE_RETRY berbeda karena harus mengambil raw baru.

## Konflik, fee dan kiriman

Kelompok old/new dibaca langsung dari Google, dilengkapi nilai, Note, formula, ledger dan hash. `Masukkan Data Baru` menyimpan keputusan immutable, memeriksa lagi sumber/tujuan/mapping/snapshot, observasi aktual dan keseluruhan kelompok, lalu menulis melalui Writer tunggal. `Tetapkan Data Lama` membaca lagi dan mempertahankan nilai/Notes aktual tanpa POST; keputusan serta pointer retained-cell tersimpan. Sukses dan error hanya diakui setelah seluruh kelompok sesuai keputusan. Persetujuan stale kembali menjadi konflik terbuka. Error terdahulu dan keputusan/audit tetap tersimpan.

Preview tidak dapat menulis lewat keputusan konflik. Formula dan merge tidak bisa ditimpa hanya karena admin memilih NEW. Note manual tetap dilindungi; value-only update mempertahankannya. Fee clear hanya boleh mengenai fee robot dengan ownership yang terbukti. No fee pada sumber yang benar-benar berhasil dibaca dan target kosong adalah no-op, termasuk bila header tulis tidak diperlukan. No fee dengan data lama menjadi konflik; formula TOTAL berada di luar payload fee. Gagal membaca sumber tidak pernah dianggap no fee.

Resolver fee hanya memperluas pencarian header pada dua modul fee: antara anchor website dan baris tanggal pertama (maksimal 12 baris), dengan deduplikasi origin merged header. Resolver rekening/BIAYA/KET tidak dilemahkan. Beberapa fee tetap digabung menggunakan aturan parser/deskripsi V4.1.

Kiriman DEPO didahulukan, lalu QRIS dengan urutan BCA/MANDIRI/BRI/BNI. Warna cokelat yang disetujui/pengecualian biru, nominal bertanda dan format Note tetap. Ketersediaan slot diperiksa lagi di Stage C dari sel aktual; retry prioritas tidak memakai reservasi lama sebagai izin menimpa. Transaksi berubah/hilang menghasilkan seluruh kelompok koreksi pada slot lama, bukan append baru. TransferLineage menjaga token asli TransferOwnership serta current/pending intent; retire/reactivate tetap tercatat. Slot retired yang dipakai ownership lain tidak boleh direbut kembali otomatis.

## Rekonsiliasi, metrik dan audit

Laporan deposit harian hanya GET, melalui koordinat registry dan validasi yang sama. BALANCE individual/konsolidasi memakai angka Google dan aturan exact-zero lama. KEEP, preview, hasil lama atau retry sumber campuran tidak membuat KLOP buatan.

`telemetry.py` mengukur attempt dengan ContextVar, waktu inclusive per tahap dan waktu exclusive tanpa menghitung nested coordinator dua kali, actual request attempt/retry, byte respons, RSS/peak dan total durasi. Exclusive coordinator mencakup kalkulasi, SQLite dan scheduling; bukan murni aritmetika. Readback mencakup global proof serta pembacaan semua sel; waktunya beririsan dengan destination_validation sehingga jangan menjumlahkan seluruh kolom inclusive. Attempt metadata dibatasi 20 terakhir dalam DTO Run; seluruh audit metrik tetap tersimpan. Metrik tidak berisi key atau payload keuangan.

Pacing 1,2 detik bersifat global antar API/worker melalui file lock singkat, boot ID dan monotonic timestamp. Lock dilepas saat menunggu/network; GET/backoff dapat dibatalkan. POST dengan hasil tak pasti tidak dicoba ulang oleh transport: Writer terlebih dahulu memeriksa seluruh hasil. Batas 1536M/8 MiB tidak dinaikkan. Benchmark fixture, dokumentasi Google, keterbatasan token, dan penerimaan live dijelaskan di PERFORMANCE_V42.md.
