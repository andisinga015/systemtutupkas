# Pemilihan sumber dan isolasi V4.2.2

## Identitas dokumen

| Nama Admin | document_type | source_period | Tanggal |
|---|---|---|---|
| DOC A — DEPO/WD | DEPO | FIRST_HALF | 1–15 |
| DOC B — DEPO/WD | DEPO | SECOND_HALF | 16–akhir bulan |
| DOC QRIS | QRIS | MONTHLY | Seluruh bulan |
| DOC DEPO lama belum ditetapkan | DEPO | UNASSIGNED | Tidak dapat dipilih sebagai sumber Run |

Unique key: website_id + document_type + year + month + source_period. Check constraint database dan validator API menolak kombinasi lain. UNASSIGNED hanya untuk migrasi/record lama; API simpan normal tidak menerimanya. Source period wajib saat menyimpan URL baru.

## Urutan eksekusi

`source_configuration(website, selected_date)` memfilter **sebelum snapshot/item dibuat**: tepat satu periode DEPO dan QRIS bulanan bila dikonfigurasi. Tanggal 15 memakai A, tanggal 16 memakai B, termasuk Februari/leap year/akhir bulan. Tidak ada pencarian tanggal pada dua workbook untuk memilih sumber. Tidak ada fallback ke periode lain.

Dokumen DEPO yang relevan wajib valid dan memiliki tab aktif/tersedia dengan identitas terbukti. Dokumen periode lain tidak dapat memenuhi kekurangan itu. QRIS boleh tidak dikonfigurasi sesuai perilaku optional lama; QRIS yang sudah dikonfigurasi tetapi invalid tetap error, bukan nol transaksi.

Snapshot Run menyimpan doc ID, revision ID, spreadsheet ID, tab aktif, identitas/lineage, aliases dan source_period. SourceSnapshot menambah period pada provenance; financial payload/checksum/perhitungan tetap sama. Hanya sumber terpilih menghasilkan DEPO/WD/SALDO/expense/kiriman items dan snapshot/index reads. BIAYA hanya mengagregasi input DEPO terpilih.

Signature sumber menyertakan periode dan hanya dokumen yang relevan pada tanggal itu. Edit A tidak membuat Run tanggal 18 stale; edit B tidak membuat Run tanggal 10 stale. Signature destination terpisah. Source Date Index tetap memakai identity_id yang berasal dari doc ID, spreadsheet ID dan internal sheet ID; posisi row A tidak dapat dipakai di B.

## Sheet aktif/nonaktif

Selector UI mencocokkan **type + period**, lalu memanggil API dengan doc ID aktual. Tab hanya dimuat dari revision aktif dokumen itu. Website/bulan/tahun/dokumen/revision menjadi konteks request; perubahan konteks membersihkan pilihan dan bulk action. AbortController dan nomor urutan request melindungi respons terlambat, termasuk A→B→A ketika transport mengabaikan abort.

POST activation memeriksa setiap tab ID terhadap **doc ID, revision aktif, availability dan verified identity**. Request frontend membawa revision yang dilihatnya; server menolak revision lama. Sebelum mutation, SQLite write gate dan refresh record memastikan revision aktual. Validate/refresh tidak memegang write lock selama menunggu Google; pin doc/revision/spreadsheet/period diperiksa kembali setelah respons sebelum menerapkan metadata.

URL baru dan tab baru default nonaktif. Tidak ada penyalinan aktivasi antar A/B/bulan. Rename yang terbukti mempertahankan lineage/history menurut aturan V4.2.1, tetapi nama baru tetap memerlukan pengaktifan Admin; tab yang disabled tidak direaktivasi otomatis. Deleted/recreated dengan ID baru tetap error/revalidation dan tidak mewarisi ownership.

Per-document counts aktif/nonaktif/unavailable dihitung pada revision aktif. Dashboard tetap menghitung state aktual tanpa menggandakan revision historis.

## Retry dan histori

Retry hanya mengotorisasi item yang dipilih, memakai doc ID asal yang sama dan periode yang sesuai tanggal Run. Tidak ada pemindahan item A ke B berdasarkan nama. Source URL/revision/identity baru tetap memerlukan jalur reauthorization lama. Total BIAYA juga menolak dependency dari periode lama yang belum ditetapkan/salah periode. Successful items dan snapshot lamanya tidak dihitung ulang otomatis.

Histori lama tanpa period tetap bisa dibaca tanpa memodifikasi JSON lama. Untuk retry, DEPO lama harus ditetapkan Admin ke periode yang benar, pada record ID yang sama. Source context campuran yang belum konsisten tidak dianggap current hanya karena satu item telah diotorisasi ulang. Range resync yang melintasi 15/16 mengaitkan histori setiap tanggal hanya dengan exact source document records untuk tanggal tersebut dan destination pin yang sama.

Crawl All tetap satu website setiap saat. Masing-masing website memakai settingnya sendiri; missing DOC B di FANTASI menghasilkan configuration error FANTASI dan tidak menghalangi VELBETT yang valid.

## Yang tetap V4.2.1

Financial fixed columns, separator/closing/carry-forward, fee, warna kiriman, bank allocation, Notes, formula/merge/manual guards, NEW/KEEP, owner/duplicate/idempotent recovery, read-only report dan true KLOP tidak diubah. Semua file `backend/app/engine/*`, destination reader/registry dan Sheets transport tetap byte-identik V4.2.1. Mapping READY tidak direbuild karena save/validate/assign sumber. Batch validation, atomic write batching dan grouped readback tetap aktif. Tidak ada perubahan concurrency, quota pacing, MemoryMax, cap API atau kredensial.
