# Audit pembacaan dan pengujian memori V4

Pengujian lokal 23 Oktober 2026. Data sintetis; tidak ada akses key, dokumen Google asli atau VPS produksi. Hasil ini membuktikan pembacaan terbatas pada fixture yang diuji, belum membuktikan OOM produksi sudah definitif hilang.

## Jalur V3 yang diperiksa

`operations.py` membuat snapshot dan item untuk preview, individual, massal serta retry. Semuanya memakai `engine/runner.py::process_run`. Fungsi `source()` V3 memanggil `SheetsClient.grid()` sekali untuk tab aktif saat itu, kemudian `Day.resolve()` mencari pemisah tanggal. Deposit, withdrawal, saldo, input biaya DEPO, biaya QRIS, fee dan kiriman menggunakan Day yang sama. V3 **sudah** memakai cache tab; akar masalah bukan fetch terpisah untuk setiap metrik.

`sheets.py::grid()` V3 meminta metadata, kemudian `spreadsheets.get` dengan `ranges="'nama tab'"`, `includeGridData=true`, dan seluruh `effectiveFormat`, effective/user values, Notes serta formattedValue. Tidak ada batas rentang baris/kolom pada permintaan data. `requests.Response.json()` mematerialisasi body JSON, lalu loop membuat `Cell` untuk setiap sel yang dikembalikan. Sel yang kosong tetapi diformat tetap dapat muncul. Respons mentah bersarang dan salinan `Cell` hidup bersamaan selama konversi. Batas lama 2.000.000 sel metadata tidak membatasi byte respons atau jumlah objek format Python.

V3 membuang cache tab lama tetapi local `day` dapat tetap memegang objek terakhir saat pembacaan tab berikutnya. Rencana menyimpan referensi snapshot tujuan hingga akhir run. `writer.py` membaca grid tujuan lagi sebelum tulis dan sesudah tulis, untuk membandingkan struktur, semua formula, nilai dan Note. `reconciliation.py::fresh_balance()` membaca laporan secara fresh 2–5 kali agar nol stabil. Validasi/refresh konfigurasi hanya menggunakan metadata, tidak melakukan crawl finansial. Seluruh jalur ini diperiksa; tidak ditemukan penggunaan values.get/batchGet pada jalur sumber V3.

Bukti kode dibandingkan dengan ZIP V3 ber-SHA256 `2437bc71fd4369b4525700e3f3bdfd8239d650abfaa1ec5173b68b640c68bc65`. Log produksi pengguna membuktikan OOM saat SOURCE_READING; audit kode menunjukkan alokasi full-grid sebagai penyebab yang masuk akal dan dapat direproduksi sintetis. Ini bukan heap profile proses Run #5 yang telah mati; atribusi setiap byte pada insiden produksi belum tersedia.

## Pembacaan V4

1. Metadata ringan berisi sheet ID, judul dan ukuran grid.
2. `values:batchGet` dengan rentang kolom tunggal, 200 baris per chunk, maksimal 16 kolom per batch default. Tidak meminta format/Note pada penelusuran awal. Seluruh area diperiksa bertahap untuk membuktikan keunikan tanggal terpilih dan berikutnya, termasuk pemisah pada kolom yang tidak biasa. Hanya koordinat calon tanggal relevan ditahan; tiap respons nilai segera dibuang.
3. Format/merge untuk calon pemisah divalidasi lewat `spreadsheets.get` dengan maksimal 32 rentang sel tunggal dan field mask khusus. Serial tanggal numerik dikonfirmasi dengan numberFormat; angka uang tidak langsung dianggap tanggal. Tanggal lain di dalam interval juga diperiksa untuk menjaga penolakan urutan ambigu.
4. Hanya nilai B/C/D/E/F serta kolom biaya/kiriman/total/saldo yang diperlukan pada interval hari terpilih dibaca. DEPO tetap H/I/J/K/L; QRIS G/H/I/J/K. CEK SALDO dan BALANCE M DEPO tidak ikut menggantikan kolom saldo.
5. Metadata warna D dibaca hanya untuk calon deskripsi kiriman yang sudah cocok aturan V3. Warna aktual masih diklasifikasikan parser yang sama, termasuk biru yang dikecualikan dan cokelat yang disetujui.
6. Hari kosong mencari saldo ke belakang dalam chunk satu kolom dan berhenti pada nilai nonblank pertama. Nilai rusak tetap error. `SparseRows` menyimpan indeks baris absolut tanpa membuat puluhan ribu baris kosong, sehingga source_cell dan fingerprint kiriman tetap sama.
7. Anchor tanggal dibaca ulang setelah nilai/warna agar pergeseran baris yang teramati gagal dengan SOURCE_CHANGED. Tidak ada cache posisi lintas run; metadata sheetId berasal dari pembacaan aktual. Context milik satu run/WEB/tanggal/revisi/tab, dibagi antar modul, lalu dilepas.

Tidak ada server-side text-search fiktif. V4 sengaja tidak berhenti pada penemuan pertama: tanpa token revisi isi yang dapat dipercaya, itu dapat melewatkan duplikat pemisah tanggal di bawahnya dan mengubah keputusan V3. Tidak ada cache global atau penggantian aturan tanggal demi speed.

**Optimisasi yang dihentikan karena konflik batas:** membatasi discovery hanya ke A atau ke kolom marker tertentu belum aman untuk semantik V3, yang menerima separator hitam/merged di kolom mana pun. Karena itu discovery dapat membaca seluruh nilai histori secara bertahap, tanpa mematerialisasinya sebagai source grid. Larangan literal mengunduh semua nilai transaksi historis belum terpenuhi untuk tahap discovery; pembacaan payload finansial/format setelah discovery sudah dibatasi ke hari terpilih. Mempersempit penelusuran lebih lanjut memerlukan jaminan marker region yang berlaku pada semua dokumen, atau perubahan validasi tanggal yang tidak diizinkan dalam patch ini. Optimisasi tersebut dihentikan; tidak ada asumsi kolom marker baru yang diterapkan diam-diam.

Tujuan/laporan tetap memerlukan snapshot semua sel yang digunakan pemetaan dinamis dan pemeriksaan formula global. Snapshot ini dibatasi 250.000 sel, setiap permintaan maksimal 8.000 sel default, field format dibatasi warna/numberFormat, dan respons dibuang sesudah konversi. Formula, Note, semua website, batas tanggal dan merged ranges tetap disertakan. Writer dan resolver tidak diubah. Respons terkompresi dibaca streaming dengan batas **8 MiB sesudah dekompresi** sebelum json.loads, timeout eksplisit dan retry terbatas. Sumber/tujuan terlalu besar menjadi error, bukan pemotongan transaksi atau pelewatan proteksi.

## Pengukuran terkontrol

Data lengkap: [PERFORMANCE_V4.json](PERFORMANCE_V4.json). Setiap proses dibatasi **RLIMIT_AS 1536 MiB** di Linux. Ini batas address space subprocess, bukan pengujian cgroup systemd MemoryMax di VPS. Fixture API menghasilkan hanya rentang yang diminta; tidak menyimpan seluruh histori. Histori mencakup tanggal/merged markers, nilai keuangan dan area terformat; bagian relevan memakai fixture DEPO nyata secara struktur dengan transaksi, biaya dan kiriman. Varian terpilih 18 Oktober adalah hari kosong dengan saldo carry-forward.

| Jalur fixture | Baris histori | API calls | Baca sumber | Total probe | Peak RSS | Respons terbesar | Baris ditahan |
|---|---:|---:|---:|---:|---:|---:|---:|
| V3 full-grid | 4.000 | 2 | 2.074,01 ms | 2.075,45 ms | 265,84 MiB | 32.087.679 byte | 4.310 |
| V4 bounded | 4.000 | 27 | 80,28 ms | 81,48 ms | 45,22 MiB | 22.409 byte | 3 |
| V4 bounded | 40.000 | 207 | 604,04 ms | 605,28 ms | 44,11 MiB | 22.447 byte | 3 |

Ketiganya menghasilkan **0 / 0 / 2.355.000**, tanpa OOM. V4 tetap sekitar 45 MiB saat histori meningkat 10 kali. Peak RSS V4 pada kasus 4.000 baris sekitar 83% lebih rendah pada fixture ini. Baseline memakai loop alokasi grid V3 yang disalin untuk benchmark offline; tidak menjalankan aplikasi V3 pada produksi. Waktu mencakup generator/serialisasi fixture di proses lokal, tanpa jaringan, autentikasi, throttle 1,2 detik atau backoff Google. Waktu total di tabel adalah probe sumber, bukan durasi seluruh 43 item website.

Probe seluruh run V4 juga dijalankan di subprocess RLIMIT_AS 1536 MiB dengan **tiga tab sumber** besar, engine/SQLite/API konfigurasi sebenarnya, dan transport tujuan/laporan fixture terisolasi. [PERFORMANCE_WEBSITE_V4.json](PERFORMANCE_WEBSITE_V4.json) mencatat:

| Histori per tab | Source API calls | Baca sumber | Seluruh processing | Peak RSS | Output |
|---|---:|---:|---:|---:|---|
| 4.000 baris | 84 | 277,15 ms | 514,20 ms | 129,04 MiB | SUCCESS, 22 sel |
| 40.000 baris | 624 | 1.908,54 ms | 2.147,67 ms | 128,60 MiB | SUCCESS, 22 sel |

Keduanya mempertahankan nilai, alamat, deskripsi, Note dan bank golden output V3; 43 pembacaan snapshot tujuan fixture tetap dilakukan untuk proteksi/read-back/rekonsiliasi, tanpa OOM. Fingerprint memakai baris sumber absolut sesuai V3; benchmark yang menambah histori memang menggeser baris sumber. Tes golden terpisah dengan input identik memeriksa fingerprint juga. Angka ini belum mencakup 43 item dan seluruh workbook nyata Run #5. Tujuan fixture kecil; batas snapshot tujuan besar belum diukur live.

**Jumlah request total meningkat**, dari 2 respons besar menjadi chunk yang aman. V3 sudah satu snapshot per tab; mengurangi jumlah HTTP di bawah V3 sambil menjaga batas respons dan bukti keunikan global bertentangan dengan koreksi ini. V4 menghilangkan field/rentang berat yang tidak diperlukan dan tidak mengulang pembacaan per metrik; tidak mengklaim HTTP count total atau durasi live pasti turun. Dengan throttle yang ada, 27 request memerlukan setidaknya sekitar 31 detik interval, dan 207 request sekitar 4 menit, ditambah jaringan/parse. Perubahan cache posisi yang lebih agresif ditunda agar tidak mengubah keputusan finansial saat sumber berubah.

## Regresi dan batas bukti

Golden output dibuat dengan menjalankan **engine V3 dari ZIP yang sudah dikirim**, pada fixture lengkap DOC DEPO + dua DOC QRIS. V4 dibandingkan terhadap 22 cell writes: tab/alamat/nilai/deskripsi/Note/bank/fingerprint transaksi harus sama persis. Parser sumber, resolver tujuan/laporan, writer, angka dan rekonsiliasi juga identik byte demi byte terhadap V3. Fixture tambahan menguji hari aktif/kosong, carry-forward, beberapa fee, brown/blue, tanggal numerik, tanggal hilang/duplikat/ambigu, angka rusak, formula konflik, slot overflow, retry dan pembatalan.

Tes memori subprocess menguji pertumbuhan 52.000 → 520.000 sel historis dengan workload hari terpilih tetap, peak RSS <256 MiB dan pertumbuhan <32 MiB, respons terbesar <100.000 byte serta konteks <25 baris. Tes lain menguji respons HTTP diputus pada batas byte, metadata format sel tunggal, rentang values bounded, snapshot tujuan berbagi beberapa band, dan fail sebelum alokasi ketika batas terlampaui.

Kasus seluruh website dengan tab besar, Note panjang, ribuan merged ranges, formula Google aktual, latensi/throttle, serta beberapa siklus run dalam proses worker VPS **belum diukur live**. Default batas dapat menolak workbook berlebih secara aman. Tidak ada klaim worker lengkap selalu memakai 45 MiB; angka tersebut hanya probe sumber sintetis. Jalankan [UPGRADE_V4.md](UPGRADE_V4.md) dan simpan pengukuran live PREVIEW sebelum memberi izin tulis.

Reproduksi:

```bash
.venv/bin/python backend/tests/performance/measure_source.py --version v3 --history 4000 --limit-mib 1536
.venv/bin/python backend/tests/performance/measure_source.py --version v4 --history 4000 --limit-mib 1536
.venv/bin/python backend/tests/performance/measure_source.py --version v4 --history 40000 --limit-mib 1536
.venv/bin/python backend/tests/performance/measure_website.py --history 40000
.venv/bin/pytest backend/tests/test_v4_access.py -q
```
