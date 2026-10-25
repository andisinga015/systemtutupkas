# Baseline V4.2.2 — 25 Oktober 2026 (Asia/Hovd)

Source dasar adalah **ZIP V4.2.1 lengkap**, SHA256 `67e8dc5656c398ee8cae63b892529799dd19588e7fe9f036fe4dc122c14e9180`, 5.433.336 byte. Parent GitHub `5b3b6202e8ef5aeea886e256b5f4190f1ce531ec`. Oracle diekstrak ke direktori terpisah untuk pembandingan; arsip lama tidak ditimpa.

Sebelum perubahan core, seluruh **343 test V4.2.1 lulus pada Python 3.12.14, 306,62 detik**, satu warning Starlette/httpx deprecation. Schema V4.2 dan V4.2.1 sama: `c92448ce233b`. Constraint lama hanya mengizinkan satu DEPO per website/bulan; selector UI hanya mengenali DEPO/QRIS dan menggunakan pencarian berdasarkan type saja. `source_configuration()` mengikutsertakan seluruh dokumen sebulan tanpa pemilihan setengah bulan.

V4.2.2 menambah source period dan mengganti constraint menjadi website/type/year/month/period. DEPO lama menjadi **UNASSIGNED**, QRIS menjadi **MONTHLY**. Tidak ada inferensi berdasarkan judul spreadsheet atau tanggal Run lama. Penetapan Admin mempertahankan ID record, revision, lineage dan status tab yang sama.

Seluruh file engine finansial, destination reader/registry/Sheets writer V4.2.1, source reader/index/lineage, auth, quota dan worker dibandingkan byte-for-byte terhadap ZIP dasar dan tetap identik. Hanya pemilihan/otorisasi/provenance sumber, konfigurasi UI/API serta migrasi/deployment yang diubah. Lihat [CHANGED_FILES_V422.md](CHANGED_FILES_V422.md).

Tidak ada akses SSH atau Google produksi dalam pengujian. Observasi produksi Run #9 dari laporan terdahulu bukan pengukuran live V4.2.2. Benchmark memakai transport terisolasi dan JSON hasil lengkap disertakan.
