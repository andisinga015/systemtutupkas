# Benchmark V4.2 — 24 Oktober 2026

Fixture lazy terisolasi memakai engine, SQLite, snapshot/index/registry/writer asli; **tidak ada koneksi Google, kredensial, latensi jaringan atau pacing real**. Tiga tab sumber masing-masing mempunyai 40.000 baris histori sintetis. Tujuan delapan tab beralokasi 1.000 × 1.500, blok digeser ke kanan/bawah untuk membuktikan koordinat absolut. Kasus VELBETT 18 Oktober 2026 memiliki interval populated yang ekuivalen secara aman; angka/koordinat/Notes/bank dibandingkan dengan golden V4.1 (22 sel). Bukan pembuktian isi workbook asli tanggal tersebut.

| Tahap/attempt | Waktu lokal (ms) | GET termasuk metadata | POST finansial | Retry transport |
|---|---:|---:|---:|---:|
| Mapping cold | 131.311 | 218 | 0 | 0 |
| Processing awal, indeks cold | 2819.514 | 1381 | 16 | 0 |
| Preview pertama setelah write | 345.757 | 211 | 0 | 0 |
| Steady warm Preview | 267.145 | 49 | 0 | 0 |
| Retry satu deposit gagal | 150.766 | 28 | 0 | 0 |

Retry mengotorisasi satu item yang dibuat gagal untuk uji, mengambil snapshot raw baru, mempertahankan semua hasil sukses lain dan melakukan 0 POST karena hasil aktual sudah sama. Tidak ada remapping tujuan pada Preview/retry. Semua 22 nilai/koordinat/Notes/bank cocok; laporan selalu GET. Cold mapping terpisah dari durasi Run.

| Metrik tahap | Processing awal (ms) | Warm Preview (ms) | Retry item (ms) |
|---|---:|---:|---:|
| Discovery tanggal | 1922.840 | 1.863 | 0.874 |
| Payload sumber + validasi di luar discovery | 74.866 | 73.498 | 26.452 |
| Lookup registry | 40.561 | 35.793 | 29.202 |
| Kalkulasi + SQLite/coordinator (exclusive) | 262.860 | 137.430 | 76.761 |
| Validasi tujuan (inclusive) | 459.763 | 18.404 | 10.399 |
| Actual POST fixture | 0.856 | 0.000 | 0.000 |
| Baca ulang + global proof | 270.364 | 0.000 | 0.000 |
| Rekonsiliasi | 25.545 | 0.000 | 14.758 |

Tahap inclusive dapat beririsan (misalnya readback mencakup validasi global); jangan menjumlahkannya sebagai total. Exclusive coordinator mencakup kontrol/SQLite selain aritmetika. Angka 0 pada tahap tulis/verifikasi berarti tidak ada penulisan, bukan verifikasi yang dipalsukan. Durasi total diukur monotonic; API counters menghitung setiap request attempt dan token Drive. Metrik fixture menambahkan hitungan di wire adapter yang sama, sementara runtime nyata mengukur SheetsClient.request.

| Fallback tanpa token versi | Waktu lokal (ms) | GET | POST |
|---|---:|---:|---:|
| Processing cold | 2818.061 | 1564 | 16 |
| Preview, cold proof wajib | 2196.378 | 596 | 0 |
| Retry satu item | 733.507 | 285 | 0 |

Peak RSS executable benchmark bertoken **133.16 MiB**, fallback **133.18 MiB**; RLIMIT_AS 1536 MiB, tidak OOM. Respons terbesar processing 44,266 byte, batas runtime tetap 8.388.608 byte. Snapshot terpilih disimpan, bukan 40.000 baris histori lengkap.

## Arti angka dan batas penerimaan

- Baseline V4.1 offline lama mencatat 1.319 request pada website dengan fixture berbeda; ini bukan perbandingan apples-to-apples karena V4.2 menambah proof versi/identitas dan batas crash. Cold Run V4.2 masih mahal dan **tidak diklaim lebih cepat** dari baseline tersebut.
- Preview warm turun dari 596 GET pada proof tanpa token menjadi 49 GET pada steady token yang sama. Retry item turun dari 285 menjadi 28 GET. Perubahan isi setelah POST membuat versi record per tab tertinggal; Preview pertama 211 GET melakukan validasi global ulang sebelum steady warm.
- Pacing global tetap 1,2 detik. Batas bawah jarak awal request adalah `(N−1) × 1,2`: 49 GET sudah sekitar 57,6 detik, 28 GET sekitar 32,4 detik, belum latensi/backoff. **Preview live <60 detik belum terbukti.** Angka lokal subdetik bukan janji durasi Google.
- Cold source discovery masih menelusuri nilai pada pita kolom bounded untuk membuktikan uniqueness tanggal; tanpa API pencarian tanggal server-side dan tanpa bukti versi, histori tidak boleh dilewati dengan asumsi. Tidak ada full-grid source materialization/formatted history.
- Drive API atau izin metadata yang tidak tersedia menonaktifkan fast path. Token konfigurasi/dua anchor saja tidak dipakai sebagai bukti bahwa tidak ada duplicate separator baru.
- Golden penuh dengan fingerprint transaksi asli tetap diuji pada fixture asal; benchmark menambahkan offset histori sumber sehingga token transaksi berbasis posisi berbeda secara sengaja, tetapi nilai/Notes/bank dan alamat tujuan harus identik.

## Reproduksi

```bash
.venv/bin/python backend/tests/performance/measure_v42_website.py --history 40000
.venv/bin/python backend/tests/performance/measure_v42_website.py --history 40000 --without-revision
```

JSON lengkap disertakan dalam PERFORMANCE_WEBSITE_V42.json dan PERFORMANCE_FALLBACK_V42.json. Uji lama untuk sumber besar/tujuan lebar/preformatted blanks tetap dijalankan dalam full-suite.

## Dokumentasi API yang dirujuk

- https://developers.google.com/workspace/sheets/api/reference/rest/v4/spreadsheets.values/batchGet
- https://developers.google.com/workspace/sheets/api/reference/rest/v4/spreadsheets/batchUpdate
- https://developers.google.com/workspace/sheets/api/limits
- https://developers.google.com/workspace/drive/api/reference/rest/v3/files/get

Retrieval HTTPS halaman dokumentasi pada lingkungan ini gagal karena proxy (ProxyError); tidak diklaim telah dibuka atau diuji live. Endpoint GET/POST dan scope mengikuti kontrak API; penerimaan asli memakai key/izin/kuota VPS masih pending.
