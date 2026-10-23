# Benchmark tujuan V4.1 — 23 Oktober 2026

Baseline adalah ZIP V4 SHA256 `830a1db4beb31d3acca0d387843026fc77c47bcbe1b5a8a0b42b84f3225ed02e`. Reader V4 menolak ketiga fixture tujuan di bawah dengan `SHEET_TOO_LARGE`; V4.1 menghasilkan tulisan **VERIFIED**. Tidak ada perbandingan durasi terhadap run V4 yang berhasil pada ukuran tersebut.

## Tujuan lebar: planning, pre-write dan read-back

Probe memakai autentikasi/SQLite aplikasi, writer sebenarnya, transport API palsu dengan JSON dan cap streaming sebenarnya, serta fixture lazy yang tidak mengalokasikan grid penuh. Satu operasi menulis satu sel SALDO; tiga snapshot global melindungi planning, pre-write dan post-write. Tidak ada panggilan Google atau latensi jaringan.

| Alokasi tujuan | Isi fixture | GET / POST | Respons terbesar | Mapping | Pre-write | Read-back | Total | Peak RSS proses |
|---|---|---:|---:|---:|---:|---:|---:|---:|
| 1.000 × 1.000 | Sparse | 59 / 1 | 3.994 B | 18,84 ms | 18,79 ms | 16,34 ms | 54,03 ms | 127,82 MiB |
| 1.000 × 1.500 | Sparse | 83 / 1 | 3.986 B | 22,86 ms | 25,11 ms | 22,97 ms | 70,98 ms | 128,47 MiB |
| 1.000 × 1.500 | Semua blank terformat | 83 / 1 | 4.431.506 B | 7.994,91 ms | 7.725,60 ms | 7.824,64 ms | 23.545,18 ms | 172,79 MiB |

Source-read probe ini **0 ms**, sebab input writer sudah disiapkan; ini bukan klaim kecepatan source. Target 1.500 kolom adalah row 74/column 1421 (zero-based), **BBR75**. Hanya **164 sel struktur** diindeks. File indeks temporer 45.056 B untuk sparse dan 61.440 B untuk blank terformat. Baseline RSS sebelum operasi masing-masing 127,51 / 128,16 / 127,92 MiB. Peak mencakup harness, import autentikasi/SQLAlchemy, JSON, HTTP dan reader; jangan menyebutnya penggunaan reader saja atau mengurangkan baseline sebagai ukuran heap akurat.

Blank terformat sengaja mengembalikan CellData untuk seluruh alokasi. Total respons kumulatif **311.511.280 B** dalam tiga snapshot; setiap respons tetap di bawah 8 MiB dan dilepas setelah diindeks. Seluruh transfer tersebut tidak ditahan sekaligus di RAM. Ini menunjukkan batas memori sekaligus biaya global scan bila Google mengirim format pada banyak sel kosong. Data mentah: [PERFORMANCE_DESTINATION_V41.json](PERFORMANCE_DESTINATION_V41.json).

## Satu website lengkap

Tiga tab sumber masing-masing mempunyai 4.000 baris histori; seluruh tab tujuan dialokasikan 1.000 × 1.500. Engine menulis 22 sel dalam 16 operasi atomik dan rekonsiliasi sampai **SUCCESS**. Nilai, koordinat absolut, Notes, bank dan identitas transaksi cocok dengan golden V4. Offset fixture hanya ada di kode uji, bukan pemetaan produksi.

| Ukuran | Hasil |
|---|---:|
| Request sumber | 84 GET |
| Request tujuan | 1.219 GET + 16 POST = 1.235 |
| Semua request | 1.319 |
| Source-read | 216,44 ms |
| Seluruh proses offline | 884,42 ms |
| Respons terbesar | 22.409 B |
| Peak RSS proses | 129,41 MiB |
| OOM | Tidak |

Data mentah: [PERFORMANCE_WEBSITE_V41.json](PERFORMANCE_WEBSITE_V41.json).

**Durasi offline tidak memprediksi durasi VPS.** Pada interval request 1,2 detik, 1.319 request sendiri setara kira-kira 26,4 menit, sebelum latensi Google dan retry. Jumlah request dipengaruhi dimensi tab, jumlah item dan layout. Batching mempertahankan batas setiap rentang V4; global formula/struktur tetap dipindai. Tidak ada klaim run produksi selesai dalam satu detik.

## Batas dan metode memori

Probe Linux memakai **RLIMIT_AS 1536 MiB**, tanpa OOM. RLIMIT_AS membatasi address space; ini berbeda dengan cgroup systemd `MemoryMax=1536M` dan bukan bukti penerimaan VPS. Host pengujian mempunyai cgroup memory.max 32 GiB. Peak utama adalah `VmHWM` dari `/proc/self/status`; `ru_maxrss` mentah juga dilaporkan. Linux dapat mewariskan peak parent pytest ke child melalui fork/exec. Instrumentasi benchmark lama dikoreksi agar peak parent tidak disalahartikan sebagai peak reader. Ambang pengujian sumber tidak dilonggarkan dan kode source reader tidak berubah.

Memori dibatasi melalui:

- Rentang asli maksimal 8.000 sel, batch maksimal 64.000 sel dan 32 rentang. Respons over-cap dipecah iteratif menjadi kelompok rentang lebih kecil. Satu rentang yang masih over-cap gagal eksplisit.
- HTTP streaming tetap maksimal **8.388.608 B**; retry/timeout V4 tetap. Tidak ada kenaikan MemoryMax atau guard sumber.
- Indeks struktur SQLite temporer privat maksimal **64 MiB**, page cache 512 KiB, mmap nonaktif. Sel transaksi biasa dibuang; formula global diurutkan/hash lalu tabel staging dibuang.
- Cache aktual maksimal **128 sel / estimasi konservatif 4 MiB**. Sel besar tidak dicache. Sel target yang tidak diindeks tetap dibaca API, tidak dianggap kosong.
- Indeks/response ditutup pada sukses, error, cancel, pre-write, post-write dan setiap fresh report. Persistent DB tidak memakai PRAGMA indeks temporer.

Struktur/signature JSON sama dengan V4 pada fixture differential. Formula global memakai hash berurutan dari row/column/formula dan count. Merge global untuk sheetId yang tepat dipertahankan dengan urutan V4 di indeks disk. Metadata merge tidak dapat dipaginasi API Google. Vektor yang melampaui cap byte menghasilkan `MERGE_METADATA_LIMIT`; metadata yang tidak terbaca tidak dianggap tidak ada merge. Ini batas struktur aktual, bukan penolakan alokasi kosong.

Scan global dilakukan bertahap untuk mempertahankan invariant V4. Tidak ada penghentian discovery pada kolom tetap. Ukuran struktur nyata tetap mempunyai batas eksplisit dan gagal tanpa melewatkan record bila batas tercapai.

## Reproduksi dan live pending

Jalankan dari root paket dengan dependensi backend terpasang:

```bash
.venv/bin/python backend/tests/performance/measure_destination.py --width 1000
.venv/bin/python backend/tests/performance/measure_destination.py --width 1500
.venv/bin/python backend/tests/performance/measure_destination.py --width 1500 --formatted-blanks
.venv/bin/python backend/tests/performance/measure_v41_website.py --history 4000
.venv/bin/pytest backend/tests/test_v41_destination.py -q
```

Benchmark memakai batas address space 1536 MiB secara default. Google/workbook Run #7, quota/latensi, 43 item aktual, RSS systemd, merge payload aktual, instalasi dan rollback VPS **belum diuji live**. Ikuti [UPGRADE_V41.md](UPGRADE_V41.md) untuk PREVIEW VELBETT 18 Oktober lebih dahulu, tanpa replay Run #7 saat upgrade. Laporan source V4 tetap diarsipkan dalam [PERFORMANCE_V4.md](PERFORMANCE_V4.md).
