# Performa V4.2.2 dibanding V4.2.1

Diukur 25 Oktober 2026 (Asia/Hovd), Python 3.12.14, fixture terisolasi tanpa Google latency/quota waiting/pacing. Baseline menjalankan **source V4.2.1 asli dari ZIP**. Sesudah patch memakai 8 snapshot, 43 item, 57 sel, 13 kiriman dan **10 tab DOC A aktif tambahan yang tidak boleh diproses** pada 18 Oktober.

| Ukuran | V4.2.1 asli | V4.2.2 | V4.2.2 tanpa token |
|---|---:|---:|---:|
| Selected source snapshots | 8 | 8 | 8 |
| Financial items | 43 | 43 | 43 |
| Tab DOC A aktif tambahan | 0 | 10 | 10 |
| Read sumber periode tidak terpilih | 0 | **0** | **0** |
| Destination GET | 77 | **77** | 114 |
| Financial POST | 1 | **1** | 7 |
| Response bytes tujuan | 117.986 | **117.986** | 221.906 |
| GET ≥2 MiB | 0 | **0** | 0 |
| Wall clock offline, ms | 1.329,188 | 1.293,431 | 1.527,273 |
| Peak RSS, MiB | 152,89 | 157,32 | 157,38 |
| Address-space limit, MiB | 1.536 | 1.536 | 1.536 |

Semua 43 status dan seluruh 57 koordinat/nilai/deskripsi/Notes/bank/transaction fingerprint **persis sama** dengan baseline. Tokened output SHA256 `32c0ec33ef1164c5d971b66a16d6cd74ab842e6155053b5a34e48a59fc93f095`. Tanpa token urutan antar tab berbeda karena single-tab windows lama, tetapi himpunan output lengkap dan status per item sama.

Peak RSS sesudah mencakup sepuluh cloned source grids dan state registrasi fixture tambahan yang disiapkan sebelum timer. Kenaikan ~4,4 MiB pada proses fixture itu tidak disamarkan sebagai nol; tidak ada OOM/cap increase. Wire counts/bytes identik dan source A tidak dibaca dalam Run. Pengukuran tunggal wall-time bukan statistik latensi Google.

| Tahap | GET sebelum/sesudah | Response bytes sebelum/sesudah |
|---|---:|---:|
| Planning | 13 / 13 | 5.945 / 5.945 |
| Pre-write validation | 24 / 24 | 11.631 / 11.631 |
| Writing | 0 / 0 (1 POST) | 241 / 241 |
| Readback verification | 24 / 24 | 92.786 / 92.786 |
| Reconciliation | 16 / 16 | 7.383 / 7.383 |

JSON lengkap: `PERFORMANCE_BEFORE_V422.json`, `PERFORMANCE_AFTER_V422.json`, `PERFORMANCE_FALLBACK_V422.json`. Reproduksi dari root V4.2.2:

```bash
.venv/bin/python backend/tests/performance/measure_v422_periods.py --engine-root /path/oracle/tutupkas-lion-group
.venv/bin/python backend/tests/performance/measure_v422_periods.py
.venv/bin/python backend/tests/performance/measure_v422_periods.py --no-token
```

Cold source registration/mapping tidak termasuk timer. Source network memakai fixture, bukan Google nyata. Fixture bukan seluruh 7.936 target produksi Run #9. Pacing global tetap 1,2 detik: 78 destination requests saja mempunyai jarak start sekitar 92,4 detik sebelum latensi/sumber. **Tidak ada klaim live <60 detik.** Batas/fallback proof global V4.2.1 tetap berlaku; lihat `PERFORMANCE_V421.md` dan `INVARIANTS_V421.md`. Tidak ada Chromium scraping atau lima worker baru.
