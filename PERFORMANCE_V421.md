# Benchmark destination writer V4.2 → V4.2.1

Diukur 25 Oktober 2026 (Asia/Hovd), Python 3.12.14. Oracle V4.2 diekstrak dari ZIP/SHA pada [BASELINE_V421.md](BASELINE_V421.md). Angka berikut adalah satu pengukuran fixture offline dengan transport yang sama, tanpa Google latency, OAuth, quota wait atau source network. Bukan pengukuran produksi.

Fixture: VELBETT 18 Oktober 2026; 8 snapshot sumber, 43 financial items, 57 physical updates (13 kiriman) dan 8 tab tujuan. BIAYA 256 × 64 mempunyai formatted blanks dan Note historis panjang di luar target. Mapping READY dibuat lewat API asli sebelum timer dimulai; angka bukan benchmark cold registration atau 7.936 target produksi.

| Tahap | V4.2 GET | V4.2.1 GET | V4.2 respons byte | V4.2.1 respons byte |
|---|---:|---:|---:|---:|
| Planning | 34 | 13 | 3.761 | 5.945 |
| Pre-write validation | 220 | 24 | 50.450.383 | 11.631 |
| Writing | 0 | 0 | 639 | 241 |
| Readback verification | 390 | 24 | 50.508.933 | 92.786 |
| Reconciliation read-only | 16 | 16 | 7.387 | 7.383 |
| **Total** | **660** | **77** | **100.971.103** | **117.986** |

| Ukuran | V4.2 | V4.2.1 | V4.2.1 tanpa token |
|---|---:|---:|---:|
| Financial POST | 36 | 1 | 7 |
| Destination GET | 660 | 77 | 114 |
| Total response byte | 100.971.103 | 117.986 | 221.906 |
| GET ≥2 MiB | 12 | 0 | 0 |
| Respons terbesar yang tercatat | 8.388.608 | 51.843 | 51.843 |
| Wall clock, ms | 3.024,637 | 1.184,567 | 1.275,149 |
| Peak RSS, MiB | 155,29 | 151,58 | 152,09 |
| Address-space limit, MiB | 1.536 | 1.536 | 1.536 |
| Item SUCCESS | 43 | 43 | 43 |

GET termasuk permintaan metadata Drive yang disimulasikan dan permintaan respons besar yang ditolak cap. Byte mencakup respons POST kecil; request payload tidak dijumlahkan sebagai response. Byte parsial percobaan yang ditolak dihitung sampai cap oleh fixture; transport produksi sekarang mencatat byte aktual yang diterima per chunk. Metrik jaringan sumber tidak diukur oleh fixture ini. Tidak ada proses mati/OOM pada pengukuran; ini bukan bukti bahwa semua workbook tidak mungkin mencapai cap.

**Koordinat, nilai, deskripsi, Notes, bank, fingerprint transaksi dan status setiap item identik terhadap oracle asli.** Output tokened sama urutan dan SHA256: `32c0ec33ef1164c5d971b66a16d6cd74ab842e6155053b5a34e48a59fc93f095`. Tanpa token urutan antar tab berubah karena single-tab windows; himpunan output lengkap dan status per item tetap sama, DEPO→QRIS di KET tetap urut. JSON menyimpan output lengkap dan jumlah proof per tab/tahap, sehingga hasil bisa diperiksa tanpa hanya percaya hash.

## Kasus expense_total yang tertinggal

Test `test_single_leftover_expense_total_uses_narrow_proof_without_source_or_success_replay` membuat Run V4.2-style berhenti setelah fee sebelumnya menulis dan membuat content revision BIAYA stale. Kemudian hanya total diotorisasi retry menggunakan batching. Hasil: **satu POST untuk satu sel BIAYA B3=33.000**, tidak membaca sumber, tidak mengubah hasil sukses lain, 8 snapshot/checksum/payload tetap sama dan mapping READY yang sama. Global BIAYA stale tetap dibuktikan, memakai field projection yang aman; bukan full Note/body rescan. Pada Run 43 item normal, BIAYA hanya satu shared post-write proof dan warm prewrite tidak scan global. Kedua kondisi diuji terpisah.

## Reproduksi

Ekstrak V4.2 asli ke lokasi oracle terpisah lalu dari root V4.2.1:

```bash
.venv/bin/python backend/tests/performance/measure_v421_writer.py --engine-root /path/oracle/tutupkas-lion-group --legacy
.venv/bin/python backend/tests/performance/measure_v421_writer.py
.venv/bin/python backend/tests/performance/measure_v421_writer.py --no-token
.venv/bin/python -m pytest backend/tests/test_v421_writer.py -q
```

Hasil pengukuran rilis tersimpan di `PERFORMANCE_BEFORE_V421.json`, `PERFORMANCE_AFTER_V421.json`, dan `PERFORMANCE_FALLBACK_V421.json`. Test regresi juga membandingkan transport legacy/current serta threshold request/byte, tanpa assertion wall-time rapuh.

## Batas penerimaan

**Target seluruh website <60 detik belum dicapai/dibuktikan secara live.** Default pacing global 1,2 detik tidak diubah: 77 GET + 1 POST saja mempunyai jarak start sekitar 92,4 detik, sebelum latensi, sumber, token OAuth atau retry quota. Waktu ~1,18 detik di atas hanya komputasi fixture tanpa pacing, tidak boleh dipakai sebagai janji waktu Google. Menurunkan pacing atau melemahkan global formula/date/merge proof bukan bagian patch ini.

Projection hanya aktif dengan bukti geometry yang cukup. Full fallback pada header sparse, banyak formula/value text, metadata merge sangat besar, workbook jauh lebih lebar atau concurrent edits masih dapat mahal. Registration/mapping tetap menggunakan reader asli. Pengukuran akses Google asli/Run #9/VPS, efek rate-limit, ukuran dataset produksi dan peak RSS proses produksi masih **pending**. Jalankan penerimaan di salinan yang diotorisasi dan gunakan klasifikasi log baru untuk menghitung GET/byte per phase sebelum meneruskan Run produksi.
