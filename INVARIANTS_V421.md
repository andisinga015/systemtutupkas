# Bukti perlindungan V4.2.1

Tidak ada perubahan koordinat atau perhitungan finansial. `engine/parser.py`, `engine/resolver.py`, report/reconciliation, source reader/index/snapshots, models/migrations dan auth tetap byte-identik V4.2. Jalur single-group keputusan admin dan Preview tetap menggunakan guard lama. Tabel ini menginventarisasi guard global dan lokal yang dipakai writer.

| Invariant V4.2 | Bukti V4.2.1 dan perilaku bila tidak terbukti |
|---|---|
| Monthly registry READY; destination ID/URL/revision dan mapping paling baru sama dengan pin Run | `assert_current()` saat refresh, persiapan dan tepat sebelum HTTP; tidak ada remapping otomatis |
| Identitas tab Google tetap sama | Properties sheetId dibandingkan record registry, termasuk shared metadata closed epoch |
| Global structural signature: dua header awal, seluruh tanggal, bank BCA/MANDIRI/BRI/BNI, TOTAL dan semua merge | Predicate penuh sepanjang alokasi tetap diindeks/hash pada disk; allowed financial overlays memakai fungsi V4.2 yang sama. Versi unchanged hanya menggantikan proof yang sudah tersimpan |
| Extent dua header dan lebar/batas terakhir website tidak berubah karena formatted blank/Note | Full fields empat baris awal; thin body hanya bila kedua extent header fresh menutup allocated width dan bukti mapping lama sesuai. Selain itu restart **reader V4.2 lengkap**. Body sparse tidak boleh menentukan batas baru yang diasumsikan |
| Formula global di mana pun, termasuk di luar website terpilih | userEnteredValue formula tetap dipindai; digest koordinat+teks+count sama dengan V4.2. Signature sebelum/sesudah harus cocok; perubahan saat PREPARED tidak diadopsi |
| Positif/unik batas website, tanggal/account, fee header/merged-origin, slot bank dan laporan | Predicate website/report/fee/date tetap lengkap; `RegistryView.check_validations()` menjalankan resolver lama dan membandingkan koordinat yang disetujui. Empat baris awal tetap punya background untuk header nomor hari KET |
| Snapshot anchor lengkap termasuk Note/formula, juga fee header di bawah dua baris | Anchor di-GET dengan full fields dan dibandingkan; Note body tidak dihilangkan dari pembacaan anchor/target |
| Tidak ada struktur berubah di antara tab proof yang dibagikan | Versi file diperiksa sebelum dan sesudah preflight serta final target read. Post metadata/global/target proof ditutup dengan token sama sebelum acknowledgement. Tanpa token tidak berbagi mutable proof lintas tab |
| Target manual, formula atau merged cell dilindungi | `check_write()` lama pada actual full CellData; final actual snapshot GET lagi sebelum POST. Formula/merge tetap tidak dapat dilewati oleh NEW |
| Nilai, deskripsi, amount, dan Note tetap satu kelompok | Persiapan, POST, konflik dan readback per logical group lengkap; ledger grup baru VERIFIED setelah setiap sel cocok |
| Note manual tidak dihapus, Note QRIS/kiriman dan ownership sama | V4.2 `check_write()`, owner_key, transfer prepare/verified tetap digunakan; updateCells mask hanya userEnteredValue,note |
| Tidak ada dua intent menulis sel sama | Duplicate physical coordinate guard sebelum persiapan serta duplicate sheetId/row/col guard sebelum transport |
| Source authority, rencana lengkap/checksum, snapshot lengkap/checksum | Diverifikasi sebelum persiapan dan final POST boundary. Aggregate BIAYA memvalidasi input signed sukses dan snapshot dependency; tidak mengganti sumber atau menghitung ulang sukses diam-diam |
| Klaim retry atomik, lock tanggal, satu worker | Klaim conditional UPDATE/date lock/flock tidak berubah; QRIS transfer retry tetap QUEUED selama DEPO transfer Run yang sama masih PENDING, menjaga urutan alokasi lama |
| KET DEPO sebelum QRIS; slot milik lama/identitas/duplicate protection | Alokasi lama digunakan dengan reservation bersama pada observasi fresh; prefix planner bukan otorisasi tulis. Kiriman berubah tetap meminta keputusan seluruh kelompok |
| NEW / KEEP eksplisit, stale approval dan audit | Jalur konflik V4.2 dipertahankan; new test menguji kedua keputusan, Note manual dan mismatch BALANCE setelah KEEP |
| Accepted POST lalu timeout/crash tidak membuat duplikat | PREPARED durable sebelum HTTP; tidak ada POST ulang otomatis. Readback atau explicit retry mengenali nilai+Note+ownership yang sama; recover_structure menerima hanya overlay PREPARED yang persis cocok |
| Stop | Sebelum HTTP tidak kirim; setelah submit seluruh readback dicoba dahulu. Boundary tepat setelah phase WRITING juga memeriksa cancel. Upgrade menahan pekerjaan lama |
| Preview/read-only report/true KLOP | Preview tidak memakai financial batch; report tidak pernah menjadi output. Status parsial/KEEP mismatch tidak memaksa KLOP atau Clear |
| Observasi mutable tidak indefinite; cap resource lama tetap | Maksimum 64 kelompok, 8.000 sel, 8 MiB payload; 64 MiB observasi gabungan pre/final/post; full fields actual dibagi 32 sel; disk index lama tetap bounded. Tidak ada MemoryMax/response cap/pacing dinaikkan |

## Bukti otomatis

`backend/tests/test_v421_writer.py` menambahkan 60 kasus: 8 snapshot/43 item/57 sel, BIAYA shared scan, retry hanya total tertinggal, koordinat/Note persis, conflict NEW/KEEP, stale authority/mapping/snapshot/dependency, formula/Note/merge/manual, global date/formula/fee/epoch changes, response metadata fallback, token hilang, accepted POST lalu crash/timeout, independent group failures, cancellation boundaries, duplicate target, Preview, alokasi prioritas retry, diferensial full/thin reader, header sparse/padded/numeric/merge serta header hari baris 3/4. Test upgrade memverifikasi protected rows dan mendeteksi ledger corruption. Seluruh 283 kasus lama tetap ada.

## Batas yang sengaja dipertahankan

READY mapping adalah otorisasi koordinat, bukan bukti bahwa workbook tidak bisa diedit. Tanpa token file stabil atau setelah POST, global formula/date/merge/boundary proof masih perlu membaca seluruh predicate grid secara bounded. Formula/teks bernilai besar, header sparse, merge metadata besar, izin Drive tidak tersedia atau aktivitas editor eksternal dapat tetap memperlambat/menggagalkan proof. Tidak ada global guard yang dihapus demi waktu.

Google Sheets tidak menyediakan compare-and-set lintas GET dan POST. Perubahan pada celah terakhir GET→POST tidak dapat dijamin dicegah oleh API ini; perubahan yang tampak saat readback/global proof ditolak dan tidak diakui sukses. Gunakan kontrol editor pada penerimaan live. Batas ini sudah ada di V4.2, bukan klaim transaksi lintas Google/SQLite baru. Pengujian offline tidak membuktikan semua perilaku/quota Google produksi atau waktu <60 detik.
